# 003 - Thread实现

### 一、线程标识符

Linux中，每个进程有一个pid，类型pid_t，由getpid()取得。Linux下的POSIX线程也有一个id，类型pthread_t，由pthread_self()取得，该id由线程库维护，其id空间是各个进程独立的（即不同进程中的线程可能有相同的id）。Linux中的POSIX线程库实现的线程其实也是一个进程（LWP），只是该进程与主进程（启动线程的进程）共享一些资源而已，比如代码段，数据段等。

有时候我们可能需要知道线程的真实pid。比如进程P1要向另外一个进程P2中的某个线程发送信号时，既不能使用P2的pid，更不能使用线程的pthread id，而只能使用该线程的真实pid，称为tid。

有一个函数gettid()可以得到tid，但glibc并没有实现该函数，只能通过Linux的系统调用syscall来获取。 return syscall(SYS_gettid)

### 二、pthread_atfork ###
```c++
#include <pthread.h>
int pthread_atfork(void (*prepare)(void), void (*parent)(void), void (*child)(void));
```


调用fork时，内部创建子进程前在父进程中会调用prepare，内部创建子进程成功后，父进程会调用parent ，子进程会调用child。

### 三、Thread实现(部分代码)

```c++
typedef boost::function<void ()> ThreadFunc;
explicit Thread(const ThreadFunc&, const string& name = string());
~Thread();
static int numCreated() { return numCreated_.get(); }


pid_t gettid()
{
  return static_cast<pid_t>(::syscall(SYS_gettid));
}


// start --> pthread_create
void Thread::start()
{
  assert(!started_);
  started_ = true;
  errno = pthread_create(&pthreadId_, NULL, &startThread, this);
  if (errno != 0)
  {
    //LOG_SYSFATAL << "Failed in pthread_create";
  }
}
// join ---> pthread_create
int Thread::join()
{
  assert(started_);
  return c(pthreadId_, NULL);
}
```

简单总结：
1. 封装了pthread库，使得start，join,可以直接调用
2. 调用了原子库，线程数原子加减
3. 缓存了tid,避免每次系统调用




```c++
#ifndef _THREAD_H_
#define _THREAD_H_

#include <pthread.h>
#include <boost/function.hpp>

class Thread
{
public:
        typedef boost::function<void ()> ThreadFunc;
        explicit Thread(const ThreadFunc& func);

        void Start();
        void Join();

        void SetAutoDelete(bool autoDelete);

private:
        static void* ThreadRoutine(void* arg);
        void Run();
        ThreadFunc func_;
        pthread_t threadId_;
        bool autoDelete_;
};

#endif // _THREAD_H_
```

```c++
#include "Thread.h"
#include <iostream>
using namespace std;


Thread::Thread(const ThreadFunc& func) : func_(func), autoDelete_(false)
{
}

void Thread::Start()
{
        pthread_create(&threadId_, NULL, ThreadRoutine, this);
}

void Thread::Join()
{
        pthread_join(threadId_, NULL);
}

void* Thread::ThreadRoutine(void* arg)
{
        Thread* thread = static_cast<Thread*>(arg);
        thread->Run();
        if (thread->autoDelete_)
                delete thread;
        return NULL;
}

void Thread::SetAutoDelete(bool autoDelete)
{
        autoDelete_ = autoDelete;
}

void Thread::Run()
{
        func_();
}
```

使用：
```c++

void ThreadFunc()
{
        cout<<"ThreadFunc ..."<<endl;
}

void ThreadFunc2(int count)
{
        while (count--)
        {
                cout<<"ThreadFunc2 ..."<<endl;
                sleep(1);
        }
}


int main(void)
{
        Thread t1(ThreadFunc);
        Thread t2(boost::bind(ThreadFunc2, 3));
        Foo foo(3);
        Thread t3(boost::bind(&Foo::MemberFun, &foo));
        Foo foo2(3);
        Thread t4(boost::bind(&Foo::MemberFun2, &foo2, 1000));

        t1.Start();
        t2.Start();
        t3.Start();
        t4.Start();

        t1.Join();
        t2.Join();
        t3.Join();

```