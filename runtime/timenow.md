# GO语言中time返回的是什么时间

## 结论
    2157年之前，Now返回的是1885年到现在的秒数（挂钟时间，wall字段记录）和单调时间（从进程启动到当前时间的间隔，单位纳秒，ext字段记录）

    2157年之后，wall 字段的最高位为 0，只使用低 30 位记录 nsec，ext记录从公元元年1月1日到当前的时间的秒数


## 源码

```
func Now() Time {
    sec, nsec, mono := now()
    mono -= startNano
    sec += unixToInternal - minWall
    if uint64(sec)>>33 != 0 {
        return Time{uint64(nsec), sec + minWall, Local}
    }
    return Time{hasMonotonic | uint64(sec)<<nsecShift | uint64(nsec), mono, Local}
}

//go:linkname time_now time.now
func time_now() (sec int64, nsec int32, mono int64) {
	sec, nsec = walltime()
	return sec, nsec, nanotime()
}

// $GOROOT/src/runtime/sys_linux_amd64.s, line 178
// func walltime() (sec int64, nsec int32)
TEXT runtime·walltime(SB),NOSPLIT,$0-12
获取挂钟时间，先尝试调用vdso_clock_gettime CLOCK_REALTIME 精度纳秒，失败则调用vdso_gettimeofday精度为微妙

// $GOROOT/src/runtime/sys_linux_amd64.s, line 236
TEXT runtime·nanotime(SB),NOSPLIT,$0-8
获取单调时间，先尝试调用vdso_clock_gettime CLOCK_MONOTONIC获取从开机到现在的时间间隔，不受挂钟时间更改的影响，失败则调用vdso_gettimeofday获取挂钟时间
```

```
mono -= startNano

// $GOROOT/src/time/time.go, line 1090
var startNano int64 = runtimeNano() - 1

// $GOROOT/src/time/time.go, line 1081
//go:linkname runtimeNano runtime.nanotime
func runtimeNano() int64
```
用进程初始化时的单调时间，去减当前的单调时间，得到从进程初始化到当前的时间间隔

```
sec += unixToInternal - minWall

// $GOROOT/src/time/time.go, line 440
unixToInternal int64 = (1969*365 + 1969/4 - 1969/100 + 1969/400) * secondsPerDay

// $GOROOT/src/time/time.go, line 153
minWall      = wallToInternal

// $GOROOT/src/time/time.go, line 443
wallToInternal int64 = (1884*365 + 1884/4 - 1884/100 + 1884/400) * secondsPerDay
```

时间戳的秒部分 sec 加上了从1885年到1970年之间的秒数,`回到未来3`剧情梗

## VDSO (Virtual Dynamically-lined Shared Object)

vsdo是一个由内核提供的虚拟.so文件，它不在磁盘上，而在内核里，内核将其映射到一个地址空间中，被所有程序共享，正文段大小为一个页面。

vdso将内核态的调用映射到用户态的地址空间中，使得调用开销更小，路径更好。

在用户运行新的进程时，系统会将该共享库Linux-vdso.so.1映射到进程空间中，不同的进程映射到进程空间的线性地址是不同的，所以可以看到VDSO在不同的进程地址空间中有不同的线性地址。当有系统调用触发时就会调用VDSO页面中的__kernel_vsyscall处的汇编语句，进而出发sysenter指令进入系统调用。

传统的系统调用方式是通过软中断指令int 0x80实现的

当用户态的进程调用一个系统调用时，cpu从用户态切换到内核态开始执行一个内核函数。对于X86架构来说，有两种不同的方式调用系统调用：

（1）执行int $0x80(可编程异常)汇编语言指令。在Linux内核的老版本中，这是从用户态切换到内核态的唯一方式。执行iret汇编指令返回到用户态。

（2）执行sysenter汇编语言指令。在Intel Pentium II微处理器芯片中引入了这条指令，现在Linux 2.6内核支持这条指令。执行sysexit汇编指令返回到用户态。


只有在调用clock_gettime、gettimeofday、getcpu、time这些系统调用时，才会使用vdso