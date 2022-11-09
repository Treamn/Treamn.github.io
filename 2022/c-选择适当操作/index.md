# C++选择适当操作


## 其他运算符  

### 逻辑运算符  
逻辑运算符&&、||和！接受算数类型以及指针类型的运算对象。将其转换成bool类型。最后返回一个bool类型的结果。只有当逻辑上确实需要时，&&和||才会对第二个实参求值。  

### 位逻辑运算符  
位逻辑运算符&,|,^,~,>>,<<作用域整型对象，即char、short、int、long、long long以及对应的unsigned版本，以及bool、wchar_t、char16_t和char32_t等类型。一个普通的enum(非enum class)可被隐式的转换成整数类型，从而作为位逻辑运算符的运算对象。算数类型转换决定了结果的类型。  


### 条件表达式  
某些if语句可以改写成条件表达式：
```cpp
if(a <= b)
    max = b;
else
    max = a;
```
可以改写为：
```cpp
max = (a <= b) ? b : a;
```
条件表达式能用在常量表达式中。  
在条件表达式c?e1:e2中使用一堆可选的表达式e1和e2，这对表达式的类型必须相同，或者它们都能一是的转换成同一种类型T。对于算数类型来说，可使用常规的算数类型转换和规则找到公共类型T；对于其他表达式，要求e1能隐式转换成e2的类型，或者e2能隐式转换成e1的类型。此外，throw表达式也能作为条件表达式的一个分支：
```cpp
void fct(int* p)
{
    int i = (p) ? p : std::runtime_error{"unexpected nullptr};
}
```

### 递增与递减  
++和--既可以作为前缀运算符，也可以作为后缀运算符。++x的值时x的新值(递增后的值)，y=++x等价于y=(x=x+1)。相反，x++是x的旧值，y=x++等价于y=(t=x,x=x+a,t)。  
当++和--作用于指针时，将会直接操作指针所指数组中的元素，p++令p指向下一个元素。  

## 自由存储  
new分配的对象“位于自由存储之上”。 
```cpp
struct Enode{
    Token_value oper;
    Enode* left;
    Enode* right;
};
Enode* expr(bool get)
{
    Enode* left = term(get);

    for(;;){
        switch(ts.current().kind){
            case Kind::plus:
            case Kind::minus:
                left = new Enode{ts.currert().kind,left,term(true)};
                break;
            default:
                return left;
        }
    }
}
```
在Kind::plus和Kind::minus分支中，在自由存储上新建一个Enode并将其初始化为{ts.currert().kind,left,term(true)}。所得的指针赋给left并最终从expr()返回回来。  
使用{}列表的形式传递实参，也可以使用传统的()列表形式指定初始化器。但是，如果使用符号=初始化一个用new创建的对象，会引发程序错误：
```cpp
int* p = new int = 7;
```
如果某一类型含有默认构造函数，则可以省略掉初始化器。但是如果对内置类型这么做，变量就会处于未初始化的状态：
```cpp
auto pc new complex<double>; //初始化为{0,0}
auto pi = new pi; //未被初始化
```
最好使用{}，这样可以确保变量执行默认初始化：
```cpp
auto pc new complex<double>{}; //初始化为{0,0}
auto pi = new pi{}; //初始化为0
```
代码生成器先使用expr()创建的Enode，然后将其删除：
```cpp
void generate(Enode* n)
{
    switch(n->oper){
    case Kind::plus:
    //..
        delete n;
    }
}
```
对于一个用new创建的对象来说，必须使用delete显式的将它销毁，只有将其销毁，它占用的空间才能被其他new使用。   
delete运算符只能作用于new返回的指针或nullptr。  

