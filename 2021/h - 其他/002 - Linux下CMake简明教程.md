
- Date: 2021-12-23
- Time:  13:06

# 001 - Linux下CMake简明教程

CMake是开源、跨平台的构建工具，可以让我们通过编写简单的配置文件去生成本地的Makefile，这个配置文件是**独立于运行平台和编译器**的，这样就不用亲自去编写Makefile了，而且配置文件可以直接拿到其它平台上使用，无需修改，非常方便。

## 一 安装CMake
`sudo apt install cmake`

## 二 简单总结
1. CMakeLists.txt
	1. cmake_minimum_required (VERSION 2.8)
	2. project (demo)
	3. **aux_source_directory (dir, var)**。|  不建议用法
		1. 把指定目录下所有的源文件存储在一个变量中
		2. 弊端，会把目录下所有源文件都加进来，替代品：set
		3. 弊端，增量编译会有问题
		4. cmake 官方文档，明确不建议这么使用
	4. set( SRC_LIST ./main.c ./testFunc1.c ./testFunc.c)
	5. add_executable(main ${SRC_LIST})
	6. 不同目录的话
		1. include_directories (test_func test_func1)
		2. aux_source_directory (test_func SRC_LIST)
		3. aux_source_directory (test_func1 SRC_LIST1)
		4. add_executable (main main.c ${SRC_LIST} ${SRC_LIST1})
	7. 正规一点的组织结构
		1. add_subdirectory (src)
			1. 可以向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制的存放位置
		2. set (EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
	8. 动态库和静态库的编译控制
		1. set (SRC_LIST ${PROJECT_SOURCE_DIR}/testFunc/testFunc.c)
		2. add_library (testFunc_shared SHARED ${SRC_LIST}
		3. add_library (testFunc_static STATIC ${SRC_LIST})
		4. set_target_properties (testFunc_shared PROPERTIES OUTPUT_NAME "testFunc")
		5. set_target_properties (testFunc_static PROPERTIES OUTPUT_NAME "testFunc")
		6. set (LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
2. cmake .
3. make
4. (删除) make clean



[参考自：](https://blog.csdn.net/whahu1989/article/details/82078563)



### 例子 ：

```Cmake
cmake_minimum_required(VERSION 3.0)

project(naruto)

set(CXX_FLAGS
 -O2
 -g
 -Wall
 -Wextra
 -Werror
 -std=c++14
 -pthread
)

# ---[ Flags
string(REPLACE ";" " " CMAKE_CXX_FLAGS "${CXX_FLAGS}")

# ---[ Includes
# include directories
INCLUDE_DIRECTORIES(
 ${PROJECT_SOURCE_DIR}/src
 ${PROJECT_SOURCE_DIR}/include
 ${PROJECT_SOURCE_DIR}/src/util
 /usr/local/include
 /usr/include
)

LINK_DIRECTORIES(
 ${PROJECT_BINARY_DIR}/libs  
 /usr/local/lib
 /use/lib
)

SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/libs)

# ---[ Subdirectories
add_subdirectory(src)
add_subdirectory(test)

# add google test lib
add_subdirectory(third_party/googletest)
enable_testing()
include_directories(${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})


message(STATUS "CMAKE_CXX_FLAGS=" "${CMAKE_CXX_FLAGS}")
message(STATUS "PROJECT_BINARY_DIR=" "${PROJECT_BINARY_DIR}")
message(STATUS "PROJECT_SOURCE_DIR=" "${PROJECT_SOURCE_DIR}")
message(STATUS "CMAKE_BINARY_DIR=" "${CMAKE_BINARY_DIR}")
```

```CMake
PROJECT(naruto)

file(GLOB_RECURSE srcs ${PROJECT_SOURCE_DIR} *.cpp )

# static library
ADD_LIBRARY(naruto STATIC ${srcs})
SET_TARGET_PROPERTIES(naruto PROPERTIES OUTPUT_NAME "naruto")
SET_TARGET_PROPERTIES(naruto PROPERTIES CLEAN_DIRECT_OUTPUT 1)
```




[参考](https://www.bilibili.com/video/BV1vR4y1u77h?p=3&spm_id_from=pageDriver)
[[从零开始详细介绍CMake.pdf]]



