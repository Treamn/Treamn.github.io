# Review


### cin.get()  
在接收到任意输入后再执行下一条语句。

### 声明语句  
**使用变量前必须声明它。**  
尽可能在首次使用变量前声明它。  
要将信息项存储在计算机中，必须之处信息的存储位置和所需的内存空间。在C++中，完成这种任务相对简便的方法是使用声明语句之处存储类型，并提供位置标签。  
声明通常指出要存储的数据类型和程序对存储在这里的数据使用的名称。  
程序中的声明语句叫做定义声明语句，简称定义。这意味着它将导致编译器为变量分配内存空间。  


### 赋值语句  
C++中有一项不寻常的特性，可以连续使用赋值运算符。  
```cpp
int s;
int b;
int y;
y = b = s = 88;
```
赋值从右往左进行，先将88赋给s，再将s的值赋给b，最后将b的值赋给y。 


### 函数  
原型只描述函数接口，描述的是发送给函数的信息和返回的信息。  
而定义中包含函数的代码。  
应该在首次使用函数之前提供原型，通常的做法是把原型放到main()函数定义的前面。  
```cpp
void simon(int n){
    cout << n << "times." <<endl;
}
int simple = simon(3); //错误，因为simon没有返回值。
```

****

### 运算符sizeof  
可以对类型名或变量名使用sizeof运算符。对类型名(如int)使用sizeof运算符时，应将名称放在括号中；但对变量名使用sizeof运算符，括号则是可选的。
```cpp
int max = INT_MAX;
cout << "int is " << sizeof(int) << "bytes."
cout << "int is " << sizeof(max) << "bytes."
cout << "int is " << sizeof max << "bytes."
```

### 头文件limits  
头文件climits定义了符号常量来表示类型的限制。详细可[参考](https://www.apiref.com/cpp-zh/cpp/header/climits.html)。

### C++11初始化方式  
```cpp
int emus{7}  //set eum to 7
int rheas = {12}  // setrhea to 12
int rocs = {}  // set roc to 0
int psychics{}  //set psychics to 0
```






