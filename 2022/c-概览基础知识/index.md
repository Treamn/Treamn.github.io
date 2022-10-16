# C++概览基础知识


## 类型  
在定义一个变量时，如果变量的类型可以由初始化器推断得到，则无须显式指定其类型：  
```cpp
auto b = true;     //bool
auto ch = 'x';     //char
auto i = 123;      //int
auto d = 1.2;      //double
auto z = sqrt(y);  //sqrt(y)返回的类型
```
可以使用=的初始化与auto配合，因为在此过程中不存在可能引发错误的类型转换。  
当没有明显的理由需要显式指定数据类型时，一般使用auto。  
- 该定义位于一个较大的作用域中，我们希望代码的读者清楚的知道其类型；
- 我们希望明确规定某个变量的范围和精度（如希望使用double而非float）。  

通过使用auto可以帮助避免冗余，并无须书写长类型名。  




## 常量  
- const：大致意思是“我承诺不改变这个值”。主要用于说明接口，这样在把变量传入函数时就不必担心变量会在函数内被改变。编译器负责确认并执行const的承诺。 
- constexpr：大致意思是”在编译时求值“。主要英语说明常量，作用是允许将数据至于只读内存中以及提升性能。 
```cpp
const int dmv = 17;                   //常量
int var = 17;                         //不是常量
const double max1 = 1.4*square(dmv);  //如果square(17)是常量则正确
const double max2 = 1.4*square(var);  //错误，var不是常量
const double max3 = 1.4*square(var);  //OK，可在运行时求值
double sum(const vector<double>&);    //sum不会更改参数值
vector<double> v{1.2, 3.4, 4.5};      //v不是常量
const doublr s1 = sum(v);             //OK，在运行时求值
constexpr double s2 = sum(v);         //错误，sum(v)不是常量表达式
```
如果某个函数用在常量表达式中，则该表达式在编译时求值，则函数必须定义成constexpr。
```cpp
constexpr double square(double x){
    return x*x;
}
```
要想定义成constexpr，函数必须非常简单：函数中只能有一条用于计算某个值的return语句。constexpr函数可以接受非常量实参，但其结果将不会是一个常量表达式。当程序的上下文不需要常量表达式时，我们可以使用非常量表达式实参来调用constexpr函数，这样就不用把同一个函数定义两次：其中一个用于常量表达式，另一个用于变量。  



## 指针  
指针的声明如下：
```cpp
char* p;  //该指针指向字符
```
*表示”指向....“。指针变量中存放着一个相应类型对象的地址：
```cpp
char* p = &v[3]; //p指向v的第四个元素
char x = *p;     //*p是p所指的对象
```
在表达式中，前置一元运算符*表示”...的内容“，而前置一元运算符&表示”...的地址“。

```cpp
void print(){
    int v[] = {0,1,2,3,4};

    for(auto x : v)
        cout << x << '\n';
}
```
如果我们不希望把v的值拷贝到变量x中，而只想令x指向一个元素，则可以书写如下代码：
```cpp
void increment(){
    int v[] = {0,1,2,3,4};

    for (auto& x : v)
        ++x;
}
```
一元后值运算符&表示”...的引用“，引用类似于指针，唯一的区别就是无须使用前置运算符*访问所引用的值。同样，一个引用在初始化之后就不能再引用其他对象。当用于声明语句时，运算符(如&、*和[])称为声明运算符。
```cpp
T a[n];  //n个T组成的数组
T* p;    //指向T的指针
T& r;    //T的引用
T f(A);  //一个函数，接受A类型的实参，返回T类型的结果
```
当没有对象可指或我们希望有一种”没有可用对象“(如在列表的末尾)，令指针取值为nullptr("空指针")，所有指针类型共享同一个nullptr：
```cpp
double* pd = nullptr;
Link<Record>* lst = nullptr; //指向一个Record的Link指针
int x = nullptr;             //错误
``` 
通常情况下，当我们希望指针实参指向某个东西时，最好检查以下是否确实如此。
```cpp
int count_x(char* p, char x)
//统计p[]中x出现的次数
//假定p指向一个字符数组，该数组的结尾处是0；或者p不指向任何东西
{
    if(p==nullptr) return 0;
    int count = 0;
    for(;*p != 0; ++p)
        if(*p == x)
            ++count;
    return count;
}
```
有两点需要注意：一是可以使用++将指针移动到数组的下一元素；二是在for语句中如果不需要初始化操作，则可以省略它。  
count_x()的定义假定char*是一个c风格字符串，也就是说指针指向了一个字符数组，该数组的结尾处是0。


## 结构  
```cpp
struct Vector{
    int sz;       //元素数量
    double* elem; //指向元素的指针
};

void vector_int(Vector& v, int s)
{
    v.elem = new double[s]; //分配一个数组，包含s个double值
    v.sz = s;
}
```
v的elem成员被赋予了一个由new运算符生成的指针而sz成员的值则是元素的个数，Vextor&中的符号&指定通过非常量引用的方式传递v，这样vector_int()就能修改传入其中的向量了。   
new运算符是从一块名为自由存储(free store)的区域中分配内存。 
Vector的一个简单应用如下：
```cpp
double read_and_sum(int s)
{
    Vector v;
    vector_int(v,s);
    for(int i = 0; i != s; ++1)
        cin>>v.elem[i]
    
    double sum = 0;
    for(int i= 0; i != s; ++i)
        sum += v.elem[i];
    return sum;
}
```
访问struct成员的方式有两种：一种是通过名字或引用，这时我们使用.(点运算符)；另一种是通过指针，这时用到的是->。
```cpp
void f(Vector v, Vector& rv, Vector* pv)
{
    int i1 = v.sz;  //通过名字访问
    int i2 = rv.sz; //通过引用访问
    int i4 = pv->sz;//通过指针访问
}
```


## 类
类含有一系列成员，可能是数据、函数或者类型。类的public成员定义该类的接口，private成员则只能通过接口访问。
```cpp
class Vector{
public:
    Vector(int s): elem{new double[s]}, sz{s}{}  //构建一个vector
    double& operator[](int i){return elem[i]}  //通过下标访问元素
    int size(){return sz;}
private:
    double* elem; //指向元素的指针
    int sz;       //元素的数量
};
```
我们仅能通过Vector的接口访问其表示形式，上面的read_and_sum()可简化为：
```cpp
double read_and_sum(int s)
{
    Vector v(s);
    for(int i = 0; i != v.size(); ++1)
        cin>>v[i]
    
    double sum = 0;
    for(int i= 0; i != v.size(); ++i)
        sum += v[i];
    return sum;
}
```
与所属类同名的函数称为构造函数，即它是用来构造类的对象的函数。与普通函数不同，构造函数的作用是初始化类的对象，因此定义一个构造函数可以解决类变量为初始化的问题。该构造函数使用成员初始化器列表来初始化vector的成员：
```cpp
:elem{new double[s]}, sz{s}
```
这条语句的含义是：首先从自由空间获取s个double类型的元素，并用一个指向这些元素的指针初始阿护elem；然后用s初始化sz。  
访问元素的功能是由一个下标函数提供的，这个函数名为operator[]，它的返回值是对相应元素的引用(double&)。



