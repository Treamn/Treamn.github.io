# C++概览抽象机制


# 类

三种重要类的基本支持:
- 具体类；
- 抽象类；
- 类层次中的类。  
很多有用的类都可以归为这三个类别，其他类可以看成是这些类别的简单变形或组合。 

 ## 具体类型  
 具体类的基本思想是他们的行为“就像内置类型一样”。  
 具体类型的典型特征是，它的表现形式是其定义的一部分。在很多重要的例子中，表现形式只不过是一个或几个指向保存在别处的数据的指针，但这种表现形式出现在具体类的每一个对象中。这使得实现可以在时空上达到最优，尤其是它允许我们：
 - 把具体类型的对象置于栈、静态分配的内存或其他对象中； 
 - 直接引用对象(而非仅仅通过指针或引用)；
 - 创建对象后立即进行完整的初始化(比如使用构造函数)； 
 - 拷贝对象。   

 如果想提高灵活性，具体类型可以将其表现形式的主要部分防止在自由存储(动态内存、堆)中，然后通过存储在类对象内部的另一部分访问它们。vector和string的实现机理正是如此，我们可以把它们看做是带有精致接口的资源管理器。 

### 一种算数类型  
一种“经典的用户自定义算数类型”是complex：
```cpp
class complex{
    double re,im; //表现形式，两个爽双精度浮点数
public:
    complex(double r, double i):re{r}, im{i}{} //用两个标量构建该复数
    complex(double r):re{r},im{0}{} //用一个标量构建该复数
    complex():re{0},im{0}{} //默认的复数的{0,0}

    double real() const{return re;}
    void real(double d){re = d;}
    double imag() const{return im;}
    void imag(doubre d){im = d;}

    complex& operator+=(complex z){re+=z.re, im+=z.im; return *this;}
    //加到re和im上然后返回
    complex& operator-=(complex z){re-=z.re, im-=z.im; return *this;}

    complex& operator*=(complex); // 在类外的某处进行定义
    complex& operator*=(complex);  
};
```
这是对标准库complex略作简化后的版本，类定义把二审仅包含需要访问其表现形式的操作。它的表现形式非常简单，也是大家约定俗成的。  
无需实参就可以调用的构造函数称为*默认构造函数*，complex()是类complex的默认构造函数。通过定义默认构造函数，可以有效防止该类型的对象未初始化。  
在负责返回实部和虚部的函数中，const说明符表示这两个函数不会修改所调用的对象。  
很多有用的操作并不需要直接访问complex的表现形式，因此她们的定义可以与类的定义分离开：
```cpp
complex operator+(complex a, complex b){return a += b;}
complex operator-(complex a, complex b){return a -= b;}
complex operator-(complex a){return {-a.real(), -a.imag()};} // 一元负号
complex operator*(complex a, complex b){return a *= b;}
complex operator/(complex a, complex b){return a /= b;}
```
此处使用C++的一个特性，即。以传值方式传递实参实际上是把一份副本传递给函数，因此我们修改形参(副本)不会影响主调函数的实参，并可以将结果作为返回值。  
```cpp
bool operator==(complex a, complex b) //相等
{
    return a.real == b.real() && a.imag() == b.imag();
}
bool operator!=(complex a, complex b) //不等
{
    return !(a == b);
}

complex sqrt(complex);
```
可以像下面这样使用complex：
```cpp
void f(complex z){
    complex a {2.3}; //用2.3构建出{2.3,0.0}
    complex b {1/a};
    complex c {a+z*complex{1,2.3}}

    if(c != b)
        c = -(b/a)+2*b;
}
```
编译器自动将计算complex值的运算符转换成对应的函数调用，例如c!=b意味着operator!=(c,b),而1/a意味着operator/(complex{1},a)。  
在使用用户自定义的运算符("重载预算符")时，应当尽量小心谨慎，并且尊重常规的使用习惯。不能定义一元运算符/，因为其语法在语言中已被固定。同样，布尔诺能够改变一个运算符操作内置类型时的含义，因此不能定义运算符+令其执行int的减法。  
### 容器  
容器是指一个包含若干元素的对象，因为Vector的对象都是容器，所以称Vector是一种容器类型。Vector作为double的容器有许多优点“易于理解，建立有用的不变式，提供包含边界检查的范围功能，并提供size()以允许遍历其元素。然而，它还是存在一个致命缺陷：使用new分配元素，但是从来没有释放这些元素，尽管C++定义了一个垃圾回收的接口，可将未使用的内存提供给新对象，但不能保证垃圾回收器总是可用的。某些情况下不能使用回收功能，有时处于逻辑或性能的考虑宁愿使用更精确的资源释放控制。因此，我们需要一种机制以确保构造函数分配的内存一定会被销毁，这种机制就叫析构函数： 
```cpp
class Vector{
private:
    double* elem;//elem指向一个包含sz个double的数组
    int sz;
public:
    Vector(int s): elem{new double[s]},sz{s}{ //构造函数，请求资源
        for(int i = 0; i != s; ++i)
            elem[i] = 0; //初始化元素
    }
    ~Vector(){delete[] elem;} // 析构函数，释放资源 
    double& operator[](int i);
    int size() const;
};
```
析构函数得命名规则是一个求补运算符~后接类的名字，从含义上来说它是构造函数的补充。Vector的构造函数使用new运算符从自由存储分配一些内存空间，析构函数则用delete运算符释放该空间以达到清理资源的目的。这一切都无需Vector使用者的干预，只需像使用普通的内置类型变量那样使用Vector对象就行：
```cpp
void fct(int n){
    Vector v(n);
    // 使用v
    {
        Vector v2(2*n);
        //使用v和v2
    }//v2在此被销毁
}//v在此被销毁
```
构造函数负责分配元素接空间并正确的初始化Vector策划能够元，析构函数负责释放空间。这就是所谓的数据句柄模型。常用来管理在对象生命周期中大小会发生变化的数据。在构造按函数中请求资源，然后在析构函数中释放它们的技术称为*资源获取即初始化*，简称RAII，使得规避”裸new操作“的风险，该技术可以避免在普通代码中分配内存，而是把分配操作隐藏在行为良好的抽象类的实现内部。同样，也应避免”裸delete操作“。**避免裸new和裸delete可以使代码远离各种潜在风险，避免资源泄漏。**  

