# 003 - using over typedef

```c++
using A = int;
using B = int(*)(int, int);
using C = auto(*)(int, int) -> int;
```

```c++
auto(*) means "pointer-to-function" now
```

[[004 - default member initialization]]