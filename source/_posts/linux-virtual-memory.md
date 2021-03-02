---
title: linux虚拟内存
date: 2021-02-27 14:42:29
tags: linux
categories: linux
---
linux的虚拟内存空间是通过CPU中的MMU来实现的。要使得MMU工作，操作系统必须填充页表的数据结构，剩下的部分交给cpu来处理。
