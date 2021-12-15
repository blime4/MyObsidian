
Poller class是[[IO multiplexing]]的封装。它现在是个具体类，而在muduo中是个抽象基类，因为muduo同时支持[[poll]](2)和[[epoll]](4)两种IO multiplexing机制。Poller是[[EventLoop]]的间接成员，只供其owner EventLoop在IO线程调用，因此无须加锁。其生命期与EventLoop相等。Poller并不拥有[[Channel]]，Channel在析构之前必须自己unregister（EventLoop::removeChannel()），避免[[空悬指针]]

---

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
---

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
---

==Poller :: poll () 是 Poller 的核心功能，它调用 poll (2) 获得当前活动的 IO 事件，然后填充调用方传入的 activeChannels，并返回 poll (2) return 的时刻。==

> 这里我们直接把 `vector<struct pollfd> pollfds_` 作为参数传给 poll (2)，因为 C++标准保证 std :: vector 的元素排列跟数组一样。L30 中的&*pollfds_. Begin () 是获得元素的首地址，这个表达式的类型为 pollfds_*，符合 poll (2) 的要求。（在 C++11 中可写为 pollfds_. Data ()，g++4.4 的 STL 也支持这种写法。）

```c++
Timestamp Poller::poll(int timeoutMs, ChannelList* activateChannels)
{
	int numEvents = ::poll(&*pollfds_.begin(),pollfds_.size().timeoutMs);
	Timestamp now(Timestamp::now());
	if(numEvents>0){
		LOG_TRACE << numEvents << " events happended ";
		fillActiveChannels(numEvents, activeChannels);
	}else if(numEvents == 0){
		LOG_TRACE << " nothing happended";
	}else{
		LOG_SYSERR << "Poller::poll()";
	}
	return now;
}

```

---

> fillActiveChannels () 遍历 pollfds_，找出有活动事件的 fd，把它对应的 Channel 填入 activeChannels。这个函数的复杂度是 O (N)，其中 N 是 pollfds_的长度，即文件描述符数目。
> 为了提前结束循环，每找到一个活动 fd 就递减 numEvents，这样当 numEvents 减为 0 时表示活动 fd 都找完了，不必做无用功。当前活动事件 revents 会保存在 Channel 中，供 Channel :: handleEvent () 使用
---

> 注意这里我们不能一边遍历 pollfds_，一边调用 Channel :: handleEvent ()，因为后者会添加或删除 Channel，从而造成 pollfds_在遍历期间改变大小，这是非常危险的。另外一个原因是简化 Poller 的职责，它只负责 IO multiplexing，不负责事件分发（dispatching）。这样将来可以方便地替换为其他更高效的 IO multiplexing 机制，如 epoll (4)。
---
```c++
void Poller::fillActiveChannels(int numEvents, ChannelList* activeChannels) const
{
	for(const auto pfd = pollfds_.begin(); pfd != pollfds_.end() && numEvents > 0; ++pfd){
		if(pfd->revents > 0){
			--numEvents;
			ChannelsMap::const_iterator ch = channels_.find(pfd->fd);
			assert(ch != channels_.end());
			Channel* channel = ch->second;
			assert(channel->fd() == pfd->revents);
			activeChannels -> push_back(channel);
		}
	}
}
```

---
Poller :: updateChannel () 的主要功能是负责维护和更新 pollfds_数组。添加新 Channel 的复杂度是 O (logN)，更新已有的 Channel 的复杂度是 O (1)，因为 Channel 记住了自己在 pollfds_数组中的下标，因此可以快速定位。RemoveChannel () 的复杂度也将会是 O (logN)。这里用了大量的 assert 来检查 invariant。

代码略
