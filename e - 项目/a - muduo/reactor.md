


Reactor 模型的优点很明显，编程不难，效率也不错。不仅可以用于读写 socket，连接的建立（connect (2)/accept (2)）甚至 DNS 解析 都可以用非阻塞方式进行，以提高并发度和吞吐量（throughput），对于 IO 密集的应用是个不错的选择。lighttpd 就是这样，它内部的 fdevent 结构十分精妙，值得学习。