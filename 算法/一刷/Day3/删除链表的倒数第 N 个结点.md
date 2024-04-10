### 思路

双指针法，秒了

``` c++
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* dummyHead = new ListNode(0, head);
        ListNode *fast = dummyHead, *slow = dummyHead;
        while(n--&&fast){
            fast = fast->next;
        }
        fast = fast->next;
        while(fast){
            fast = fast->next;
            slow = slow->next;
        }
        ListNode* del = slow->next;
        slow->next = slow->next->next;
        delete del;
        return dummyHead->next;
    }
};
```