### 内存管理  
自由存储的问题包括：
- 对象泄漏：使用new，但是忘了用delete释放掉。  
- 提前释放：在尚有其他指针指向该对象并且后续仍会使用对象的情况下过早的delete。 
- 重复释放：同一对象被释放两次。  
提前释放可能造成难以想象的影响：  
```cpp
int* p1 = new int{99};
int* p2 = p1;
delete p1; //此时p2所指的不是有效对象
p1 = nullptr;
char* p3 = new char{'x'}; //此时p3可能指向p2
*p2 = 999; // 该行可能错误
cout << *p3 << '\n'; //输出内容不是x
```
重复释放的问题在于资源管理器通常无法追踪资源的所有者： 
```cpp
void sloppy{}
{
    int* p = new int[1000];
    delete[] p;
    delete[] p;
}
```
在第二的delete时，*p的内存区域可能被重新分配，此时重新分配的内容可能受到影响。  
有两种方法可以避免上述问题：  
1. 除非万不得已不要把对象放在自由存储上，优先使用作用域内的变量。 
2. 在自由存储上构建对象时，把它的指针放在一个管理器对象中，此类对象通常含有一个析构函数，可以确保释放资源。如string，vector等标准库容器。

### 数组  
new还能用来创建对象的数组：
```cpp
char* save_string(const char* p)
{
    char* s = new char[strlen(p)+1];
    strcpy(s,p);
    return s;
}

int main(int argc, char* argv[])
{
    if(argc < 2) exit(1);
    char* p = save_string(argv[1]);
    delete[] p;
}
```
普通的delete用于删除单个对象，delete[]负责删除数组。  
除非必须直接使用char*，否则一般情况下，标准库string是更好地选择，它可以简化save_string():
```cpp
string save_string(const char* p)
{
    return string{p};
}
int main(int argc, char* argv[])
{
    if(argc < 2) exit(1);
    string s = save_string(argvp[1]);
}
```
不必再纠结new[\]和delete\[]。  
delete和delete[]必须清楚分配的对象有多大，才能准确释放new分配的空间。这意味着用new的标准实现分配的对象要比静态对象所占的空间稍大一点。超出的部分至少要容的下对象的尺寸。  
vector本身就是一个对象，因此可以使用普通的new和delete分配和释放vector：
```cpp
void f(int n){
    vector<int>* p = new vector<int>(n);
    int* 1 = mew int[n];
    delete p;
    delete[] q;
}
```
delete[]只能用于两种情况，一种是指向由new创建的数组的指针，另一种是空指针。  
不要用new创建局部对象：
```cpp
void f1{}
{
    X* p = new X;
    delete p;
}
```

### 获取内存空间   
自由存储运算符new、delete、new[\]和delete\[]的实现位于\<new>头文件中：
```cpp
void* operator new(size_t); //为单个对象分配空间
void* operator delete(void* p); //如果p为真，释放new()分配的全部空间
void* operator new[](size_t); //为数组分配空间
void operator delete[](void* p); //如果p为真，释放new[]()分配的所有空间
```
当运算符new需要为对象分配空间时，调用operator new()分配适量的字节。

### 重载new  
默认情况下，new运算符在自由存储上创建它的对象。如果想在别的地方分配对象：
```cpp
class X{
    public:
        X(int);
};
```
如果想把对象放置在别的地方，可以提供一个含有额外实参的分配函数，然后在使用new的时候传入指定的额外实参：
```cpp
void* operator new(size_t,void* p){return p;} //显式运算符，将对象置于别处

void* buf = reinterpret_cast<void*>(0xF00F); //一个明确的地址
X* p2 = new(buf) X; //在buf处构建X
```
通常把提供额外的实参给operator new()的new(buf) X语法称为放置语法。每个operator new()都接受一个尺寸作为它的第一个实参，而该尺寸的对象时隐式提供的。每个perator new()都以size_t作为它的第一个实参：
```cpp
void* operator new(size_t sz,void* p) noexcept; //将大小为sz的对象置于p处
void* operator new[](size_t sz,void* p) noexcept; //将大小为sz的对象置于p处

void* operator delete(void* p,void*) noexcept; //若p为真，令*p无效
void* operator delete[](void* p,void*) noexcept; //若p为真，令*p无效
```
放置式new还能从某一特定区域分配内存：
```cpp
class Arena{
public:
    virtual void* alloc(size_t) = 0;
    virtual void free(void*) = 0;
};

void operator new(size_t sz, Arena* a)
{
    return a -> alloc(sz);
}
```
现在能在不同的Arena里分配任意类型的对象：
```cpp
extern Arena* Persistent;
extern Arena* Shared;

void g(int i){
    X* p = new(Persistent) X(i); //在某持续性存储上分配X
    X* 1 = new(Shared) X(i); //在共享内存上分配X
}
```

