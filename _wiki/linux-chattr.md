---
layout: wiki
title: chattr
categories: linux
description: linux chattr 命令
keywords: linux, chattr
---

## Linux chattr 命令详解

### 常见命令参数

```txt
A：即Atime，告诉系统不要修改对这个文件的最后访问时间。
S：即Sync，一旦应用程序对这个文件执行了写操作，使系统立刻把修改的结果写到磁盘。
a：即Append Only，系统只允许在这个文件之后追加数据，不允许任何进程覆盖或截断这个文件。如果目录具有这个属性，系统将只允许在这个目录下建立和修改文件，而不允许删除任何文件。
b：不更新文件或目录的最后存取时间。
c：将文件或目录压缩后存放。
d：当dump程序执行时，该文件或目录不会被dump备份。
D:检查压缩文件中的错误。
i：即Immutable，系统不允许对这个文件进行任何的修改。如果目录具有这个属性，那么任何的进程只能修改目录之下的文件，不允许建立和删除文件。
s：彻底删除文件，不可恢复，因为是从磁盘上删除，然后用0填充文件所在区域。
u：当一个应用程序请求删除这个文件，系统会保留其数据块以便以后能够恢复删除这个文件，用来防止意外删除文件或目录。
t:文件系统支持尾部合并（tail-merging）。
X：可以直接访问压缩文件的内容。
```

### 常用的命令展示

```txt
chatter: 锁定文件，不能删除，不能更改
        +a: 只能给文件添加内容，但是删除不了，
            chattr +a  /etc/passwd
        -d: 不可删除
        加锁：chattr +i  /etc/passwd       文件不能删除，不能更改，不能移动
        查看加锁： lsattr /etc/passwd      文件加了一个参数 i 表示锁定
        解锁：chattr -i /home/omd/h.txt    - 表示解除
        隐藏chattr命令：
```

```sh
which chattr
mv /usr/bin/chattr  /opt/ftl/
cd /opt/ftl/ 
mv chattr h    -->更改命令，使用别名h隐藏身份
/opt/ftl/h +i /home/omd/h.txt   -->利用h 行驶chattr命令
lsattr /home/omd/h.txt    -->查看加密信息

#恢复隐藏命令
mv h /usr/bin/chattr
chattr -i /home/omd/h.txt
lsattr /home/omd/h.txt
```
