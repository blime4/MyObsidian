
cuBLAS Library for Tensor Operations

`libcublasLt.so.11` 是 NVIDIA CUDA 库之一，**它是基于 `libcublas.so.11` 的高层 API**，**提供了高级的矩阵乘法运算的计算优化和自动化**。`libcublasLt.so.11` 的主要作用是提高矩阵乘法计算的性能和效率，特别是在大规模矩阵计算和深度学习中的应用。

`libcublasLt.so.11`库中提供了一个可编程的矩阵乘法调度程序，可以自动优化矩阵乘法的算法选择和计算参数设置，以达到最优的性能和效率。此外，`libcublasLt.so.11`库还提供了一些高级的功能，如[[矩阵操作分类]]、[[批处理]]、[[多GPU并行]]等，可以进一步提升矩阵乘法计算的效率和性能。

与 `libcublas.so.11` 库相比，`libcublasLt.so.11` 库的**优势在于它提供了自动优化的算法和计算参数，能够更好地适应不同的硬件和软件环境，从而获得更高的计算效率。**

同时，由于`libcublasLt.so.11`库是基于`libcublas.so.11`库的高层API，因此也能够直接使用`libcublas.so.11`库中的API，并具备相似的调用方式和参数。因此，对于需要大规模矩阵计算和深度学习应用的用户来说，`libcublasLt.so.11`库是一个非常有用的工具。