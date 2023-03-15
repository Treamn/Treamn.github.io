# C++学习之关联容器



## map  
map是关键字－值对的集合。　  
`map<string, int>　word`分别指定关键字的类型和值的类型。      　　　　
`word.first`和`word.second`分别用于取关键字和值。　







## set  
set是关键字的集合。  
`set<string> exclude`定义一个元素类型为string的set。　　　






## mutilmap和mutilset　　
一个map和set中的关键字必须是唯一的。但是mutilmap和mutilset没有此限制，它们允许多个元素具有相同的关键字。　　



## pair类型  

定义在头文件utility中。　　  

　　　
一个pair保存两个数据成员。　　　

pair的数据成员是public的，两个成员分别为first和second。　　　

map中每个元素是一个pair对象，包含一个关键字和一个关联的值。　　

## 关联容器操作   
- key_type: 此容器类型的关键字类型　　
- mapped_type:每个关键字关联的类型；只适用于map  
- value_ttype:　对于set，与key_type相同；对于map，为pair<const key_type, mapped_type>
```cpp
set<string>::value_type v1;         //v1是一个string
set<string>::key_type v1;           //v2是一个string
map<string, int>::value_type v3;    //v3是一个pair<const string, int>
map<string, int>::key_type v4;      //v4 is string
map<string, int>::mapped_type v5;   //v5 is int
```

### 关联容器迭代器　　

set的迭代器是const的  
虽然set类型同时定义了iterstor和const_iterator类型，但两种类型都只允许只读访问set中的元素。set中的关键字是const的，可以使用迭代器读取，但不能修改：  
```cpp
set<int> iset = {0,1,2,3,4,5,6,7,8,9};
set<int>::iteratir set_it = iset.begin();
if(set_it != iset.end()){
    *set_it = 42;            //wrong, elements in set s read only
    cout << *set_it << endl; //right
}
```

通常不对关联容器使用泛型算法。  
 

## 添加元素   
可使用insert()向map和set中添加元素，需要注意，map中的元素是pair的，因此在向map中插入元素时，需要创建pair。
```cpp
vector<int> ivec{2,4,6,8,2,4,6,8};
set<int> set2;
set2.insert(ivec.begin(), ivec.end());
```
```cpp
map<string, int> word_count;
word_count.insert({word,1});
word_count.insert(make_pair(word,1));
word_count.insert(pair<string, int>(word, 1));
word_count.insert(map<string, int>::value_type(word,1));
```

对于不包含重复关键字的容器，添加单一元素的insert和emplace返回一个pair。pair的first成员是一个迭代器，second成员是一个bool值。如果关键字已经在容器中，则insert什么也不做，且pair的bool部分为false。如果关键字不存在，元素被插入容器中，且bool值为true。     
如果希望添加具有相同关键字的多个元素，则应该使用multimap而不是map。  


## 删除元素   
关联容器提供一个额外的erase操作，接受一个key_type参数。此版本删除所有匹配给定关键字的元素(如果存在的话)，返回实际删除的元素的数量。


## map的下标操作  
map下标运算符接受一个所引，获取与此关键字相关联的值，如果关键字不在map中，会为它创建一个元素并插入到map中，关联值将进行值初始化。  


## 对map使用find代替下标操作  
如果只想知道一个元素是否在map中，而不想改变map。这样就不能使用下标进行操作，应该使用find。  



## 在multimap和multiset中查找元素  
如果一个multimap和multiset中有多个元素具有给定关键字，则这些元素在容器中会相邻存储。  
查找在multimap和multiset中出现的元素，可以使用lower_bound和upper_bound。lower_bound返回的迭代器会指向第一个具有给定元素关键字的元素，upper_bound返回的迭代器会指向最后一个具有给定元素关键字之后的位置。如果元素不在容器中，则lower_bound和upper_bound返回相同的迭代器。  
```cpp
mulitmap<string, int> author;
for(auto beg = autohor.lower_bund('sreach_item'), end = author.upper_bound('search_item'); beg != end; beg++){
    cout << beg->second << endl;
}
```

### equal_range函数  
此函数接受一个关键字，返回一个迭代器pair。如果关键字存在，第一个迭代器指向第一个与关键字匹配的元素，第二个迭代器指向最后一个匹配元素之后的位置。  
```cpp
for(auto pos = author.equal_range('search_item'); pos.first != pos.second; pos.first++){
    cout << pos.first->second << endl;
}
```


