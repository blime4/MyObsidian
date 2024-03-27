# std :: condition_variable
std :: condition_variable 即条件变量。Linux 下使用 Pthread 库中的 pthread_cond_t 及其相关函数提供了与条件变量相关的功能。C++11 提供的条件变量用法和 Pthrad 库类似，都需要一个互斥锁配合使用，原理也是一样的。

```c
#include <iostream>                // std::cout
#include <thread>                // std::thread
#include <mutex>                // std::mutex, std::unique_lock
#include <condition_variable>    // std::condition_variable

std::mutex mtx; // 全局相互排斥锁.
std::condition_variable cv; // 全局条件变量.
bool ready = false; // 全局标志位.

void do_print_id(int id)
{
	std::unique_lock <std::mutex> lck(mtx);
	while (!ready) // 假设标志位不为 true, 则等待...
            cv.wait(lck); // 当前线程被堵塞, 当全局标志位变为 true 之后,
	// 线程被唤醒, 继续往下运行打印线程编号id.
	std::cout << "thread " << id << '\n';
}

void go()
{
	std::unique_lock <std::mutex> lck(mtx);
	ready = true; // 设置全局标志位为 true.
	cv.notify_all(); // 唤醒全部线程.
}

int main()
{
	std::thread threads[10];
	// spawn 10 threads:
	for (int i = 0; i < 10; ++i)
		threads[i] = std::thread(do_print_id, i);

	std::cout << "10 threads ready to race...\n";
	go(); // go!

	for (auto & th : threads)
		th.join();

	return 0;
}
```
运行结果如下：
```
10 threads ready to race...
thread 9
thread 7
thread 5
thread 3
thread 1
thread 0
thread 2
thread 6
thread 4
thread 8
```
需要注意的是，std::condition_variable 的拷贝构造函数被禁用，仅仅提供了默认构造函数。
## 2、std::condition_variable::wait

函数原型如下：
```c
void wait (unique_lock<mutex>& lck);
template <class Predicate>
void wait (unique_lock<mutex>& lck, Predicate pred)
```

std::condition_variable提供了两种 wait() 函数。

当前线程调用 wait() 后将被堵塞(此时当前线程应该获得了锁（mutex），最好还是设获得锁 lck)，直到另外某个线程调用 notify_* 唤醒了当前线程。

在线程被堵塞时，该函数会自己主动调用 lck.unlock() 释放锁，使得其它被堵塞在锁竞争上的线程得以继续运行。另外，一旦当前线程获得通知(notified，一般是另外某个线程调用 notify_* 唤醒了当前线程)，wait()函数也是自己主动调用 lck.lock()。

在另外一种情况下（即设置了 Predicate）。仅仅有当 pred 条件为 false 时调用 wait () 才会堵塞当前线程。而且在收到其它线程的通知后仅仅有当 pred 为 true 时才会被解除堵塞

案例如下：
```c
#include <iostream>                // std::cout
#include <thread>                // std::thread, std::this_thread::yield
#include <mutex>                // std::mutex, std::unique_lock
#include <condition_variable>    // std::condition_variable

std::mutex mtx;
std::condition_variable cv;

int cargo = 0;
bool shipment_available()
{
	return cargo != 0;
}

// 消费者线程.
void consume(int n)
{
	for (int i = 0; i < n; ++i) {
		std::unique_lock <std::mutex> lck(mtx);
		cv.wait(lck, shipment_available);
		std::cout << cargo << '\n';
		cargo = 0;
	}
}

int main()
{
	std::thread consumer_thread(consume, 10); // 消费者线程.

	// 主线程为生产者线程, 生产 10 个物品.
	for (int i = 0; i < 10; ++i) {
		while (shipment_available())
			std::this_thread::yield();
		std::unique_lock <std::mutex> lck(mtx);
		cargo = i + 1;
		cv.notify_one();
	}

	consumer_thread.join();

	return 0;
}
```
运行结果如下：
```
1
2
3
4
5
6
7
8
9
10
```

## 3、std::condition_variable::wait_for

函数原型如下：
```c
template <class Rep, class Period>
cv_status wait_for (unique_lock<mutex>& lck, const chrono::duration<Rep,Period>& rel_time);

template <class Rep, class Period, class Predicate>
bool wait_for (unique_lock<mutex>& lck, const chrono::duration<Rep,Period>& rel_time, Predicate pred);
```

