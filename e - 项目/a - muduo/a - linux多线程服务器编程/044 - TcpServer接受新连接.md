

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
	

}

```