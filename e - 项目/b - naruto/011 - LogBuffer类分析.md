# 011 - LogBuffer类分析

是生产者消费者模型的一个解耦，相对于一个队列
```c++
#ifndef NARUTO_LOG_BUFFER_H_
#define NARUTO_LOG_BUFFER_H_
#include <sys/types.h>
namespace hardcode {
namespace naruto {

class LogBuffer {
public:
  LogBuffer(size_t total = 1024 * 1024 * 10);
  ~LogBuffer();

  void clear();
  void append(const char *data, size_t len);
  const char *data() const;
  size_t length() const;
  size_t available() const;
  // for used by GDB
  const char *debugString();
  void setCookie(void (*cookie)()) { cookie_ = cookie; }

private:
  char *data_;
  const size_t total_;
  size_t available_;
  size_t cur_;
  // Must be outline function for cookies.
  static void cookieStart();
  static void cookieEnd();

  void (*cookie_)();
};

} // namespace naruto

} // namespace hardcode

#endif


```
```c++

LogBuffer::LogBuffer(size_t total) : total_(total), available_(total), cur_(0) {
  data_ = new char[total];
  setCookie(cookieStart);
}

LogBuffer::~LogBuffer() {
  delete[] data_;
  setCookie(cookieEnd);
}

size_t LogBuffer::available() const { return available_; }

void LogBuffer::clear() {
  cur_ = 0;
  available_ = total_;
}

void LogBuffer::append(const char *data, size_t len) {
  memcpy(data_ + cur_, data, len);
  cur_ += len;
  available_ -= len;
}

const char *LogBuffer::data() const { return data_; }

size_t LogBuffer::length() const { return cur_; }

void LogBuffer::cookieStart() {}

void LogBuffer::cookieEnd() {}
```
