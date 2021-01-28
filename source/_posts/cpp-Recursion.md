---
title: c/c++ 递归
date: 2021-01-20 11:44:22
tags: c/c++
categories: c/c++
---
c/c++中，递归函数就是一个调用自身的函数。由于递归函数需要调用自身，所以如果函数没有退出或者终止调用的条件的话，那么将会导致不断的调用自身，栈内存将会被耗尽。看下面的例子：
```c++
#include <iostream>

void countDown(int count) {
    std::cout << "push " << cout << '\n';
    countDown(count-1);
}

int main() {
    countDown(10);
    return 0;
}
```
这个例子中，`countDown`函数没有终止调用条件，会无限制调用下去，最终导致栈溢出。要编写一个正常的递归函数，那么该函数必须要有终止调用的条件。
<!-- more -->

#### 递归终止条件
`递归终止条件`是一个能够终止递归函数调用自身的条件。通常情况下，`递归终止条件`通过if语句调用。将上面的例子，改写成如下的例子：
```c++
#include <iostream>

void countDown(int count) {
    std::cout << "push " << cout << '\n';
    if (count > 0) {
        countDown(count-1);
    }
    std::cout << "pop " << cout << '\n';
}

int main() {
    countDown(10);
    return 0;
}
```
上面的代码会生成如下的输出:
```
push 5
push 4
push 3
push 2
push 1
push 0
pop 0
pop 1
pop 2
pop 3
pop 4
pop 5
```
可以看到，在递归函数之前的代码是顺序运行的，而在递归函数之后的代码是倒序运行的。
#### 递归应用
下面介绍几个递归应用的例子。  
1. 计算正数的阶乘。  
n! = n\*(n-1)\*...\*2*1。  
递归终止条件: 0! = 1
```c++
#include <iostream>

int factorial(int num) {
    if (num <= 0)
        return 1;
    return num*factorial(num-1);
}

int main() {
    int sample = factorial(3);
    std::cout << "3! = " << sample << '\n';
    return 0;
}
```

2. 计算一个正整数的各位的和。(例如：964: 9+6+4=19)  
递归终止条件: n<10,return n%10
```c++
#include <iostream>

int count(int n) {
    if (n < 10)
        return n;
    return count(n/10) + n%10;
}

int main() {
    int sample[]{2345,55677};
    for (auto s : sample) {
        std::cout << s << "==========> " << count(s) << '\n';
    }
    return 0;
}
```
下面是需要保存结果的情况:
```c++
#include <iostream>
#include <vector>

static std::vector<int> record;
int count(int n) {
    int r = 0;
    if (n < 10) {
        record.clear();
        record.push_back(n);
        return n;
    }
    r = count(n/10);
    int y = n%10;
    record.push_back(y);
    return r+y;
}

int main() {
    int sample[]{2345,55677};
    std::cout << "test ....\n";
    for (auto s : sample) {
        count(s);
        for (auto r : record)
            std::cout << s << "==========> " << r << '\n';
    }
    return 0;
}
```
3. 以二进制形式显示一个正数。（如：9： 1001）
```c++
#include <iostream>
#include <vector>

static std::vector<int> record;
int binary(int x) {
    if (x <=1 ) {
        record.clear();
        record.push_back(x);
        return x;
    }   
    int bin = binary(x/2);
    record.push_back(x%2); // 放在递归之后，调用是倒序的
    return bin;
}

int main() {
    int test[]{9,10,11};
    for (auto t : test) {
        binary(t);
        for (auto r : record) {
            std::cout << r;
        }
        std::cout << '\n';
    }
    return 0;
}
```
如何处理负数呢？可以将binary函数的形参类型改为`unsinged int`类型，这样最高位就表示符号位了。  

#### 总结
应该尽量少用递归，而应该看是否能够通过循环来解决问题。