[[one loop per thread + thread pool]]

[[multiple reactor + thread pool]]

多个reactor是共享一个线程池，所以就说明了 reactor in processes 是不可以的，因为多个进程不能共享一个线程池

![[Pasted image 20211231035958.png]]