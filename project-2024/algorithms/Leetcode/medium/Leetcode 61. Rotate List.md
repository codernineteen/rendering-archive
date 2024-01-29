
- #solved #medium #array #linked-list 
- took 20min to solve

# 1. Problem description
Given the `head` of a linked list, rotate the list to the right by `k` places.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/11/13/rotate1.jpg)

**Input:** head = [1,2,3,4,5], k = 2
**Output:** [4,5,1,2,3]

**Constraints:**

- The number of nodes in the list is in the range `[0, 500]`.
- `-100 <= Node.val <= 100`
- `0 <= k <= 2 * 109`

# 2. Intuition

This problem is quite similar to [189.Rotate an array](https://leetcode.com/problems/rotate-array/description).
The only difference is that we need to solve same kind of problem against Linked list data structure.

We need to find the length of given linked list first by traversing it first.

I made two ideas to solve this problem
- In a naive way, copy all the values into a vector and use reverse algorithm to rotate it. Then, create a new linked list again.
- Another approach is that Changing rotate part as head of linked list and make previous tail point to previous head so that we can solve a problem in-place memory.

Because the approach is totally same with problem 189, i will skip the explanation.
# 3. Solution

- naive approach.
```cpp
class Solution {

public:

    ListNode* rotateRight(ListNode* head, int k) {
        if(k == 0 || !head) return head;
        ListNode* temp = head;
        vector<int> vec;
        while(temp) {
            vec.push_back(temp->val);
            temp = temp->next;
        }

        int n = vec.size();
        k = k % n;
        if(k == 0) return head;


        reverse(vec.end()-k, vec.end());
        reverse(vec.begin(), vec.end()-k);
        reverse(vec.begin(), vec.end());

        ListNode* res = new ListNode(vec[0]);
        temp = res;
        for(int i = 1; i < vec.size(); i++) {
            temp->next = new ListNode(vec[i]);
            temp = temp->next;
        }
  
        return res;
    }
};
```

- In place approach
```cpp
class Solution {
public:
    ListNode* rotateRight(ListNode* head, int k) {
        if(k == 0 || !head) return head;
        
        int n = 0;   
        ListNode* temp = head;
        ListNode* last = nullptr;
        while(temp) {
            if(!temp->next) last = temp; // remember last node.
            temp = temp->next;
            n++;
        }

        k %= n;
        if(k==0) return head; 

        temp = head;
        ListNode* pivot;
        ListNode* prev = nullptr;
        while(temp) {
            n--;
            if(n+1 == k) { // if we find a rotation target.
                pivot = temp; // make current node as head
                prev->next = nullptr; // make previous node as tail
                last->next = head; // make last node point to previous head
                break;
            }

            prev = temp;
            temp = temp->next;
        }

        return pivot;
    }
};
```

