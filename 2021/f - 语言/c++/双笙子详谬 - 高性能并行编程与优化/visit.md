std::[[visit]] , std::variant 这种模式称为[[静态多态]]，和[[虚函数]]，[[抽象类]]这种[[动态多态]]相对

```c++

void print(std::variant<int, float> const & v){
	std::visit([&](auto const &t){
		std::cout<<t<<std::endl;
	}, v);
}

```
std:visit：还支持多个参数其实还可以有多个 variant作为参数。
相应地 lambda的参数数量要与之匹配。
std:visit会自动罗列出所有的排列组合！
所以如果 variant有n个类型，那 lambda就要被编译n^2次，编译可能会变慢。
但是标准库能保证运行时是O（1）的（他们用函数指针实现分支，不是暴力 if-else）。
```c++

auto add(std::variant<int,float> const & v1,
		 std::variant<int,float> const & v2){
	std::variant<int, float> ret;
	std::visit([&](auto const &t1, auto const &t2){
		ret = t1+t2;
	}, v1 , v2);
	return ret;
}
```


优化版本，应用RAII思想，ret初始化时赋值：
```c++
auto add(std::variant<int,float> const & v1,
		 std::variant<int,float> const & v2){
	std::visit([&](auto const &t1, auto const &t2)
		-> std::variant<int, float>{
		return t1+t2;
	}, v1 , v2);
	return ret;
}

```