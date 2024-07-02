
- #failed #medium #twopointer

# 1. Problem

Given an integer array nums, return all the triplets `[nums[i], nums[j], nums[k]]` such that `i != j`, `i != k`, and `j != k`, and `nums[i] + nums[j] + nums[k] == 0`.

Notice that the solution set must not contain duplicate triplets.

**Example 1:**

**Input:** nums = [-1,0,1,2,-1,-4]
**Output:** [[-1,-1,2],[-1,0,1]]
**Explanation:** 
nums[0] + nums[1] + nums[2] = (-1) + 0 + 1 = 0.
nums[1] + nums[2] + nums[4] = 0 + 1 + (-1) = 0.
nums[0] + nums[3] + nums[4] = (-1) + 2 + (-1) = 0.
The distinct triplets are [-1,0,1] and [-1,-1,2].
Notice that the order of the output and the order of the triplets does not matter.

# 2. Solution
- sort nums vector first
- Based on current position 'i', create complement of the i-th number in the nums vector.
- Set left(=i+1) and right(=last index of vector)
- If current sum is larger than complement, decrement right
- If current sum is smaller than complement, increment left

One of the most important thing in this algorithm is to remove duplicates from result vector.
To achieve this, When we found a triplet whose sum is equal to zero, we can increment left and decrement right until the next indices have equal value.
Also, before we go to next iteration in for-loop, we can ignore next iterators in the same manner.
```cpp
class Solution {

public:

    vector<vector<int>> threeSum(vector<int>& nums) {

        vector<vector<int>> res;

  

        sort(nums.begin(), nums.end());

  

        for(int i = 0; i < nums.size(); i++) {

            int comp = -nums[i];

            int l = i+1;

            int r = nums.size()-1;

            while(l < r) {

                int sum = nums[l] + nums[r];

  

                if(sum < comp) {

                    l++;

                }

                else if(sum > comp) {

                    r--;

                }

                else {

                    vector<int> triplet = {-comp, nums[l], nums[r]};

                    res.push_back(triplet);

  

                    while(l < r && nums[l] == triplet[1]) l++;

                    while(l < r && nums[r] == triplet[2]) r--;

                }

            }

  

            while(i+1 < nums.size() && nums[i+1] == nums[i]) i++;

        }

  

        return res;

    }

};
```