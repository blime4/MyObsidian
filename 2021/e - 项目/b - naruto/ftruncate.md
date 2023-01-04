

**ftruncate**（改变文件大小）

定义函数 int **ftruncate**(int fd,off_t length);

```c++
       int truncate(const char *path, off_t length);
       int ftruncate(int fd, off_t length);

```

函数说明

**ftruncate**()会将参数fd指定的文件大小改为参数length指定的大小。

参数fd为已打开的[文件描述词](https://baike.baidu.com/item/%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E8%AF%8D)，而且必须是以写入模式打开的文件。

如果原来的文件大小比参数length大，则超过的部分会被删去。

返回值

执行成功则返回0，失败返回-1，错误原因存于[errno](https://baike.baidu.com/item/errno/11040395)。

[错误代码](https://baike.baidu.com/item/%E9%94%99%E8%AF%AF%E4%BB%A3%E7%A0%81)

EBADF 参数fd文件描述词为无效的或该文件已关闭。

EINVAL 参数fd 为一socket 并非文件，或是该文件并非以写入模式打开。