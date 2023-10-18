

每个 Channel 对象自始至终只属于一个 [[EventLoop]]，因此每个 Channel 对象都只属于某一个 [[IO 线程]]。每个 Channel 对象自始至终只负责一个[[文件描述符]]（fd）的 IO 事件分发，但它并不拥有这个 fd，也不会在析构的时候关闭这个 fd。Channel 会把不同的 IO 事件分发为不同的回调，例如 ReadCallback、WriteCallback 等，而且“回调”用 boost :: function 表示，用户无须继承 Channel，Channel 不是基类。==muduo 用户一般不直接使用 Channel，而会使用更上层的封装，如 [[TcpConnection]]。==Channel 的生命期由其 owner class 负责管理，它一般是其他 class 的直接或间接成员。以下是 Channel 的 public interface：

```c++

class Channel : boost::nocopyable{
public:
	typedef boost::function<void()> EventCallback;
	Channel(EventLoop* loop,int fd);
	void handleEvent();
	void setReadCallback(const EventCallback& cb){readCallback_ = cb;}
	void setWriteCallback(const EventCallback& cb){writeCallback_ = cb;}
	void setErrorCallback(const EventCallback& cb){errorCallback_ = cb;}
	int fd() const {return fd_;}
	int events() const {return events_;}
	...
	void enableReading( events_ |= kReadEvent;update();)
	...
private:
	void update();
	static const int KNoneEvent;
	static const int KReadEvent;
	static const int KWriteEvent;
	
	EventLoop* loop_;
	const int fd_;
	int events_;
	int revents_;
	int index_; 	// used by Poller
	EventCallback readCallback_;
	EventCallback writeCallback_;
	EventCallback errorCallback_;
}
```

其中events_是它关心的IO事件，由用户设置；revents_是目前活动的事件，由EventLoop/Poller设置；这两个字段都是bit pattern，它们的名字来自poll(2)的struct pollfd。

==Channel :: [[handleEvent]] () 是 Channel 的核心;
它由 EventLoop :: loop () 调用，它的功能是根据 revents_的值分别调用不同的用户回调。==


[[完整的Channel]]
