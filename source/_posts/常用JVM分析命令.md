---
title: 常用JVM分析命令
date: 2017-12-24 13:34:01
categories: java
tags: [java,jvm]
comments: true

---
>常用JVM分析命令基本就是jps、jstack、jmap、jstat几个命令，配合top等命令可以定位大多数的问题。<!-- more -->

### jps
JVM Process Status Tool,显示指定系统内所有的HotSpot虚拟机进程。

#### 命令格式:
```shell
jps [options] [hostid]
```

#### option参数:
```shell
-l : 输出主类全名或jar路径
-q : 只输出LVMID
-m : 输出JVM启动时传递给main()的参数
-v : 输出JVM启动时显示指定的JVM参数
```

#### 用例:
```shell
$ jps -l
46644 sun.tools.jps.Jps
46582 com.intellij.rt.execution.application.AppMain
46378 org.jetbrains.idea.maven.server.RemoteMavenServer
46581 org.jetbrains.jps.cmdline.Launcher
```

### jstat
JVM statistics Monitoring,是用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。

#### 命令格式:
```shell
jstat [option] pid [interval] [count]
[option] : 操作参数
pid : 本地虚拟机进程ID
[interval] : 连续输出的时间间隔
[count] : 连续输出的次数
```

#### option参数:
```shell
-class : class loader的行为统计
-compiler : HotSpt JIT编译器行为统计
-gc : 垃圾回收堆的行为统计
-gccapacity : 各个垃圾回收代容量(young,old,perm)和他们相应的空间统计
-gccause :垃圾收集统计概述（同-gcutil），附加最近两次垃圾回收事件的原因 
-gcutil : 垃圾回收统计概述
others : ...
```

#### 用例:
```shell
$ jstat -gcutil 46582  1000
  S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT   
  0.00   0.00   4.00  42.73  14.19     15    0.059     4    0.573    0.632
  0.00   0.00   4.00  42.73  14.19     15    0.059     4    0.573    0.632
  0.00   0.00   4.00  42.73  14.19     15    0.059     4    0.573    0.632
  0.00   0.00   4.00  42.73  14.19     15    0.059     4    0.573    0.632
  0.00   0.00   4.00  42.73  14.19     15    0.059     4    0.573    0.632
```

### jstack
jstack用于生成java虚拟机当前时刻的线程快照。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。

#### 命令格式:
```shell
jstack [option] pid
```

#### option参数:
```shell
-F : 当正常输出请求不被响应时，强制输出线程堆栈
-l : 除堆栈外，显示关于锁的附加信息
-m : 如果调用到本地方法的话，可以显示C/C++的堆栈
```

#### 用例:
```shell
$ jstack -l 46582
Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.79-b02 mixed mode):

"Attach Listener" daemon prio=5 tid=0x00007fb30a186800 nid=0x3307 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"DestroyJavaVM" prio=5 tid=0x00007fb30a08d000 nid=0xd03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Thread-1" prio=5 tid=0x00007fb30991e800 nid=0x4c03 runnable [0x0000000114bb5000]
   java.lang.Thread.State: RUNNABLE
	at java.util.HashMap.put(HashMap.java:494)
	at jvm.ResizeDemo$2.run(ResizeDemo.java:26)
	at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
	- None
```

#### 注意:
* stack文件是瞬时线程堆栈,一定要多个时刻的stack文件,以便于确定哪些线程是真正有问题的。
* top -Hp pid可以查看某个进程内的线程状态,线程id换算成16进制对应stack文件中nid的值,通常用来定位有问题的线程(cpu消耗过高等)
```shell
top -Hp 17072
PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
18167 work      20   0 8793m 2.5g  14m S  10.6 16.3  16:19.55 java
17927 work      20   0 8793m 2.5g  14m S  1.0 16.3   5:15.71 java
20516 work      20   0 8793m 2.5g  14m S  1.0 16.3   3:52.57 java
```

### jmap
JVM Memory Map,用于生成heap dump文件,可以使用-XX:+HeapDumpOnOutOfMemoryError参数来让虚拟机出现OOM的时候自动生成dump文件,以便分析问题。

#### 命令格式:
```shell
jmap [option] pid
```

#### option参数:
```shell
dump : 生成堆快照
heap : 显示Java堆详细信息
histo : 显示堆中对象的统计信息
others : ...
```

#### 用例:
```shell
$ jmap -dump:live,format=b,file=dump.bin 46582
Dumping heap to /Users/xiao1zhao2/blog/dump.bin ...
Heap dump file created
```
生成的dump文件一般用mat等分析工具进行查看。