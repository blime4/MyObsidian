在 PyTorch 中，默认情况下会使用 CUDA Runtime API，这个默认行为的宏定义是 `TORCH_CUDA_CUDART`。

与 `TORCH_CUDA_CU_API` 指示 PyTorch 使用 CUDA Driver API 不同，`TORCH_CUDA_CUDART` 宏定义指示 PyTorch 在运行时使用 CUDA Runtime API。**这个宏定义通常不需要显式地设置，因为它是默认行为**。如果你想使用 CUDA Runtime API，可以在代码中省略这个宏定义，或者显式地设置它为 1。

需要注意的是，如果你要使用 CUDA Runtime API，你需要在编译 PyTorch 时包含 CUDA Runtime API 的头文件和库文件。如果你使用的是 PyTorch 的预编译二进制版本，通常已经包含了这些文件，你不需要额外安装。