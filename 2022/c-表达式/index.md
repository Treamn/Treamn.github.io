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

### 底层输入  
在函数get()中，用依次读取单个字符的方式替换面向类型的默认输入操作。  
首先，用换行等价替代分号指示表达式的结束：  
```cpp
Token Token_stream::get(){
    char ch;

    do{ // 跳过除‘\n之外的其他空白符’
        if(!ip->get(ch)) return ct={Kind::end};
    }while(ch != '\n' && iaspace(ch));
}

switch(ch){
    case ';':
    case '\n':
        return ct = {Kind::print};

```
do和while语句十分类似，唯一区别就是do循环的受控语句部分至少会执行一次。   
当调用ip->get(ch)时，从输入流*ip读取一个字符存放到ch中。默认情况下，get()不会像>>那样跳过空白符。如果cin中没有字符可供读取，则(!ip->get(ch))的条件满足；此例中，程序返回Kind::end以结束计算器的执行。   
标准库函数isspace()提供对空白符的检测方法；如果c是一个空白字符，则isspace返回一个非零值，否则返回0.类似的检测函数还有isdigit()(是否是数字)、isalpha()(是否是字母和isalnum()(是否是字母或数字)。  
跳过空白符后，下一个字符决定接下来的单词是什么类型。  
\>>运算符的机制是读取字符串的内容直到遇见空白符为止。为解决这一问题，我们每次只读取一个字符，当该字符不是字母或者数字时终止读取过程：
```cpp
default: //名字，名字=，或者错误
    is(isalpha(ch)){
        string_value = ch;
        while(ip->get(ch) && isalnum(ch))
            string_value += ch; //把ch加到string_value末尾
        ip->putback(ch);
        return ct={Kind::name};
    }
```

### 错误处理  
error()函数负责统计错误数量、输出错误消息，然后返回：
```cpp
int no_of_errors;

double error(const string& s){
    no_of_errors++;
    cerr << "error:" << s << '\n';
    return 1;
}
```
其中，cerr是一个不带缓冲的输出流，常用于错误报告。  
error()函数之所以要求要返回一个值是因为错误通常发生在表达式的求值过程中，所以，我们要么放弃整个求值过程，要么返回一个不会造成后续错误的值。  

### 驱动程序  
所有程序模块编写完成后，我们还需要一个驱动控制程序开始执行，其中main()负责启动程序及报告错误；calculate()负责完成实际的计算任务： 
```cpp
Token_stream ts{cin};

void calculate(){
    for(;;){
        ts.get();
        if(ts.current().kind == Kind::end) break;
        if(ts.current().print == Kind::end) continue;
        cout << expr(false) << '\n';
    }
}

int main(){
    table["pi"] = 3.14;
    table["e"] = 2.71;

    calculate();

    return no_of_errors;
}
```
主循环的核心任务是读取表达式并输出答案，用下面的代码行来实现：
```cpp
cout << expr(false) << '\n';
```
实参false告诉expr()无须调用ts.get()来读取单词。  

### 头文件  
```cpp
#include<iostream>
#include<string>
#include<map>
#include<cctype>
```

### 命令行参数  
main()被传入两个实参，分别是：argc指明实参的数量，argv代表实参的数组。argv的类型是char\*[argc+1]。argv[0]表示程序的名字。因此argc的值至少是1.实参列表以0作为结束符，即，argv[argc]==0。    
基本的思想是像从输入流读取数据一样读取命令行字符串的内容，从字符串湖区数据的流叫isstringstream：
```cpp
Token_stream ts {cin};

int main(int argc, char* argv[]){
    switch(argc){
        case 1:
            braek;
        case 2:
            ts.set_input(new isstringstream{argv[1]});
            break;
        default:
            error("too many arguments");
            return 1;
    }

    table["pi"] = 3.14;
    table["e"] = 2.71;

    calculate();

    return no_of_errors;
}
```
为了便于传递和分发程序的实参，用一个简单地函数创建一个vector<string>：
```cpp
vector<string> arguments(int argc, char* argv[]){
    vector<string> res;
    for(int i = 0; i != argc; ++i)
        res.push_back(argv[i]);
    return res;
}
```


## 运算符概述  

### 结果  
算数运算符的结果类型由一组称为“常见算数类型转换”的规则决定。这些规则的基本目标是产生“最大的”运算对象类型的结果。例如，如果一个二元运算符由一个浮点型运算对象，则相应的运算基于浮点数运算规则执行，所得的结果也十一个浮点值。类似的，如果它有一个long运算对象，则运算基于长整型运算规则进行，所得结果是long。在开始执行运算前，尺寸小于int的运算对象先转换成int类型。   
只要逻辑上说的通，对于接受左值运算对象的运算符来说，它的结果是一个表示该左值运算对象的左值：
```cpp
void f(int x,int y){
    int j = x = y; //x=y的值是x在执行赋值运算之后的结果值
    int* p = &++x; //p指向x
    int* q = &(x++); //错误：x++不是一个左值
    int* p2 = &(x>y?x:y); //具有较大值的int的地址
    int& r = (x<y>)?x:1; //错误，1不是左值
}
```
如果?:的第二个和第三个运算对象都是左值且类型相同，则该运算符的运算结果是同一个类型的左值。   
sizeof的结果是名为size_t的无符号整数类型，该类型定义在\<cstddef\>中。两个指针相减的结果是名为ptrdiff_t的带符号整数类型，同样定义在\<cstddef\>中。  

### 求值顺序  
逗号运算符(,)，逻辑与运算符(&&)，逻辑或运算符(||)规定它们的左侧运算对象先被求值，然后才是右侧运算对象。

## 常量表达式  
- constexpr：编译时求值。  
- const：在作用域内不改变其值。  

### 符号化常量  
常量最重要的一个用处是为值提供符号化的名字。这样可以起到把信息局部化的作用。  

### 常量表达式中的const  
const常用于表示接口。同时，const也可以表示常量值：  
```cpp
const int x = 7;
const string s = 'asdf';
const int y = sqrt(x);
```
以常量表达式初始化的const可以用在常量表达式中。与constexpr不同的是，const可以用非常量表达式初始化，但是此时该const将不能用作常量表达式：
```cpp
constexpr int xx = x; //OK
constexpr string ss = s; //错误，s不是常量表达式
constexpr int yy = y; //错误，sqrt(x)不是常量表达式
```
发生错误的原因是string不是字面值常量类型，sqrt()不是一个constexpr函数。  

### 字面值常量类型  
在常量表达式中可以使用简单的用户自定义类型：
```cpp
struct Point{
    int x,y,z;
    constexpr Point up(int d){return {x,y,z+d};}
    constexpr Point move(int dx,int dy){return {x+dx, y+dy};}
};
```
含有constexpr构造函数的类称为字面值常量类型。构造函数必须足够简单才能声明成constexpr，其中“简单”的含义是它的函数体必须为空且所有成员都是用潜在的常量表达式初始化的： 
```cpp
constexpr Point origo{0,0};
constexpr int z = origo.x;

constexpr Point a[]={
    origo, Point{1,1},Point{2,2},origo.move(3,3)
};
constexpr int x = a[1].x; //x值变为1
constexpr Point xy{0,sqrt(2)}; //错误，sqrt(2)不是常量表达式
```
即使把数组声明成constexpr，仍然能够访问该数组的元素及对象成员。  
可以定义constexpr函数使其接受字面值常量类型的实参：
```cpp
constexpr int square(int x){
    return x*x;
}

constexpr int radial_distance(Point p){
    return isqrt(square(p.x)+square(p.y)+square(p.z));
}
constexpr Point p1{10,20,30};
constexpr p2{p1.up(20)};
constexpr int dist = radial_distance(pa);
```
因为没有一个便于使用的constexpr浮点型平方根函数，所以用了int而非double。  

### 引用参数  
=和+=等用于修改对象的操作不能是constexpr的。相反，real()和imag()等简单读取对象内容的操作可以是constexpr的，我们在编译时用一条给定的常量表达式对它们求值。  


## 隐式类型转换  
### 提升  
保护值不被改变的隐式类型转换通擦汗能够称为提升。在执行算数运算前，通常先把较短的整数类型通过整型提升成int。

### 类型转换  
使用{}列表能防止窄化计算的发生：
```cpp
void f(double d){
    char c{d}; //错误，编译器发现程序试图把双精度浮点数转换成字符类型  
}
```

#### 整数类型转换  
整数能被转换成其他整数类型。一个普通的枚举值也能转换成整数类型。  
如果目标类型时unsigned的，则结果值所占的二进制位数以目标类型为准：
```cpp
unsigned char uc = 1023;//二进制1111111111:uc值变为二进制11111111，即255
```
如果目标类型是signed的，则当原值能用目标类型表示时，它不会发生改变；反之，结果值依赖于具体实现。  

#### 浮点数类型转换
如果原值能用目标类型完整的表示，则所得的结果与原值相等。如果原值介于两个相邻的目标值之间，则结果取它们中的一个。  

### 指针和引用类型的转换  
任何指向对象类型的指针都能隐式的转换成void*。指向派生类的指针能隐式的转换成指向其可访问的且明确无二义的基类的指针。指向函数的指针和指向成员的指针不能隐式的转换成void*。  
求值结果为0的常量表达式能隐式的转换成任意指针类型的空指针。类似的，求值结果为0的常量表达式也能隐式的转换成指向成员的指针类型：
```cpp
int* p = (1+2)*(2*(1-1));
```
最好直接使用nullptr。 
T*可以隐式地转换成const T\*。类似的，T&能隐式的转换成const T&。


