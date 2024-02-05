
# 1. Problem description
Given the `head` of a singly linked list and two integers `left` and `right` where `left <= right`, reverse the nodes of the list from position `left` to position `right`, and return _the reversed list_.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/02/19/rev2ex2.jpg)

**Input:** head = [1,2,3,4,5], left = 2, right = 4
**Output:** [1,4,3,2,5]

**Example 2:**

**Input:** head = [5], left = 1, right = 1
**Output:** [5]

**Constraints:**

- The number of nodes in the list is `n`.
- `1 <= n <= 500`
- `-500 <= Node.val <= 500`
- `1 <= left <= right <= n`
# 2. Intuition
My intuition follows below image.
![](../../../../images/Pasted%20image%2020240205143227.png)

- do normal linked list until the index is right before 'left' position
- start reversing the linked list from 'left' to 'right'
- If we are at right position, finish reversing steps and 
	- make the left-1 node point to node at 'right'
	- make the node at 'left' point to node at 'right+1'

To generalize this algorithm for every cases, we need to add dummy node in front of the linked list to avoid edge case where left position is equal to '1'

# 3. Solution

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* reverseBetween(ListNode* head, int left, int right) {
        if(left == right) return head;

        ListNode* dummy = new ListNode(0);
        dummy->next = head;

        int pos = 0;
        ListNode* last = nullptr;
        ListNode* prev = nullptr;
        ListNode* temp = nullptr;
        ListNode* cur = dummy;

        while(cur) {
            temp = cur->next;

            if(pos == left-1) last = cur; 
            else if(left <= pos && pos < right) { cur->next = prev; }
            else if(pos == right) {
                last->next->next = temp;
                last->next = cur;
                cur->next = prev;
                break;
            }

            prev = cur;
            cur = temp;
            pos++;
        }

        return dummy->next;
    }
};
```