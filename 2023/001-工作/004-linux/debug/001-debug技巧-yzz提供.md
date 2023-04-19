```
1. ps -aux | grep python
2. sudo cat /proc/PID/stack(main thread)
	1. sudo cat /proc/PID/tasks/*/stack(threads)
3. gdb attach
```

Example：
```
sudo cat /proc/26864/stack 
[<0>] futex_wait_queue_me+0xb9/0x120
[<0>] futex_wait+0x100/0x270
[<0>] do_futex+0x370/0x4e0
[<0>] __x64_sys_futex+0x13f/0x190
[<0>] do_syscall_64+0x57/0x190
[<0>] entry_SYSCALL_64_after_hwframe+0x44/0xa9
```

```
这是一个函数调用栈，描述了运行中的程序在执行futex相关操作时所调用的各个函数以及相应的内存地址偏移量：

第一行 [<0>] futex_wait_queue_me+0xb9/0x120：表示正在执行 futex 操作，具体地说是 futex_wait_queue_me 函数。该函数将当前进程添加到等待队列中，并将进程状态设置为“等待唤醒”。

第二行 [<0>] futex_wait+0x100/0x270：表示正在执行 futex 等待操作，具体地说是 futex_wait 函数。这个函数会进行自旋等待 futex 变量值的改变，如果等待超时则会通过 futex_wait_requeue_pi 或者 futex_wait_queue_me 函数进入睡眠状态。

第三行 [<0>] do_futex+0x370/0x4e0：表示正在执行 do_futex 操作。do_futex 是 Linux 内核中与 futex 相关联的系统调用之一，负责对 futex 进行操作，如等待、解除阻塞等。

第四行 [<0>] __x64_sys_futex+0x13f/0x190：表示正在执行 __x64_sys_futex 系统调用函数。这是用户空间程序使用 futex 的接口，主要提供针对 futex 的功能。

第五行 [<0>] do_syscall_64+0x57/0x190：表示正在执行 do_syscall_64 系统调用函数。该函数是进程与内核进行交互的接口，它会把需要执行的系统调用传递给内核。

最后一行 [<0>] entry_SYSCALL_64_after_hwframe+0x44/0xa9：表示进程正在从内核态返回用户态，这是发生在执行完系统调用后的过程，entry_SYSCALL_64_after_hwframe 函数负责恢复现场并返回用户程序。
```