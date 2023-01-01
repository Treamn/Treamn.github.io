# C++函数


## 函数声明  
函数声明负责指定函数的名字、返回值的类型以及调用该函数所需的参数数量和类型：
```cpp
Elem* next_elem(); //无须参数，返回Elem*
void exit(int); //int类型的参数，无返回值
double sqrt(double); //double类型的参数，返回double
```

### 函数声明的组成要件  
函数声明除了指定函数的名字、一组参数以及函数的返回类型外，还包括多种限定符和修饰符：
- 函数的名字，必选
- 参数列表，可以为空，必选
- 返回类型，可以是void，可以是前置或后置形式(使用auto)，必选
- inline，表示一种愿望，通过内联函数体实现函数调用 
- constexpr，表示当给定常量表达式作为实参时，应该可以在编译时对函数求值  
- noexcept，表示该函数不允许抛出异常
- 链接说明，例如static
- [[noreturn]]，表示该函数不会用常规的调用/返回机制返回结果

此外，成员函数还能被限定为：
- virtual，表示该函数可以被派生类覆盖 
- override，表示该函数必须覆盖基类中的一个虚函数
- final，表示该函数不能被派生类覆盖
- static，表示函数不与某一特定的对象关联
- const，表示该函数不能修改其对象的内容  


### 函数定义  
函数的定义和全部声明必须对应同一类型。不过，为了与C语言兼容，会自动忽略参数类型的顶层const，下面两条声明语句对应的是同一个函数：
```cpp
void f(int); //类型是void(int)
void f(const int); //类型是void(int)
```
函数f()可以定义成：
```cpp
void f(int x); //允许在此处修改x
```
或者定义成
```cpp
void f(const int x); //不允许在此处修改x
```
不论对哪种情况而言，都只是函数调用者提供的实参的一个副本。  
通常通过不命名某个参数来表示该参数未在函数定义中使用：
```cpp
void search(table* t, const char* key, const char*){
    //未用到第三个参数
}
```
一般来说，未命名的参数有助于简化代码并提升代码的可扩展性。  
除了函数之外，还能调用其他一些东西，它们遵循函数的大多数规则，比如参数传递规则：  
- 构造函数：严格来说不是函数，它没有返回值，可以初始化基类和成员，无法得到其地址。
- 析构函数：不能被重载，无法得到其地址。
- 函数对象：不是函数，不能被重载，但是其operaotr()是函数。
- lambda表达式：是定义函数对象的一种简写形式。  

### 返回值  
传统上，返回类型位于函数声明语句一开始的地方。然而，也可以在函数声明中把返回类型写在参数列表之后：
```cpp
string to_string(int a);
auto to_string(int a)->string;
```
前置的auto关键字表示函数的返回类型放在参数列表之后。后置返回类型则由符号->引导。 
后置返回类型的必要性源于函数模板声明，因为其返回类型是依赖于参数的：
```cpp
template<class T,class U>
auto product(const vector<T>& x,const vector<U>& y) -> decltype(x*y);
```
函数如果调用了它自身，称之为递归。
函数可以包含多条reutrn语句：
```cpp
int fac2(int n)
{
    if(n > 1)
        return n*fac2(n-1);
    return 1;
}
```
与参数传递的语义类似，函数返回值的语义也与拷贝初始化的语义一致。return语句初始化一个返回类型的变量，编译器检查返回表达式的类型是否与函数的返回类型吻合，并在必要时执行标准的或者用户自定义的类型转换：
```cpp
double f(){return 1;}
```
每次调用函数时，重新分配它的实参以及局部变量的拷贝。一旦函数返回了结果，所占的存储空间就被重新分配了。因此，不应该返回指向局部而非static变量的指针，我们无法预计该指针所指位置的内容将发生什么样的改变：
```cpp
int* fp()
{
    int local = 1;
    return &local;
}
```
引用有时也会发生类似的错误：
```cpp
int& fr()
{
    int local = 1;
    return local;
}
```
并不存在void值，不过，可以调用void函数令其作为另一个void函数的返回值：
```cpp
void g(int* p);
void h(int* p)
{
    return g(p); //OK，等价与“g(p)；return；”
}
```
当编写模板函数返回类型是模板参数时，这种返回类型有助于避免某些特殊情况。  
return语句的形式属于下述5种之一：
- 执行一条return语句。
- “跳转到函数末尾”，也就是说，直接到达函数体末端。这种情况只允许出现在无返回值的函数中。 
- 抛出一个未被局部捕获的异常。 
- 在一个noexcept函数中抛出一个异常且没有局部捕获，造成程序终止。
- 直接或间接请求一个无返回值的系统函数。


