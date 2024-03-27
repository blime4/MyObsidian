![[Pasted image 20211216110434.png]]

---

其中的“X”表示[[TcpConnection]]通常会在此时析构。

---

### [[Channel]] 的改动
Channel class新增了CloseCallback事件回调，并且断言（assert()）在事件处理期间本Channel对象不会析构

---

```c++

Channel::~Channel(){
	assert(!eventHandling_);
}

void Channel::handleEvent(){
	eventHandling_ = true;
	if(revents_ & POLLNVAL){
		LOG_WARN << "Channel::handle_event() POLLNVAL";
	}
	if((revents_ &  POLLHUP) && !(revents_ & POLLHIN)){
		LOG_WARN << "Channel::handle_event() POLLNUP";
		if(closeCallback_) closeCallback_();
	}
	...
	eventHandling_ = false;
}
```