==SIGPIPE的默认行为是终止进程，在命令行程序中这是合理的，但是在网络编程中，这意味着如果对方断开连接而本地继续写入的话，会造成服务进程意外退出。==

假如服务进程繁忙，没有及时处理对方断开连接的事件，就有可能出现在连接断开之后继续发送数据的情况。

解决办法很简单，在程序开始的时候就忽略SIGPIPE，可以用C++[[全局对象]]做到这一点。

```c++

class IgnoreSigPipe{
public:
	IgnoreSigPipe(){
		::signal(SIGPIPE,SIG_IGN);
	}
};

IgnoreSigPipe initObj;
```


如果客户端关闭了套接字socket，而服务器调用了一次write，服务器会接收到一个[[RST]] segment(TCP传输层)，如果服务器端再次调用write，这时就会产生SIGPIPE信号。

[[SIGCHLD]]