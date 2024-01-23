
- #Meidum #failed
- 2024/1/5

# Description
There is an integer array `nums` sorted in ascending order (with **distinct** values).

Prior to being passed to your function, `nums` is **possibly rotated** at an unknown pivot index `k` (`1 <= k < nums.length`) such that the resulting array is `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]` (**0-indexed**). For example, `[0,1,2,4,5,6,7]` might be rotated at pivot index `3` and become `[4,5,6,7,0,1,2]`.

Given the array `nums` **after** the possible rotation and an integer `target`, return _the index of_ `target` _if it is in_ `nums`_, or_ `-1` _if it is not in_ `nums`.

You must write an algorithm with `O(log n)` runtime complexity.

**Example 1:**

**Input:** nums = [4,5,6,7,0,1,2], target = 0
**Output:** 4

**Example 2:**

**Input:** nums = [4,5,6,7,0,1,2], target = 3
**Output:** -1

# Solution
One of half arrays based on middle position will be always sorted.
According to correlation between (mid, high, low) and target, we can apply modified binary search here using more conditions

```cpp
class Solution {

public:

  

    int search(vector<int>& nums, int target) {

        int low = 0, high = nums.size()-1;

        while(low <= high) {

            int mid = (low+high)/2;

  

            if(nums[mid] == target) {

                return mid;

            }

  

            if(nums[mid] >= nums[low]) {

                if(nums[mid] < target || nums[low] > target ) {

                    low = mid + 1;

                }

                else {

                    high = mid - 1;

                }

            }

            else {

                if(nums[mid] > target || nums[high] < target) {

                    high = mid - 1;

                }

                else {

                    low = mid + 1;

                }

            }

        }

  

        return -1;

    }

};
```