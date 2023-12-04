



[[线程安全]] 
[[2021/e - 项目/a - muduo/a - linux多线程服务器编程/Singleton]]
[[pthread_once]]

```c++

template<typename T>
class Singleton : boost::nocopyable{
public:
	static T& instance(){
		pthread_once(&ponce_,&Singleton::init);
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


使用方法：
```c++
Foo& foo = Singleton<Foo>::instance();
```