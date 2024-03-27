

配置类，暴露给用户，设置日志的配置
+ 每一个buffer的大小，这里设置为4000
+ 双缓冲
+ 日志级别
+ 日志文件的配置
	+ 日志文件路径
	+ 文件写入大小
	+ 文件写入间隔
	+ 文件对端类型
```c++
#ifndef NARUTO_LOG_CONFIG_H_
#define NARUTO_LOG_CONFIG_H_

#include <stdint.h>
#include <string>

#include "file_writer_type.h"
#include "log_level.h"

namespace hardcode{
namespace naruto{
struct LogConfig{
	uint32_t log_buffer_size = 4000;
	uint32_t log_buffer_nums = 2;
	LogLevel log_level = LogLevel::INFO;
	struct FileOption{
		std::string file_path;
		//日志滚动大小和时间间隔，仅限asyncfileappender
	    uint32_t log_flush_file_size;
	    uint32_t log_flush_interval;
	    FileWriterType file_writer_type;
	} file_option;
};
}
}
#endif
```

log_config:
包括日志文件的大小，设置缓存区数量，[[双缓冲机制]]
日志级别，写入地址，滚动大小，时间间隔，文件写入方式

[[004 - LogLevel类分析]]
[[005 - FileWriterType类分析]]

