- Date: 2021-12-27
- Time:  17:36
[参考自：](https://www.cnblogs.com/zengzy/p/5113910.html）

# [[IO多路复用]]之select

## select接口

`#include <sys/select.h>`
_int_ **select**(_int_ _nfds_, _fd_**_**_set_ _*readfds_, _fd_**_**_set_ _*writefds_, _fd_**_**_set_ _*exceptfds,_ _struct_ _timeval_ _*timeout_);

-   readfds，读流集合，也就是程序员希望从这些描述符中读内容
-   writefds，写流集合，也就是程序员希望向这些描述符中写内容
-   exceptfds，异常流集合，也就是中间过程发送了异常
-   nfds，上面三种事件中，最大的文件描述符+1
-   timeout，程序员的容忍度，可等待的时间
```c++
struct timeval{
 long tv_sec;//second
 long tv_usec;//minisecond
}
```
 timeout有三种取值：
-   NULL，select一直阻塞，直到readfds、writefds、exceptfds集合中至少一个文件描述符可用才唤醒
-   0，select不阻塞
-   timeout_value，select在timeout_value这个时间段内阻塞

如果非得与“多路”这个词关联起来，那就是readfds+writefds+exceptfds的数量和就是路数。

另外，还有一组与fd_set 有关的操作

-   **FD_SET**(_fd_, _fdset)，把fd加入_fdset集合中
-   **FD_CLR**(_fd_, _fdset)，把fd从_fdset集合中清除
-   **FD_ISSET**(_fd_, _fdset)，判定fd是否在_fdset集合中
-   **FD_ZERO**(_fdset)，清除_fdset有描述符

## select实现原理

select的实现依赖于设备的驱动函数poll，poll的功能是检查设备的哪条条流可用（一个设备一般有三条流，读流，写流，设备发生异常的异常流），如果其中一条流可用，返回一个mask（表示可用的流），如果不可用，把当前进程加入设备的流等待队列中，例如读等待队列、写等待队列，并返回资源不可用。

 select正是利用了poll的这个功能，首先让程序员告知自己关心哪些io流（用文件描述符表示，也就是上文的readfds、writefds和exceptfds），并让程序员告知自己这些流的范围（也就是上文的nfds参数）以及程序的容忍度（timeout参数），然后select会把她们拷贝到内核，在内核中逐个调用流所对应的设备的驱动poll函数，当范围内的所有流也就是描述符都遍历完之后，他会检查是否至少有一个流发生了，如果有，就修改那三个流集合，把她们清空，然后把发生的流加入到相应的集合中，并且select返回。如果没有，就睡眠，让出cpu，直到某个设备的某条流可用，就去唤醒阻塞在流上的进程，这个时候，调用select的进程重新开始遍历范围内的所有描述符。

 直接看这个步骤可能会好理解些

-   1、拷贝nfds、readfds、writefds和exceptfds到内核
-   2、遍历`[0,nfds)`范围内的每个流，调用流所对应的设备的驱动poll函数
-   3、检查是否有流发生，如果有发生，把流设置对应的类别，并执行4，如果没有流发生，执行5。或者timeout=0，执行4
-   4、select返回
-   5、select阻塞当前进程，等待被流对应的设备唤醒，当被唤醒时，执行2。或者timeout到期，执行4



# select 的缺点（部分）
select需要我们指定文件描述符的最大值，然后取`[0，nfds)`这个范围内的值查看是在集合readfds,writefds或execptfds中，也就是说这个范围内存在一些不是我们感兴趣的文件描述符，cpu做了一些无用功，[[poll]]对她进行了改进


# select、poll的些许缺点

这两个多路复用实现的特点是：

-   每次调用select和poll都要把用户关心的事件集合（select为readfds,writefds,exceptfds集合，poll为fds结构体数组）从用户空间到内核空间。
-   如果某一时间段内，只有少部分事件是活跃的（用户关心的事件集合只有少部分事件会发生），会浪费cpu在对无效事件轮询上，使得效率较低，比如，用户关心1024个tcp socket的读事件，当是，每次调用select或poll时只有1个tcp链接是活跃的，那么对其他1023个事件的轮询是没有必要的。



select支持的文件描述符数量较小，一般只有1024，poll虽然没有这个限制，但基于上面两个原因，poll和select存在同样一个缺点，就是包含大量文件描述符的数组被整体复制于用户态和内核的地址空间之间，而且不论这些文件描述符是否就绪，每次都会轮询所有描述符的状态，使得他们的开销随着文件描述符数量的增加而线性增大。[[epoll]]针对这几个缺点进行了改进，不再像select和poll那样，每次调用select和poll都把描述符集合拷贝到内核空间，而是一次注册永久使用；另一方面，epoll也不会对每个描述符都轮询时间是否发生，而是只针对事件已经发生的文件描述符进行资源抢占（因为同一个描述符资源（如可读或可写）可能阻塞了多个进程，调用epoll的进程需要与这些进程抢占该相应资源）。