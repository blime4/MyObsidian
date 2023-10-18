[[one loop per thread + thread pool]]
重点理解：   [[998 - muduo的类图]] [[997 - muduo的时序图]]


创建了EventLoop对象的线程称为IO线程，其功能是运行事件循环（EventLoop::loop）

EventLoop对象构造的时候，会检查当前线程是否已经创建了其他EventLoop对象，如果已创建，终止程序（LOG_FATAL）  因为 one loop per thread 每个线程最多只能有一个EventLoop对象。

EventLoop 是对[[reactor]]的封装


具体类的学习步骤：
[[EventLoop]]
[[Poller]] 、[[Channel]]



