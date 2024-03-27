### 声明Lambda表达式

Lambda表达式完整的声明格式如下：

```c
[capture list] (params list) mutable exception-> return_type { function body }
```

各项具体含义如下：

* capture list：捕获外部变量列表
* params list：形参列表
* mutable指示符：用来说用是否可以修改捕获的变量
* exception：异常设定
* return type：返回类型
* function body：函数体

此外，我们还可以省略其中的某些成分来声明“不完整”的Lambda表达式，常见的有以下几种：

| 序号 | 格式                                                        |
| ---- | ----------------------------------------------------------- |
| 1    | [capture list] (params list) -> return_type {function body} |
| 2    | [capture list] (params list) {function body}                |
| 3    | [capture list] {function body}                              |

```c++
	sort(lbvec.begin(), lbvec.end(), [](int a, int b) -> bool { return a < b; });   // Lambda表达式
```
#### 值捕获
```c++
	auto f = [a] { cout << a << endl; };
	f(); // 输出：123

	//或通过“函数体”后面的‘()’传入参数
	[](int a){cout << a << endl; }(1233435);
```
#### 引用捕获
```c++
	int a = 123;
	auto f = [&a] { cout << a << endl; };
	a = 321;
	f(); // 输出：321
```

[[隐式捕获]]有两种方式，分别是[=]和[&]。[=]表示以值捕获的方式捕获外部变量，[&]表示以引用捕获的方式捕获外部变量。

```c++
	auto f = [=] { cout << a << endl; };    // 值捕获
	auto f = [&] { cout << a << endl; };    // 引用捕获
```


### 三、修改捕获变量

前面提到过，在 Lambda 表达式中，如果以传值方式捕获外部变量，则函数体中不能修改该外部变量，否则会引发编译错误。那么有没有办法可以修改值捕获的外部变量呢？这是就需要使用 mutable 关键字，该关键字用以说明表达式体内的代码可以修改值捕获的变量：

```c++
int main()
{
    int a = 123;
    auto f = [a]()mutable { cout << ++a; }; // 不会报错
    cout << a << endl; // 输出：123
    f(); // 输出：124
}
```
如果不想使用mutable关键字，可以采取引用方式捕获外部变量，而非传值。

### 四、Lambda 表达式的参数
Lambda表达式的参数和普通函数的参数类似，那么这里为什么还要拿出来说一下呢？原因是在Lambda表达式中传递参数还有一些限制，主要有以下几点：

-   参数列表中不能有默认参数
-   不支持可变参数
-   所有参数必须有参数名