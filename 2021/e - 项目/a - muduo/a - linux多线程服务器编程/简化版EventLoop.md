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

```c++

// 线程局部存储
__thread EventLoop* t_loopInThisThread = 0;
EventLoop::EventLoop()
	: looping_(false),
	  threadId_(CurrentThread::tid())
{
	LOG_TRACE << "EventLoop Created " << this<< " in thread " << threadId_;
	// one loop per thread的保证
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

[[one loop per thread]]顾名思义每个线程只能有一个 EventLoop 对象，因此 EventLoop 的构造函数会检查当前线程是否已经创建了其他 EventLoop 对象，遇到错误就终止程序（LOG_FATAL)。EventLoop 的构造函数会记住本对象所属的线程（threadId_）。创建了 EventLoop 对象的线程是 IO 线程，其主要功能是运行事件循环 EventLoop :: loop ()。EventLoop 对象的生命期通常和其所属的线程一样长，它不必是 heap 对象。

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