---
layout: post
comments: true
title: "浅谈系统线程数限制"
description: "对系统线程数限制的认识"
categories: ["线程", "系统"]
---

## Linux进程与线程

概念就不提了，Richard Stevens的描述：
>
fork is expensive. Memory is copied from the parent to the child, all descriptors are duplicated in the child, and so on. Current implementations use a technique called copy-on-write, which avoids a copy of the parent's data space to the child until the child needs its own copy. But, regardless of this optimization, fork is expensive.
IPC is required to pass information between the parent and child after the fork. Passing information from the parent to the child before the fork is easy, since the child starts with a copy of the parent's data space and with a copy of all the parent's descriptors. But, returning information from the child to the parent takes more work.
Threads help with both problems. Threads are sometimes called lightweight processes since a thread is "lighter weight" than a process. That is, thread creation can be 10–100 times faster than process creation.
All threads within a process share the same global memory. This makes the sharing of information easy between the threads, but along with this simplicity comes the problem.

Linux中创建进程用fork操作，线程用clone操作。通过ps -ef看到的是进程列表，线程可以通过ps -eLf来查看。
用top命令的话，通过H开关也可以切换到线程视图。

具体到Java线程模型，规范是没有规定Java线程和系统线程的对应关系的，不过目前常见的实现是一对一的。

## 问题排查思路

如果创建不了Java线程，报错是

***Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread***

下面是常见的问题原因：

* 内存太小

在Java中创建一个线程需要消耗一定的栈空间，默认的栈空间是1M(可以根据应用情况指定-Xss参数进行调整)，栈空间过小或递归调用过深，可能会出现StackOverflowError。

对于一个进程来说，假设一定量可使用的内存，分配给堆空间的越多，留给栈空间的就越少。这个限制常见于**32位Java应用**，进程空间4G，用户空间2G(Linux下3G，所以通常堆可以设置更大一些)，减去堆空间大小(通过-Xms、-Xmx指定范围)，减去非堆空间(其中永久代部分通过PermSize、MaxPermSize指定大小，在Java8换成了MetaSpace，默认不限制大小)，再减去虚拟机自身消耗，剩下的就是栈空间，假设剩下300M，那么理论上就限制了只能开300线程。不过对于64位应用，由于进程空间近乎无限大，所以可以不考虑这个问题。

* ulimit限制

线程数还会受到系统限制，系统限制通过ulimit -a可以查看到。

```
caixj@Lenovo-PC:~$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7823
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 7823
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

相关的限制有

```
max memory size - 最大内存限制，在64位系统上通常都设置成unlimited  
max user processes - 每用户总的最大进程数(包括线程) 
virtual memory - 虚拟内存限制，在64位系统上通常都设置成unlimited 
```

这些参数可以通过ulimit命令(当前用户临时生效)或者配置文件/etc/security/limits.conf(永久生效)进行修改。

* 参数sys.kernel.threads-max限制

表示系统全局的总线程数限制。设置方式有:

```
# 方式1 运行时限制,临时生效
echo 999999 > /proc/sys/kernel/threads-max
# 方式2 修改/etc/sysctl.conf，永久生效
sys.kernel.threads-max = 999999
```

* 参数sys.kernel.pid_max限制

表示系统全局的PID号数值的限制。设置方式有:

```
# 方式1 运行时限制,临时生效
echo 999999 > /proc/sys/kernel/pid_max
# 方式2 修改/etc/sysctl.conf，永久生效
sys.kernel.pid_max = 999999
```

* 参数sys.vm.max_map_count限制

表示单个程序所能使用虚拟内存映射数量的限制。设置方式有:

```
# 方式1 运行时限制,临时生效
echo 999999 > /proc/sys/vm/max_map_count
# 方式2 修改/etc/sysctl.conf，永久生效
sys.vm.max_map_count = 999999
```

在其他资源可用的情况下，单个vm能开启的最大线程数是这个值的一半。

```
Attempt to protect stack guard pages failed.
and OpenJDK 64-Bit Server VM warning: Attempt to deallocate stack guard pages failed. error messages are emitted
by JavaThread::create_stack_guard_pages(), and it calls os::guard_memory(). 
In Linux, this function is mprotect(). 
```

# cgroup限制

例如suse sp2的发行说明，见https://www.suse.com/releasenotes/x86_64/SUSE-SLES/12-SP2/#fate-320358

```
If you notice regressions, you can change a number of TasksMax settings.

To control the default TasksMax= setting for services and scopes running on the system, use the system.conf setting DefaultTasksMax=. This setting defaults to 512, which means services that are not explicitly configured otherwise will only be able to create 512 processes or threads at maximum.

For thread- or process-heavy services, you may need to set a higher TasksMax value. In such cases, set TasksMax directly in the specific unit files. Either choose a numeric value or even infinity.

Similarly, you can limit the total number of processes or tasks each user can own concurrently. To do so, use the logind.conf setting UserTasksMax (the default is 12288).

nspawn containers now also have a TasksMax value set, with a default of 16384.
```


