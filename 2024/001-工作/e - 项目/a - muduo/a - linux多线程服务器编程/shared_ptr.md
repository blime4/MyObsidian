


Shared_ptr 是引用计数型智能指针，在 Boost 和 std :: tr1 里均提供，也被纳入 C++11 标准库，现代主流的 C++编译器都能很好地支持。shared_ptr<T>是一个类模板（class template），它只有一个类型参数，使用起来很方便。引用计数是自动化资源管理的常用手法，当引用计数降为 0 时，对象（资源）即被销毁。[[weak_ptr]] 也是一个引用计数型智能指针，但是它不增加对象的引用次数，即弱（weak）引用。

---
	
## Shared_ptr 本身不是 100％线程安全的
---

虽然我们借shared_ptr来实现线程安全的对象释放，但是shared_ptr本身不是100％线程安全的。它的引用计数本身是安全且无锁的，但对象的读写则不是，因为shared_ptr有两个数据成员，读写操作不能原子化。
 ---
	
shared_ptr 的线程安全级别和内建类型、标准库容器、std :: string 一样，即：
+ 一个 shared_ptr 对象实体可被多个线程同时读取
+ 两个 shared_ptr 对象实体可以被两个线程同时写入，“析构”算写操作；
+ 如果要从多个线程读写同一个 shared_ptr 对象，那么需要加锁。

	---
	
	
## 具体分析
	
	---
	
Shared_ptr 有两个专门的函数来检查引用计数：unique（）在 shared_ptr 是指针的唯一所有者时返回 true（这时 shared_ptr 的行为类似 scoped_ptr 或 unique_ptr）；use_count（）返回当前指针的引用计数。要小心，use_count（）应该仅仅用于测试或调试，它不提供高效率的操作，而且有的时候它是不可用的（极少数情形）。而 unique（）是可靠的，任何时候它都可用，而且它比 use_count（）==1 的速度更快。
	
	

[[make_shared]]（）函数可以接收若干个参数，然后把它们传递给类型T的构造函数，创建一个shared_ptr＜T＞的对象并返回。通常使用make_shared（）函数要比直接创建shared_ptr对象的方式快且高效，因为它内部仅分配一次内存，消除了shared_ptr构造时的“开销”。
	
```c++
	auto sp = make_shared<string>("make_shared");
	auto spv = make_shared<vector<int>>(10,2);
	assert(spv->size()==10);
```