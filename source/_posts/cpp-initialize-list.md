---
title: c++成员初始化列表
date: 2021-02-25 19:49:56
tags: c++
categories: c++
---
c++成员初始化列表，用于c++构造函数中。c++成员初始化列表的语法为：在构造函数名小括号之后，大括号之前，在小括号之后使用':'，然后是'成员变量{初始化值}'。看如下例子:
```c++
#include <iostream>
#include <cstdint>

class RGBA
{
private:
    std::uint_fast8_t m_red;
    std::uint_fast8_t m_green;
    std::uint_fast8_t m_blue;
    std::uint_fast8_t m_alpha;

public:
    RGBA(std::uint_fast8_t red=0, std::uint_fast8_t green=0, std::uint_fast8_t blue=0, std::uint_fast8_t alpha=255)
        : m_red{red}, m_green{green}, m_blue{blue}, m_alpha{alpha}
        {

        }
    void print()
    {
        std::cout << "r=" << static_cast<int>(m_red) << " g=" << static_cast<int>(m_green) << " b=" << static_cast<int>(m_blue) << " a=" << static_cast<int>(m_alpha) << '\n';
    }
};

int main()
{
    RGBA teal{ 0, 127, 127 };
    teal.print();
 
    return 0;
}

```

##### best practice
在编写构造函数时，应该使用成员初始化列表。在初始话时，应该将成员初始化列表中的成员名顺序保持跟成员申明的顺序一致。

##### 委派构造函数
如果在一个类中有多个构造函数，有可能需要在一个构造函数中调用另一个构造函数，这种情况下，不能直接在构造函数体中调用另一给构造函数，应该通过初始化列表来调用。看下面的例子:
```c++
class Foo
{
public:
    Foo()
    {
        // code to do A
    }
    Foo(int value)
    {
        Foo();
        // codo to do B
    }
};
```
上面的构造函数`Foo(int value)`中调用了`Foo()`构造函数，但是这样并不生效，在函数体中调用第一个构造函数只会重新实例化一个Foo类。正确的做法应该如下:
```c++
#include <string>
#include <iostream>

class Employee
{
private:
    int m_id{};
    std::string m_name{};

public:
    Employee(int id=0, const std::string &name=""):
        m_id{id}, m_name{name}
    {
        std::cout << "Employee " << m_name << " created.\n"
    }

    Employee(const std::string &name) : Employee{0, name}
    {}
};
```
从上面的例子可以看出，第二个`Employee`构造函数调用了第一个构造函数。