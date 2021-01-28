---
title: c++ 内存管理
date: 2020-12-23 21:21:07
tags: c/c++
categories: c/c++
---
本文主要记录下c/c++中使用堆内存的注意事项。

#### 堆内存使用完后，指针设置为nullptr
在使用完new(malloc)的内存后，应该习惯在释放完该内存后，将指针设置为`nullptr`。
```c++
int *p = new int;
*p = 10;
delete p;
p = nullptr;
```

#### 应该关注new的内存是否分配成功
在申请内存过程中，有极小概率的情况申请分配内存失败，应该处理出现申请分配内存失败的情况。有一种简便的方法用于申请内存时，如果申请失败，那么返回一个空指针，这各方法时通过在new与类型之间加入常量`std::nothrow`实现的。
```c++
int *p = new (std::nothrow) int;
if (!p) {
    // 错误处理
    std::cout << "No enough memory!";
}
```