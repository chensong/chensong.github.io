---
layout: post
title: Linux too many files open
categories: [Blog, linux]
description: Linux too many files open
keywords: linux
---

在Linux下，我们使用ulimit -n 命令可以看到单个进程能够打开的最大文件句柄数量(socket连接也算在里面)。系统默认值1024。

对于一般的应用来说(象Apache、系统进程)1024完全足够使用。但是如何象squid、mysql、java等单进程处理大量请求的应用来说就有点捉襟见肘了。如果单个进程打开的文件句柄数量超过了系统定义的值，就会提到“too many files open”的错误提示。如何知道当前进程打开了多少个文件句柄呢？下面一段小脚本可以帮你查看：

```sh
lsof -n |awk '{print $2}'|sort|uniq -c |sort -nr|more   
```

在系统访问高峰时间以root用户执行上面的脚本，可能出现的结果如下：

```sh
# lsof -n|awk '{print $2}'|sort|uniq -c |sort -nr|more   
    131 24204  
     57 24244  
     57 24231  
     56 24264  
```

其中第一行是打开的文件句柄数量，第二行是进程号。得到进程号后，我们可以通过ps命令得到进程的详细内容。

```sh
ps -aef|grep 24204  
mysql    24204 24162 99 16:15 ?        00:24:25 /usr/sbin/mysqld  
```
哦，原来是mysql进程打开最多文件句柄数量。但是他目前只打开了131个文件句柄数量，远远底于系统默认值1024。

但是如果系统并发特别大，尤其是squid服务器，很有可能会超过1024。这时候就必须要调整系统参数，以适应应用变化。Linux有硬性限制和软性限制。可以通过ulimit来设定这两个参数。方法如下，以root用户运行以下命令：

```sh
ulimit -HSn 4096  
```
以上命令中，H指定了硬性大小，S指定了软性大小，n表示设定单个进程最大的打开文件句柄数量。个人觉得最好不要超过4096，毕竟打开的文件句柄数越多响应时间肯定会越慢。设定句柄数量后，系统重启后，又会恢复默认值。如果想永久保存下来，可以修改.bash_profile文件，可以修改 /etc/profile 把上面命令加到最后。(findsun提出的办法比较合理)

=================================================================================

Too many open files经常在使用linux的时候出现，大多数情况是您的程序没有正常关闭一些资源引起的，所以出现这种情况，请检查io读写，socket通讯等是否正常关闭。 

如果检查程序没有问题，那就有可能是linux默认的open files值太小，不能满足当前程序默认值的要求，比如数据库连接池的个数，tomcat请求连接的个数等。。。 

查看当前系统open files的默认值，可执行：


```sh
[root@pororo script]# ulimit -a   
core file size           (blocks, -c) 0  
data seg size            (kbytes, -d) unlimited   
scheduling priority              (-e) 0  
file size                (blocks, -f) unlimited   
pending signals                  (-i) 128161  
max locked memory        (kbytes, -l) 32  
max memory size          (kbytes, -m) unlimited   
open files                       (-n) 800000  
pipe size             (512 bytes, -p) 8  
POSIX message queues      (bytes, -q) 819200  
real-time priority               (-r) 0  
stack size               (kbytes, -s) 10240  
cpu time                (seconds, -t) unlimited   
max user processes               (-u) 128161  
virtual memory           (kbytes, -v) unlimited   
file locks                       (-x) unlimited  
```

如果发现open files项比较小，可以按如下方式更改： 

1. 检查/proc/sys/fs/file-max文件来确认最大打开文件数已经被正确设置。

```sh
# cat /proc/sys/fs/file-max  
```

如果设置值太小，修改文件/etc/sysctl.conf的变量到合适的值。这样会在每次重启之后生效。 如果设置值够大，跳过这一步。


```sh
# echo 2048 > /proc/sys/fs/file-max  
```

编辑文件/etc/sysctl.conf，插入下行：


```sh
fs.file-max = 8192  
```

2. 在/etc/security/limits.conf文件中设置最大打开文件数， 下面是一行提示：


```sh
#<domain>   <type>   <item>   <value>  
```

添加如下这行:


```sh
* - nofile 8192  
```

这行设置了每个用户的默认打开文件数为2048。 注意"nofile"项有两个可能的限制措施。就是<type>项下的hard和soft。 要使修改过得最大打开文件数生效，必须对这两种限制进行设定。 如果使用"-"字符设定<type>, 则hard和soft设定会同时被设定。 

硬限制表明soft限制中所能设定的最大值。 soft限制指的是当前系统生效的设置值。 hard限制值可以被普通用户降低。但是不能增加。 soft限制不能设置的比hard限制更高。 只有root用户才能够增加hard限制值。 

当增加文件限制描述，可以简单的把当前值双倍。 例子如下， 如果你要提高默认值1024， 最好提高到2048， 如果还要继续增加， 就需要设置成4096。 

最后用ulimit -a再次查看，open files的值，没什么问题的话，就已经改过来了。
