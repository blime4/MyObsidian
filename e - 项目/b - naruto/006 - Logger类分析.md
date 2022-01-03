# 006 - Logger类分析
```c++
#ifndef KANGAROO_LOG_H_
#define KANGAROO_LOG_H_
#include <map>
#include <string>
#include <functional>
#include <list>

#include "log_appender_interface.h"
#include "log_config.h"
#include "mutex.h"
#include "singleton.h"

//中间的一大堆

#define log_fatal(fmt, args...) hardcode::naruto::Logger::fatal(fmt, ##args)
#define log_error(fmt, args...) hardcode::naruto::Logger::error(fmt, ##args)
#define log_warn(fmt, args...) hardcode::naruto::Logger::warn(fmt, ##args)
#define log_debug(fmt, args...) hardcode::naruto::Logger::debug(fmt, ##args)
#define log_info(fmt, args...) hardcode::naruto::Logger::info(fmt, ##args)

#endif

```

[[007 - log_appender_interface类分析]]
[[008 - mutex类分析]]
[[009 - singleton类分析]]

```c++
// 中间那大段
namespace hardcode{
namespace naruto{
static LogConfig kLogConfig;
class Logger{
public:
	static Logger *getLogger() { return Singleton<Logger>::getInstance();}
	static void setGlobalConfig(const LogConfig &log_config){
		kLogConfig = log_config;
	}

	template<class F, class... Args>
	void registerHandle(F&& f, Args&&... args){
		using RetType = decltype(f(args...));
		auto task = std::make_shared<RetType()>(
					std::bind(std::forward<F>(f),std::forward<Args>(args)...)	
					);
		functors_.emplace([task](){
			(*task)();
		});
	}

	void info(const char *format, ...);
	void debug(const chat *format, ...);
	void warn(const char *format, ...);
	void error(const char *format, ...);
	void fatal(const char *format, ...);

	// 需要加锁的
	void addAppender(const std::string &appender_name,
					LogAppenderInterface::Ptr appender);
	void delAppender(const std::string &appender_name);
	void clearAppender();

private:
	void writeLog(LogLevel log_level, const char *file_name,
				  const char *function_name, int32_t line_num, const char *fmt,
				  va_list ap);


private:
	using Task = std::function<void()>;
	Mutex mutex_;
	std::map<std::string, LogAppenderInterface::Ptr> appenders_;
	std::list<Task> functors_;
};
}
}

```

[[make_shared]]
[[va_list]]
[[static]]
[[右值引用]]
[[forward]]
[[emplace运算]]



感觉比较厉害的一段代码：
```c++
	template<class F, class... Args>
	void registerHandle(F&& f, Args&&... args){
		using RetType = decltype(f(args...));
		auto task = std::make_shared<RetType()>(
					std::bind(std::forward<F>(f),std::forward<Args>(args)...)	
					);
		functors_.emplace([task](){
			(*task)();
		});
	}

```


```c++
void Logger::info(const char *format, ...){
	if(!format){
		return;
	}
	va_list ap;
	va_start(ap, format);
	writeLog(LogLevel::INFO, __FILE__ , __FUNCTION__, __LINE__ , format, ap);
	va_end(ap);
};

```

[[vprintf]]
[[strrchr]]

```c++
void Logger::writeLog(LogLevel log_level, const char *file_name,
                      const char *function_name, int32_t line_num,
                      const char *fmt, va_list ap) {
  if (log_level < kLogConfig.log_level) {
    return;
  }
  std::string str_result;
  if (nullptr != fmt) {
    size_t length = vprintf(fmt, ap) + 1; //获取格式化字符串长度
    std::vector<char> fmt_bufs(length,
                               '\0'); //创建用于存储格式化字符串的字符数组
    int writen_n = vsnprintf(&fmt_bufs[0], fmt_bufs.size(), fmt, ap);
    if (writen_n > 0) {
      str_result = &fmt_bufs[0];
    }
  }
  if (str_result.empty()) {
    return;
  }
  const auto &getSourceFileName = [](const char *file_name) {
    return strrchr(file_name, '/') ? strrchr(file_name, '/') + 1 : file_name;
  };

  std::string prefix;
  prefix.append(Timestamp::nowStrTime() + "-");
  prefix.append(getLogLevelStr(log_level) + "-");
  prefix.append(getSourceFileName(file_name));
  prefix.append("-");
  prefix.append(function_name);
  prefix.append("-");
  prefix.append(std::to_string(line_num) + "-");
  prefix.append(str_result);
  while (!functors_.empty()) {
     Task task = std::move(functors_.front());
     functors_.pop_front();
     task();
  }
  MutexGuard guard(mutex_);
  for (const auto &appender : appenders_) {
    appender.second->append(prefix.data(), prefix.size());
  }
}
```

