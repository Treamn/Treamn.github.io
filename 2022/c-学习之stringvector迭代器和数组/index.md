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
- front函数返回字符串中的首位字符。用法同back函数。
- back函数返回字符串中的末位字符。
```cpp
int main(){
    {
        string s("Exemplary");
        char& back = s.back();
        back = 's';
        cout << s << endl;    // Exemplars
    }
    {
        string const c("Exemplary");
        char const& back = c.back();
        cout << back << endl; // y
    }
}
```
- capacity函数返回当前为string分配的存储。
```cpp
#include <iostream>
#include <string>
using namespace std;

void show_capacity(string const& s){
    cout << " " << s << " has capacity " << s.capacity() << endl;
}

int main(){
    string s("Exemplar");
    show_capacity(s);  // Exemplar has capacity 15

    s += " is an example.";
    show_capacity(s);  //  Exemplar is an example. has capacity 30
}
```
- find函数用于搜索指定范围中满足特定判别标准的首个元素的迭代器，若找不到则返回last。
```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <iterator>
using namespace std;

int main(){
    vector<int> v{1,2,3,4};
    int n1 = 3;
    int n2 = 5;
    
    auto res1 = find(begin(v), end(v), n1);
    auto res2 = find(begin(v), end(v), n2);

    (res1 != end(v)) ? cout << "v contains " << n1 << endl : cout << "v does nor contain " << n1 << endl;
    // v contains 3
    (res2 != end(v)) ? cout << "v contains " << n2 << endl : cout << "v does nor contain " << n2 << endl;
    // v does nor contain 5
}
```
- find_first_of，寻找等于给定字符序列中字符之一的首个字符。搜索只考虑区间 [pos, size()) 。若区间中不存在字符，则返回 npos 。
- find_last_of, 寻找等于给定字符序列中字符之一的最后字符。不指定准确的搜索算法。搜索只考虑区间 [0, pos] 。若区间中不存在这种字符，则返回 npos 。用法同find_first_of。
```cpp
#include <iostream>
#include <string>
using namespace std;

int main(){
    const string s("Hello World!");
    const string search_str("o");
    const char* search_cstr = "Good Bye!";
    cout << s.find_first_of(search_str) << endl;
    // 4
    cout << s.find_first_of(search_str, 5) << endl;
    // 7
    cout << s.find_first_of(search_cstr) << endl;
    // 1
    cout << s.find_first_of(search_cstr, 5, 2) << endl;
    // 7
}

```
- find_first_not_of, 寻找不等于给定字符序列中任何字符的首个字符。搜索只考虑区间 [pos, size()) 。若区间中不存在字符，则将返回 npos 。
- find_lat_not_of, 寻找不等于给定字符序列中任何字符的最后字符。搜索只考虑区间 [0, pos] 。若区间中不存在这种字符，则返回 npos 。
```cpp
#include <string>
#include <iostream>
 
int main() {
    std::string to_search = "Some data with %MACROS to substitute";
 
    std::cout << "Before: " << to_search << '\n';
 
    auto pos = std::string::npos;
    while ((pos = to_search.find('%')) != std::string::npos) {
        // 宏名中容许大写字母、小写字母和数字
        const auto after = to_search.find_first_not_of("ABCDEFGHIJKLMNOPQRSTUVWXYZ"
                                                       "abcdefghijklmnopqrstuvwxyz"
                                                       "0123456789", pos + 1);
 
        // 现在 to_search[pos] == '%' 而 to_search[after] == ' ' （在 'S' 后）
 
        if(after != std::string::npos)
            to_search.replace(pos, after - pos, "some very nice macros");
    }
 
    std::cout << "After: " << to_search << '\n';
}
Before: Some data with %MACROS to substitute
After: Some data with some very nice macros to substitute
```
- rfind,寻找等于给定字符序列的最后子串。搜索始于 pos ，即找到的子串必须不始于 pos 后的位置。若将 npos 或任何不小于 size()-1 的值作为 pos 传递，则在整个字符串中搜索。即从尾部开始寻找最后出现的给定子串。
```cpp
#include <string>
#include <iostream>
 
void print(std::string::size_type n, std::string const &s)
{
    if (n == std::string::npos) {
        std::cout << "not found\n";
    } else {
        std::cout << "found: \"" << s.substr(n) << "\" at " << n << '\n';
    }
}
 
int main()
{
    std::string::size_type n;
    std::string const s = "This is a string";
 
    // 从字符串尾反向搜索
    n = s.rfind("is");
    print(n, s);  //5
    // 从位置 4 反向搜索
    n = s.rfind("is", 4);
    print(n, s);  //2
    // 寻找单个字符
    n = s.rfind('s');
    print(n, s);  //10
    // 寻找单个字符
    n = s.rfind('q');
    print(n, s);  //not found
}

```

