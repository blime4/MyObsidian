TCP模块还有一个重要的任务，就是提高网络利用率，降低丢包率，并保证网络资源对每条数据流的公平性。这就是所谓的[[拥塞控制]]。



拥塞控制的四个部分：
[[慢启动]]（slow start）、
[[拥塞避免]]（congestion avoidance）、
[[快速重传]]（fastretransmit）、
[[快速恢复]]（fast recovery）。

总结：
> 慢启动： 指数启动
> 拥塞避免：+1避免



![[Pasted image 20211229182347.png]]

+ SWND（Send Window，发送窗口)
	+ + 太小，会引起明显的网络延迟
	+ + 太大，则容易导致网络拥塞
+ SMSS（SenderMaximum Segment Size，发送者最大段大小）
+ RWND 接收通告窗口
+ CWND (Congestion Window 拥塞窗口)

实际的SWND值是RWND和CWND中的较小者

![[Pasted image 20211229181226.png]]