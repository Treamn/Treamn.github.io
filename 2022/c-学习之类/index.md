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

### 构造函数初始值列表  
我们定义变量时习惯于立即对其进行初始化，而非先定义、再赋值：
```cpp
string foo = "Hello World!"; //定义并初始化
string bar;   //默认初始化成string对象
bar = "Hello World!";  //为bar赋一个新值
```
就对象的数据成员而言，初始化和赋值也有类似的区别。如果没有在构造函数的初始值列表中显式的初始化成员，则该成员将在构造函数体之前执行默认初始化。
```cpp
Sales_data::Sales_data(const string &s, unsigned cnt, double price){
    bookNo = s;
    units_sold = cnt;
    revenue = cnt * price;
}
```
先前的版本是初始化了它的数据成员，而这个版本是对数据成员进行赋值操作。

### 构造函数的初始值有时必不可少 
有时可以忽略数据成员初始化和赋值之间的差异，但并非总能这样。如果成员是const或者引用的话，必须将其初始化，类似的，当成员属于某种类类型且该类没有定义默认构造函数时，也必须将这个成员初始化。
```cpp
class ConstRef{
    public:
        ConstRef(int ii);
    private:
        int i;
        const int ci;
        int &ri;
};
```
和其他
常量对象或引用一样，成员ci和ri都必须被初始化。因此，如果我们没有为它们提供构造函数初始值的话将会引发错误：
```cpp
ConstRef::ConstRef{int ii}{
    i = ii;
    ci = ii; //错误，不能给const赋值
    ri = i;  //错误，ri没初始化
}
```
随着构造函数体一开始执行，初始化就完成。初始化const或者引用类型的数据成员的唯一机会就是通过构造函数初始值，因此该构造函数的正确形式是：
```cpp
ConstRef::ConstRef(int ii): i(ii), ci(ii), ri(ii){}
```

### 成员初始化的顺序  
成员的初始化顺序与它们在类定义中的出现顺序一致：第一个成员先被初始化，然后第二个，以此类推。构造函数初始值列表中初始值的前后位置不会影响实际的初始化顺序。  
如果一成员是用另一个成员来初始化的，那么这两个成员的初始化顺序就很关键了。
```cpp
class X{
    int i;
    int j;
public:
    //i在j之前被初始化
    X(int val):j(val), i(j){}
};
```
实际上，i先被初始化，因此这个初始值的效果是试图使用未定义的值j初始化i。  
**最好令构造函数初始值的顺序和成员声明的顺序保持一致。而且如果可能的话，尽量避免使用某些成员初始化其他成员。**  
可能的话，最好使用构造函数的参数作为成员的初始值，而尽量避免使用同一个对象的其他成员。这样的好处是可以不必考虑成员的初始化顺序。  

