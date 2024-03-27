![[Pasted image 20211231035345.png]]


[[round robin]]

根据千兆网口，确定线程数
能适应更大的突发IO



[[multiple reactor]]方案有两种
一种是：    [[reactor]] in threads ， 又叫 [[one loop per thread]]
另一种是：reactor in processes ，这种就不能加线程池了


第一种加上线程池，就是muduo推荐的
[[multiple reactor + thread pool ]]
就是 [[one loop per thread + thread pool]]