### 初始化容器  
容器的作用是保存元素，因此我们需要找到一种便利的方式将元素存入容器中。为了做到这一点，一种可能的方式时先用若干元素创建一个Vector，然后依次为这些元素赋值。显然这不是最好的办法，下面列举两种更简洁的途径。
- 初始化器列表构造函数：使用元素的列表进行初始化； 
- push_back()：在序列的末尾添加一个新元素。
它们的声明形式如下所示：
```cpp
class Vector{
public:
    Vector(std::initializer_list<double>); //使用一个列表进行初始化
    void push_back(double); //在末尾添加一个元素，容器长度加1
}
```
其中，push_back()可用于添加任意数量的元素：
```cpp
Vector read(istream& is){
    Vector v;
    for(double d; is>>d); //浮点值读入d
        v.push_back(d); //把d加到v中
    return v;
}
```
上面的循环操作负责执行输入操作，它的终止条件是遇到文件末尾或者格式错误。在此之前，每个读入的数依次添加到Vector尾部，最后v的大小就是读取的元素数量。  
用于定义初始化器列表构造函数的std::initializer_list是一种标准库类型，编译器可以辨识它：当使用{}列表时，如{1,2,3,4}，编译器会创建一个initializer_list类型的对象并将其提供给车供需。因此，可以写：
```cpp
Vector v1 = {1,2,3,4,5};
Vector v2 = {1.2,3.4,6.7,8};
```
Vector的初始化器列表构造函数可以定义成如下形式：
```cpp
Vector::Vector(std::initializer_list<double> lst)
    :elem{new double[lst.size()]},sz{lst.size()}
{
    copy(lst.begin(),list.end(),elem); //从lst复制内容到elem中
}
```

