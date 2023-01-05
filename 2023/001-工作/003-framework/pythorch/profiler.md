

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

```python
print(prof.key_averages().table(sort_by="cpu_time_total", row_limit=10))

# ---------------------------------  ------------  ------------  ------------  ------------
#                              Name      Self CPU     CPU total  CPU time avg    # of Calls
# ---------------------------------  ------------  ------------  ------------  ------------
#                   model_inference       5.509ms      57.503ms      57.503ms             1
#                      aten::conv2d     231.000us      31.931ms       1.597ms            20
#                 aten::convolution     250.000us      31.700ms       1.585ms            20
#                aten::_convolution     336.000us      31.450ms       1.573ms            20
#          aten::mkldnn_convolution      30.838ms      31.114ms       1.556ms            20
#                  aten::batch_norm     211.000us      14.693ms     734.650us            20
#      aten::_batch_norm_impl_index     319.000us      14.482ms     724.100us            20
#           aten::native_batch_norm       9.229ms      14.109ms     705.450us            20
#                        aten::mean     332.000us       2.631ms     125.286us            21
#                      aten::select       1.668ms       2.292ms       8.988us           255
# ---------------------------------  ------------  ------------  ------------  ------------
# Self CPU time total: 57.549ms
```

> To get a finer granularity (/grænjʊ'lærɪtɪ/ 细粒度) of results and include operator input shapes, pass `group_by_input_shape=True` (note: this requires running the profiler with `record_shapes=True`):

```python
print(prof.key_averages(group_by_input_shape=True).table(sort_by="cpu_time_total", row_limit=10))

# (omitting some columns)
# ---------------------------------  ------------  -------------------------------------------
#                              Name     CPU total                                 Input Shapes
# ---------------------------------  ------------  -------------------------------------------
#                   model_inference      57.503ms                                           []
#                      aten::conv2d       8.008ms      [5,64,56,56], [64,64,3,3], [], ..., []]
#                 aten::convolution       7.956ms     [[5,64,56,56], [64,64,3,3], [], ..., []]
#                aten::_convolution       7.909ms     [[5,64,56,56], [64,64,3,3], [], ..., []]
#          aten::mkldnn_convolution       7.834ms     [[5,64,56,56], [64,64,3,3], [], ..., []]
#                      aten::conv2d       6.332ms    [[5,512,7,7], [512,512,3,3], [], ..., []]
#                 aten::convolution       6.303ms    [[5,512,7,7], [512,512,3,3], [], ..., []]
#                aten::_convolution       6.273ms    [[5,512,7,7], [512,512,3,3], [], ..., []]
#          aten::mkldnn_convolution       6.233ms    [[5,512,7,7], [512,512,3,3], [], ..., []]
#                      aten::conv2d       4.751ms  [[5,256,14,14], [256,256,3,3], [], ..., []]
# ---------------------------------  ------------  -------------------------------------------
# Self CPU time total: 57.549ms
```

> Profiler can also be used to analyze performance of models executed on GPUs:

```python
model = models.resnet18().cuda()
inputs = torch.randn(5, 3, 224, 224).cuda()

with profile(activities=[
        ProfilerActivity.CPU, ProfilerActivity.CUDA], record_shapes=True) as prof:
    with record_function("model_inference"):
        model(inputs)

print(prof.key_averages().table(sort_by="cuda_time_total", row_limit=10))


# (omitting some columns)
# -------------------------------------------------------  ------------  ------------
#                                                    Name     Self CUDA    CUDA total
# -------------------------------------------------------  ------------  ------------
#                                         model_inference       0.000us      11.666ms
#                                            aten::conv2d       0.000us      10.484ms
#                                       aten::convolution       0.000us      10.484ms
#                                      aten::_convolution       0.000us      10.484ms
#                              aten::_convolution_nogroup       0.000us      10.484ms
#                                       aten::thnn_conv2d       0.000us      10.484ms
#                               aten::thnn_conv2d_forward      10.484ms      10.484ms
# void at::native::im2col_kernel<float>(long, float co...       3.844ms       3.844ms
#                                       sgemm_32x32x32_NN       3.206ms       3.206ms
#                                   sgemm_32x32x32_NN_vec       3.093ms       3.093ms
# -------------------------------------------------------  ------------  ------------
# Self CPU time total: 23.015ms
# Self CUDA time total: 11.666ms
```

2. 