#### nothrow new  
有的程序不允许出现异常，此时，可以使用nothrow的new和delete：
```cpp
void f(int n){
    int* p = new(nothrow) int[n];
    if(p == nullptr){
        //...
    }
    operator delete(nothrow,p); //释放*p
}
```
如果没能分配到有效的内存，上面的operator new不会抛出bad_alloc，而是返回一个nullptr。  

## 列表  
{}列表的表现形式有两种：  
1. 限定为某种类型，形如T{...}，意为创建一个T类型的对象并用T{...}初始化它。   
2. 未限定的{...}，类型根据上下文确定。 


### 实现模型   
{}列表的实现模型由三部分组成：  
- 如果{}列表被用作构造函数的实参，则其实现过程与使用()列表类似。
- 如果{}列表被用于初始化一个聚合体(一个数组或一个为提供构造函数的类)的元素，则列表的每个元素分别初始化聚合体中的一个元素。  
- 如果{}列表被用于构建一个initializer_list对象，则列表的每个元素分别初始化initializer_list的底层数组的一个元素。  

### 限定列表  
把初始化器列表用作表达式的基本思想是：如果能用T x{v};初始化一个变量x，那么也能用T{v}或new T{v}的形式创建一个对象并将其当成一条表达式。  
```cpp
struct S{int a,b;};

void f(){
    S v{7,8}; //直接初始化一个变量
    v = S{7,8}; //用限定列表进行赋值
    S* p = new S{7,8}; //使用限定列表在自由存储上构建对象
}
```
使用限定列表构建对象与直接初始化的规则相同。  

### 未限定列表  
当我们明确知道所用类型时，可以使用未限定列表。它只能被用作一条表达式，并且仅限于以下场景：
- 函数实参
- 返回值
- 赋值运算符的右侧运算对象 
- 下标
例如：
```cpp
int f(double d, Matrix& m)
{
    int v{7}; //直接初始化
    int v2 = {7}; //拷贝初始化
    int v3 = m[{2,3}]; //假设m接受一个值作为其下标

    v = {8}; //赋值运算的右侧运算对象
    v += {88}; //赋值运算的右侧运算对象
    {v} = 9; //错误，不能作为赋值运算的左侧运算对象
    v = 7+{10}; //错误，不能作为赋值运算的左侧运算对象
    f({10.0}); //函数实参
    return {11}; //返回值
}
```
{}列表是处理同质、变长列表最简单的方法，但是注意0个元素的情况是例外。此时，应该使用默认的构造函数。  
只有当{}列表的所有元素类型相同时，才能推断该列表的类型：
```cpp
auto x0 = {}; //错误，缺少元素
auto x1 = {1}; //initializer_list<int>
auto x2 = {1,2}; //initializer_list<int>
auto x3 = {1,2,3}; //initializer_list<int>
auto x4 = {1,2.0}; //错误，元素类型不相同
```

## lambda表达式   
lambda表达式也称未lambda函数。是定义和使用匿名函数的一种简便方式。   
一条lambda表达式包含以下组成要件：
- 一个可能为空的捕获列表，指明定义环境中哪些名字能被用在lambda表达式内，以及这些名字的访问方式是拷贝还是引用。 
- 一个可选的参数列表，指明lamdba表达式所需的参数。 
- 一个可选的mutable修饰符，指明该lamdba表达式可能会修改它自身的状态。
- 一个可选的noexcept修饰符。 
- 一个可选的->形式的返回类型声明。 
- 一个表达式体，指明要执行的代码。  


