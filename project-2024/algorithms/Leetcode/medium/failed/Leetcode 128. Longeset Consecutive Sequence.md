
#failed #medium #array #hashmap 

# 1. Problem
Given an unsorted array of integers `nums`, return _the length of the longest consecutive elements sequence._

You must write an algorithm that runs in `O(n)` time.

**Example 1:**

**Input:** nums = [100,4,200,1,3,2]
**Output:** 4
**Explanation:** The longest consecutive elements sequence is `[1, 2, 3, 4]`. Therefore its length is 4.

**Example 2:**

**Input:** nums = [0,3,7,2,5,8,4,6,0,1]
**Output:** 9

**Constraints:**

- `0 <= nums.length <= 105`
- `-109 <= nums[i] <= 109`

# 2. Intuition

- Convert a vector into a set to search consecutive elements in O(1) time.
- Assuming that current number is  'num', if there is no 'num-1' in the set, we start to find the consecutive sequence based on this pivot 'num'.
- After finding, Store the length of sequence if it longer than current max.

# 3. Solution

```cpp
class Solution {
    int res = 1;
public:
    int longestConsecutive(vector<int>& nums) {
        set<int> numSet(nums.begin(), nums.end());
        for(auto num : nums) {
            if(numSet.find(num) != numSet.end()) {
                int next = num+1;
                while(numSet.find(next) != numSet.end()) next++;
                res = max(res, next-num);
            }
        }

        return res;
    }
};
```