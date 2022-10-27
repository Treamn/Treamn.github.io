# C++语句


## 选择语句  
### switch语句  
#### case分支中的声明  
C++允许在switch语句的块内声明变量，但是不能不初始化：  
```cpp
void f(int i){
    switch(i){
        case 0:
            int x; //未初始化
            int y = 3; //错误：程序可能跳过该声明（显式初始化）
            string s; //错误：程序可能跳过该声明（隐式初始化）
    }
}
```
如果确实需要在switch语句中使用变量，最好把变量的声明和使用限定在一个块中。  

### 条件中的声明  
要想避免不限因误用变量，最好的办法是把变量的作用域限定在一个较小的范围内。此外，应该尽量延迟局部变量的定义，直到能给他赋初值为止。  
条件中的声明语句只能声明并初始化一个变量或const。

## 循环语句  
### 退出循环  
break语句负责“跳出”最外层switch语句或循环语句。    
当需要中途离开循环体的时候，可以使用break语句。通常情况下，应该让完整退出的条件位于while语句或for语句的条件部分。  
有时候不想完全脱出循环，只想到达循环体的末尾。continue可以跳过循环语句循环体的剩余部分：
```cpp 
void find_prime(vector<string>& v){
    for(int i = 0; i != v.size(); ++i){
        if(!prime(v[i])) contiune;
        return v[i];
    }
}
```
contiune之后继续执行递增循环变量的语句，然后检验昏眩条件是否满足。因此，find_prime()等价于下面的形式： 
```cpp
void find_prime(vector<string>& v){
    for(int i = 0; i != v.size(); ++i){
        if(!prime(v[i])){
            return v[i];
        }
    }
}
```

