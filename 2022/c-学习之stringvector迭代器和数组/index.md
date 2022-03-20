# C++学习之string、vector、迭代器和数组



## string
string表示可变长的字符序列，使用string类型前需要包含string头文件，同时需要在命名空间中声明。需要注意，string对象对大小写敏感。
```cpp
#incude <string>
using std::string;
```
- 在最终的字符串中保留输入时的空白符，可以使用getline函数。
```cpp
string line;
getline(cin, line);
```
getline函数的参数是一个输入流和一个string对象，函数从给定的输入流中读入内容，遇到换行符则停止读入，然后将读入的内容存到string对象中。需要注意，如果一开始就读到换行符的话，就会得到一个空的string。  
- empty函数判断一个string对象是否为空。  
- size函数返回string对象的长度(无符号整数)。  
- 在使用‘+’对string进行操作时候，**必须确保**运算符两侧的对象至少有一个是string。
- cctype头文件中定义了一组标准库函数对string进行处理。
- 如果想要使用循环改变string中字符的值。必须将循环变量定义成引用类型。
---

## vector
vector存放某种给定类型的可变长序列，由于vector容纳者其他对象，因此也被称为容器。同string，要使用vector，必须包含相应的头文件，同时在命名空间中声明。
```cpp
#include <vector>
using std::vector;
vector<int>  ivec;         //ivec保存int类型的对象
vector<int>  ivec2(ivec);  //创建ivec2，并把ivec的值复制给ivec2
```
- 使用push_back函数可以往vector对象中压入元素。
- 需要注意，如果循环体内包含有向vector对象添加元素的语句，则不能使用范围for循环。
- 访问vector中元素的方法和访问string中字符的方法是差不多的，包括empty，size等。
- 如果在for循环中对vector元素继续赋值操作，则需要将循环变量设置为引用类型。
---

## 迭代器
在迭代器中，begin成员负责返回指向第一个元素的迭代器。end则负责指向尾元素的下一个位置（尾后元素）。  
- begin和end的返回值类型是由对象是否是常量决定，如果对象是常量则返回const_iterator，如果对象不是常量，则返回iterator。
- 如果对象只需要读操作而无需写操作的话，最好使用常量类型，此时可以使用**cbegin**和**cend**函数，上述函数不论对象本身是否为常量，返回值都是const_iterator。
### 解引用
解引用迭代器可以获得迭代器所指的对象，如果该对象恰好是类，就有可能希望进一步访问他的成员。  
例如，对于一个由字符串组成的vector对象来说，检查其元素是否为空，令it为迭代器，检查it所指字符串是否为空即可。需要注意，(*it)的圆括号必不可少。
```cpp
(*it).empty();  //先解引用，再调用empty
*it.empty();    //错误，it是个迭代器
```
或直接使用箭头运算符(->)，箭头运算符把解引用和成员访问两个操作结合在一起。
```cpp
(*it).empty();  //判断it是否为空
it->empty();    //判断it是否为空
```
- 但凡是使用迭代器的循环体，都不要向迭代器所属容器添加元素。
- string和vector的迭代器可以使用加减操作，使得迭代器每次的移动跨过多个元素。
- 在对两个迭代器进行运算时，参与运算的两个迭代器必须指向的是同一容器中的元素。
- 迭代器可用于实现二分搜索。
---

## 数组
数组也是存放类型相同的对象的容器，与vector类似，但与vector不同的是，数组的大小确定不变，不能随意向数组中增加元素。
- 如果不清楚元素的确切数量，则使用vector
- 数组在定义时需指定类型，不允许使用auto关键字推断。
- 不能将数组的内容拷贝给其他数组做初始值。
```cpp
int *ptrs[10];             //ptrs是含有10个整形指针的数组     
int (*Parray)[10] = &arr;  //Parray指向一个含有10个整数的数组
int (&arrRef)[10] = arr;   //arrRef引用一个含有10个整数的数组
```
### 指针和数组
- 当使用数组作为一个auto变量的初始值时，推断得到的类型是指针。
```cpp
int ia[] = {0,1,2};
auto ia2(ia);              //ia2是一个指针，指向ia的第一个元素
```
- 指向数组的指针可以当作迭代器进行使用，但是尾后元素需要自行获取。
```cpp
int arr[] = {0,1,2,3,4};
int *p = arr;     //p指向arr第一个元素
int *e = arr[5];   //e指向arr尾元素的下一位置
```
- C++11引入名为begin和end的函数用于获取数组首元素和尾后元素,这两个函数定义在iterator头文件中。
```cpp
int arr[] = {0,1,2,3,4};
int *p = begin(arr);   //p指向arr第一个元素
int *e = end(arr);     //e指向arr尾元素的下一位置
```
- 指针运算和迭代器运算类似。
- 假如结果指针指向了一个元素，则允许解引用该结果指针。
```cpp
int arr[] = {0,2,4,6,8};
int last = *(ar + 4);     //last值为8 
```
### 多维数组
多维数组就是数组的数组。
- C++11中新增or语句对多维数组进行处理。
```cpp
int ia[3][4];
size_t cnt = 0;
for(auto &row : ia)         //对于外层数组的每一个元素
    for(auto &col : row){   //对于内层数组的每一个元素
        col = cnt;
        cnt++;
    }
```
- 在遍历多维数组时，要将外围for循环的循环变量设置为引用型，这样是为了避免数组被自动转换为指针。
### 指针和多维数组
```cpp
int ia[3][4];
for(auto p = begin(ia); p != end(ia); p++){  //q指向内层元素的首元素
    for(auto q = begin(*p); q != end(*q); q++)
        cout << *q;  //输出q所指的值
}
```

