反汇编：
```
objdump -D libkineto.a | less
```
打印共享库依赖
```
ldd xxx.so
```

```cmake
cmake_minimun_required(VERSION 3.12)
project(hellocmake LANGUAGES C CXX)

add_library(hellolib STATIC hello.cpp)
add_executable(a.out main.cpp)
target_link_libraries(a.out PUBLIC hellolib)
target_include_directories(a.out PUBLIC hellolib)
target_add_definitions(myapp PUBLIC MY_MACRO=1)
target_add_definitions(myapp PUBLIC -DMY_MACRO=1)
target_compile_options(myapp PUBLIC -fopenmp)
target_sources(myapp PUBLIC hello.cpp other.cpp)

以及可以通过下列指令（不推荐使用）：👇 不推荐使用👇
include_directories(/opt/cuda/include)
link_directories(/opt/cuda)
add_definitions(MY_MACRO=1)
add_compile_options(-fopenmp)
👆 不推荐使用

# 子库

add_subdirectory(fmt)
add_executable(a.out main.cpp)
target_link_libraries(a.out PUBLIC fmt)
```

[[header-only]] 

## find_pa