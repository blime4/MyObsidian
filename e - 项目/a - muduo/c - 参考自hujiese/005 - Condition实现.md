# 005 - Condition实现

```c++
class Condition : boost::noncopyable
{
 public:
  explicit Condition(MutexLock& mutex)
    : mutex_(mutex)
  {
    pthread_cond_init(&pcond_, NULL);
  }

  ~Condition()
  {
    pthread_cond_destroy(&pcond_);
  }

  void wait()
  {
    pthread_cond_wait(&pcond_, mutex_.getPthreadMutex());
  }

  // returns true if time out, false otherwise.
  bool waitForSeconds(int seconds);

  void notify()
  {
    pthread_cond_signal(&pcond_);
  }

  void notifyAll()
  {
    pthread_cond_broadcast(&pcond_);
  }

 private:
  MutexLock& mutex_;
  pthread_cond_t pcond_;
};
```

只是对pthread库的封装
+ MutexLock& mutex_;
+ pthread_cond_t pcond_;

+ pthread_cond_init(&pcond_, NULL);
+ pthread_cond_destroy(&pcond_);
+ pthread_cond_wait(&pcond_, mutex_.getPthreadMutex());
+ pthread_cond_signal(&pcond_);
+ pthread_cond_broadcast(&pcond_);


条件变量，需要[[mutex]]来锁和解锁实现阻塞，通知