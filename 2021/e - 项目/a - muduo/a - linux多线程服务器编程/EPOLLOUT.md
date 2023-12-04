EPOLLOUT事件

内核中的socket发送缓冲区满了则为高电平，否则为低电平

epoll的LT为高电平触发；ET为边缘触发。