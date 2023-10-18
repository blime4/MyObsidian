[Git - 子模块 (git-scm.com)](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97) | 并未看完

```
# 1.
git submodule add xxx.git

# 2.
git clone xxx.git
git submodule init

# or
git clone --recurse-submodules xxx.git

# 3.
git submodule update --init --recursive
```