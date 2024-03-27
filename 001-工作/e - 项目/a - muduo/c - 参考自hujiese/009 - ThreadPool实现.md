# 009 - ThreadPool实现

[[threadpool]]

首先程序创建线程池：
```c++=
muduo::ThreadPool pool("MainThreadPool");
```
这一步将调用线程池类的构造函数：
```c++
explicit ThreadPool(const string& name = string());

ThreadPool::ThreadPool(const string& name)
  : mutex_(),
    cond_(mutex_),
    name_(name),
    running_(false)
{
}

```

初始化了互斥锁和条件变量，线程池名和运行标记。
接下来执行：
```c++
pool.start(5);
```
这一步调用了线程池的start函数：
```c++
void start(int numThreads);

void ThreadPool::start(int numThreads)
{
  assert(threads_.empty());
  running_ = true;
  threads_.reserve(numThreads);
  for (int i = 0; i < numThreads; ++i)
  {
    char id[32];
    snprintf(id, sizeof id, "%d", i);
    threads_.push_back(new muduo::Thread(
          boost::bind(&ThreadPool::runInThread, this), name_+id));
    threads_[i].start();
  }
}
```
该函数传入参数为线程池创建线程的数量，也就是说该线程池中的线程在线程池创建后就确定了，无法修改。在函数内部首先判断线程数组是否为空，线程数组如下：
```c++
<br class="Apple-interchange-newline"><div></div>

boost::ptr_vector<muduo::Thread> threads_;
```

该部分将创建的线程保存到线程数组中，然后对每一个线程数组中的成员调用start函数启动线程。需要注意的是每个线程绑定的函数：ThreadPool::runInThread，该函数定义如下：

```c++
void ThreadPool::runInThread()
{
  try
  {
    while (running_)
    {
      Task task(take());
      if (task)
      {
        task();
      }
    }
  }
  catch (const Exception& ex)
  {
    fprintf(stderr, "exception caught in ThreadPool %s\n", name_.c_str());
    fprintf(stderr, "reason: %s\n", ex.what());
    fprintf(stderr, "stack trace: %s\n", ex.stackTrace());
    abort();
  }
  catch (const std::exception& ex)
  {
    fprintf(stderr, "exception caught in ThreadPool %s\n", name_.c_str());
    fprintf(stderr, "reason: %s\n", ex.what());
    abort();
  }
  catch (...)
  {
    fprintf(stderr, "unknown exception caught in ThreadPool %s\n", name_.c_str());
    throw; // rethrow
  }
}
```
由于上一步已经将running_设置为true了，所以执行
```c++

Task task(take());

```

Task定义如下：
```c++
typedef boost::function<void ()> Task;
```

take函数定义如下：
```c++
ThreadPool::Task ThreadPool::take()
{
  MutexLockGuard lock(mutex_);
  // always use a while-loop, due to spurious wakeup
  while (queue_.empty() && running_)
  {
    cond_.wait();
  }
  Task task;
  if(!queue_.empty())
  {
    task = queue_.front();
    queue_.pop_front();
  }
  return task;
}
```