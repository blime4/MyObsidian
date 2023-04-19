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
```