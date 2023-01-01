# C++结构联合与枚举


## 结构  
数组是相同类型的集合，相反，struct是任意类型元素的集合：
```cpp
struct Address{
    const char* name;
    int number;
    const char* street;
    const char* town;
    char state[2];
    const char* zip;
};
```
声明Address类型的变量和声明其他类型变量的方式一模一样，可以用点运算符(.)为每个成员分别赋值：
```cpp
void f(){
    Address jd;
    jd.name = "Jim Dandy";
    jd.number = 61;
}
```
struct类型的变量能使用{}的形式初始化：
```cpp
Address jd = {
    "Jim Dandy",
    61,
    "South ST",
    "New Providence",
    {'N','J'},
    "07974"
};
```
不能用字符串"NJ"初始化jd.state。字符串以符号'\0'结尾，因此“NJ”实际上包含3个字符。   
通常使用->运算符访问结构的内容：
```cpp
void print_addr(Address* p){
    cout << p->name;
}
```
如果p是一个指针，则p->m等价于(*p).m。  
除此之外，也能以引用的方式传递struct，并使用.运算符访问它：
```cpp
void print_addr2(const Address& r){
    cout << p.name;
}
```
结构类型的对象可以被赋值、作为实参传入函数，或者作为函数的结果返回：
```cpp
Address current;

Address set_current(Address next){
    address prev = current;
    current = next;
    return prev;
}
```

### struce的名字  
类型名字只要一出现就能马上使用了，无须等到该类型的声明全部完成：
```cpp
struct Link{
    Link* previous;
};
```
但是，只有等到struct的声明全部完成，才能声明它的对象：
```cpp
struct No_good{
    No_good member; //错误，递归定义
};
```
要想让两个或更多struct互相引用，必须提前声明好struct的名字：
```cpp
struct List;

struct Link{
    Link* pre;
    Link* suc;
    List* member_of;
    int data;
};

struct List{
    Link* head;
};
```
如果没有一开始声明List，则在稍后声明Link时使用List*类型的指针将造成错误。  
可以在真正定义一个struct类型之前就使用它的名字，只要在此过程中不使用成员的名字和结构的大小就行。然而，直到struct的声明全部完成之前，它都是一个不完整的类型。

### 结构与类  
struct是一种class，它的成员默认是public的。struct可以包含成员函数，尤其是构造函数：
```cpp
struct Points{
    vector<Point> elem;
    Points(Point p0){elem.push_back(p0);}
    Points(Point p0, Points p1){elem.push_back(p0);
    elem.push_back(p1);}
}

Points x0; //错误，缺少默认构造函数
// ....
```
如果只是按照默认的顺序初始化结构的成员，则不需要专门定义一个构造函数：
```cpp
struct Points{
    int x, y;
}

Points p0; //如果位于局部作用域中，则p0未初始化
Points p1{}; 
Points p2{1};
Points p3{1,2};
```
如果需要改变实参的顺序、检验实参的有效性、修改实参或者建立不变式，则应该编写一个专门的构造函数。
```cpp
struct Address{
    const char* name;
    int number;
    const char* street;
    const char* town;
    char state[2];
    const char* zip;

    Address(const string n, int nu, const string& s, const string& t, const string& st, int z);
};
```

### 结构与数组  
可以构建struct的数组，也可以让struct包含数组：
```cpp
struct Point{
    int x,y;
};


int x2 = points[2].x;

struct Array{
    Point elem[3];
};


int y2 = points2.elem[2].y;
```
把内置数组置于struct内部意味着我们可以把数组当成一个对象来用，可以在初始化和赋值时直接拷贝struct：
```cpp
Array shift(Array a, Point b){
    for(int i =0 ; i != 3; ++i){
        a.elem[i].x += p.x;
        a.elem[i].y += p.y;
    }
    return a;
}
```

### 类型等价  
对两个struct来说，即使它们的成员相同，它们本身仍是不同的类型。   


