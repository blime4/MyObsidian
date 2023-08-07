
[PyTorch系列「一」PyTorch JIT —— trace/ script的代码组织和优化方法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/410507557)

## trace 和 script 的区别：

1、trace只记录走过的tensor和对tensor的操作，不会记录任何控制流信息，如if条件句和循环。因为没有记录控制流的另外的路，也没办法对其进行优化。好处是trace深度嵌入python语言，复用了所有python的语法，在计算流中记录数据流。

2、script会去理解所有的code，真正像一个编译器一样去进行lexer、parser、Semantic analusis的分析「也就是词法分析语法分析句法分析，形成AST树，最后再将AST树线性化」。script相当于一个嵌入在Python/Pytorch的DSL，其语法只是pytorch语法的子集，这意味着存在一些op和语法script不支持，这样在编译的时候就会遇到问题。此外，script的编译优化方式更像是CPU上的传统编译优化，重点对于图进行硬件无关优化，并对IF、loop这样的statement进行优化。