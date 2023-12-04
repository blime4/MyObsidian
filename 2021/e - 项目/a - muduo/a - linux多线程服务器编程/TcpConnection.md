
TcpConnection是muduo里唯一默认使用[[shared_ptr]]来管理的class，也是唯一继承[[enable_shared_from_this]]的class，这源于其模糊的生命期
```c++

class TcpConnection : boost::nocopyable,
					  public boost::enable_shared_from_this<TcpConnection>
{
public:
	void setCloseCallback(const CloseCallback& cb){closrCallback_ = cb;}
	void connectEstablished(); // should be called only once
	void connectDestoryed(); // should be called only once
	//thread safe
	void send(const std::string& message);
	//thread safe
	void shutdown();
	
private:
	enum StateE {KConnecting, KConnected,KDisconnecting,KDisconnected};
	void setState(StateE s) {state_ = s};
	void handleRead();
	void handleWrite();
	void handleClose();
	void handleError();
	
	void sendInLoop(const std::string& message);
	void shutdownInLoop();
	
	Buffer inputBuffer_;
	Buffer outputBuffer_;
	
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

![[Pasted image 20211216153440.png]]

---

TcpConnection :: handleRead () 会检查 read (2) 的返回值，根据返回值分别调用 
MessageCallback_、handleClose ()、handleError ()。

---

```c++
void TcpConnection::handleRead(){

	char buf[65536];
	ssize_t n = ::read(channel_->fd(),buf,sizeof(buf));
	if(n>0){
		messageCallback_(shared_from_this(),buf,n);
	} else if(n == 0){
		handleClose();
	} else{
		handleError();
	}
}
```

---

TcpConnection :: handleClose () 的主要功能是调用 closeCallback_，这个回调绑定到 [[TcpServer]] :: removeConnection ()。

---

```c++
void TcpConnection::handleClose(){
	loop_->assertInLoopThread();
	LOG_TRACE << "TcpConnection::handleClose state = " << state_;
	assert(state_ == KConnected);
	channel_->disableAll();
	closeCallback_(shared_from_this());
	
}

void TcpConnection::handleError(){
	int err = sockets::getSocketError(channel_->fd());
	LOG_ERROR << "....xxx";
}

```

---

TcpConnection :: connectDestroyed () 是 TcpConnection 析构前最后调用的一个成员函数，它通知用户连接已断开。其中的 L68 与上面的 L97 重复，这是因为在某些情况下可以不经由 handleClose () 而直接调用 connectDestroyed ()。

---

```c++

void TcpConnection::connectDestoryed(){
	loop_->assertInLoopThread();
	LOG_TRACE << "TcpConnection::handleClose state = " << state_;
	assert(state_ == KConnected);
	channel_->disableAll();
	closeCallback_(shared_from_this());
	
	loop_->removeChannel(get_pointer(channel_));
}
```


---

shutdown () 是线程安全的，它会把实际工作放到 shutdownInLoop () 中来做，后者保证在 IO 线程调用。如果当前没有正在写入，则关闭写入端。代码注释给出了两个值得改进的地方。

---

```c++

void TcpConnection::shutdown(){
	if(state_ == kConnected){
		setState(kDisconnecting);
		loop_->runInLoop(boost::bind(&TcpConnection::shutdownInLoop, this));
		
	}
}

void TcpConnection::shutdownInLoop(){
	loop_->assertInLoopThread();
	if(!channel_->isWriting()){
		socket_->shutdownWrite();
	}
}
```

---

由于新增了 kDisconnecting 状态，TcpConnection :: connectDestroyed () 和 TcpConnection :: handleClose () 中的 assert () 也需要相应的修改，代码从略。

---

Send () 也是一样的，如果在非 IO 线程调用，它会把 message 复制一份，传给 IO 线程中的 sendInLoop () 来发送。这么做或许有轻微的效率损失，但是线程安全性很容易验证，我认为还是利大于弊。如果真的在乎这点性能，不如让程序只在 IO 线程调用 send ()。另外在 C++11 中可以使用[[移动语义]]，避免[[内存拷贝]]的开销。

---

```c++

void TcpConnection::send(){
	if(state_ == kConnected){
		if(loop_->isInLoopThread()){
			sendInLoop(message);
		}
		else{
			loop->runInLoop(boost::bind(&TcpConnection::sendInLoop, this, message));
		}
	}
}
```

sendInLoop () 会先尝试直接发送数据，如果一次发送完毕就不会启用 WriteCallback；如果只发送了部分数据，则把剩余的数据放入 outputBuffer_，并开始关注 writable 事件，以后在 handlerWrite () 中发送剩余的数据。如果当前 outputBuffer_已经有待发送的数据，那么就不能先尝试发送了，因为这会造成数据乱序。

代码略


---

[[047 - 完善TcpConnection]]]
