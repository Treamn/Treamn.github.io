# C++学习之语句及异常


## 简单语句
- 空语句  
空语句中只含有一个单独的分号。  
```cpp
;  //空语句
```
如果程序的某个地方，语法上需要一条语句，但是逻辑上不需要，此时应该使用空语句。  
例如，想读取输入流的内容直到遇见一个特定值，除此之外什么也不做。
```cpp
while(cin >> s && s != sought)
    ;  //空语句
```
---
## 条件语句
- switch语句  
case关键字和它对应的值一起被称为case标签，case标签必须是整型常量表达式。  
如果没有任何一个case标签能够匹配上switch表达式的值，程序将执行紧跟在default标签之后的语句。
---

## 循环语句
- while  
只要条件为真，while语句就重复的执行循环体。  
当不确定要迭代多少次时，使用while循环比较合适。或想要在循环结束后访问循环控制变量。
- 传统for语句  
```cpp
for(init-statement; condition; expression)
    statement;
```
- 范围for语句
```cpp
for(declaration: expression)
    statement;
```
expression表示的必须是一个序列，如数组或vector或string等类型的对象，这些类型的共同特点是拥有能返回迭代器的begin和end成员。  
不能通过范围for语句增加vector对象的元素。因为在范围for语句中预存了end()值，一旦添加元素，end函数的值可能无效。

- do while语句  
与while语句非常相似，但是do while语句先执行循环体再检查条件。
```cpp
do
    statement;
while (condition);
```
不允许在条件部分定义变量。

---

## 跳转语句
- break语句  
终止离它最近的while、do while、for或switch语句，并从这些语句后第一条语句开始执行。

- continue语句
终止最近的循环中的当前迭代，并立即开始下一次迭代。
```cpp
string buf;
while(cin >> buf && !buf.empyt()){
    if(buf[0] != '_')
        continue;  //读取下一输入
}
```
---
## try语句块和异常处理
- throw表达式  
throw表达式包含关键字throw和紧随其后的一个表达式，其中表达式的类型就是抛出的异常类型。
```cpp
if(item1.isbn() != item2.isbn())
    throw runtime_error("Data must refer to same ISBN");
```
runtime_error是标准异常类型的一种，定义在stdexcept头文件中。
- try语句块
```cpp
try{
    program-statement;
}catch(exception-declaration){
    handler-statements;
}catch(exception-declaration){
    handler-statements;
}
```
catch子句包含三部分：关键字catch、括号内一个对象的声明以及一个块。选中某个catch子句处理异常后，执行与之对应的块。catch一旦执行完成，则跳转到try语句块最后一个catch子句**之后**的那条语句继续执行。
```cpp
while(cin >> item1 >> item2){
    try{
        //添加两个sales_item对象的代码
        //如果添加失败，抛出一个runtime_error异常
    }catch(runtime_error err){
        //提醒用户两个ISBN必须一致，询问是否重新输入
        cout << err.what() << "\nTry Again? Entry y or n" << endl;
        char c;
        cin >> c;
        if(!cin || c == 'n')
            break;
    }
}
```
---
## 标准异常  
- exception  
exception头文件定义最通用的异常类exception。只报告异常的发生，不提供任何额外信息。
- stdexcept  
异常类              |  异常类型
-------------------|--------------
exception          |  最常见的问题   
runtime_error      |  只有在运行时才能检测出的问题
range_error        |  运行时错误：生成的结果超过了有意义的值域范围
overflow_error     |  运行时错误：计算上溢
underflow_error    |  运行时错误：计算下溢
logic_error        |  程序逻辑错误
domain_error       |  逻辑错误：参数对应的结果值不存在
invalid_error      |  逻辑错误：无效参数
length_error       |  逻辑错误：试图创建一个超出该类型最大长度的对象
out_of_range       |  逻辑错误：使用一个超出有效范围的值

- new头文件定义bad_alloc异常类型，将在之后介绍
- type_info头文件定义了bad_cast异常类型，将在之后介绍

只能以默认初始化的方式初始化exception、bad_alloc和bad_cast对象，不允许为这些对象提供初始值。  
其他异常类型的行为刚好相反：应使用string或C风格字符串初始化这些类型的对象，但不允许使用默认初始化的方式。当创建此类对象时，必须提供初始值，该初始值含有错误相关的信息。
