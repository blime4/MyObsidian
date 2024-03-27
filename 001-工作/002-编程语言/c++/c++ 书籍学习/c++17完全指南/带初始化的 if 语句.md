# 带初始化的 if 语句

![[Pasted image 20220114033404.png]]


例子：
```c++

if(std::lock_guard lg{collMutex}; !coll.empty()){
	std::cout<< coll.front() << std::endl;
}

```

这里是对`std::lock_guard<std::mutex> lg{collMutex};`做了[[009 - 类模板参数推导]]

其他例子：
```c++

if(auto x = qqq1(), y = qqq2(); x!=y){
	std::cout << "return values " << x << " and " << y << "differ\n";
}

```


另一个例子是向 map 或者 unordered map 插入元素。你可以像下面这样检查是否成功：
```c++
std::map<std::string, int> coll;
...
if (auto [pos, ok] = coll.insert({"new", 42}); !ok) {
	// 如 果 插 入 失 败， 用pos处 理 错 误
	const auto& [key, val] = *pos;
	std::cout << "already there: " << key << '\n';
}
```

