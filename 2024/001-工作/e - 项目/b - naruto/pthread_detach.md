- Date: 2021-12-25
- Time:  22:07

1.linux线程执行和windows不同，pthread有两种状态joinable状态和unjoinable状态，如果线程是joinable状态，当线程函数自己返回退出时或pthread_exit时都不会释放线程所占用堆栈和线程描述符（总计8K多）。只有当你调用了pthread_join之后这些资源才会被释放。若是unjoinable状态的线程，这些资源在线程函数退出时或pthread_exit时自动会被释放。

2.unjoinable属性可以在pthread_create时指定，或在线程创建后在线程中pthread_detach自己, 如：pthread_detach(pthread_self())，将状态改为unjoinable状态，确保资源的释放。或者将线程置为 joinable,然后适时调用pthread_join.

3.其实简单的说就是在线程函数头加上 pthread_detach(pthread_self())的话，线程状态改变，在函数尾部直接 pthread_exit线程就会自动退出。省去了给线程擦屁股的麻烦。








[参考自：](https://blog.csdn.net/weibo1230123/article/details/81410241)