[[倒计时]]（CountDownLatch） 是一种常用且易用的同步手段。它主要有两种用途：

+ 主线程发起多个子线程，等这些子线程各自都完成一定的任务之后，主线程才继续执行。通常用于主线程等待多个子线程完成初始化。
+ 主线程发起多个子线程，子线程都等待主线程，主线程完成其他一些任务之后通知所有子线程开始执行。通常用于多个子线程等待主线程发出“起跑”命令。



```c++
class CountDownLatch{
public:
	CountDownLatch(int count)
		: mutex_(),
		  condition_(mutex_),
		  count_(count)
	{}
private:
	mutable MutexLock mutex_;
	Condition condition_;
	int count_;
}
```

