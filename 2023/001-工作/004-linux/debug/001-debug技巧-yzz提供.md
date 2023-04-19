```
1. ps -aux | grep python
2. sudo cat /proc/PID/stack(main thread)
	1. sudo cat /proc/PID/tasks/*/stack(threads)
3. gdb attach
```