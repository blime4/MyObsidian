
具体来说，NVRTC 是一个运行时编译器，可以在运行时将 CUDA 代码编译成可执行的二进制代码，以便在 GPU 上运行。PyTorch JIT 编译器中包含了对 NVRTC 的支持，可以使用 NVRTC 编译包含 CUDA 操作的 TorchScript 代码，从而实现对 CUDA 的支持。

![[Pasted image 20230216144622.png]]

