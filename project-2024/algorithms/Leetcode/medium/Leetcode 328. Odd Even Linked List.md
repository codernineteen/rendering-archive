
- #solved #medium #linked-list #in-place
- took 14min to solve.
# 1. Problem description

Given the `head` of a singly linked list, group all the nodes with odd indices together followed by the nodes with even indices, and return _the reordered list_.

The **first** node is considered **odd**, and the **second** node is **even**, and so on.

Note that the relative order inside both the even and odd groups should remain as it was in the input.

You must solve the problem in `O(1)` extra space complexity and `O(n)` time complexity.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/03/10/oddeven-linked-list.jpg)

**Input:** head = [1,2,3,4,5]
**Output:** [1,3,5,2,4]


# 2. Intuition

Pre-process :
- Store the very first even node.

Common steps in a traversal :
- Update last node of odd indices every time.
- store an address of next node first
- make current node point to next of next node.
- update current node as temporary node that we stored before.

**Edge case**
- if a given head is null pointer, return the head right away.

![](../../../../images/Pasted%20image%2020240206121207.png)


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
    ListNode* oddEvenList(ListNode* head) {
        if(head == nullptr) return head;

        int pos = 1;
        
        ListNode* cur = head;
        ListNode* firstEven = cur->next;
        ListNode* lastOdd = head;
        ListNode* temp;

        while(cur->next) {
            // odd node.
            if(pos % 2 == 1) lastOdd = cur;

            temp = cur->next;
            cur->next = cur->next->next;
            cur = temp;

            pos++;
        }

        if(pos % 2 == 1) lastOdd = cur;
        lastOdd->next = firstEven;
        return head;
    }
};
```
