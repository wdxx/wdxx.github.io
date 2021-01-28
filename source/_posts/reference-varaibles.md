---
title: 引用变量
date: 2021-01-13 20:23:36
tags: c/c++
categories: c/c++
---
引用变量是c++的基本变量。用于给其它变量或对象定义别名。c++支持三种类型的引用变量：
1. 非const变量的引用
2. const变量的引用
3. 在c++11中引入的右值(r-value)的引用
<!-- more -->

#### 非const变量的引用
一个非const变量的引用通过`&`符号来进行申明。
```c++
int value{5};
int &ref{value};
```
上面的语法中，`&`不表示取地址，而是表示`引用`。非常量值的引用通常简称为`引用`。跟指针语法中的`*`符号使用一样，`&`不论是靠近类型名还是引用变量名都是可以的。
```c++
// 这两个语法是一样的
int &ref1{value};
int& ref2{value};
```

#### 引用变量作为别名
引用变量通常表示它所引用的变量的值。在这种情况下，引用变量就是一个别名。例如：
```c++
int x{5}; // 普通变量
int &y{x}; // y是x的引用
int &z{y}; // z是y的引用
```
在上面的代码片段中，对x、y、z三个变量的赋值或者取值都是一样的。下面是引用变量的使用的例子：
```c++
#include <iostream>

int main() {
    int value{5};
    int &ref{vaule};

    value = 6; // value=ref=6
    ref = 7; // vaule=refs=7

    std::cout << value << '\n'; // prints 7
    ref++;
    std::cout << value << '\n'; // prints 8
    return 0;
}
```
通过上面的代码运行过程可以知道，`ref`就是`value`的别名。可以通过如下代码知道他们其实是指向相同的地址的：
```c++
std::cout << &erf;
std::cout << &value;
```
#### 左值与右值
在c++中，变量是左值类型。一个左值，它的值在内存中拥有一个地址。因为所有变量都有地址，所以所有变量都是左值。左值之所以叫做左值，是因为它是唯一一个能够存在于赋值语句左侧的值。所以在赋值语句中，`=`号左侧需要时一个左值。`5 = 6;`会报错，因为`5`不是左值（5没有内存地址）。当一个左值被赋值的时候，左值对应地址的值会被修改。
于左值相对应的就是右值。右值是一个非左值表达式，例如数字`5`与非右值表达式`2 + x`。下面是一些赋值语句的例子，展示了右值是如果evaluate(估值)的。
```c++
int y;
y = 4;      // 4是右值，它被估值为4 然后赋值给左值y
y = 2 + 5;  // 2 + 5 是右值，它被赋值为7， 然后赋值给左值y

int x;
x = y;  // 这种情况下，y是右值，他被估值为7，然后赋值给左值x
x = x;  // 右边的x是右值，它被估值为7，然后赋值给左值x
x = x + 1; // 同上
```
从上面的代码片段可以知道，一个变量在不同的语境的既可能作为右值也可能作为左值。

#### 引用变量必须初始化
引用变量必须在创建时初始化：
```c++
int value{5};
int &ref{value}; // 正确
int &invalidRef; // 错误
```
跟指针不一样，指针可以保存一个空指针，但是没有一个引用是空引用。非const变量的引用不能被初始化为对const变量的引用，同时也不能被初始化为对一个右值的引用。
```c++
int x{5};
int &ref1{x}; // 正确

const int y{7};
int &ref2{y}; // 错误

int &ref3{7}; // 错误
```

#### 引用变量不能被重新赋值
引用变量已经初始化，就不能被再次赋值。
```c++
int value1{5};
int value2{6};

int &ref{vaule1};
ref = value2;  // 这里不是将ref重新赋值为对value2的引用，而是将value2的值赋值给ref，所以ref = value1 = 6
```

#### 引用作为函数参数
引用作为函数参数是相对于c来说的一个重要应用了。在引用作为函数参数的情况下，引用就是参数的别名，所以在调用的时候形参名其实就是实参的别名，不存在将实参的值拷贝到形参中。所以这种使用方法提高了程序的性能。使用引用作为函数参数，跟使用指针作为函数参数类似，不过使用引用作为函数参数更易于理解。  
在引用作为函数参数的情况下，形参是实参的别名，所以修改形参的值就修改了实参的值。
```c++
#include <iostream>

void changN(int &value) {
    value = 6;
}

int main() {
    int n = 9;
    std::cout << n << '\n';  // prints 9

    changN(n);
    std::cout << n << '\n';  // prints 6
    return 0;
}
```
所以，当函数需要修改实参的值的时候，可以使用非const引用变量作为函数的参数。

#### 数组引用作为函数参数
在c语言中，一般传入数组名或者指针作为函数的参数，这样形参就变成了指针了，对数组长度是无法获取的，c中的做法一般是在传入指针的同时将长度也传入函数。但是如果使用数组的引用作为函数参数，就不会有这个问题。
```c++
#include <iostream>
/* 注意使用数组引用作为函数参数时，需要指定数组长度 */
void printElements(int &array[5]) {
    int length{static_cast<int>(std::size(array))}; // 注意std::size需要c++17的支持
    // int length{sizeof(array)/sizeof(array[0])};  // 不支持c++17时用这个
    for (int i{0}; i<length; i++) {
        std::cout << array[i] << '\n';
    }
}

int main() {
    int arr[]{2,4,6,8,10};
    printElements(arr);

    return 0;
```

#### 使用引用快捷访问内部成员
引用还有一个应用的比较少的场景是对对象内部成员的引用。看下面的代码：
```c++
struct Something
{
    int value1;
    float value2;
};

struct  Other
{
    Something something;
    int othervalue;
};
Other other;
```
当在`other`中，需要访问`something`成员的`value1`时，需要通过`other.something.value1`来访问，如果成员嵌套的足够深，那么代码就显得冗余了。引用使得在这种情况下，访问内嵌成员更简单：
```c++
int &ref{other.something.value1};
ref = 5; // 跟 other.something.value1 = 5; 效果一样
```
不过，这种情况下使用引用可能使得程序可读性变差，具体应用根据场景决定。

#### 引用 vs 指针
引用其实是在编译器内部通过指针来实现的，一般就是一个常量指针。使用引用的情况程序可读性要好一些。因为引用在申明的时候就必须显示初始化（不能初始化为null），并且只能初始化一次，所以安全性要比指针高。`如果给定任务可以通过引用或指针解决，则通常应首选引用。 指针仅应在引用不足以应付的情况下使用（例如，动态分配内存）。`

#### 常量引用
常量引用通常用于函数参数，因为传入后不能修改实参，所以更安全。
```c++
#include <iostream>
 
void printIt(const int& x)
{
    std::cout << x;
}
 
int main()
{
    int a{ 1 };
    printIt(a); // non-const l-value
 
    const int b{ 2 };
    printIt(b); // const l-value
 
    printIt(3); // literal r-value
 
    printIt(2+b); // expression r-value
 
    return 0;
}
```
总结：当使用引用作为函数参数时，如果参数是非指针类型或者非基本数据类型(int,float...)时，使用常量引用。