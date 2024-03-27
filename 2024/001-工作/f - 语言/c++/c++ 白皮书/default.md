在程序员重载了自己上面提到的C++编译器默认生成的函数之后，C++编译器将不在生成这些函数的默认形式。 
但是C++允许我们使用=default来要求编译器生成一个默认函数，如

```c++
struct Point {
    Point()=default;
    Point(int _x, int _y) : x(_x), y(_y) {}
    int x = 0;
    int y = 0;
}
```

这样相当于无参的构造函数也是可以使用的