# C++表达式


## 桌面计算器实例  
计算器包含四个部分：分析器、输入函数、符号表和驱动。实际上，它的功能类似一个微型编译器：其中分析器负责分析语法，输入函数负责处理输入及词法分析，符号表存放永久信息，驱动处理初始化、输出和错误。

### 分析器  
下面是计算器程序遵循的一套语法： 
```cpp
program:
    end  //end是输入的结束
    expr_list end

expr_list:
    expression print //print是换行或分号
    expression print expr_list

expression:
    expression + term
    expression - term
    term

term:
    term / primary
    term * primary
    primary

primary:
    number //number是一个浮点型字面值常量
    name   //name是一个标识符
    nae=expression
    - primary
    (expression)
```
换句话说，程序就是以分号隔开的一段表达式序列。表达式的基本单元是数字，名字以及运算符等。其中，名字不需要在使用之前提前声明。   
此处使用的是递归下降语法分析机制，这是一种被广泛接受且含义明确的自顶向下技术。   
在处理输入的环节，分析器使用Token_stream，它负责把读入字符的过程以及字符的组成情况封装到Token中。也就是说Token_stream的作用是“单词化”，负责把字符流转换成Token。其中Token是一个形如{单词类型，值}的对，在{数字，123.45}的例子中，123.45被转换成浮点数值。   
Token的定义如下所示： 
```cpp
enum class Kind:char{
    name,number,end,
    plus='+', minus='-', mul='*',div='/', print=';', assign='=',lp='(', rp=')'
};

struct Token{
    Kind kind;
    srting string_value;
    double number_value;
}
```
把每个单词表示成字符对应的整数形式是一种便捷有效的手段，有助于进行调试。  
Token_stream接口的形式是：  
```cpp
class Token_stream{
public:
    Token get(); //读取并返回下一单词
    const Token& current(); //刚刚读入的单词
};
```

每个分析函数接受一个名为get的bool类型实参，它指明函数是否需要调用Token_stream::get()以获取下一个单词。每个分析函数求值“它的”表达式并返回相应的值。函数expr()处理加法和减法，它包含一个循环，依次寻找加法和减法所需的项：
```cpp
double expr(bool get)
{
    double left = term(get);

    for(;;){ //读作forever
        switch(ts.current().kind){
            case Kind::plus:
                left += term(true);
                break;
            case Kind::plus:
                left -= term(true);
                break;
            default:
                return left;
        }
    }
}
```
C++规定赋值运算符可与下面的二元运算符一起使用：
```cpp
+ - * / % & | ^ << >>
```
因此，下面的赋值运算符都是合法的：
```cpp
= += -= *= /= %= &= |= ^= <<= >>=
```
其中，^是异或运算符，<<和>>分别是左移和右移运算符。  
函数term执行乘法和除法操作，其方式与expr()处理加法和减法的方式一样：
```cpp
double term(bool get)
{
    double left = prim(get);

    for(;;){ //读作forever
        switch(ts.current().kind){
            case Kind::nul:
                left *= term(true);
                break;
            case Kind::div:
                if(auto d = prim(true)){
                    left /= term(true);
                    break;
                }
                return error("divide by 0");
            default:
                return left;
        }
    }
}
```
除数为0的除法其结果是未定义的，通常会依法严重的程序错误。  
当确实需要使用变量d并且立即对它进行初始化时，才会把该变量引入程序中。因此，仅当d不等于0时执行除法赋值语句left/=d。  
函数prim()处理初等项的方式很像expr()和term()。因为需要进入调用层次的狭义层，所以在这里需要多做一点事情，而且不必使用循环：
```cpp
double term(bool get)
{
    if(get) ts.get();

    switch(ts.current().kind){
            case Kind::number:{
                double v = ts.current().number_value;
                ts.get();
                break;
            }
            case Kind::name:{
                double& v = tabel[ts.current().string_value];
                if(ts.get().kind == Kind::assign) v = expr(true);
                return v;
            }
            case Kind::minus:
                return -prim(true);
            case kind::lp:{
                auto e = expr(true);
                if(ts.get().kind != Kind::rp) return error("')'expected");
                ts.get();
                return e;
            }
            default:
                return error("primary expected");
    }
}
```
当发现Token是一个number时，它的值被置于它的number_value中。与之类似，当发现Token十一个name时，它的值被置于对应的string_value中。  
相对于分析初等项表达式所需实际的Token数量来说，prim永远多读取一个。这是因为在某些情况下它必须这么做，因此从一致性的考虑除法，它干脆在所有的情况下都统一了起来。当分析函数只想移动到下一个Token时，它不需要使用ts.get()的返回值。此时，可以从ts.current获取结果。   
在对某个名字执行具体操作前，计算器受限向前查看该名字是否被赋值或者只是从中读取了内容。两种情况下都要用到符号表，符号表的类型是map：
```cpp
map<string,double> table;
```
也就是说，当table以string作为索引时，所得到的结果值是与该string对应的double。例如，假定用户的输入是：
```cpp
radius = 6378.388；
```
则计算器将进入case Kind::name并且执行  
```cpp
double& v = table["radius"];
v = 6378.388;
```
引用v用于保存与radius关联的double，expr()从输入的字符序列计算得到值。  

