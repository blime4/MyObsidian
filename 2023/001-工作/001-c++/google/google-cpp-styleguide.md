
## [0. 扉页 — Google 开源项目风格指南 (zh-google-styleguide.readthedocs.io)](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/)

## [1.5. ` #include ` 的路径及顺序]( https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/headers/#include )
	`dir/foo.cc` 或 `dir/foo_test.cc` 的主要作用是实现或测试 `dir2/foo2.h` 的功能, `foo.cc` 中包含头文件的次序如下:
		1. `dir2/foo2.h` (优先位置, 详情如下)
		2. C 系统文件
		3. C++ 系统文件
		4. 其他库的 `.h` 文件
		5. 本项目内 `.h` 文件

## [2.5. 静态和全局变量](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/scoping/#section-6)

**禁止定义静态储存周期非 POD 变量：**
- **静态存储周期（Static Storage Duration）：** 静态存储周期的变量在程序的整个运行周期内存在，其内存分配在程序启动时进行，直到程序结束时才被释放。这包括全局变量和静态变量。
- **非POD变量（Non-POD Variable）：** POD（Plain Old Data）是一种 C++ 的术语，指的是没有用户定义的构造函数、析构函数和包含的对象也是 POD 类型的数据类型。如果一个变量是非POD，它可能包含用户定义的构造函数、析构函数等。

> 禁止定义静态存储周期的非 POD 变量的规则可能是为了防止在程序启动或结束时发生复杂的初始化和清理操作，以确保这些变量在其生命周期内的状态是可控的。

**禁止使用含有副作用的函数初始化 POD 全局变量：**

- **POD全局变量：** 类似上述，这指的是包含简单数据类型的全局变量，而不涉及复杂的构造和析构。
- **含有副作用的函数：** 意味着该函数在执行时会导致不仅仅是返回一个值，还会对其他程序状态产生影响，可能是修改全局变量、文件 I/O、或者其他可能影响程序状态的操作。

> 这个规则的目的可能是为了确保在初始化 POD 全局变量时不引入复杂的副作用，以避免在程序启动时出现不可预测的行为。使用含有副作用的函数初始化全局变量可能导致不确定的初始化顺序或者在多线程环境下的竞态条件问题。