### 实现模型  
如果把lambda表达式看成是一种定义并使用函数对象的便捷方式：
```cpp
void print_modulo(const vector<int>& v,ostream& os, int m)
{
    for_each(begin(v),end(v),
        [&os,m](int x){if(x%m==0) os << x << '\n';}
        );
}
```
上述函数等价于
```cpp
class Modulo_print{
    ostream& os;
    int m;
public:
    Modulo_print(ostream& s, int mm:os(s),m(mm)){}
    void operator(int x)const
        {if(x%m==0) os << x << '\n';}
};
```
lambda的主体部分变为了operator()()的函数体。因为lambda并不返回值，所以operator()()是void。默认情况下，operator()()是const，因此在lambda内部无法修改捕获的变量。  
把由lambda生成的类的对象称为闭包对象：
```cpp
void print_modulo(const vector<int>& v,ostream& os, int m)
{
    for_each(begin(v),end(v),Modulo_print{os,m});
}
```
如果lambda通过引用捕获它的每个局部变量，则其闭包对象可以油画成简单地包含一个指向外层栈框架的指针。  

### lambda的替代品  
很多lambda表达式很小且只用一次。此时，最一种比较现实的做法是定义一个局部类，并且定义的位置就在使用之前：
```cpp
void print_modulo(const vector<int>& v,ostream& os, int m)
{
class Modulo_print{
    ostream& os;
    int m;
public:
    Modulo_print(ostream& s, int mm:os(s),m(mm)){}
    void operator(int x)const
        {if(x%m==0) os << x << '\n';} 
};
for_each(begin(v),end(v),Modulo_print{os,m});
}
```
与之相比，使用lambda明显更优，如果确需一个名字，就命名为lambda：
```cpp
void print_modulo(const vector<int>& v,ostream& os, int m)
{
    auto Modulo_print = [&os,m](int x){if(x%m==0) os << x << '\n';};
    for_each(begin(v),end(v),Modulo_print);
}
```
可泛化print_modulo()，令其可以处理更多的容器类型：  
```cpp
template<class C>
void print_modulo(const C& v, ostream& os, int m)
{
    for(auto x:v)
        if(x %m == 0) os << x << '\n';
}
```

