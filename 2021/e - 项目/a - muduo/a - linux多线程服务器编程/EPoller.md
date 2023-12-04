EPoller 的关键数据结构如下，其中 events_不是保存所有关注的 fd 列表，而是一次 [[epoll_wait]] (2) 调用返回的活动 fd 列表，它的大小是自适应的。

```c++

typedef std::vector<struct epoll_event> EventList;
typedef std::map<int,Channel*> ChannelMap;

int epollfd_; // ::epoll_create()
EventList events_;
ChannelMap channels_;

```

___

Struct epoll_event 的定义如下，注意 epoll_data 是个 [[union]]，muduo 使用的是其 ptr 成员，用于存放 Channel*，这样可以减少一步 look up。

```c++

typedef union epoll_data{
	void *ptr;
	int fd;
	uint32_t u32;
	uint64_t u64;
}epoll_data_t;

struct epoll_event{
	uint32_t events;
	epoll_data data;
}
```

---

为了减少转换，muduo Channel 没有自己定义 IO 事件的常量，而是直接使用 poll (2) 的定义（POLLIN、POLLOUT 等等），在 Linux 中它们和 epoll (4) 的常量相等。

![[Pasted image 20211217152335.png]]

---

### ==EPoller :: poll () 的关键代码如下。==

```c++
Timestamp EPoller::poll(int timeoutMs, ChannelList* activeChannels){
	int numEvents = ::epoll_wait(epollfd_, &*events_.begin(),static_cast<int>(events_.size()),timeoutMs);
	Timestamp now(Timestamp::now());
	if(numEvents > 0){
		LOG_TRACE << "...xxx";
		fillActiveChannels(numEvents, activeChannels);
		if(implicit_cast<size_t>(numEvents) == events_.size()){
			events_.resize(events_.size()*2);
			/*
			如果当前活动fd的数目填满了events_，那么下次就尝试接收更多的活动fd。events_的初始长度是16（kInitEventListSize），其会根据程序的IO繁忙程度自动增长，但目前不会自动收缩。
			*/
		}
	}
}
```

---

EPoller :: fillActiveChannels () 的功能是将 events_中的活动 fd 填入 activeChannels，其中 L90～L93 是在检查 invariant。

```c++
void EPoller::fillActiveChannels(int numEvents, ChannelList* activeChannels) const
{
	assert(implicit_cast<size_t>(numEvents)<=events_.size());
	for(int i = 0; i<numEvents;++i){
		Channel* channel = static_cast<Channel*>(events_[i].data.ptr);
		channel->set_revents(events_[i].events);
		activeChannels->push_back(channel);
	}
}
```