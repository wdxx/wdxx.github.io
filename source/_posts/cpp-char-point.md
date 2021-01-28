---
title: c++对char类型指针的处理
date: 2020-12-24 10:20:23
tags: c/c++
categories: c/c++
---
本文主要记录c++对char类型指针的处理。
```c++
#include <iostream>
int main() {
    char Name[]{"Lucy"};
    std::cout << Name << '\n';

    const char *mName{"Lucy"};
    std::cout << mName << '\n';

    return 0;
}
```
<!--more-->
上面程序的输出是一样的，但是编译器对`Name`与`mName`的处理是有区别的。`Name`是普通的字符数组，是一个局部变量，生命周期跟普通的变量一样。`mName`的生命周期存在于整个文件，跟`static`变量类似。看下面的例子：
```c++
const char* getName() {
    return "Lucy";
}
```
上面函数并不会导致返回未知，因为返回的变量类似于`static`变量。再看下面的例子：
```c++
char* getName() {
    char Name[]{"Lucy"};
    return Name;
}
```
上面函数的返回时未知的，它返回的是一个野指针，因为`Name`是局部变量，在函数返回时就被释放了，所以返回未知。
### c++对cosnt char* 类型的优化
如果两个`const char *`指向相同内容的字符串，那么编译器可能会优化为两个指针指向同一个位置，看下面的例子。
```c++
#include <iostream>
int main(int argc, char const *argv[])
{
    char *NameA{"Name"};
    const char *NameB{"Name"};

    NameA[1] = 'A';
    std::cout << NameA << '\n';
    std::cout << NameB <<  '\n';

    std::cout << (long)NameA << '\n';
    std::cout << (long)NameB << '\n';
    return 0;
}
```
执行后可以看到`NameA`与`NameB`的值是相同的，也就是他们指向同一个内存地址。