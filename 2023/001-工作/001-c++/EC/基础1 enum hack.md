
> C++11 引入了 constexpr，使得更多的编译时计算可以直接在代码中使用，而不必依赖于 enum hack。

  
C++ enum hack（也称为 "enum trick" 或 "enum hack"）是一种技术，它允许在编译时计算某些值。这通常通过在枚举类型中定义一个带有特定值的成员，从而实现在编译时进行条件编译或计算。

```c++
template <int N>
struct Fibonacci {
    enum { value = Fibonacci<N-1>::value + Fibonacci<N-2>::value };
};

template <>
struct Fibonacci<0> {
    enum { value = 0 };
};

template <>
struct Fibonacci<1> {
    enum { value = 1 };
};

```