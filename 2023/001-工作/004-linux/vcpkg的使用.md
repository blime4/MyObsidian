这里必须狠狠夸夸微软，今年来给开源界贡献了非常多的产品以及工具，vcpkg 就是其中的一个。

简单来说，它就是基于 CMake 的一个包管理工具，在安装完成后，一条命令就能安装绝大多数的依赖，也有越来越多的第三方包也开始接受并提供 vcpkg 的支持。虽然还没有囊括所有的第三方包，交叉编译也存在一定的问题，但是我相信如果接下来能有包管理工具统一 C/C++ 的话，vcpkg 就是候选之一。

它的安装很简单：
```
git clone https://github.com/Microsoft/vcpkg.git  
cd vcpkg  
./bootstrap-vcpkg.sh  
```

如果需要安装包：
```
vcpkg install opencv  
```
然后在你的项目中，使用 `find_package` 即可：
```
find_package(OpenCV REQUIRED)  
target_link_libraries(target ${OpenCV_LIBS})
```
  

最后，在 CMake 配置阶段：
```
cmake .. -DCMAKE_TOOLCHAIN_FILE=path/to/vcpkg/scripts/buildsystems/vcpkg.cmake  
```

就可以了，对于 OpenCV 这类有着系统支持的包，或许没有提升太多的效率，但是对于一些没有多少系统支持的包，比如 GoogleTest、nlohmann-json、prometheus-cpp 等，再也不需要费劲去自己下载编译了。

之后，为了方便随时运行 vcpkg，可以将它加到系统 PATH：
```
export PATH="path/to/vcpkg/:$PATH"  
```

是不是很简单？只是，目前它还是不完善，我之所以不在现有的团队项目中使用它，就是因为交叉编译的问题没解决，它目前支持的交叉编译不多，比如面对 Android 还有 iOS 的交叉编译，它就基本上成了废物一般，打算等日后它完善了再用。

另外，它相对于我上面的方案来说，编译时间还是不能省去的。