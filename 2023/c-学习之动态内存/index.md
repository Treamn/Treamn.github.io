# C++学习之动态内存


## 动态内存与智能指针   
动态内存的管理是通过一对运算符来完成的：new，在动态内存中卫队才能够分配空间并返回一个指向该对象的指针；delete接受一个动态对象的值怎，销毁该对象，并释放与之关联的内存。    
忘记释放内存时，就会产生内存泄漏。   
可使用两种智能指针来管理动态对象：  
- share_ptr：允许多个指针指向同一个对象； 
- unique_ptr：“独占”所指向的对象；       

除上述两种，标准库还定义一个weak_ptr的伴随类，是一种弱引用，指向shared_ptr所管理的对象。   
这三种类型均定义在memory头文件中。   


### shared_ptr   
```cpp
shared_ptr<string> p1;     //point to string
shared_ptr<list<int>> p2;  //point to a list with int
```
智能指针与普通指针的使用方法类似，解引用一个智能指针返回它所指向的对象。如果在一个条件判断中使用智能指针，效果就是检测其是否为空：
```cpp
if(p1 && p1->empty()){   //if p1 not empty, check the string p1 point is empty
    *pa = 'hi';   //if p1 point a empty string, 解引用p1,将一个新值赋予string
}
```


#### make_shared   
最安全的分配和使用动态内存的方法是调用一个名为make_shared的函数。此函数在动态内存中分配一个对象并初始化它，返回指向对象的shared_ptr。   
```cpp
shared_ptr<int> p3 = make_shared<int>(42); //p3指向42
shared_ptr<string> p4 = make_shared<string>(10,'9'); //p4指向“9999999999”
shared_ptr<int> p5 = make_shared<int>(); //p5指向0
```
也可以使用auto定义一个对象保存make_shared
```cpp
auto p6 = make_shared<vector<string>>();
```


#### shared_ptr的拷贝和赋值    
进行拷贝或赋值操作时，每个shared_ptr都会记录有多少个其他shared_ptr指向相同的对象。   
```cpp
auto p = make_shared<int>(42);  //p指向的对象只有p一个引用者
auto q(p); //p和q指向相同的对象，此对象有两个引用者  
```
可以认为每个shared_ptr都有一个关联的计数器，通常称为引用计数。    
无论何时拷贝一个shared_ptr，计数器都会递增。当给shared_ptr赋予一个新值或是shared_ptr被销毁时，计数器就会递减。    
一旦一个shared_ptr的计数器变为0，就会自动释放自己管理的对象。   
```cpp
auto r = make_shared<int>(42); //r指向的int只有一个引用者
r = q; //给r赋值，让它指向另一个地址
       //递增q的计数器
       //递减r的计数器
       //自动释放r原来指向的对象
```


#### shared_ptr自动销毁所管理的对象   
当指向一个对象的最后一个shared_ptr被销毁时，shared_ptr类会通过析构函数自动销毁此对象。



#### shared_ptr自动释放相关联的内存    
当动态对象不再被使用是，shared_ptr类会自动释放动态对象。    


#### 使用动态生存期的资源的类   
策划能够序使用动态内存出于以下三种原因之一：  
1. 程序不知道自己需要使用多少对象  
2. 程序不知道所需对象的准确类型  
3. 程序要在多个对象间共享数据

### 直接管理内存   


#### 使用new动态分配和初始化对象
在自由空间分配的内存时无名的，因此new无法为其分配的对象命名，而是返回一二给指向该对象的指针：   
```cpp
int *pi = new int; //pi指向一个动态分配的、未初始化的无名对象
```
此new表达式在自由空间构造一个int型对象，并返回指向该对象的指针。   
默认情况下，动态分配的对象是默认初始化的，这意味着内置类型或组合类型的对象的值将是未定义的，而类类型对象将用默认构造函数进行初始化：  
```cpp
string *ps = new string; //初始化为空的string
int *pi = new int;  //pi指向一个未初始化的int
```
对于内置类型，值初始化的内置类型对象有着良好定义的值，而默认初始化的对象的值则是未定义的。 
```cpp
int *pi1 = new int;   //*pi1的值未定义
int *pi2 = new int(); //*pi2为0
```
如果提供一个括号包围的初始化器，就可以使用auto从此初始化器来推断我们想要的分配的对象的类型。但是，由于编译器要用出是啊胡气得类型来推断要分配的类型，只有当括号中仅有单一初始化器时才可以使用auto：
```cpp
auto p1 = new aotu(obj); //p指向一个与obj类型相同的对象
                         //该对象用obj进行初始化
auto p2 = new auto{a,b,c};  //错误：括号中只能有单个初始化器
```
如果obj是一个int，那么p1就是一个int*；如果obj是一个string，那么p1就是一个string*。


