### 思路

考虑了很久，没想出来

首先可以确定的是，如果节点有额外的存储空间就能很容易的解出来，那就考虑能不能增加额外的空间，但题设空间复杂度为1，那就说明有隐含条件没有想到。

![微信图片_20240108231836](.\环形列表II.assets\微信图片_20240108231836.jpg)

``` c++
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode *slow = head, *fast = head;
        while(fast && fast->next){
            fast = fast->next->next;
            slow = slow->next;
            if(fast == slow){
                ListNode *index1 = head;
                ListNode *index2 = fast;
                while(index1 != index2){
                    index1 = index1->next;
                    index2 = index2->next;
                }
                return index1;
            }
        }
        return NULL;
    }
};
```

