---
title: linux top about cpu
date: 2016-12-07 12:48:13
categories: PHP
tags:
 - linux
 - cpu
---

```
Tasks: 179 total,   4 running, 175 sleeping,   0 stopped,   0 zombie
%Cpu(s): 10.3 us,  1.3 sy,  0.0 ni, 88.5 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:   3012668 total,  1073356 used,  1939312 free,    52772 buffers
KiB Swap:  1308668 total,        0 used,  1308668 free.   409160 cached Mem

```

`us` 用户态 cpu执行用户进程的时间，包括nice时间，一般用户空间cpu高点比较好

`sy` 系统 cpu在内核运行时间，包括软硬终中断时间，如果此值较高，说明系统有问题

`id` idle 系统空闲
`wa` waiting 系统等待io操作所花费的时间，系统不应该花费太多时间在此，如果有，说明io有瓶颈
`hi` hard irq time 硬中断，一般由硬件发出
`si` soft irq time 软中断，由信号发出，程序指令发出，比如等待io请求，一般没有硬件的参与
`st` steal time，被强制等待 虚拟cpu的时间，比如为另一个虚拟处理器服务

以上输出根据 `/proc/stat`

```
tb@tb:~$ cat /proc/stat
cpu  5803 228 1006 54723 212 0 22 0 0 0
cpu0 5803 228 1006 54723 212 0 22 0 0 0
intr 105215 41 238 0 0 0 0 0 0 0 0 0 0 288 0 0 741 0 0 0 3170 1912 14235 58 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
ctxt 1172838
btime 1481084918
processes 2570
procs_running 6
procs_blocked 0
softirq 48007 4 24384 126 3233 14075 0 271 0 0 5914
```
<a href="http://download.csdn.net/detail/u010703523/9704275" >了解更多</a>


