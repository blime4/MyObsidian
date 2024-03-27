- Date: 2021-12-24
- Time:  12:19




emplace系列函数的用法：
+ emplace函数需要对应的参数对象有对应的构造函数，不然编译报错
+ emplace函数在容器中直接构造元素。传递给 emplace函数的参数必须与元素类型的构造函数相匹配。


# emplace_back的优点

emplace_back能就地通过参数构造对象，不需要拷贝或者构造内存，相比 push_back能更好的避免内存的拷贝和移动，提升容器插入元素的性能。
大多数情况都应该使用 emplace系列函数：
emplace； emplace_back； emplace_hit emplace_fornt emplace_after




当调用 push_back或 insert，成员函数时，是把元素类型的对象传递给它们，这些对象被拷贝到容器中。而当我们调用一个 emplace系列函数时，则是将相应参数传递给元素类型的构造函数。
这样 emplace_back能就地通过参数构造对象，不需要拷贝操作，相比push_back能更好的避免内存的拷贝和移动，提升容器插入元素的性能

例子：
```c++

emplace_back ---> 只调用一次 ctor
push_back    ---> 一次ctor , 一次move

```



[参考自：](https://www.bilibili.com/video/BV1q7411x7RK?from=search&seid=6072566781298030133&spm_id_from=333.337.0.0)