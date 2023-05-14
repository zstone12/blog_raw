---
title: bcc/MySQL相关踩坑过程
date: 2021-04-08 22:54:17
tags:
---

### 背景

毕设做的是用 ebpf 对 MySQL tracing/monitor 的一个项目。本着站在巨人的肩膀上前行的想法，先看看bcc 中MySQL相关的工具。如果能在bcc工具基础上修改，那就再好不过了。

<!--more--> 

### 1. bcc

对 bcc 项目进行全局搜索。

bcc/tools 目录下相关于MySQL 的工具有 `dbstat.py` `dbslower.py`

代码看着都还挺像回事的。

 `Copyright 2017, Sasha Goldshtein` 

都是这位大佬`Sasha Goldshtein` 在17年完成的。

直接开搞。RUN IT!

```bash
$ python3 dbstat.py mysql 
```

出现了报错。

```python
raise USDTException("USDT failed to instrument PID %d" % pid)
  bcc.usdt.USDTException: USDT failed to instrument PID 4803
```

google it!

在bcc的issue中提到了这个报错。

https://github.com/iovisor/bcc/issues/2330

yonghong-song 提到

>  You need to recompile `mysqld` with usdt support. See
>
> https://github.com/iovisor/bcc/blob/master/man/man8/mysqld_qslower.8#L17-L18

顺藤摸瓜。

发现默认的 MySQL 并没有开启 dtrace。

需要在编译 MySQL 的时候带上参数`-DENABLE_DTRACE=1`。

同时也发现在`dbstat.py` `dbslower.py`的注释中都有提到这一点。

```
dbstat.py
# This tool uses USDT probes, which means it needs MySQL and PostgreSQL built
# with USDT (DTrace) support.

dbslower.py
# Script works in two different modes:
# 1) USDT probes, which means it needs MySQL and PostgreSQL built with USDT (DTrace) support.
```

**现在问题就转化为编译安装MySQL并在编译时带上参数。事情在现在看来都是可解决的。**

### 2 .compile MySQL

步骤看着挺简单。

```
git clone --depth=1 https://github.com/MariaDB/server mariadb
cmake . -DENABLE_DTRACE=1
make -j $NUMPROCS
sudo make install
```

当`make` 的时候遇到报错

类似https://github.com/iovisor/bcc/issues/2233

```bash
make[2]: DTRACE-NOTFOUND: Command not found
```

貌似是缺失dtrace . 需要确认一下。

`issue2233` 里面提到了一个重要文档。
https://dev.mysql.com/doc/refman/5.7/en/dba-dtrace-server.html

问题又转化成安装 dtrace.

### 三: dtrace

#### 3.1dtrace 是什么？

```
DTrace 是动态追踪技术的鼻祖，它于 21 世纪初诞生于 Solaris 操作系统，
是由原来的 Sun Microsystems 公司的工程师编写的，
先后被移植到 Linux、FreeBSD、NetBSD 及 Mac OS X 等操作系统上。
iOS 系统也有，大名鼎鼎的 Instrument 工具就是基于 DTrace 实现的，
而且更多的功能还在随着 iOS 系统进行版本迭代。
```

#### 3.2 dtrace with MySQL

处理刚才提到的 MySQL 官方文档。

```
MySQL includes support for DTrace probes on these platforms:

Solaris 10 Update 5 (Solaris 5/08) on SPARC, x86 and x86_64 platforms

OS X 10.4 and higher

Oracle Linux 6 and higher with UEK kernel (as of MySQL 5.7.5)

Enabling the probes should be automatic on these platforms. To explicitly enable or disable the probes during building, use the -DENABLE_DTRACE=1 or -DENABLE_DTRACE=0 option to CMake.

If a non-Solaris platform includes DTrace support, building mysqld on that platform includes DTrace support.
```

翻译如下:

```
MySQL在这些平台上支持DTrace探针：

Solaris 10 Update 5 (Solaris 5/08) on SPARC, x86 and x86_64 platforms
OS X 10.4 and higher
Oracle Linux 6 and higher with UEK kernel (as of MySQL 5.7.5)

在这些平台上应该是自动的 启用探针。 要在构建过程中明确启用或禁用探针，请使用CMake的-DENABLE_DTRACE = 1或-DENABLE_DTRACE = 0选项。

如果非Solaris平台包含DTrace支持，则在该平台上构建mysqld将包含DTrace支持。
```

得到了关键信息。在ubuntu 或 Linux 下使用 dtrace 需要编译。

只是更明确了一下。

**问题现在聚焦于解决编译 MySQL 时的报错。**

#### 3.3 dtrace4linux

google  如何在linux 平台上安装dtrace 指向了这么一个仓库。

https://github.com/dtrace4linux/linux

```
This is a port of the Sun DTrace user and kernel code to Linux. No linux kernel code is touched in this build, but what is produced is a dynamically loadable kernel module. This avoids licensing issues and allows people to load and update dtrace as they desire.

The goal of this project is to make available DTrace for the Linux platforms. By making it available for everyone, they can use it to optimise their systems and tools, and in return, I get to benefit from their work.
```

问题好像变得只需要需要dtrace4linux就行了。

ubuntu/fedora 上就能有 dtrace.

```
$ tools/get-deps-arch.sh	# if using ArchLinux
$ tools/get-deps.pl 		# if using Ubuntu
$ tools/get-deps-fedora.sh	# RedHat/Fedora

$ make all
$ make install
$ make load           (need to be root or have sudo access)
```

执行上面的命令。在`make all`遇到报错。 

ubuntu下遇到。

```
fatal error: /usr/include/sys/types.h: No such file or directory
  142 |  # include "/usr/include/sys/types.h"
      |            ^~~~~~~~~~~~~~~~~~~~~~~~~~
```

切到 fedora 该问题解决。但是又碰到新问题。

```
Make all error - FATAL ERROR: cannot find old_rsp FATAL ERROR: build.pl aborting
```

google it.

发现issue https://github.com/dtrace4linux/linux/issues/113

> 大家都遇到了同样的问题，且没有解决方案。

再看了看 dtrace4linux 应该是不再维护了。

至此这条路是走不通了。

way 2: `Solaris 10` 直接自带。但不是 linux 系统, make no sense.

way 3: https://blogs.oracle.com/linux/dtrace-on-fedora

直接编译dtrace内核模块 编译fedora. 

耗时几个小时，	

测试在拉fedora 源码这一步就因为网络不稳定进行不下去了。

而且这种方式看起来过于暴力了。

---

### 4.Last

另一个方向: use uprobe

关键的一条信息:

http://minervadb.com/wp-content/uploads/2020/12/Dynamic-Tracing-for-Finding-and-Solving-MySQL-Performance-Problems-on-Linux-MinervaDB-Database-Platforms-Virtual-Conference-2020.pdf

page 14

>  hard way to go...

