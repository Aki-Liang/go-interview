# 操作系统调度进程算法

## FCFS

先来先服务（First Come First Serve）

每个进程按进入内存的时间先后排成一队。每当 CPU 上的进程运行完毕或者阻塞，选择队伍最前面的进程执行

缺点： 短进程的响应时间太长了，用户交互体验会变差。

## SPN

「短任务优先」（Shortest Process Next，SPN）

每次选择预计处理时间最短的进程

长进程经常得不到 CPU 资源，造成了「饥饿」现象。如果短进程源源不断加入队列，长进程们将永远得不到执行的机会

## HRRN

「高响应比优先」（Highest Response Ratio Next，HRRN）

响应比 = （等待时间+要求服务时间）/ 要求服务时间。响应比高的会先执行

每次调度前，都要重新计算所有等待进程的响应比

## RR

时间片轮转算法（Round Robin，RR）

轮转法是基于适中的抢占策略的，以一个周期性间隔产生时钟中断，当中断发生后，当前正在运行的进程被置于就绪队列中，然后基于先来先去服务策略选择下一个就绪作业的运行。这种技术也称为时间片，因为每个进程再被抢占之前都给定一片时间。

优点：公平；响应快，适用于分时操作系统；

缺点：由于高频的进程切换,因此有一定开销；不区分任务的紧急程度。

## 优先级调度算法
* 算法思想：随着计算机的发展，特别是实时操作系统的出现，越来越多的应用场景需要根据任务的紧急程度来决定处理进程的顺序、
* 算法规则：调度时选择优先级最高的进程
* 用于作业/进程：既可用于作业调度，也可用于进程调度。甚至还会用于在之后学习的IO调度
* 是否是可抢占的：抢占式和非抢占式都有。非抢占式只需要在进程主动放弃处理机时进程调度即可，而抢占式还需在就绪队列变化时，检查是否会发生抢占。
* 优点：有优先级区分紧急程度，适用于实时操作系统。可灵活地调整对各个作业/进程的编号程度。
* 缺点：若有源源不断的高优先级进程到来，低优先级进程可能会导致饥饿。
* 是否会导致饥饿：会

根据优先级是否可以动态改变，可将优先级分为静态优先级和动态优先级两种。

* 静态优先级：创建进程时就确定，之后一直不变
* 动态优先级：创建进程的时候有个初始值，之后会根据情况动态地调整优先级。

## 多级反馈队列调度算法

* 算法思想：对其他调度算法的综合
* 算法规则：
1、设置多级就绪队列，各级队列优先级从高到低，时间片从小到大
2、新进程到达时先进入1级队列，按照先来先服务原则排队等待被分配时间片，若用完时间片还未结束，则进程进入下一级队列，则重新放回该队列队尾。
3、只有第k级队列为空时，才会为k+1级队头的进程分配时间片
用于进程调度
* 是否可抢占：抢占式算法，在K级队列的进程运行过程中，如果更上级别的队列中新进入了一个进程，则由于新进程处于优先级更高的队列中，因此新进程先在处理机运行，原来运行的进程放回k级队列队尾。
* 优点：对各类型进程相对公平（FCFS的优点）；每个新到达的进程都可以很快就得到响应（RR的优点）；短进程只用较少的时间就可完成。（SPF的优点）；不必实现估计进程的运行时间（避免用户作假）；可灵活地调整对各类进程的偏好程度，比如CpU密集型进程、I/O密集型进程（拓展：可以将因I/O而阻塞的进程重新放回原队列，这样I/O型进程就可以保持较高优先级）
* 是否会导致饥饿： 会

-----
## 资料

https://blog.csdn.net/qq_43672652/article/details/105348498

https://cloud.tencent.com/developer/article/1549497