### inline函数  
函数可以被定义为inline：
```cpp
inline int fac(int n){
    return(n<2)?1:n*fac(n-1);
}
```

## constexpr函数  
通常，函数无法在编译时求值，因此就不能在常量表达式中被调用。但是将函数指定为constexpr，就能向编译器传递这样的信息，即，如果给定了常量表达式作为实参，则希望该函数能被用在常量表达式中：
```cpp
constexpr int fac(int n)
{
    return(n > 1)? n*fac(n-1) : 1;
}
constexpr int f9 = fac(9);
```
当constexpr出现在函数定义中时，含义是“如果给定了常量表达式作为实参，则该函数应该能用在常量表达式中”。而当constexpr出现在对象定义中时，它的含义是“在编译时对初始化器求值”：
```cpp
void f(int n)
{
    int f5 = fac(5); //可能在编译时求值
    int fn = fac(n); //在运行时求值（n是变量）

    constexpr int f6 = fac(6); //必须在编译时求值
    constexpr int fnn = fac(n); //错误，无法确保在编译时求值（n是变量）

    char a[fac(4)]; //OK，数组的尺寸必须是常量，而fac()恰好是constexpr
    char a2[fac(n)]; //错误，数组的尺寸必须是常量，而n是一个变量
}
```
函数必须足够简单才能在编译时求值：constexpr函数必须包含一条独立的return语句，没有循环，也没有局部变量。同时，constexpr函数不能有副作用。也就是说，constexpr函数应该是一个纯函数：
```cpp
int glob;

constexpr void bad1(int a)
{ //错误，constexpr函数不能是void
    glob = a; //错误，在constexpr函数中有副作用
}

constexpr int bad2(int a)
{
    if(a >= 0) return a;
    else return -a; //错误，在constexpr函数中有if语句
}

constexpr int bad3(int a)
{
    sum = 0; //错误，在constexpr函数中有局部变量
    for(int i = 0; i < a; +=i) //错误，在constexpr函数中有循环
        sum += fac(i);
    return sum;
}
```
与普通的constexpr函数相比，constexpr构造函数的规则有所区别：只允许简单地执行成员初始化操作。  


#### constexpr与引用  
constexpr函数不允许有副作用，因此不能向非局部对象写入内容：
```cpp
constexpr int ftbl[]{1,2,3,5,8,13};

constexpr int flb(int n)
{
    return (n < sizeof(ftbl)/sizeof(*ftbl)) ? ftbl[n] : flb(n);
}
```
constexpr函数可以接受引用实参。尽管它不能通过这些引用写入内容，但是const引用参数同样有用。

#### 条件求值  
constexpr函数之外的条件表达式不会在编译时求值，这意味着它可以请求运行时的值：
```cpp
constexpr int check(int i)
{
    return(low <= i && i < high) ? i : throw out_of_range();
}
constexpr int low = 0;
constexpr int high = 99;

constexpr int val = check(f(x,y,z));
```
其中，假定low和high是设计时未知而编译时已知的配置参数。此时，f(x,y,z)计算的是依赖于实现的值。


### [[noreturn]]函数  
形如[[...]]的概念被称为属性，属性可以置于任何位置。  
把[[noreturn]]放在函数声明语句的开始位置表示我们不希望函数返回任何结果：
```cpp
[[noreturn]] void exit(int);
```
如果函数被设定为[[noreturn]]，但是函数的内部仍然返回了某个值，将产生未定义的行为。  

### 局部变量  
定义在函数内部的名字称为局部名字。当线程执行到局部变量或常量的定义处时，它们将被初始化。将变量声明成static，则在函数的所有调用中都使用唯一的一份静态分配的对象：
```cpp
void f(int a)
{
    while(a--){
        static int n = 0;
        int x = 0;

        cout << n++ << x++;
    }
}

int main()
{
    f(3);
}
```
上述代码的输出是：  
0,0  
1,0  
2,0



## 参数传递  

