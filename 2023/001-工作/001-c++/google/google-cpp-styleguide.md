
[0. 扉页 — Google 开源项目风格指南 (zh-google-styleguide.readthedocs.io)](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/)

[1.5. ` #include ` 的路径及顺序]( https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/headers/#include )
>  `dir/foo.cc` 或 `dir/foo_test.cc` 的主要作用是实现或测试 `dir2/foo2.h` 的功能, `foo.cc` 中包含头文件的次序如下:

> 	1. `dir2/foo2.h` (优先位置, 详情如下)
> 	2. C 系统文件
> 	3. C++ 系统文件
> 	4. 其他库的 `.h` 文件
> 	5. 本项目内 `.h` 文件