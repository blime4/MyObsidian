# 带初始化的 switch 语句

通过使用带初始化的 switch 语句，我们可以在控制流之前初始化一个对象/实体。例如，我们可以先声明

一个文件系统路径，然后再根据它的类别进行处理：

```c++
namespace fs = std::filesystem;
...
switch (fs::path p{name}; status(p).type()) {
	case fs::file_type::not_found:
		std::cout << p << " not found\n";
		break;
	case fs::file_type::directory:
		std::cout << p << ":\n";
		for (const auto& e : std::filesystem::directory_iterator{p}) {
			std::cout << "- " << e.path() << '\n';
		}
		break;
	default:
		std::cout << p << " exists\n";
		break;
}

```

这里，初始化的路径 p 可以在整个 switch 语句中使用。