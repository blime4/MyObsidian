[[async]]
[[future]]
[[promise]]


+ 如果不想让 std::async 帮你自动创建线程，想要手动创建线程，可以直接用std::promise
+ 然后在线程返回的时候，用 set_value()设置返回值。在主线程里，用 get_future()获取其std::future对象，进一步get()可以等待并获取线程返回值。


```c++

std::promise<int> pret;
std::thread t1([&]{
	auto ret = download("hello.zip");
	pret.set_value(ret);
});
std::future<int> fret = pret.get_future();

interact();
int ret = fret.get();
std::cout<<ret<<std::endl;

t1.join();

```