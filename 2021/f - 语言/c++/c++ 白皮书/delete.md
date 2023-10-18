

## 1. 禁止使用编译器默认生成的函数
```c++
DataOnly (const DataOnly & rhs) = delete; //禁止使用该函数
DataOnly & operator=(const DataOnly & rhs) = delete; //禁止使用该函数
```
## 2. delete 关键字可用于任何函数，不仅仅局限于类的成员函数
```c++
void fun(int a) {} 
void fun(float a) = delete;
```

## 3.模板特化：在模板特例化中，可以用delete来过滤一些特定的形参类型