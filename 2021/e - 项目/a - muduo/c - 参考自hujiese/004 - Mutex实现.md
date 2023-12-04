# 004 - Mutex实现
```c++


  MutexLock()
    : holder_(0)
  {
    int ret = pthread_mutex_init(&mutex_, NULL);
    assert(ret == 0); (void) ret;
  }

  ~MutexLock()
  {
    assert(holder_ == 0);
    int ret = pthread_mutex_destroy(&mutex_);
    assert(ret == 0); (void) ret;
  }
  void lock()
  {
    pthread_mutex_lock(&mutex_);
    holder_ = CurrentThread::tid();
  }

  void unlock()
  {
    holder_ = 0;
    pthread_mutex_unlock(&mutex_);
  }

```

> pid_t holder_ 保存当前的tid值，初始化的时候，设置为0， lock() 设置为 线程id
>  调用 pthread_mutex_init 初始化mutex
>  pthread_mutex_destroy销毁
>  pthread_mutex_lock 加锁
>  pthread_mutex_unlock 解锁


```c++
class MutexLockGuard : boost::noncopyable
{
 public:
  explicit MutexLockGuard(MutexLock& mutex)
    : mutex_(mutex)
  {
    mutex_.lock();
  }

  ~MutexLockGuard()
  {
    mutex_.unlock();
  }

 private:

  MutexLock& mutex_;
};

```

MutexLock类只是简单地对mutex进行封装而已。