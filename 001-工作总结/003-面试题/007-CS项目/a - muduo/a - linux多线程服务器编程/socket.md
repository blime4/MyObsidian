[[muduo的Socket类分析]]


# socket原理详解
Date: 2021-12-27
- Time:  17:36
[参考自：](https://www.cnblogs.com/zengzy/p/5107516.html)

当我们使用不同的协议进行通信时就得使用不同的接口，还得处理不同协议的各种细节，这就增加了开发的难度，软件也不易于扩展。于是UNIX BSD就发明了socket这种东西，socket屏蔽了各个协议的通信细节，使得程序员无需关注协议本身，直接使用socket提供的接口来进行互联的不同主机间的进程的通信。

![[Pasted image 20211227171443.png]]

bsd在实现上把socket设计成一种文件，然后通过虚拟文件系统的操作接口就可以访问socket，而访问socket时会调用相应的驱动程序，从而也就是使用底层协议进行通信。（vsf也就是unix提供给我们的面向对象编程，如果底层设备是磁盘，就对磁盘读写，如果底层设备是socket就使用底层协议在网中进行通信，而对外的接口都是一致的）。

socket结构体关键域有so_type,so_pcb。so_type常见的值有：

-   [[SOCK_STREAM]] 提供有序的、可靠的、双向的和基于连接的字节流服务，当使用Internet地址族时使用TCP。
-   [[SOCK_DGRAM]] 支持无连接的、不可靠的和使用固定大小（通常很小）缓冲区的数据报服务，当使用Internet地址族使用UDP。
-   [[SOCK_RAW]] 原始套接字，允许对底层协议如IP或ICMP进行直接访问，可以用于自定义协议的开发。

so_pcb表示socket控制块，其又指向一个结构体，该结构体包含了当前主机的ip地址(inp_laddr)，当前主机进程的端口号(inp_lport)，发送端主机的ip地址(inp_faddr)，发送端主体进程的端口号(inp_fport)。so_pcb是socket类型的关键结构，不亚于进程控制块之于进程，在进程中，一个pcb可以表示一个进程，描述了进程的所有信息，每个进程有唯一的进程编号，该编号就对应pcb；socket也同时是这样，每个socket有一个so_pcb，描述了该socket的所有信息，而每个socket有一个编号，这个编号就是socket描述符。说到这里，我们发现，socket确实和进程很像，就像我们把具体的进程看成是程序的一个实例，同样我们也可以把具体的socket看成是网络通信的一个实例。


## socket编程接口
### 3.1、socket接口

int **socket**(int protofamily, int so_type, int protocol);

-   protofamily 指协议族，常见的值有：  
    AF_INET，指定so_pcb中的地址要采用ipv4地址类型  
    AF_INET6，指定so_pcb中的地址要采用ipv6的地址类型  
    AF_LOCAL/AF_UNIX，指定so_pcb中的地址要使用绝对路径名  
    当然也还有其他的协议族，用到再学习了
-   so_type 指定socket的类型，也就是上面讲到的so_type字段，比较常用的类型有：  
    [[SOCK_STREAM]]  
    [[SOCK_DGRAM]]  
    [[SOCK_RAW]]
-   protocol 指定具体的协议，也就是指定本次通信能接受的数据包的类型和发送数据包的类型，常见的值有：  
    [[IPPROTO_TCP]]，TCP协议  
    [[IPPROTO_UDP]]，UPD协议  
    0，如果指定为0，表示由内核根据so_type指定默认的通信协议
	![[Pasted image 20211227172124.png]]

这里解释一下上图，上图其实是使用AF_INET,SOCK_DGRAM,IPPRTO_UDP实例化之后的一个具体的socket。

那为什么要通过这三个参数来生成一个socket描述符？

答案就是通过这三个参数来确定一组固定的操作。我们说过抽象的socket对外提供了一个统一、方便的接口来进行网络通信，但对内核来说，每一个接口背后都是及其复杂的，同一个接口对应了不同协议，而内核有不同的实现，幸运的是，如果确定了这三个参数，那么相应的接口的映射也就确定了。在实现上，BSD就把socket分类描述，每一个类别都有进行通信的详细操作，分类见下图。而对socket的分类，就好比对unix设备的分类，我们对设备write和read时，底层的驱动是有各个设备自己提供的，而socket也一样，当我们指定不同的so_type时，底层提供的通信细节也由相应的类别提供。
![[Pasted image 20211227172311.png]] 图4 socket层次图


### 3.2、bind接口

int **bind**(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

bind函数就是给图三种so_pcb结构中的地址赋值的接口

-   sockfd   是调用socket()函数创建的socket描述符
-   addr     是具体的地址
-   addrlen  表示addr的长度
struct sockaddr其实是void的typedef，其常见的结构如下图（图片来源传智播客邢文鹏linux系统编程的笔记），这也是为什么需要addrlen参数的原因，不同的地址类型，其地址长度不一样：
![[Pasted image 20211227172540.png]]
图5 地址结构图

### 3.3、connect接口

int **connect**(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

这三个参数和bind的三个参数类型一直，只不过此处strcut sockaddr表示对端公开的地址。三个参数都是传入参数。connect顾名思义就是拿来建立连接的函数，只有像tcp这样面向连接、提供可靠服务的协议才需要建立连接


### 3.4、listen接口

int **listen**(int sockfd, int backlog)

告知内核在sockfd这个描述符上监听是否有连接到来，并设置同时能完成的最大连接数为backlog。3.6节还会继续解释这个参数。当调用listen后，内核就会建立两个队列，一个[[SYN队列]]，表示接受到请求，但未完成三次握手的连接；另一个是[[ACCEPT队列]]，表示已经完成了三次握手的队列

-   sockfd 是调用socket()函数创建的socket描述符
-   backlog 已经完成三次握手而等待accept的连接数

### 3.5、accept接口

int **accept**(int listen_sockfd, struct sockaddr *addr, socklen_t *addrlen)

accept函数就是从ACCEPT队列中拿一个连接，并生成一个新的描述符，新的描述符所指向的结构体so_pcb中的请求端ip地址、请求端端口将被初始化。

从上面可以知道，accpet的返回值是一个新的描述符，我们姑且称之为new_sockfd。那么new_sockfd和listen_sockfd有和不同呢？不同之处就在于listen_sockfd所指向的结构体so_pcb中的请求端ip地址、请求端端口没有被初始化，而new_sockfd的这两个属性被初始化了。

### 3.6、listen、connect、accept流程及原理
![[Pasted image 20211227173219.png]]
图片改自[http://blog.csdn.net/russell_tao/article/details/9111769](http://blog.csdn.net/russell_tao/article/details/9111769)）

1.  服务器端在调用listen之后，内核会建立两个队列，SYN队列和ACCEPT队列，其中ACCPET队列的长度由backlog指定。
2.  服务器端在调用accpet之后，将阻塞，等待ACCPT队列有元素。
3.  客户端在调用connect之后，将开始发起SYN请求，请求与服务器建立连接，此时称为第一次握手。
4.  服务器端在接受到SYN请求之后，把请求方放入SYN队列中，并给客户端回复一个确认帧ACK，此帧还会携带一个请求与客户端建立连接的请求标志，也就是SYN，这称为第二次握手
5.  客户端收到SYN+ACK帧后，connect返回，并发送确认建立连接帧ACK给服务器端。这称为第三次握手
6.  服务器端收到ACK帧后，会把请求方从SYN队列中移出，放至ACCEPT队列中，而accept函数也等到了自己的资源，从阻塞中唤醒，从ACCEPT队列中取出请求方，重新建立一个新的sockfd，并返回。

这就是listen,accept,connect这三个函数的工作流程及原理。从这个过程可以看到，在connect函数中发生了两次握手。


# 由socket定义的这一组API提供如下两点功能：

一是**将应用程序数据从用户缓冲区中复制到TCP/UDP内核发送缓冲区**，以交付内核来发送数据，或者是**从内核TCP/UDP接收缓冲区中复制数据到用户缓冲区**，以读取数据；

二是应用程序可以通过它们来**修改内核中各层协议的某些头部信息或其他数据结构**，从而精细地控制底层通信的行为。比如可以通过setsockopt函数来设置IP数据报在网络上的存活时间。


> 1. 用户缓冲区和TCP/UDP缓冲区的数据复制
> 2. 修改协议的头部信息或其他
