---
title: c/c++ 变量
date: 2020-12-04 14:29:08
tags:
---
### 关于c/c++变量溢出问题
1. 无符号变量溢出
无符号变量没有负数，只有正数。例如：unsigned char var = 280;其实际值为280%256=24。

2. 有符号变量溢出
有符号数最高位表示符号位，1为负数，0为正数。例如：unsigned char var = 128,溢出了，最高为1，所以为负数，根据负数为反码+1，可以知道实际值为-128。
<!-- more -->

### 关于c/c++变量，不同平台或者架构int变量size不一样问题
1. 在不同的平台或者编译器中，标准类型int所占用的size可能不一样，为了解决这给问题，C99中引入了`stdint.h`头文件来定义指定宽度类型。如:`std::int8_t`、`std::uint8_t`、`std::uint16_t`、`std::int32_t`、`std::int64_t`等。所以在C中可以通过引入`stdint.h`来使用这些类型。在C++11中，这些类型被纳入到`cstdint`头文件中。
使用如上头文件的时候，需要注意，可能某些平台并不支持，所以在代码可移植性方面较差。
2. c++标准定义的类型(实际使用中，推荐用该方法，因为他是安全的)
std::int_fast8_t
std::int_fast32_t
std::uint_fast16_t
std::int_least8_t
std::uint_least16_t
std::int_least_64_t

### constexpr关键字
`constexpr`关键字是C++11中引入的关键字，用于申明常量的值需要在编译时确定。
```c++
constexpr int a =25; // ok
constexpr int sum{4+11}; // ok
constexpr int var; // not okay
```

### 关于全局变量与局部变量
1. 同名局部变量在其作用域内会屏蔽全局变量，也就是局部变量会被优先使用
```c++
#include <iostream>
int Var = 9;
int main() {
    int Var = 2;
    std::cout << "Var = " << Var << '\n';
    return 0;
}
```
上面的程序运行结果为`Var = 2`。在C++中可以通过范围运算符`::`来指定使用全局变量。
```c++
#include <iostream>
int Var = 9;
int main() {
    int Var = 2;
    std::cout << "Var = " << ::Var << '\n';
    return 0;
}
```
上面的程序运行结果为`Var = 9`。

### 关于C++17中的`inline variables`
c++17中为变量引入了`inline variable`类型。`inline variable`允许在多个文件中定义，而不会违反编译检查规则，当在多个文件中定义了同名的`inline variable`时，编译器在编译时将所有的同名`inline variables`实例化为一个变量。请看如下例子中的使用：
constants.h:
```c++
#ifndef CONSTANTS_H
#define CONSTANTS_H

namespace constants {
    inline constexpr double pi {3.14159};
    inline constexpr double avogadro {6.022141};
    inline constexpr double gravity {9.2};
}
#endif
```
main.cpp:
```c++
#include "constants.h"
#include <iostream>

int main() {
    std::cout << "Enter a radius: ";
    int radius;
    std::cin >> radius;
    std::cout << "The circumference is: " << 2.0 * radius * constants::pi << '\n'; 
    return 0;
}
```

### c++中别名与typedef
typedef 用法：
```c++
// typedef 定义的别名应该在后面加上`_t`，表面它是一个类型而不是变量
typedef int Score_t;
typedef int* Pint_t;
typedef float distance_t; 
```
别名的用法：
```c++
using score_t = int;
using distance_t = float;
using pairlist_t = std::vector<std::pair<std::string, int>>;
```