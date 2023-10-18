[[TCPNoDelay]] 和 [[TCPkeepalive]] 都是常用的 TCP 选项，前者的作用是禁用 [[Nagle 算法]] ，避免连续发包出现延迟，这对编写低延迟网络服务很重要。后者的作用是定期探查 TCP 连接是否还存在。一般来说如果有应用层[[心跳]]的话，TCP keepalive 不是必需的，但是一个通用的网络库应该暴露其接口。

> (如果没有应用层心跳，而对方机器突然断电，那么本机不会收到 TCP 的 FIN 分节。在没有发送消息的情况下，这个“连接”可能一直保持下去。)

---

TCPNoDelay
```c++
void TcpConnection::setTcpNoDelay(bool on){
	socket_->setTcpNoDelay(on);
}

void Socket::setTcpNoDelay(bool on){
	int optval = on ? 1 : 0;
	::setsockopt(sockfd_, IPPRO_TCP, TCP_NODELAY, &optval, sizeof(optval));
}
```