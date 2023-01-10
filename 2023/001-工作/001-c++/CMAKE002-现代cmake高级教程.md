## usage
![[截图_20230110102257.png]]
![[截图_20230110102615.png]]
![[截图_20230110102740.png]]
```
cmake -B build -G Ninja
```

## 添加源文件
1. 添加一个可执行文件作为构建目标
	```
	add_executable(main main.cpp)
	```
2. 先创建目标，再添加源文件
	```
	add_executable(main)
	target_sources(main PUBLIC main.cpp)
	```
3. 启用 CONFIGURE_DEPENDS 选项，当添加新文件时，自动更新变量
	```
	add_executable(main)
	file(GLOB sources CONFIGURE_DEPENDS *.cpp *.h)
	target_sources(main PUBLIC ${sources})
	```
4. 