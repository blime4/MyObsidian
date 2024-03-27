
[PyTorch系列「一」PyTorch JIT —— trace/ script的代码组织和优化方法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/410507557)

## trace 和 script 的区别：

1、trace只记录走过的tensor和对tensor的操作，不会记录任何控制流信息，如if条件句和循环。因为没有记录控制流的另外的路，也没办法对其进行优化。好处是trace深度嵌入python语言，复用了所有python的语法，在计算流中记录数据流。

2、script 会去理解所有的 code，真正像一个编译器一样去进行 lexer、parser、Semantic analusis 的分析「也就是词法分析语法分析句法分析，形成 AST 树，最后再将 AST 树线性化」。script 相当于一个嵌入在 Python/Pytorch 的 DSL，其语法只是 pytorch 语法的子集，这意味着存在一些 op 和语法 script 不支持，这样在编译的时候就会遇到问题。此外，script 的编译优化方式更像是 CPU 上的传统编译优化，重点对于图进行硬件无关优化，并对 IF、loop 这样的 statement 进行优化。

解决方法是将这两种方式混合使用：
```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class MyModule(nn.Module):
    def __init__(self):
        super(MyModule, self).__init__()
        # torch.jit.trace produces a ScriptModule's conv1 and conv2
        self.conv1 = torch.jit.trace(nn.Conv2d(1, 20, 5), torch.rand(1, 1, 16, 16))
        self.conv2 = torch.jit.trace(nn.Conv2d(20, 20, 5), torch.rand(1, 20, 16, 16))

    def forward(self, input):
        input = F.relu(self.conv1(input))
        input = F.relu(self.conv2(input))
        return input

scripted_module = torch.jit.script(MyModule())
```


## PyTorch Trace 源码浅析

走一遍 forward，然后把计算 node insert 到 graph 中。

1. python 调用 jit. trace 方法，输入是一个 `func` +一个 `example inputs`，输出是一个 `scriptModule` return an `executable` or `class::ScriptFunction`

	- The resulting recording of a standalone function produces `ScriptFunction`.
	- The resulting recording of `nn.Module.forward` or `nn.Module` produces  
	    `ScriptModule`.
```python
# python接口，输入python::function和example_inputs，进行trace
torch.jit.trace(
    func,  # (callable or torch.nn.Module) – function 或者 torch.nn.Module
    example_inputs, # (tuple or torch.Tensor) – tracing时作为例子的输入
    ...
)
```

```python
# trace内包一层_create_function_from_trace  
def trace(
    func,
    example_inputs,
    optimize=None,
    check_trace=True,
    check_inputs=None,
    check_tolerance=1e-5,
    strict=True,
    _force_outplace=False,
    _module_class=None,
    _compilation_unit=_python_cu,
):

    var_lookup_fn = _create_interpreter_name_lookup_fn(0)
    traced = torch._C._create_function_from_trace(
        name, func, example_inputs, var_lookup_fn, strict, _force_outplace
    )
    # 通过torch._C进行C++代码对func进行trace
```
C++部分：
```cpp
// jit/python/script_init.cpp    

// _create_function_from_trace是Python和C++的接口
// 建图、创建compilation_unit、创建返回的函数

m.def(
      "_create_function_from_trace",
      [](const std::string& qualname,
         const py::function& func,
         const py::tuple& input_tuple,
         const py::function& var_lookup_fn,
         bool strict,
         bool force_outplace) {
        auto typed_inputs = toTraceableStack(input_tuple);
        std::shared_ptr<Graph> graph = std::get<0>(tracer::createGraphByTracing(
            func, typed_inputs, var_lookup_fn, strict, force_outplace));

        auto cu = get_python_cu();
        auto name = c10::QualifiedName(qualname);
        auto result = cu->create_function(
            std::move(name), std::move(graph), /*shouldMangle=*/true);
        StrongFunctionPtr ret(std::move(cu), result);
        didFinishEmitFunction(ret);
        return ret;
      });


建图：这个过程是建图核心，这个闭包值得好好研究一下
闭包做的事情就是将inputs py::cast之后，给到func进行out的验算
那么哪里调用了真实的insert op node？


tracer::createGraphByTracing
 auto outs = tracer::trace(
      std::move(trace_inputs),
      [&func](Stack inputs) -> Stack {
        size_t num_func_inputs = inputs.size();
        py::tuple py_inputs(num_func_inputs);
        for (size_t i = 0; i < num_func_inputs; ++i) {
          py_inputs[i] = py::cast(inputs[i]);
        }
          
        auto out = func(*py_inputs);  //////////答案就在这里！
          
        if (out.ptr() == Py_None) {
          AT_ERROR(
              "The traced function didn't return any values! Side-effects are not "
              "captured in traces, so it would be a no-op.");
        }
        return {toTypeInferredIValue(out)};
      },
      lookup_fn_adapter,
      strict,
      force_outplace,
      self);

首先还是先进入到trace函数，看下trace具体做的事情
注意上一层是把func作为函数指针(std::function)的方式传到了trace函数中

1. set一个TracingState实例，所有状态都在这里面了，这个步骤相当于初始化
2. 遍历一下input并进行建图
3. 好，核心在invoke function，也就是上一层传下来的函数指针，auto out_stack = traced_fn(inputs);走完这一行，整个图已经建好，只差输出
4. 把输出再放到图上就行
5. 扫尾：inline -> FixupTraceScopeBlocks -> NormalizeOps

std::pair<std::shared_ptr<TracingState>, Stack> trace(
    Stack inputs,
    const std::function<Stack(Stack)>& traced_fn,
    std::function<std::string(const Variable&)> var_name_lookup_fn,
    bool strict,
    bool force_outplace,
    Module* self) {
  try {
    // Start tracing, treating 'inputs' as inputs to the trace, which can be
    // varied on subsequent invocations of the trace.  Any other variables
    // will be treated as constants.
    if (isTracing()) {
      AT_ERROR("Tracing can't be nested");
    }
      
    auto state = std::make_shared<TracingState>();
      # setTracingState 将state 这个实例set下来，在之后计算节点get出来insert计算过程
    
      setTracingState(state);

    // if we are a module, then make sure the modules parameters are in the map
    // and mapped to accesses to the self object
    if (self) {
      Value* self_value = state->graph->insertInput(0, "self")->setType(
          self->_ivalue()->type());
      gatherParametersAndBuffers(state, self_value, *self, {"__module"});
    }

    for (IValue& input : inputs) {
      input = addInput(state, input, input.type(), state->graph->addInput());
    }
    auto graph = state->graph;

    getTracingState()->lookup_var_name_fn = std::move(var_name_lookup_fn);
    getTracingState()->strict = strict;
    getTracingState()->force_outplace = force_outplace;

    // Invoke the traced function
    auto out_stack = traced_fn(inputs);

    // Exit a trace, treating 'out_stack' as the outputs of the trace.  These
    // are the variables whose values will be computed upon subsequent
    // invocations of the trace.
    size_t i = 0;
    for (auto& output : out_stack) {
      // NB: The stack is in "reverse" order, so when we pass the diagnostic
      // number we need to flip it based on size.
      state->graph->registerOutput(
          state->getOutput(output, out_stack.size() - i));
      i++;
    }
    setTracingState(nullptr);

    if (getInlineEverythingMode()) {
      Inline(*graph);
    }
    FixupTraceScopeBlocks(graph, self);
    NormalizeOps(graph);
    return {state, out_stack};
  } catch (...) {
    tracer::abandon();
    throw;
  }
}
```