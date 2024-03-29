# C++类型与声明


## 类型  

### 布尔值  
根据定义，当我们想把布尔值转换成整数时，trye转为1，false转为0。反之，整数值也能在需需要的时候隐式的转换成布尔值，其中非0整数值对应true，而0对应false。
```cpp
bool b1 = 7; //b1为true
bool b2{7}; //错误，发生了窄化变换

int i1 = true; // i1=1
int i2{true}; // i2=1
```
如果既想使用{}初始化器列表防止窄化转换的发生，同时又想把int转换为bool，可以显式声明如下：
```cpp
voif f(int i){
    bool b{i != 0};
}
```
如有必要，指针也能被隐式的转换成bool。其中，非空指针对应true，值为nullptr的指针对应false。如：
```cpp
void g(int* p){
    bool b = p; //窄化成true或false
    bool b2{p != nullptr}; //显式的检查指针是否为空
    if (p){

    }
}
```
与if(p!=nullptr)相比，if(p)更好，不但简洁，而且可以直接表达“p是否有效”，同时也不易出错。


### 字符类型  
绝大多数情况下，char占8个二进制位。  
signed char存放-127~127之间的值；unsigned char存放0~255之间的值。  
**需要注意的是**，字符类型属于整形，因此，可以在字符类型上执行算数运算和位逻辑运算。如：
```cpp
void digits(){
    for(int i = 0; i != 10; ++i)
        cout << static_cast<char>('0' + i);
}
```
上面的代码把10个阿拉伯数字输出到cout。字符字面值常量‘0’先转换为它对应的整数值，再与i相加；所得的int再转回char并被输出到cout。‘0+i’得到的结果本来是一个int，因此如果不加上static_cat<char>的话，输出的结果将会是48,49...而不是0，1...  

### 带符号字符和无符号字符  
虽然从本质上来说，char的行为无非与signed char抑制或者与unsigned char一致，但这3个名字代表的类型的确各不相同。我们不能混用指向这三种字符类型的指针：
```cpp
void f(char c,signed char sc, unsigned char uc){
    char* pc = &uc; //错误，不存在对应的指针转换规则
    signed char* psc = pc; //错误，不存在对应的指针转换规则
    unsigned char* puc = pc; //错误，不存在对应的指针转换规则
    psc = puc; //错误，不存在对应的指针转换规则
}
```
3种char类型的变量可以相互赋值，但是把一个特别大的值赋给带符号的char是未定义的行为：
```cpp
void g(char c,signed char sc, unsigned char uc){
    c = 255; //如果普通的char是带符号的且占8位，则该语句的行为依赖于具体实现
    c = sc; //OK
    c = uc; //如果普通的char是带符号的uc的值特别大，则该语句的行为依赖于具体实现
    sc = uc; //如果uc的值特别大，则该语句的行为依赖于具体实现
    uc = sc; //OK：转换成无符号类型
    sc = c; //如果普通的char是带符号的uc的值特别大，则该语句的行为依赖于具体实现
    uc = c; //OK：转换成无符号类型
}
```
再举个例子，假设char占8位：
```cpp
signed char sc = -160;
unsigned char uc = sc; //uc==116(因为256-140=116)
cout << uc; //输出‘t’
char count[256]; //假设是占8位的char（未初始化的）
++count[sc]; //严重错误：越界访问
++count[uc]; //OK
```

### 字符字面值常量  
字符字面值常量是指单引号内的一个字符，如‘a’和‘0’等。  

### void 
从语法结构上来说，void属于基本类型。但是它只能用作其他复杂类型的一部分，不存在任何void类型的对象。void有两个作用：一是作为函数的返回类型用以说明函数不返回任何实际的值；二是作为指针的基本类型部分表明指针所指对象的类型位置：
```cpp
void x; //错误，不存在void类型的对象
void& r; //错误，不存在void的引用
void f(); //函数f不返回任何实际的值
void* pv; //指针所指的对象类型未知
```

## 声明  
大多数声明同时也是定义。我们可以把定义看成是一种特殊的声明，它提供了在策划能够许志鸿使用该实体所需的一切信息。

