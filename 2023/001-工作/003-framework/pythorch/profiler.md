
> Pytorch profiler
---
#### 学习流程 ：

1. 都看，官方介绍、外网介绍、代码啥的
2. 进展这种你可以先看个大概，比如说 profiler，先用起来，现在能用起来么？如果能的话，看看主要有什么功能是吧? 先用起来，至少 NV 上是可以用起来的是吧? 基础的用起来，有啥功能，能 profile 到什么程度?
3. 如果有外网讲解代码的，可以先看眼大概的组成，其次就是按照功能的重要性，去 debug，调试看代码为什么这么写，把你不懂的地方 debug 一遍看懂就好了? 有没有 testcase 之类的?
4. testcase 这种比较简单 debug 进去, 用 gdb 跟进去
---

#### 学习预期

profiler 工具完全支持  
1. 熟悉 profiler 原理，使用清楚，预研下 KineTo 模式是否能支持。  
2. 精度对比工具优化。
3. 熟悉 prof，交付 pro 测试.

---

#### 参考资料
1. 外网讲解代码：[PyTorch Community Voices | PyTorch Profiler | Sabrina & Geeta - YouTube](https://www.youtube.com/watch?v=m6ouC0XMYnc)
2. 官方介绍 : [PyTorch Profiler — PyTorch Tutorials 1.12.1+cu102 documentation](https://pytorch.org/tutorials/recipes/recipes/profiler_recipe.html)
4. What's new in PyTorch Profiler 1.9： https://pytorch.org/blog/pytorch-profiler-1.9-released/ 
5. Introducing PyTorch Profiler :  https://pytorch.org/blog/introducing-pytorch-profiler-the-new-and-improved-performance-tool/ 
6. Profiler: https://pytorch.org/docs/stable/profiler.html 
7. Profiler Recipes: https://pytorch.org/tutorials/recipes/recipes/profiler.html 
8. VSCode TensorBoard support: https://devblogs.microsoft.com/python/python-in-visual-studio-code-february-2021-release/ 
9. GTC 2021-PROFILING PYTORCH MODELS FOR NVIDIA GPUS:  https://gtc21.event.nvidia.com/media/Profiling%20PyTorch%20Models%20for%20NVIDIA%20GPUs%20%5BS31644%5D/1_nuwnw731
10. Optimizing PyTorch Performance batch size with PyTorch Profiler: https://opendatascience.com/optimizing-pytorch-performance-batch-size-with-pytorch-profiler/ 
11. Kubeflow PyTorch Samples: https://github.com/kubeflow/pipelines/tree/master/samples/contrib/pytorch-samples 

---

#### 学习进展

23.1.5 - 23.1.6
1. 学习外网讲解代码 | done
2. 学习 profiler recipe | done

23.1.9 
1. 编译了 libkineto. a 试了试，如何编译 | done
2. 梳理了如何我们如果要支持 libkineto，可能需要支持的，必要的 31 个 api | done
3. 梳理 libkineto 的代码结构 | doing

23.1.10
1. 在 NV，DL 上编译 kineto, DL 是 cpu only 版本的。想看看 cpu only 的话，能否获得什么信息。与 GPU 相比还差 | delay
2. CUPTI 接口，梳理出来
3. 跑一下 NV +kineto 的 profiler ，获取一下数据，和之前的数据进行对比
	1. 问同事，找到我们之前的 profiler 数据
4. todo:
	1. 对比分析 kineto 提供了什么功能 | 目前是分析 42 和 43 f
	2. 分析 kineto 和 tensorboard 是如何结合起来的
	3. 统计我们真正需要什么的 CUPTI 接口
	4. 思考我们的 profilering 大概需要完成到什么功能
	5. 与 hc 讨论能支持什么我们的 CUPTI 的接口
5. 

---

### Typical Steps to Optimize Your Model

1. Caputure Profile > Aggregate and analyze results > Optimize Model 
2. Caputure Profile > Aggregate and analyze results > Optimize Model 
3. Caputure Profile > Aggregate and analyze results > Optimize Model 
4. ...

---

### Five big futures 
![[Pasted image 20230106112821.png]]

---

### Usage

-   `activities` - a list of activities to profile:
    -   `ProfilerActivity.CPU` - PyTorch operators, TorchScript functions and user-defined code labels (see `record_function` below);
    -   `ProfilerActivity.CUDA` - on-device CUDA kernels;
-   `record_shapes` - whether to record shapes of the operator inputs;
-   `profile_memory` - whether to report amount of memory consumed by model’s Tensors;
-   `use_cuda` - whether to measure execution time of CUDA kernels.

---

#### how we can use profiler to analyze the execution time、

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

---

#### Using profiler to analyze memory consumption
> To enable memory profiling functionality pass `profile_memory=True`.

```python
model = models.resnet18()
inputs = torch.randn(5, 3, 224, 224)

with profile(activities=[ProfilerActivity.CPU],
        profile_memory=True, record_shapes=True) as prof:
    model(inputs)

print(prof.key_averages().table(sort_by="self_cpu_memory_usage", row_limit=10))

# (omitting some columns)
# ---------------------------------  ------------  ------------  ------------
#                              Name       CPU Mem  Self CPU Mem    # of Calls
# ---------------------------------  ------------  ------------  ------------
#                       aten::empty      94.79 Mb      94.79 Mb           121
#     aten::max_pool2d_with_indices      11.48 Mb      11.48 Mb             1
#                       aten::addmm      19.53 Kb      19.53 Kb             1
#               aten::empty_strided         572 b         572 b            25
#                     aten::resize_         240 b         240 b             6
#                         aten::abs         480 b         240 b             4
#                         aten::add         160 b         160 b            20
#               aten::masked_select         120 b         112 b             1
#                          aten::ne         122 b          53 b             6
#                          aten::eq          60 b          30 b             2
# ---------------------------------  ------------  ------------  ------------
# Self CPU time total: 53.064ms

print(prof.key_averages().table(sort_by="cpu_memory_usage", row_limit=10))

# (omitting some columns)
# ---------------------------------  ------------  ------------  ------------
#                              Name       CPU Mem  Self CPU Mem    # of Calls
# ---------------------------------  ------------  ------------  ------------
#                       aten::empty      94.79 Mb      94.79 Mb           121
#                  aten::batch_norm      47.41 Mb           0 b            20
#      aten::_batch_norm_impl_index      47.41 Mb           0 b            20
#           aten::native_batch_norm      47.41 Mb           0 b            20
#                      aten::conv2d      47.37 Mb           0 b            20
#                 aten::convolution      47.37 Mb           0 b            20
#                aten::_convolution      47.37 Mb           0 b            20
#          aten::mkldnn_convolution      47.37 Mb           0 b            20
#                  aten::max_pool2d      11.48 Mb           0 b             1
#     aten::max_pool2d_with_indices      11.48 Mb      11.48 Mb             1
# ---------------------------------  ------------  ------------  ------------
# Self CPU time total: 53.064ms
```

---

#### Using tracing functionality
> Profiling results can be outputted as a . json trace file:

```python
model = models.resnet18().cuda()
inputs = torch.randn(5, 3, 224, 224).cuda()

with profile(activities=[ProfilerActivity.CPU, ProfilerActivity.CUDA]) as prof:
    model(inputs)

prof.export_chrome_trace("trace.json")

# (`chrome://tracing`):
```

---

#### Examining stack traces
> Profiler can be used to analyze Python and TorchScript stack traces:
```python
with profile(
    activities=[ProfilerActivity.CPU, ProfilerActivity.CUDA],
    with_stack=True,
) as prof:
    model(inputs)

# Print aggregated stats
print(prof.key_averages(group_by_stack_n=5).table(sort_by="self_cuda_time_total", row_limit=2))

# (omitting some columns)
# -------------------------  -----------------------------------------------------------
#                      Name  Source Location
# -------------------------  -----------------------------------------------------------
# aten::thnn_conv2d_forward  .../torch/nn/modules/conv.py(439): _conv_forward
#                            .../torch/nn/modules/conv.py(443): forward
#                            .../torch/nn/modules/module.py(1051): _call_impl
#                            .../site-packages/torchvision/models/resnet.py(63): forward
#                            .../torch/nn/modules/module.py(1051): _call_impl
#
# aten::thnn_conv2d_forward  .../torch/nn/modules/conv.py(439): _conv_forward
#                            .../torch/nn/modules/conv.py(443): forward
#                            .../torch/nn/modules/module.py(1051): _call_impl
#                            .../site-packages/torchvision/models/resnet.py(59): forward
#                            .../torch/nn/modules/module.py(1051): _call_impl
#
# -------------------------  -----------------------------------------------------------
# Self CPU time total: 34.016ms
# Self CUDA time total: 11.659ms

# Note the two convolutions and the two callsites in `torchvision/models/resnet.py` script.

# (Warning: stack tracing adds an extra profiling overhead.)

```

---

#### Visualizing data as a flamegraph | 目测没啥用

> Execution time (`self_cpu_time_total` and `self_cuda_time_total` metrics) and stack traces can also be visualized as a flame graph. To do this, first export the raw data using `export_stacks` (requires `with_stack=True`):

```python
prof.export_stacks("/tmp/profiler_stacks.txt", "self_cuda_time_total")
```

---

#### Using profiler to analyze long-running jobs
1. `schedule`
2. `on_trace_ready`
To illustrate how the API works, let’s first consider the following example with `torch.profiler.schedule` helper function:
```python
from torch. profiler import schedule

my_schedule = schedule(
    skip_first=10, # ignore the first 10 steps
    wait=5, # idling
    warmup=1, # during this phase profiler starts tracing, but the results are discarded;
    active=3, # profiler traces and records data
    repeat=2)
```
1. p. step () : To send the signal to the profiler that the next step has started,
2. p.step_num  : The current profiler step
```python
def trace_handler (p):
    output = p.key_averages().table(sort_by="self_cuda_time_total", row_limit=10)
    print(output)
    p.export_chrome_trace("/tmp/trace_" + str(p.step_num) + ".json")

with profile(
    activities=[ProfilerActivity.CPU, ProfilerActivity.CUDA],
    schedule=torch.profiler.schedule(
        wait=1,
        warmup=1,
        active=2),
    on_trace_ready=trace_handler
) as p:
    for idx in range(8):
        model(inputs)
        p.step()
```

---

## Pytorch Profiler Architecture
[[profiler架构分析]]
![[截图_20230106114600.png]]、

## Kineto

https://github.com/pytorch/kineto

enable:
1. **performance observability and diagnostics** across common ML bottleneck components
2. **actionable recommendations** for common issues
3. integration of external system-level profiling tools
4. integration with popular visualization platforms and analysis pipelines

### Libkineto
Libkineto is an in-process profiling library integrated with the PyTorch Profiler with special focus on low-overhead GPU timeline tracing.

> Currently only NVIDIA GPUs are supported.
> TODO： Check if we can supported
> 最基础的维度比说说就是计时间的，这个应该不需要这个CUPTI；再细粒度的profile你发的这个收集一些GPU的traces或者metrics，那可能就需要我们自己的sdk支持了，这个可能我们不支持，得确认

#### Dependencies

Libkineto requires gcc 5+ and:

-   NVIDIA CUPTI: used to collect traces and metrics from NVIDIA GPUs.
-   fmt: used for its convenient and lightweight string formatting functionality.
-   googletest: required to build and run Kineto's tests.
    -   **googletest is not required** if you don't want to run Kineto tests. By default, building of tests is **on**. Turn it off by setting `KINETO_BUILD_TESTS` to **off**.

You can download [NVIDIA CUPTI](https://developer.nvidia.com/CUPTI-CTK10_2), [fmt](https://github.com/fmt), [googletest](https://github.com/google/googletest) and set `CUDA_SOURCE_DIR`, `FMT_SOURCE_DIR`, `GOOGLETEST_SOURCE_DIR` respectively for cmake to find these libraries. If the fmt and googletest variables are not set, cmake will build the git submodules found in the `third_party` directory. If `CUDA_SOURCE_DIR` is not set, libkineto will fail to build.


#### Source Code

[kineto/CMakeLists.txt at main · pytorch/kineto (github.com)](https://github.com/pytorch/kineto/blob/main/libkineto/CMakeLists.txt)
```cmake
# function to extract filelists from libkineto_defs.bzl file

find_package(PythonInterp)
function(get_filelist name outputvar)
execute_process(
	COMMAND "${PYTHON_EXECUTABLE}" -c
	"exec(open('libkineto_defs.bzl').read());print(';'.join(${name}))"
	WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
	OUTPUT_VARIABLE _tempvar)
	string(REPLACE "\n" "" _tempvar "${_tempvar}")
	set(${outputvar} ${_tempvar} PARENT_SCOPE)
endfunction()

# Define file lists
if (LIBKINETO_NOCUPTI AND LIBKINETO_NOROCTRACER)
	get_filelist("get_libkineto_cpu_only_srcs(with_api=False)" LIBKINETO_SRCS)
	message(INFO " CUPTI unavailable or disabled - not building GPU profilers")
elseif(NOT LIBKINETO_NOROCTRACER)
	get_filelist("get_libkineto_roctracer_srcs(with_api=False)" LIBKINETO_SRCS)
	message(INFO " Building with roctracer")
else()
	get_filelist("get_libkineto_cupti_srcs(with_api=False)" LIBKINETO_SRCS)
endif()
get_filelist("get_libkineto_public_headers()" LIBKINETO_PUBLIC_HEADERS)
get_filelist("get_libkineto_api_srcs()" LIBKINETO_API_SRCS)
```

get_libkineto_cpu_only_srcs : 
```
[
	"src/AbstractConfig.cpp",
	"src/CuptiActivityProfiler.cpp",
	"src/ActivityProfilerController.cpp",
	"src/ActivityProfilerProxy.cpp",
	"src/ActivityType.cpp",
	"src/Config.cpp",
	"src/ConfigLoader.cpp",
	"src/CuptiActivityApi.cpp",
	"src/DaemonConfigLoader.cpp",
	"src/Demangle.cpp",
	"src/GenericTraceActivity.cpp",
	"src/ILoggerObserver.cpp",
	"src/IpcFabricConfigClient.cpp",
	"src/Logger.cpp",
	"src/init.cpp",
	"src/output_csv.cpp",
	"src/output_json.cpp",
]
```
get_libkineto_cupti_srcs:
```
[
	"src/CudaDeviceProperties.cpp",
	"src/CudaUtil.cpp",
	"src/CuptiActivityApi.cpp",
	"src/CuptiActivityPlatform.cpp",
	"src/CuptiCallbackApi.cpp",
	"src/CuptiEventApi.cpp",
	"src/CuptiMetricApi.cpp",
	"src/CuptiRangeProfiler.cpp",
	"src/CuptiRangeProfilerApi.cpp",
	"src/CuptiRangeProfilerConfig.cpp",
	"src/CuptiNvPerfMetric.cpp",
	"src/Demangle.cpp",
	"src/EventProfiler.cpp",
	"src/EventProfilerController.cpp",
	"src/WeakSymbols.cpp",
	"src/cupti_strings.cpp",
]
```
![[截图_20230106173638.png]]
get_libkineto_public_headers
```
[
	"include/AbstractConfig.h",
	"include/ActivityProfilerInterface.h",
	"include/ActivityTraceInterface.h",
	"include/ActivityType.h",
	"include/Config.h",
	"include/ClientInterface.h",
	"include/GenericTraceActivity.h",
	"include/IActivityProfiler.h",
	"include/ILoggerObserver.h",
	"include/ITraceActivity.h",
	"include/TraceSpan.h",
	"include/ThreadUtil.h",
	"include/libkineto.h",
	"include/time_since_epoch.h",
]

```
---

### CUPTI
![[Pasted image 20230109193657.png]]
#### 2.13. Samples
[CUPTI :: CUPTI Documentation (nvidia.com)](https://docs.nvidia.com/cupti/r_main.html#r_samples__cupti_correlation)

#### kineto 需要的一些 api
| kineto 需要的一些 api                  | 支持情况 |
| -------------------------------------- | -------- |
| cuptiActivityDisable                   | 不支持   |
| cuptiActivityEnable                    | 不支持   |
| cuptiActivityFlushAll                  | 不支持   |
| cuptiActivityGetNextRecord             | 不支持   |
| cuptiActivityGetNumDroppedRecords      | 不支持   |
| cuptiActivityPopExternalCorrelationId  | 不支持   |
| cuptiActivityPushExternalCorrelationId | 不支持   |
| cuptiActivityRegisterCallbacks         | 不支持   |
| cuptiDeviceEnumEventDomains            | 不支持   |
| cuptiDeviceGetEventDomainAttribute     | 不支持   |
| cuptiDeviceGetNumEventDomains          | 不支持   |
| cuptiEnableCallback                    | 不支持   |
| cuptiEventDomainGetAttribute           | 不支持   |
| cuptiEventGetAttribute                 | 不支持   |
| cuptiEventGetIdFromName                | 不支持   |
| cuptiEventGroupGetAttribute            | 不支持   |
| cuptiEventGroupReadEvent               | 不支持   |
| cuptiEventGroupSetAttribute            | 不支持   |
| cuptiEventGroupSetDisable              | 不支持   |
| cuptiEventGroupSetEnable               | 不支持   |
| cuptiEventGroupSetsCreate              | 不支持   |
| cuptiEventGroupSetsDestroy             | 不支持   |
| cuptiGetDeviceId                       | 不支持   |
| cuptiGetResultString                   | 不支持   |
| cuptiMetricEnumEvents                  | 不支持   |
| cuptiMetricGetAttribute                | 不支持   |
| cuptiMetricGetIdFromName               | 不支持   |
| cuptiMetricGetNumEvents                | 不支持   |
| cuptiMetricGetValue                    | 不支持   |
| cuptiSetEventCollectionMode            | 不支持   |
| cuptiSubscribe                         | 不支持   |

