---
title: 如何在kernel中获取当前进程的信息
date: 2020-08-18 16:37:06
tags:  [kernel,pid]
categories: [kernel,pid]
---
&emsp;&emsp;在kernel中有时候可能需要根据当前进行的某些信息进行逻辑处理，本文就是介绍如果在kernel中如果获取当前进程的信息。
&emsp;&emsp;在kernel中关于进程信息的一个重要的结构体就是task_struct，定义如下:
```c
struct task_struct {
//这个是进程的运行时状态，-1代表不可运行，0代表可运行，>0代表已停止。
 volatile long state;
 /*
flags是进程当前的状态标志，具体的如：
0x00000002表示进程正在被创建；
0x00000004表示进程正准备退出；
0x00000040 表示此进程被fork出，但是并没有执行exec；
0x00000400表示此进程由于其他进程发送相关信号而被杀死 。
*/
 unsigned int flags;
//表示此进程的运行优先级
 unsigned int rt_priority;
//这里出现了list_head结构体，详情请参考
 struct list_head tasks;
//这里出现了mm_struct 结构体，该结构体记录了进程内存使用的相关情况，详情请参考
 struct mm_struct *mm;
/* 接下来是进程的一些状态参数*/
 int exit_state;
 int exit_code, exit_signal;
//这个是进程号
 pid_t pid;
//这个是进程组号
 pid_t tgid;
//real_parent是该进程的”亲生父亲“，不管其是否被“寄养”。
 struct task_struct *real_parent;
//parent是该进程现在的父进程，有可能是”继父“
 struct task_struct *parent;
//这里children指的是该进程孩子的链表，可以得到所有孩子的进程描述符，但是需使用list_for_each和list_entry，list_entry其实直接使用了container_of，详情请参考
 struct list_head children;
//同理，sibling该进程兄弟的链表，也就是其父亲的所有孩子的链表。用法与children相似。
 struct list_head sibling;
//这个是主线程的进程描述符，也许你会奇怪，为什么线程用进程描述符表示，因为linux并没有单独实现线程的相关结构体，只是用一个进程来代替线程，然后对其做一些特殊的处理。
 struct task_struct *group_leader;
//这个是该进程所有线程的链表。
 struct list_head thread_group;
//顾名思义，这个是该进程使用cpu时间的信息，utime是在用户态下执行的时间，stime是在内核态下执行的时间。
 cputime_t utime, stime;
//下面的是启动的时间，只是时间基准不一样。
 struct timespec start_time; 
 struct timespec real_start_time;
//comm是保存该进程名字的字符数组，长度最长为15，因为TASK_COMM_LEN为16。
 char comm[TASK_COMM_LEN];
/* 文件系统信息计数*/
 int link_count, total_link_count;
/*该进程在特定CPU下的状态*/
 struct thread_struct thread;
/* 文件系统相关信息结构体*/
 struct fs_struct *fs;
/* 打开的文件相关信息结构体*/
 struct files_struct *files;
 /* 信号相关信息的句柄*/
 struct signal_struct *signal;
 struct sighand_struct *sighand;
 /*这些是松弛时间值，用来规定select()和poll()的超时时间，单位是纳秒nanoseconds */
 unsigned long timer_slack_ns;
 unsigned long default_timer_slack_ns;
}；
```
<!--more-->
如果需要获取其中的某些信息，获取对应的`task_struct`中的字段就可以了。在实际应用中，经常需要处理的就是获取当前进程的信息，这时候可以借助内核宏`current`，通过该宏可以获取到当前进程的`task_struct`。下面是针对`task_struct`的应用示例：
```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/sched.h> //task_pid_nr

/* This function is called when the module is loaded. */
int simple_init(void)
{
       printk(KERN_INFO "Loading Module\n");
       printk("The process id is %d\n", (int) task_pid_nr(current));
       printk("The process vid is %d\n", (int) task_pid_vnr(current));
       printk("The process name is %s\n", current->comm);
       printk("The process tty is %d\n", current->signal->tty);
       printk("The process group is %d\n", (int) task_tgid_nr(current));
       printk("\n\n");
   //return -1; //debug mode of working
   return 0;
}

/* This function is called when the module is removed. */
void simple_exit(void) {
    printk(KERN_INFO "Removing Module\n");
}

/* Macros for registering module entry and exit points. */
module_init( simple_init );
module_exit( simple_exit );

MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("Simple Module");
MODULE_AUTHOR("SGG");
```
在直接读取这些字段的时候可能引起竞争，所以可以使用`get_task_comm()`等函数来进行读取。`get_task_comm()`函数的定义如下：
```c
#define get_task_comm(buf, tsk) ({            \
   BUILD_BUG_ON(sizeof(buf) != TASK_COMM_LEN); \
   __get_task_comm(buf, sizeof(buf), tsk);     \
 })

 char *__get_task_comm(char *buf, size_t buf_size, struct task_struct *tsk)
 {
   task_lock(tsk);
   strncpy(buf, tsk->comm, buf_size);
   task_unlock(tsk);
   return buf;
 }
```
