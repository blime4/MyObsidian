# 014 - Muti-Reactor多线程模型

![[Pasted image 20211228193815.png]]


[[负载均衡]]的思想

+ 改进思路
	+ 保留原先 Single- Reactor引入的线程池外，新扩展了 Reactor线程。引入多个 Reactor线程。也称为[[主从结构]]
	+ mainReactor主要负责一些接收客户端连接，按照负载均衡的策略，分发给subReactor处理
	+ subReactor负责处理
		+  IO线程处理IO操作
		+ 工作线程处理业务逻辑

两种拓展思路：
+ 单进程（多线程）模式
	+ 典型实现：
	+ [[netty]]
	+ [[memcached]]
+ 多进程模式:
	+ [[nginx]]


![[Pasted image 20211228194500.png]]