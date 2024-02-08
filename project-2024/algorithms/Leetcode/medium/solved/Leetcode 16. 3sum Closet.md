
# 1. Problem description
Given an integer array `nums` of length `n` and an integer `target`, find three integers in `nums` such that the sum is closest to `target`.

Return _the sum of the three integers_.

You may assume that each input would have exactly one solution.

**Example 1:**

**Input:** nums = [-1,2,1,-4], target = 1
**Output:** 2
**Explanation:** The sum that is closest to the target is 2. (-1 + 2 + 1 = 2).

**Example 2:**

**Input:** nums = [0,0,0], target = 1
**Output:** 0
**Explanation:** The sum that is closest to the target is 0. (0 + 0 + 0 = 0).

**Constraints:**

- `3 <= nums.length <= 500`
- `-1000 <= nums[i] <= 1000`
- `-104 <= target <= 104`

# 2. Intuition

- Brute force.
In the worst case, The total amount of steps required in brutes force approach is 125,000,000. 
It is corresponded to about 1 second , so i thought it is enough to implement brute force approach first.
My idea is that we can think this problem as finding shortest gap between current sum and target and no gap is the best case that we can have.

# 3. Solution

- brute force

Time complexity = O(n^3)

```cpp
class Solution {

public:

    int threeSumClosest(vector<int>& nums, int target) {
        long int n = nums.size();
        long int res = INT_MAX;

        for(int i = 0; i < n; i++) {
            if(res == target) return target;

            long int first = nums[i];
            for(int j = i+1; j < n; j++) {
                long int second = nums[j];
                for(int k = j+1; k < n; k++) {
                    long int third = nums[k];
                    long int sum = first + second + third;
                    if(abs(target-sum) < abs(target - res)) res = sum;
                }
            }
        }

        return res;
    }
};
```


- sort optimization

Time complexity = O(n log n + n^2) =  O(n^2)

If we sort an array in increasing order, we can solve this problem with two for-loops.

- Outer for loop - initialize first number and third number for current inner iteration.
- Based on whether current sum is smaller than target or bigger than target, we determine how we will move third and second.
	- If current sum is larger than target, we need to decrement third.
	- In opposite case, we need to increment second.

During above iterations, we record the best sum whose distance from target is the shortest. (0 distance means that we found the target sum in the array)
```cpp
class Solution {
public:
    int threeSumClosest(vector<int>& nums, int target) {
        sort(nums.begin(), nums.end());
        int res = nums[0] + nums[1] + nums[2];
        
        for(int first = 0; first < nums.size()-2; first++) {
            if(first > 0 && nums[first] == nums[first-1]) continue;
            int second = first + 1;
            int third = nums.size()-1;
            while(second < third) {
                int cur = nums[first] + nums[second] + nums[third];
                if(cur == target) return target;
                if(abs(target-cur) < abs(target-res)) res = cur;
                
                if(cur > target) third--;
                else second++;
            }
        }

        return res;

    }
};
```