## 抽象类型
complex和Vector等类型之所以被称为具体类型，时因为它们的表现形式和俗语定义的一部分。这一点上，它们和内置类型很相似。相反，抽象类型则将使用者和类的实现细节完全隔离开来。为了做到这一点，我们分离接口与表现形式并放弃纯局部变量。因为对抽象类型的表现形式一无所知(甚至对大小也不了解)，所以必须从自由存储为对象分配空间，然后通过引用或指针访问对象。   
首先，为Container类设计接口，Container类可以看成是比Vector更抽象的一个版本：
```cpp
class Container{
public:
    virtual double& oprtator[](int) = 0; //纯虚函数
    virtual int size() const=0;
    //常量成员函数
    virtual ~Container(){} //析构函数
};
```
对于后面定义的特定容器来说，上面这个类时纯粹的接口。关键字virtual的意思是“可能随后在其派生类中冲重新定义”。把用关键字virtual声明的函数称为虚函数。Container的派生类负责为Container接口提供具体实现。看起来有点奇怪的=0说明该函数时纯虚函数，意味着Container的派生类必须定义这个函数。因此，不能够单纯定义一个Container的对象，Container只是作为接口出现，它的派生类负责具体实现operator[]()和size()。**含有纯虚函数的类称为抽象类**。
Container的用法是：
```cpp
void use(Container& s){
    const int sz = c.size();

    for(int i = 0; i != sz; ++i)
        cout << c[i] << '\n';
}
```
如果一个类负责为其他一些类提供接口，那么我们把前者称为多态类型。  
最为一个抽象类，Container中没有构造函数。另一方面，Container需要有一个析构函数，且该析构函数是virtual的。  
一个容器为了实现抽象类Container接口所需的函数，可以使用具体类Vector：  
```cpp
 class Vector_container:public Container{
    Vector v;
public:
    Vector_container(int s):v(s){} //含有s个元素的vector
    ~Vector_container(){}

    double& operator[](int i){return v[i];}
    int size() const{return v,size();}
 };
```
Vector_container类派生自Container类，而Container类称为Vector_container的基类。  
成员operqtor[]()和size()覆盖了基类中对应的成员。构造函数~Vector_container()则覆盖了基类的析构函数~Container()。成员v的析构函数~Vector被其类的析构函数~Vector_container()隐式调用。 
对于use(Container&)这样的的函数来说，可以在完全不了解一个Container实现细节的情况下使用它，但还需某个函数(g)为其创建可供操作的对象。
```cpp
void g(){
    Vector_container vc{10,9,8,7,6,5,4,3,2,1,0};
    use(vc);
}
```
因为use()只知道Container的接口而不了解Vector_container，因此对于Container的其他实现，use()仍能正常工作。
```cpp
class List_container:public Container{
    std::list<double> ld; //一个double类型的标准库list
public:
    List_container(){} //空列表
    List_container(initializer_list<double> il): ld{il}
    ~List_container(){}
    double& operator[](int i);
    int size() const {return ld.size();}
};

double& List_container::operator[](int i)
{
    for(auto& x:ld){
        if(i == 0) return x;
        --i;
    }
    throw out_of_range("List container");
}
```
这段代码中，类的表现形式是一个标准库list<double>。一般情况下，我们不会使用list实现一个带下标的容器。  
我们可以通过一个函数创建一个List_container，然后让use()使用它：
```cpp
void h()
{
    List_container lc = {1,2,3,4,5,6};
    use(lc);
}
```
这段代码的关键点在于use(Container&)并不清楚它的实参是Vector_container、List_container还是其他，它根本不需要知道。只要链接Container定义的接口就可以了。因此，不论List_container的实现发生了改变还是使用Container的一个全新派生类，都不需要重新编译use(Container&)。  
灵活性背后的唯一不足是，只能通过引用或指针操作对象。

## 虚函数  
进一步思考Container的用法： 
```cpp
void use(Container& c)
{
    const int sz = c.size();

    for(int i = 0; i != sz; ++i)
        cout << c[i] << '\n';
}
```
use中的c[i]是如何解析到正确的operator[]()的？当h()调用use()时，必须调用List_container的operator[]()；当g()调用use()时，必须调用Vector_container的operator[]().要想达到这种效果，Container对象就必须包含一些有助于它在运行时选择正确函数的信息。常见的做法时编译器将虚函数的名字转换成函数指针表中对应的索引值，这张表就是所谓的虚函数表或简称vtbl。每个含有虚函数的类都有它自己的vtbl用于辨识虚函数。  
即使调用函数不清楚对象的大小和数据布局，vtbl中的函数也能确保对象被正确使用。调用函数的实现只需要知道Container中vtbl指针的位置以及每个虚函数对应的索引就可以了。这种虚调用机制的效率非常接近“普通函数调用”机制，而它的空间开销包括两部分：如果类包含虚函数，则该类的每个对象需要一个额外的指针；另外每个这样的类需要一个vtbl。

