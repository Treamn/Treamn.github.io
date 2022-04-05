# C++学习之函数


## 函数基础
典型的函数定义包括以下部分：返回类型、函数名、形参列表和函数体。  
形参列表中形参用逗号隔开，其中每个形参都是一个含有声明符的声明。即使两个形参的类型一样，也必须把两个类型写出来。
```cpp
int fun(int a,b)      // false 
int fun(int a, int b) // true
```
```cpp
#include <iostream>
using namespace std;

int jc(int a){
    int res = 1;
    for(int i = 1; i <= a; i++){
        res *= i;
    }
    return res;
}

int main(){
    int a;
    cin >> a;
    cout << jc(a) << endl;
}
```

## 局部对象
名字有作用域，对象有生命周期。
### 自动对象   
对于普通局部变量对应的对象来说，当函数的控制路径经过变量定义语句时创建该对象，当到达定义的块末尾时销毁它。把志存在于块执行期间的对象成为自动对象。  
### 局部静态对象
在程序的执行路径第一次经过对象定义语句时初始化，直到程序终止才被销毁，在此期间即使对象所在的函数结束执行也不会对他有影响。 将局部变量定义为static类型来获得这样的对象。
```cpp
#include <iostream>
using namespace std;

size_t count_calls(){
    static size_t ctr = 0;
    return ++ctr;
}

int main(){
    for(size_t i = 0; i != 10; ++i){
        count_calls();
    }
    cout << count_calls(); // 11, 如果ctr没有被定义为static，则输出1.
    return 0;
}
```
 
## 函数声明  
函数只能定义一次，但是可以声明多次。函数的声明和定义十分类似，唯一的区别就是函数声明无需函数体，一个分号替代即可。  
函数的三要素（返回类型、函数声明和形参类型）描述了函数的接口，说明了调用该函数所需的全部信息。函数声明也被成为函数原型。

## 参数传递
形参的类型决定了形参和实参交互方式。如果形参时引用类型，它将绑定到对应的实参上；否则，将实参的值拷贝后赋给形参。  
当形参是引用类型时，我们说它对应的实参被**引用传递**或者函数被**传引用调用**。  
当实参的值被拷贝给形参时，形参和实参时两个相互独立的对象。我们说这样的实参被**值传递**或者函数被**传值调用**。
### 指针形参
当执行指针拷贝操作时，拷贝的时指针的值。拷贝之后，两个指针是不同的指针。因为指针时我们可以间接访问它所指的对象，所以通过指针可以修改它所指对象的值。
```cpp
#include <iostream>
using namespace std;

int main(){
    int n = 0, i = 42;
    int* p = &n;
    int* q = &i;
    cout << *p << *q << endl;// 0, 42
    *p = 4;
    cout << n << endl;// 4
    p = q;
    cout << *p << n << endl;// 42, 4
}
```
### 传引用参数
- 使用引用避免拷贝  
拷贝大的类类型对象或者容器对象比较低效，甚至有的类类型根本不治持拷贝操作（包括IO类型在内）。当某种类型不治持拷贝操作时，函数只能通过引用形参访问该类型的对象。  
- 使用形参返回额外信息  
*？？？还没整明白*

## const形参和实参
和其他初始化过程一样，当用实参初始化形参时会忽略掉顶层const。当形参有顶层const时，传给他常量对象或者非常量对象都是可以的。   
```cpp
void func(const int i){}   // 该函数只能读取i，不能对i的值做修改。
```
在调用func函数时，既可以传入const int，也可以传入int。
### 指针或引用型参与const
可以使用非常量初始化一个底层const对象，但是反过来不行；同时一个普通的引用类型必须用同类型的对象初始化。
```cpp
int i = 42;
const int *cp = &i; //对
const int &r = i;   //对
const int &r2 = 42; //对
int *p = co;        //错，类型不匹配
int &r3 = r;        //错，类型不匹配
int &r4 = 42;       //错，类型不匹配
```
### 尽量使用常量引用
把函数不会改变的形参定义成普通引用是一种比较常见的错误，这样做会给调用者一种误导，即函数可以修改它实参的值。此外，使用引用而非常量引用也会极大限制函数所能接受的实参类型。

## 数组形参
数组的两个特殊性质：  
 1.不允许拷贝数组；  
 2.使用数组时通常会将其转换为指针。  
 因为不能拷贝数组，所以无法以值传递的方式使用数组参数。因为数组会被转换为指针，所以为函数传递一个数组时，实际上传递的事指向数组首元素的指针。  
 虽然不能以值传递的方式传递数组，但是可以将形参写成类似数组的形式。
 ```cpp
void print(const int*);
void print(const int[]);
void print(const int[10]);
 ```
