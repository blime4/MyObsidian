# weak_ptr

---

weak_ptr 不控制对象的生命期，但是它知道对象是否还活着（想象成用棉线轻轻拴住堆上的对象）。如果对象还活着，那么它可以提升（promote）为有效的 shared_ptr；如果对象已经死了，提升会失败，返回一个空的 [[shared_ptr]]。“提升／lock ()”行为是线程安全的。

---

```cpp
template<class T>
class weak_ptr{
public:
	weak_ptr();
	template<class Y> weak_ptr(shared_ptr<Y> const & r);
	weak_ptr(weak_ptr const & r);
	~weak_ptr();
	weak_ptr & operator = (weak_ptr const & r);
	
	long use_count() const ;		// 引用计数
	bool expired() const ; 	 		// 是否为失效指针
	bool empty() const ;  	 		// 是否为空指针， 非标准
	shared_ptr<T> lock() const; 	// 获取。提示为shared_ptr;
	
	void reset();
	boid swap(weak_ptr<T> & b);

};


```

---

Weak_ptr 被设计为与 shared_ptr 协同工作，可以从一个 shared_ptr 或另一个 weak_ptr 对象构造以获得资源的观测权。但 weak_ptr 没有共享资源，它的构造不会引起指针引用计数的增加。同样，weak_ptr 析构时也不会导致引用计数减少，它只是一个“静静的观察者”。

---

## 用法

---

使用 weak_ptr 的成员函数 use_count（）可以观测资源的引用计数，另一个成员函数 expired（）的功能等价于 use_count（）== 0，但 expired（）更快，它表示被观测的资源（也就是被 shared_ptr 管理的资源）已经不复存在。

---


weak_ptr 没有重载 operator* 和-＞，因为它不共享指针，不能操作资源，这正是它“弱”的原因。


---

## Lock ()函数

但它可以使用一个非常重要的成员函数 lock（）从被观测的 shared_ptr 获得一个可用的 shared_ptr 对象，把“弱”关系转换为“强”关系，从而操作资源。但当 expired（）== true 时，lock（）函数将返回一个存储空指针的 shared_ptr。

---

```cpp

shared_ptr<int> sp(new int(10)); 		// 一个shared_ptr
assert(sp.use_count() == 1);

weak_ptr<int> wp(sp); 					// 从shared_ptr创建weak_ptr
assert(wp.use_count() == 1);			// weak_ptr不影响引用计数
assert(!wp.empty());					// weak_ptr此时为空指针

if(!wp.expired()){
	shared_ptr<int> sp2 = wp.lock();	// 获取一个shared_ptr
	*sp2 = 100;
	assert(wp.use_count()==2);			
}										// 退出作用域，sp2 自动析构，引用计数减一

assert(wp.use_count() == 1);
sp.reset();
assert(wp.expired());
assert(!wp.lock()); 					// weak_ptr将获得一个空指针

```

---

## 对象自我管理

---

Weak_ptr 的一个==重要用途是获得 this 指针的 shared_ptr，==使对象自己能够生产 shared_ptr 管理自己：对象使用 weak_ptr 观测 this 指针，这并不影响引用计数，在需要的时候就调用 lock（）函数，返回一个符合要求的 shared_ptr 供外界使用。

---

这个解决方案是一种惯用法，在头文件＜boost/enable_shared_from_this.hpp＞里定义一个助手类[[enable_shared_from_this]]＜T＞

---


## 打破循环引用 （面试常问）

---

有时代码中可能会出现[[循环引用]]，这时 shared_ptr 的引用计数机制就会失效，导致无法正确释放资源

---

例子：
```cpp
class node{									// 一个用于链表节点的类
public:
	~node(){ cout << "deleted" <<endl; }
	typedef shared_ptr<node> ptr_type;		// 指针类型使用shared_ptr
	ptr_type next;							// 后继指针
}；

int main(){
	auto p1 = make_shared<node>();			// 两个节点对象
	auto p2 = make_shared<node>();
	
	p1->next = p2;							// 形成循环链表
	p2->next = p1;
	
	assert(p1.use_count() == 2);			// 每个shared_ptr的引用计数都是2
	assert(p2.use_count() == 2);
}											// 退出作用域，shared_ptr 无法正常析构
```

---

在上述代码中，两个节点对象互相持有对方的引用，每一个shared_ptr的引用计数都是2，因此在析构时引用计数没有减至0，不会调用删除操作，不会导致内存泄漏。

---

这时我们可以使用weak_ptr，因为它不会增加智能指针的引用计数，这样就把原来的“强”引用改成“弱”引用，在可能存在循环引用的地方打破了循环，而在真正需要shared_ptr的时候调用weak_ptr的lock（）函数：

---

```cpp
class node{                                 // 一个用于链表节点的类
public:
    ~node(){ cout << "deleted" <<endl; }
    typedef weak_ptr<node> ptr_type;      	// 指针类型使用shared_ptr
    ptr_type next;                          // 后继指针
}；

int main(){
    auto p1 = make_shared<node>();          // 两个节点对象
    auto p2 = make_shared<node>();
    
    p1->next = p2;                          // 形成循环链表
    p2->next = p1;							// 使用了 weak_ptr ，所以正常
    
    assert(p1.use_count() == 1);            // 每个shared_ptr的引用计数都是1
    assert(p2.use_count() == 1);			// 没有了循环引用
	
	if(!p1->next().expired()){				// 检查 弱引用 是否有效
		auto p3 = p1->next.lock();			// 调用 lock() 获取强引用 
	}
}                                           // 退出作用域，shared_ptr 正常析构

```


