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


