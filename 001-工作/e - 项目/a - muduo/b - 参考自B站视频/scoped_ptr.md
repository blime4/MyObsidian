# boost :: scoped_ptr 

---

一个[[智能指针]]

Scoped_ptr 是一个与 [[auto_ptr]]/[[unique_ptr]] 很类似的智能指针，它包装了 new 操作符在堆上分配的动态对象，能够保证动态创建的对象在任何时候都可以被正确地删除。但 scoped_ptr 的所有权更加严格，不能转让，一旦 scoped_ptr 获取了对象的管理权，我们就无法再从它那里收回来。

---

> 不能转让性，是由将拷贝构造函数，赋值操作，相等操作私有化实现的。

```c++
template<class T>
class scoped_ptr{
private:
	T* px;											// 原始指针
	scoped_ptr(scoped_ptr const &);					// 拷贝构造函数私有化
	scoped_ptr & operator = (scoped_ptr const &);	// 赋值操作私有化
	void operator == (scoped_ptr const &) const;	// 相等操作私有化

}
```

---

Scoped_ptr 拥有一个很好的名字，它向代码的阅读者传递了明确的信息：这个智能指针只能在本作用域里使用，不希望被转让。

---

除 “* ” “-＞”之外，scoped_ptr 没有定义其他操作符，所以不能对 scoped_ptr 进行“++”“--”等指针算术操作。与普通指针相比，它只有很小的接口，这一点使得使用指针更加安全、更容易且不容易被误用。



---

标准容器不能容纳 scoped_ptr，因为 scoped_ptr 不能拷贝和赋值。标准容器可以容纳原始指针，但这就丧失了容器的许多好处，因为标准容器无法自动管理指针类型的元素，所以必须编写大量额外的代码来保证指针最终被正确删除，这通常很麻烦而且容易出错。存储 shared_ptr 的容器与存储原始指针的容器的功能几乎一样，但 [[shared_ptr]] 为程序员做了指针的管理工作，可以任意使用 shared_ptr 而不用担心资源泄漏。