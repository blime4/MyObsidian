
作为对[[reactor]]的封装

例子 1 ： [[040 - 实现一个什么都不做的 EventLoop]]

EventLoop 也提供 unregister 功能。EventLoop 新增了 removeChannel () 成员函数，它会调用 [[Poller]] :: removeChannel ()


[[简化版EventLoop]]