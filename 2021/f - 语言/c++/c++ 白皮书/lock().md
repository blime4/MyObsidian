

锁对象还提供了一种防止最常见形式的死锁的方法：

```cpp
void f()
{
    // ...
    unique_lock<mutex> lck1 {m1,defer_lock};  // 还未得到 m1
    unique_lock<mutex> lck2 {m2,defer_lock};
    unique_lock<mutex> lck3 {m3,defer_lock};
    // ...
    lock(lck1,lck2,lck3);  // 获取所有三个互斥锁
    // ... 操作共享数据 ...
}   // 隐式释放所有互斥锁
```

这里，`lock()` 函数“同时”获取所有 `mutex` 并隐式释放所有[[互斥锁]]