### 输入  
首先，给出Token_stream的完整定义： 
```cpp
class Token_stream{
public:
    Token_stream(istream& s):ip{&s},owns{false}{}
    Token_stream(istream* p):ip{p},owns{true}{}

    Token_stream(){close();}

    Token get();
    Token& current();

    void set_input(istream& s){close(); ip = &s; owns=false;}
    void set_input(istream* p){close(); ip = p; owns=true;}

private:
    void close(){if(owns) delete ip;}

    istream* ip;
    bool owns;
    Token ct{Kind::end};
}
```
我们用一个输入流初始化Token_stream，从该输入流中获取它的字符。Token_stream占有一个以指针方式传递的istream，而不是以引用的方式传递istream。当指针指向需要析构的资源，而这样的指针被包含在类的内部。  
Token_stream保存三个值：一个指向其输入流的指针(ip)、一个用于指示输入流所有权的布尔值(ows)和当前的单词(gt)。  
一开始的语句从*ip读取第一个非空白字符到ch，并且检查读取的操作是否成功：
```cpp
Token Token_stream::get(){
    char ch = 0;
    *ip >> ch;

    switch(ch){
        case 0:
            return ct={Kind::end};
    
```
默认情况下，>>运算符会跳过空白，并当输入操作失败时不更改ch的值。  
表达式终结符、括号和匀速阿福的处理方式很简单，直接返回它们的值即可： 
```cpp
case ';';
case '*';
case '/';
case '+';
case '-';
case '(';
case ')';
case '=';
    return ct = {static_cast<Kind>(ch)};
```
因为在char和kind之间没有隐式类型转换规则，所以必须使用static_cast。   
数字的处理方式是：  
```cpp
case '0':case '1':case '2':case '3':case '4':case '5':case '6':case '7':case '8':case '9':case '.':
ip -> putback(ch);
*ip->ct.number_value;
ct.kind-Kind::number;
return ct;
```
如果单词不是输入结束符、运算符、变电符号或者数字，则它必然是一个名字。处理名字的方式与数字类似：
```cpp
default:
    if(isalpha(ch)){
        ip -> putback(ch);
        *ip >> ct.string_value;
        ct,kind = Kind::name;
        return ct;
    }
```
最后，可嗯会得到一个错误。处理错误的一种简单但是有效的方法是调用error函数，然后返回一个print单词：
```cpp
error("bad token");
return ct = {Kind::print};
```
标准库函数isalpha()可以令我们不必把每个字符作为一个单独的case标签。>>运算符在字符串内连续读取直到遇到一个空白符时停止。因此，在运算符使用某个名字作为它的运算对象之前，用户必须加上一个空白符表示名字的结束。  
下面是完整的输入函数： 
```cpp
Token Token_stream::get(){
    char ch = ;
    *ip>>ch;

    switch(ch){
        case 0:
            return ct = {Kind::end};
        case ';';
        case '*';
        case '/';
        case '+';
        case '-';
        case '(';
        case ')';
        case '=';
        return ct = {static_cast<Kind>};
        case '0':case '1':case '2':case '3':case '4':case '5':case '6':case '7':case '8':case '9':case '.':
        ip -> putback(ch);
        *ip->ct.number_value;
        ct.kind-Kind::number;
        return ct;
    default:
    if(isalpha(ch)){
        ip -> putback(ch);
        *ip >> ct.string_value;
        ct,kind = Kind::name;
        return ct;
    }
    error("bad token");
    return ct = {Kind::print};
    }

}
```
