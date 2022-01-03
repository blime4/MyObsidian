# 007 - BlockingQueue实现


究其本质，muduo_BlockinngQueue利用了条件变量来进行阻塞队列的操作，比较简单。

```c++
 private:
  mutable MutexLock mutex_;
  Condition         notEmpty_;
  std::deque<T>     queue_;
```

通过[[mutex]]实现[[条件变量]]，通过条件变量，实现[[阻塞队列]]


```c++
 void put(const T& x)
  {
    MutexLockGuard lock(mutex_);
    queue_.push_back(x);
    notEmpty_.notify(); // TODO: move outside of lock
  }

  T take()
  {
    MutexLockGuard lock(mutex_);
    // always use a while-loop, due to spurious wakeup
    while (queue_.empty())
    {
      notEmpty_.wait();
    }
    assert(!queue_.empty());
    T front(queue_.front());
    queue_.pop_front();
    return front;
  }

  size_t size() const
  {
    MutexLockGuard lock(mutex_);
    return queue_.size();
  }
```

[[008 - BoundedBlockingQueue实现]]


