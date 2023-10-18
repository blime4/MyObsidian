# 009 - 加载程序（bootloader）

加载程序是能认识文件系统的格式的

1. 从文件系统中读取启动配置信息
	1. 不同操作系统不一样
2. 可选的操作系统内核列表加载参数
3. 依据配置加载指定内核并跳转到内核执行



bootloader 做的事情
+ 使能[[保护模式]]（ protection mode）&[[段机制 ]]（segment-level protection） 从 1M 的地址空间变成 4G
+ 从硬盘上读取 kernel in ELF 格式的 kernel (跟在 MBR 后面的扇区）并放到内存中固定位置
+ 跳转到  OS 的入口点（ entry point）执行，这时控制权到了 Os 中
