[[002 - Linux下CMake简明教程]]

cmake 例子分析：
001 ： set (UPSTREAM_FIND_CUDA_DIR "${CMAKE_CURRENT_LIST_DIR}/upstream/")
```bash
这是一行CMake脚本代码，用于设置一个CMake变量的值。让我们逐步解释它：

1.  `set`: 这是CMake的一个命令，用于设置一个变量的值。
    
2.  `UPSTREAM_FIND_CUDA_DIR`: 这是要设置的变量名。在这里，变量名为"UPSTREAM_FIND_CUDA_DIR"。
    
3.  `"${CMAKE_CURRENT_LIST_DIR}/upstream/"`: 这是要赋给变量的值。`${CMAKE_CURRENT_LIST_DIR}`是一个CMake内置变量，表示当前处理的CMakeLists.txt所在的目录。在这里，它将指向CMakeLists.txt所在的目录。然后，通过附加"/upstream/"字符串，将变量设置为当前目录下的"upstream/"子目录的路径。
    

总结起来，这行代码将变量`UPSTREAM_FIND_CUDA_DIR`设置为CMakeLists.txt所在目录下的"upstream/"子目录的路径。在后续的CMake脚本中，可以使用`${UPSTREAM_FIND_CUDA_DIR}`来引用这个路径。
```