### 声明多个名字  
在声明语句中，运算符只作用域紧邻的一个名字，归于后续的其他名字是无效的：
```cpp
int* p,y; //int* p;int y;
int x, *p; // int x; int *p;
int v[10], *pv; // int v[10]; int *pv;
```
### 初始化  
初始化器有四种可能的形式：
```cpp
X a1{v};
X a2 = {v};
X a3 = v;
X a4(v);
```
这些形式中，只有第一种不受限制。使用{}的初始化称为列表初始化，它能防止窄化转换,这句话的意思是：
- 如果一种整型存不下另一种整型的值，则后者不会被转化成前者。例如，允许char到int的转换，但是不允许int到char的转换。  
- 如果一种浮点类型存不下另一种浮点型的值，则后者不会被转换成前者。例如，允许float到double的转换，但是不允许double到float的转换。
- 浮点型的值不能转换成整型值。  
- 整型值不能转换成浮点型的值。  

当使用auto关键字从初始化器推断变量的类型时，没必要采用列表初始化的方式。而且如果初始化器是{}列表，则推断到的数据类型肯定不是我们想要的结果：
```cpp
auto z1 {99}; //initialize_list<int>
zuto z2 = 99; //int
```
因此在使用auto时应该选择=的初始化方式。  

### 缺少初始化器

如果没有指定初始化器，则全局变量、名字空间变量、局部static变量和static成员(统称为静态对象)将会执行相应的数据类型的列表{}初始化：
```cpp
int a; //等同于‘int a{}’，a为0
double d; //等同于‘double d{}’, d为0.0
```
对于局部变量和自由存储上的对象来说，除非它恩无畏于用户自定义类型的默认构造函数中，否则不会执行默认初始化：
```cpp
void f(){
    int x; //x没有一个定义良好的值
    char buf[1024]; //buf[i]没有一个定义良好的值

    int* p{new int}; //*p没有一个定义良好的值
    char* q{new char[1024]};//q[i]没有一个定义良好的值

    string s; //s==""
    vector<char> v; //v =={}

    string* ps{new string}; //*ps是“”
}
```
如果想对内置类型的局部变量或者用new创建的内置类型的对象执行初始化，使用{}形式：
```cpp
void f(){
    int x{}; //x==0
    chat buf[1024]{}; //buf[i]==0

    int* p{new int{10}}; //*p为10
    char* q{new char[1024]{}} //q[i]==0
}
```

## 推断类型：auto和decltype()  
- auto根据对象的初始化器推断对象的数据类型，可能是变量、const、或constexpr类型。
- decltype(expr)推断的对象不是一个简单的初始化器，有可能是函数的返回类型或者成员的类型。

### auto类型修饰符  
表达式的类型越难懂、越难书写，auto就越有用：
```cpp
template<class T> void f1(vector<T>& arg)
{
    for(vector<T>::terator p= arg.begin(); p != arg.end(); ++P)
        *p = 7;
    for(auto p = arg.begin(); p != arg.end(); ++P)
        *p = 7;
}
```
对上面的程序来说，使用auto显然是更好地选择。  
与使用明确的类型名相比，使用auto可能会使得定位类型错误的难度增大，为了解决auto可能造成的影响，最常规的方法就是保持函数的规模较小。  
我们可以推断出类型添加修饰符或说明符，比如const或&：
```cpp
void f(vector<int>& v)
{
    for(const auto&x:v){//x类型是const int&
        ...
    }
}
```
表达式的类型永远不会是引用类型，因为表达式回隐式的执行解引用操作：
```cpp
void g(int& v)
{
    auto x = v; //x类型是int
    auto& y = v; //y类型是int&
}
```

### decltype修饰符  
很多时候我们既想推断得到类型，又不想在此过程中定义一个初始化的变量，此时，应该使用声明类型修饰符decltype(expr)。其中，所推得的结果是expr的声明类型：
```cpp
template<class T, class U>
auto operator+(const Matrix<T>& a, const Matrix<U>& b) -> Matrix<decltype(T{} + U{})>
```

## 对象和值  
对象是指一块连续的存储区域，左值是指向对象的一条表达式。  
“左值”的字面意思是“能用在赋值运算符左侧的东西”，但其实不是所有左值都嫩用在赋值运算符的左侧，左值有可能指某个常量。

### 左值和右值  
当考虑对象的寻址、拷贝、移动等操作时，有两种属性非常关键。  
- 有身份：在程序中有对象的名字，或指向该对象的指针，或该对象的引用，这样我们就能判断两个对象是否相等或对象的值是否发生改变。  
- 可移动：能把对象的内容移动出来。

## 类型别名  
```cpp
using Pchar = char*; //字符串指针
using Pf = int(*)(double); //函数指针，接受一个double返回一个int
Pchar p1 = nullptr; // p1类型是char*
char* p3 = p1; //正确
```
不孕须在类型别名前加修饰符（如unsigned）：
```cpp
using Char = char;
using Uchar = unsigned Char; //错误
using Uchar = unsigned char; //正确
```