### 默认实参和构造函数  
Sales_data默认构造函数的行为与只接受一个string实参的构造函数差不多。唯一的区别就是接受string实参的构造函数使用这个实参初始化bookNo，而默认构造函数（隐式的）使用string的默认构造函数初始化bookNo。我们可以将它们写成一个使用默认实参的构造函数。  
```cpp
class Sales_data{
public:
    // 定义默认构造函数，令其只接受一个string实参的构造函数功能相同
    Sales_data(std::string s = “ ”):bookNo(s){}
    // 其他构造函数与之前一致
    Sales_data(std::string s, unsigned cnt, double rev):bookNo(s), units_sold(cnt), revenue(rev*cnt){}
    Sales_data(std::istream &is){
        read(is,*this);
    }
};
```
当没有给定实参，或者给定一个string实参时，两个版本的类创建了相同的对象。  
**如果一个构造函数为所有参数都提供了默认实参，则它实际上也定义了默认构造函数**
### 委托构造函数  
一个委托构造函数使用它所属类的其他构造函数执行它自己的初始化过程，或者说它把自己的一些（或全部）职责委托给了其他构造函数。  
一个委托构造函数也有一个成员初始值的列表和一个函数体。在委托构造函数内，成员初始值列表只有一个唯一的入口，就是类名本身。和其他成员的初始值一样，类名后边紧跟圆括号括起来的参数列表，参数列表必须与类中另一个构造函数相匹配。
```cpp
class Sales_data{
public:
    //非委托构造函数使用对用的实参初始化成员
    Sales_data(std::string s, unsigned cnt, double price):bookNo(s), units_sold(cnt), revenue(cnt*price){}
    //其余构造函数全都委托给另一个构造函数
    Sales_data():Sales_data("", 0, 0){}
    Sales_data(std::string s):Sales_data(s,0,0){}
    Sales_data(std::istream &is): Sales_data(){
        read(is, *this);
    }
};
```
在上面的例子中，除了一个构造函数外，其他的都委托了他的工作。第一个构造函数接受三个实参，使用这些实参初始化数据成员，然后结束工作。我们定义默认构造函数令其使用三参数的构造函数完成初始化过程，它也无须执行其他任务。  
接受istream&的构造函数也是委托构造函数，它委托给默认构造函数，默认构造函数函数又委托给三参数构造函数。当受委托的构造函数执行完成后，接着执行istream&构造函数体的内容。它的构造函数体调用read函数读取给定的istream。  
当一个构造函数委托给另一个构造函数时，受委托的构造函数的初始值列表和函数体被依次执行。在Sales_data类中，受委托的构造函数体恰好是空的。假如函数体包含有代码的话，将先执行这些代码，然后控制权才会交还给委托者的函数体。  

### 默认构造函数的作用  
当对象被默认初始化或值初始化时自动执行默认构造函数。默认初始化在以下情况发生：  
- 当我们在块作用域内不使用任意初始值定义一个非静态变量或者数组时。  
- 当一个类本身含有类类型的成员且使用合成的默认构造函数时。  
- 当类类型的成员没有在构造函数初始值列表中显式的初始化时。  
值初始化在以下情况发生：  
- 在数组初始化的过程中如果我们提供的初始值数量少于数组的大小时。  
- 当我们不使用初始值定义一个局部静态变量时。  
- 当我们通过书写形如T()的表达式显式的请求值初始化时，其中T是类型名。  
类必须包含一个默认构造函数以便在上述情况下使用，其中大多数情况非常容易判断。  
不那么明显的一种情况是类的某些数据成员缺少默认构造函数：
```cpp
class NoDefault{
public:
    NoDefault(const std::string&);
    // 还有其他成员，但是没有其他构造函数了
};
struct A
{  //默认情况下，my_mem是public的
    NoDefault my_eme;
};
A a; // 错误，不能为A合成构造函数
struct B
{
    B() {} //错误，b_member没有初始值
    NoDefault b_member;
};
```
**在实际中，如果定义了其他构造函数，那么最好也提供一个默认构造函数。**

