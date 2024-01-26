### 思路

一开始考虑按单个数组创建哈希，但是这样复杂度变得很大O（n^3）， 看了题解

``` c++
class Solution {
public:
    int fourSumCount(vector<int>& nums1, vector<int>& nums2, vector<int>& nums3, vector<int>& nums4) {
        unordered_map<int, int> m;
        int res=0;
        for(auto a : nums1){
            for(auto b : nums2){
                m[a+b]++;
            }
        }
        for(auto c : nums3){
            for(auto d : nums4){
                if(m.find(-c-d) != m.end()){
                    res += m[-c-d];
                }
            }
        }
        return res;
    }
};
```

