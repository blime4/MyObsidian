# 以 boost :: function 和 boost :: bind 取代虚函数


[[继承]]和[[虚函数]]是万恶之源，这条“贼船”上去就不容易下来。不过还好，在 C++里我们有别的办法: 以 boost :: function 和 boost :: bind 取代虚函数。

在 C++这种非 GC 语言中，使用虚函数作为事件回调接口有其本质困难，即如何管理派生类对象的生命期。这样会带来一个问题：如何才能保证此刻程序的其他地方没有保存着这个即将销毁的对象的指针呢？

---

## 基本用途

boost :: function 就像 C#里的 delegate (委托) 可以指向任何函数，包括成员函数。当用 bind 把某个成员函数绑到某个对象上时，我们得到了一个 closure（闭包）。例如：

```c++
class Foo{
public:
	void methodA();
	void methodInt(int a);
	void methodString(const string& str);
};

class Bar{
public:
	void methodB();
};

boost::funtion<void()> f1;		// 无参数，无返回值

Foo foo;
f1 = boost::bind(&Foo::methodA, &foo);
f1();							// 调用foo.methodA();

Bar bar;
f1 = boost::bind(&Bar::methodB, &bar);
f1();							// 调用bar.methodB();

f1 = boost::bind(&Foo::methodInt, &foo, 123);
f1();

boost::function<void(int)> f2;	// int 参数，无返回值
f2 = boost::bind(&Foo:methodInt, &foo, _1);				// 这里的_1是占位，还有_2,_3等
f2(53);							

```

---

## 例子 1 ： 线程库 （简单）

### 常规 OO 设计

写一个 Thread base class，含有（纯）虚函数 Thread :: run ()，然后应用程序派生一个 derived class，覆写 run ()。程序里的每一种线程对应一个 Thread 的派生类。例如 Java 的 Threadclass 可以这么用。

缺点：如果一个 class 的三个 method 需要在三个不同的线程中执行，就得写 helper class (es) 并玩一些 OO 把戏。

### 基于 boost :: function 的设计 　

令Thread是一个具体类，其构造函数接受ThreadCallback对象。
应用程序只需提供一个能转换为 ThreadCallback 的对象（可以是函数），即可创建一份 Thread 实体，然后调用 Thread :: start () 即可。
Java 的 Thread 也可以这么用，传入一个 Runnable 对象。
C#的 Thread 只支持这一种用法，构造函数的参数是 delegate ThreadStart。
boost :: thread 也只支持这种用法。

```c++
// 一个基于 boost::function 的 Thread class 基本结构
class Thread{
public:
	typedef boost::function<void()> ThreadCallback;
	Thread(ThreadCallback cb)
		: cb_(cb){}
	void start(){
		/* some magic to call run() in new created thread*/
	}
	
private:
	void run(){
		cb_();
	}
	ThreadCallback cb_;
}
```

> 其实写多了，就会发现，这样写的程序，通用模板就是：
> 1.  Typedef boost :: funtction<void ()> 为一个什么什么 Callback, 这里的参数也通常都是无参数 void ()
> 2. 在私有成员中定义什么什么 Callback的一个私有变量
> 3. 在类的构造函数的时候，填入需要绑定的回调函数。
> 
> 具体的例子可以参考：陈硕大佬 muduo 网络库的 muduo :: Channel ，muduo :: Acceptor 等。

上面的例子使用方式：
```c++
class Foo{
public:
	void runInThread();
	void runInAnotherThread(int);
};

Foo foo;
Thread thread1(boost::bind(&Foo::runInThread, &foo));
Thread thread2(boost::bind(&Foo::runInAnotherThread, &foo, 43));
thread1.start();
thread2.start();
```
> 简单解析一下：
> 使用 boost :: function 和 boost :: bind 的方法，就可以实现 Foo 和 Thread的解耦。
> 启动两个线程，线程对象可以直接使用 Foo的成员函数，而他们并非继承关系，妙不妙。

---

## 例子 2 ： 网络库 （困难）

以 boost :: function 作为桥梁，NetServer class 对其使用者没有任何类型上的限制，*只对成员函数的参数和返回类型有限制。* 使用者 EchoService 也完全不知道 NetServer 的存在，只要在 main () 里把两者装配到一起，程序就跑起来了。

```c++
class Connection;
class NetServer : boost :: nocopyable{
public:	
	typedef boost::function<void (Connection*)> ConnectionCallback; 
	typedef boost ::function<void (Connection*，const void*, int len)> MessageCallback;
	
	NetServer(uint16_t port);
	~NetServer();
	
	void registerConnectionCallback(const ConnectionCallback&);
	void registerMessageCallback(const MessageCallback&);
	
private:
	...
};
```

```c++
class EchoService{
public:
	typedef boost::function<void(Connection*, const void *,int )> SendMessageCallback;
	EchoService(const SendMessageCallback& sendMsgCb): sendMsgCb_(sendMsgCb) {}
	
	void onMessage(Connection* conn , const void* buf , int size){
		sendMsgCb_(conn, buf, size);
	}
	
	void onConnection(Connection* conn){
		printf("Connection xxx");
	}
	
private:
	SendMessageCallback sendMsgCb_;
}
```
```c++
int main(){
	NetServer server(7);
	EchoService echo(boost::bind(&NetServer::sendMessage, &server, _1,_2,_3));
	server.registerConnectionCallback(boost::bind(&EchoService::onConnection,&echo, _1));
	server.registerMessageCallback(boost::bind(&EchoService::onMessage,&echo, _1,_2,_3));
	server.run();
}
```
简单解析：
> 这里就可以看出，其实 function+bind 的技法，就是一个注册和监听的技法。
> server 注册了两个回调函数，将回调echo的两个成员函数。
> 这样就实现了 NetServer 和 EchoService 的解耦。


## [[058 - 什么时候使用继承]]
