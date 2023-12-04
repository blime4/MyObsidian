# 002 - Exception

### 一、Exception类实现

-   backtrace，栈回溯，保存各个栈帧的地址
    
-   backtrace_symbols，根据地址，转成相应的函数符号
    
-   `abi::__cxa_demangle`


比较关键的部位就是两个函数backtrace和backtrace_symbols，下面将介绍这两个函数的用法。

在头文件"execinfo.h"中声明了三个函数用于获取当前线程的函数调用堆栈：
```c++
#include <execinfo.h>

 int backtrace(void **buffer, int size);

 char **backtrace_symbols(void *const *buffer, int size);

 void backtrace_symbols_fd(void *const *buffer, int size, int fd);
```

下面将具体介绍用法：
```c++
int backtrace(void **buffer,int size)
```
该函数用与获取当前线程的调用堆栈,获取的信息将会被存放在buffer中,它是一个指针数组。参数 size 用来指定buffer中可以保存多少个void* 元素。函数返回值是实际获取的指针个数,最大不超过size大小在buffer中的指针实际是从堆栈中获取的返回地址,每一个堆栈框架有一个返回地址。

注意某些编译器的优化选项对获取正确的调用堆栈有干扰,另外内联函数没有堆栈框架;删除框架指针也会使无法正确解析堆栈内容。


```c++
char ** backtrace_symbols (void *const *buffer, int size)
```

backtrace_symbols将从backtrace函数获取的信息转化为一个字符串数组. 参数buffer应该是从backtrace函数获取的数组指针,size是该数组中的元素个数(backtrace的返回值)，函数返回值是一个指向字符串数组的指针,它的大小同buffer相同.每个字符串包含了一个相对于buffer中对应元素的可打印信息.它包括函数名，函数的偏移地址,和实际的返回地址

现在,只有使用ELF二进制格式的程序和苦衷才能获取函数名称和偏移地址.在其他系统,只有16进制的返回地址能被获取.另外,你可能需要传递相应的标志给链接器,以能支持函数名功能(比如,在使用GNU ld的系统中,你需要传递(-rdynamic))

backtrace_symbols生成的字符串都是malloc出来的，但是不要最后一个一个的free，因为backtrace_symbols是根据backtrace给出的call stack层数，一次性的malloc出来一块内存来存放结果字符串的，所以，像上面代码一样，只需要在最后，free backtrace_symbols的返回指针就OK了。这一点backtrace的manual中也是特别提到的。

注意:如果不能为字符串获取足够的空间函数的返回值将会为NULL。


```三、__cxa_demangle打印具体函数信息```

上面测试可以总结出：

-   1.  backtrace可以在程序运行的任何地方被调用，返回各个调用函数的返回地址，可以限制最大调用栈返回层数。
        
-   2.  在backtrace拿到函数返回地址之后，backtrace_symbols可以将其转换为编译符号，这些符号是编译期间就确定的
        

但是从测试的结果看，打印的异常栈内容都是给编译器看的，里面都有编译符号，但是根据backtrace_symbols返回的编译符号，`abi::__cxa_demangle`可以找到具体地函数方法：
