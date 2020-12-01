---
layout: post
title: Windows 系统提交 git 文件时，设置 chmod
categories: [Blog, git]
description: Windows 系统提交 git 文件时，设置 chmod
keywords: git, chmod
---

## 在 Windows 平台上需要使用下面的命令把文件标记为可执行

```sh
git update-index --chmod=+x <your_file>
```