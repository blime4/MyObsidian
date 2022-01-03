Unique_ptr 是在 C++标准中定义的新的智能指针，用来取代曾经的 auto_ptr。根据 C++标准定义（C++11.20.7.1），unique_ptr 不仅能够代理 new 创建的单个对象，也能够代理 new[] 创建的数组对象.

unique_ptr 的基本能力与 [[scoped_ptr]] 相同，同样可以在作用域内管理指针，也不允许拷贝构造和拷贝赋值（但支持[[转移语义]]）。

> 但unique_ptr要比scoped_ptr有更多的功能：unique_ptr可以像原始指针一样进行比较，可以像shared_ptr一样[[定制删除器]]，也可以安全地放入标准容器。因此，如果读者使用的编译器支持C++11标准，那么可以毫不犹豫地使用unique_ptr来代替scoped_ptr。