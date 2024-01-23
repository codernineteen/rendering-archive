
- #solved #medium #brute-force #back-tracking
- took 14 min to solve


# 1. Problem description
Given an integer array `nums` of **unique** elements, return _all possible_ 

_subsets_

 _(the power set)_.

The solution set **must not** contain duplicate subsets. Return the solution in **any order**.

**Example 1:**

**Input:** nums = [1,2,3]
**Output:** [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]

**Example 2:**

**Input:** nums = [0]
**Output:** [[],[0]]

**Constraints:**

- `1 <= nums.length <= 10`
- `-10 <= nums[i] <= 10`
- All the numbers of `nums` are **unique**.

# 2. Solution
During recursion, decrement number of elements in a subset of powerset and if number of rest of elements reaches zero, add it to result.

```cpp
class Solution {
public:
    void solve(vector<vector<int>>& res, vector<int>& nums, vector<int>& container, int rest, int curIdx) {
        if(curIdx >= nums.size() && rest > 0) return;
        if(rest == 0) {
            res.push_back(container);
            return;
        }

        for(int i = curIdx; i < nums.size(); i++) {
            container.push_back(nums[i]);
            rest--;
            solve(res, nums, container, rest, i+1);
            rest++;
            container.pop_back();
        }
    }

  

    vector<vector<int>> subsets(vector<int>& nums) {
        vector<vector<int>> res;
        vector<int> temp;
        for(int i = 0; i <= nums.size(); i++) {
            solve(res, nums, temp, i, 0);
        }
  
        return res;
    }
};
```