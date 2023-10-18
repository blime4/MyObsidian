noexcept

-   [9.2 noexcept 的修饰和操作](https://changkun.de/modern-cpp/zh-cn/09-others/#9-2-noexcept-%E7%9A%84%E4%BF%AE%E9%A5%B0%E5%92%8C%E6%93%8D%E4%BD%9C)


C++11 将异常的声明简化为以下两种情况：

1.  函数可能抛出任何异常
2.  函数不能抛出任何异常

并使用 `noexcept` 对这两种行为进行限制，例如：
```c++
void may_throw(); // 可能抛出异常  
void no_throw() noexcept; // 不可能抛出异常
```


使用 `noexcept` 修饰过的函数如果抛出异常，编译器会使用 `std::terminate()` 来立即终止程序运行。

`noexcept` 还能够做操作符，用于操作一个表达式，当表达式无异常时，返回 `true`，否则返回 `false`。

