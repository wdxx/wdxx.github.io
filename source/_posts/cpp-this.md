---
title: c++中this指针
date: 2021-02-26 15:54:54
tags: c++
catgories: c++
---
在c++中，当对象在调用成员函数时，该对象的成员函数是如何分辨出该操作哪个对象的成员的变量的呢？看下面的例子。
```c++
#include <iostream>

class Simple
{
private:
    int m_id;
public:
    Simple(int id) : m_id{id}
    {}

    void setID(int id)
    {
        m_id = id;
    }

    int getID() {return m_id;}
};

int main()
{
    Simple simple{1};
    simple.setID(2);
    std::cout << simple.getID() << '\n';

    return 0;
}
```
上面的例子中的simple.setID(2)是如何知道要设置的是simple对象的m_id成员变量的值的呢？这个就是`this`指针的作用的了。
<!-- more -->

##### c++编译器对非静态成员函数的处理
c++编译器在处理非静态成员函数时，如`simple.setID(2)`，会转换成`setID(&simple, 2)`，也就是说，非静态成员函数，在编译的时候，编译器会为该函数添加一个参数，放在第一个参数的位置。这个参数的名字就是`this`,它的类型为指向该类的一个常量指针变量。所以下面的函数:
```c++
void setID(int id)
{
    m_id = id;
}
```
会被编译器转换成：
```c++
void setID(Simple* const this, int id)
{
    this->m_id = id;
}
```
这样，当调用成员函数时，就能够正确的操作成员变量了。  
总结:  
1. 当调用`simple.setID(2)`时，实际调用的是setID(&simple, 2);
2. 在setID函数内部，`this`指针保存了simple对象的地址。
3. 在setID函数内的所有成员变量在处理的时候都会加上`this->`操作符。所以`m_id = id`在实际执行的时候，是在执行`this->m_id = id`。

#####this指针的实际使用
this指针实际使用的时候一般是将this指针指向的对象返回，经常用于运算符重载中。看下面的例子:
```c++
#include <iostream>

class Calc
{
private:
    int m_value{0};

public:
    Calc& add(int value)
    {
        m_value += value;
        return *this;
    }

    Calc& sub(int value)
    {
        m_value -= value;
        return *this;
    }

    Calc& mult(int value)
    {
        m_value *= value;
        return *this;
    }

    int getValue() {return m_value;}
};

int main()
{
    Calc calc{};
    calc.add(3).mult(5).sub(9);

    std::cout << "3 * 5 - 9 = " << calc.getValue() << '\n';
    return 0;
}

```
