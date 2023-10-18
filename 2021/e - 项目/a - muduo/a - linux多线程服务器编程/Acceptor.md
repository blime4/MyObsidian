先定义Acceptor class，用于[[accept]](2)新TCP连接，并通过回调通知使用者。它是内部class，供[[TcpServer]]使用，生命期由后者控制。Acceptor的接口如下：

---

```c++
class Acceptor : boost::nocopyable{
public:
	typedef boost::function<int sockfd, const InetAddress&> NewConnectionCallback;
	Acceptor(EventLoop* loop, const InetAddress& listenAddr);
	~Acceptor();
	
	void setNewConnectionCallback(const NewConnectionCallback& cb){newConnectionCallback_ = cb;}
	bool listenning() const { return listenning_;}
	void listen();
	
private:
	void hanldeRead();
	
	EventLoop* loop_;
	Socket acceptSocket_;
	Channel acceptChannel_;
	NewConnectionCallback newConnectionCallback_;
}

```

---

Acceptor 的数据成员包括 [[socket]]、[[Channel]] 等。其中 Socket 是一个 [[RAIIhandle]]，封装了 socket 文件描述符的生命期。Acceptor 的 socket 是 listening socket，即 server socket。Channel 用于观察此 socket 上的 readable 事件，并回调 Acceptor :: handleRead ()，后者会调用 accept (2) 来接受新连接，并回调用户 callback。


--- 

Acceptor 的构造函数和 Acceptor :: listen () 成员函数执行创建 TCP 服务端的传统步骤，即调用 [[socket]] (2)、[[bind]] (2)、[[listen]] (2) 等 [[Sockets API]]，其中任何一个步骤出错都会造成程序终止 2，因此这里看不到错误处理。

---

```c++
Acceptor::Acceptor(EventLoop* loop, const InetAddress& listenAddr)
	: loop_(loop),
	  acceptSocket_(sockets::createNonblockingOrDie()), // 创建非阻塞的socket
	  acceptChannel_(loop,acceptSocket_.fd()),
	  listenning_(false)
{
	acceptSocket_.setReuseAddr(true);
	acceptSocket_.bindAddress(listenAddr);
	acceptChannel_.setReadCallback(boost::bind(&Acceptor::handleRead, this));
}


void Acceptor::listen(){
	loop_->assertInLoopThread();
	listenning_ = true;
	acceptSocket_.listen();
	acceptChannel_.enableReading();
}
```

---

Acceptor 的接口中用到了 InetAddress class，这是对 struct sockaddr_in 的简单封装，能自动转换字节序.
Acceptor 的构造函数用到 createNonblockingOrDie () 来创建非阻塞的 socket.

---

```c++
int sockets::createNonblockingOrDie(){
	int sockfd = ::socket(AF_INET, SOCK_STREAM | SOCK_NONBLCOK | SOCK_CLOEXEC, IPPROTO_TCP);
	
	if(sockfd < 0){
		LOG_SYSFATAL << "sockets::createNonblockingOrDie";
	}
	return sockfd;
}

```


---

Acceptor :: listen () 的最后一步让 acceptChannel_在 socket 可读的时候调用 Acceptor :: handleRead ()，后者会接受（accept (2)）并回调 newConnectionCallback_。这里直接把 socket fd 传给 callback，这种传递 int 句柄的做法不够理想，在 C++11 中可以先创建 Socket 对象，再用[[移动语义]]把 Socket 对象 std :: move () 给回调函数，确保资源的安全释放。

---

```c++

void Acceptor::handleRead(){
	loop_->assetInLoopThread();
	InetAddress peerAddr(0);
	
	int connfd = acceptSocket_.accept(&peerAddr);
	if(connfd >= 0){
		if(newConnectionCallback_){
			newConnectionCallback_(connfd, peerAddr);
		} else{
			sockets::close(connfd);
		}
	}
}
```

注意这里的实现没有考虑文件描述符耗尽的情况，muduo 的处理办法见§7.7。还有一个改进措施，在拿到大于或等于 0 的 connfd 之后，非阻塞地 poll (2) 一下，看看 fd 是否可读写。正常情况下 poll (2) 会返回 writable，表明 connfd 可用。如果 poll (2) 返回错误，表明 connfd 有问题，应该立刻关闭连接。

Acceptor :: handleRead () 的策略很简单，每次 accept (2) 一个 socket。
==另外还有两种实现策略，==
一是每次循环 accept (2)，直至没有新的连接到达；
二是每次尝试 accept (2) N 个新连接，N 的值一般是 10。
后面这两种做法适合[[短连接服务]]，而 muduo 是为[[长连接服务]]优化的，因此这里用了最简单的办法。

---


下面写个小程序来试验 Acceptor 的功能，它在 9981 端口侦听新连接，连接到达后向它发送一个字符串，随即断开连接。

```c++
void newConnection(int sockfd, const muduo::InetAddress& peerAddr){
	printf("newConnection(): accepted a new connection from %s\n" , peerAddr.toHostPort().c_str());
	::write(sockfd, "How are you?\n" 13);
	muduo::sockets::close(sockfd);
}


int main(){

	printf("main(): pid = %d\n", getpid());
	
	muduo::InetAddress listenAddr(9981);
	muduo::EventLoop loop;
	
	muduo::Acceptor acceptor(&loop, listenAddr);
	acceptor.setNewConnectionCallback(newConnection);
	acceptor.listen();
	
	loop.loop();
}
```



