---
layout: post
title: Shell 数组
categories: [Blog, shell]
description: Linux Shell 数组
keywords: shell
---

## 1. Shell 数组

### 1.1 定义数组变量

```sh
array_name=(value1 value2 ... valueN)
```

```sh
# 样例 1
A=(aa bb cc dd)
# 样例 2
A[0]=aa
A[1]=bb
A[2]=cc
A[3]=dd
```

### 1.2 读取数组的元素

```sh
${array_name[index]}
# 获取第 1 个
echo ${A[0]}
```

### 1.3 读取全部元素

```sh
${array_name[*]}
${array_name[@]}
```

```sh
# 区别
A=(aa bb cc dd)

echo ${A[*]}
for i in "${A[*]}"; do
    echo $i
done
# stdout
aa bb cc dd
aa bb cc dd

echo ${A[@]}
for i in "${A[@]}"; do
    echo $i
done
# stdout
aa bb cc dd
aa
bb
cc
dd
```

### 1.4 删除元素

```sh
# 清空全部
unset array_name
unset array_name[@]
unset array_name[*]
# 删除指定元素
unset array_name[index]
```

### 1.5 获取数组元素个数和 index

```sh
# 元素个数
${#array_name[@]}
# 元素 index
${!array_name[@]}
```

```sh
A=(aa bb cc dd)
echo count=${#A[@]}
echo index=${!A[@]}
# stdout
count=4
index=0 1 2 3

B=(aa bb cc dd)
unset B[1]
echo count=${#B[@]}
echo index=${!B[@]}
# stdout
count=3
index=0 2 3

C[0]=aa
C[1]=bb
C[4]=dd
echo count=${#C[@]}
echo index=${!C[@]}
# stdout
count=3
index=0 1 3
```

遍历数组推荐用法

```sh
A=(aa bb cc dd)
for i in ${!A[@]}; do
    echo item[$i]=${A[$i]}
done
# stdout
item[0]=aa
item[1]=bb
item[2]=cc
item[3]=dd
```

### 1.6 获取元素的字符串长度

```sh
${#array_name[index]}
```

```sh
A=(aa bb cc dd)
echo ${A[0]}
# console 输出
2
```

### 1.7 数组合并

```sh
A=(aa bb)
B=(qq ${A[@]} ww)
echo ${B[@]}
# stdout
qq aa bb ww

A=(aa bb)
B=(cc dd)
C=(${A[@]} ${B[@]})
echo ${C[@]}
# stdout
aa bb cc dd
```
