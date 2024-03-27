[[BoundedBlockingQueue]]
[[007 - BlockingQueue实现]]

```c++

 private:
  mutable MutexLock          mutex_;
  Condition                  notEmpty_;
  Condition                  notFull_;
  boost::circular_buffer<T>  queue_;


public:
  explicit BoundedBlockingQueue(int maxSize)
    : mutex_(),
      notEmpty_(mutex_),
      notFull_(mutex_),
      queue_(maxSize)
  {
  }

  void put(const T& x)
  {
    MutexLockGuard lock(mutex_);
    while (queue_.full())
    {
      notFull_.wait();
    }
    assert(!queue_.full());
    queue_.push_back(x);
    notEmpty_.notify(); // TODO: move outside of lock
  }

  T take()
  {
    MutexLockGuard lock(mutex_);
    while (queue_.empty())
    {
      notEmpty_.wait();
    }
    assert(!queue_.empty());
    T front(queue_.front());
    queue_.pop_front();
    notFull_.notify(); // TODO: move outside of lock
    return front;
  }

  bool empty() const
  {
    MutexLockGuard lock(mutex_);
    return queue_.empty();
  }

```