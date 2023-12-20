
**[[Bitwise Constness]] ([[Physical Constness]]):**

- 在C++中，一个成员函数被声明为const时，它意味着在该函数内**不能修改对象的非静态成员**。这是通过在函数的声明或定义中使用const关键字实现的。
- Bitwise constness更注重物理层面，即在函数中是否发生了**对对象二进制表示的任何修改**。
- 这种 const 性质通过编译器来强制执行，编译器确保在 const 成员函数中不能修改对象的非 const 成员。

```c++
class MyClass {
public:
    void memberFunction() const {
        // 在这里不能修改非const成员变量
    }
};
```

**Logical Constness:**

- Logical constness关注的是更高层次的语义，即在用户的角度上，是否感知到对象的修改。即使在const成员函数中，逻辑上可能依然可以修改对象的某些状态，但这种修改不会对用户可见。
- 这通常通过在const成员函数中使用mutable关键字来实现，该关键字允许在const函数中修改被声明为mutable的成员。