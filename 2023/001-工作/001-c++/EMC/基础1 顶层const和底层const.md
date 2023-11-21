![[Pasted image 20231121182247.png]]
1. 对象拷贝：底层 const 必须一致。
	1. 在C++中，底层const（low-level const）指的是指针或引用所指向的对象是不可修改的，而顶层const（top-level const）指的是对象本身是不可修改的。在对象拷贝时，如果涉及到底层const，那么拷贝的对象的底层const属性必须与被拷贝的对象的底层const属性一致，否则可能导致编译错误或者意外行为。
2. 