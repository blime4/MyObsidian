C/C++的二进制兼容性（[[binary compatibility]]）有多重含义，本文主要在“库文件单独升级，现有可执行文件是否受影响”这个意义下讨论，==我称之为 library（主要是 shared library，即动态链接库）的 [[ABI]]（application binary interface）。==


C++编译器ABI的主要内容包括以下几个方面：
+ 函数参数传递的方式，比如 x86-64 用寄存器来传函数的前 4 个整数参数；
+ 虚函数的调用方式，通常是 vptr/vtbl 机制，然后用 vtbl[offset]来调用；
+ struct 和 class 的内存布局，通过偏移量来访问数据成员；
+ [[name mangling]]；
+ RTTI 和异常处理的实现（以下本文不考虑异常处理）。