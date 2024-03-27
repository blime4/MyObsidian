[[Buffer]] 是非阻塞 TCP 网络编程必不可少的东西（§7.4），本节介绍用 Buffer 来处理数据输入，下一节介绍数据输出。Buffer 是另一个具有值语义的对象。Buffer 是[[非阻塞 TCP]] 网络编程必不可少的东西（§7.4），本节介绍用 Buffer 来处理数据输入，下一节介绍数据输出。Buffer 是另一个具有值语义的对象。

---

[[MessageCallback]]
```c++

typedef boost::function<void (const TcpConnectionPtr&, Buffer* buf, Timestamp)> MessageCallback;
```

 - [ ] [[不懂]]
