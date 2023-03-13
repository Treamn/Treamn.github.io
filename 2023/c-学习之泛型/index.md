# C++学习之泛型


泛型算法可用于不同类型的容器和不同类型的元素,大多数算法定义域algorothm库中。
## 只读算法
  
### find()  
```cpp
#include <algorithm>

vector<int> vec{1,2,3,4,5};
int val = 4;
auto result = find(vec.cbegin(), vec.cend(), val);
```
查找vec中是否含有元素4。如果有，返回true，否则返回flase。范查找范围可自行拟定。    
当只对容器中的元素进行读取操作时，迭代器选用cbegin和cend。

### count()  
conunt()用法同find()，用于统计元素在指定容器中出现的次数。  

### accumulate()  
accumulate()对容器指定范围内元素求和，初始值可自行指定。
```cpp
int sum = accumulate(vec.cbegin(), vec,cend(), 0);
```
此代码功能越等于
```cpp
int sum = 0;
for(auto &x : vec){
    sum ++ x;
}
```

### equal()  
确定两个序列是否保存相同的值，如果所有对应元素都相等，返回itrue，否则返回false。
```cpp
vector<int> vec1;
equal(vec.cbegin(), vec.cend(), vec1.begin());
```
注意，vec1中的值至少要与vec一样多。范围可自行指定。


## 写容器算法  
### fill()   
```cpp
fill(vec.begin(), vec.end(), 0);//将所有元素置为0
```
fill()将给定范围内所有元素重置为给定的元素。  

### fill_n()  
```cpp
fill_n(vec.begin(), vec.size(), 0);
```
fill_()接受一个单迭代器，一个计数器和一个值。将给定值赋予迭代器指向的元素开始的指定个元素。    
** 注意: ** fill_n()不可用于空容器。    


### back_inserter()  
定义在头文件iterator中的一个函数。  
通常使用back_inserter()来创建一个迭代器。 
```cpp
//正确，back_insert创建一个插入迭代器，用来向vec中添加元素
fill_n(back_inserter(vec), 10, 0);
```

## 拷贝算法  
### copy()  
该算法接受三个迭代器，前两个表示一个输入范围，第三个表示目的序列的起始位置。   
传递给copy的目的序列至少要包含和输入序列一样多的元素。  
```cpp
int a1[] = {0,1,2,3,4,5,6,7,8,9};
int a2[sizeof(a1) / sizeof(*a1)];
auto ret = copy(begin(a1), end(a1), a2);
```
copy返回的是目的位置迭代器的值。ret恰好指向拷贝到a2的尾后元素的位置。   


### replace()  
此算法接受四个参数：前两格式迭代器，表示输入序列；第三个是要搜索的值，最后一个是新值。
```cpp
replace(list.begin(), list.end(), 0, 42);
//将list中所有的0改为42
```


### replace_copy()  
此算法接受额外第三个迭代器参数，指出调整后序列的保存位置：
```cpp
replace_copy(list.cbegin(), list.cend(), back_inserter(ivec), 0, 42);
```
调整后，原序列不变，新序列保存在ivec中。




