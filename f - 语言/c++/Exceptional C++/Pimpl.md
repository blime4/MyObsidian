
# Pimpl


（Pointer to Implementation）是通过一个私有的成员指针，将指针所指向的类的内部实现数据进行隐藏

### 例子 1 ：[[shared_ptr 如何用于 pimpl]] 

首先我们声明一个类 sample，它仅向外界暴露了最小的细节，其真正的实现在内部类 impl，sample 用一个 shared_ptr 来保存它的指针：

---

头文件：
```c++
class sample{

private:
	class impl;				// 不完整的内部类声明
	shared_ptr<impl> p;		// shared_ptr成员变量
public:
	smaple();				// 构造函数
	void print();			// 提供给外界的接口
}
```

---

在 sample 的 cpp 中完整定义 impl 和其他功能：
```c++
class sample::impl{					// 内部类的实现
public:
	void print(){ cout << "impl print" << endl;}		
};

sample::sample():p(new impl){}      // 构造函数初始化shared_ptr

void sample::print() { p->print();}	// 调用pimpl实现的print()

```


---

最后是 [[001 - 桥接模式]]的使用，具体如下：
```c++
sample s;
s.print();
```


桥接模式非常有用，它可以任意改变具体的实现并使外界对此一无所知，同时它可以减少源文件之间的编译依赖，使程序获得更多的灵活性。而 shared_ptr 是实现桥接模式的最佳工具，它解决了指针的共享问题和引用计数问题


### 例子 2：
```c++

class Graphics{

public:
	Graphics();
	~Graphics();

	void drawLine(Point p0, Point p1);
	void drawLine(int x0,int y0, int x1, int y1);

private:
	class Impl;				// 头文件只放声明
	boost::scoped_ptr<Impl> impl;
}
```
  
  ---
  
  在库的实现中把调用转发（forward）给实现 Graphics :: Impl，这部分代码位于. so/. dll 中，随库的升级一起变化。
  
  ---
  
  
```c++
#include<graphics.h>

class Graphics::Impl{

public:
	void drawLine(Point p0, Point p1);
	void drawLine(int x0,int y0, int x1, int y1);

}
  
Graphics::Graphics()
	:impl(new impl)
{
}

Graphics::~Graphics(){
}

void Graphics::drwaLine(Point p0, Point p1){
	impl->drwaLine(p0,p1);
}

void Graphics::drwaLine(int x0,int y0, int x1, int y1){
	impl->drwaLine(x0,y0,x1,y1);
}
  
```
  
  ---
  
  如果要加入新的功能，不必通过继承来扩展，可以原地修改，且很容易保持二进制兼容性。先动头文件：
  
  ---
  
  ```c++
  
class Graphics{
public:
  	Graphics();
	~Graphics();
	
	void drawLine(Point p0, Point p1);
	void drawLine(int x0,int y0, int x1, int y1);
+	void drawLine(double x0, double y0, double x1, double y1);

private:
  	class Impl;				// 头文件只放声明
	boost::scoped_ptr<Impl> impl;
}
  
  ```
  
  ---
  
  然后在实现文件里增加 forward，这么做不会破坏二进制兼容性，因为增加 non-virtual 函数不影响现有的可执行文件。
  
  ---

```c++
#include<graphics.h>

class Graphics::Impl{

public:
	void drawLine(Point p0, Point p1);
	void drawLine(int x0,int y0, int x1, int y1);
+   void drawLine(double x0, double y0, double x1, double y1);

}
  
Graphics::Graphics()
	:impl(new impl)
{
}

Graphics::~Graphics(){
}

void Graphics::drwaLine(Point p0, Point p1){
	impl->drwaLine(p0,p1);
}

void Graphics::drwaLine(int x0,int y0, int x1, int y1){
	impl->drwaLine(x0,y0,x1,y1);
}


+void Graphics::drwaLine(double x0,double y0, double x1, double y1){
+	impl->drwaLine(x0,y0,x1,y1);
+}
+
```

> 采用 pimpl 多了一道 explicit forward 的手续，带来的好处是可扩展性与[[二进制兼容性]]，这通常是划算的。Pimpl 扮演了编译器防火墙的作用。

---

参考链接：https://www.jb51.net/article/122557.htm

