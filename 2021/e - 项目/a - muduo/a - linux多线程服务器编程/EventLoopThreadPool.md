用 [[one loop per thread]] 的思想实现[[多线程 TcpServer]] 的关键步骤是在新建 [[TcpConnection]] 时从 event loop pool 里挑选一个 loop 给 TcpConnection 用。也就是说多线程 TcpServer 自己的 [[EventLoop]] 只用来接受新连接，而新连接会用其他 EventLoop 来执行 IO。

> （单线程 TcpServer 的 EventLoop 是与 TcpConnection 共享的。）muduo 的 event loop pool 由 EventLoopThreadPool class 表示，接口如下，实现从略。

---

```c++

class EventLoopThreadPool : boost::nocopyable{
public:
	EventLoopThreadPool(EventLoop* baseloop);
	~EventLoopThreadPool();
	void setThreadNum(int numThreads) {numThreads_ = numThreads; };
	void start();
	EventLoop* getNextLoop();
	
private:
	EventLoop* baseloop_;
	bool started_;
	int numThreads_;
	int next_;
	boost::ptr_vector<EventLoopThread> threads_;
	std::vector<EventLoop*> loops_;
}
```

