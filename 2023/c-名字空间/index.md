# C++名字空间


## 名字空间  
名字空间的概念用来直接表示本属一体的一组属性。名字空间的成员都位于相同的作用域中，无须特殊符号即可互相访问，而从名字空间外访问它们就需要显式符号：
```cpp
namespace Graph_lib{
    class Shape{};
    class Line:public Shape{};
    class Poly_line:public Shape{}; //相连的Line序列
    class Text:public Shape{}; //文本标签

    Shape operator+(const Shape&, const Shape&); //形状组合

    Graph_reader open(const char*); //打开Shape文件
}
```
类似的，文本库可以命名为Text_lib:
```cpp
namespace Text_lib{
    class Glyph{};
    class Word{}; //Glyph序列
    class Line{}; //Line序列
    class Text{}; //Text序列

    File* open(const char*); //打开文本文件

    Word operator+(const Line&, const Line&); //连接
}
```
只要设法取一些独特的名字，如Graph_lib和Text_lib，就可以一起编译这两组声明而不会产生冲突。   
一个名字空间应该表达某种逻辑结构：一个名字空间中的声明应该一起提供一些特性，使得在用户看来它们是一个整体，而且能反映一组共同的设计策略。    
一个名字空间就形成一个作用域，在一个名字空间中，稍后的声明可以引用之前定义的成员，但不能从名字空间之外引用其他成员：  
```cpp
class Glyph{};
class Line{};

namespace Text_lib{
    class Glyph{};
    class Word{}; //Glyph序列
    class Line{}; //Line序列
    class Text{}; //Text序列

    File* open(const char*); //打开文本文件

    Word operator+(const Line&, const Line&); //连接
}

Glyph glyph(Line& In, int i);
```
在本例中，Text_lib::operator+()声明中的Word和Line引用的是Text_lib::Word和Text_lib::Line。全局的Line不会影响局部名字查找。相反，全局glyph()声明中的Glyph和Line引用的则是全局::Glyph和::Line。此（非局部）查找也不会收Text_lib的Glyph和Line的影响。  
为了饮用名字空间的成员，我们可以使用完整的限定名字。例如，如果我们希望glyph()使用Text_lib中的定义：
```cpp
Text_lib::Glyph glyph(Text_lib::Line& In, int i); 
```
从名字空间外引用成员的其他方法包括using声明，using指示和参数依赖查找。  

### 显式限定  
我们可以在名字空间的定义中声明一个成员，稍后用”名字空间名：：成员名“的语法定义它。  
名字空间的成员必须用如下语法引入：
```cpp
namespace namespace-name{

}
```
例如：
```cpp
namespace Parser{
    double expr(bool); //声明
    double term(bool);
    double prim(bool);
}

double val = Parser::expr(); //使用

double parser::expr(bool b){ //定义

}
```
不能使用限定符语法在名字空间的定义之外为其声明一个新成员，这是为了捕获拼写错误和类型不匹配之类的错误，以及便于在名字空间声明中查找所有名字：  
```cpp
void Paeser::logical(bool); //错误，Parser中没有logical()
double Parser::trem(bool);  //错误，Parser中没有trem()(拼写错误)
double Parser::prim(int);  //错误，Parser::prim接受一个bool类型的参数（类型错误）
```
一个名字空间形成一个作用域，通常作用域规则也适用于名字空间。全局作用域也是一个名字空间，可以显式地用::来引用：
```cpp
int f(); //全局函数

int g(){
    int f; //局部变量：屏蔽了全局函数
    f();   //错误；不能调用一个整型变量
    ::f(); //正确，调用一个全局函数
}
```
类也是名字空间。  


### using声明  
using声明将一个代用名引入了作用域，最好尽量保持代用名的局部性以避免混淆。  
当用于一个重载的名字时，using声明会应用于其所有重载版本：
```cpp
namespace N{
    void f(int);
    void f(string);
};

void g(){
    using N::f;
    f(789); //N::f(int)
    f("Bruce"); //N::f(string)
}
```


### using指示  
using指示允许我们在所在的作用域中无须使用限定即可访问某个名字空间的所有名字：  
```cpp
using namespace std;

vector<string> split(const string& s){
    vector<string> res;  
    istringstream iss(s);
    for(string buf; iss>>buf;)
        res.push_back(buf);
    return res;
}
```
使用全局using指示必须小心谨慎，因为过度使用using指示会导致名字冲突。  
除了极特殊的情况外，不要在头文件中将一个using指示置于全局作用域中。  


### 参数依赖查找  
接受参数类型为用户自定义类型X的函数常常于X定义在相同的名字空间中。因此，如果在使用函数的上下爱问中找不到函数的定义，可以在函数的名字空间中查找它：
```cpp
namespace Chrono{
    class Date{};

    bool operator == (const Date&, const std::string&);

    std::string format(const Dtae&);
}

void f(Chrono::Date d, int i){
    std::string s = format(d); //Chrono::format()
    std::string t = format(i); //错误，作用域中没有format()的定义
}
```
参数依赖查找对于运算符运算对象和模板参数特别有用。  
注意，名字空间本身必须处于函数的作用域中，且函数声明必须在函数查找和使用之前。  
一个函数自然可以接受来自多个名字空间的参数：
```cpp
void f(Chrono::Date d, std::string s){
    if(d == s){
        //...
    }else if(d == "August 4, 1914"){
        //...
    }
}
```
当一个类成员调用一个命名函数时，编译器会优先选择同一个类的其他成员及其基类而不是基于参数类型查找到的函数：
```cpp
namespace N{
    struct S{int i};
    void f(S);
    void g(S);
    void h(int);
}

struct Base{
    void f(N::S);
};

struct D : Base{
    void mf();

    void g(N::S x){
        f(x);  //调用Base::f()
        mf(x); //调用D::mf()
        h(1);  //错迕，没有可用的h(int)
    }
};
```
- 如果一个参数是一个类成员，关联名字空间即为类本身和包含类的名字空间。  
- 如果一个参数是一个名字空间的成员，则关联名字空间即为外层的名字空间。  
- 如果一个参数是内置类型，则没有关联名字空间。  




