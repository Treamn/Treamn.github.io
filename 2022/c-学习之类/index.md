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

- 除了可以将普通的函数定义成友元，类还可以把其他类定义成友元，也可以把其他类的成员函数定义成友元。  
### 类之间的友元关系
假设需要为window_mgr类添加一个名为clear的成员，负责将一个指定的Screen的内容设为空白。为完成这一任务，clear需要访问Screen的私有成员；要令这种行为合法，Screen需要将window_mgr指定成它的友元。  
```cpp
class Screen{
    friend class window_mgr;
};
```
如果一个类指定了友元类，则友元类的成员函数可以访问此类包括非公有成员在内的所有成员。通过上面的声明，window_mgr被指定为Screen的友元，因此可以将window_mgr的clear成员写成如下形式：
```cpp
class window_mgr{
    public:
        using ScreenIndex = std::vector<Screen>::size_type;
        void clear(ScreenIndex);
    private:
        std::vector<Screen> screens{Screen(24, 80, ' ')};
};

void window_mgr::clear(ScreenIndex i){
    Screen &s = screens[i];  //s是想清空的屏幕
    s.contents = string(s.height * s.width, ' '); //将选定的屏幕重置为空白
}
```
需要注意的是，友元关系不存在传递性。也就是说，如果window_mgr有它自己的友元，则这些友元不能理所当然的具有访问Screen的特权。
### 令成员函数作为友元
除了令整个window_mgr作为友元之外，Screen还可以只为clear提供访问权限。当把一个成员函数声明成友元时，必须明确指出该成员函数属于哪个类：
```cpp
class Screen{
    // window_mgr::clear必须在Screen类之前被声明
    friend void window_mgr::clear(ScreenIndex);
    // Screen类的剩余部分
};
```
要令某个成员函数作为友元，必须仔细组织程序的结构以满足声明和定义的彼此依赖关系。在本例中，按照如下方式设计程序：
- 首先定义window_mgr类，其中声明clear函数，但是不能定义它。在clear使用Screen的成员之前必须先声明Screen。  
- 接下来定义Screen，包括对于clear的友元声明。
- 最后定义clear，此时它才可以使用Screen的成员。

### 函数重载和友元
尽管重载函数的名字相同，但他们仍是不同的函数。因此，如果想把一组重载函数声明成它的友元，需要对这组函数的每一个分别声明：
```cpp
extern std::ostream& storeOn(sdt::ostream &, Screen &);
extern BitMap& storeOn(BitMap &, Screen &);
class Screen{
    friend std::ostream& storeOn(sdt::ostream &, Screen &);
};
```
Screen类把接受ostream&的storeOn声明成它的友元，但是接受BitMap&作为参数的版本仍然不能访问Screen。

### 友元声明和作用域
类和非成员函数的声明不是必须在它们的友元声明之前。当一个名字第一次出现在一个友元声明中，我们隐式地假定该名字在当前作用域中是可见的。然而，友元本身不一定真的声明在当前作用域中。  
甚至就算在类的内部定义该函数，我们也必须在类的外部提供相应的声明从而使得函数可见。换句话说，即使我们仅仅是用声明友元的类的成员调用该友元函数，他也必须是声明过的：
```cpp
struct X {
    frined void f();
    X() { f(); }
    void g();
    void h();
};
void X::g() { return f(); } // 错误，f没有被声明
void f();
void X::h() { return f(); } // 正确，现在f的声明在作用域中
```

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

---
## 返回*this的成员函数
```cpp
class Screen{
    public:
        typedef std::string::size_type pos;
        Screen &set(char);
        Screen &set(pos, pos, char);
};
inline Screen &Screen::set(char c){
    contents[cursor] = c; //
    return *this;  //将this对象作为左值返回
}
inline Screen &Screen::set(pos r, pos col, char ch){
    contents[r * width + col] = ch;
    return *this;
}
```
返回引用的函数是左值的，意味着这些函数返回的是对象本身而非对象的副本，如果将一系列的操作连接在一条表达式中的话： 
```cpp
myScreen.move(4,0).set('#');
```
如果令move和set返回的是Screen而非Screen&的话，上述语句行为将大不相同，等价于：
```cpp
Screen temp = myScreen.move(4,0); //对返回值进行拷贝
temp.set('#');  //  不会改变myScreen的contents
```
如果定义的返回类型不是引用，则move的返回值将是*this的副本，因此调用set只能改变临时副本，而不能改变没有Screen的值。
### 从const成员函数返回*this
一个const成员函数如果以引用的形式返回*this，那么它的返回类将是常量引用。

### 基于const的重载
```cpp
class Screen{
    public:
        typedef std::string::size_type pos;
        Screen &display(std::ostream &os){
            do_display(os);
            return *this;
        }
        Screen &display(std::ostream &os) const{
            do_display(os);
            return *this;
        }
    private:
        void do_display(std::ostream &os) const{
            os << contents;
        }
```
当display调用do_display时，它的this指针隐式的传递给do_display。当display的非常量版本调用do_display时，它的this指针将隐式的从指向非常量的指针转换成指向常量的指针。  
当do_display完成后，display函数各自返回解引用this所得的对象。在非常量版本中，this指向一个非常量对象，因此display返回一个非常量的引用；而const成员则返回一个常量引用。  
当在某个对象上调用display时。该对象是否是const决定了应该调用disply的哪个版本。
```cpp
Screen myScreen(5,3);
const Screen blank(5,3);
myScreen.set('#').display(cout);  //非常量版本
blank.display(cout);  //常量版本
```

---
## 类类型
对于两个类来说，即使它们的成员完全一样，这两个类也是两个不同的类型。

