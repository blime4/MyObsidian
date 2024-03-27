![[Pasted image 20211231043644.png]]

白色部分表示外部类，对外可见
+ [[EventLoop]]        ---> 事件循环的抽象
	+ 一个EventLoop 包括多个Channel（聚合关系）， 可以捕捉多个通道的可读可写事件
	+ EventLoop 不负责 Channel生命期的控制 ， Channel的生命期由 TcpConnections， Acceptor，Connector 控制。
+ [[TcpConnection]] ---> 对已连接套接字的抽象
	+ 一旦被动或主动连接，就会得到一个TcpConnection
+ [[TcpServer]]
+ [[TcpClient]]
灰色部分为内部类，对外不可见
+ [[Poller]]                ---> IO复用的抽象, muduo中唯一使用面向对象的类
	+ [[PollPoller]]
	+ [[EPollPoller]]
+ [[Channel]]            ---> 对IO事件注册于响应的封装
	+ update() 成员函数负责注册或者更新IO的可读可写等事件
		+ 调用Chanel的Update 更新IO可读可写等事件，就会调用到EventLoop的updateChannel()，从而调用Poller的updateChannel(). 
		+ 从而将文件描述符fd的可读可写事件注册或者更新到Poller中
	+ handleEvent() 是对所发生的IO事件进行处理
	+ 不拥有文件描述符，关闭Channel的时候，不会关闭文件描述符
		+ fd 是由套接字Socket所拥有，控制的, 由FileDescriptor管理
+ [[FileDescriptor]]
+ [[socket]]
+ [[Acceptor]]              ---> 对被动连接的抽象，关注的是监听套接字的可读事件
+ [[Connector]]            ---> 对主动连接的抽象


EventLoop 和 Poller 的关系是组合， 一个Eventloop 里面有一个Poller， EventLoop的loop（）函数调用的是Poller中的 poll() 函数

Channel是TcpConnection, Acceptor, Connector 的成员，跟他们的关系是组合
