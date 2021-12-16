

本节会介绍[[TcpServer]]并初步实现[[TcpConnection]]，本节只处理连接的建立，下一节处理连接的断开，再往后依次处理读取数据和发送数据。


TcpServer 新建连接的相关函数调用顺序见图（有的函数名是简写，省略了 poll (2) 调用）。其中 Channel :: handleEvent () 的触发条件是 listening socket 可读，表明有新连接到达。TcpServer 会为新连接创建对应的 TcpConnection 对象。

---

![[Pasted image 20211216005615.png]]


---

TcpServer class 的功能是管理 accept (2) 获得的 TcpConnection。TcpServer 是供用户直接使用的，生命期由用户控制。TcpServer 的接口如下，用户只需要设置好 callback，再调用 start () 即可。

---

```c++
class TcpServer : boost::noncopyable{

public:
	TcpServer(EventLoop* loop, const InetAddress& listenAddr);
	~TcpServer();
	
	void start();
	void setConnectionCallback(const ConnectionCallback& cb){ connectionCallback_ = cb; }
	void setMessageCallback(const MessageCallback& cb){ messageCallback_ = cb; }
private:
	
	void newConnection(int sockfd, const InetAddress& peerAddr);
	typedef std::map<std::string, TcpConnectionPtr> ConnectionMap;
	EventLoop* loop_;							// the acceptor loop
	const std::string name_;
	boost::scoped_ptr<Acceptor> acceptor_;		// avoid revealing Acceptor
	ConnectionCallback connectionCallback_;
	MessageCallback messageCallback_;
	bool started_;
	int nextConnId_;							// always in loop thread
	ConnectionMap connections_;
}

```

---


每个 TcpConnection 对象有一个名字，这个名字是由其所属的 TcpServer 在创建 TcpConnection 对象时生成，名字是 ConnectionMap 的 key。
在新连接到达时，[[Acceptor]] 会回调 newConnection ()，后者会创建 TcpConnection 对象 conn，把它加入 ConnectionMap，设置好 callback，再调用 conn->connectEstablished ()，其中会回调用户提供的 ConnectionCallback。代码如下。

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
	conn->connectEstablished();
}

```


---

TcpConnection是muduo里唯一默认使用[[shared_ptr]]来管理的class，也是唯一继承[[enable_shared_from_this]]的class，这源于其模糊的生命期
```c++

class TcpConnection : boost::nocopyable,
					  public boost::enable_shared_from_this<TcpConnection>
{
public:
private:
	enum StateE {KConnecting, KConnected,};
	void setState(StateE s) {state_ = s};
	void handleRead();
	
	EventLoop* loop_;
	std::string name_;
	StateE state_;
	
	
	boost::scoped_ptr<Socket> socket_;
	boost::scoped_ptr<Channel> channel_;
	InetAddress localAddr_;
	InetAddress peerAddr_;
	
	ConnectionCallback connectionCallback_;
	MessageCallback messageCallback_;
};

```

---

[[045 - TcpServer断开连接]]