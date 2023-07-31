
# 1.8.0
https://pytorch.org/docs/1.8.0/jit.html?highlight=jit#creating-torchscript-code

## 区别 ：torch. jit. script 和 torch. jit. trace

在 PyTorch 中，`torch.jit.script` 和 `torch.jit.trace` 是两种用于将 PyTorch 模型转换为 Torch 脚本（TorchScript）的方法，以便在 PyTorch JIT（即时编译）中进行优化和加速。

1.  `torch.jit.script`： `torch.jit.script` 是一种方法，可以将普通的PyTorch代码（包括模型、函数或脚本）转换为TorchScript。TorchScript是一种中间表示（IR），类似于Python，但在执行时可以更高效地运行。使用`torch.jit.script`时，PyTorch会将代码转换为TorchScript，然后对其进行优化和编译，以提高性能和运行速度。这样，您可以在需要高性能或部署到其他平台的情况下使用TorchScript代码，而不需要依赖Python环境。