### 类的声明
像可以把函数的声明和定义分开来一样，也可以先声明类而不定义它。  
```cpp
class Screen;
``` 
这种声明有时被称为前向声明，向程序中引入名字Screen并指明Screen是一种类类型。对于类型Screen来说，在它声明之后定义之前是一个不完全类型，此时只知道Screen是一个类类型，但不清楚其包含哪些成员。  
不完全类型只能在非常有限的情景下使用：可以定义指向这种类型的指针或引用，也可以声明（但不能定义）以不完全类型作为参数或返回类型的函数。  
直到类被定义之后数据成员才能被声明成这种类类型。只有当类全部完成后类才算被定义，所以一个类的成员类型不能是该类自己。然而，一旦一个类的名字出现后，他就被认为是声明过了（但尚未定义），因此类允许包含指向它自身类型的引用或指针。

---
## 类的作用域
每个类都会定义自己的作用域，在类的作用域之外，普通的数据和函数成员只能由对象、引用或者指针使用成员访问运算符来访问。对于类类类型成员则使用作用域运算符进行访问。不论哪种情况，跟在运算符之后的名字都必须是对应类的成员：
```cpp
Screen::pos ht = 24, wd = 80; // 使用Screen定义的pos类型
Screen scr(ht, wd, ' ');
Screen *p = &scr;
char c = scr.get(); // 访问scr对象的get成员
c = p -> get(); // 访问p所指对象的get成员
```

### 作用域和定义在类外部的成员类
一旦遇到类名，定义的剩余部分就在类的作用域之内，这里的剩余部分包括参数列表和函数体。结果就是，可以直接使用类的其他成员而无须再次授权。
```cpp
void window_mgr::clear(ScreenIndex){
    Screen &s = screen[i];
    s.contents = string(s.height * s.width, ' ');
}
```
因为编译器在处理参数列表之前已经明确了我们当前正位于window_mgr类的作用域中，所以不必再专门说明ScreenIndex是window_mgr声明的。出于同样的原因，编译器也能知道函数体中用到的screens也是在window_mgr类中定义的。  
另一方面，函数的返回类型通常出现在函数名之前。因此当成员函数定义在类的外部时，返回类型中使用的名字都位于类的作用域之外。这时，返回类型必须指明它是哪个类的成员。例如，向window_mgr类中添加一个新的名为addScreen的函数，负责向显示器添加一个新的屏幕。这个成员的返回类型将是ScreenIndex，用户可以通过它定位到指定Screen：
```cpp
class window_mgr{
    public:
        ScreenIndex addScreen(const creen&);
};
window_mgr::ScreenIndex window_mgr::addScreen(const Screen &s){
    screens.push_back(s);
    return screen.size() - 1;
}
```
因为返回类型出现在类名之前，所以事实上它是位于window_mgr类的作用域之外的。在这种情况下，要想使用ScreenIndex作为返回类型，我们必须明确指定哪个类定义了它。

---
## 名字查找与类的作用域
- 首先在名字所在的块中寻找其声明语句，只考虑在名字的使用之前出现的声明。
- 如果没找到，继续查找外层作用域。
- 如果最终没有找到匹配的声明，则程序报错。
对于定义在类内部的成员函数来说，解析其中名字的方式与上述的查找规则有所区别。类的定义分两步处理：
- 首先，编译成员的声明。
- 直到类全部可见后才编译函数体。
按照这种两阶段的方式处理类可以简化类代码的组织方式。因为成员函数体直到整个类可见后才会被处理，所以它能使用类中定义的任何名字。

### 用于类成员声明的名字查找
两阶段的处理方式只适用于成员函数中使用的名字。声明中使用的名字，包括返回类型或者参数列表中使用的名字，都必须确保在使用前可见。如果某个成员的声明使用了类中尚未出现的名字，则编译器将会在定义该类的作用域中继续查找。例如：
```cpp
typedef double Money;
string bal;
class Account{
    public:
        Money balance(){
            return bal;
        }
    private:
        Money bal；
};
```
当编译器看到balance函数的声明语句时，将在Account类的范围内寻找对Money的声明。编译器只考虑Account中在使用Money前出现的声明，因为没找到匹配的成员，所以编译器会接着到类的外层作用域中查找。在本例中，编译器会找到Money的typedef语句。另外ia，balance函数体在整个类可见后才被处理，因此，该函数返回名为bal的成员，而非外层作用域的string对象。

### 类姓名要特殊处理
在类中，如果成员使用了外层作用域中的某个名字，而该名字代表一种类型，则类不能在之后重新定义该名字。
```cpp
typedef double Money;
class Account{
    public:
        Money balance(){
            return bal;
        }
    private:
        typedef double Money; // 错误，不能重新定义Money
        Money bal;
};
```
需要注意的是，即使Account中定义的Money类型与外层作用域一致，上述代码仍然是错误的。  
类型名的定义通常出现在类的开始处，这样就能确保所有使用该类型的成员都出现在类名的定义之后。

### 成员定义中普通块作用域的名字查找
c成员函数中使用的名字按照如下方式解析:
- 首先，在成员函数内查找该名字的声明。和前面一样，只有在函数使用之前出现的声明才被考虑。
- 如果在成员函数内没有找到，则在类内继续查找，这时类的所有成员都可以被考虑。
- 如果类内也没找到该名字的声明，在成员函数定义之前的作用域内继续查找。   
一般来说，不建议使用其他成员的名字作为某个成员函数的参数。
```cpp
int height;
class Screen{
    public:
        typedef std::string::size_type pos;
        void dummy_fcn(pos height){
            cursor = width * height; //使用哪个height
        }
    private:
        pos cursor = 0;
        pos height = 0, width = 0;
};
```
当编译器处理dummy_fcn中的乘法表达式时，






