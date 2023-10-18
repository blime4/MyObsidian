
[[零拷贝]]

```c++
void *mmap(void *addr, size_t length, int prot, int flags,
                  int fd, off_t offset);
```
ctor
```c++
buffer_ = (char *)mmap(NULL, mem_size_, PROT_READ | PROT_WRITE,
                             MAP_SHARED, fd_, 0);

if (buffer_ == MAP_FAILED) {
	fprintf(stderr, "mmap file failed,errno=%d", errno);
}
```
dtor
```c++
if (fd_ >= 0) {
  close(fd_);
  fd_ = -1;
}
if (buffer_ != MAP_FAILED) {
  munmap(buffer_, mem_size_);
}
```

If  addr  is  NULL, then the kernel chooses the (page-aligned) address at which to create the mapping; this is the most portable method of creating a new mapping.