[【CMake系列】（一）入门 | 须臾之学 (xizhibei.me)](https://blog.xizhibei.me/2020/03/09/cmake-1-introduction/) | √
```
cmake_minimum_required (VERSION 3.0)  
project (demo VERSION 0.1.0 DESCRIPTION "Demo project")  
# 添加 demo_lib 静态库  
add_libary(demo_lib STATIC lib/a.cpp lib/b.cpp)  
# 指定头文件所在位置  
target_include_directories(demo_lib PUBLIC ${CMAKE_SOURCE_DIR}/include}  
# 同上  
add_executable(demo main.cpp)  
# 这里不需要再次 target_include_directories 了，因为我们在设置了 include 是 demo_lib 需要的，CMake 会自动添加  
# 将 demo_lib 库链接至 demo 执行文件  
target_link_libraries(demo demo_lib)
```

[【CMake 系列】（二）第三方依赖管理 | 须臾之学 (xizhibei.me)](https://blog.xizhibei.me/2020/03/15/cmake-2-third-party-dependances-management/) | √
```
find_package(Boost 1.50 REQUIRED COMPONENTS filesystem)  
target_link_libraries(lib PUBLIC Boost::filesystem)
```

[【CMake 系列】（三）ExternalProject 实践 | 须臾之学 (xizhibei.me)](https://blog.xizhibei.me/2020/03/23/cmake-3-external-project-practise/) | √
[[vcpkg的使用]]

[【CMake 系列】（四）用 GoogleTest 测试 | 须臾之学 (xizhibei.me)](https://blog.xizhibei.me/2020/04/05/cmake-4-test-with-google-test/)
```

```