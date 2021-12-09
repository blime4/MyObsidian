[[异步IO]]
[[DMA]]特性

![[capture-2021-12-09-16-32-55.png]]


1. 理论上[[proactor]]比[[reactor]]效率更高
2. 充分利用[[DMA ]]特性
3. [[boost asio]]实现的[[proactor]]，实际上并不是真正意义上的[[异步IO]],而是用[[epoll]]模拟实现的
4. 其实Linux目前没有比较成熟的[[异步IO]]服务器