## 类层次  
所谓类层次是指通过派生创建的一组类，在框架中有序排列。  
类层次提供两种便利：
- 接口继承：派生类对象可以用在任何需要基类的地方。也就是说，基类看起来是派生类的接口一样。这样的类通常是抽象类。  
- 实现继承：基类负责提供可以简化派生类实现的函数或数据。这样的基类通常含有数据称呀和构造函数。  
具体类，尤其是表现形式不复杂的类，其行为非常类似于内置类型：我们将其定义为局部变量，通过它们的名字访问它们，随意拷贝它们等。类层次中的类则有所区别：我们倾向于通过new在自由存储中为其分配空间，然后通过指针或引用访问它们。例如，我们设计这样一个函数，首先从输入流中读入描述形状的数据，然后构造对应的Shape对象：  
```cpp
enum class Kind{circle, triangle, smiley};

Shape* read_shape(istream& is){
    //从is中读取形状描述信息，找到形状的种类k
    switch(k){
        case Kind::circle:
            return new Circle{p, r};
        case Kind::triangle:
            return new Triangle{p1, p2, p3};
        case Kind::smiley:
            Smiley* ps = new Smiley{p,r};
            ps->add_eye(e1);
            ps->add_eye(e2);
            ps->set_mouth(m);
            return ps;

    }
}
```
程序使用该函数的方式如下所示：
```cpp
void use()
{
    std::vector<Shape*> v;
    while(cin)
        v.push_back(read_shape(cin));
        draw_all(v);
        rotate_all(v, 45);
        for(auto P:v) delete p;
}
```

## 拷贝和移动  
默认情况下，我们可以拷贝对象，不论用户自定义类型的对象还是内置类型的对象都是如此。拷贝的默认含义是逐成员的复制。如：
```cpp
void test(complex z1)
{
    complex z2{z1}; // 拷贝初始化
    complex z3;
    z3 = z2; //拷贝赋值
}
```
因为赋值和初始化操作都复制了complex的全部两个成员，所以在上述操作之后z1,z2,z3的值变得完全一样。  
当设计一个类时，必须仔细考虑对象是否会被拷贝以及如何拷贝的问题。  
 
### 拷贝容器  
当一个类作为资源句柄时，换句话说，当这个类负责通过指针访问一个对象时，采用默认的逐成员复制方式通常意味着错误。逐成员复制将违反资源句柄的不变式。例如，下面所示的默认拷贝将产生Vector的一份拷贝，而这个拷贝所指向的元素与原来的元素是同一个：
```cpp
void bad_copy(Vector v1)
{
    Vector v2 = v1;
    v1[0] = 2; //v2[0]也是2
    v2[1] = 3; //v1[1]也是3
}
```
类对象的拷贝操作可以通过两个成员来定义：拷贝构造函数与拷贝赋值运算符：
```cpp
class Vector{
private:
    double* elem;
    int sz;
public:
    Vector(int s);
    ~Vector(){delete[] elem;}

    Vector(const Vector& a);  //拷贝构造函数
    Vector& operator = (const Vector& a); //拷贝赋值运算符

    double& operator[](int i);
    const double& operator[](int i) const;

    int size() const;
};
```
对于Vector来说，拷贝构造函数的正确定义应该为指定数量的元素分配空间，然后把元素复制到空间中。  
```cpp
Vector::Vector(const Vector& a) //复制构造函数
    :elem{new double{sz}}, //为元素分配空间
    sz{a.sz}
{
    for(int i=0; i!=sz; ++i) //复制元素
        elem[i] = a.elem[i];
}
```
在拷贝构造函数之外我们还需要一个拷贝复制运算符：
```cpp
Vector& Vector::operator=(const Vector& a)
{
    double* p = new double[a.sz];
    for(int i=0; i!=sz; ++i) 
        p[i] = a.elem[i];
    delete[] elem; //删除旧元素
    elem = p;
    sz = a.sz;
    return *this;
}
```
其中，名字this预定义在成员函数中，它指向调用该成员函数的那个对象。  
类X的拷贝构造函数和拷贝赋值运算符接受的实参类型通常是const X&。


