---
layout: post
title: Linux ulimit详解
categories: [program]
tags: [linux]
---

今天在跑程序的时候，因数据量过大，直接导致core dump了。因为程序中有个函数的内部变量中有一个是大数组，超过了系统分配给每个进程的最大stack size。后来找到这个ulimit，把stack size改了，就没问题了。下面把这个命令的详细记录如下，以备不时之需。

（以下内容转自(http://www.ej38.com/showinfo/linux-202107.html)， 并整理部分内容。）

命令ulimit

1 说明：

用于shell启动进程所占用的资源。属于shell的内建命令。

2 语法格式：
{% highlight bash %}
ulimit [-acdfHlmnpsStvw] [size]
{% endhighlight %}

3 参数介绍：

-H       设置硬件资源限制.

-S       设置软件资源限制.

-a       显示当前所有的资源限制.

-c size  设置core文件的最大值.单位:blocks

-d size  设置数据段的最大值.单位:kbytes

-f size  设置创建文件的最大值.单位:blocks

-l size  设置在内存中锁定进程的最大值.单位:kbytes

-m size  设置可以使用的常驻内存的最大值.单位:kbytes

-n size  设置内核可以同时打开的文件描述符的最大值.单位:n

-p size  设置管道缓冲区的最大值.单位:kbytes

-s size  设置堆栈的最大值.单位:kbytes

-t size  设置CPU使用时间的最大上限.单位:seconds

-v size  设置虚拟内存的最大值.单位:kbytes

4 举例

在Linux下，如果程序比较大，经常会遇到“段错误”（segmentation fault）这样的问题，主要就是由于Linux系统初始的堆栈大小（stack size）太小的缘故，一般为10M。我一般把stack size设置成256M，这样就没有段错误。

命令：

ulimit   -s 262140

如果要系统自动记住这个配置，就编辑/etc/profile文件，在 “ulimit -S -c 0 > /dev/null 2>&1”行下，添加“ulimit   -s 262140”，保存重启系统就可以了！

1. 在RH8的环境文件/etc/profile中,我们可以看到系统是如何配置ulimit的:

{% highlight bash %}
#grep ulimit /etc/profile
ulimit -S -c 0 > /dev/null 2>&1
{% endhighlight %}

这条语句设置了对软件资源和对core文件大小的设置

2如果我们想要对由shell创建的文件大小作些限制,如:

{% highlight bash %}
#ll h
-rw-r–r– 1 lee lee 150062 7月 22 02:39 h
#ulimit -f 100 #设置创建文件的最大块(一块=512字节)
#cat h>newh
File size limit exceeded
#ll newh
-rw-r–r– 1 lee lee 51200 11月 8 11:47 newh
{% endhighlight %}

文件h的大小是150062字节,而我们设定的创建文件的大小是512字节x100块=51200字节
当然系统就会根据你的设置生成了51200字节的newh文件.
3]可以像实例1]一样,把你要设置的ulimit放在/etc/profile这个环境文件中.
用途
设置或报告用户资源极限。
语法
ulimit [ -H ] [ -S ] [ -a ] [ -c ] [ -d ] [ -f ] [ -m ] [ -n ] [ -s ] [ -t ] [ Limit ]
描述
ulimit 命令设置或报告用户进程资源极限，如 /etc/security/limits 文件所定义。文件包含以下缺省值极限：
fsize = 2097151
core = 2097151
cpu = -1
data = 262144
rss = 65536
stack = 65536
nofiles = 2000
当新用户添加到系统中时，这些值被作为缺省值使用。当向系统中添加用户时，以上值通过 mkuser 命令设置，或通过 chuser 命令更改。
极限分为软性或硬性。通过 ulimit 命令，用户可将软极限更改到硬极限的最大设置值。要更改资源硬极限，必须拥有 root 用户权限。
很多系统不包括以上一种或数种极限。 特定资源的极限在指定 Limit 参数时设定。Limit 参数的值可以是每个资源中指定单元中的数字，或者为值 unlimited。要将特定的 ulimit 设置为 unlimited，可使用词 unlimited。
注：在 /etc/security/limits 文件中设置缺省极限就是设置了系统宽度极限， 而不仅仅是创建用户时用户所需的极限。
省略 Limit 参数时，将会打印出当前资源极限。除非用户指定 -H 标志，否则打印出软极限。当用户指定一个以上资源时，极限名称和单元在值之前打印。如果未给予选项，则假定带有了 -f 标志。
由于 ulimit 命令影响当前 shell 环境，所以它将作为 shell 常规内置命令提供。如果在独立的命令执行环境中调用该命令，则不影响调用者环境的文件大小极限。以下示例中正是这种情况：
nohup ulimit -f 10000
env ulimit 10000
一旦通过进程减少了硬极限，若无 root 特权则无法增加，即使返回到原值也不可能。
关于用户和系统资源极限的更多信息，请参见 AIX 5L Version 5.3 Technical Reference: Base Operating System and Extensions Volume 1 中的 getrlimit、setrlimit 或 vlimit 子例程。
标志
-a     列出所有当前资源极限。
-c     以 512 字节块为单位，指定核心转储的大小。
-d     以 K 字节为单位指定数据区域的大小。
-f     使用 Limit 参数时设定文件大小极限（以块计），或者在未指定参数时报告文件大小极限。缺省值为 -f 标志。
-H     指定设置某个给定资源的硬极限。如果用户拥有 root 用户权限，可以增大硬极限。任何用户均可减少硬极限。
-m     以 K 字节为单位指定物理存储器的大小。
-n     指定一个进程可以拥有的文件描述符的数量的极限。
-s     以 K 字节为单位指定堆栈的大小。
-S     指定为给定的资源设置软极限。软极限可增大到硬极限的值。如果 -H 和 -S 标志均未指定，极限适用于以上二者。
-t     指定每个进程所使用的秒数。

退出状态
返回以下退出值：

0     成功完成。

>0     拒绝对更高的极限的请求，或发生错误。

示例

要将文件大小极限设置为 51,200 字节，输入：

{% highlight bash %}
ulimit -f 100
{% endhighlight %}