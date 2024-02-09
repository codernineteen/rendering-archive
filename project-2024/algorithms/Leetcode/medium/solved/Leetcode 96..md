
- #solved  #medium #dynammic-programming 
- took 40min to solve

# 1. Problem description

Given an integer `n`, return _the number of structurally unique **BST'**s (binary search trees) which has exactly_ `n` _nodes of unique values from_ `1` _to_ `n`.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/01/18/uniquebstn3.jpg)

**Input:** n = 3
**Output:** 5

**Example 2:**

**Input:** n = 1
**Output:** 1

**Constraints:**

- `1 <= n <= 19`

# 2. Intuition

The given example in problem description is big hint.

Let's expand the example as the case when the number of nodes is equal to four.
We don't need to make all the possible trees ourselves and that is not the purpose we utilize algorithm. 
Keeping in mind the tree structure following 'Binary search tree' format, we can abstract the generations of sub trees under the root if we ensure that the subtrees don't violate rule of BST.

Here is the example about what i mentioned.

If we position node '4' as right-most root, the possible subtrees will remain left side of the root.
![](../../../../../images/Pasted%20image%2020240209203825.png)

If we position node '4' as intermediate in whole tree, the possible trees will be equal to (left-side cases x right-side cases)
![](../../../../../images/Pasted%20image%2020240209204115.png)

# 3. Solution

- Time complexity : $O(N^2)$
- space complexity : $O(N)$
```cpp
class Solution {
    int res = 0;
public:
    int numTrees(int n) {
        vector<int> dp(n+1, 0);
        dp[1] = 1;
        
        for(int i = 2; i < n+1; i++) {
            for(int pos = 1; pos <= i; pos++) {
                int numLeft = pos - 1;
                int numRight = i - pos;
                if(numLeft == 0) dp[i] += dp[numRight];
                else if(numRight == 0) dp[i] += dp[numLeft];
                else dp[i] += (dp[numLeft] * dp[numRight]);
            }
        }
        
        return dp[n];
    }
};
```