- erase函数用于从string中移除指定字符。
```cpp
#include <iostream>
#include <algorithm>
#include <string>
using namespace std;

int main(){
    string s("This is an example");
    s.erase(0,5);
    cout << s <<endl; //is an example

    s.erase(find(s.begin(), s.end(), ' '));
    cout << s <<endl; //isan example

    s.erase(s.find(' ')); // 从‘ ’擦除到尾部
    cout << s << endl;//isan

}
```
- clear函数如同执行erase(begin(),end())，即从string中移除所有字符。
- insert函数用于向string的index位置插入指定字符。
- append函数可以附字符到string结尾。
```cpp
#include <string>
#include <iostream>
 
int main() {
    std::basic_string<char> str = "string";
    const char* cptr = "C-string";
    const char carr[] = "Two and one";
 
    std::string output;
 
    // 1) 后附 char 3 次。
    // 注意，这是仅有的接受 char 的重载。
    output.append(3, '*');
    std::cout << "1) " << output << "\n";
 
    //  2) 后附整个 string
    output.append(str);
    std::cout << "2) " << output << "\n";
 
    // 3) 后附字符串的一部分（此情况为最后 3 个字母）case)
    output.append(str, 3, 3);
    std::cout << "3) " << output << "\n";
 
    // 4) 后附 C 字符串的一部分
    // 注意，因为 `append` 返回 *this ，我们能一同链式调用
    output.append(1, ' ').append(carr, 4);
    std::cout << "4) " << output << "\n";
 
    // 5) 后附整个 C 字符串
    output.append(cptr);
    std::cout << "5) " << output << "\n";
 
    // 6) 后附范围
    output.append(std::begin(carr) + 3, std::end(carr));
    std::cout << "6) " << output << "\n";
 
    // 7) 后附 initializer_list
    output.append({ ' ', 'l', 'i', 's', 't' });
    std::cout << "7) " << output << "\n";
}
```
- start_withend_with函数检查string是否始于给定前缀或是否终于给定后缀。（C++20）
- contain检测string中是否含有指定子串。（C++23）
- replace函数用新字符串替换string中指定位置字符。
```cpp
#include <iostream>
#include <string>
using namespace std;

int main(){
    string str("hello world");
    str.replace(0,5, "red");
    cout << str << endl; // red world
    str.replace(str.begin(), str.begin()+3, 2 , 'A');
    cout << str << endl; // AA world
}
```
- substr函数返回子串[pos, pos+count)，若请求的子串越过字符串的结尾，即count大于size()-pos，则返回的子串为[pos, size())。
```cpp
#include <string>
#include <iostream>
using namespace std;

int main(){
    string s("0123456789abcdefghij");
    cout << s.substr(10) << endl;
    // abcdefghij
    cout << s.substr(5, 3) << endl;
    // 567
    cout << s.substr(s.size()-3, 50) << endl;
    // hij
}
```
- swap函数交换string和other的内容。
```cpp
#include <iostream>
#include <string>
using namespace std;

int main(){
    string a("aaa");
    string b("bbb");
    a.swap(b);
    cout << a << " " << b;
    // bbb aaa
}
```
- stoi,stol,stoll 转译字符串 str 中的有符号整数值。数字需要在string的首位置。
- stof,srod,stold 转译 string str 中的浮点值。
- stoul,stoull 转译字符串 str 中的无符号整数值。
```cpp
#include <iostream>
#include <string>
 
int main()
{
    std::string str1 = "45";
    std::string str2 = "3.14159";
    std::string str3 = "31337 with words";
    std::string str4 = "words and 2";
 
    int myint1 = std::stoi(str1);
    int myint2 = std::stoi(str2);
    int myint3 = std::stoi(str3);
    // 错误： 'std::invalid_argument'
    // int myint4 = std::stoi(str4);
 
    std::cout << "std::stoi(\"" << str1 << "\") is " << myint1 << '\n';
    std::cout << "std::stoi(\"" << str2 << "\") is " << myint2 << '\n';
    std::cout << "std::stoi(\"" << str3 << "\") is " << myint3 << '\n';
    //std::cout << "std::stoi(\"" << str4 << "\") is " << myint4 << '\n';
}
std::stoi("45") is 45
std::stoi("3.14159") is 3
std::stoi("31337 with words") is 31337
```
- to_string 将需要转换的数据转换成字符串。 

