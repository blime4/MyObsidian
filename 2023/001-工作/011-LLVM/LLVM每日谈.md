
专栏地址：[LLVM每日谈 - 知乎 (zhihu.com)](https://www.zhihu.com/column/llvm-clang)
官网：[The LLVM Compiler Infrastructure Project](https://llvm.org/)

---
## [LLVM每日谈之一 LLVM是什么](https://zhuanlan.zhihu.com/p/26127007)
## [LLVM每日谈之二 LLVM IR](https://zhuanlan.zhihu.com/p/26127100)
1. [Clang - Getting Started (llvm.org)](https://clang.llvm.org/get_started.html
	1. **clang -cc1 ~/t.c -ast-print**
		1. 具体解释如下
			- `clang` ：Clang 编译器的命令行工具。
			- `-cc1` ：指示运行 Clang 的编译器前端而不是驱动程序。
			- `~/t.c` ：指定的 C 语言源代码文件的路径（`~/t.c`）。
			- `-ast-print` ：要求将源代码的抽象语法树打印到标准输出。
	2. **clang ~/t.c -S -emit-llvm -o -**
		1. 具体解释如下：
			- `-S`：指示生成汇编代码而不是目标文件。
			- `-emit-llvm`：指示生成 LLVM IR（Intermediate Representation，中间表示）代码。
			- `-o -`：将输出重定向到标准输出，即将生成的 LLVM 汇编代码输出到终端。
	1. 
2. 

