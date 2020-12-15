---
title: 匿名命名空间与inline命名空间
date: 2020-12-07 14:16:40
tags: c/c++
---
c/c++中除了正常使用的命名空间外，还提供了匿名命名空间与inline命名空间，这两种命名空间用在特殊的场合。
### 匿名命名空间
匿名命名空间就是在`namespace`关键字后没有带名字标识符的命名空间，看如下例子。
<!-- more -->
```c++
#include <iostream>

namespace { // 匿名命名空间
    void doSomething() {
        std::cout << "v1\n";
    }
}

int main() {
    doSomething();
    return 0;
}
```
上面的例子会正常输出：
`v1`
官方的说法是，所有的匿名命名空间中定义的内容，都会作为其父命名空间的一部分。所以在上面例子中的`main`函数中可以调用匿名命名空间中的内容，但是在定义该命名空间的文件外`doSomething`不可见。所以命名空间的作用相同与`static`作用。

### inline命名空间
`inline命名空间`经常用于对同一函数的多个版本管理。看下面的例子：
```c++
#include <iostream>

inline namespace v1 {
    void doSomething() {
        std::cout << 'v1\n';
    }
}

namespace v2 {
    void doSomething() {
        std::cout << "v2\n";
    }
}

int main() {
    v1::doSomething(); // 调用v1命名空间的doSomething()函数
    v2::doSomething(); // 调用v2命名空间的doSomething()函数
    doSomething(); // 调用inline命名空间中的函数v1::doSomething()

    return 0;
}
```
从上面的例子中可以看到，同一文件中，不加命名空间默认调用`inline 命名空间`中的内容，如果新的版本的函数定义的普通命名空间中，加上命名空间名字调用即可。