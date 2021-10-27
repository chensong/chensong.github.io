---
layout: wiki
title: dd
categories: linux
description: linux dd 命令
keywords: linux, dd
---

## Linux dd 命令详解

### 常见命令参数

```txt
if=文件名：输入文件名，默认为标准输入。即指定源文件。
of=文件名：输出文件名，默认为标准输出。即指定目的文件。
ibs=bytes：一次读入bytes个字节，即指定一个块大小为bytes个字节。
obs=bytes：一次输出bytes个字节，即指定一个块大小为bytes个字节。
bs=bytes：同时设置读入/输出的块大小为bytes个字节。
cbs=bytes：一次转换bytes个字节，即指定转换缓冲区大小。
skip=blocks：从输入文件开头跳过blocks个块后再开始复制。
seek=blocks：从输出文件开头跳过blocks个块后再开始复制。
count=blocks：仅拷贝blocks个块，块大小等于ibs指定的字节数。
conv=<关键字>，关键字可以有以下11种：
    conversion：用指定的参数转换文件。
    ascii：转换ebcdic为ascii
    ebcdic：转换ascii为ebcdic
    ibm：转换ascii为alternate ebcdic
    block：把每一行转换为长度为cbs，不足部分用空格填充
    unblock：使每一行的长度都为cbs，不足部分用空格填充
    lcase：把大写字符转换为小写字符
    ucase：把小写字符转换为大写字符
    swap：交换输入的每对字节
    noerror：出错时不停止
    notrunc：不截短输出文件
    sync：将每个输入块填充到ibs个字节，不足部分用空（NUL）字符补齐。
--help：显示帮助信息
--version：显示版本信息
```

### 常用的命令展示

linux 快速生成大文件

dd命令可以轻易实现创建指定大小的文件，如

```sh
dd if=/dev/zero of=test bs=1M count=1000
```

会生成一个 1000M 的 test 文件，文件内容为全 0（因从/dev/zero中读取，/dev/zero 为 0 源）。

但是这样为实际写入硬盘，文件产生速度取决于硬盘读写速度，如果欲产生超大文件，速度很慢。
在某种场景下，我们只想让文件系统认为存在一个超大文件在此，但是并不实际写入硬盘。

则可以

```sh
dd if=/dev/zero of=test bs=1M count=0 seek=100000
```

此时创建的文件在文件系统中的显示大小为 100000MB，但是并不实际占用 block，因此创建速度与内存速度相当。

seek 的作用是跳过输出文件中指定大小的部分，这就达到了创建大文件，但是并不实际写入的目的。

当然，因为不实际写入硬盘，所以你在容量只有 10G 的硬盘上创建 100G 的此类文件都是可以的。
