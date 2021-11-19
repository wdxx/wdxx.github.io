### GCC

在Linux开发中，一般使用gcc编译器进行编译，在安装完gcc编译器后，安装工具链的主机已经带有了所有的编译所需要的工具以及标准库文件。对于交叉编译来说，编译环境与非交叉编译环境在工具使用上对开发这基本时透明的，因为工具已经带有了完整的库。由于gcc编译工具链是针对特定平台来进行定制的（如：arm-linux-gcc, aarch64-linux-gcc, mips-linux-gcc ...），所以每个gcc工具带有的库都不一样，这里以arm gcc编译工具链（gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf）说明。

这个工具是`linaro`提供的针对`arm`在`linux`上的交叉编译工具链。工具链包含如下目录：

```
├── arm-linux-gnueabihf
├── bin
├── gcc-linaro-7.5.0-2019.12-linux-manifest.txt
├── include
├── lib
├── libexec
└── share
```

- bin  bin目录包含编译时用到的二进制工具，如`arm-linux-gnueabihf-gcc`，这个是在host（编译器运行的计算机）上运行的。
- include 顶层的include目录同样是给host用的，所以里面的东西也是host相关架构的。
- lib 目录同样是给host使用的。
- libexec host上运行交叉编译器，host运行时需要用到的库。
- share 该工具链附带的一些文件，如man手册，info信息

这里剩下一个`arm-linux-gnueabihf`目录，这个目录包含了交叉编译时，`target`机器（你编译的程序需要运行的机器）上的程序依赖的库文件。

#### arm-linux-gnueabihf

`arm-linux-gnueabihf`叫做`triple`，这里目录里面的内容如下：

```
├── bin
├── include
├── lib
└── libc
```

- bin  bin目录里面是`binutils`，跟外层目录里面的`bin`内容一致（这个目录的可能不全）。
- lib  lib目录包含`target`上的`标准c++库`
- include include目录包含了`标准库c++头文件`
- libc  libc目录是全集，包含了`target`上所有的`标准c库`，`c运行时库`，`标准c++库`，`头文件`

这个目录下面的标准库，是在交叉编译链接时需要用到的。

### Clang

clang编译器相比gcc来说要简洁很多，同时提供的功能也多。`clang`的目录结构如下：

```
.
├── bin
├── include
├── lib
├── libexec
└── share
```

这几个目录跟gcc中相同名字的目录的功能基本都一样，最主要的区别是没有了`$triple`目录（arm-linux-gnueabihf）。为什么没有这个目录的主要原因就是`clang 可以在一次编译支持多种不同架构的机器码的生成，但是标准库这些由编译器编译出来的库是需要每个架构一份的，如果为所有架构都保存一份到clang编译器里面，那么就会导致太臃肿了，所以clang的编译器是没有带有标准库相关的链接库的`。在使用clang进行编译的时候如果编译器没有自动找到这些标准库，那么需要在命令行手动指定这些标准库的位置。一般在不是交叉编译的条件下，clang会自动查找当前host环境的标准库，所以一般都能找到（如：/usr/lib/gcc/x86_64-linux-gnu/）。解决这个问题的方法为指定`--gcc-toolchain``=<arg>`或者`--sysroot``=<arg>``, ``--sysroot`` <arg>`参数。具体参数含义参考[官方Documeng](https://clang.llvm.org/docs/ClangCommandLineReference.html)。

参考：

1. https://clang.llvm.org/docs/CrossCompilation.html
2. https://stackoverflow.com/questions/67461415/clang-linker-does-not-recognise-linux-libraries
