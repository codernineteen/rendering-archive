
- #solved #easy #hashmap 
- took 10min to solve

# 1. Problem description
Given an integer array `nums` and an integer `k`, return `true` _if there are two **distinct indices**_ `i` _and_ `j` _in the array such that_ `nums[i] == nums[j]` _and_ `abs(i - j) <= k`.

**Example 1:**

**Input:** nums = [1,2,3,1], k = 3
**Output:** true

**Example 2:**

**Input:** nums = [1,0,1,1], k = 1
**Output:** true

**Example 3:**

**Input:** nums = [1,2,3,1,2,3], k = 2
**Output:** false

**Constraints:**

- `1 <= nums.length <= 105`
- `-109 <= nums[i] <= 109`
- `0 <= k <= 105`

# 2. Intuition

Under constraints, it is possible to use brute force approach whose time complexity is $O(n^2)$.

Then can we use quick sort to improve time complexity up to $O(n \log(n))$?
The answer is still 'No'.
If the problem were to find exsistence of duplicate, it would be enough to use sort approach. However, We need to satisfy another condition 'abs(i-j) <= k'. 
That's why we can't change the original order of given array.

After then, i checked whether i can solve this problem in $O(n)$.

My idead is :
- If there is no key of current number in hash map, we create new pair about the key with current index as its value.
- If there is already a key inserted before, we can check second condition.
	- If the value of exist key satisfy '$abs(i-value) \le k$', we can return true right away.
	- If not, we need to update the value of key as current index for later possible comparisions.

After tested this idea with a few of cases, i got to know this is a good solution to solve this problem.

# 3. Solution

```cpp
class Solution {
public:
    bool containsNearbyDuplicate(vector<int>& nums, int k) {
        unordered_map<int, int> um;

        for(int i = 0; i < nums.size(); i++)
        {
            if(um.find(nums[i]) != um.end()) {
                if(abs(um[nums[i]]-i) <= k) return true;
                else um[nums[i]] = i;
            }
            else um[nums[i]] = i;
        }

        return false;
    }
};
```