## 联合   
union是一种特殊的struct，它的所有成员都分配在同一个地址空间上。因此，一个union实际占用的空间大小与其最大的成员一样。自然的，在同一时刻union只能保存一个成员的值：
```cpp
enum Type{str, num};

struct Entry{
    char* name;
    Type t;
    char* s; //如果t==str，使用s
    int i; //如果t==num，使用i
};

void f(Entry* p){
    if(p->t == str)
        cout << p->s;
}
```
在此例中，成员s和i永远不会被同时使用，因此空间被浪费了。我们可以把它们指定成union的成员，解决上述问题：
```cpp
union Value{
    char* s;
    int i;
};
```

### 枚举  
枚举类型用于存放用户指定的一组整数值。枚举类型的每种取值各自对应一个名字，我们把这些值叫做枚举值：
```cpp
enum class Color{red, green, blue};
```
上述代码定义了一个名为Color的枚举类型，它的枚举值时red、green和blue。"一个枚举"简称”一个enum“。
枚举类型分为两种：  
1. enum class，它的枚举值名字（如red）位于enum的局部作用域内，枚举值不会隐式的转换成其他类型。 
2. ”普通的enum“，它的枚举值名字与枚举类型本身位于同一个作用域中，枚举值隐式的转换成整数。  

通常情况下，建议使用enum class，它很少会产生意料之外的结果。  

### enum class 

enum class是一种限定了作用域的强类型枚举：
```cpp
enum class Traffic_light{red, yellow, green};
enum class Warning{green,yellow,orange,red};

Warning a1 = 7; //错误：不存在int向Warning的类型转换
int a2 = green; //错误：green位于它的作用域之外
int a3 = Warning::green; //错误：不存在Warning向int的类型转换
Warning a4 = Warning::green;

void f(Traffic_light x){
    if(x == 9) //错误，9不是一个Traffic_light
    if(x == red) //错误，当前作用域中没有red
    if(x == Warning::red) //错误：x不是一个Warning
    if(x == Traffic_light::red)
}
```
两个enum的枚举值不会互相冲突，它们位于各自enum class的作用域中。  
枚举常用一些整数类型表示，每个枚举值是一个整数。我们把用于某个枚举的类型称为它的基础类型。基础类型必须是一种带符号或无符号的整数类型，默认是int，可以显式的指定：  
```cpp
enum class Warning:int{green,yellow,orange,red}; //sizeof(Warning) == sizeof(int)
```
如果认为上述定义太浪费空间，可以用char代替int：
```cpp
enum class Warning:char{green,yellow,orange,red}; //sizeof(Warning) == 1
```
默认情况下，枚举值从0开始，依次递增：
```cpp
static_cast<int>(Warning::green)==0
static_cast<int>(Warning::yellow)==1
static_cast<int>(Warning::orange)==2
static_cast<int>(Warning::red)==3
```
有了Warning之后，用Warning变量代替普通的int变量使得用户和编译器都能更好地了解该变量的真正用途： 
```cpp
void f(Warning key)
{
    switch(key){
    case Warning::green:
    //...
    case Warning::orange:
    //...
    case Warning::red:
    //...
        break;
    }
}
```
可以用整型常量表达式初始化枚举值：
```cpp
enum class Print_flags{
    ackonwledge=1;
    paper_empty=2;
    busy=4;
    out_of_black=8;
    out_of_color=16;
};
```
enum属于用户自定义的类型，因此可以为它定义|和&运算符：
```cpp
constexpr Printer_flags operator|(Print_flags a, Print_flags b){
    return static_cast<Print_flags>(static_cast<int>(a)|static_cast<int>(b))
}

constexpr Printer_flags operator&(Print_flags a, Print_flags b){
    return static_cast<Print_flags>(static_cast<int>(a)&static_cast<int>(b))
}
```
因为enum class不知吃隐式类型转换，所以必须在这里使用显式的类型转换：
```cpp
void try_to_print(Printer_flags x)
{
    if(x&Printer_flags::ackonwledge){
        //...
    }
    else if(x&Printer_flags::busy){
        //...
    }
    else if(x&(Printer_flags::out_of_black | Printer_flags::out_of_color))
}
```
将operator}()和operator&()定义成constexpr函数，这样就能用于常量表达式：
```cpp
void g(Printer_flags x){
    switch(x){
        casse Printer_flags::out_of_black | Printer_flags::out_of_color:
        //...
        break;
    }
}
```
C++允许先声明一个enum class，稍后再给出定义：
```cpp
enum class Color_code:char; //声明
void foobar(Color_code* p); //使用声明
//...
enum class Color_code:char{ //定义
    ren,yellow,green,blue
};
```
一个整数类型的值可以显式的转换成枚举类型。如果这个值属于枚举的基础类型的取值范围，则转换是有效的；否则，如果超出了合理的表示范围，则转换的结果是未定义的:
```cpp
enum class Flag:char{x=1,y=2,z=4,e=8};

Flag f0{}; //f0默认值是0
Flag f1 = 5; //类型错误，5不属于Flag
Flag f2 = Flag{5}; //错误。不许窄化成enum class
Flag f3 = static_cast<Flag>(5); //不近人情的转换
Flag f4 = static_cast<Flag>(999); //错误，999不是一个char类型的值
```
因为绝大多数整数值根本不再某一枚举类型的合理表示范围之内，因此不允许从整数到枚举类型的隐式转换。  
每个枚举值对应一个整数，我们可以显式的把整数抽取出来：
```cpp
int i = static_cast<int>(Flag::y); //i==2
char c = static_cast<char>(Flag::e); //c == 8
```
对enum class执行sizeof的结果是对其基础类型执行sizeof的结果，如果没有显式的指定基础类型，则枚举类型的尺寸等于sizeof(int)。

