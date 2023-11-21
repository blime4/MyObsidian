链接：[比飞鸟贵重的多_HKL的个人空间-比飞鸟贵重的多_HKL个人主页-哔哩哔哩视频 (bilibili.com)](https://space.bilibili.com/218427631/channel/collectiondetail?sid=1849137)
## C01 
> 所有编程技术不断优化与升级都在做三件事
> 1. 提升效率
> 2. 减少重复
> 3. 减轻依赖

编译单元只有 c, cpp... 不需要头文件，头文件在预编译阶段。

## C02
Windows 编译动态库：
>1. `add_library(xxx SAHRD yyy.cpp)`
>2. `__declspec(dllexport)`
>3. `include_directories(./bin)`
>4. `add_executable(main main.cpp)`
>5. `target_lin_libraries(main ./bin/xxx.lib)`
>	1. 虽然是动态库，但这里链接的是 lib 而不是 dll
>




