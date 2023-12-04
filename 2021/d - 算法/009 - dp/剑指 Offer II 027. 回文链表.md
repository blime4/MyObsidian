
[[算法-回文]]
[[算法-第三遍]]


```c++
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        vector<int> tmp;
        while(head){
            tmp.push_back(head->val);
            head = head->next;
        }
        int l = 0, r = tmp.size() -1;
        while(l<r){
            if(tmp[l++] != tmp[r--]) return false;
        }
        return true;
    }
};
```