### 捕获  
lambda的主要用途是封装一部分代码以便将其用作参数。lambda允许我们“内联的”这么做，而无须命名一个函数然后在别处使用它。有些lambda无须访问它的局部环境，这样的lambda使用空引入符[]定义:
```cpp
void algo(vector<int>& v)
{
    sort(v.begin(),v.end()); //排列值
    sort(v.begin(),v.end(),[](int x,int y){return abs(x) < ans(y);}); //排列绝对值
}
```
如果需要访问局部名字，就必须明确指出，否则会产生错误： 
```cpp
void f(vector<int>& v)
{
    bool sensitive = true;
    sort(v.begin(),v.end(),)
        [](int x,int y){return sensitive ? x < y : abs(x) < ans(y);}  // 错误，无权访问
    ;
}
```
对于一条lambda表达式来说，它的第一个字符永远是[。lambda引入符的形式有很多种：
- []：空捕获列表。lambda内部无法使用其外层上下文中的任何局部名字。  
- [&]：通过引用隐式捕获。所有局部名字都能使用，所有局部变量都通过引用访问。 
- [=]：通过值隐式捕获。所有局部名字都能使用，所有名字都指向局部变量的副本。
- [捕获列表]：显式捕获；捕获列表是通过值或引用的方式捕获的局部变量的名字列表。以&为前缀的变量名字通过引用捕获，其他变量通过值捕获，捕获列表中可以出现this，或紧跟...的名字以表示元素。  
- [&,捕获列表]：对于名字没有出现在捕获列表中的局部变量，通过引用隐式捕获。捕获列表中可以出现this。列出的名字不能以&为前缀。捕获列表中的变量名通过值的方式捕获。  
- [=,捕获列表]：对于名字没有出现在捕获列表中的局部变量，通过值隐式捕获。捕获列表中可以出现this。列出的名字不能以&为前缀。捕获列表中的变量名通过值的方式捕获。  

以&为前缀的局部名字总是通过引用捕获，相反，不义&为前缀的局部名字总是通过值捕获。只有通过引用的捕获允许修改调用环境中的变量。  


#### lambda与生命周期  
lambda的生命周期可能比它的调用者长。当把lambda传递给另外一个线程或者被调用者把lambda存在别处以供后续使用时，这种情况就会发生：  
```cpp
void setup(Menu& m)
{
    Point p1,p2,p3;
    m.add("draw triangle",[&]{m.draw(p1,p2,p3)};); 
}
```
如果lambda的生命周期可能比调用者长，就必须确保所有局部信息都被拷贝到闭包对象中，并且这些值应该通过return机制或者适当的实参返回：
```cpp
m.add("draw triangle",[=]{m.draw(p1,p2,p3)};);
```

#### 名字空间名字  
因为名字空间变量(包括全局变量)永远是可访问的，所以无须“捕获”它们。

#### lambda与this  
当lambda被用在成员函数中时，我们将this添加到捕获列表中，这样类的成员就位于可被捕获的名字集合中了：
```cpp
class Request{
    function<map<string,string>(const map<string,string>&)> oper; //操作
    map<string,string> values; //参数
    map<string,strign> results; //结果
public:
    Request(const string& s);
    void execute()
    {
        [this](){return=oper(values);} //根据结果执行相应操作
    }
};
```
成员通过引用的方式捕获。也就是说，[this]意味着成员是通过this访问，而非拷贝到lambda中。  

#### mutable的lambda  
生成的函数对象的operator()()是一个const成员函数。只有在极少数情况下，如果确实希望修改状态，则可以把lambda声明成mutable的：
```cpp
void algo(vector<int>& v)
{
    int count = v.size();
    std::generate(v.begin(),v.end(),
        [count]()mutable{return --count;}
    );
}
```

### 调用和返回  
向lambda传递参数的规则和向函数传递参数是一样的，从lambda返回结果也是如此。有两点需要注意：
1. 如果一条lambda表达式不接任何参数，则其参数列表可以被忽略。因此，lambda表达式的最简形式是[]{}。  
2. lambda表达式的返回类型能由lambda表达式本身推断得到。  

如果在lambda的主体部分不包含return语句，则该lambda的返回类型是void。如果lambda的主体部分只包含一条return语句，则该lambda的返回类型宁国市该return表达式的类型。其他情况下，必须显式的提供一个返回类型：
```cpp
void g(double y)
{
    [&]{f(y);} //返回类型是void
    auto z1 = [=](int x){return x+y;} //返回类型是double
    auto z2 = [=,y]{if (y) return 1;else return 2;} //错误，lambda主体部分过于复杂，无法推断其类型
    auto z3 = [y](){return 1:2;} // 返回类型是int
    auto z4 = [=,y]()->int{if (y) return 1;else return 2;} //OK，显式的返回类型
}
```

### lambda的类型  
任意两个lambda类型都不相同，一旦两个lambda具有相同的类型，模板实例化机制就无法辨识它们了。lambda是一种局部类类型，含有一个构造函数以及一个const成员函数operator()()。lambda除了能作为参数外，还能用于初始化一个声明为auto或std::function<R(AL)>的变量。其中R是lambda的返回类型，AL是它的类型参数列表。  
例如，编写一个能更改C风格字符串中字符的lambda：
```cpp
auto rev = [&rev](char* b, char* e){
    if(1<e-b){swap(*b,*--e);rev(++b,e);}
};
```
然而，由于无法推断出一个auto变量的类型之前使用它，因此上面的写法行不通。相反，应该先引入一个新名字，然后使用它：
```cpp
void f(string& s1, string& s2)
{
    function<void(char* b,char* e)> rev = [&](char* b,char* e){
    if(1<e-b){swap(*b,*--e);rev(++b,e);}};
    rev(&s1[0],&s1[0]+s1.size());
    rev(&s2[0],&s2[0]+s2.size());
}
```
这样就可以确保在使用rev之前知道它的类型。  
如果只想给lambda起个名字，而不递归的使用它，则可以考虑使用auto：
```cpp
void f(vector<string>& vs1, vector<string>& vs2)
{
    auto rev = [&](char* b,char* e){
    if(1<e-b){swap(*b,*--e);rev(++b,e);}};
    rev(&s1[0],&s1[0]+s1.size());
    rev(&s2[0],&s2[0]+s2.size());
}
```
如果一个lambda什么也不捕获，则可以将它赋值给一个指向正确类型的指针：
```cpp
double(*p1)(double) = [](double a){return sqrt(a);};
double(*p2)(double) = [&](double a){return sqrt(a);}; //错误，lambda捕获了内容
double(*p3)(int) = [](int a){return sqrt(a);}; //错误，参数类型不匹配
```


## 显式类型转换
C++提供多种显式类型转换的操作，这些操作在便利程度以及安全性上都有所不同：
- 构造，使用{}符号提供对新值类型安全的构造。 
- 命名的转换，提供不同等级的类型转换：
> - const_cast,对某些声明为const的对象获得写入的权利
> - static_cast，反转一个定义良好的隐式类型转换
> - reinterpret_cast，改变位模式的含义
> - dynamic_cast，动态的检查类层次的关系
- C风格的转换，提供命名的类型转换或其组合
- 函数化符号，提供C风格转换的另一种形式 


### 构造  
用值e构建一个类型为T的值可以表示为T{e}：
```cpp
auto d1 = double{2};
double d2 {double{2}/4};
```
符号T{v}有一个好处是，它只执行“行为良好的”；类型转换：
```cpp
void f(int);
void f(double);

void g(int i, double d)
{
    f(i); //调用f(int)
    f(double{i}); //错误，{}拒绝执行整数向浮点数的类型转换

    f(d); //调用f(double)
    f(int{d}); //错误，{}拒绝截断的行为
    f(static_cast<int>(d)); //调用f(int)，传入的是个截断的值

    f(round(d)); //调用f(double)，传入的是个四舍五入的值
    f(static_cast<int>(lround(d))); //调用f(int)，传入的是个四舍五入的值
    // 如果round(d)溢出int的范围，它仍会被截断
}
```
一旦在程序中使用目标类型作为显式的限定，则在这种情况下就不允许行为不正常的类型转换：
```cpp
void g2(char* p)
{
    int x = int{p}; //错误，不存在char*向int的类型转换
    using Pint = int*;
    int* p2 = Pint{p}; //错误，不存在char*向int*的类型转换
}
```
对T{v}来说，“行为非常良好”的含义是存在v向T的“非窄化”类型转换或者有一个T的类型正确的构造函数。   
构造函数符号T{}用于表示类型T的默认值：
```cpp
template<class T> void f(const T&);

void g3()
{
    f(int{}); //默认的int值
    f(complex<double>{}); 默认的complex值
}
```
对于内置类型来说，显式的使用其构造函数得到的值是该类型对应的0值。因此，int{}可以看成是0的另一种写法。对用户自定义类型T来说，如果含有默认构造函数，则T{}的结果由默认构造函数定义；否则，由每个成员的默认构造函数MT{}定义。

### 命名转换  
- static_cast执行关联类型之间的转换，比如一种指针类型向同一个类层次中其他指针类型的转换，或者整数类型向枚举类型的转换，或浮点类型向整数类型的转换。还能执行构造函数和转换运算符定义的类型转换。
- reinterpret_cast处理非关联类型之间的转换，比如整数向指针的转换以及指针向另一个非关联指针类型的转换。
- const_cast，参与转换的类型尽在const修饰符及volstile修饰符上有所区别。
- dynamic_cast执行指针或者引用向类层次体系的类型转换，并在运行时检查。

**在决定使用显式类型转换之前，清华时间仔细考虑一下是否真的必须这么做。**

### 函数形式的转换  
用值e构建类型T的值的过程可以表示为函数形式的符号T(e):
```cpp
void f(double d)
{
    int i = int(d); //截断
    complex z = complex(d); //从d构建一个coplex
}
```
T(e)有时称为函数形式的转换，不幸的是，对于内置类型T来说，T(e)等价于(T)e，这意味着对大多数内置类型来说，T(e)并不安全：
```cpp
void f(double d, char* p)
{
    int a = int(d); //截断
    int b = int(p); //不可移植
}
``
即使是从一种较长的整数类型向较短的整数类型的显式转换也会导致不可移植的依赖于实现的行为。



