

```c++

class Timer : boost::nocopyable{
public:
	Timer(const TimerCallback& cb, Timestamp when, double interval)
		: callback_(cb),
		  expiration_(when),
		  interval_(interval),
		  repeat_(interval > 0.0),
		  sequence_(s_numCreated_.incremntAndGet())
		  /*
		  保持现有的设计，让TimerId包含Timer*。但这是不够的，因为无法区分地址相同的先后两个Timer对象。因此每个Timer对象有一个全局递增的序列号int64_t sequence_（用原子计数器（AtomicInt64）生成），TimerId同时保存Timer*和sequence_，这样TimerQueue::cancel()就能根据TimerId找到需要注销的Timer对象。
		  */
	{
	}
private:

	const TimerCallback callback_;
	Timestamp expiration_;
	const double interval_;
	const bool repeat_;
	const int64_t sequence_;
	
	static AtomicInt64 s_numCreated_;
}
```

---

[[TimerQueue]]新增了cancel()接口函数，这个函数是[[线程安全]]的。