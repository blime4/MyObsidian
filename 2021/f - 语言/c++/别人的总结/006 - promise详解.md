# 006 - [[promise]] 详解
# std :: promise

promise 对象可以保存某一类型 T 的值，该值可被 [[005 - future详解]] 对象读取（可能在另外一个线程中），因此 promise 也提供了一种[[线程同步]]的手段。在 promise 对象构造时可以和一个共享状态（通常是std::future）相关联，并可以在相关联的共享状态(std::future)上保存一个类型为 T 的值。

可以通过 get_future 来获取与该 promise 对象相关联的 future 对象，调用该函数之后，两个对象共享相同的共享状态(shared state)

* promise 对象是异步 [[Provider]]，它可以在某一时刻设置共享状态的值。
* future 对象可以异步返回共享状态的值，或者在必要的情况下阻塞调用者并等待共享状态标志变为 ready，然后才能获取共享状态的值。

下面以一个简单的例子来说明上述关系：
```c
#include <iostream>       // std::cout
#include <functional>     // std::ref
#include <thread>         // std::thread
#include <future>         // std::promise, std::future
//#include <Windows.h>
void print_int(std::future<int>& fut) {
	int x = fut.get(); // 获取共享状态的值.
	std::cout << "value: " << x << '\n'; // 打印 value: 10.
}

int main()
{
	std::promise<int> prom; // 生成一个 std::promise<int> 对象.
	std::future<int> fut = prom.get_future(); // 和 future 关联.
	std::thread t(print_int, std::ref(fut)); // 将 future 交给另外一个线程t.
	/*Sleep(1000);*/
	prom.set_value(10); // 设置共享状态的值, 此处和线程t保持同步.
	t.join();
	return 0;
}
```
该例子传递了一个std::future变量fut给另一个线程，注意，我前面的代码特地注释掉了Sleep(1000)，如果去掉该注释，线程t需要等待1s时间，主线程中将future的值设置为10后才会输出结果。这便是异步调用的思想。

## 1、std::promise 构造函数

类型|函数格式|说明
---|---|---
default (1)	|promise();|默认构造函数，初始化一个空的共享状态
with allocator (2)	|template ` <class Alloc>`  promise (allocator_arg_t aa, const Alloc& alloc);|带自定义内存分配器的构造函数，与默认构造函数类似，但是使用自定义分配器来分配共享状态
copy [deleted] (3)	|promise (const promise&) = delete;|拷贝构造函数，被禁用
move (4)	|promise (promise&& x) noexcept;|移动构造函数

另外，std :: promise 的 operator= **没有拷贝语义**，即 std :: promise 普通的赋值操作被禁用，operator= **只有 move 语义**，所以 std ::promise 对象是禁止拷贝的。

---

案例:
```c
#include <iostream>       // std::cout
#include <thread>         // std::thread
#include <future>         // std::promise, std::future

std::promise<int> prom;

void print_global_promise() {
	std::future<int> fut = prom.get_future();
	int x = fut.get();
	std::cout << "value: " << x << '\n';
}

int main()
{
	std::thread th1(print_global_promise);
	prom.set_value(10);
	th1.join();

	prom = std::promise<int>();    // prom 被move赋值为一个新的 promise 对象.

	std::thread th2(print_global_promise);
	prom.set_value(20);
	th2.join();

	return 0;
}
```

## 2、std::promise::get_future

该函数返回一个与 promise 共享状态相关联的 future 。返回的 future 对象可以访问由 promise 对象设置在共享状态上的值或者某个异常对象。只能从 promise 共享状态获取一个 future 对象。

在调用该函数之后，promise 对象通常会在某个时间点准备好 (设置一个值或者一个异常对象)，如果不设置值或者异常，promise 对象在析构时会自动地设置一个 [[future_error]] 异常 ([[broken_promise]]) 来设置其自身的准备状态。上面的例子中已经提到了 get_future，此处不再重复。

## 3、std::promise::set_value
* generic template (1)
    ```c
    void set_value (const T& val); 
    void set_value (T&& val);
    ```
* specializations (2)
    ```c
    void promise<R&>::set_value (R& val);   // when T is a reference type (R&)  
    void promise<void>::set_value (void);   // when T is void
    ```

设置共享状态的值，此后 promise 的共享状态标志变为 ready。

## 4、std::promise::set_exception

下面这个案例中，线程1从终端接收一个整数，线程2将该整数打印出来，如果线程1接收一个非整数，则为promise设置一个异常(failbit) ，线程2通过std::future::get 中抛出该异常：
```c
#include <iostream>       // std::cin, std::cout, std::ios
#include <functional>     // std::ref
#include <thread>         // std::thread
#include <future>         // std::promise, std::future
#include <exception>      // std::exception, std::current_exception

void get_int(std::promise<int>& prom) {
	int x;
	std::cout << "Please, enter an integer value: ";
	std::cin.exceptions(std::ios::failbit);   // throw on failbit 设置一个异常，故障位failbit
	try {
		std::cin >> x;                         // sets failbit if input is not int
		prom.set_value(x);
	}
	catch (std::exception&) {
		prom.set_exception(std::current_exception());
	}
}

void print_int(std::future<int>& fut) {
	try {
		int x = fut.get();
		std::cout << "value: " << x << '\n';
	}
	catch (std::exception& e) {
		std::cout << "[exception caught: " << e.what() << "]\n";
	}
}

int main()
{
	std::promise<int> prom;
	std::future<int> fut = prom.get_future();

	std::thread th1(get_int, std::ref(prom));
	std::thread th2(print_int, std::ref(fut));

	th1.join();
	th2.join();
	return 0;
}
```
如果正常输入整数，结果如下：
```
Please, enter an integer value: 6
value: 6
```
如果输入的是字符或者其他东西，那么：
```
Please, enter an integer value: a
[exception caught: ios_base::failbit set: iostream stream error]
```

## 5、std::promise::set_value_at_thread_exit

设置共享状态的值，但是不将共享状态的标志设置为 ready，当线程退出时该 promise 对象会自动设置为 ready。如果某个 std::future 对象与该 promise 对象的共享状态相关联，并且该 future 正在调用 get，则调用get的线程会被阻塞，当线程退出时，调用future::get 的线程解除阻塞，同时 get 返回 set_value_at_thread_exit 所设置的值。注意，该函数已经设置了 promise 共享状态的值，如果在线程结束之前有其他设置或者修改共享状态的值的操作，则会抛出 [[future_error]]( [[promise_already_satisfied]] )。

## 6、std::promise::swap

交换 promise 的共享状态。