# 040 - 实现一个什么都不做的 EventLoop

[[简化版EventLoop]]：
[[加入Poller后的EventLoop]]

---
[[041- Reactor模式的核心内容]]

---

我们现在可以终止事件循环，只要将 quit_设为 true 即可，但是 quit () 不是立刻发生的，它会在 EventLoop :: loop () 下一次检查 while (! quit_) 的时候起效（L53）。如果在非当前 IO 线程调用 quit ()，延迟可以长达数秒，将来我们可以唤醒 EventLoop 以缩小延时。但是 quit () 不是中断或 signal，而是设标志，如果 EventLoop :: loop () 正阻塞在某个调用中，quit () 不会立刻生效。

```c++
void EventLoop::quit(){
	quit_ = true;
}

```

---

EventLoop :: updateChannel () 在检查断言之后调用 Poller :: updateChannel ()，EventLoop 不关心 Poller 是如何管理 Channel 列表的。

```c++

void EventLoop::updateChannel(Channel* channel){
	assert(channel->ownerLoop() == this);
	assertInLoopThread();
	poller_ -> updateChannel(channel);
}
```

---

[[042 - 用timerfd实现了一个单次触发的定时器]]

---

### EventLoop 的改动 2

EventLoop 新增了几个方便用户使用的定时器接口，这几个函数都转而调用 TimerQueue :: addTimer ()。注意这几个 EventLoop 成员函数应该允许跨线程使用，比方说我想在某个 IO 线程中执行超时回调。这就带来线程安全性方面的问题，muduo 的解决办法不是加锁，而是把对 TimerQueue 的操作转移到 IO 线程来进行，这会用 EventLoop :: runInLoop () 函数。


```c++
TimerId EventLoop::runAt()(const Timestamp& time , const TimerCallback& cb){
	return timerQueue_->addTimer(cb,time,0.0);
}
TimerId EventLoop::runAfter(double delay, const TimerCallback& cb){
	Timestamp time(addTime(Timestamp::now(),delay));
	return runAt(time,cb);
}
TimerId EventLoop::runEvery(double interval, const TimerCallback& cb){
	Timestamp time(addTime(Timestamp::now(),interval));
	return timerQueue_->addTimer(cb,time,interval);
}

```

---

EventLoop :: runInLoop () 函数

EventLoop 有一个非常有用的功能：在它的 [[IO 线程]]内执行某个用户任务回调，即 EventLoop :: runInLoop (const Functor& cb)，其中 Functor 是 boost :: function<void ()>。如果用户在当前 IO 线程调用这个函数，回调会同步进行；如果用户在其他线程调用 runInLoop ()，cb 会被加入队列，IO 线程会被唤醒来调用这个 Functor。

---

```c++
void EventLoop::runInLoop(const Functor& cb){
	if(isInLoopThread){
		cb();
	} else {
		queueInLoop(cb);
	}
}
```

---

有了这个功能，我们就能轻易地在线程间调配任务，比方说把 TimerQueue 的成员函数调用移到其 IO 线程，这样可以在不用锁的情况下保证[[线程安全]]性。

QueueInLoop () 的实现很简单，将 cb 放入队列，并在必要时唤醒 IO 线程。
```c++
void EventLoop::queueInLoop(const Functor& cb){
	{
		MutexLockGuard lock(mutex_);
		pendingFunctors_.push_back(cb);
	}
	if(!isInLoopThread()||callingPendingFunctors_){
		wakeup();
	}
}
```


