

[[future]]

[[async 的底层实现：promise]]
## async
+ std:async接受一个带返回值的 lambda，自身返回一个std:future对象。
+ lambda的函数体将在另一个线程里执行。
+ 接下来你可以在main里面做一些别的事情， download会持续在后台悄悄运行。
+ 最后调用 future的get0方法，如果此时download还没完成，会等待 download完成，并获取 download的返回值。

```c++


std::future<int> fret = std::async([&]{
	return download("hello.zip");
});
interact();
int ret = fret.get();
std::cout<<ret<<std::endl;


```

fret.get()会阻塞等待一下，或者可以
## 显示地等待：wait()
除了get()会等待线程执行完毕外，wait()也可以等待他执行完，但是不会返回其值。
```c++

std::future<int> fret = std::async([&]{
	return download("hello.zip");
});
interact();
std::cout<<"waiting for download complete..."<<std::endl;
fret.wait();
std::cout<<"wait returned!"<<std::endl;
int ret = fret.get();
std::cout<<ret<<std::endl;


```

![[Pasted image 20220113114148.png]]



## 等待一段时间 ： wait_for()

+ 只要线程没有执行完，wait()会无限等下去
+ 而 wait_for()则可以指定一个最长等待时间，用 chrono里的类表示单位。他会返回std:future_status表示等待是否成功。
+ 如果超过这个时间线程还没有执行完毕，则放弃等待，返回 future status:timeout
+ 如果线程在指定的时间内执行完毕，则认为等待成功，返回 future status:ready。
+ 同理还有 wait_until其参数是一个时间点。

```c++

std::future<int> fret = std::async([&]{
	return download("hello.zip");
});

interact();

while(1){
	std::cout<<"waiting for download complete..."<<std::endl;
	auto stat = fret.wait_for(std::chrono::milliseconds(1000));
	if(stat == std::future_status::ready){
		std::cout<<"Future is ready!"<<std::endl;
	} else {
		std::cout<<"Future not ready!"<<std::endl;
	}
}

int ret = fret.get();
std::cout<<ret<<std::endl;

```

线程式异步👆

## 惰性求值，函数式异步，而非线程式异步👇

## std::launch::deferred做参数

>+ std:async的第一个参数可以设为std:launch:deferred，这时不会创建一个线程来执行，他只会把 lambda函数体内的运算推迟到 future的get()被调用时。也就是main中的 Interact计算完毕后。
>+ 这种写法， download的执行仍在主线程中，他只是函数式编程范式意义上的异步，而不涉及到真正的多线程。可以用这个实现惰性求值（ lazy evaluation）之类。
```c++

std::future<int> fret = std::async(std::launch::defered, [&]{
	return download("hello.zip");
});

interact();

int ret = fret.get();

std::cout<<ret<<std::endl;
```

![[Pasted image 20220113115742.png]]