### 引用参数  
```cpp
void f(int val, int& ref)
{
    ++val;
    ++ref;
}
```
调用f()时，++val递增第一个实参在当前函数内的副本，而++ref递增第二个实参本身。
```cpp
void g()
{
    int i = 1;
    int j = 1;
    f(i,j);
}
```
调用(i,j)递增j的值，但是不会递增i的值。第一个实参i是以传值的方式传入函数，第二个实参j是以传引用的方式传入函数的。应该尽量避免修改引用类型的实参；但是，遇到大对象时，引用传递比值传递更有效。此时，应该将引用类型的参数声明成const的，表明使用引用只是处于效率的考虑，而非想让函数修改对象的值:
```cpp
void f(const Large& arg)
{
    //不允许修改“arg”的值
}
```
如果在还是函数的声明中有某个引用参数未被指定为const，则倾向于认为函数将修改该参数的值。  
类似的，指针类型的参数被声明成const意味着该指针所指对象的值不会被函数改变：
```cpp
int strlen(const char*);
char* strcpy(char* to, const char* from);
int strcmp(const char*, const char*);
```
字面值，常量以及需要执行类型转换的参数可以被传递给const T&参数，但是不能传递给普通的非const T&参数。一方面，允许向const T&的转换以确保凡是能传给T类参数的值都能传给const T&类型的参数：
```cpp
float fsqrt(const float&);

void g(double d)
{
    float r = faqrt(2.0f); //传递存放2.0f的临时变量的引用
    r = fsqrt(r); //传递r的引用
    r = fsqrt(d); //传递存放static_cast<float>(d)的临时变量的引用
}
```
另一方面，禁止向非const引用参数转换可以有效规避风险：
```cpp
void fsqrt(float& i);

void g(double d, float r)
{
    update(2.0f); //错误，常量实参
    update(r); //传递r的引用
    update(d); //错误，需要执行类型转换
}
```
引用传递的准确描述是左值引用传递，原因是函数不能接受一个右值引用作为它的参数。左值能绑定在左值引用上(不能绑定在右值引用上)，右值能绑定在右值引用上(不能绑定在左值引用上)：
```cpp
void f(vector<int>&); //非const左值引用参数
void f(const vector<int>&); //const左值引用参数
void f(vector<int>&&); //右值引用参数

void g(vector<int>& vi, const vector<int>& cvi)
{
    f(vi); //f(vector<int>&)
    f(cvi); //f(const vector<int>&)
    f(vector<int>{1,2,3,4}); //f(vector<int>&&)
}
```
对于右值引用，最常见的用处是定义构造函数或者移动赋值运算。  
选择参数传递方式：  
1. 对于小对象使用值传递的方式
2. 对无需修改的大对象使用const引用传递
3. 如果需要返回计算结果，最好使用return修改对象
4. 使用右值实现移动和转发
5. 如果找不到合适的对象，则传递指针
6. 除非万不得已，否则不要使用引用传递  


### 数组参数  
当数组作为函数的参数时，实际传入的是指向该数组首元素的指针：
```cpp
int strlen(const char*)

void f()
{
    char v[] = "Annemarie";
    int i = strlen(V);
    int j = strlrn("Nicholas");
}
```
当作为参数被传入函数时，类型T[]会被转换成T*。这意味着如果对数组参数的元素赋值，就会改变数组元素的实际值。  
数组类型的参数与指针类型的参数等价：
```cpp
void odd(int* p);
void odd(int a[]);
void odd(int buf[1020]);
```
这三条语句是等价的，声明的是同一个函数。  
对被调函数来说，数组的尺寸是不可见的，可以再传入一个参数表示数组的大小：
```cpp
void compute1(int* vec_ptr, int vec_size);
```
更好地做法是传入某些容器的引用。  
如果向传入一个数组而非容器或指向数组首元素的指针，可以将参数类型声明成数组的引用：
```cpp
void f(int(&r)[4]);

void g()
{
    int a1[] = {1,2,3,4};
    int a2[] = {1,2};

    f(a1); //OK
    f(a2); //错误，元素个数优有误
}
```
对于数组引用类型的参数来说，元素个数也是其类型的一部分，因此，数组的引用的灵活性远不如指针或容器。通常在模板中才使用数组引用，此时元素个数可以通过推断得到：
```cpp
template<class T, int N> void f(T(&r)[N]);
{
    //...
}

int a1[10];
double a2[100];

void g()
{
    f(a1); //T是int，N是10
    f(a2); //T是double，N是100
}
```
这样的后果是调用f()所用的数组类型有多少个，对应的函数定义就有多少个。  
多维数组的情况比较复杂，一般用指针的数组替代，此时无需特殊处理：
```cpp
const char* day[] = {
    "mon","tue","wed","thu","fri","sat","sun"
};
```
一如既往，vector即类似类型比内置的底层数组和指针更好。

