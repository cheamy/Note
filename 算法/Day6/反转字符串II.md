### 思路

一开始想的是遍历一遍，后来看题解发现，其实for循环的时候， i+=（2*k）就能很方便的模拟题意

``` c++
class Solution {
public:
    string reverseStr(string s, int k) {
        int flag = 1;
        for(int i = 0; i < s.size(); i+=(2*k)){
            if(i + k > s.size()){
                reverse(s.begin()+i, s.end());
            }else{
                reverse(s.begin()+i, s.begin()+i+k);
            }
        }
        return s;
    }
};
```

