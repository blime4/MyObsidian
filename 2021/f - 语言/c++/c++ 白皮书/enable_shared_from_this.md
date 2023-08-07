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

> 获得一个用 shared_ptr 管理过的 this 指针。自己管理自己。


ChatGPT

`std::enable_shared_from_this` 是 C++ 标准库中的一个模板类，用于在某个类中获取一个指向该类对象的 `std::shared_ptr`，即通过一个对象自身来获取一个共享指针的引用。这通常用于解决对象间的循环引用，尤其在使用 `std::shared_ptr` 进行内存管理时。

具体来说，当一个类继承自 `std::enable_shared_from_this`，这个类的对象就能够通过成员函数 `shared_from_this()` 来获取一个指向自身的 `std::shared_ptr`。这对于在类内部使用 `std::shared_ptr` 来管理对象的生命周期特别有用，因为它可以确保在对象还有相关的 `std::shared_ptr` 引用时不会被提前销毁。

这个机制在处理对象间的循环引用时非常有帮助，因为循环引用可能导致对象无法正常释放。通过在类中继承 `std::enable_shared_from_this` 并使用 `shared_from_this()`，可以避免出现循环引用造成的内存泄漏问题。

以下是一个简单的示例：

cppCopy code
```c++
#include <iostream>
#include <memory>

class MyClass : public std::enable_shared_from_this<MyClass> {
public:
    std::shared_ptr<MyClass> getShared() {
        return shared_from_this(); // 获取指向自身的 shared_ptr
    }
};

int main() {
    std::shared_ptr<MyClass> obj1 = std::make_shared<MyClass>();
    std::shared_ptr<MyClass> obj2 = obj1->getShared();

    std::cout << "Use count for obj1: " << obj1.use_count() << std::endl;
    std::cout << "Use count for obj2: " << obj2.use_count() << std::endl;

    return 0;
}

```


在这个示例中，`MyClass` 继承自 `std::enable_shared_from_this<MyClass>`，并在成员函数 `getShared()` 中使用 `shared_from_this()` 来获取一个指向自身的 `std::shared_ptr`。这确保了 `obj1` 和 `obj2` 共享同一个对象，并且它们的引用计数是正确的，避免了潜在的内存泄漏问题。