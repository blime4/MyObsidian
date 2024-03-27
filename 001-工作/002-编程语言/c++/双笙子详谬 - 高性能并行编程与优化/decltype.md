



注意：

```c++

int a;

SHOW(decltype(a));      // int    
SHOW(decltype((a)));    // int &    会将（a）看成一个表达式

```


decltype(auto)万能推导



一个例子：
```c++

template<class T1,class T2>
auto add(std::vector<T1> const & a, std::vector<T2> const & b){
	using T0 = decltype(T1{} + T2{});
	std::vector<T0> ret;
	for(size_t i；i<std::min(a.size(),b.size();++i)）{
		ret.push_back(a[i]+b[i]);
	}
	return ret;
}

```

