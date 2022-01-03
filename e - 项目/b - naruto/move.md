# std::move

```c++
while(!functors_.empty()){
	Task task = std::move(functors_.front());
	functors_.pop_front();
	task();
}

```

