
Poller class是[[IO multiplexing]]的封装。它现在是个具体类，而在muduo中是个抽象基类，因为muduo同时支持[[poll]](2)和[[epoll]](4)两种IO multiplexing机制。Poller是[[EventLoop]]的间接成员，只供其owner EventLoop在IO线程调用，因此无须加锁。其生命期与EventLoop相等。Poller并不拥有[[Channel]]，Channel在析构之前必须自己unregister（EventLoop::removeChannel()），避免[[空悬指针]]

```c++

class Poller : boost::nocopyable{

public:
	typedef std::vector<Channel*> ChannelList;
	Poller(EventLoop* loop);
	~Poller();
	
	Timestamp poll(int timeoutMs, ChannelList* activateChannels);
	void updateChannel(Channel* channel);
	
	void assertInLoopThread(){ ownerLoop_ -> assertInLoopThread();}
private:
	void fillActiveChannels(int numEvents, ChannelList* activeChannels) const;
	typedef std::vector<struct pollfd> PollFdList;
	typedef std::map<int,Channel*> ChannelMap;
	
	EventLoop* ownerLoop_;
	PollFdList pollfds_;
	ChannelMap channels_;
}

```

Poller 的构造函数和析构函数都很简单，因其成员都是标准库容器。
```c++

Poller::Poller(EventLoop* loop)
	:ownerLoop_(loop)
{
}
Poller::~Poller()
{
}
```