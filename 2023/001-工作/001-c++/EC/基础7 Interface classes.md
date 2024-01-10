  
接口类（Interface classes）是一种 C++ 中用于实现抽象接口的机制，通常涉及[[纯虚函数]]。接口类定义了一组纯虚函数，它们没有实际的实现，而是由继承该接口的具体类提供实现。这样的设计允许多个类共享相同的接口，但可以有不同的实现。

```c++
#include <iostream>

// 定义接口类
class Shape {
public:
    // 纯虚函数，表示接口
    virtual void draw() const = 0;

    // 虚析构函数，确保子类的析构函数被正确调用
    virtual ~Shape() {}
};

// 具体类：圆形
class Circle : public Shape {
public:
    // 实现接口中的纯虚函数
    void draw() const override {
        std::cout << "Drawing a circle." << std::endl;
    }
};

// 具体类：矩形
class Rectangle : public Shape {
public:
    // 实现接口中的纯虚函数
    void draw() const override {
        std::cout << "Drawing a rectangle." << std::endl;
    }
};

int main() {
    // 使用接口类指针
    Shape* shape1 = new Circle();
    Shape* shape2 = new Rectangle();

    // 调用接口方法，会根据实际类型调用对应的实现
    shape1->draw();
    shape2->draw();

    // 释放内存
    delete shape1;
    delete shape2;

    return 0;
}

```