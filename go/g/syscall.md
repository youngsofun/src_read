

# syscall
syscall/asm_linux_amd64.s

```
func Syscall(trap int64, a1, a2, a3 int64) (r1, r2, err int64)
	CALL	runtime·entersyscall(SB)
	...
	SYSCALL
	...
	CALL	runtime·exitsyscall(SB)

func Syscall6(trap, a1, a2, a3, a4, a5, a6 uintptr) (r1, r2, err uintptr)

func RawSyscall(trap, a1, a2, a3 uintptr) (r1, r2, err uintptr)

```

### 例：read

```
// os/file.go
func (f *File) Read(b []byte) (n int, err error)

// os/file_unix.go
func (f *File) read(b []byte) (n int, err error)
	return fixCount(syscall.Read(f.fd, b))

// syscall/syscall_unix.go
func Read(fd int, p []byte) (n int, err error) 
	return read(fd, p)

// syscall/zsyscall_darwin_amd64.go
func read(fd int, p []byte) (n int, err error)
	r0, _, e1 := Syscall(SYS_READ, uintptr(fd), uintptr(_p0), uintptr(len(p)))

// syscall/asm_linux_amd64.s

// func Syscall(trap int64, a1, a2, a3 int64) (r1, r2, err int64);
// Trap # in AX, args in DI SI DX R10 R8 R9, return in AX DX
// Note that this differs from "standard" ABI convention, which
// would pass 4th arg in CX, not R10.

TEXT	·Syscall(SB),NOSPLIT,$0-56
	CALL	runtime·entersyscall(SB)
	MOVQ	a1+8(FP), DI
	MOVQ	a2+16(FP), SI
	MOVQ	a3+24(FP), DX
	MOVQ	$0, R10
	MOVQ	$0, R8
	MOVQ	$0, R9
	MOVQ	trap+0(FP), AX	// syscall entry
	SYSCALL
	CMPQ	AX, $0xfffffffffffff001
	JLS	ok
	MOVQ	$-1, r1+32(FP)
	MOVQ	$0, r2+40(FP)
	NEGQ	AX
	MOVQ	AX, err+48(FP)
	CALL	runtime·exitsyscall(SB)
	RET
ok:
	MOVQ	AX, r1+32(FP)
	MOVQ	DX, r2+40(FP)
	MOVQ	$0, err+48(FP)
	CALL	runtime·exitsyscall(SB)
	RET
```

runtime·entersyscall 和 runtime·exitsyscall相关参见 [调度](schedule.md) 一节

1. go 中 read 调用 Syscall前，编译器将 trap , a1, a2, a3, r1, r2, err （64bit 地址或int）压栈，从 0(FP) 到 48(FP), 共 48/8 + 1 = 7个, 没错。
2. Syscall 函数根据 linux 内核 syscall 对 amd64 寄存器的规则（参考文献1）将堆栈上的数据存入寄存器，调用syscall
3. linux 系统调用 如果出错返回一负数。 CMPQ 和 JLS指令检查ax中的返回值如果是负数，就把错误码放到 err中，否则把AX，DX分别放到 r1, r2， err = 0。 对于linux amd64，只用AX传返回值，r2 应该实际不会用到。



### Syscall6 

是更完整的系统调用，该用的寄存器都用了，但多数用Syscall的三个参数就够了。

```
// func Syscall6(trap, a1, a2, a3, a4, a5, a6 uintptr) (r1, r2, err uintptr)
TEXT ·Syscall6(SB),NOSPLIT,$0-80
	MOVQ	a1+8(FP), DI
	MOVQ	a2+16(FP), SI
	MOVQ	a3+24(FP), DX
	MOVQ	a4+32(FP), R10
	MOVQ	a5+40(FP), R8
	MOVQ	a6+48(FP), R9
	MOVQ	trap+0(FP), AX	// syscall entry
```

###  RawSyscall 或 RawSyscall6
* 用于不阻塞的系统调用，实现区别就是没有没有 runtime·entersyscall 和 runtime·exitsyscall

```
func socket(domain int, typ int, proto int) (fd int, err error) {
	r0, _, e1 := RawSyscall(SYS_SOCKET, uintptr(domain), uintptr(typ), uintptr(proto))
	fd = int(r0)
	if e1 != 0 {
		err = errnoErr(e1)
	}
	return
}
```

* syscall/zsyscall_darwin_amd64.go 中的是用perl脚本以 syscall/syscall_linux_amd64.go为输入生成的，依据如下注释，sys和sysnb决定用不用RawXX，再看参数个数决定用不用XX6；另外一些特殊系统调用如 gettimeofday 单独处理。

```
//sys	accept(s int, rsa *RawSockaddrAny, addrlen *_Socklen) (fd int, err error)
//sysnb	socket(domain int, typ int, proto int) (fd int, err error)
```



# 参考文献

1. [X86 Assembly/Interfacing with Linux](https://en.wikibooks.org/wiki/X86_Assembly/Interfacing_with_Linux)
2. [linux系统调用src快速索引](https://filippo.io/linux-syscall-table/)
3. [The Linux kernel](http://www.win.tue.nl/~aeb/linux/lk/lk-4.html)
4. [X86-64寄存器和栈帧](http://www.searchtb.com/2013/03/x86-64_register_and_function_frame.html)