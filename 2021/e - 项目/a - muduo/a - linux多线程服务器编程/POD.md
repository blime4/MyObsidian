
POD 类型是 C++中常见的概念，用来说明类/结构体的属性，具体来说它是指没有使用面相对象的思想来设计的类/结构体。POD 的全称是 Plain Old Data，Plain 表明它是一个普通的类型，没有虚函数虚继承等特性；Old 表明它与 C 兼容。


POD类型在C++中有两个独立的特性：

-   支持静态初始化（static initialization）
-   拥有和 C 语言一样的内存布局（memory layout）


参考：https://zhuanlan.zhihu.com/p/56161728