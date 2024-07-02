
- #failed #medium #dynammic-programming 
- tried 1h

# 1. Problem description

Given an `m x n` binary `matrix` filled with `0`'s and `1`'s, _find the largest square containing only_ `1`'s _and return its area_.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/11/26/max1grid.jpg)

**Input:** matrix = [["1","0","1","0","0"],["1","0","1","1","1"],["1","1","1","1","1"],["1","0","0","1","0"]]
**Output:** 4

**Example 2:**

![](https://assets.leetcode.com/uploads/2020/11/26/max2grid.jpg)

**Input:** matrix = [["0","1"],["1","0"]]
**Output:** 1

**Example 3:**

**Input:** matrix = [["0"]]
**Output:** 0

**Constraints:**

- `m == matrix.length`
- `n == matrix[i].length`
- `1 <= m, n <= 300`
- `matrix[i][j]` is `'0'` or `'1'`.


# 2. Solution

By traversing from the top-left to bottom-right,
we can follow this recurrence :
$$length(i, j)=min(\space length(i-1,j),\space length(i, j-1),\space length(i-1,j-1)\space) + 1$$, only when element at (i, j) is 1

The recurrence equation means that the length of current position(i, j) will be follow the minimum stacked length from the three directions.

```cpp
class Solution {

public:
    int maximalSquare(vector<vector<char>>& matrix) {
        if(matrix.empty())  return 0;
        int m = matrix.size(), n = matrix[0].size();

        vector<vector<int>> dp(m, vector<int>(n, 0));
        int side = 0;
        for(int i=0; i<m;i++) {
            for(int j=0; j<n;j++) {
                if(!i || !j || matrix[i][j] == '0') {
                    dp[i][j] = matrix[i][j] - '0';
                }
                else {
                    dp[i][j] = min(dp[i-1][j-1], min(dp[i-1][j], dp[i][j-1])) + 1;
                    side = max(dp[i][j], side);
               }
            }
        }

        return side*side;
    }
};
```


# reference
- solutions
	- https://leetcode.com/problems/maximal-square/solutions/600149/python-thinking-process-diagrams-dp-approach/?envType=study-plan-v2&envId=top-interview-150
	- https://leetcode.com/problems/maximal-square/solutions/61803/c-space-optimized-dp/?envType=study-plan-v2&envId=top-interview-150