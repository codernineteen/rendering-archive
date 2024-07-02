
- #solved #easy #search
- Took 10min to solve

# 1. Problem
Given a sorted array of distinct integers and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order.

You must write an algorithm with `O(log n)` runtime complexity.

**Example 1:**

**Input:** nums = [1,3,5,6], target = 5
**Output:** 2

**Example 2:**

**Input:** nums = [1,3,5,6], target = 2
**Output:** 1

**Example 3:**

**Input:** nums = [1,3,5,6], target = 7
**Output:** 4

**Constraints:**

- `1 <= nums.length <= 104`
- `-104 <= nums[i] <= 104`
- `nums` contains **distinct** values sorted in **ascending** order.

# 2. Idea

As the description said, we need to solve this problem in O(log n) time complexity.
Because the given array is sorted in ascending order with distinct integers, we can use binary search here.

Instead of implementing bin search our own, let's use upper_bound function from algorithms STL.
- If the result of upper bound is end of nums, return size of nums vector which is equal to next position against last element in the vector.
- If upper bound returns an iterator in vector, we just return its index by subtracting it with nums.begin()

# 3. Solution
```cpp
class Solution {

public:

    int searchInsert(vector<int>& nums, int target) {

        auto found = lower_bound(nums.begin(), nums.end(), target);

        if(found == nums.end()) return nums.size();

        else {

            return found - nums.begin();

        }

    }

};
```