### 使用标记指定数组长度
要求数组本身包含一个结束标记，这种方法的典型示例是C风格字符串，C风格字符串存储在字符数组中，并且在最后一个字符后面跟着一个空字符。
```cpp
void print(const char* cp){
    if(cp)
        while(*cp)
            cout << *cp++;
}
```
适用于有明显结束标记且该标记不会与普通数据混淆的情况。

### 使用标准库规范
管理数组实参的第二种技术是传递指向首元素和尾后元素的指针。
```cpp
void print(const int* beg, const int* end){
    while(beg != end){
        cout << *beg++ << endl;
    }
}
int j[2] = {0, 1};
print(begin(j), end(j));
```

### 显式传递一个表示数组大小的形参
```cpp
void print(const int ia[], size_t size){
    for(size_t i = 0; i != size; ++i){
        cout << ia[i] << endl;
    }
}
int j[] = {0,1};
print(j, end(j) - begin(j));
```
只要传递给函数的size步超过数组实际大小，函数就是安全的。

### 数组形参和const
当函数不需要对数组元素执行写操作时，数组形参应该是指向const的指针。  
当函数确实要改变元素值的时候，才把形参定义成指向非常量的指针。

### 数组引用形参
C++允许将变量定义成数组的应用，同理，形参也可以是数组的引用。此时，引用形参绑定到对应的实参上，也就是绑定在数组上。  
```cpp
void print(int (&arr)[10]){
    for (auto elem : arr)
        cout << elem << endl;
}
int j[2] = {0,1};
int k[10] = {0,1,2,3,4,5,6,7,8,9};

print(j); // 错误，实参不是含有10个整数的数组
print(k); // 正确
```

### 传递多维数组
将多维数组传给函数时，真正传递的是指向数组首元素的指针。因为处理的是数组的数组，所以首元素本身就是一个数组，指针就是一个指向数组的指针。数组第二维的大小都是数组类型的一部分。
```cpp
void print(int (*matrix)[10], int rowSize)
// 等价定义
void print(int matrix[][10], int rowSize)
```

## 含有可变形参的函数
有时无法提前预知应该向函数传递几个实参，此时最好使用同一个函数实现该项功能。   
- initializer_list形参  
如果函数的实参数量未知但是全部实参类型都相同，此时可以使用initializer_list类型的形参。  
initializer_list是一种标准库类型，用于表示某种特定类型的值的数组。  
和vector一样，initializer_list也是一种模板类型，在定义initializer_list对象时，必须说明列表所含元素的类型。  
```cpp
initializer_list<int> li;
```
和vector不一样的是，initializer_list对象中的元素永远是常量值，无法对initializer_list对象中元素的值。  
如果想向initializer_list形参中传递一个值的序列，则必须把序列放在一对花括号内。
```cpp
void error_msg(initializer_list<string> il){
    for(auto beg = il.begin(); beg != il.end(); ++beg)
        cout << *beg << " ";
    cout << endl;
}

if(expected != actual)
    error_msg({"function", expected, actual});
else
    error_msg({"functionX", "okay"});
```
含有initializer_list形参的函数也可以同时拥有其他形参。
```cpp
void error_msg(ErrCode e,initializer_list<string> il){
    cout << e,msg() << ": ";
    for(const auto &elem : il)
        cout << elem << " ";
    cout << endl;
}

if(expected != actual)
    error_msg(ErrCode(42), {"function", expected, actual});
else
    error_msg(ErrCode(0), {"functionX", "okay"});
```

## 返回类型和return语句
### 无返回值函数  
没有返回值的return语句只能用在返回类型是void的函数中。  
通常，如果void函数如果想在中间位置提前退出，可以使用return语句。此时return语句的作用类似于break语句。  

### 有返回值的函数
- 不要返回局部对象的引用或指针  
函数完成后，它所占用的存储空间也随之被释放。因此，函数终止意味着局部变量的引用意味着局部变量的引用将指向不再有效的内存区域。

## 函数重载
如果同一作用域的几个函数名字相同但形参列表不同，称之为重载函数。(main函数不能重载)   
重载函数最好只用于操作非常相似的函数。   
虽然函数的名称相同，但编译器会格局实参的类型确定应该调用哪个函数。  
```cpp

```
但是，不允许两个函数除了返回值类型之外其他所有的要素都相同。假设有两个函数，他们的形参列表一样，的那是返回类型不同，那么第二个函数的声明是错误的。
```cpp
int lookup(const Account&);
bool lookup(const Account&); //错误，与上一函数相比，仅返回类型不同。
```