### 列表参数  
一个由{}限定的列表可以作为下述形参的实参：  
1. 类型std::initialize_list<T>，其中列表的值能隐式的转换成T  
2. 能用列表中的值初始化的类型  
3. T类型数组的引用，其中列表的值能隐式的转换成T  
```cpp
template<class T>
void f1(initialize_list<T>);

struct S{
    int a;
    string s;
};
void f2(S);

template<class T,int N>
void f3(T(&r)[N]);

void f4(int);

void g()
{
    f1({1,2,3,4}); //T是int，initialize_list的大小是4
    f2({1,"MKS"}); //f2(S{1,"MKS"});
    f3({1,2,3,4}); //T是int，N是4
    f4({1}); //f4(int{1});
}
```
如果存在二义性，则initialize_list参数的函数会被优先考虑。


### 数量未定的参数  
```cpp
int print(const char* ...);
```
这条语句规定对标准库函数printf()的调用必须至少有一个C风格字符串的参数，同时也可以有也可以没有其他参数：  
```cpp
printf("Hello world!\n");
printf("My name is %s %s",first_name, second_name);
printtf("%d + %d = %d",2,3,5);
```
想使用未限定类型的参数时，应该优先考虑使用重载函数、带默认参数的函数、接受initializer_list参数的函数或者可变参数模板。只有当参数数量和参数类型都不确定且可变模板也不适用时，才考虑使用省略号参数。

### 默认参数  
```cpp
class complex{
    double re,im;
public:
    coplex(double r,double i):re{r},im{i}{}
    complex(double r):re{r},im{0}{}
    complex():re{0},im{0}{}
};
```
如果想在complex中加入一些调试、跟踪和统计的代码。只要加在一个地方就可以了：
```cpp
complex(double r={}, double i={}):re{r},im{i}{}
```
如果用户提供的参数数量不足，则使用预置的默认参数。  
默认参数在函数声明时执行类型检查，在调用函数时求值：
```cpp
class X{
public:
    static int def_arg;
    void f(int = def_arg);
};
int X::def_arg = 7;

void g(X& a)
{
    a.f(); //maybe f(7)
    a.def_arg = 9;
    a.f(); //f(9)
}
```
最好避免使用值可能发生改变的默认参数。  
只能为参数列表中位置靠后的参数提供默认值：
```cpp
int f(int, int =0, char* =nullptr);
int g(int =0, int =0,char*); //错误
int h(int =0,int,char* =nullptr) //错误
```
在同一作用域的一系列声明语句中，默认参数不能重复或者改变：
```cpp
void f(int x=7);
void f(int =7); //错误，不允许重复默认参数
void f(int =8); //错误，默认参数不一致
void g()
{
    void f(int x=9);
}
```

## 重载函数  

为不同数据类型的同一种操作起相同的名字称为重载。  

### 自动重载解析  

当调用函数fct时，由编译器决定使用名为fct的那组函数中的哪一个，一句是考察实参类型与作用域中名为fct的哪个函数的形参类型最匹配。当找到了最佳匹配时，调用该函数；反之，引发编译器错误：
```cpp
void print(double);
void print(long);

void f()
{
    print(1L);  //print(long)
    print(1.0); //print(double)
    print(1);   //错误，具有二义性
}
```
为了合理的解决问题，我们采用如下顺序尝试一系列评判准则：  
[1] 精确匹配：也就是说，无须类型转换或者仅需简单地类型转换即可实现匹配  
[2] 执行提升后匹配：也就是说，执行了整数提升(bool转为int)以及float转为double    
[3] 执行标准类型转换后实现匹配，T\*转换为void*，以及int转换为unsigned int   
[4] 执行用户自定义的类型转换后实现匹配  
[5]使用函数声明中的省略号...进行匹配  
在该体系中，如果某次函数调用在能找到匹配的最高层级上发现了不止一个可用匹配，则本次调用将因产生了二义性而被拒绝，例如：
```cpp
void print(int);
void print(const chat*);
void print(double);
void print(long);
void print(char);

void h(char c, short s, float f){
    print(c); //精确匹配，print(char)
    print(i); //精确匹配，print(int)
    print(s); //精确匹配，print(int)
    print(f); //float转换为double的提升

    print('a'); //精确匹配，print(char)
    print(49); //精确匹配，print(int)
    print(0); //精确匹配，print(int)
    print("a"); //精确匹配，print(const char*)
    print(nullptr); //nullptr_t转换const char*的提升
}
```
重载函数与函数声明的次序无关。

### 重载与返回类型  

在重载解析过程中补考虑函数的返回类型，这样可以确保对运算符或者函数调用的解析独立于上下文：
```cpp
float sqrt(float);
double sqrt(double);

voi f(double da, float fla){
    float fl = sqrt(da); //sqrt(double)
    double d = sqrt(da); //sqrt(double)
    fl = sqrt(fla); //sqrt(float)
    d = sqrt(fla); //sqrt(float)
}
```

