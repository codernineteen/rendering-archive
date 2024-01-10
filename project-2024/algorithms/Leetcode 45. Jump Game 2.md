
- #failed #medium #BFS 
- 1h 6min try

# 1. Problem
You are given a **0-indexed** array of integers `nums` of length `n`. You are initially positioned at `nums[0]`.

Each element `nums[i]` represents the maximum length of a forward jump from index `i`. In other words, if you are at `nums[i]`, you can jump to any `nums[i + j]` where:

- `0 <= j <= nums[i]` and
- `i + j < n`

Return _the minimum number of jumps to reach_ `nums[n - 1]`. The test cases are generated such that you can reach `nums[n - 1]`.

**Example 1:**

**Input:** nums = [2,3,1,1,4]
**Output:** 2
**Explanation:** The minimum number of jumps to reach the last index is 2. Jump 1 step from index 0 to 1, then 3 steps to the last index.

**Example 2:**

**Input:** nums = [2,3,0,1,4]
**Output:** 2

**Constraints:**

- `1 <= nums.length <= 104`
- `0 <= nums[i] <= 1000`
- It's guaranteed that you can reach `nums[n - 1]`

# 2. Solution
- Based on current range from start to end, Find next farthest end and update 'end' to the farthest end and 'start' to 'end+1'

```cpp
class Solution {

public:

    int jump(vector<int>& nums) {

        int n = nums.size(), step = 0, start = 0, end = 0;

        while(end < n-1) {

            step++;

            int maxend = end+1;

            for(int i = start; i <= end; i++) {

                if(i+nums[i] >= n-1) return step;

                maxend = max(maxend, i+nums[i]);

            }

            start=end+1;

            end=maxend;

        }

  

        return step;

    }

};
```