### 普通的enum  
“普通的enum”是指C++在提出enum class之前提供的枚举类型。普通的enum的枚举值位于enum本身所在的作用域中，它们隐式的转换成某些整数类型的值：
```cpp
enum Traffic_light{red, yellow, green};
enum Warning{green,yellow,orange,red};

//错误：yellow被重复定义(取值相同)
//错误：red被重复定义(取值不同)

Warning a1 = 7; //错误：不存在int向Warning的类型转换
int a2 = green; //ok：green位于其作用域中，隐式转换成int类型
int a3 = Warning::green; //OK：Warning向int的类型转换
Warning a4 = Warning::green; //OK

void f(Traffic_light x){
    if(x == 9) //OK(d但是Traffic_light不包含枚举值9)
    if(x == red) //错误，当前作用域中有两个red
    if(x == Warning::red) //OK
    if(x == Traffic_light::red)
}
```
我们在同一个作用域的两个普通枚举中都定义了red，从而避免一个难以发现的错误。可以人为地消除枚举值的二义性，以实现对于普通enum的“清理”：
```cpp
enum Traffic_light{tl_red, tl_yellow, tl_green};
enum Warning{green,yellow,orange,red};

void f(Traffic_light x){
    if(x == 9) //OK
    if(x == red) //OK
    if(x == Traffic_light::red)
} //错误：red不是一个Traffic_light类型的值   
```
从编译器角度来看，x==red是合法的，但它几乎肯定是一个程序缺陷。把名字注入外层作用域(当使用enum时会发生这种情况，但是使用enum class和class不会)的行为称为名字空间污染，在规模较大的程序中应避免这么做。   
可以为普通的枚举指定基础类型，就像对enum class所做的一样。此时，允许先声明枚举类型，稍后再给出定义：  
```cpp
enum Traffic_light:char{tl_red, tl_yellow, tl_green}; //基础类型是char
enum Color_code:char; //声明
void foobar(Color_code* p); //使用声明
enum Color_code:char{green,yellow,orange,red}; //定义  
```
如果没有指定枚举的基础类型，则不能把它的声明和定义分开。

### 未命名的enum  
一个普通的enum可以是未命名的：
```cpp
enum{arrow_up=1, arrow_down, arrow_sideways};
```
如果只需要一组整型变量，而不是用于声明变量的类型，则可以使用未命名的enum。

