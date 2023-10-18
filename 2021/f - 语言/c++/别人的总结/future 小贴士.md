
# future 小贴士

[[888 - 具体略过了，以后再学]]

future为了[[三五法则]]，删除了拷贝构造/赋值函数。如果需要浅拷贝，实现共享同个 future对象，可以用sta:shared_future。
如果不需要返回值，std:async里 lambda的返回类型可以为Void，这时 future对象的类型为`std::future<void>`。
同理有`std::promise<void>`，他的set_valuer()不接受参数，仅仅作为同步用，不传递任何实际的值。

![[Pasted image 20220113121116.png]]

