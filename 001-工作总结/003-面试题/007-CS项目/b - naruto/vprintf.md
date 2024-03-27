The functions vprintf(), vfprintf(), vdprintf(), vsprintf(), vsnprintf() are equivalent to the functions printf(),fprintf(), dprintf(), sprintf(), snprintf(), respectively, 

except that they are called with a va_list instead of a variable number of arguments.  

These functions do not call the va_end  macro.   Because  they  invoke  the  va_arg
macro, the value of ap is undefined after the call.


## 描述
C 库函数 `int vprintf(const char *format, va_list arg)`使用参数列表发送格式化输出到标准输出 stdout。

## 声明
下面是 vprintf() 函数的声明。

`int vprintf(const char *format, va_list arg)`

---


**format**  这是字符串，包含了要被写入到标准输出 stdout 的文本。它可以包含嵌入的 format 标签，format 标签可被随后的附加参数中指定的值替换，并按需求进行格式化。format 标签属性是 `%[flags][width][.precision][length]specifier`，具体讲解如下：

**arg** 一个表示可变参数列表的对象。这应被 `<stdarg> `中定义的 va_start 宏初始化。

---

## 返回值

如果成功，则返回写入的字符总数，否则返回一个负数。