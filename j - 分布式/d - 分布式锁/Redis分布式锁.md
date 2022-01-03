

```
set resourceName value ex 5 nx
```
ex 超时
nx 独占

业务是，删除的时候，要判断这个锁是不是自己持有的，不然删除了别人持有的锁，会出安全问题。

