

[[MutexLock]]封装临界区（critical section），这是一个简单的资源类，用[[003 - RAII机制]]手法
 封装互斥器的创建与销毁。临界区在Windows上是struct CRITICAL_SECTION，是[[可重入]]的；在Linux下是pthread_mutex_t，默认是不可重入的2
 。MutexLock一般是别的class的数据成员。
[[MutexLockGuard]] 封装临界区的进入和退出，即加锁和解锁。==MutexLockGuard 一般是个栈上对象，它的作用域刚好等于临界区域。==
> 这就能很好的实现栈回调析构函数

这两个class都不允许拷贝构造和赋值，
