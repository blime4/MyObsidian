`std::cref` 用于创建一个常量引用包装器，允许通过常量引用传递对象，但不允许在函数内部修改对象。这对于确保对象在传递过程中不被修改的情况非常有用。

```c++
#include <iostream>
#include <functional>

void printValue(const int& value) {
    std::cout << "Value: " << value << std::endl;
}

int main() {
    int x = 42;

    // 使用 std::cref 创建常量引用包装器
    std::cref(printValue)(x);

    return 0;
}
```