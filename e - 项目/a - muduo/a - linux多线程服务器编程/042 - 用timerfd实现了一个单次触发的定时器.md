
有了 [[EventLoop]]、[[Poller]]、[[Channel]]，我们写个小程序简单地测试一下功能。 用 timerfd 实现了一个单次触发的定时器，这个程序利用 Channel 将 timerfd 的 readable 事件转发给 timerout () 函数。

---

```c++
#include<sys/timerfd.h>

muduo::EventLoop* g_loop;

void timeout(){
	printf("Timeout!\n");
	g_loop->quit();
}

int main(){
	muduo::EventLoop loop;
	g_loop = &loop;
	
	int timerfd = ::timerfd_create(CLOCK_MONOTONIC, TFD_NONBLOCK | TFD_CLOEXEC);
	muduo::Channel channel(&loop, timerfd);
	channel.setReadCallback(timeout);
	channel.enableReading();
	
	struct itimerspec howlong;
	bzero(&howlong, sizeof(howlong));
	howlong.it_value.tv_sec = 5;
	::timerfd_settime(timefd, 0 , &howlong, NULL);
	
	loop.loop();
	::close(timerfd);
}

```

---

由于 [[poll]] (2) 是 [[level trigger]]，在 timeout () 中应该 read () timefd，否则下次会立刻触发。在现阶段采用 level trigger 的好处之一是可以通过 strace 命令直观地看到每次 poll (2) 的参数列表，容易检查程序的行为。