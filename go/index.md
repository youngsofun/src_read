
# go1.5 源代码分析

go1.5的可读性有所增强，不细扣一些和cpu或编译相关的内容，复杂性并不高，大致读懂并掌握全貌是能做到的，主要是把各个细节完整串起来。性价比较高，避免重复学习。

预计内容：

1. 必要的asm
2. [goroutine](g/goroutine.md)
3. [调度](g/schedule.md) 
4. [启动过程](g/startup.md)
3. stack
4. malloc
5. gc
5. 结构
	1. chan
	2. mutex
	3. timer
	4. map
	5. net
	6. refect
7. 工具 (prof 等)
8. 有用的点