### 重载和const形参  
一个拥有顶层const的形参无法和另一个没有顶层const的形参区分开来。  
```cpp
Record lookup(phone);
Record lookup(const phone);  //重复声明了Record lookup(phone)
```
如果形参是某种类型的指针或引用，则通过区分其指向的是常量对象还是非常量对象可以实现函数重载，此时的const是底层的。  
```cpp
Record lookup(Account&);
Reocrd lookup(const Account&);
Record lookup(Account*);
Reocrd lookup(const Account*);
```
### const_cast和重载
```cpp
const string &shortString(const string &s1, const string &s2){
    reutrn s1.size() <= s2.size() ? s1 : s2;
}
```
这个函数的参数和返回值类型都是const string的引用。我们可以对两个非常量的string实参调用这个函数，但返回的结果依然是const string的引用。因此，我们需要一种新的shortString函数，当它的实参不是常量时，得到的结果是一个普通的引用，此时可使用const_cast。
```cpp
string &shortString(string &s1, string &s2){
    auto &r = shortString(const_cast<const string&>(s1), const_cast<const string&>(s2));
    return const_cast<string&>(r);
}
```

## 默认实参  
在函数的很多次调用中他们都被赋予一个相同的值，此时，将这个反复出现的值称为函数的默认实参。
```cpp
typedef string::size_type sz;
string screen(sz ht = 24, sz wid = 80, char backgrnd = ' ');

string window;
window = screen(); // screen(24,80,' ')
window = screen(66,256,'#'); // screen(66,256,'#')
```
### 默认实参声明  
对于函数声明，通常习惯将其放在头文件中，并且一个函数只声明一次，但多次声明同一函数也是合法的。  
但是，需要注意，在给定的作用域中，一个形参只能被赋予一次默认实参。换句话说，就是在函数的后续声明中，只能为之前没有默认值的形参添加默认实参，而且该形参右侧所有形参必须都有默认值。
```cpp
string screen(sz, sz, char = ' ');
string screen(sz, sz, char = '*'); // 错误，不能修改一个已经存在的默认值
string screen(sz = 24, sz = 24, hcar); //正确，添加默认实参
```
### 默认实参初始值  
局部变量不能作为默认实参。除此之外，只要表达式的类型能够转换成形参所需的类型，该表达式就能作为默认实参。  
```cpp
sz wd = 80;
char def = ' ';
sz ht();
string screen(sz = ht(), sz = wd, char = def);
string window = screen(); // screen(ht(), 80, ' ')

void f2(){
    def = '*';
    sz wd = 100;
    window = screen(); // screen(ht(), 80, '*')
}
```
def的值在函数f2中被改变，所以screen会调用这个更新过的值，但是wd只是函数内部声明的一个局部变量，和传给screen的实参没有任何关系。

