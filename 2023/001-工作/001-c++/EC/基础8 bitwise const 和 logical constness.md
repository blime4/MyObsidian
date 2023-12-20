
**Bitwise Constness (Physical Constness):**

- 在C++中，一个成员函数被声明为const时，它意味着在该函数内不能修改对象的非静态成员。这是通过在函数的声明或定义中使用const关键字实现的。
- Bitwise constness更注重物理层面，即在函数中是否发生了对对象二进制表示的任何修改。
- 这种const性质通过编译器来强制执行，编译器确保在const成员函数中不能修改对象的非const成员。