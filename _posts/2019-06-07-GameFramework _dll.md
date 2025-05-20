---
layout: post
title: GameFramework DLL
date: 2019-06-07 16:53:52
description: DLL相关的问题
tags: GameFramework
categories: GameFramework
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

# 开发中遇到的问题

## Assembly Definition 一直提示Unsafe

在热更分包过程中，确定程序集是至关重要的。但是在GameProto包中加入luban的一些lib，因为字符转换的脚本中有UnSafe声明的代码，导致一直识别不到

> Unsafe
> 是C#中对指针进行操作时的函数声明

### 解决方法

在playersetting中修改支持All Alow Unsafe Code

> 仍有概率项目识别不出 已经修改 可能跟unity版本有关