----
## 内联函数和constexpr函数  
内联函数和constexpr函数通常定义在头文件中。
### 内联函数可以避免函数调用的开销
将函数指定为内联函数(inline)，通常就是在他的每个调用点上内联的展开。  
`cout << shotrString(s1, s2) << endl;`  
会在编译过程中展开成如下形式  
`cout << (s1.size() < s2.size() ? s1 : s2) << endl;`  
从而消除shortString函数的运行时开销。
在函数的返回类型前加上关键字inline，就可以将函数声明为内联函数。  
```cpp
inline const string& shortString(const String &s1, const String &s2){
    return s1.size() < s2.size() ? s1 : s2;
}
```
***需要注意，内联函数只是向编译器发出的一个请求，编译器可以选择忽略这个请求。***   
一般来说，内联机制用于优化规模较小，流程直接，频繁调用的函数。  
### constexpr函数
contexpr函数是指能用与常量表达式的函数。定义constexpr函数的方法和其他函数类似，不过要遵循几项约定：**函数的返回类型以及所有形参的类型都得是字面值类型，而且函数体中有且只有一条return语句。**  
```cpp
constexpr int new_sz() { return 42; }
constexpr int foo = new_sz();
```
constexpr函数体内也可以包含其他语句，只要这些语句在运行时不执行操作即可。例如，contexor函数中可以有空语句、类型别名以及using声明。  
也允许constexpr函数的返回值并非一个常量。
`constexpr size_t scale(size_t cnt) { return new_sz() * cnt; }`
当scale的实参是常量表达式时，它的返回值也是常量表达式，反之则不然。
```cpp
int arr[scale(2)]; // 正确
int i = 2;
int arr[scale(i)]; // 错误，scale(i)不是常量表达式
```
---
## 函数匹配  
```cpp
void f();
void f(int);
void f(int, int);
void f(double, double = 3.14);

f(5.6);  // 调用f(double, double)
```
---
## 函数指针
函数指针指向的是函数而非对象。和其他指针一样，函数指针指向某种特定类型。函数的类型由它的返回类型和形参类型共同决定。
```cpp
bool lengthCompare(const string &, const string &);
bool (*pf)(const string &, const string&)
```
*pf两端的括号必不可少，如果没有括号，则pf是一个返回值为bool指针的函数。
### 使用函数指针
当把函数名作为一个值使用时，该函数自动转换为指针，按照如下形式可以将lengthCompare函数的地址赋给pf
```cpp
pf = lengthCompare;
pf = &lengthCompare; // 等价赋值语句，取址符时可选的
```
此外，可以直接使用指向函数的指针调用该函数，无须提前解引用指针：
```cpp
bool b1 = pf("hello", "goodbye");
bool b2 = (*pf)("hello", "goodbye"); //等价调用
bool b3 = lengthCompare("hello", "goodbye"); //另一个等价调用
```
在指向不同的函数类型的指针间不存在转换规则。但是和往常一样，可以为函数指针赋一个nullptr或值为0的整型常量表达式，表示该指针没有指向任何一个函数。
```cpp
string::size_type sumLength(const string&, const string&);
bool cstringCompare(const char*, const char*);
pf = 0; // 正确，此时pf为空指针
pf = sumLength; // 错误，返回值类型不匹配
pf = cstringCompare; // 错误，形参类型不匹配
pf = lengthComapre; // 正确，函数和指针的类型精确匹配
```
### 重载函数的指针
当使用重载函数时，上下文必须清晰的界定到底选用哪个函数。如果定义了指向重载函数的指针，编译器通过指针类型决定选用哪个函数，指针类型必须和重载函数中某一个精确匹配。
```cpp
void ff(int*);
void ff(unsigned int);

void (*pf1)(unsigned int) = ff; //pf1指向ff(unsigned)
void (*pf2)(int) = ff; //错误，没有ff与此形参列表匹配
double (*pf3)(int*) = ff; // 错误，返回类型不匹配
```

### 函数和指针形参
和数组类似，虽然不能定义函数类型的形参，但是形参可以是指向函数的指针。
```cpp
void useBigger(const string &s1, const string &s2, bool pf(const string &, const string &)); // 第三个形参是函数类型，会自动转换成指向函数的指针
void useBigger(const string &s1, const string &s2, bool (*pf)(const string &, const string &));  // 等价声明，显式的将形参定义为指向函数的指针

useBigger(s1, s2, lengthCompare); //自动将lenthCompare转换成指向该函数的指针
```
直接使用函数指针类型显得冗长而繁琐。类型别名和decltype可以简化使用函数指针的代码。
```cpp
//Func和Func2时函数类型
typedef bool Func(const string&, const string&);
typedef decltype(lengthCompare) Func2;//等价的类型


//FuncP和FuncP2是指向函数的指针
typedef bool(*FuncP)(const string&, const string&);
typedef decltype(lengthCompare) *FuncP2 // 等价的类型
```
需要注意的是，decltype返回函数类型，此时不会将函数类型自动转换为指针类型。因为decltype的结果是哈书类型，所以只有在结果前加上*才能得到指针。可以使用如下的形式重新声明useBigger：
```cpp
void useBigger(const string&, const string&, Func);
void useBigger(const string&, const string&, *FuncP2);
```

### 返回指向函数的指针
和数组类似虽然不能返回一个函数，但是可以返回指向函数类型的指针。然而，必须把返回类型写成指针类型，编译器不会自动将函数返回类型当成对应的指针类型处理。与往常一样，要想声明一个返回函数指针的函数，最简单的方法是使用类型别名：
```cpp
using F = int(int*, int); //F为函数类型
using PF = int(*)(int*, int); // PF是指针类型

PF f1(int); // 正确，PF是指向函数的指针，f1返回指向函数的指针
F f1(int); //错误，F是函数类型，f1不能返回一个函数
F *f1(int); //正确，显式的指定返回类型是指向函数的指针

int (*f1(int))(int*, int);
//或使用尾置返回类型
auto f1(int) -> int(*)(int*, int);
```


