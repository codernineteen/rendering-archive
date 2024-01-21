
- #solved #easy #binary-tree
- took 24min to solve
# 1. Problem description
Given the `root` of a binary tree, _check whether it is a mirror of itself_ (i.e., symmetric around its center).

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/02/19/symtree1.jpg)

**Input:** root = [1,2,2,3,4,4,3]
**Output:** true

**Example 2:**

![](https://assets.leetcode.com/uploads/2021/02/19/symtree2.jpg)

**Input:** root = [1,2,2,null,3,null,3]
**Output:** false

**Constraints:**

- The number of nodes in the tree is in the range `[1, 1000]`.
- `-100 <= Node.val <= 100`

# 2. Intuition

While level order traversal on the given tree, keep tracking of nodes in the current level by storing it into temporary vector.
When the number of nodes in the vector is equal to $2^{level}$ , we start to inspect whether current level forms symmetric structure or not. 

# 3. Solution
- iterative solution
```cpp
/**
 * Definition for a binary tree node
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */

class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        queue<TreeNode*> nexts;
        nexts.push(root->left);
        nexts.push(root->right);

        int level = 1;
        int numOfNodes = pow(2, level);
        vector<int> temp;

        while(!nexts.empty()) {
            if(temp.size() == numOfNodes) {
                numOfNodes = pow(2, ++level);
  
                bool flag = false;
                if(temp.size() % 2 != 0) return false;
                for(int i = 0, j = temp.size()-1; i < temp.size()/2; i++, j--) {
                    if(temp[i] != temp[j]) return false;
                    if(temp[i] != -200) flag = true;
                }
                if(!flag) break;

                temp.clear();
            }
            else {
                auto t = nexts.front();
                nexts.pop();

                if(t) {
                    temp.push_back(t->val);
                    nexts.push(t->left);
                    nexts.push(t->right);
                }
                else {
                    temp.push_back(-200);
                    nexts.push(nullptr);
                    nexts.push(nullptr);
                }
            }
        }

        return true;
    }

};
```

My first iterative approach is quite inefficient because it takes additional overhead to inspect whether a level is symmetric or not by comparing all the nodes in the level every time.

Belows are the recursive solution that is more efficient from leetcode.
- recursive approach
Based on current left and right node, we compare for each of 
- left's right child VS right's left child (inner nodes)
- left's left child VS right's right child (outer nodes)
- ![](../../../images/Pasted%20image%2020240121212121.png)
```cpp
class Solution {
public:
    bool test(TreeNode* left, TreeNode* right) {
        if(!left && !right) return true;
        else if(!left || !right) return false;
        else if (left->val != right->val) return false;

        return test(left->right, right->left) && test(left->left, right->right);
    }

    bool isSymmetric(TreeNode* root) {
        if(!root) return true;
        return test(root->left, root->right);
    }
};
```

