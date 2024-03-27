
`std::declval` 是一个 C++ 标准库中的工具函数，用于**生成一个右值引用**，通常用于在 decltype 表达式中**获取类型而不需要创建实际对象**。它的主要用途是在某些情况下，当我们需要使用 decltype 推导出一种类型，但是我们并不想创建该类型的实例，而是希望使用该类型的右值引用。

声明：
```c++
template <typename T>
typename std::add_rvalue_reference<T>::type declval() noexcept;
```

例子：
```c++
#include <iostream>
#include <utility>

struct MyType {
    int value;
};

int main() {
    // 使用 decltype 推导 MyType 的右值引用类型
    using RvalueRefType = decltype(std::declval<MyType>());
    // 输出类型名
    std::cout << "Type name: " << typeid(RvalueRefType).name() << std::endl;
    return 0;
}

```