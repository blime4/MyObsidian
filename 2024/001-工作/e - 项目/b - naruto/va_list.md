- Date: 2021-12-23
- Time:  23:10



VA_LIST 是在C语言中解决变参问题的一组宏，变参问题是指参数的个数不定，可以是传入一个参数也可以是多个;可变参数中的每个参数的类型可以不同,也可以相同;可变参数的每个参数并没有实际的名称与之相对应，用起来是很灵活。


va_list表示可变参数列表类型，实际上就是一个char指针fmt。

```c++
#include <stdarg.h> 
 
int AveInt(int,...);
 
 void main()
{
   printf("%d/t",AveInt(2,2,3));
   printf("%d/t",AveInt(4,2,4,6,8));
 
   return;
}
 
int AveInt(int v,...)
{
   int ReturnValue=0;
   int i=v;
 
   va_list ap ;
   va_start(ap,v);
 
   while(i>0)
   {
       ReturnValue+=va_arg(ap,int) ;
       i--;
   ｝
   va_end(ap); 
   return ReturnValue/=v;
}
```

VA_LIST的用法：   
（1）首先在函数里定义一具VA_LIST型的变量，这个变量是指向参数的指针；   
（2）然后用VA_START宏初始化变量刚定义的VA_LIST变量；   
（3）然后用VA_ARG返回可变的参数，VA_ARG的第二个参数是你要返回的参数的类型（如果函数有多个可变参数的，依次调用VA_ARG获取各个参数）；   
（4）最后用VA_END宏结束可变参数的获取。

```c++

// 可变参数是由宏实现的，但是由于硬件平台的不同，编译器的不同，宏的定义也不相同，下面是VC6.0中x86平台的定义 ：
 
typedef char * va_list;     // TC中定义为void*
#define _INTSIZEOF(n)    ((sizeof(n)+sizeof(int)-1)&~(sizeof(int) - 1) ) //为了满足需要内存对齐的系统
#define va_start(ap,v)    ( ap = (va_list)&v + _INTSIZEOF(v) )     //ap指向第一个变参的位置，即将第一个变参的地址赋予ap
#define va_arg(ap,t)       ( *(t *)((ap += _INTSIZEOF(t)) - _INTSIZEOF(t)) )   /*获取变参的具体内容，t为变参的类型，如有多个参数，则通过移动ap的指针来获得变参的地址，从而获得内容*/
#define va_end(ap) ( ap = (va_list)0 )   //清空va_list，即结束变参的获取
```


[参考自：](https://blog.csdn.net/dengzhilong_cpp/article/details/54944676)