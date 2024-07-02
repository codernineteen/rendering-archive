
- #solved #medium #linked-list 
- took 41 min to solve

# 1. Problem description
You are given the head of a singly linked-list. The list can be represented as:

L0 → L1 → … → Ln - 1 → Ln

_Reorder the list to be on the following form:_

L0 → Ln → L1 → Ln - 1 → L2 → Ln - 2 → …

You may not modify the values in the list's nodes. Only nodes themselves may be changed.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/03/04/reorder1linked-list.jpg)

**Input:** head = [1,2,3,4]
**Output:** [1,4,2,3]

**Constraints:**

- The number of nodes in the list is in the range `[1, 5 * 104]`.
- `1 <= Node.val <= 1000`

# 2. Intuition
Let's skim through the constraints first.
1. there is no mention about memory usage, so maybe we can use additional data structure to solve this problem.
2. The max length of linked list is 50000. It seems that we may not be able to solve with brute force approach.

To solve this problem in O(n) time complexity, i made up three neccessary traversals.

1. Get length
	Because the input is singly linked list, we can't know the length of the list without traversal.
2. Forward and reverse jumping insertion
	With this visualization, we can see that we need to insert elements forward until we reach $L_n/2$ elements.
	After we reach the last slot in the vector, then we start to insert elements in a reverse order by decrementing iterator two times at once.
	![](../../../../../images/Pasted%20image%2020240303140643.png)
3. Make current node point to next elements in the vector
	While iterating the vector, we need to make the current element point ot next node so that we can reorder linked-list appropriately.
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
    void reorderList(ListNode* head) {
        if(head->next == nullptr || head->next->next == nullptr) return; // when length of linked list is shorter than two

        int len = 0;
        ListNode* cur = head;
        while(cur != nullptr) {
            len++;
            cur = cur->next;
        }

        cur = head;
        vector<ListNode*> vec(len);
        int it = 0;
        bool isForward = true;

        while(cur != nullptr) {
            vec[it] = cur;

            if(isForward) it+=2;
            else it -= 2;

            if(it == len) {isForward = false; it--;}
            else if(it > len) {isForward = false; it = len-2;}
    
            cur = cur->next;
        }
    

        for(int i=0; i < len-1; i++) {
           vec[i]->next = vec[i+1];
        }
        vec[len-1]->next = nullptr; // mark next of last node as nullptr
    }
};
```