---

## vector
vector存放某种给定类型的可变长序列，由于vector容纳者其他对象，因此也被称为容器。同string，要使用vector，必须包含相应的头文件，同时在命名空间中声明。
```cpp
#include <vector>
using std::vector;
vector<int>  ivec;         //ivec保存int类型的对象
vector<int>  ivec2(ivec);  //创建ivec2，并把ivec的值复制给ivec2
```
- front和back函数分别实现访问当前容器的首位元素或末位元素。
```cpp
#include <vector>
#include <iostream>
 
int main()
{
    std::vector<char> letters {'o', 'm', 'g', 'w', 't', 'f'};
 
    if (!letters.empty()) {
        std::cout << "The first character is: " << letters.front() << '\n';  // o
        std::cout << "The last character is: " << letters.back() << '\n';  // f
    }  
}
```
- begin,cbegin,rbegin迭代器见上述string部分内容。
- max_size函数返回容器可容纳元素数量最大值。
- shrink_to_fit 请求移除未使用的容量,是减少 capacity() 到 size()非强制性请求。
- emplace（待补充）
- emplace_back（待补充）


- 使用push_back函数可以往vector对象尾压入元素。
- 使用pop_back函数可以移除vector对象尾元素。
```cpp
#include <iostream>
#include <vector>
using namespace std;

void print(vector<int>& v){
    for(auto i : v)
        cout << i << " ";
    cout << endl;
}
int main(){
    vector<int> v{1,2,3,4,5};
    print(v); // 1 2 3 4 5
    v.push_back(6);
    print(v); // 1 2 3 4 5 6
    v.pop_back();
    v.pop_back();
    print(v); // 1 2 3 4
}
```
- back_inserter函数可以往容器尾端添加元素。(C++20)。
- front_inserter函数可以向容器首端添加元素。(C++20)，用法同back_insert函数。
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <iterator>
using namespace std;

int main(){
    vector<int> v{1,2,3,4,5};
    fill_n(back_inserter(v), 3, -1);
    for(auto i : v)
        cout << i << " ";  // 1,2,3,4,5,-1,-1,-1

}
```
- fill_n函数（待整理）
- 需要注意，如果循环体内包含有向vector对象添加元素的语句，则不能使用范围for循环。
- 访问vector中元素的方法和访问string中字符的方法是差不多的，包括empty，size等。
- 如果在for循环中对vector元素继续赋值操作，则需要将循环变量设置为引用类型。
---

## 迭代器
在迭代器中，begin成员负责返回指向第一个元素的迭代器。end则负责指向尾元素的下一个位置（尾后元素）。  
- begin和end的返回值类型是由对象是否是常量决定，如果对象是常量则返回const_iterator，如果对象不是常量，则返回iterator。
- 如果对象只需要读操作而无需写操作的话，最好使用常量类型，此时可以使用**cbegin**和**cend**函数，上述函数不论对象本身是否为常量，返回值都是const_iterator。
- rbegin和rend函数，与begin、end函数用法相同，但是不同之处在于，rbegin指向最后一个元素，rend则指向首位字符的前一位置。
```cpp
#include <iostream>
#include <string>
#include <algorithm>
#include <string>
using namespace std;

int main(){
    string s("Exenplar!");
    *s.rbegin() = 'y';
    cout << s << endl;  // Exemplary

    string c; 
    copy(s.crbegin(), s.crend(), back_inserter(c));
    cout << c << endl;  // yralpmexE
}
```

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

