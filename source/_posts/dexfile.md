---
title: dexfile 文件格式
date: 2020-07-24 17:30:31
tags: android
categories: android
---
Android APK中，最终java文件会被编译为classes.dex文件。如果要分析APK就得了解dex文件的格式。
dex文件的格式在AOSP源码的`dalvik/libdex/DexFile.h`有定义。