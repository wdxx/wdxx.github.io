---
title: c++中的struct与class
date: 2021-01-28 20:40:39
tags: c/c++
categories: c/c++
---
在c++中既可以使用strcut也可以使用class。那么他们有什么区别呢？看下面的例子。
```c++
struct DateStruct {
    int year{};
    int month{};
    int day{};
};

class DateClass {
public:
    int m_year{};
    int m_month{};
    int m_day{};
};
```
c++中，上面定义的DateStruct与DateClass是一样的，也就是说DateSruct中的成员类型是`public`的。注意上面只是申明了两个了类型，并没有定义，所以不会分配内存。
<!-- more -->

##### 成员函数
除了了数据，class或者sruct还可以包含函数。定义在类中的函数叫做成员函数或者方法。成员函数可以定义在类里面，也可以定义的类定义外面。看下面的示例:
```c++
class DateStruct {
private:
    int m_year{};
    int m_month{};
    int m_day{};

public:
    void print() {
        std::cout << m_year << '-' << m_month << '-' << m_day << '\n';
    }

};
```
跟结构体的使用一样，类使用选择运算符(`.`)来访问类中的变量与成员函数。  
成员变量使用"m_"前缀进行命名，以区分函数形参与函数局部变量。这个命名规则很有用。首先，当我们看到对带有“ m_”前缀的变量的赋值时，我们知道我们正在更改类实例的状态。 其次，与在函数中声明的函数参数或局部变量不同，成员变量在类定义中声明。 因此，如果我们想知道如何声明带有“ m_”前缀的变量，我们知道应该在类定义中查找而不是在函数中查找。另外，按照约定，类名应以大写字母开头。

##### 使用规则
如果某个抽象只包含数据不包含方法，使用`struct`，如果包含函数，那么使用`class`。