---
title: android simpleperf工具使用入门
date: 2020-05-15 16:12:06
tags: android
---
&emsp;&emsp;Simpleperf 是一个通用的命令行 CPU 性能剖析工具，在Android NDK中已经自带了。需要注意的是，如果自带的simpleperf不能使用，就需要在适配当前Android设备的源码中编译了。simpleperf的源码在AOSP的system/extras/simpleperf下，其中doc目录详细介绍了如何使用该工具。
<!--more-->

## 在源码中编译simpleperf
&emsp;&emsp;可以通过如下命令在Andorid设备的源码中编译simpleperf:
```bash
source build/envsetup.sh
lunch aosp_arm64-eng
make simpleperf -j32
```
最后将会生成out/target/product/xxx/system/xbin/simpleperf，这就是需要push到Android设备的simpleperf。
### push simpleperf
&emsp;&emsp;在push之前需要需要保证Android设备具有root权限，一般通过adb root选项进行提权。然后通过如下命令将simpleperf push到设备。
```bash
adb root
adb shell mkdir /data/bin
adb push <编译出来的simpleperf> /data/bin/simpleperf
adb shell setenforce 0
```
进行了上述步骤后，就可以在设备中使用simpleperf了。
```bash
adb shell /data/bin/simpleperf --version
# simpleperf I command.cpp:131] Simpleperf version 1.c65e6065befd
#
```

## simpleperf使用介绍
&emsp;&emsp;simpleperf的record命令用于转储已需要分析的进程的样本。 每个样本可以包含诸如样本生成时间，自上一个样本以来的事件数，线程的程序计数器，线程的调用链之类的信息。后续的分析步骤都是基于这个样本信息来分析的。
### 采集样本数据
首先确认需要分析的进程。比如分析当前cpu占用最高的进程。
```bash
top -m 5 # 得到执行的进程名字
## 根据pidof得到需要分析的进程id号
pidof <进程名>
```
得到进程id后就可以进行数据采集了。如果使用simpleperf的recod命令请参[考官方文档](https://android.googlesource.com/platform/system/extras/+/refs/heads/master/simpleperf/doc/executable_commands_reference.md)中的`The record command`小节。
如下的命令记录进行号为11961的record：
```bash
/data/bin/simpleperf record -p 11961 -o /sdcard/perf_11961.data --duration 360
```
结果保存在/sdcard/perf_11961.data,记录持续时间为360秒。这里只记录了默认的`cpu-cycles`event，如果需要记录更加多的event需要指定-e参数。通过`simpleperf list`命令查看支持哪些events。

### 查看特定进程调用的动态库
通过report命令可以查看这种report数据：
```bash
./simpleperf report --sort dso -i /sdcard/perf_11961.data
simpleperf W dso.cpp:361] failed to read symbols from [vdso]: File not found
Cmdline: /data/bin/simpleperf record -p 11961 -o /sdcard/perf_11961.data --duration 360
Arch: arm64
Event: cpu-cycles (type 0, config 0)
Samples: 649
Event count: 136430910

Overhead  Shared Object
57.91%    [kernel.kallsyms]
20.03%    /system/lib64/libart.so
9.43%     /system/lib64/libandroidfw.so
5.05%     /system/framework/arm64/boot-framework.oat
3.58%     /system/lib64/libc.so
1.67%     /system/framework/arm64/boot-core-oj.oat
0.89%     /system/lib64/libbinder.so
0.39%     /system/lib64/libc++.so
0.38%     /system/lib64/libopenjdk.so
0.27%     /system/lib64/libz.so
0.15%     /system/lib64/libandroid_runtime.so
0.13%     [vdso]
0.07%     /system/framework/arm64/boot-core-libart.oat
0.05%     /system/lib64/libutils.so
```

### 查看函数调用关系
simpleperf report还可以导出函数调用关系,--symfs可以指定符号表的目录。
```bash
/data/bin/simpleperf report -g  -i /sdcard/perf_19357.data --symfs . > /sdcard/g_19357.dat
```

#### 参考文档
1. <https://blog.csdn.net/zhuyong006/article/details/103112571>
2. <https://developer.android.com/ndk/guides/simpleperf>