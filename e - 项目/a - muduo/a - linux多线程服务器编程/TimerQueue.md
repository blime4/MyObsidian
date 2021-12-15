Muduo 的定时器功能由三个 class 实现，TimerId、Timer、TimerQueue，用户只能看到第一个 class，另外两个都是内部实现细节。

TimerQueue 的接口很简单，只有两个函数 addTimer () 和 cancel ()。本节我们只实现 addTimer ()，cancel () 的实现见此处。addTimer () 是供 [[EventLoop]] 使用的，EventLoop 会把它封装为更好用的 runAt ()、runAfter ()、runEvery () 等函数。

---

```c++
class TimerQueue : boost::noncopyable
{
public:
	TimerQueue(EventLoop* loop);
	~TimerQueue();
	TimerId addTimer(const TimerCallback& cb, Timestamp when, double interval);
	
	...
private:
	...
}

```
---

值得一提的是 TimerQueue 的数据结构的选择，TimerQueue 需要高效地组织目前尚未到期的 Timer，能快速地根据当前时间找到已经到期的 Timer，也要能高效地添加和删除 Timer。

具体来说，以 pair<Timestamp, Timer*>为 key，这样即便两个 Timer 的到期时间相同，它们的地址也必定不同。

---

以下是 TimerQueue 的数据成员，这个结构利用了现成的容器库，实现简单，容易验证其正确性，并且性能也不错。TimerList 是 set 而非 map，因为只有 key 没有 value。TimerQueue 使用了一个 [[Channel]] 来观察 timerfd_上的 readable 事件。注意 TimerQueue 的成员函数只能在其所属的 IO 线程调用，因此不必加锁。