#### 动态分配的const对象   
用new分配const对象是合法的。类似其他任何const对象，一个动态分配的const对象 **必须**进行初始化。


#### 内存耗尽   
如果可用内存全部被使用，new表达式就会失败。默认情况下，如果new布尔诺能够分配所要求的内存空间，就会抛出一个bad_alloc的异常。但可以通过改变使用new的方式阻止抛出异常：
```cpp
int *p1 = new int; //如果分配失败，则bad_alloc
int *p2 = new (nothrow) int; //如果分配失败，则返回一个空指针
```
bad_alloc和nothrow都定义在头文件new中。

#### 指针值和delete   
delete表达式接受一个指针，指向我们想要释放的对象。   
传递给delete的指针必须指向动态内存分配的内存，或者是一个空指针。释放一块并非new分配的内存，或者将相同的指针释放多次，其行为是未定义的：  
```cpp
int i, *pi1 = & i, *pi2 = nullptr;
double *pd = new double(33), *pd = pd;
delete i;    //错误，i不是指针
delete pi1;  //未定义：pi1指向一个局部变量
delete pd;   //正确
delete pd2;  //未定义，pd2指向的内存已经被释放了
delete pi2;  //正确
```
const对啊省得值不能被改变，但其本身是可以被销毁的。想要释放一个const动态对象，只需要delete它的指针。
```cpp
const int *pci = new const int(1024);
delete pci;
```


#### 动态对象的生存其直到被释放为止  
对于一个有内置指针管理的动态对象，直到被显式释放之前它都是存在的。    
返回指向动态内存的指针(而不是智能指针)的函数给其调用者增加了一个额外负担——调用者必须记得释放内存。
```cpp
// factory返回一个指针，指向一个动态分配的对象
Foo* factory(T arg){
    return new Foo(arg); //调用者负责释放此内存
}
```
调用者可能忘记释放内存，解决这个问题的方法是在use_factory中记得释放内存：
```cpp
void use_factory(T arg){
    Foo *p = factory(arg);
    delete p;
}
```


#### delete之后充值指针值   
delete一个指针后，指针值就变得无效了，但很多机器上指针仍然保存和动态内存的地址。在delete之后，指针就变成了空悬指针。    
如果需要保留指针可以在delete之后将nullptr赋予指针，这样就清楚的指出指针不指向任何对象。   
但是，delete内存之后重置指针的方法只对这个指针有效，对其他任何仍指向内存的指针是没有作用的。  
```cpp
int *p(new int(42)); //p指向动态内存
auto q = p;          //p和q指向相同内存
delete p;            //p和q均无效
p = nullptr;         //指定p为空
```
重置p对q没有任何作用。




### shared_ptr和new结合使用   
如果不初始化一个智能指针，它就会被初始化成一个空指针。   
```cpp
shared_ptr<double> p1;  //shared_ptr可以指向一个double  
shared_ptr<int> p2(new int(42));  //p2指向一个值为42的int
```
接受指针参数的智能指针构造函数是explicit的。因此，不能将一个内置指针隐式转换为一个智能指针，必须使用直接初始化形式来初始化一个只能智能指针。  
```cpp
shared_ptr<int> p1 = new int(1024); //错误，必须使用直接初始化形式
shaerd_ptr<int> p2(new int(1024));  //正确
```
出于相同原因，一个返回shared_ptr的函数不能在其返回语句中隐式转换一个普通指针，必须将shared_ptr显式绑定到一个想要返回的指针上：
```cpp
shared_ptr<int> clone(int p){
    return new int(p); //错误，隐式转换为shared_ptr
    reutrn shared_ptr<int>(new int(p)); //正确，显式的创建shared_ptr
}
```

#### 不要混合使用普通指针和智能指针   
使用内置指针访问智能指针所负责的对象是很危险的，因为无法知道对象何时会被销毁。   

#### 也不要使用get初始化另一个智能指针或为智能指针赋值   

智能指针类型定义一个名为get的函数，它返回一个内置指针，指向智能指针管理的对象。此函数是为了这样的情况设计的：我们需要向不能使用智能指针的代码传递一个内置指针。使用get返回的代码的不能够delete此指针。  
将另一个智能指针也绑定到get返回的指针上是错误的：
```cpp
shared_ptr<int> p(new int(42));
int *q = p.get(); //正确，但使用q时要注意，不要让它管理的指针被释放
{
    //未定义，两个独立的shared_ptr指向相同的内存
    shared_ptr<int>(q);
}//程序块结束，q被销毁，指向的内存被释放
int foo = *p; //未定义，p指向的内存已经被释放
```

