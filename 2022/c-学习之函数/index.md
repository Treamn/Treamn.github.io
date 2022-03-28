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



















