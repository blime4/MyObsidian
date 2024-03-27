# 012 - LogFile类分析


logFile文件是对对端的抽象，有MMapFileWriter和AppendFileWriter两种对端的实现

负责日志文件的写入

## FileWriter
```c++
class FileWriter {
public:
  FileWriter() = default;
  virtual ~FileWriter() = default;
  virtual void append(const char *msg, int32_t len) = 0;
  virtual void flush() = 0;
  virtual uint32_t writtenBytes() const = 0;
};

```
## LogFile
```c++
class LogFile {
public:
  LogFile(const std::string &basename, int32_t roll_size,
          int32_t flush_interval, int32_t check_interval,
          FileWriterType file_writer_type);
  ~LogFile();

  void append(const char *logline, int32_t len);
  void flush();
  bool rollFile();

private:
  std::string basename_;
  uint32_t roll_size_;
  uint32_t flush_interval_;
  //多久进行检查一次，因为可能日志没满，但是跨天了
  uint32_t check_freq_count_;
  uint32_t count_;
  time_t start_of_period_;
  time_t last_roll_;
  time_t last_flush_;
  std::shared_ptr<FileWriter> file_;
  FileWriterType file_writer_type_;
  constexpr static int kRollPerSeconds = 60 * 60 * 24;
};

```
[[ftruncate]]
[[mmap]]
## MMapFileWriter
```c++
class MMapFileWriter : public FileWriter {
public:
  MMapFileWriter(const std::string &basename, uint32_t mem_size) 
	{
    mem_size_ = mem_size;
    writed_ = 0;
    if (fd_ >= 0) {
      close(fd_);
    }
    fd_ = open(basename.c_str(), O_RDWR | O_CREAT,
               S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH);
    if (fd_ < 0) {
      fprintf(stderr, "open new file failed,errno=%d", errno);
    } else {
      int n = ftruncate(fd_, mem_size);
      (void)n;
      buffer_ = (char *)mmap(NULL, mem_size_, PROT_READ | PROT_WRITE,
                             MAP_SHARED, fd_, 0);
      if (buffer_ == MAP_FAILED) {
        fprintf(stderr, "mmap file failed,errno=%d", errno);
      }
    }
  }
  ~MMapFileWriter() {
    if (fd_ >= 0) {
      close(fd_);
      fd_ = -1;
    }
    if (buffer_ != MAP_FAILED) {
      munmap(buffer_, mem_size_);
    }
  }
  void append(const char *msg, int32_t len) {
    if (len > mem_size_ - writed_) {
      fprintf(stderr, "mmap memory overflow ,errno=%d", errno);
      return;
    }
	// memcpy is thread safety
    memcpy(buffer_ + writed_, msg, len);
    writed_ += len;
  }
  void flush() {
    if (buffer_ != MAP_FAILED) {
      msync(buffer_, mem_size_, MS_ASYNC);
    }
  }
  uint32_t writtenBytes() const { return writed_; }

private:
  int fd_;
  char *buffer_;
  int32_t mem_size_;
  int32_t writed_;
};
```
## AppendFileWriter
```c++
class AppendFileWriter : public FileWriter {
public:
  AppendFileWriter(const std::string &filename)
      : fp_(::fopen(filename.c_str(), "ae")) {
    ::setbuffer(fp_, buffer_, sizeof buffer_);
  }
  ~AppendFileWriter() {
    if (fp_) {
      ::fclose(fp_);
    }
  }
  void append(const char *msg, int32_t len) {
    size_t n = ::fwrite_unlocked(msg, 1, len, fp_);
    size_t remain = len - n;
    while (remain > 0) {
      size_t x = ::fwrite_unlocked(msg + n, 1, remain, fp_);
      if (x == 0) {
        int err = ferror(fp_);
        if (err) {
          fprintf(stderr, "AppendFile::append() failed %s\n", strerror(err));
        }
        break;
      }
      n += x;
      remain = len - n; // remain -= x
    }

    writen_ += len;
  }
  void flush() { ::fflush(fp_); }
  uint32_t writtenBytes() const { return writen_; }

private:
  FILE *fp_;
  char buffer_[64 * 1024];
  uint32_t writen_ = 0;
};
```

## LogFile cpp
```c++
std::string getLogLevelStr(LogLevel log_level){
	switch (log_level){
#define XX(name)
	case LogLevel::name;
		return #name;
		break;
	
		XX(DEBUG);
		XX(INFO);
		XX(WARN);
		XX(FATAL);
#undef XX
	default:
		return "UNKNOW";
	}
	return "UNKNOW";
}

```

```c++
void LogFile::append(const char *msg, int32_t len) {
  file_->append(msg, len);
  if (file_->writtenBytes() > roll_size_) {
    rollFile();
  } else {
    ++count_;
    //隔多久需要检查一下
    if (count_ >= check_freq_count_) {
      count_ = 0;
      time_t now = ::time(NULL);
      time_t this_period = now / kRollPerSeconds * kRollPerSeconds;
      if (this_period != start_of_period_) {
        rollFile();
      } else if (now - last_flush_ > flush_interval_) {
        last_flush_ = now;
        file_->flush();
      }
    }
  }
}
```

```c++
void LogFile::flush() { file_->flush(); }

bool LogFile::rollFile() {
  time_t now = 0;
  std::string filename = getLogFileName(basename_, &now);
  time_t start = now / kRollPerSeconds * kRollPerSeconds;

  if (now > last_roll_) {
    last_roll_ = now;
    last_flush_ = now;
    start_of_period_ = start;
    if (file_writer_type_ == FileWriterType::MMAPFILE) {
      file_.reset(new MMapFileWriter(filename, roll_size_));
    } else {
      file_.reset(new AppendFileWriter(filename));
    }
  }
  return true;
}

```