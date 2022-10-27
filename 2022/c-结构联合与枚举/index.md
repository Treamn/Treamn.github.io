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
Points x1{{100,200}}; //一个Point
Points x1{{100,200},{300,400}}; //两个Point
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

Point points[3]{{1,2},{3,4},{5,6}};
int x2 = points[2].x;

struct Array{
    Point elem[3];
};

Array points2 {{1,2},{3,4},{5,6}};
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


