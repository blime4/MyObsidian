[[EventLoopThreadPool]]

[[TcpServer]]的改动

```c++

class TcpServer : boost::noncopyable{

public:
    TcpServer(EventLoop* loop, const InetAddress& listenAddr);
    ~TcpServer();
    
    void start();
    void setConnectionCallback(const ConnectionCallback& cb){ connectionCallback_ = cb; }
    void setMessageCallback(const MessageCallback& cb){ messageCallback_ = cb; }
	
	...
	void setThreadNum(int numThreads);
	
	
private:
    
    void newConnection(int sockfd, const InetAddress& peerAddr);
    typedef std::map<std::string, TcpConnectionPtr> ConnectionMap;
    EventLoop* loop_;                           // the acceptor loop
    const std::string name_;
    boost::scoped_ptr<Acceptor> acceptor_;      // avoid revealing Acceptor
    ConnectionCallback connectionCallback_;
    MessageCallback messageCallback_;
    bool started_;
    int nextConnId_;                            // always in loop thread
    ConnectionMap connections_;
	
	...
	// thread safe
	void removeConnection(const TcpConnectionPtr& conn);
	// not thread safe, but in loop
	void removeConnectionInLoop(const TcpConnectionPtr& conn);
	boost::scoped_ptr<EventLoopThreadPool> threadPoll_;
}


```

多线程 TcpServer 的改动很简单，新建连接只改了 3 行代码。原来是把 TcpServer 自用的 loop_传给 [[TcpConnection]]，现在是每次从 EventLoopThreadPool 取得 `ioLoop`。

---
```c++
void TcpServer::newConnection(int sockfd, const InetAddress& peerAddr){
    
    InetAddress localAddr(sockets::getLocalAddr(sockfd));
	
	... 
	EventLoop* ioloop = threadPool_->getNextloop();
	
    TcpConnectionPtr conn(
        new TcpConnection(loop_,connName,sockfd,localAddr,peerAddr));
    connections_[connName] = conn;
    conn->setConnectionCallback(connectionCallback_);
    conn->setMessageCallback(messageCallback_);
	
	...
	//unsafe
    conn->setCloseCallback(boost::bind(&TcpServer::removeConnection, this, _1));
	ioloop->runInLoop(boost::bind(&TcpConnection::connectEstablished, conn))
	
	
}

```

---

连接的销毁也不复杂，把原来的 removeConnection () 拆为两个函数，因为 TcpConnection 会在自己的 ioLoop 线程调用 removeConnection ()，所以需要把它移到 TcpServer 的 loop_线程（因为 TcpServer 是无锁的）。L98 再次把 connectDestroyed () 移到 TcpConnection 的 ioLoop 线程进行，是为了保证 TcpConnection 的 ConnectionCallback 始终在其 ioLoop 回调，方便客户端代码的编写。

```c++


void TcpServer::removeConnection(const TcpConnectionPtr& conn){
	...
	loop_->runInLoop(boost::bind(&TcpServer::removeConnectionInLoop, this, conn));
}


void TcpServer::removeConnectionInLoop(const TcpConnectionPtr& conn){
	loop_->assertInLoopThread();
	LOG_INFO << "...xxx";
	size_t n = connections_.erase(conn->name());
	assert(n == 1); (void)n;
	
	...
	EventLoop* ioLoop = conn->getLoop();
	ioLoop->queueInLoop(boost::bind(&TcpConnection::connectDestoryed,conn));
}

```


总而言之，[[TcpServer]] 和 [[TcpConnection]] 的代码都只处理单线程的情况（甚至都没有 [[mutex]] 成
员），而我们借助 EventLoop :: runInLoop () 并引入 EventLoopThreadPool 让多线程 TcpServer 的实现易如反掌。

[[999 - muduo总结]]
==注意 ioLoop 和 loop_间的线程切换都发生在连接建立和断开的时刻，不影响正常业务的性能。==

---

Muduo 目前采用最简单的 [[round-robin]] 算法来选取 pool 中的 EventLoop，不允许 TcpConnection 在运行中更换 EventLoop，这对长连接和短连接服务都是适用的，不易造成偏载。Muduo 目前的设计是每个 TcpServer 有自己的 EventLoopThreadPool，多个 TcpServer 之间不共享 EventLoopThreadPool。将来如果有必要，也可以多个 TcpServer 共享 EventLoopThreadPool，比方说一个服务有多个等价的 TCP 端口，每个 TcpServer 负责一个端口，而来自这些端口的连接 (s) 共享一个 EventLoopThreadPool。

另外一种可能的用法是一个 EventLoop aLoop 供两个 TcpServer 使用（a 和 b）。其中 a 是单线程服务，aLoop 既要 accept (2) 连接也要执行 IO；而 b 是多线程服务，有自己的 EventLoopThreadPool，只用 aLoop 来 accept (2) 连接。ALoop 上还可以运行几个 TcpClient。这些搭配都是可行的，==这也正是 EventLoop 的灵活性所在，可以根据需要在多个线程间调配负载。==


---
