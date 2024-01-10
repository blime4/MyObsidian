C++模板元编程（TMP）是一种在编译时进行元编程的技术，其中通过使用模板和元函数来生成代码。Expression templates 是 TMP 的一种应用，它通常用于优化表达式的求值，特别是在数值计算中。

以下是一个简单的例子，演示了如何使用 expression templates 来表示和优化简单的向量加法表达式：

```c++
#include <iostream>

// 基本的向量类模板
template <typename T, size_t Size>
class Vector {
public:
    T data[Size];

    Vector() {}

    // 构造函数，用于初始化向量
    Vector(const T& value) {
        for (size_t i = 0; i < Size; ++i) {
            data[i] = value;
        }
    }

    // 运算符重载，实现向量加法
    Vector<T, Size> operator+(const Vector<T, Size>& other) const {
        Vector<T, Size> result;
        for (size_t i = 0; i < Size; ++i) {
            result.data[i] = data[i] + other.data[i];
        }
        return result;
    }

    // 打印向量的成员函数
    void print() const {
        for (size_t i = 0; i < Size; ++i) {
            std::cout << data[i] << " ";
        }
        std::cout << std::endl;
    }
};

// 表达式模板类
template <typename T, size_t Size, typename Expr>
class VectorExpression {
public:
    // 求值函数，用于计算表达式的值
    void evaluate(Vector<T, Size>& result) const {
        static_cast<const Expr&>(*this).evaluate(result);
    }
};

// 表达式模板：向量加法
template <typename T, size_t Size>
class VectorAdditionExpression : public VectorExpression<T, Size, VectorAdditionExpression<T, Size>> {
public:
    const Vector<T, Size>& v1;
    const Vector<T, Size>& v2;

    VectorAdditionExpression(const Vector<T, Size>& a, const Vector<T, Size>& b) : v1(a), v2(b) {}

    // 求值函数，实现向量加法
    void evaluate(Vector<T, Size>& result) const {
        for (size_t i = 0; i < Size; ++i) {
            result.data[i] = v1.data[i] + v2.data[i];
        }
    }
};

// 运算符重载，用于创建表达式模板
template <typename T, size_t Size>
VectorAdditionExpression<T, Size> operator+(const Vector<T, Size>& a, const Vector<T, Size>& b) {
    return VectorAdditionExpression<T, Size>(a, b);
}

int main() {
    // 定义两个向量
    Vector<double, 3> v1(1.0);
    Vector<double, 3> v2(2.0);

    // 使用表达式模板进行向量加法
    Vector<double, 3> result = v1 + v2;

    // 打印结果
    std::cout << "Result: ";
    result.print();

    return 0;
}

```

在这个例子中，`Vector`类表示一个简单的向量，支持向量加法。`VectorExpression`类是一个基类，用于表示表达式模板，而`VectorAdditionExpression`类表示向量加法表达式。通过使用`VectorExpression`和`VectorAdditionExpression`，我们可以在不实际执行向量加法的情况下构建和优化表达式，从而在编译时进行性能优化。