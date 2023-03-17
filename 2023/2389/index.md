# Leetcode2389




## 题目描述   
给你一个长度为 n 的整数数组 nums ，和一个长度为 m 的整数数组 queries 。

返回一个长度为 m 的数组 answer ，其中 answer[i] 是 nums 中 元素之和小于等于 queries[i] 的子序列的最大长度  。

子序列是由一个数组删除某些元素（也可以不删除）但不改变剩余元素顺序得到的一个数组。



## 解题思路   
因为仅求元素和小于等于 queries[i] 的子序列的最大长度，因此可以先对 nums 进行排序，排序后不影响最终结果。     
首先，声明一个长度为 n+1 大小的数组 f 用于存储 nums 中前 i 个元素之和， 声明一个长度为 m 的数组 ans 用于存储最终结果。      
其次，由于求最大长度的子序列，因此使用 upper_bound() 查找 queries[i] 是否在 f 中出现，如果没出现，将 ans[i] 置零；如果出现，则将 ans[i] 置为当前位置-1。 此处 -1 是由于 upper_bound() 返回的是查找元素之后的位置，因此需要 -1。     
最终，返回ans，本题结束。  
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    vector<int> answerQueries(vector<int>& nums, vector<int>& queries) {
        int n = nums.size(), m = queries.size();
        sort(nums.begin(), nums.end());
        
        vector<int> f(n+1);
        for(int i = 0; i < n; i++){
            f[i+1] = f[i] + nums[i];
        }

        vector<int> ans(m);
        for(int i = 0; i < m; i++){
            ans[i] = upper_bound(f.begin(), f.end(), queries[i]) - f.begin() - 1;
        }
        return ans;
    }
};
```




