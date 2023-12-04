[[RAII]]

# 参考资料

- [热心观众整理的学习资料](https://github.com/jiayaozhang/OpenVDB_and_TBB)
- [C++ 官方文档](https://en.cppreference.com/w/)
- [C++ 核心开发规范](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md)
- [LearnCpp 中文版](https://learncpp-cn.github.io/)
- [C++ 并发编程实战](https://www.bookstack.cn/read/Cpp_Concurrency_In_Action/README.md)
- [因特尔 TBB 编程指南](https://www.inf.ed.ac.uk/teaching/courses/ppls/TBBtutorial.pdf)
- [并行体系结构与编程 (CMU 15-418)](https://www.bilibili.com/video/av48153629/)
- [深入理解计算机原理 (CSAPP)](http://csapp.cs.cmu.edu/)
- [CMake “菜谱”](https://www.bookstack.cn/read/CMake-Cookbook/README.md)
- [CMake 官方文档](https://cmake.org/cmake/help/latest/)
- [Git 官方文档](https://git-scm.com/doc)
- [GitHub 官方文档](https://docs.github.com/en)

当代：C++17 CTAD / compile-time argument deduction / 编译期参数推断
![[Pasted image 20230113001327.png]]
![[Pasted image 20230113001334.png]]

---


当代：C++17 引入常用数值算法
![[Pasted image 20230113001418.png]]

未来：C++20 引入区间（ranges）
![[Pasted image 20230113001700.png]]

未来：C++20 引入模块（module）
![[Pasted image 20230113001730.png]]

未来：C++20 允许函数参数为自动推断（auto）
![[Pasted image 20230113001802.png]]


[[使用 {} 和 () 调用构造函数，有什么区别？]]

•一旦我们定义了自己的构造函数，编译器就不会再生成默认的无参构造函数。
![[Pasted image 20230113003735.png]]
![[Pasted image 20230113003742.png]]

有自定义构造函数时仍想用默认构造函数：= default（续）
•如果还想让编译器自动生成默认的无参构造函数，可以用 C++11 新增的这个语法：
![[Pasted image 20230113003828.png]]