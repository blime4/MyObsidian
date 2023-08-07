# C++11 新特性之十：enable_shared_from_this

[[this 指针]]
[[shared_ptr]]
[[weak_ptr]]

```c++

template<class T>
class enable_shared_from_this{			// 辅助类，需要继承使用
public:
	shared_ptr<T> shared_from_this();	// 工厂函数，产生this指针的shared_ptr
}
```

> 获得一个用 shared_ptr 管理的 this指针。自己管理自己。
