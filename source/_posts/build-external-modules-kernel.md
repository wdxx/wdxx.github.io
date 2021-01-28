---
title: 编译kernel外部模块
date: 2021-01-25 15:08:06
tags: [编译,kernel]
categories: [编译,kernel]
---
本文翻译自kernel[官方文档](https://www.kernel.org/doc/html/v5.4/kbuild/modules.html),介绍怎么编译kernel源码目录之外的代码模块。  
#### 1 介绍
Linux kernel使用`kbuild编译系统`来编译。kernel模块编译需要保持kbuild架构兼容，并且选择正确的gcc编译flag。kernel编译系统提供了在源码目录内编译以及在源码目录外编译的功能。这两种编译方式是类似的，所有的模块在最初开发阶段都是在源码外编译的。  
本文所涵盖内容的目标是那些对在kernel源码目录之外编译模块感兴趣的开发者。源码目录外的模块的开发者应该提供一个makefile,该makefile隐藏了很多复杂细节，编译时只需要执行make命令就可以编译该模块。这个很容易实现，在第3小节会有一个详细的示例。
<!-- more -->

#### 2 怎么编译外部模块
为了编译外部模块，必要条件是需要有一个已经编译过的kernel代码。同时kernel config中需要使能模块加载（CONFIG_MODULE）。如果你使用的是发行版内核，发行版将提供内核的软件包。  
另外一种方法是使用`make modules_prepare`命令。这个命令能够保证编译外部模块所需要的依赖都已经被编译出来。`modules_prepare`这个目标就是抓门用于为编译外部模块而准备的。  
注意：即使设置了CONFIG_MODVERSIONS，“ modules_prepare”也不会构建Module.symvers。 因此，需要执行完整的内核编译以使模块版本控制工作。  

##### 2.1 编译命令
下面的编译命令用于编译外部模块:
```bash
$ make -C <path_to_kernel_src> M=$PWD
```
`Kbuild编译系统`通过编译命令中的`M=<dir>`来确定当前需要编译哪个模块。  
如果要针对当前正在运行的内核进行编译使用如下命令：
```bash
$ make -C /lib/modules/`uname -r`/build M=$PWD
```
要安装刚刚构建的模块，请将目标“ modules_install”添加到命令中：
```bash
$ make -C /lib/modules/`uname -r`/build M=$PWD modules_install
```

##### 2.2 编译选项
($KDIR表示内核源码路径。)  
make -C $KDIR M=$PWD  
###### -C $KDIR
kernel源码所在目录。make -C执行时，实际上会先到$KDIR目录执行，然后再返回。
###### M=$PWD
通知kbuild一个外部模块即将编译。“M”的值是外部模块（kbuild文件）所在目录的绝对路径。

##### 2.3 编译目标
在构建外部模块时，仅“make”目标的子集可用。也就是说不是所有的kernel编译命令都可以在外部模块目录下执行。  
make -C $KDIR M=$PWD [target]  
默认情况下会编译当前目录的模块，所以不需要指定任何目标。所有的编译输出都将输出在$PWD目录。但是执行时，内核源码不能被更新，这是外部模块成功编译的前提。  

###### modules
`modules`是编译外部模块时的默认目标。即使没有指定任何目标，也会执行这个目标的编译动作。  
###### modules_install
该目标用于安装外部模块。模块默认安装路径为/lib/modules/<kernel_release>/extra/,但是也可以通过INSTALL_MOD_PATH来指定。

###### clean
清除模块目录中生成的所有文件。

###### help
列举所有支持的目标。 


##### 2.4 编译单独的文件
内核中可以通过命令单独编译组成一个模块的单独文件。这个对整个kernel源码还是外部模块都是生效的。看下面的例子：（foo.ko由bar.o、baz.o组成）:
```
make -C $KDIR M=$PWD bar.lst
make -C $KDIR M=$PWD baz.o
make -C $KDIR M=$PWD foo.ko
make -C $KDIR M=$PWD ./
```



