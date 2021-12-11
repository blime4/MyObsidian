



用普通 mutex 替换读写锁。解决办法都基于同一个思路，那就是用 shared_ptr 来管理共享数据。原理如下：
+ [[shared_ptr]] 是引用计数型智能指针，如果当前只有一个观察者，那么引用计数的值为 1
+ 对于write端，如果发现引用计数为1，这时可以安全地修改共享对象，不必担心有人正在读它。
+ 对于read端，在读之前把引用计数加1，读完之后减1，这样保证在读的期间其引用计数大于1，可以阻止并发写。
+ ==比较难的是，对于 write 端，如果发现引用计数大于 1，该如何处理？sleep () 一小段时间肯定是错的。==

简单例子:
```c++
typedef std::vector<Foo> FooList;
typedef boost::shared_ptr<FooList> FooListPtr;
MutexLock mutex;
FooListPtr g_foos;


void traverse(){
	FooListPtr foos;
	{
		MutexLockGuard lock(mutex);
		foos = g_foos;
		assert(!g_foos.unique());
	}
	
	for(auto it : foos){
		it->doit();
	}
}


void post(const Foo& f){
	MutexLockGuard lock(mutex);
	if(!g_foos.unique()){
		g_foos.reset(new FooList(*g_foos));
		printf("copy the whole list\n");
	}
	assert(g_foos.unique());
	g_foos->push_back(f);
}
```


																																																																																																																																																																																																																																																																																																																																							```````````````````````aff```````````````																																																																																																																																																																																																																																																																																																																																																																																																																																																																																									