#### 其他shared_ptr操作   
可以使用reset将一个新的指针赋予一个shard_ptr：
```cpp
p = new int(1024);       //错误，不能将指针赋予shared_ptr
p.reset(new int(1024));  //正确，p指向一个新对象
```
reset会更新计数，如果需要的话，会释放p所指向的对象。reset经常和unique一起使用，；癌控制多个shared_ptr共享的对象。在改变底层成员之前，我们检查自己是否是当前对象仅有的用户。如果不是，在改变之前要制作一份新的拷贝：  
```cpp
if(!p.unique())
    p.reset(new string(*p)); //我们不是唯一用户，分配新拷贝
*p += newVal;  //现在是唯一用户，可以改变对象的值
```


### 智能指针和异常   
如果使用智能指针，即使程序块过早结束，智能指针类也能确保在内存不再需要时将其释放：
```cpp
void f(){
    shared_ptr<int> sp(new int(1924)); //分配一个新对象
    //代码抛出异常，且在f中未被捕获
}//在函数结束时，shared_ptr自动释放内存
```
发生异常时，直接管理的内存不会自动释放。如果使用内置指针管理内存，且在new之后对应的delete前发生异常，则内存不会被释放：
```cpp
void f(){
    int *p = new int(42); //动态分配一个新对象
    //代码抛出异常，且在f中未被捕获
    delete p; //在退出之前释放内存
}
```


### unique_ptr   
一个unique_ptr”拥有“它所指向的对象。与shared_ptr不同，某个时刻只能有一个unique_ptr指向一个给定对象。当unique_ptr被销毁时，它所指向的对象也被销毁。     
与shared_ptr
不同，没有类似make_shared的标准库函数返回一个unqiue_ptr。当定义一个unique_ptr时，需要将其绑定到一个new返回的指针上。类似shared_ptr，初始化unique_ptr必须采用直接初始化形式：
```cpp
unique_ptr<double> p1;               //可以指向一个double的unique_ptr
unique_ptr<int> p2(new int(42));     //p2指向一个值为42的int
```
由于一个unique_ptr拥有它指向的对象，因此unique_ptr不支持普通的拷贝和赋值操作：
```cpp
unique_ptr<string> p1(new string("Stegosaurus"));
unique_ptr<string> p2(p1);  //错误，unique_ptr不知吃拷贝
unique_ptr<string> p3;
p3 = p2; //错误，unique_ptr不支持赋值
```
虽然不能拷贝或赋值unique_ptr，但可以通过调用release和reset将指针的所有权从一个unique_ptr转移给另一个unique：
```cpp
//将所有权从p1转移给p2
unique_ptr<string> p2(p1.release()); //release将p1置空
unique_ptr<string> p3(new string("Trex"));
//将所有权从p3转移给p2
p2.reset(p3.release()); //reset释放了p2原来指向的内存
```
release成员返回unique_ptr当前保存的指针并将其置空，因此，p2被初始化为p1原来保存的指针，而p1被置空。   
reset成员接受一个可选的指针参数，令unique_ptr重新指向给定的指针。如果unique_ptr不为空，它原来指向的对象被释放。因此，对p2调用reset释放了用“Stegosaurus”初始化的string所使用的内存，将p3对指针的所有权转移给p2，并将p3置空。    
调用release会切断unique_ptr和它原来管理的对象间的联系。release返回的指针通常被用来初始化另一个智能指针或给另一个智能指针赋值。在本例中，管理内存的责任简单地从一个智能指针转移给另一个。但是，如果不用另一个智能指针来保存release返回的指针，我们的策划能够序就要负责资源的释放。  
```cpp
p2.release();  //错误，p2不会释放内存，而且我们丢失了指针
auto p = p2.release();  //正确，但是必须要记得delete(p)
```



#### 传递unique_ptr参数和返回unique_ptr   
不能拷贝unique_ptr的规则有一个例外：我们可以拷贝或赋值一个将要被销毁的unique_ptr。最常见的例子是从函数返回一个unique_ptr：
```cpp
unique_ptr<int> clone(int p){
    //正确，从int*创建一个unique_ptr<int>
    return unique_ptr<int>(new int(p));
}
```
还可以返回一个局部对象的拷贝：
```cpp
unique_ptr<int> clone(int p){
    unique_ptr<int> ret(new int(p));
    reutrn ret;
}
```



### weak_ptr
weak_ptr是一种不控制所指向对象生存期的智能指针，它指向一个有shared_ptr管理的对象。   
当创建一个weak_ptr时，要用一个shared_ptr初始化它：
```cpp
auto p = make_shared<int>(42);
weak_ptr<int> wp(p);  //wp弱共享p，p的引用计数未改变
```
由于对象可能不存在，不能直接使用weak_ptr直接访问对象，必须调用lock。此函数检查weak_ptr所指向的对象是否存在。如果存在，lock返回一个指向共享对象的shared_ptr。










