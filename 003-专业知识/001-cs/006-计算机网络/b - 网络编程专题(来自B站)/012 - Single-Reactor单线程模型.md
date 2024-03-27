# 012 - Single-Reactor单线程模型
![[Pasted image 20211228192646.png]]

> 此处单线程主要针对IO操作而言，简而言之，IO中的 accept()、read()、 write()都是在一个线程完成的



存在问题：
目前该模型中，除了I/O操作在 Reactor线程外，**业务逻辑处理操作也在 Reactor线程上**，当业务逻辑处理比较耗时时，会大大降低了I/O请求的处理效率典型

实现redis（4.0之前）


我的理解：

Single-Reactor单线程模型，Reactor既负责IO操作（accept，read,write），又负责业务处理操作（如图中的解码编码操作），如果业务处理操作比较耗时的话，IO请求的相应就会受到影响。


改进版：
[[013 - Single-Reactor线程池模型]]