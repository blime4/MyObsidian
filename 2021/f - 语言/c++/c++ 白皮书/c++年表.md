[[c++第一个十年]]



- 1979 年：工作始于“带类的 C”，它变成了 C++；拥有了第一个非研究性的用户；
  - 语言：`class`、构造函数/析构函数、`public`/`private`、简单继承、函数参数类型检查
  - 库：`task`（协程和仿真支持）、用宏参数化的 `vector`
- 1985 年：C++ 的首次商业发行；TC++PL1 [Stroustrup 1985b]
  - 语言：`virtual` 函数、运算符重载、引用、常量
  - 库：`complex` 算法，流输入输出
- 1989–91 年：ANSI 和 ISO 标准化开始；TC++PL2 [Stroustrup 1991]
  - 语言：抽象类、多重继承、异常、模板
  - 库：输入输出流（但没有 `task`）
- 1998 年：C++98、第一个 ISO C++ 标准 [Koenig1998]、TC++PL3 [Stroustrup 1997]
  - 语言：`namespace`、具名类型转换[^1]、`bool`、`dynamic_cast`
  - 库：STL（容器和算法）、`string`、`bitset`
- 2011 年：C++11 [Becker 2011]，TC++PL4 [Stroustrup 2013]
  - 语言：内存模型、`auto`、范围 `for`、`constexpr`、lambda 表达式、用户定义字面量……
  - 库：`thread` 和锁、`future`、`unique_ptr`、`shared_ptr`、`array`、时间和时钟、随机数、无序容器（哈希表）……
- 2014 年：C++14 [du Toit 2014]
  - 语言：泛型 lambda 表达式、`constexpr` 函数中的局部变量、数字分隔符……
  - 库：用户定义字面量……
- 2017 年：C++17 [Smith 2017]
  - 语言：结构化绑定、变量模板、模板参数的构造函数推导……
  - 库：文件系统、`scoped_lock`、`shared_mutex`（读写锁）、`any`、`variant`、`optional`、`string_view`、并行算法……
- 2020 年：C++20 [Smith 2020]
  - 语言：`concept`、`module`、协程、三路比较、改进对编译期计算的支持……
  - 库：概念、范围、日期和时区、`span`、格式、改进的并发和并行支持……