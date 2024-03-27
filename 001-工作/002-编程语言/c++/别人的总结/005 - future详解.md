# 005 - [[future]] 详解


` <future>`  头文件中包含了以下几个类和函数：

* Providers 类：std::promise, std::package_task
* Futures 类：std::future, shared_future.
* Providers 函数：std::async()
* 其他类型：std::future_error, std::future_errc, std::future_status, std::launch.

这一头文件是异步编程 sync API 的调用基础。

[[006 - promise详解]]
[[007 - package_task详解]]