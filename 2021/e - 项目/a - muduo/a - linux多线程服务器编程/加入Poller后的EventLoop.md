
[[简化版EventLoop]]
### EventLoop 的改动

EventLoop class 新增了 quit () 成员函数，还加了几个数据成员，并在构造函数里初始化它们。注意 EventLoop 通过 [[scoped_ptr]] 来间接持有 Poller。
```c++
...
class EventLoop : boost::nocopyable{
public:
	...
	void updateChannel(Channel* channel);
	void removeChannel(Channel* channel);
private:
	void abortNotInLoopThread();
	bool looping_; // atomic
	const pid_t threadId_;

	... // add
	bool quit_;    // 事件退出 atomic
	bool eventHandling_; // 当前是否处于事件处理的状态
	TimeStamp pollReturnTime; 
	boost::scoped_ptr<Poller> poller_; // Poller的智能指针，管理Poller的生命周期。
	ChannelList activeChannels_;       // Poller返回的活动通道
	Channel* currentActiveChannel_;    // 当前正在处理的活动通道
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
		// 活动通道清除
		activeChannels_.clear();
		// 调用Poller得到当前的活动通道
		poller_->poll(kPollTimeMs, &activeChannels_);
		for(const auto it = activeChannels_.begin(); it != activeChannels_.end(); ++it){
			(*it)->hanldeEvent(); //调用handleEvent处理通道
		}
	}
	
	LOG_TRACE << "EventLoop "<<this << "stop looping";
	looping_ = false;
}

```