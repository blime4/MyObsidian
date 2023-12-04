



variant是更加安全的union，复合[[RAII]]思想

```c++

std::variant<int,float> v = 3;

std::cout<<std::get<int>(v) <<endl;

v = 3.14f;

std::cout<<std::get<float>(v)<<endl;
std::cout<<std::get<int>(v) <<endl;   // 运行时报错

```

```c++

void print(std::variant<int,float> const & v){
	if(std::holds_alternative<int>(v)){
		std::cout<<std::get<int>(v)<<std::endl;
	}else{
		std::cout<<std::get<float>(v)<<std::endl;
	}

	/*
	or

	if(v.index() == 0){
		std::cout<<std::get<0>(v)<<std::endl;
	}else if (v.index() == 1){
		std::cout<<std::get<1>(v)<<std::endl;
	}
	
	*/
}
```


用variant不用visit，就像看四大名著不看红楼梦


std::[[visit]] , std::variant 这种模式称为[[静态多态]]，和[[虚函数]]，[[抽象类]]这种[[动态多态]]相对

```c++

void print(std::variant<int, float> const & v){
	std::visit([&](auto const &t){
		std::cout<<t<<std::endl;
	}, v);
}

```