## 移动容器  
我们能通过定义拷贝构造函数和拷贝赋值运算符来控制拷贝过程，但是对于大容量的容器来说拷贝过程有可能耗费巨大。以下面代码为例：
```cpp
Vector operator+(const Vector& a, const Vector& b)
{
    if(a.size()!=b.size())
        throw Vector_size_mismatch{};
    
    Vector res(a.size());
    for(int i=0; i!=a.size(); ++i)
        res[i] = a[i] + b[i];
    return res;
}
```
要想从+运算符返回结果，需要把局部变量res的内容拷贝到调用者可以访问的地方。可能这样使用+”
```cpp
void f(const Vector& x, const Vector& y, const Vector& z)
{
    Vector r;
    r = x+y+z;
}
```
有时我们并不真的想要一个副本，而只想把计算结果从函数中取出来，相对于copy一个对象，我们更希望移动它：
```cpp
class Vector{
    Vector(const Vector& a);
    Vector& operator=(const Vector& a);

    Vector(Vector&& a); //移动构造函数
    Vector& operator=(Vector&& a); //移动赋值运算符
};
```
基于上述定义，编译器选择移动构造函数来执行从函数中移出返回值的任务。这意味着r=x+y+z不需要再拷贝Vector，只是移动它就够了。  
定义Vector移动构造函数的过程非常简单： 
```cpp
Vector::Vector(Vector&& a)
    :elem{a.elem}, //从a中夺取元素
    sz{s.sz}
{
    a.elem = nullptr; //现在a已经没有元素了
    a.sz = 0;
}
```
符号&&的意思是“右值引用”，我们可以给该引用绑定一个右值。“右值”的含义与“左值”正好相反，左值的大致含义是“能出现在赋值运算符左侧的内容”，因此右值大致上就是我们无法为其赋值的值，比如函数调用返回的一个整数。进一步，右值引用的含义就是引用了一个别人无法赋值的内容。Vector的operator+运算符的局部变量res就是一个示例。  
移动构造函数不接受const实参：毕竟移动构造函数最终要删除掉它实参中的值。移动赋值运算符的定义与之类似。  
当右值引用被用作初始化器或者赋值操作的右侧运算对象时，程序将使用移动操作，  
移动之后，源对象所进入的状态应该能允许运行析构函数。通常，我们也应该允许为一个移动操作之后的源对象赋值。  
程序员可以知道一个值在什么地方不再被使用，但是编译器做不到，因此最好在程序中写的明确一点：
```cpp
Vector f()
{
    Vector x(1000);
    Vector y(1000);
    Vector z(1000);

    z = x; //执行拷贝
    y = std::move(x); //执行移动
    return z； //执行移动
}
```
其中，标准库函数move()负责返回实参的右值引用。

## 资源管理  
通过定义构造函数，拷贝操作，移动操作和析构函数，程序员就能对受控资源的全生命周期进行管理。而且移动构造函数还允许对象从一个作用域简单便捷地移动到另一个作用域。采取这种方式，我们不能或不希望拷贝到作用域之外的对象就能简单高效地移动出去了。不妨以表示并发活动的标准库thread和含有百万个double的Vector为例，前者“不能”执行拷贝操作，而后者我们“不希望“拷贝它。
```cpp
std::vector<thread> my_threads;

Vector init(int n)
{
    thread t{heartbeat}; //同时运行heartbeat
    my_threads.push_back(move(t)); //把t移动到my_threads
    Vector vec(n);
    for(int i = 0; i < vec.size(); ++i)
        vec[i] = 777;
    return vec; //把vec移动到init()之外
}

auto v = init(); //启动heartbeat，初始化v
```
在很多情况下，用Vector和thread这样的资源句柄比的效果要好。事实上，以unique_ptr为代表的”智能指针“本身就是资源句柄。  

## 抑制操作  



