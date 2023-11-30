
[0. 扉页 — Google 开源项目风格指南 (zh-google-styleguide.readthedocs.io)](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/)

[1.5. ` #include ` 的路径及顺序]( https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/headers/#include )
	`dir/foo.cc` 或 `dir/foo_test.cc` 的主要作用是实现或测试 `dir2/foo2.h` 的功能, `foo.cc` 中包含头文件的次序如下:
		1. `dir2/foo2.h` (优先位置, 详情如下)
		2. C 系统文件
		3. C++ 系统文件
		4. 其他库的 `.h` 文件
		5. 本项目内 `.h` 文件

[2.5. 静态和全局变量](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/scoping/#section-6)
	禁止定义静态储存周期非 POD 变量
	禁止使用含有副作用的函数初始化 POD 全局变量
		因为多编译单元中的静态变量执行时的构造和析构顺序是未明确的，这将导致代码的不可移植。

