---
title: printk_ratelimited 导致log打印不全
date: 2020-06-30 20:48:19
tags: [kernel,printk,调试]
categories: kernel
---
在Android调试过程中，发现启动log中出现如下log：
```
[   48.471014] printk: init: 46 output lines suppressed due to ratelimiting
[   48.617226] init: [libfs_mgr]dt_fstab: Skip disabled entry for partition vendor
[   48.631023] init: [libfs_mgr]Warning: unknown flag: wrappedkey
[   48.639762] init: DM_DEV_STATUS failed for system_ext: No such device or address
[   48.647430] init: Could not update logical partition
[   48.653314] init: DM_DEV_STATUS failed for product: No such device or address
[   48.660748] init: Could not update logical partition
```
也就是说`printk: init: 46 output lines suppressed due to ratelimiting`有部分log没有打印出来，解决办法就是修改kernel源码。修改文件位于`include/linux/ratelimit.h`。
```c
/**
  * 下面表示5秒内最多打印10条记录，可以将DEFAULT_RATELIMIT_BURST修改成50，那么5秒内就可以打印50条了。当然也不是越大越好，因为需要考虑考log对系统服务器的影响，不要引起阻塞。
 **/
#define DEFAULT_RATELIMIT_INTERVAL      (5 * HZ)
#define DEFAULT_RATELIMIT_BURST         10  // 可以修改成50
```