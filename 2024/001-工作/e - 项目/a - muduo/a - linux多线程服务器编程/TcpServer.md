[[044 - TcpServer接受新连接]]

### TcpServer 的改动 

TcpServer 向 TcpConnection 注册 CloseCallback，用于接收连接断开的消息。

```c++
void TcpServer::newConnection(int sockfd, const InetAddress& peerAddr){
	loop_->assertInLoopThread();
	char buf[32];
	snprintf(buf, sizeof(buf), "#%d", nextConnId_);
	++nextConnId_;
	std::string connName = name_ + buf;
	
	LOG_INFO << "TcpServer::newConnection [" << name_
			 << "] - new connection [" << connName
			 << "] from "<<peerAddr.toHostPort();
	InetAddress localAddr(sockets::getLocalAddr(sockfd));
	TcpConnectionPtr conn(
		new TcpConnection(loop_,connName,sockfd,localAddr,peerAddr));
	connections_[connName] = conn;
	conn->setConnectionCallback(connectionCallback_);
	conn->setMessageCallback(messageCallback_);
	conn->setCloseCallback(boost::bind(&TcpServer::removeConnection, this, _1));
	conn->connectEstablished();
}

```

--- 

通常 TcpServer 的生命期长于它建立的 [[TcpConnection]]，因此不用担心 TcpServer 对象失效。在 muduo 中，TcpServer 的析构函数会关闭连接，因此也是安全的。TcpServer :: removeConnection () 把 conn 从 ConnectionMap 中移除。这时 TcpConnection 已经是命悬一线：如果用户不持有 TcpConnectionPtr 的话，conn 的引用计数已降到 1。注意这里一定要用 EventLoop :: queueInLoop ()，否则就会出现此处讲的对象生命期管理问题。另外注意这里用 boost :: bind 让 TcpConnection 的生命期长到调用 connectDestroyed () 的时刻。

---
```c++

void TcpServer::removeConnection(const TcpConnectionPtr& conn){

	loop_->assertInLoopThread();
	LOG_INFO << "...xxx";
	size_t n = connections_.erase(conn->name());
	assert(n == 1); (void)n;
	loop_->queueInLoop(boost::bind(&TcpConnection::connectDestoryed,conn));
}
```

---

EventLoop 和 Poller 的改动本节 TcpConnection 不再是只生不灭，因此要求 EventLoop 也提供 unregister 功能。EventLoop 新增了 removeChannel () 成员函数，它会调用 Poller :: removeChannel ()，后者定义如下，复杂度为 O (logN)。

[[EventLoop]]
[[Poller]]