# C++学习之运算符


## 优先级与结合律
- 优先级会影响程序的正确性，这一点也适用于解引用和指针运算。
```cpp
int ia[] = {0,2,4,6,8};
int last = *(ia + 4);    //此时last值为ia[4]的值
last = *ia + 4;          //此时last值为[0]+4
```
- 进行比较运算时，除非比较的对象是布尔类型，否则不要使用布尔字面值true和false作为比较对象。
---

## 递增和递减运算符
- 前置版本  
首先将运算对象加一或减一，然后将改变后的对象作为求值结果(先加再用)。
- 后置版本  
首先将运算对象加一或减一，但求值结果是运算对象改变之前那个值的副本(先用再加)。
```cpp
int i = 0, j;
j = ++i;  // j = 1, i = 1
j = i++;  // j = 1, i = 2
```
---

## 算数转换
- 运算符的运算对象将被转换为最宽的类型  
例如，如果一个运算对象的类型是long double，那么不论另一个运算对象的类型是什么，它都会被转换成long double型。  
另一种更普遍的情况是，当表达式中既有浮点类型和整数类型时，整数型将被转换成相应的浮点类型。
---

## 其他隐式类型转换
- 数组转换为指针  
```cpp
int ia[10];
int* ip = ia; //ia转换成指向数组首元素的指针
```
- 指针的转换  
  - 转换成布尔类型  
  ```cpp
  char *cp = get_string();
  if(cp) /*...*/       //如果指针不是0，条件为真
  while(*cp) /*...*/   //如果*cp不是空字符，条件为真
  ```
  - 转换成常量
  ```cpp
  int i;
  const int &j = i;    //非常量转换为const int的引用
  const int *p = &i;   //非常量的地址转换为const的地址
  int &r = j, *q = p;  //错误，不允许const转换为非常量
  ```
---

## 显式转换
一个命名的强制类型转换具有如下形式：  
` cast-name<type>(expression) `  
type是转换的目标类型，expression是要转换的值。casy-name是static_cast、dynamic_cast、const_cast和reinterpret_cast中的一种。


- static_cast  
适用于任何具有明确定义的类型转换，只要不包含底层const，都可以使用static_cast。  
当把较大的算数类型赋值给较小的类型时，static_cast非常有用。  
static_cast同样适用于编译器无法自动执行的类型转换。
```cpp
#include <iostream>
#include <vector>
using namespace std;

struct B{
    int m = 0;
    void hello() const{
        cout << "this is B" << endl;
    }
};

struct D:B{
    void hello() const{
        cout << "this is D" << endl;
    }
};

int main(){
    D d;
    B& br = d;
    br.hello(); // this is B
    D& another_d = static_cast<D&>(br);
    another_d.hello(); // this is D

}
```





- const_cast   
只能改变运算对象的底层const,可以用来移除常量性或易变性。
函数指针和成员函数指针无法用于const_cast。
const_cast使得到非const类型的应用或指针能够实际指代const对象，货或到非volatile类型的引用或指针能够实际指代volatile对象。
```cpp
#include <iostream>
using namespace std;

struct type{
    int i;
    type(): i(3){} // 构造函数
    void f(int v) const{
        // this->i = v;
        const_cast<type*>(this)->i = v; 
    }
};

int main(){
    int i = 3;
    const int& rci = i;
    cout << rci << endl; // 3
    const_cast<int&>(rci) = 4;
    cout << rci << endl; // 4

    type t;
    t.f(5);
    cout << t.i << endl;

}
```



- reinterpret_cast  
与static_const不同，但与const_cast类似。reinterpret_cast是一个编译时指令，通常为运算对象的位模式提供较低层次上的重新编译。

```cpp
int *io;
char *pc = reinterpret_cast<char*>(ip);
string str(pc) //错误，pc所指的真实对象是一个int而非字符。
```
