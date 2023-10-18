# shared_ptr 如何用于 [[Pimpl]]

首先我们声明一个类 sample，它仅向外界暴露了最小的细节，其真正的实现在内部类 impl，sample 用一个 [[shared_ptr]] 来保存它的指针：

---

头文件：
```c++
class sample{

private:
	class impl;				// 不完整的内部类声明
	shared_ptr<impl> p;		// shared_ptr成员变量
public:
	smaple();				// 构造函数
	void print();			// 提供给外界的接口
}
```

---

在 sample 的 cpp 中完整定义 impl 和其他功能：
```c++
class sample::impl{					// 内部类的实现
public:
	void print(){ cout << "impl print" << endl;}		
};

sample::sample():p(new impl){}      // 构造函数初始化shared_ptr

void sample::print() { p->print();}	// 调用pimpl实现的print()

```


---

最后是 [[001 - 桥接模式]]的使用，具体如下：
```c++
sample s;
s.print();
```


桥接模式非常有用，它可以任意改变具体的实现并使外界对此一无所知，同时它可以减少源文件之间的编译依赖，使程序获得更多的灵活性。而 shared_ptr 是实现桥接模式的最佳工具，它解决了指针的共享问题和引用计数问题