### 重载与作用域  
重载发生在一组重载函数的成员内部，也就是说，重载函数应该位于同一个作用域内，不同的非名字空间作用域中的函数不会重载：
```cpp
void f(int);

void g(){
    void f(double);
    f(1); //调用d(double)
}
```

基类和派生类提供的作用域不同，因此默认情况下基类函数和派生函数不会发生重载：
```cpp
struct Base{
    void f(int);
};

struct Derived:Base{
    void f(double);
};

void g(Derived& d){
    d.f(1); //Derived::f(double)
}
```

### 多实参解析  
对于一组重载函数以及一次调用来说，如果该调用对于各函数的参数类型在计算的效率和精度上差别明显，则可以应用重载解析规则从中选出最合适的函数：
```cpp
int pow(int, int);
double pow(double, double);
complex pow(double, complex);
complex pow(complex, int);
complex pow(complex, complex);

void k(coomplex z){
    int i = pow(2,2);
    double d = pow(2.0, 2.0);
    complex z2 = pow(2, z);
    complex z3 = pow(z, 2);
    complex z4 = pow(z, z);
}
```
当重载函数包含两个或者多个参数时，上述的解析规则将作用于每一个参数，并且选出该参数的最佳匹配函数。如果某个函数是其中一个参数的最佳匹配，停驶在其他参数上也是更优的匹配或者至少不弱于别的函数，则该函数就是最终确定的最佳匹配函数。如果找不到符合上述条件的函数，则本次调用将因二义性的原因被拒绝：
```cpp
void g(){
    double d = pow(2.0, 2); //错误，是调用pow(int(2.0),2)还是pow(2.0double(2))
}
```
对于该调用来说，2.0的最佳匹配函数是pow(double, double)，2的最佳匹配函数是pow(int, int)，因此存在二义性，是一次错误的调用。  

### 手动重载解析  
某个函数的重载版本过少或过多都可能导致二义性，例如：
```cpp
void f1(char);
void f1(long);

void f2(char*);
void f2(int*);

void k(int i){
    f1(i); //二义性：f1(char)还是f1(long)
    f2(0); //二义性：f2(char*)还是f2(int*)
}
```
在可能的情况下，尽量把一组重载函数当成整体来看，考察其对于函数的恶语义来说是否有意义。  


## 前置和后置条件
我们把函数调用是应该遵循的约定称为前置条件，把函数返回值应该遵循的约定称为后置条件：
```cpp
int area(int len, int wid)
/*
计算长方形的面积  

前置条件：长方形的长和宽都是正数

后置条件：返回值是正数  

后置条件：返回值是长方形的面积，其中长方形的长和宽分别是len和wid
*/
{
    return len*wid;
}
```

## 函数指针  
程序员只能对函数做两种操作：调用它或者获取它的地址。通过获取函数地址得到的指针能被用来调用该函数：
```cpp
void error(string s)

void (*efct)(string); //指向函数的指针，该函数接受一个字符串参数，不返回任何东西

void f(){
    efct = &error; //efct指向error
    efct("error"); //通过efct调用error
}
```
编译器发现efct是个函数指针，因此会调用它所指的函数。也就是说，解引用函数指针时可以用*，也可以不用；同样，在获取函数地址时可以用&，也可以不用：  
```cpp
void (*f1)(string) = &error;
void (*f2)(string) = error;

void g(){
    f1("Vasa"); //上下等价
    (*f1)("Mary Rose");
}
```
函数指针的参数类型声明与函数本身类似。进行指针赋值操作时，要求完整的函数类型都必须精确匹配：
```cpp
void (*pf)(string);
void f1(string);
int f2(string);
void f3(int*);

void f(){
    pf = &f1; // OK
    pf = &f2; //错误，返回类型错误
    pf = &f3; //错误，返回类型错误

    pf("Hera"); // OK
    pf(1) //错误，返回类型错误

    int i = pf("Zeus"); // 错误，试图将void赋给nit
}
```
C++允许将一个函数指针转换为别的指针类型，但之后必须把得到的结果指针转换回它原来的类型，否则就会出现意想不到的情况：
```cpp
using P1 = int(*)(int*);
using P2 = void(*)(void);

void f(Pa pf){
    P2 pf2 = reinterpret_cast<P2>(pf)
    pf2(); //可能发生严重错误
    P1 pf1 = reinterpret_cast<P1>(f2);  // 把pf2转换回来
    int x = 7;
    int y = pf1(&x)
}
```


