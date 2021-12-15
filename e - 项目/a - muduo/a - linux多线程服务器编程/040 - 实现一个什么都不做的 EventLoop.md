

```c++
class EventLoop : boost::nocopyable{
public:
	EventLoop();
	~EventLoop();
	void loop();
	void assertInLoopThread(){
		if(!isInLoopThread()){
			abortNotInLoopThread();
		}
	}
	bool isInLoopThread() const {return threadId_ == CurrentThread::tid();}
private:
	void abortNotInLoopThread();
	bool looping_; // atomic
	const pid_t threadId_;
};

```
---

[[one loop per thread]]顾名思义每个线程只能有一个 EventLoop 对象，因此 EventLoop 的构造函数会检查当前线程是否已经创建了其他 EventLoop 对象，遇到错误就终止程序（LOG_FATAL)。EventLoop 的构造函数会记住本对象所属的线程（threadId_）。创建了 EventLoop 对象的线程是 IO 线程，其主要功能是运行事件循环 EventLoop :: loop ()。EventLoop 对象的生命期通常和其所属的线程一样长，它不必是 heap 对象。

---

```c++
__thread EventLoop* t_loopInThisThread = 0;
EventLoop::EventLoop()
	: looping_(false),
	  threadId_(CurrentThread::tid())
{
	LOG_TRACE << "EventLoop Created " << this<< " in thread " << threadId_;
	if(t_loopInThisThread){
		LOG_FATAL << "Another EventLoop " << t_loopInThisLoop << " exists in this thread" <<threadId_;	
	}
	else{
		t_loopInThisLoop = this;
	}
}

EventLoop::~EventLoop(){
	assert(!looping_);
	t_loopInThisLoop = NULL;
}
```

--- 

这里的 loop () 什么事都不做，等 5 秒就退出
```c++
void EventLoop::loop(){
	assert(!looping_);
	assertInLoopThread();
	looping_ = true;
	::poll(NULL,0,5*1000);
	LOG_TRACE << "EventLoop "<<this << "stop looping";
	looping_ = false;
}

```

---

### EventLoop 的改动

EventLoop class 新增了 quit () 成员函数，还加了几个数据成员，并在构造函数里初始化它们。注意 EventLoop 通过 [[scoped_ptr]] 来间接持有 Poller。
```c++
...
class EventLoop : boost::nocopyable{
public:
	...
private:
	void abortNotInLoopThread();
	bool looping_; // atomic
	const pid_t threadId_;

	... // add
	bool quit_;
	boost::scoped_ptr<Poller> poller_;
	ChannelList activeChannels_;
};

```

---

EventLoop :: loop () 有了真正的工作内容，它调用 [[Poller]] :: poll () 获得当前活动事件的 Channel 列表，然后依次调用每个 [[Channel]] 的 handleEvent () 函数。

---

```c++
void EventLoop::loop(){
	assert(!looping_);
	assertInLoopThread();
	looping_ = true;
	
	... // add
	quit_ = false;
	while(!quit_){
		activeChannels_.clear();
		poller_->poll(kPollTimeMs, &activeChannels_);
		for(const auto it = activeChannels_.begin(); it != activeChannels_.end(); ++it){
			(*it)->hanldeEvent();
		}
	}
	
	LOG_TRACE << "EventLoop "<<this << "stop looping";
	looping_ = false;
}

```

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