### 使用默认构造函数  
下面的obj的声明可以正常编译通过：
```cpp
Sales_data obj();  // 正确：定义了一个函数而非对象
if (obj.isbn == Primer_5th_ed.isbn()) // 错误，obj是一个函数
Sales_data obj2; // 正确，obj2是一个对象而非函数。
```




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
当编译器处理dummy_fcn中的乘法表达式时，首先在函数作用域内查找表达式中用到的名字。函数的参数位于函数作用域内，因此dummy_fcn函数体内用到的名字height指的是参数声明。  
此例中，height参数隐藏了同名的成员。如果想绕开上面的查找规则1，应该将代码变为：  
```cpp
// 成员函数中的名字不应该隐藏同名成员
void Screen::dummy_fcn(pos height){
    cursor = width * this->height;
    //另一种表示该成员的方式
    cursor = width * Screen::height;
}
```
最好的确保使用height成员的方法是给参数起个其他名字：
```cpp
//建议的写法：不要把成员名字作为参数或其他局部变量使用
void Screen::dummy_fcn(pos ht){
    cursor = width * height;
}
```
在此例中，编译器查找名字height时，在dummy_fcn函数内部是找不到的。编译器接着会在Screen内查找匹配的声明，即使height的声明在dummy_fcn使用它之后，编译器也能正确地解析函数使用的是名为height的成员。
### 类作用域之后，在外围的作用域中查找  
如果编译器在函数和类的作用域中都没有找到名字，它将接着在外围的作用域中查找。在我们的例子中。名字height定义在外层作用域中。且位于Screen的定义之前。然而，外层作用域中的对象被名为height的成员隐藏掉了。因此，如果我们需要的是外层作用域中的名字，可以显式的通过作用域运算符进行请求：
```cpp
//不建议的写法，不要隐藏外层作用域中可能被用到的名字
void Screen::dummy_fcn(pos height){
    cursor = width * ::height;
}
```
### 在文件中名字出现处对其进行解析  
当成员定义在类的外部时，名字查找的第三步不仅要考虑定义之前的全局作用域中的声明，还要考虑在成员函数定义之前的全局作用域中的声明。
```cpp
int height;
class Screen{
public:
    typedef std::string::size_type pos;
    void setHeight(pos);
    pos height = 0;  //隐藏外层作用域中的height
};
Screen::pos verify(Screen::pos);
void Screen::setHeight(pos var){
    //var：参数
    //height：类的成员
    //verify：全局函数
    height = verify(var);
}
```
全局函数verify的声明在Screen的定义之前是不可见的。然而，名字查找的第三步包括了成员函数出现之前的全局作用域。在此例中，verify的声明位于setHeight的定义之前，因此可以被正常使用。

---
## 隐式的类类型转换  
如果构造函数只接受一个实参，则它实际上定义了转换为此类类型的隐式转换机制，有时我们将这种构造函数称为转换构造函数。  
**能通过一个实参调用的构造函数定义了一条从构造函数的参数类型向类类型隐式转换的规则。**  
在Saels_data类中，接受string的构造函数和接受istream的构造函数分别定义了两种类型向Sales_data隐式转换的规则。也就是说，在需要使用Sales_data的地方，我们可以使用string和istream作为替代：
```cpp
string null_book = "9-999-99999-9";
// 建立一个临时的Sales_data对象
//该对象的units_sold和revenue等于0，bookNo等于null_boook
item.combine(null_book);
```
这里我们用一个string实参调用Sales_data的combine成员。该调用是合法的，编译器用给定的string自动创建了一个Sales_data对象。新生成的这个（临时）Sales_data对象被传递给combine。

### 只允许一步类类型转换  
编译器只会自动的执行一步类型转换。例如，下边的代码隐式的使用了两种转换规则，所以它是错误的：
```cpp
// 错误，需要用户定义的两种转换：
// （1）把“9-999-99999-9”转换成string
// （2）再把临时的string转换为Sales_data
item.combine("9-999-99999-9");
```
如果想完成上述调用，可以显式的把字符串转换为string或Sales_data对象： 
```cpp
// 正确：显式的转换成string，隐式的转化成Sales_data
item.combine(string("9-999-99999-9"));
// 正确：隐式的转换为string，显式的转换为Slaes_data
item.combine(Sales_data("9-999-99999-9"));
```

### 类类型转换不是总有效
是否需要从string到Sales_data的转换依赖于我们对用户使用该转换的看法。在此例中，这种转换可能是对的。null_book中的string可能表示了一个不存在的ISBN编号。  
另一个是从istream到Sales_data的转换：
```cpp
// 使用istream构造函数创建一个函数传递给combine
item.combine(cin);
```
这段代码隐式的把cin转换成Sales_data，这个转换执行了接受一个istream的Sales_data构造函数。该构造函数通过读取标准输入创建了一个临时的Sales_data对象，随后将得到的对象传递给combine。  
Sales_data对象是个临时量，一旦combine完成我们就不能再访问它。实际上，我们构建一个对象，先将它的值加到item中，随后将其丢弃。  

