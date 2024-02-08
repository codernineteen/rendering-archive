
- #solved #medium #array #dynammic-programming
- took 5min to solve
# 1. Problem description

Given an integer array `nums`, find the subarray with the largest sum, and return _its sum_.

**Example 1:**

**Input:** nums = [-2,1,-3,4,-1,2,1,-5,4]
**Output:** 6
**Explanation:** The subarray [4,-1,2,1] has the largest sum 6.

**Example 2:**

**Input:** nums = [1]
**Output:** 1
**Explanation:** The subarray [1] has the largest sum 1.

**Example 3:**

**Input:** nums = [5,4,-1,7,8]
**Output:** 23
**Explanation:** The subarray [5,4,-1,7,8] has the largest sum 23.

**Constraints:**

- `1 <= nums.length <= 105`
- `-104 <= nums[i] <= 104`

# 2. Intuition
It will be easier to explain with a given example `[-2,1,-3,4,-1,2,1,-5,4]`
Let's start from the beginning, which is -2 and it is our current sum.

After then, we need to determine two things for every steps.
The one determination is :
- Is current number larger than newly computed current sum ? (ex, 1 > -2 + 1)
	- If so, we initialize current sum as current number again
	- if not, keep adding number to current sum
The other determination is :
- Is current sum larger than previous max sum ?
	- If so, update max sum as current sum

![](../../../../../images/Pasted%20image%2020240130135026.png)
# 3. Solution

- Time complexity : O(n)
```cpp
class Solution {

public:

    int maxSubArray(vector<int>& nums) {
        int curSum = nums[0];
        int maxSum = nums[0];

        for(int i = 1;  i < nums.size(); i++) {
            curSum += nums[i];
            if(nums[i] > curSum) curSum = nums[i];
            maxSum = max(curSum, maxSum);
        }

        return maxSum;
    }
};
```