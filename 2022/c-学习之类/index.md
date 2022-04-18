# C++学习之类


##  构造函数
- 类通过一个或几个特殊的成员函数来控制其对象的初始化过程，这些函数叫构造函数。构造函数的任务是初始化类对象的数据成员。  
- 构造函数没有返回类型，构造函数也有一个（可能为空的）参数列表和一个（可能为空的）函数体。类可以包含多个构造函数，不同的构造函数之间必须在参数数量或参数类型上有所区别。  
### 默认构造函数 
- 如果类中没有显式的定义构造函数，那么编译器会隐式的定义一个默认构造函数。编译器创建的构造函数又被称为合成的默认构造函数，对于大多数类来说，这个合成的默认构造函数将按照如下规则初始化类的数据成员。
1. 如果存在类内的初始值，用它来初始化成员。
2. 否则，默认初始化该成员。
### =default的含义
如果我们需要默认的行为，那么可以通过在参数列表后写上 **=default**要求编译器生成构造函数。其中，=default既可以和声明一起出现在类的内部，也可以作为定义出现在类的外部。和其他函数一样，如果=default出现在类的内部，则默认构造函数是内联的；如果在类的外部，则该成员默认情况下不是内联的。  
```cpp
Sales_data(const std::string&s): book(s){}
Sales_data(const std::string&s, unsigned n, double p): book(s), units_sold(n), revenue(p*n){}
```
函数名和花括号之间的部分称为构造函数初始值列表，负责为新创建的对象的一个或几个数据成员赋初值。  
构造函数不应该轻易覆盖掉类内的初始值，除非新赋的值与原值不同。如果不能使用类内初始值，则所有构造函数都应该显式的初始化每个内置类型的成员。  

---
## 拷贝、赋值和析构
除了定义类的对象如何初始化之外，类还需要控制拷贝、赋值和销毁对象时发生的行为。如果不主动定义这些操作，编译器将替我们合成它们。  
### 某些类不能依赖于合成的版本  
尽管编译器可以替我们合成拷贝、赋值和销毁的操作，但是必须要清楚的一点是，对于某些类来说合成的版本无法正常工作。

---
## 访问控制与封装
- 定义在 **public** 说明符之后的成员在整个程序内可被访问，public成员定义类的接口。  
- 定义在 **private** 说明符之后的成员可以被类的成员函数访问，但是不能被使用该类的代码访问，private部分封装了类的实现细节。  

### 使用class或struct关键字
struct和class之间的唯一一点区别就是，struct和class的默认访问权限不同。  
如果使用struct关键字，则定义在第一个访问说明符之前的成员时public的；相反，如果使用class关键字，则这些成员是private的。

---
## 友元
类可以允许其他类或函数访问它的非公有成员，方法是令其他类或者函数成为他的友元（friend）。如果类想把一个函数作为它的友元，只需要增加一条以friend关键字展开的函数声明语句即可。
```cpp
class Sales_data{
    friend Sales_data add(const Slaes_data&, const Slaes_data&);
};
```
友元声明只能出现在类定义的内部，但是在类内出现的位置不限。友元不是类的成员也不受它所在区域访问控制级别的约束。  
一般来说，**最好**在类定义开始或结束前的位置集中声明友元。
### 友元的声明  
友元的声明仅仅指定了访问的权限，而非一个通常意义上的函数声明。如果希望类的用户能够调用某个友元函数，那么就必须在友元声明之外再专门对函数进行一次说明。
为了使友元对类的用户可见，通常把友元的声明和类本身放置在同一个头文件中。 

---
## 类的其他特性
### 定义一个类型成员 
```cpp
class Screen{
    public:
        typedef std::string::size_type pos;
    private:
        pos sursor = 0;
        pos height = 0, width = 0;
        std::string contents;
};
```
在Screen的public部分定义pos，这样用户就可以使用这个名字。  
关于pos的定义又两点需要注意。首先，我们使用了typedef，也可以等价的使用类型别名：
```cpp
class Screen{
    public:
        using pos = std::string::size_type;
};
```
其次，用来定义类型的成员必须先定义后使用。类的成员通常出现在类开始的地方。
### Screen类的成员函数
```cpp
#include <iostream>

using namespace std;

class Screen{
    public:
        typedef std::string::size_type pos;
        Screen() = default;//Screen有另一个构造函数，所以本函数是必须的。
        Screen(pos ht, pos wd, char c):height(ht), width(wd), contents(ht*wd,c){}//cursor初始值初始化为0
        char get() const{
            return contents[cursor];  //隐式内联
        }
        inline char get(pos ht, pos wd) const; //显式内联
        Screen &move(pos r, pos c);  //能在之后设置为内联
    private:
        pos cursor = 0;
        pos height = 0, width = 0;
        std::string contents;
};
```
因为已经提供一个构造函数，所以编译器不会自动生成默认的构造函数。如果类需要默认构造函数，就必须显式的把它声明出来。  
在第二个构造函数为cursor成员隐式的使用类内初始值。如果类中不存在cursor的类内初始值，就需要像其他成员一样显式的初始化cursor。

### 令成员作为内联函数
在类中，常有一些规模较小的函数适合于被声明为内联函数，定义在类内部的成员函数都是自动inline的。  
我们可以在类的内部把inline作为声明的一部分显式的声明成员函数，同样地，也能在类的外部用inline关键字修饰函数的定义。
```cpp
inline //在函数的定义处指定inline
Screen &Screen::move(pos r, pos c){
    pos row = r * width;  // 计算行的位置
    cursor = row + c;  //在行内将光标移动到指定列
    return *this;  //以左值的形式返回对象
}
char Screen::get(pos r, pos c) const{  // 在类的内部声明成inline
    pos row = r * width;  //计算行的位置
    return contents[row + c];  //返回给定字符
}
```
最好只在类外部定义的地方说明inline，这样可以使类更容易理解。inline函数应该与相应的类定义在同一个头文件中。

### 重载成员函数
和非成员函数一样，成员函数也可以被重载，只要函数之间在参数的数量和/或类型上有所区别即可。成员函数的函数匹配过程同样与非成员函数十分类似。
```cpp
Screen myscreen;
char ch = myscreen.get(); //Screen::get()
ch = myscreen.get(0,0);   // Screen::get(pos,pos)
```
### 可变数据成员
有时我们希望能修改类的某个数据成员，即使是在一个const成员函数内。可以通过在变量的声明中加入 **mutable**关键字做到这一点。  
一个可变数据成员永远不会是const，即使它是const对象的成员，因此，一个const成员函数可以改变一个可变成员的值。
```cpp
class Screen{
    public:
        void some_member() const;
    private:
        mutable size_t access_ctr; //即使在一个const对象内也能被修改
};
void Screen::some_member() const{
    ++access_ctr;  //保存一个计数值，用于记录成员函数被调用的次数
}
```
### 类数据成员的初始值
在定义好Screen类之后，我们将继续定义一个窗口管理类并用它表示显示器上的一组Screen。这个类将包含一个Screen类型的vector，每个元素表示一个特定的Screen。默认情况下，我们希望Window_mgr类开始时总是拥有一个默认初始化的Screen。在C++11标准中，最好的方式就是把这个默认值声明成一个类内初始值：
```cpp
class Windoe_mgr{
    private:
        std::vector<Screen> screens{Screen(24, 80, ' ')};
};
```
类内初始值必须使用=的初始化形式或者花括号阔气来的直接初始化形式。   
**当我们提供一个类内初始值时，必须以符号=或花括号结尾**






