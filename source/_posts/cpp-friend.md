---
title: c++ friend 关键字
date: 2021-03-01 20:08:12
tags: c++
categories: c++
---
c++中friend关键字用于使得不在本类中的成员函数能够访问本类的私有成员变量。分为三种情况:
- 某个外部函数，利用friend关键字能够访问本类中的私有成员变量
- 另一个类中的所有成员函数通过friend关键字能够访问本类中的私有成员变量。
- 另一个类中的某个成员函数通过friend关键字能够访问本类中的私有成员变量，另一个类中的其他成员函数不能访问本类中的私有成员变量。
<!-- more -->

先看看第一种情况，外部函数能够访问本类的私有成员变量。
```c++
class Accumulator
{
private:
    int m_value;
public:
    Accumulator() { m_value = 0; }
    void add(int value) { m_value += value; }

    // 申明能够访问该类内部私有成员的函数
    friend void reset(Accumulator &accmulator);
}

// 通过上面的申明，rest函数已经是Accumulator类的友元函数了
void reset(Accumulator &accmulator)
{
    accmulator.m_value = 0;
}

int main()
{
    Accumulator acc;
    acc.add(5);
    reset(acc);

    return 0;
}
```
注意上面`reset`函数的定义，reset函数因为是外部函数，所以是拿不到Accumulator类的*this指针的，所以需要传入Accumulator作为其参数。  

接下来看第二种，情况友元类，该情况下第一个类是第二个类的友元类，所以第二个类可以访问第一个类的私有成员变量。
```c++
#include <iostream>
 
class Vector3d
{
private:
    double m_x{};
    double m_y{};
    double m_z{};
 
public:
    Vector3d(double x = 0.0, double y = 0.0, double z = 0.0)
        : m_x{x}, m_y{y}, m_z{z}
    {
 
    }
 
    void print() const
    {
        std::cout << "Vector(" << m_x << " , " << m_y << " , " << m_z << ")\n";
    }

    // 申明友元类
    // 申明Point3d是Vector3d的友元类，所以Point3d可以访问Vector3d中的私有成员
    friend class Point3d;
};
 
class Point3d
{
private:
    double m_x{};
    double m_y{};
    double m_z{};
 
public:
    Point3d(double x = 0.0, double y = 0.0, double z = 0.0)
        : m_x{x}, m_y{y}, m_z{z}
    {
 
    }
 
    void print() const
    {
        std::cout << "Point(" << m_x << " , " << m_y << " , " << m_z << ")\n";
    }
 
    // 注意：跟第一种情况一样，因为不能访问Vector3d的this指针，所以同样需要传入一个Vector3d对象
    void moveByVector(const Vector3d &v)
    {
        // implement this function as a friend of class Vector3d
        m_x += v.m_x;
        m_y += v.m_y;
        m_z += v.m_z;
    }
};
 
int main()
{
    Point3d p{1.0, 2.0, 3.0};
    Vector3d v{2.0, 2.0, -3.0};
 
    p.print();
    p.moveByVector(v);
    p.print();
 
    return 0;
}
```
这种情况是，`Point3d`是类`Vector3d`的友元类，所以`Point3d`中的所有成员函数都是可以访问`Vector3d`类的私有成员变量的。

再看第三种情况。还是以`Point3d`与`Vector3d`作为例子，不过这种情况下，`Point3d`中只允许`moveByVector`函数作为`Vector3d`的友元函数。
```c++
#include <iostream>

class Vector3d;
class Point3d
{
private:
    double m_x{};
    double m_y{};
    double m_z{};
 
public:
    Point3d(double x = 0.0, double y = 0.0, double z = 0.0)
        : m_x{x}, m_y{y}, m_z{z}
    {
 
    }
 
    void print() const
    {
        std::cout << "Point(" << m_x << " , " << m_y << " , " << m_z << ")\n";
    }
 
    // 方法申明
    void moveByVector(const Vector3d &v);
};

class Vector3d
{
private:
    double m_x{};
    double m_y{};
    double m_z{};
 
public:
    Vector3d(double x = 0.0, double y = 0.0, double z = 0.0)
        : m_x{x}, m_y{y}, m_z{z}
    {
 
    }
 
    void print() const
    {
        std::cout << "Vector(" << m_x << " , " << m_y << " , " << m_z << ")\n";
    }

    // 申明 moveByVector 是本类的友元函数
    friend void Point3d::moveByVector(const Vector3d &v);
};

// 函数定义，注意顺序
void Point3d::moveByVector(const Vector3d &v)
{
    // implement this function as a friend of class Vector3d
    m_x += v.m_x;
    m_y += v.m_y;
    m_z += v.m_z;
}
 
int main()
{
    Point3d p{1.0, 2.0, 3.0};
    Vector3d v{2.0, 2.0, -3.0};
 
    p.print();
    p.moveByVector(v);
    p.print();
 
    return 0;
}
```
这种情况下，主要上要考虑到各个类之间的顺序。顺序为:申明要访问的类-->定义包含友元函数的类(其中友元函数只能申明，不能定义在类内)-->定义要访问的类(其中用friend申明友元函数)-->定义友元函数。