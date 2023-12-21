`std::ref` 用于创建一个引用包装器，允许通过引用传递对象，并在调用函数时对其进行修改。这对于需要在函数内部修改对象的情况非常有用。

```c++
#include <iostream>
#include <functional>

void modifyValue(int& value) {
    value *= 2;
}

int main() {
    int x = 10;

    // 使用 std::ref 创建引用包装器
    std::ref(modifyValue)(x);

    // 输出修改后的 x 的值
    std::cout << "Modified x: " << x << std::endl;

    return 0;
}

```