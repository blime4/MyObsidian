# pthread_once

pthread_once 一般用于一次性的线程初始化，在整个声明周期中，该方法只执行一次，从而实现一种[[线程安全]]的[[单例模式]]。

## Pthread_once () 函数语法要点
+ 头文件		： `#include<pthread.h>`
+ 函数原型 ： `int pthread_once(pthread_once_t *once_control, void(*int_routine)(void));`
	+  `once_control` :  一个静态或全局变量，初始化为 PTHREAD_ONCE_INT
	+ `init_routine` : 初始化函数的函数指针
+ 返回值：
	+ 成功： 0
	+ 出错：错误码

## 例子：muduo 网络库的 Singleton 就用到了 pthread_once。

```c++
template<typename T>
class Singleton : boost::nocopyable{		
public:
	static T& instance(){
		pthread_once(&ponce_,&Singleton::init);			// 保证ponce_只会被执回调执行一次Singleton::init
		return *value_;
	}
private:
	Singleton();
	~Singleton();
	static void init(){
		value_ = new T();
	}
private:
	static pthread_once_t ponce_;
	static T* value_;
};

// 必须在头文件中定义static 变量

template<typename T>
pthread_once_t Singleton<T>::ponce_ = PTHREAD_ONCE_INIT;
template<typename T>
T* Singleton<T>::value_ = NULL;

```

## 注意事项
在使用 pthread_once () 函数时需要注意，不能在其回调函数中调用 fork ()，否则，当再次调用 pthread_once () 时，会在子进程中导致死锁。而且，不能在回调函数中抛出一个 [[C++异常]]。

---

## 参考
+ 《Android 技术内幕（系统卷）》
+ 《linux 多线程服务器编程》
+ 《嵌入式 Linux C 语言应用开发教程》


[[B站专栏]]
[[参考]]