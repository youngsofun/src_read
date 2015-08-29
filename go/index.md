
# go1.5 源代码分析

动机两方面：

1. 优化go程序免不了要了解一些实现细节，能看到的资源太零散。
2. go代码并不多，go.15 基本go实现，好读。但是和任何的应用代码相比，其各个方面细节都更加的分散，需要综合。一次读明白，记录清楚，一劳永逸。

个人水平有些限，有些方面实际经验少，理解不够深入，但应该不影响整体：

* 系统和汇编层面理解有限，这里只关注 linux_amd64。
* 编译器相关, 代码 + 注释 + 反汇编，尽力而为。



内容 完成度：

1. 和跨平台，asm相关的必要背景知识，工具
2. [系统调用](g/syscall.md)  90%
2. [goroutine](g/goroutine.md)  80%
3. [调度](g/schedule.md) 
4. [stack]
4. [启动过程](g/startup.md)
4. malloc 
5. gc
5. 结构
	1. chan
	2. mutex
	3. timer
	4. map
	5. net
	6. refect
7. cgo
8. 工具 (prof 等)
8. 有用的点