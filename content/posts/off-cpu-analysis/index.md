---
title: "Off-CPU Analysis"
subtitle: Notes on [Off-CPU Analysis by Brendan Gregg](https://www.brendangregg.com/offcpuanalysis.html)

summary: Notes on [Off-CPU Analysis by Brendan Gregg](https://www.brendangregg.com/offcpuanalysis.html)

date: 2024-01-24T09:12:00+08:00

lastmod: 2024-01-24T09:12:00+08:00

draft: false

featured: false

authors:
  - admin

categories:
  - Performance
  - Linux
---


## Introduction

性能分析的时候，通常需要分析两个部分，包括On-CPU time和Off-CPU time两个部分。

On-CPU time就是进程在CPU上运行的时间，再细分的话可能还包括sys time(kernel部分)和user time(userland)两个部分，详细信息可见`time(1)`.

Off-CPU time就是进程不在CPU上运行的时间，一般来说，进程可能会因为某些锁、等待IO等原因阻塞，也有可能因为时间片到了，在中断的时候被操作系统放入run queue当中。

* Off-CPU tracing 就是在内核进程状态改变的函数添加一些代码(静态？or 动态？)进行一些统计
* Off-CPU sampling 可以采样所有的进程的stack trace，在进行分析的时候筛选出那些off-CPU的stack trace进行分析。但是，on-CPU sampling一般是CPU中断的时候采样，如果要采样off-CPU time的话，可能需要应用程序的每个进程都添加一个定时器，或者让kernel每隔一段时间扫描它们的栈。


## Overhead

当前主要由Perf和eBPF两种方法，eBPF可以做in-kernel summary，而其它方式可能还要往存储设备写入收集到的数据，供后续用户态的程序进行分析，这会带来比较大的开销。

## Off-CPU Analysis

通常会instrument一些内核函数，比如cpudist工具使用了kprobe(kernel dynamic tracing)instrument了`static struct rq *finish_task_switch(struct task_struct *prev)`这个函数。

这个函数在下一个将要运行的进程的上下文中被调用，可以通过eBPF函数获得PID(`bpf_get_current_pid_tgid()`)，高精度的时间戳(`bpf_ktime_get_ns()`)等信息。


off-CPU time计算伪代码
```
on context switch finish:
	sleeptime[prev_thread_id] = timestamp
	if !sleeptime[thread_id]
		return
	delta = timestamp - sleeptime[thread_id]
	totaltime[pid, execname, user stack, kernel stack] += delta
	sleeptime[thread_id] = 0

on tracer exit:
	for each key in totaltime:
		print key
		print totaltime[key]
```

## Request-Synchronous Context

有一些程序维护进程池，分析的结果可能包括大量
等待的事件，掩盖了真正需要分析的代码路径。比如，要分析SQL query执行的off-CPU time，在query比较少的时候，SQL query就可能被大量等待外部连接的进程掩盖，找不到想要的结果。

为了避免这样的情况，可以根据程序特点intrument特定的函数，比如MySQL中的`do_command() -> mysql_execute_command()`。


## Caveat

### 调度器延迟
当进程状态变为可运行的时候(Runnable)进入run queue到真正被调度到cpu上运行有一段延时。

由于是在context switch的时候结束计时，因此按照上面的算法，off-CPU time 包括了进程
在run queue中等待的情况。

当服务器负载特别大（可运行进程特别多）的时候，这样的延迟就会加剧，被称作：dispatcher latency或scheduler latency或run queue latency。

由于这些延迟在服务器负载大的时候才比较显著，在这种情况下，首先解决的不是off-CPU time的问题，而是服务器负载大的问题。

### Involuntary Context Switch

同样是服务器负载比较大的场景，每个进程的时间片可能比较小，非自愿的进程切换比较频繁。非自愿的进程切换一般是时间片用完后，中断处理程序切换其它进程，进程仍然是Runnable(TASK_RUNNING)，但还是进入了一段off-CPU的等待时间。

在这种情况下，可以只关注TASK_UNINTERRUPTIBLE和TASK_INTERRUPTIBLE这两种类型的进程。

* `TASK_UNINTERRUPTIBLE`:进程不可被signal打断，比如说一个进行原子写操作的进程。处于该状态的进程必须被显式（explictly）唤醒。
* `TASK_INTERRUPTIBLE` : 当 1）被显式唤醒 或 2）收到一个没有被屏蔽的信号 后停止睡眠。

[来自 TASK_KILLABLE on lwn.net](https://lwn.net/Articles/288056/)

> Like most versions of Unix, Linux has two fundamental ways in which a process can be put to sleep. A process which is placed in the TASK_INTERRUPTIBLE state will sleep until either (1) something explicitly wakes it up, or (2) a non-masked signal is received. The TASK_UNINTERRUPTIBLE state, instead, ignores signals; processes in that state will require an explicit wakeup before they can run again.

它们的优缺点
> There are advantages and disadvantages to each type of sleep. Interruptible sleeps enable faster response to signals, but they make the programming harder. Kernel code which uses interruptible sleeps must always check to see whether it woke up as a result of a signal, and, if so, clean up whatever it was doing and return -EINTR back to user space. The user-space side, too, must realize that a system call was interrupted and respond accordingly; not all user-space programmers are known for their diligence in this regard. Making a sleep uninterruptible eliminates these problems, but at the cost of being, well, uninterruptible. If the expected wakeup event does not materialize（**发生**）, the process will wait forever and there is usually nothing that anybody can do about it short of rebooting the system. This is the source of the dreaded, unkillable process which is shown to be in the "D" state by ps.

简单来说，TASK_INTERRUPTIBLE可以更快地响应信号，但需要内核代码检查该进程被唤醒是由于信号的原因并返回-EINTR错误，用户代码也要做相应的信号处理。

TASK_UNINTERRUPTIBLE可以避免上述的问题，如果wakeup event没有发生，那么该进程将会永久等待，也无法通过kill杀掉，只能重启系统。这种unkillable进程在ps中显示为D。



## Resources and References

* [动态追踪技术漫谈](https://blog.openresty.com.cn/cn/dynamic-tracing/)
* [TASK_KILLABLE on lwn.net](https://lwn.net/Articles/288056/)