### 抑制构造函数定义的隐式转换  
在要求隐式转换的程序上下文中，我们可以通过将构造函数声明为**explicit**加以阻止：
```cpp
class Sales_data{
public:
    Sales_data() = default;
    Sales_data(std::string s, unsigned cnt, double price):bookNo(s), units_sold(cnt), revenue(cnt*price){}
    explicit Sales_data(const std::string &s):bookNo(s){}
    explicit Sales_data(std::istream &is);
```
此时，没有任何构造函数能够用于隐式的创建Sales_data对象，之前的两种用法都无法通过编译：
```cpp
item.conbine(null_book); // 错误，string构造函数是explicit的
item.combine(cin); // 错误，istream构造函数是explicit的
```
关键字explicit只对一个实参的构造函数有效。需要多个实参的构造函数不能用于执行隐式转换，所以无须将这些构造函数指定为explicit的，只需要在类内声明构造函数时使用explicit关键字，在类外部定义时不用重复：
```cpp
// 错误：explicit关键字只允许出现在类内的构造函数声明处
explicit Sales_data::Sales_data(isteam& is){
    rean(is, *this);
}
```

### explicit构造函数只能用于直接初始化
发生隐式转换的一种情况是当我们执行拷贝形式的初始化时。此时，我们只能使用直接初始化，而不能使用explicit构造函数：
```cpp
Sales_data item1(null_book); // 正确，直接初始化
// 错误，不能将explicit构造函数用于拷贝形式的初始化过程诶
Sales_data item2 - null_book;
```

### 为转换显式的使用构造函数  
尽管编译器不会将explicit的构造函数用于隐式转换过程，但是可以使用这样的构造函数显式的强制进行转换：
```cpp
// 正确，实参是一个显式构造的Sales_data对象
item.combine(Sales_data(null_book));
// 正确：static_cast可以使用explicit的构造函数
item.combine(Sales_data(static_cast<Sales_data>(cin)))
```
在第一个调用中，直接使用Sales_data的构造函数，该调用通过接受string的构造函数创建了一个临时的Sales_data对象。  
在第二个调用中，使用static_cast执行了显式而非隐式的转换。其中，static_cast使用istream构造函数创建了一个临时的Sales_data对象。  

### 标准库中含有显式构造函数的类  
我们用过一些白赚苦衷的类含有单参数的构造函数：
- 接受一个单参数的const char*的string构造函数不是explicit的。  
- 接受一个容量参数的vector构造函数是explicit的。

---
## 聚合类  
聚合类使得用户可以直接访问其成员，并具有特殊的初始化语法形式。当一个类满足如下条件时，我们说它是聚合的。  
- 所有成员都是public的。
- 没有定义任何构造函数。
- 没有类内初始值。 
- 没有基类，也没有virtual函数。 
下面的类是一个聚合类。
```cpp
struct Data
{
    int ival;
    string s;
};
```
可以使用一个花括号括起来的成员初始值列表，并用它初始化聚合类的数据成员。
```cpp
// val1.ival = 0, val1.s = string("Anna")
Data val1 = {0, "Anna"};
```
初始值的顺序必须与声明的顺序一致。下面的例子是错误的：
```cpp
Data val2 = {"Anna"， 0};
```
与初始化数组元素的规则一致，如果初始值列表中的元素个数少于类的成员数量，则靠后的成员被值初始化。初始值列表的元素个数绝对不能超过类的成员数量。  
显式的初始化类的对象的成员存在三个明显的缺点：
- 要求类的所有成员都是pubic的。 
- 将正确初始化每个对象每个成员的重任交给了类的用户。
- 添加或删除一个成员之后，所有的初始化语句都需要更新。 

---
## 字面值常量  
constexpr函数的参数和返回值必须是字面值类型。除了算数类型、引用和指针外，某些值也是字面值类型。和其他类不同，字面值类型的类可能含有constexpr函数成员。这样的成员必须符合constexpr函数的所有要求，它们是隐式const的。  
数据成员都是字面值类型的聚合类是字面值常量类。如果一个类不是聚合类，但他符合下述要求，则它也是一个字面值常量类：
- 数据成员都必须是字面值类型。
- 类必须至少含有一个constexpr构造函数。
- 如果一个数据成员含有类内初始值，则内置类型成员的初始值必须是一条常量表达式；或者如果成员属于某种类类型，则初始值必须使用成员自己的constexpr构造函数。
- 类必须使用析构函数的默认的定义，该成员负责销毁类的对象。 

