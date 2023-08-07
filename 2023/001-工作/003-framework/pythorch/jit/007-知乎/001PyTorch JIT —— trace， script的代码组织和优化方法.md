
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