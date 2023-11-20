
专栏地址：[LLVM每日谈 - 知乎 (zhihu.com)](https://www.zhihu.com/column/llvm-clang)
官网：[The LLVM Compiler Infrastructure Project](https://llvm.org/)

---
## [LLVM每日谈之一 LLVM是什么](https://zhuanlan.zhihu.com/p/26127007)
## [LLVM每日谈之二 LLVM IR](https://zhuanlan.zhihu.com/p/26127100)
1. [Clang - Getting Started (llvm.org)](https://clang.llvm.org/get_started.html
	1. **clang -cc1 ~/t.c -ast-print**
		- `clang` ：Clang 编译器的命令行工具。
		- `-cc1` ：指示运行 Clang 的编译器前端而不是驱动程序。
		- `~/t.c` ：指定的 C 语言源代码文件的路径（`~/t.c`）。
		- `-ast-print` ：要求将源代码的抽象语法树打印到标准输出。
	2. **clang ~/t.c -S -emit-llvm -o -**
		- `-S`：指示生成汇编代码而不是目标文件。
		- `-emit-llvm`：指示生成 LLVM IR（Intermediate Representation，中间表示）代码。
		- `-o -`：将输出重定向到标准输出，即将生成的 LLVM 汇编代码输出到终端。
	3. **clang -fomit-frame-pointer -O3 -S -o - t.c**
		 -  `-fomit-frame-pointer ` ：启用帧指针的优化，这可以在一些情况下提高程序性能，但可能会影响调试
		 - `-O3` ：使用较高级别的优化级别（级别 3），对代码进行更激进的优化。
	1. 
2. 