与 std :: condition_variable :: wait () 相似，只是 wait_for 能够指定一个时间段，在当前线程收到通知或者指定的时间 rel_time 超时之前。该线程都会处于堵塞状态。而一旦超时或者收到了其它线程的通知，wait_for 返回，剩下的处理步骤和 wait () 相似。

请看以下的样例（參考），以下的样例中，主线程等待th线程输入一个值。然后将th线程从终端接收的值打印出来。在th线程接受到值之前，主线程一直等待。每一个一秒超时一次，并打印一个 "."：

```cpp
#include <iostream>           // std::cout
#include <thread>             // std::thread
#include <chrono>             // std::chrono::seconds
#include <mutex>              // std::mutex, std::unique_lock
#include <condition_variable> // std::condition_variable, std::cv_status

std::condition_variable cv;

int value;

void do_read_value()
{
	std::cin >> value;
	cv.notify_one();
}

int main()
{
	std::cout << "Please, enter an integer (I'll be printing dots): \n";
	std::thread th(do_read_value);

	std::mutex mtx;
	std::unique_lock<std::mutex> lck(mtx);
	while (cv.wait_for(lck, std::chrono::seconds(1)) == std::cv_status::timeout) {
		std::cout << '.';
		std::cout.flush();
	}

	std::cout << "You entered: " << value << '\n';

	th.join();
	return 0;
}
```
运行结果如下：
```
Please, enter an integer (I'll be printing dots):
...6.
You entered: 6
```

## 4、std::condition_variable::notify_one

唤醒某个等待(wait)线程。假设当前没有等待线程，则该函数什么也不做，假设同一时候存在多个等待线程，则唤醒某个线程是不确定的(unspecified)：

```C
#include <iostream>                // std::cout
#include <thread>                // std::thread
#include <mutex>                // std::mutex, std::unique_lock
#include <condition_variable>    // std::condition_variable

std::mutex mtx;
std::condition_variable cv;

int cargo = 0; // shared value by producers and consumers

void consumer()
{
	std::unique_lock < std::mutex > lck(mtx);
	while (cargo == 0)
		cv.wait(lck);
	std::cout << cargo << '\n';
	cargo = 0;
}

void producer(int id)
{
	std::unique_lock < std::mutex > lck(mtx);
	cargo = id;
	cv.notify_one();
}

int main()
{
	std::thread consumers[10], producers[10];

	// spawn 10 consumers and 10 producers:
	for (int i = 0; i < 10; ++i) {
		consumers[i] = std::thread(consumer);
		producers[i] = std::thread(producer, i + 1);
	}

	// join them back:
	for (int i = 0; i < 10; ++i) {
		producers[i].join();
		consumers[i].join();
	}

	return 0;
}
```


## 5、std::condition_variable::notify_all

唤醒全部的等待(wait)线程。假设当前没有等待线程，则该函数什么也不做。请看以下的样例：

```c
#include <iostream>                // std::cout
#include <thread>                // std::thread
#include <mutex>                // std::mutex, std::unique_lock
#include <condition_variable>    // std::condition_variable

std::mutex mtx; // 全局相互排斥锁.
std::condition_variable cv; // 全局条件变量.
bool ready = false; // 全局标志位.

void do_print_id(int id)
{
	std::unique_lock <std::mutex> lck(mtx);
	while (!ready) // 假设标志位不为 true, 则等待...
		cv.wait(lck); // 当前线程被堵塞, 当全局标志位变为 true 之后,
	// 线程被唤醒, 继续往下运行打印线程编号id.
	std::cout << "thread " << id << '\n';
}

void go()
{
	std::unique_lock <std::mutex> lck(mtx);
	ready = true; // 设置全局标志位为 true.
	cv.notify_all(); // 唤醒全部线程.
}

int main()
{
	std::thread threads[10];
	// spawn 10 threads:
	for (int i = 0; i < 10; ++i)
		threads[i] = std::thread(do_print_id, i);

	std::cout << "10 threads ready to race...\n";
	go(); // go!

	for (auto & th : threads)
		th.join();

	return 0;
}
```