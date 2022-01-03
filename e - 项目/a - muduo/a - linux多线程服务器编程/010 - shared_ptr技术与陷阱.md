

# [[Shared_ptr]] 技术与陷阱


## 意外延长对象的生命期
Shared_ptr 是强引用（“铁丝”绑的），只要有一个指向 x 对象的 shared_ptr 存在，该对象就不会析构。而 shared_ptr 又是允许拷贝构造和赋值的（否则引用计数就无意义了），如果不小心遗留了一个拷贝，那么对象就永世长存了。


另外一个出错的可能是 [[boost bind]]，因为 boost :: bind 会把实参拷贝一份，如果参数是个 shared_ptr，那么对象的生命期就不会短于 boost ::function对象：
```c++
class Foo{
	void doit();
};

shared_ptr<Foo> pFoo(new Foo);
boost::function<void()> func = boost::bind(&Foo::doit,pFoo); // long life foo
```
这里func对象持有了shared_ptr<Foo>的一份拷贝，有可能会在不经意间延长倒数第二行创建的Foo对象的生命期。
	
	
## 函数参数

因为要修改引用计数（而且拷贝的时候通常要加锁），shared_ptr 的拷贝开销比拷贝原始指针要高，但是需要拷贝的时候并不多。多数情况下它可以以 const reference 方式传递，一个线程只需要在最外层函数有一个实体对象，之后都可以用 const reference 来使用这个 shared_ptr。例如有几个函数都要用到 Foo 对象：
	
	![[Pasted image 20211210185700.png]]
	那么在通常情况下，我们可以传[[常引用]]（pass by const reference）：
	![[Pasted image 20211210185710.png]]
	
	
## 析构动作在创建时被捕获
这是一个非常有用的特性，这意味着：
	+ 虚析构不再是必需的。
	+ shared_ptr<void>可以持有任何对象，而且能安全地释放。
	+ shared_ptr 对象可以安全地跨越模块边界，比如从 DLL 里返回，而不会造成从模块 A 分配的内存在模块 B 里被释放这种错误。
	
	
## 析构所在的线程
 　对象的析构是同步的，当最后一个指向 x 的 shared_ptr 离开其作用域的时候，x 会同时在同一个线程析构。这个线程不一定是对象诞生的线程。这个特性是把双刃剑：如果对象的析构比较耗时，那么可能会拖慢关键线程的速度（如果最后一个 shared_ptr 引发的析构发生在关键线程）；同时，我们可以用一个单独的线程来专门做析构，通过一个 BlockingQueue<shared_ptr<void> >把对象的析构都转移到那个专用线程，从而解放关键线程。
	
	
## 现成的 RAII handle
 　==我认为 RAII（资源获取即初始化）是 C++语言区别于其他所有编程语言的最重要的特性==，一个不懂 RAII 的 C++程序员不是一个合格的 C++程序员。初学 C++的教条是“new 和 delete 要配对，new 了之后要记着 delete”；如果使用 RAII[CCS，条款 13]
 ，要改成“每一个明确的资源配置动作（例如new）都应该在单一语句中执行，并在该语句中立刻将配置获得的资源交给handle对象（如shared_ptr），程序中一般不出现delete”。shared_ptr是管理共享资源的利器，需要注意避免循环引用，==通常的做法是owner持有指向child的shared_ptr，child持有指向owner的[[weak_ptr]]。==


