
# 038 - 用 muduo 实现 echo 服务

Step 1．定义 EchoServer class，不需要派生自任何基类。
```c++

#include<muduo/net/TcpServer.h>

class EchoServer{
public:
	EchoServer(muduo::net::EventLoop* loop,
		const muduo::net::InetAddress& listenAddr);
	void start();
private:
	void onConnection(const muduo::net::TcpConnectionPtr& conn);
	void onMessage(const muduo::net::TcpConnectionPtr& conn,
		muduo::net::Buffer* buf,
		muduo::Timestamp time);
	muduo::net::EventLoop* loop_;
	muduo::net::TcpServer server_;
	
}
```
在构造函数里注册回调函数
```c++

EchoServer::EchoServer(muduo::net::EventLoop* loop,
		const muduo::net::InetAddress& listenAddr)
	:loop_(loop),
	 server_(loop,listenAddr,"EchoServer")
{
	server_.setConnectionCallback(
		boost::bind(&EchoServer::OnConnection,this,_1));
	server_.setMessageCallback(
		boost::bind(&EchoServer::onMessage,this,_1,_2,_3));
}
```

Step 2. 实现 EchoServer :: onConnection () 和 EchoServer :: onMessage ()。
```c++
void EchoServer::onConnection(const muduo::net::TcpConnectionPtr& conn)
{
	LOG_INFO << "EchoServer - " << conn->peerAddress().toIpPort() << " -> " << conn->localAddress().toIpPort() << " is " <<(conn->connected() ? "UP" : ""DOWN);	
}


void EchoServer::onMessage(const muduo::net::TcpConnectionPtr& conn,
	muduo::net::Buffer* buf,
	muduo::Timestamp time)
{
	muduo::string msg(buf->retriveAllAsString());
	LOG_INFO << conn->name() << " echo " << msg.size() << " bytes, data received at "<<time.toSting();
	conn->send(msg);
}

void EchoServer::start()

{
	server_.start();
}
```

Step 3. 在 main () 里用 EventLoop 让整个程序跑起来
```c++
#include "echo.h"
#include <muduo/base/Logging.h>
#include <muduo/net/EventLoop.h>

int main(){
	LOG_INFO << "pid = " <<getpid();
	muduo::net::EventLoop loop;
	muduo::net::InetAddress ListenAddr(2007);
	EchoServer server(&loop,listenAddr);
	server.start();
	loop.loop();
}
```