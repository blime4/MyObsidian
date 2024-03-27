
`TORCH_CUDA_CU_API` 是 PyTorch 框架中的一个宏定义，它指示 PyTorch 在运行时使用 CUDA Driver API（CU API）而不是 CUDA Runtime API（CUDART API）。

在使用 CUDA 进行 GPU 加速时，开发人员可以使用两种不同的 API 接口：CUDA Driver API 和 CUDA Runtime API。**CUDA Driver API 提供了更多的底层控制和灵活性，但需要更多的代码和更复杂的实现；**而 CUDA Runtime API 则是更高级的 API，它提供了更简洁的代码和更易于使用的接口，但是缺少一些底层控制和灵活性。

在 PyTorch 中，`TORCH_CUDA_CU_API` 宏定义的作用是指示 PyTorch 在运行时使用 CUDA Driver API，而不是默认的 CUDA Runtime API。这个宏定义通常用于解决一些 CUDA Driver API 和 CUDA Runtime API 的兼容性问题，或者在某些情况下使用 CUDA Driver API 能够提高性能。