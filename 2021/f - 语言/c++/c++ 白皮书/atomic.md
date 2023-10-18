```c++
atomic<int> x;
void increment()
{
    x++;  // 不是 x = x + 1
}

```