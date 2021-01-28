---
title: cpp_ofstream_ifstream
date: 2020-08-20 16:23:36
tags: c/c++
categories: c/c++
---
ofstream是从内存到硬盘，ifstream是从硬盘到内存，其实所谓的流缓冲就是内存空间。
在C++中，有一个stream这个类，所有的I/O都以这个“流”类为基础的，包括我们要认识的文件I/O。
stream这个类有两个重要的运算符：
1、插入器(<<)
向流输出数据。比如说系统有一个默认的标准输出流(cout)，一般情况下就是指的显示器，所以，cout<<"Write Stdout"<<'\n';就表示把字符串"Write Stdout"和换行字符('\n')输出到标准输出流。
2、析取器(>>)
从流中输入数据。比如说系统有一个默认的标准输入流(cin)，一般情况下就是指的键盘，所以，cin>>x;就表示从标准输入流中读取一个指定类型的数据。
在C++中，对文件的操作是通过stream的子类fstream(file stream)来实现的，所以，要用这种方式操作文件，就必须加入头文件fstream.h。

特别提出的是，fstream有两个子类：ifstream(input file stream)和ofstream(outpu file stream)，ifstream默认以输入方式打开文件，ofstream默认以输出方式打开文件。

