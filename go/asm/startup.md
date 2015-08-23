
```
var (
	m0 m
	g0 g
)
```

# 

```
TEXT runtime·rt0_go(SB),NOSPLIT,$0
	....
ok:
	// set the per-goroutine and per-mach "registers"
	get_tls(BX)
	LEAQ	runtime·g0(SB), CX
	MOVQ	CX, g(BX)
	LEAQ	runtime·m0(SB), AX

	// save m->g0 = g0
	MOVQ	CX, m_g0(AX)
	// save m0 to g0->m
	MOVQ	AX, g_m(CX)

	CLD				// convention is D is always left cleared
	CALL	runtime·check(SB)

	MOVL	16(SP), AX		// copy argc
	MOVL	AX, 0(SP)
	MOVQ	24(SP), AX		// copy argv
	MOVQ	AX, 8(SP)
	CALL	runtime·args(SB)
	CALL	runtime·osinit(SB)
	CALL	runtime·schedinit(SB)

	// create a new goroutine to start program
	MOVQ	$runtime·mainPC(SB), AX		// entry
	PUSHQ	AX
	PUSHQ	$0			// arg size
	CALL	runtime·newproc(SB)
	POPQ	AX
	POPQ	AX

	// start this M
	CALL	runtime·mstart(SB)

```

* rt0_go
	* 设置m0, g0
	* 设置命令行参数
	* osinit // 只干了一件事 ncpu = getproccount()
	* schedinit 
		* stackinit()
		* mallocinit()
		* mcommoninit(_g_.m)
		* goargs()
		* goenvs()
		* parsedebugvars()
		* gcinit()
		* procresize(int32(procs))
	* newproc // make & queue new G， 用来跑
	* mstart() // schedule, 把刚准备好的G跑起来
	* runtime.main()
		* newm(sysmon, nil) // sysmon听起来很霸气，但是只是一个普通的G？一个线程？什么时候开炮？
		* runtime_init
		* gcenable
		* main_init
		* main_main // 这两个是user写的吧？ sysmon 什么时候跑？
		* exit(0)

这里创建G的方法和程序中"go foo()"是类似的：

```
// 。。。 The compiler turns a go statement into a call to this.。。。

func newproc(siz int32, fn *funcval) {
	argp := add(unsafe.Pointer(&fn), ptrSize)
	pc := getcallerpc(unsafe.Pointer(&siz))
	systemstack(func() {
		newproc1(fn, (*uint8)(argp), siz, 0, pc)
	})
}
```

