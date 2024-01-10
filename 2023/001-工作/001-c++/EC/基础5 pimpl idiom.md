Pimpl（Pointer to Implementation）习惯用法是一种 C++设计模式，旨在将类的实现细节（implementation details）与其公共接口分离。这种模式的核心思想是使用指向实现类的指针，将类的私有实现部分封装起来，从而隐藏实现细节并提高类的封装性。这对于库的二进制兼容性和编译时间的优化都是有益的。

```c++

// MyClass.h

class MyClass {
public:
    MyClass();  // 构造函数
    ~MyClass(); // 析构函数

    void doSomething(); // 公共接口

private:
    // 私有实现类的前向声明
    class Impl;

    // 指向实现类的指针
    Impl* pImpl;
};


```

```c++

// MyClass.cpp

#include "MyClass.h"

// 实现类的定义
class MyClass::Impl {
public:
    void privateFunction() {
        // 实现细节
    }
};

// MyClass 构造函数的实现
MyClass::MyClass() : pImpl(new Impl()) {
    // 构造函数的其他初始化工作
}

// MyClass 析构函数的实现
MyClass::~MyClass() {
    delete pImpl; // 释放实现类的内存
}

// 公共接口的实现，通过指针调用实现类的函数
void MyClass::doSomething() {
    pImpl->privateFunction();
    // 其他操作
}

```