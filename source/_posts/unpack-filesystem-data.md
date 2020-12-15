---
title: 如何解压android系统镜像
date: 2020-05-18 16:35:01
tags: android filesystem
---
&emsp;&emsp;在android的开发过程中，经常需要用到将andorid system.img 或者 vendor.img等ext文件系统格式的镜像给解压出来分析，如果当前Linux系统有root权限，那么可以直接mount 镜像来进行提取，但是一般开发服务器是不提供root权限的，所以本文讨论如何在没有root权限的情况下对文件系统镜像进行提取。以AndroidQ super.img为例进行说明。
<!--more-->

## super.img
&emsp;&emsp;super.img是AndroidQ引入的，用于处理动态分区特性。例如将system、vendor
、product分区定义为动态分区，那么编译完成时这三个分区的镜像就会合并成一个super.img镜像，当然也能将super.img拆分成system.img、vendor.img、product.img镜像。

### 如何拆分super.img
&emsp;&emsp;Android Q 自带了lpunpack工具来对super.img进行解压。下面是lpunpack的使用介绍。
```bash
./lpunpack - command-line tool for extracting partition images from super

Usage:
  ./lpunpack [options...] SUPER_IMAGE [OUTPUT_DIR]

Options:
  -p, --partition=NAME     Extract the named partition. This can
                           be specified multiple times.
  -S, --slot=NUM           Slot number (default is 0).
```
&emsp;&emsp;例如：`lpunpack super.img`默认会将所有的分区文件提取出来，可以通过-p选择提取指定的分区镜像。需要注意的是，lpunpack工具aosp代码默认不会编译，可以通过将lpunpack加入PRODUCT_HOST_PACKAGES，然后编译，源码在aosp源码的system/extras/partition_tools目录。
&emsp;&emsp;aosp源码的system/extras/partition_tools目录下面还有几个常用工具，一起介绍下。  
lpmake：用于生成一个super.img文件，具体使用方法可以通过`lpmake -h`查看。  
lpdump：用于dump一个super.img文件中包含哪些分区等元数据。  
lpflash：用于烧录镜像到设备，`Usage: lpflash /dev/block/sdX /path/to/image/file`。

## 如何拆分system.img
&emsp;&emsp;通过上个步骤的lpunpack后，会得到system.img.如果system.img镜像的格式是sparse格式的（super.img解压后的不需要这个步骤，已经是raw格式的了），需要将其转换为raw格式，转换工具为`simg2img`，在aosp编译后的out/host/linux-x86/bin目录，转换命令如下:  
```
simg2img system.img  system.img.raw
```
system.img一般为ext文件系统，这里以ext文件系统解压作为讲解。最常用的方法就是使用e2fsprogs中的debugfs了。比如想将system.img中的内容解压到system目录，使用debugfs的命令如下:  
```bash
mkdir system
debugfs system.img  -R "rdump / system"
```
&emsp;&emsp;还有一种方法是使用nlitsme提供的extfstools工具，在github上有下载该工具的源码。但是extfstools依赖c++17与boost库，所以需要先编译gcc 6以上的版本与boost库。

### 编译gcc
&emsp;&emsp;在gcc 6以上的对c++17的特性支持比较全，关于gcc对c++版本的支持查看[这里](https://zh.cppreference.com/w/cpp/compiler_support)。所以选取了gcc-6.2.0，下载地址点[这里](https://ftp.gnu.org/gnu/gcc/gcc-6.2.0/gcc-6.2.0.tar.bz2)。  
1. 配置gcc  
```bash
mkdir build install
cd build
../$(gcc-source-scode-dir)/configure --prefix=<install目录的绝对路径> --enable-languages=c,c++
```
执行上面步骤的时候可能会出现如下报错:  
```
checking for objdir... .libs
checking for the correct version of gmp.h... no
configure: error: Building GCC requires GMP 4.2+, MPFR 2.4.0+ and MPC 0.8.0+.
Try the --with-gmp, --with-mpfr and/or --with-mpc options to specify
their locations.  Source code for these libraries can be found at
their respective hosting sites as well as at
ftp://gcc.gnu.org/pub/gcc/infrastructure/.  See also
http://gcc.gnu.org/install/prerequisites.html for additional info.  If
you obtained GMP, MPFR and/or MPC from a vendor distribution package,
make sure that you have installed both the libraries and the header
files.  They may be located in separate packages.
```
解决方法是到gcc源码目录执行`./contrib/download_prerequisites`，然后重新配置gcc。
2. 编译与安装
```bash
make -j32
make install
```

### 编译boost
&emsp;&emsp;boost的新版本下载地址请点击[这里](https://dl.bintray.com/boostorg/release/1.73.0/source/boost_1_73_0.tar.bz2)。关于新手怎么使用boost可以查看[官方文档](https://www.boost.org/doc/libs/1_73_0/more/getting_started/unix-variants.html)。

1. 配置boost  
```bash
cd <bost_source_dir>
mkdir install
./bootstrap.sh --with-toolset=gcc --prefix=<Absolute address for bost_source_dir>/install --with-libraries=all
```
2. 编译与安装  
```bash
./b2 install
```

### 编译extfstools
&emsp;&emsp;extfstools可以通过git命令从github[地址](https://github.com/nlitsme/extfstools.git)下载。由于我们的gcc与boost是安装的自定义目录，没有安装到系统目录，所以需要修改extfstools/Makefile来指定环境变量。在`CXXFLAGS`中加入`-I <Absolute address for bost_source_dir>/install/include`,在`LDFLAGS`中添加`-Wl,-rpath,'$$ORIGIN/lib64'`。这里修改链接选项的目的是使用我们新编译的gcc库。  
1. 拷贝gcc标准库  
```bash
cd <extfstools source dir>
mkdir lib64
cd lib64
cp <gcc install path>/libgcc_s.so.1 libgcc_s.so.1
cp <gcc install path>/libstdc++.so.6.0.22 libstdc++.so.6.0.22
ln -s libstdc++.so.6.0.22 libstdc++.so.6
cd -
```

2. 编译  
```bash
CXX=<gcc install path>/g++ make clean
CXX=<gcc install path>/g++ make -j32
```
编译完成后，执行`./ext2rd --help`会打印如果使用说明：  
![](/images/ext2rd_help.png)
解压system.img:  
```bash
./ext2rd system.img  ./:system
```
解压效果如下图:  
![](/images/system_filelist.png)
