
- #solved #medium #sliding-window 
- took 10min to solve

# 1. Problem description
Given an array of positive integers `nums` and a positive integer `target`, return _the **minimal length** of a_ 

_subarray_

 _whose sum is greater than or equal to_ `target`. If there is no such subarray, return `0` instead.

**Example 1:**

**Input:** target = 7, nums = [2,3,1,2,4,3]
**Output:** 2
**Explanation:** The subarray [4,3] has the minimal length under the problem constraint.

**Example 2:**

**Input:** target = 4, nums = [1,4,4]
**Output:** 1

**Example 3:**

**Input:** target = 11, nums = [1,1,1,1,1,1,1,1]
**Output:** 0

**Constraints:**

- `1 <= target <= 109`
- `1 <= nums.length <= 105`
- `1 <= nums[i] <= 104`

**Follow up:** If you have figured out the `O(n)` solution, try coding another solution of which the time complexity is `O(n log(n))`.
# 2. Intuition
We can solve this using sliding window algorithm by following two core principles.
1. If current sum is less than target, expand window by incrementing right pointer.
2. If current sum is greater or equal to target, shrink window until the sum is less than target again by incrementing left pointer.

# 3. Solution
```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int ret = INT_MAX;

        int left=0, right=0;
        int n = nums.size();
        int sum = nums[0];

        while(left <= right && right < n)
        {
            if(sum < target) {
                right++;
                if(right < n) sum += nums[right];
            }
            else {
                ret = min(ret, right-left+1);
                sum -= nums[left];
                left++;
            }
        }

        return ret == INT_MAX ? 0 : ret;   
    }
};
```