### constexpr构造函数  
尽管构造函数不能是const的，但是字面值常量类的构造函数可以是constexpr函数。事实上，一个字面值常量类必须至少提供一个constexpr构造函数。 
constexpr构造函数可以声明成=default的形式。否则constexpr构造函数就必须既符合构造函数的要求（不不能包含返回语句），又符合constexpr函数的要求（唯一可执行语句就是返回语句）。因此，constexpr构造函数体一般是空的。通过前置关键字constexpr就可以声明一个constexpr构造函数。
```cpp
class Debug{
public:
    constexpr Debug(bool b = true):hw(b), io(b), other(b){}
    constexpr Debug(bool h, bool i, bool o):hw(h), io(i), other(o){}
    constexpr bool any(){
        return hw || io || other;
    }
    void set_io(bool b){
        io = b;
    }
    void set_hw(bool b){
        hw = b;
    }
    void set_other(bool b){
        other = b;
    }
private:
    bool hw;
    bool io;
    bool other;
};
```
constexpr构造函数必须初始化所有数据成员，初始值或者使用constexpr构造函数，或者是一条常量表达式。  
constexpr构造函数用于生成constexpr对象以及constexpr函数的参数或返回类型。
```cpp
constexpr Debug io_sub(false, true, false); //调试io
if (io_sub.any()) // 等价于if(true)
    cerr << "print appropriate error message" << endl;
constexpr Debug prod(false); // 无调试
if (prod.any()) // 等价于if(false)
    cerr << "print an error message" << endl;
```
---

## 类的静态成员  
有的时候类需要它的一些成员与类本身直接相关，而不是和类的各个对象保持关联。例如，一个银行账户可能需要一个数据成员来表示当前的基准利率。在此例中，我们希望利率与类关联，而非和类的每个对象关联。从实现效率的角度来看，没必要每个对象都存储利率信息。而且更加重要的是，一旦利率浮动，我们希望所有的对象都使用新值。  
### 声明静态成员  
我们通过在成员的声明之前加上关键字static使得其与类关联在一起。和其他成员一样，静态成员可以是public或private的。静态成员的数据类型可以是常量、引用、指针、类类型等。
```cpp
class Account{
public:
    void calculate(){
        amount += amount * interestRate;
    }
    static double rate(){
        return interestRate;
    }
    static void rate(double);

private:
    string owner;
    double amount;
    static double interestRate;
    static double initRate();
};
```
类的静态成员存在与任何对象之外，对象中不包含任何与静态数据成员有关的数据，因此，每个Account对象将包含两个数据成员:owner和amount。只存在一个interestRate被它被所有Account对象共享。  
类似的，静态成员函数也不与任何对象绑定在一起，它们不包含this指针。作为结果，静态成员函数不能声明成const的，而且我们也不能在static函数体内使用this指针。这一限制既适用于this的显式使用，也对调用非静态成员的隐式使用有效。

### 使用类的静态成员  
我们使用作用域运算符直接访问静态成员：
```cpp
double r;
r = Account::rate(); // 使用作用域运算符访问静态成员
```
虽然静态成员不属于类的某个对象，但是我们仍然可以使用类的对象、引用或指针来访问静态成员。
```cpp
Account ac1;
Account *ac2 = &ac1;
// 调用静态成员函数rate的等价形式
r = ac1.rate(); // 通过Account的对象或引用
r = ac2 -> rate(); // 通过指向Account对象的指针
```
成员函数不用通过作用域运算符就能直接使用静态成员：
```cpp
class Account{
public:
    void calculate(){
        amount += amount * interestRate;
    }

private:
    string owner;
    double amount;
    static double interestRate;
    static double initRate();
};
```

