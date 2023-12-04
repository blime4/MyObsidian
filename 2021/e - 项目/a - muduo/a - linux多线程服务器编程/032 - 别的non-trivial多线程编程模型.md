

除了推荐的[[reactor]]＋[[thread poll]]，还有别的non-trivial多线程编程模型吗？

有，[[proactor]]。如果一次请求响应中要和别的进程打多次交道，那么Proactor模型往往能做到更高的并发度。当然，代价是代码变得支离破碎，难以理解。

这里举HTTP proxy为例，一次HTTP proxy的请求如果没有命中本地cache，那么它多半会：

1．[[解析域名]]（不要小看这一步，对于一个陌生的域名，解析可能要花几秒的时间）；
2．建立连接；
3．发送HTTP请求；
4．等待对方回应；
5．把结果返回给客户。

这5步中跟2个server发生了3次round-trip，每次都可能花几百毫秒：

1．向DNS问域名，等待回复；
2．向对方的HTTP服务器发起连接，等待TCP三路握手完成；
3．向对方发送HTTP request，等待对方response。

而实际上HTTP proxy本身的运算量不大，如果用线程池，池中线程的数目会很庞大，不利于操作系统的管理调度。

这时我们有两个解决思路：

1．把“域名已解析”、“连接已建立”、“对方已完成响应”做成event，继续按照Reactor的方式来编程。这样一来，每次客户请求就不能用一个函数从头到尾执行完成，而要分成多个阶段，并且要管理好请求的状态（“目前到了第几步？”）。

2．用[[回调函数]]，让系统来把任务串起来。比如收到用户请求，如果没有命中本地缓存，那么需要执行：

a．立刻发起异步的DNS解析startDNSResolve()，告诉系统在解析完之后调用DNSResolved()函数；

b．在DNSResolved()中，发起TCP连接请求，告诉系统在连接建立之后调用connectionEstablished()；

c．在connectionEstablished()中发送HTTP request，告诉系统在收到响应之后调用httpResponsed()；

d．最后，在httpResponsed()里把结果返回给客户。

.NET大量采用的BeginInvoke/EndInvoke操作也是这个编程模式。当然，对于不熟悉这种编程方式的人，代码会显得很难看。有关Proactor模式的例子可参看Boost.Asio的文档，这里不再多说。

Proactor模式依赖操作系统或库来高效地调度这些子任务，每个子任务都不会阻塞，因此能用比较少的线程达到很高的IO并发度。

Proactor 能提高吞吐，但不能降低延迟，所以我没有深入研究。另外，在没有语言直接支持的情况下 Proactor 模式让代码非常破碎，在 C++中使用 Proactor 是很痛苦的。因此最好在“线程”很廉价的语言中使用这种方式，这时 runtime 往往会屏蔽细节，程序用单线程阻塞 IO 的方式来处理 TCP 连接。