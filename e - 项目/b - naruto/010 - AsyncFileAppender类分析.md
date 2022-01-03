# 010 - AsyncFileAppender类分析
[[007 - log_appender_interface类分析]]


```c++

#ifndef NARUTO_ASYNC_FILE_APPENDER_H_
#define NARUTO_ASYNC_FILE_APPENDER_H_
#include <vector>

#include "condition.h"
#include "count_down_latch.h"
#include "log_appender_interface.h"
#include "log_buffer.h"
#include "mutex.h"
#include "thread.h"
namespace hardcode {
namespace naruto {
class AsyncFileAppender : public LogAppenderInterface {
public:
  AsyncFileAppender(const std::string &basename);
  ~AsyncFileAppender();
  void append(const char *msg, size_t len);
  void start();
  void stop();

private:

  void threadFunc();

  bool started_;
  bool running_;
  time_t persist_period_;
  std::string basename_;
  Mutex mutex_;
  Condition cond_;
  CountDownLatch countdown_latch_;
  Thread persit_thread_;
  std::unique_ptr<LogBuffer> cur_buffer_;
  std::vector<std::unique_ptr<LogBuffer>> buffers_;
};
} // namespace naruto

} // namespace hardcode

#endif

```

[[011 - LogBuffer类分析]]
[[012 - LogFile类分析]]

## threadFunc函数
1. 初始化 buffer等多线程组件
	1.1 创建一个当前 buffer的指针
	1.2 创建一个持久化的 buffer vector
	1.3 创建一个 logfile对象
2. while running循环
	2.1 每秒中被唤醒一次检查当前 buffer是否为空，为空就继续休眠
	2.2 不为空的话移动当前 buffer加入 buffers vector中
	2.3 将 buffers交换给创建的持久化 buffer vector用于消费
	2.4 重置当前的 buffer
	2.5 如果持久化 buffers太长就截断他
	2.6 遍历持久化的 buffers vector以 buffer为单位写入 logfile中
	2.7 重置当前的 buffer与持久化数组
	2.8 对日志文件fush
	2.9 如果线程被关闭，等待所有 buffer写入后彻底停止循环
3. 再次fush整个 logfile并结束函数

```c++
void AsyncFileAppender::threadFunc(){
	/*  初始化多线程组件
	1.1 创建一个当前 buffer的指针
	1.2 创建一个持久化的 buffer vector
	1.3 创建一个 logfile对象
	*/
	std::unique_ptr<LogBuffer> buffer (new LogBuffer(kLogConfig.log_buffer_size));

	std::vector<std::unique_ptr<LogBuffer>> persist_buffers;
	persist_buffers.reserve(kLogConfig.log_buffer_nums);

	logFile log_file(basename_,kLogConfig.file_option.log_flush_file_size,
					 kLogConfig.file_option.log_flush_interval, 1024,
					 kLogConfig.file_option.file_writer_type);

	countdown_latch_.countDown();

	// 2. while running循环
	while(running_){
		{
			MutexGuard gurd(mutex_);
			// wake up every persist_per_seconds_ or on Buffer is full
			// 2.1 每秒中被唤醒一次检查当前 buffer是否为空，为空就继续休眠
			// 2.2 不为空的话移动当前 buffer加入 buffers vector中
			if (buffers_.empty()) {
		        cond_.waitForSeconds(persist_period_);
			}
			if (buffers_.empty() && cur_buffer_->length() == 0) {
				continue;
			}
			buffers_.push_back(std::move(cur_buffer_)); // todo use emplace_back instead of push_bach
	
			// reset  cur_buffer_ and buffers_
			// 2.3 将 buffers交换给创建的持久化 buffer vector用于消费
			persist_buffers.swap(buffers_);
			// 2.4 重置当前的 buffer
			cur_buffer_ = std::move(buffer);
			cur_buffer_->clear();
		}

		// if log is too large, drop it
		if (persist_buffers.size() > kLogConfig.log_buffer_size) {
			std::cerr << "log is too large, drop some" << std::endl;
			persist_buffers.erase(persist_buffers.begin() + 1, persist_buffers.end());
		}
	
	    // persist log
	    for (const auto &buffer : persist_buffers) {
			log_file.append(buffer->data(), buffer->length());
	    }
	
		// reset buffer and persist_buffers
	    buffer = std::move(persist_buffers[0]);
	    buffer->clear();
	    persist_buffers.clear();

		log_file.flush();

		// 2.9如果线程被关闭，等待所有 buffer写入后彻底停止循环
		if (!started_) {
			MutexGuard guard(mutex_);
			if (cur_buffer_->length() == 0) {
				running_ = false;
			}
		}
	}

	// 3. 再次fush整个 logfile并结束函数
	log_file.flush();

}


```



认为可能出问题的：
```c++
    void append(const char* msg, size_t len){
        MutexGuard guard(mutex_);
        if(cur_buffer_ -> available() > len){
            cur_buffer_ -> append(msg,len);
        }else{
            buffers_.push_back(std::move(cur_buffer_));
            cur_buffer_.reset(new LogBuffer(kLogConfig.log_buffer_size));
            cur_buffer_->append(msg, len);
            cond_.notifyOne();
        }
    }
```

如果len>total 一直卡在这里
[[双缓冲机制]]



其他函数：
```c++
/*
log日志处理函数：threadFunc()
是否开始，是否运行
日志文件名
文件写入间隔 ： persist_period_
文件写入线程： persit_thread_ ，回调函数：threadFunc
文件写入buffer: cur_buffer_
*/

AsyncFileAppender(const std::string &basename)
            : started_(false), running_(false),
              persist_period_(kLogConfig.file_option.log_flush_interval),
              basename_(basename), cond_(mutex_), countdown_latch_(1),
              persist_thread_(std::bind(&AsyncFileAppender::threadFunc, this), "AsyncLogging"),
              cur_buffer_(new LogBuffer(kLogConfig.log_buffer_size))
        {
            mkdir(basename_.c_str(), 0755);
            start();
        }


~AsyncFileAppender()
        {
            if (started_)
            {
                stop();
            }
        }
```

```c++
void start()
{
	started_ = true;
	running_ = true;
	persist_thread_.start();
	countdown_latch_.wait();
}
void stop()
{
	started_ = false;
	cond_.notifyOne();
	persist_thread_.join();
}

```