### 定义静态成员  
和其他成员函数一样，我们既可以在类的内部也可以在类的外部定义静态成员函数。当在类的外部定义静态成员时，不能重复static关键字，该关键字只出现在类内部的声明语句。
```cpp
void Account::rate(double newRate){
    interestRate = newRate;
}
```
和类的所有成员一样，当在指向类外部的静态成员是，必须指明成员所属的类名。static关键字只出现在类内部的声明语句中。  
因为静态数据成员不属于类的任何一个对象，所以它们并不是在创建类的对象时被定义的。这意味着它们不是由类的构造函数初始化的。而且一般来说，不能再类的内部初始化静态成员。相反地，必须在类的外部定义和初始化每个静态成员。和其他对象一样，一个静态数据成员只能定义一次。  
类似于全局变量，静态数据成员定义在任何函数之外。因此一旦它被定义，就将一直存在与程序的整个生命周期中。  
定义静态数据成员和在类的外部定义成员函数差不多。需要指定对象的类型名，然后是类名、作用域运算符以及成员自己的名字。  
```cpp
//定义并初始化一个静态成员
double Account::interestRate = initRate();
```
上面的语句定义了名为interestRate的对象，该对象是Account的静态成员，其类型是double。从类名开始，这条定义语句的剩余部分就位于类的作用域之内。因此，直接使用initRate函数。  
***注意，虽然initRate是私有的，我们也可以使用它初始化interestRate。和其他成员的定义一样，interestRate的定义也可以访问类的私有成员。***  


### 静态成员的类内初始化  
通常情况下，类的静态成员不应该在类的内部初始化。然而，我们可以为静态成员提供const整数类型的类内初始值，不过要求静态成员必须是字面值常量类型的constexpr。初始值必须是常量表达式，因为这些成员本身就是常量表达式，所以它们能用在所有适合于常量表达式的地方。例如，我们可以用一个初始化了的静态数据成员指定数组成员的维度： 
```cpp
class Account{
    public:
        static double rate(){
            return interestRate;
        }
        static void rate(double);
    private:
        static constexpr int period = 30;
        double daily_tbl[period];
};
```
如果某个静态成员的应用场景仅限于编译器可以替换它的值的情况，则一个初始化的const或constexpr static不需要分别定义。相反，如果我们将它用于值不能替换的场景中，则该成员必须由一条定义语句。  
例如，如果period的唯一用途就是定义daily_tbl的维度，则不需要在Account外面专门定义period。此时，如果我们忽略了这条定义，那么对程序非常微小的改动也可能造成编译错误，因为程序找不到该成员的定义语句。举个例子，当需要把Account::period传递给一个接受const int&的函数时，必须定义period。  
如果在类的内部提供论文一个初始值，则成员的定义不能再指定一个初始值了：  
```cpp
//一个不带初始值的静态成员的定义
constexpr int Account::period;
//初始值在类的定义内提供
```
***即使一个常量静态数据成员在类内部被初始化了，通常情况下也应该在类的外部定义一下该成员。***

### 静态成员能用与某些场景，而普通成员不能  
静态成员独立于任何对象。因此，在某些非静态数据成员可能非法的场合，静态成员却可以正常的使用。举个例子，静态数据成员可以是不完全类型。特别的，静态数据成员的类型可以就是它所属的类类型。而非静态数据成员则受到限制，值能声明成它所属类的指针或引用：
```cpp
class Bar{
    public:
        //...
    private:
        static Bar meml;  //正确：静态成员可以是不完全类型
        Bar *mem2;  //正确：指针成员可以是不完全类型
        Bar mem3;  //错误：数据成员必须是完全类型 
};
```
静态成员和普通成员的另外一个区别是我们可以通过使用静态成员作为默认实参：
```cpp
class Screen{
    public:
        //bkground表示一个在类中稍后定义的静态成员
        Screen& clear(char = bkground);
    private:
        static const char bkground;
};
```
非静态数据成员不能作为默认实参，因为它的值本身属于对象的一部分，这么做的结果是无法真正提供一个对象以便从中获取成员的值，最终将引发错误。












