

此种模型下，程序里的每个IO线程有一个[[event loop]]（或者叫[[Reactor]]），用于处理读写和定时事件（无论周期性的还是单次的）

这种方式的好处是：

·线程数目基本固定，可以在程序启动的时候设置，不会频繁创建与销毁。
·可以很方便地在线程间调配负载。
·IO事件发生的线程是固定的，同一个TCP连接不必考虑事件并发。

Eventloop代表了线程的主循环，需要让哪个线程干活，就把timer或IOchannel（如TCP连接）注册到哪个线程的loop里即可。对实时性有要求的connection可以单独用一个线程；数据量大的connection可以独占一个线程，并把数据处理任务分摊到另几个计算线程中（用线程池）；其他次要的辅助性connections可以共享一个线程。

对于non-trivial的服务端程序，一般会采用non-blocking IO＋IO multiplexing，每个connection/acceptor都会注册到某个event loop上，程序里有多个event loop，每个线程至多有一个event loop。

多线程程序对event loop提出了更高的要求，那就是“线程安全”。要允许一个线程往别的线程的loop里塞东西 ，这个loop必须得是线程安全的。