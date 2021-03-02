---
title: c++构造函数
date: 2021-01-29 11:03:06
tags: c/c++
categories: c/c++
---
c++构造函数是类中的一个特殊的成员函数，它在类实例化的时候被自动调用。构造函数经常用于初始化成员变量，或者做一些初始化步骤(如打开文件，打开数据库等)。区别于普通成员函数，构造函数有如下特点:
- 构造函数名字跟类名相同
- 构造函数没有返回值

##### 关于对象初始化
关于对象初始化，有几种方式，看下面的例子：
```c++
class Foo
{
public:
    int m_x;
    int m_y;
};

int main(){
    Foo foo1 = {4, 5}; //  initialization list
    Foo foo2 {6, 7}; //  uniform initialization
}
```
在没有显示定义构造函数的情况下，可以使用上面的初始化方式初始化`public`类型的成员变量。如果成员变量是`private`的，那么就需要自定义构造函数了。  

看下的关于构造函数的定义方法:
```c++
#include <cassert>
 
class Fraction
{
private:
    int m_numerator;
    int m_denominator;
 
public:
    Fraction() // default constructor
    {
         m_numerator = 0;
         m_denominator = 1;
    }
 
    // Constructor with two parameters, one parameter having a default value
    Fraction(int numerator, int denominator=1)
    {
        assert(denominator != 0);
        m_numerator = numerator;
        m_denominator = denominator;
    }
 
    int getNumerator() { return m_numerator; }
    int getDenominator() { return m_denominator; }
    double getValue() { return static_cast<double>(m_numerator) / m_denominator; }
};
```
上面的类可以通过如下方法进行实例化:
```c++
Fraction fiveThirds{5, 3}; // List initialization, calls Fraction(int, int)
Fraction threeQuarters(3, 4); // Direct initialization, also calls Fraction(int, int)
```
上面的实例化使用了大括号和小括号两种初始化方式，一般情况下我们应该使用大括号初始化。  

下面是一个对构造函数默认参数使用的例子:
```c++
#include <iostream>
#include <assert.h>

class Ball
{
private:
    std::string m_color;
    float m_radius;

public:
    Ball(std::string color="black", float radius=10.0)
    {
        assert(radius > 0);
        m_color = color;
        m_radius = radius;
    }

    Ball(float radius)
    {
        assert(radius > 0);
        m_color = "black";
        m_radius = radius;
    }

    void print()
    {
        std::cout << "color: " << m_color << ", radius: " << m_radius << '\n';
    }
};

int main()
{
    Ball def{};
    def.print();
 
    Ball blue{ "blue" };
    blue.print();
    
    Ball twenty{ 3.5 };
    twenty.print();
    
    Ball blueTwenty{ "blue", 20.0 };
    blueTwenty.print();
 
    return 0;
}
```

##### 类内包含类情况的构造函数调用
一个类内可能包含成员类型是其它类类型的情况。默认情况下，当类初始化时，会先调用类类型成员的构造函数。看下面的例子：
```c++
#include <iostream>
 
class A
{
public:
    A() { std::cout << "A\n"; }
};
 
class B
{
private:
    A m_a; // B contains A as a member variable
 
public:
    B() { std::cout << "B\n"; }
};
 
int main()
{
    B b;
    return 0;
}
```
上面的例子会输出：
```c++
A
B
```

##### 构造函数需要注意的地方
1. 如果没有定义构造函数，那么编译器会默认为类创建一个空构造函数，所以该类还是可以被实例化
2. 如果程序员手动为类定义了构造函数，那么编译器将不会为类自动创建构造函数了，所以如果程序员实现了带参数的构造函数，没有实现无参构造函数，那么在实例化的时候将不能使用不带参数实例化。这种情况下可以使用`<class>()=default`添加一个默认的构造函数。
3. 在编写程序时，应该在构造函数中初始化所有的内部成员变量。
4. 尽量减少类的构造函数数量
5. 需要注意，实例(对象)并不是构造函数创建的，实例是编译器在内存中创建的，然后才调用构造函数。
