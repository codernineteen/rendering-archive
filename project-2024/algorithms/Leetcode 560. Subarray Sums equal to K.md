
- #failed #medium #hashmap #complement
- tried 1h
# 1. Problem
Given an array of integers `nums` and an integer `k`, return _the total number of subarrays whose sum equals to_ `k`.

A subarray is a contiguous **non-empty** sequence of elements within an array.

**Example 1:**

**Input:** nums = [1,1,1], k = 2
**Output:** 2

**Example 2:**

**Input:** nums = [1,2,3], k = 3
**Output:** 2

**Constraints:**

- `1 <= nums.length <= 2 * 104`
- `-1000 <= nums[i] <= 1000`
- `-107 <= k <= 107`

# 2. Idea
My naive idea was using dynamic programming to reduce time for computation of sums.
The equation is :
$dp[i]=dp[i-1]+nums[i+range-1]$
where, 'i' is current index and 'range' is current range of certain sum.

This can reduce the time for sum computation as i expected, but it has still $O(n^2)$ time complexity.

By solution, we can utilize hash map to solve this problem just in time.

Our goal is to find a subarray whose sum is equal to 'k'. 
But because the subarray can exist in the middle of vector, it is not easy to determine where the subarrays exists in a vector.
Instead of targeting to find a k, we can change our perspective to find a subarray whose sum is equal to 'sum-k' when current position has 'sum'.

In this way, it is much more effective because the 'sum-k' always starts from 0-index.
Also, because current sum can be next 'sum' - k', we need to increment a value of a key whose value is equal to sum.
![](../../images/Pasted%20image%2020240112141014.png)


# 3. Solution
```cpp
class Solution {

public:

    int subarraySum(vector<int>& nums, int k) {

        unordered_map<int, int> um;

        int sum = 0, ans = 0;

        um[sum] = 1; // initial sum is equal to zero

  

        for(auto num : nums) {

            sum += num;

            if(um.find(sum-k) != um.end()) ans += um[sum-k];

            um[sum]++; // possibly can be target later -> increment

        }

        return ans;

    }

};
```