epoll (4) 是 Linux 独有的高效的 [[IO multiplexing]] 机制，
它与 poll (2) 的不同之处主要在于 poll (2) 每次返回整个文件描述符数组，
用户代码需要遍历数组以找到哪些文件描述符上有 IO 事件

（见此处的 Poller :: fillActiveChannels ()），而 epoll_wait (2) 返回的是活动 fd 的列表，需要遍历的数组通常会小得多。

==在并发连接数较大而活动连接比例不高时，epoll (4) 比 poll (2) 更高效==


[[EPoller]]


### 一、操作函数 ###

```cpp
#include <sys/epoll.h>
int epoll_create(int size);
int epoll_create1(int flags);

int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);

typedef union epoll_data {
	void    *ptr;
	int      fd;
	uint32_t u32;
	uint64_t u64;
} epoll_data_t;

struct epoll_event {
	uint32_t     events;	/* Epoll events */
	epoll_data_t data;	/* User data variable */
};
```

### 二、两种触发模式 ###

- Level-Triggered  [[level trigger]]   (高电平触发) EPOLLIN不为空就高电平，就一直水平触发
- Edge-Triggered  [[edge trigger]]

需要注意的是EPOLLIN和EPOLLOUT事件来临的时机：

- [[EPOLLIN]]事件

当内核中的socket接收缓冲区为空，则为低电平；内核中的socket接收缓冲区不为空，则为高电平。

- [[EPOLLOUT]]事件

内核中的socket发送缓冲区满了则为高电平，否则为低电平

epoll的LT为高电平触发；ET为边缘触发。


[[select、poll和epoll对比]]


[[todo]]
[以下参考自](https://www.cnblogs.com/zengzy/p/5118336.html)
- Date: 2021-12-27
- Time:  17:56
# 3、[[epoll的两种触发模式ET&LT]]

二者的差异在于level-trigger模式下只要某个socket处于readable/writable状态，无论什么时候进行epoll_wait都会返回该socket；而edge-trigger模式下只有某个socket从unreadable变为readable或从unwritable变为writable时，epoll_wait才会返回该socket，et模式注重的是状态发生改变的时候才触发。下面两幅图清晰反映了二者区别，这两幅图摘自[Epoll在LT和ET模式下的读写方式](http://blog.chinaunix.net/uid-20775448-id-3603224.html) 

 ![](https://images2015.cnblogs.com/blog/733402/201601/733402-20160110212600528-65324139.png)　　　　　　![](https://images2015.cnblogs.com/blog/733402/201601/733402-20160110212612075-480095595.png)

在ET模式下，在使用epoll_ctl注册文件描述符的事件时，应该把描述符设置为非阻塞，为什么呢？以上面左边这幅图为例，当数据到来之后，该socket实例从不可读状态边为可读状态，从该socket读取一部分数据后，再次调用epoll_wait，由于socket的状态没有发生改变（buffer上一次空到有数据可读触发了et，而这一次buffer还有数据可读，状态没改变），所以该次调用epoll_wait并不会返回这个socket的可读事件，而且之后也不会再发生改变，这个socket实例将永远也得不到处理。这就是为什么将监听的描述符设置为非阻塞的原因。

使用ET模式时，正确的读写方式应该是这样的：

设置监听的文件描述符为非阻塞 while(true){
　　epoll_wait(epoll_fd,events,max_evens);
　　读，只要可读，就一直读，直到返回0，或者-1，errno=EAGAIN/EWOULDBLOCK
}

正确的写方式应该是这样的：

设置监听的文件描述符为非阻塞 while(true){
　　epoll_wait(epoll_fd,events,max_evens);
　　写，只要可写，就一直写，直到返回0，或者-1，errno=EAGAIN/EWOULDBLOCK
}

# 4、两个问题

使用单进程单线程IO多路复用，服务器端该如何正确使用accept函数？

应该将监听的socket实例设置为非阻塞。

使用io多路复用时，一般会把监听连接的socket实例listen_fd交给select、poll或epoll管理，如果使用阻塞模式，假设，select、poll或epoll调用返回时，有大量描述符的读或写事件准备好了，而且listen_fd也可读，

我们知道，从select、poll或epoll返回到调用accept接收新连接是有一个时间差的，如果这个时间内，发起请求的一端主动发送RST复位请求，服务器会把该连接从ACCEPT队列（[socket原理详解](http://www.cnblogs.com/zengzy/p/5107516.html)，3.6节）中取出，并把该连接复位，这个时候再调用accept接收连接时，服务器将被阻塞，那其他的可读可写的描述符将得不到处理，直到有新连接时，accept才得以返回，才能去处理其他早已准备好的描述符。所以应该将listen_fd设置为非阻塞。

腾讯后台开发面试题。使用Linux epoll模型，LT触发模式，当socket可写时，会不停的触发socket可写的事件，但并不总是需要写，该如何处理？

第一种最普遍的方式，步骤如下：

1.  需要向socket写数据的时候才把socket加入epoll，等待可写事件。
2.  接受到可写事件后，调用write或者send发送数据，直到数据写完。
3.  把socket移出epoll。

这种方式的缺点是，即使发送很少的数据，也要把socket加入epoll，写完后在移出epoll，有一定操作代价。

一种改进的方式，步骤如下：

1.  设置socket为非阻塞模式
2.  调用write或者send发送数据，直到数据写完
3.  如果返回EAGAIN，把socket加入epoll，在epoll的驱动下写数据，全部数据发送完毕后，再移出epoll。

这种方式的优点是：数据不多的时候可以避免epoll的事件处理，提高效率。











