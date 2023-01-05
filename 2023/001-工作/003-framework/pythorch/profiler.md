

> Pytorch profiler


> 学习流程：

1. 都看，官方介绍、外网介绍、代码啥的
2. 进展这种你可以先看个大概，比如说 profiler，先用起来，现在能用起来么？如果能的话，看看主要有什么功能是吧? 先用起来，至少 NV 上是可以用起来的是吧? 基础的用起来，有啥功能，能 profile 到什么程度?
3. 如果有外网讲解代码的，可以先看眼大概的组成，其次就是按照功能的重要性，去 debug，调试看代码为什么这么写，把你不懂的地方 debug 一遍看懂就好了? 有没有 testcase 之类的?
4. testcase 这种比较简单 debug 进去, 用 gdb 跟进去


> 学习预期

profiler 工具完全支持  
1. 熟悉 profiler 原理，使用清楚，预研下 KineTo 模式是否能支持。  
2. 精度对比工具优化。
3. 熟悉 prof，交付 pro 测试.


> 参考资料
1. 外网讲解代码：[PyTorch Community Voices | PyTorch Profiler | Sabrina & Geeta - YouTube](https://www.youtube.com/watch?v=m6ouC0XMYnc)
2. 官方介绍 : [PyTorch Profiler — PyTorch Tutorials 1.12.1+cu102 documentation](https://pytorch.org/tutorials/recipes/recipes/profiler_recipe.html)
3. 代码：


> 学习进展

23.1.5
1. 学习外网讲解代码 | doing
2. 学习 profiler recipe | doing

### Typical Steps to Optimize Your Model

1. Caputure Profile > Aggregate and analyze results > Optimize Model 
2. Caputure Profile > Aggregate and analyze results > Optimize Model 
3. Caputure Profile > Aggregate and analyze results > Optimize Model 
4. ...

### Five big futures 



### Usage

-   `activities` - a list of activities to profile:
    -   `ProfilerActivity.CPU` - PyTorch operators, TorchScript functions and user-defined code labels (see `record_function` below);
    -   `ProfilerActivity.CUDA` - on-device CUDA kernels;
-   `record_shapes` - whether to record shapes of the operator inputs;
-   `profile_memory` - whether to report amount of memory consumed by model’s Tensors;
-   `use_cuda` - whether to measure execution time of CUDA kernels.

1. how we can use profiler to analyze the execution time
```python
with profile(activities=[ProfilerActivity.CPU], record_shapes=True) as prof:
    with record_function("model_inference"):
        model(inputs)
```

> Profiler also automatically profiles the async tasks launched with `torch.jit._fork` and (in case of a backward pass) the backward pass operators launched with `backward()` call.
> Profiler 会自动分析异步任务和反向算子。







