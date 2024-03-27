主动发起连接比被动接受连接要复杂一些，一方面是错误处理麻烦，另一方面是要考虑重试。在非阻塞网络编程中，发起连接的基本方式是调用 [[connect]] (2)，当 socket 变得可写时表明连接建立完毕。当然这其中要处理各种类型的错误，因此我们把它封装为 Connector class。

```c++
class Connector : boost :: nocopyable {
public:
	typedef boost::function<void(int sockfd)> NewConnectionCallback;
	Connector(EventLoop* loop, const InetAddress& serverAddr);
	~Connector();
	
	void setNewConnectionCallback(const NewConnectionCallback& cb){newConnectionCallback_ = cb};
	
	void start();
	void restart();
	void stop();
	
	
private:
	
}
```

Connector 只负责建立 [[socket]] 连接，不负责创建 [[TcpConnection]]，它的 NewConnectionCallback 回调的参数是 socket 文件描述符。

---

Connector 的实现有几个难点：
+ socket 是一次性的，一旦出错（比如对方拒绝连接），就无法恢复，只能关闭重来。
+ 但 Connector 是可以反复使用的，因此每次尝试连接都要使用新的 socket 文件描述符和新的 Channel 对象。
+ 要留意 Channel 对象的生命期管理，并防止 socket 文件描述符泄漏。
+ 错误代码与 accept (2) 不同，EAGAIN 是真的错误，表明本机 ephemeral port 暂时用完，要关闭 socket 再延期重试。“正在连接”的返回码是 EINPROGRESS。
+ 另外，即便出现 socket 可写，也不一定意味着连接已成功建立，还需要用 getsockopt (sockfd, SOL_SOCKET, SO_ERROR, ...) 再次确认一下。


