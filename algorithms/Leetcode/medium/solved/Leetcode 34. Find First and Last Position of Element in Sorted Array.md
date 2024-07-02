
# 1. Problem
Given an array of integers `nums` sorted in non-decreasing order, find the starting and ending position of a given `target` value.

If `target` is not found in the array, return `[-1, -1]`.

You must write an algorithm with `O(log n)` runtime complexity.

**Example 1:**

**Input:** nums = [5,7,7,8,8,10], target = 8
**Output:** [3,4]

**Example 2:**

**Input:** nums = [5,7,7,8,8,10], target = 6
**Output:** [-1,-1]

**Example 3:**

**Input:** nums = [], target = 0
**Output:** [-1,-1]

**Constraints:**

- `0 <= nums.length <= 105`
- `-109 <= nums[i] <= 109`
- `nums` is a non-decreasing array.
- `-109 <= target <= 109`


# Intuition

- Binary search seems appropriate algorithm to solve this problem in O(logn) time complexity
- lower_bound, upper_bound function in aglorithm STL seems to be useful
	- lower_bound : In given range, find an element which is greater or equal to given value.  If it doesn't exist, return end iterator
	- upper_bound : In given range, find an element which is greater that given value. If not exists, return end iterator

# Solution

1. With conditional statements.
	- Do lower_bound search first
	- If the result is not end and the iterator points to a value that we want to find, Then Do upper_bound search
	- If there is a bigger value than target, we simply update end position as upperInd - 1
	- If upper iterator is equalt to end, then we need to check the last element in 'nums' vector is equal to target. -> Update appropriately
```cpp
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        vector<int> res = {-1,-1};
        
        auto lower = lower_bound(nums.begin(), nums.end(), target);

        int lowerInd = -1, upperInd = -1;
        if(lower != nums.end()) {
            lowerInd = distance(nums.begin(), lower);
            if(nums[lowerInd] == target) {
                res[0] = lowerInd;
                
                auto upper = upper_bound(nums.begin(), nums.end(), target);
                if(upper != nums.end()) {
                    upperInd = distance(nums.begin(), upper)-1;
                    if(nums[upperInd] != target) upperInd = lowerInd;
                    res[1] = upperInd;
                }
                else {
                    if(nums.back() == target) res[1] = nums.size()-1;
                    else res[1] = lowerInd;
                }
            }
        }


        return res;
    }
};
```
![](../../../../../images/Pasted%20image%2020240108120417.png)

2. Optimized version
	We can write much more concise version of same logic.
```cpp
class Solution {

public:

    vector<int> searchRange(vector<int>& nums, int target) {

        int lb = (lower_bound(nums.begin() , nums.end() , target)-begin(nums));

        int ub = (upper_bound(nums.begin() , nums.end() , target)-begin(nums));

        if(ub == lb) return {-1,-1};

        return {lb,ub-1};

    }

};
```
![](../../../../../images/Pasted%20image%2020240108120410.png)

3. Further optimize
	If we remove unneccessary upper_bound search when the found result of lower bound is different with target, we can optimize search time further.
```cpp
class Solution {

public:

    vector<int> searchRange(vector<int>& nums, int target) {
        vector<int> res = {-1,-1};
        auto lower = lower_bound(nums.begin(), nums.end(), target);
        if(lower != nums.end() && nums[lower-nums.begin()] == target) {
            res[0] = lower-nums.begin();
        }
        else return res;

        auto upper = upper_bound(nums.begin(), nums.end(), target);
        if(upper != nums.end()) res[1] = upper - nums.begin()-1;
        else {
            if(nums.back() == target) res[1] = nums.size()-1;
            else res[1] = lower-nums.begin();
        }

        return res;

    }

};
```
![](../../../../../images/Pasted%20image%2020240108120357.png)