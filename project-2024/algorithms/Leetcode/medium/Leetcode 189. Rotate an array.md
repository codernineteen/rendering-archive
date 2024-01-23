
- #solved #medium #array
- took 13min to solve
# 1. Problem description
Given an integer array `nums`, rotate the array to the right by `k` steps, where `k` is non-negative.

**Example 1:**

**Input:** nums = [1,2,3,4,5,6,7], k = 3
**Output:** [5,6,7,1,2,3,4]
**Explanation:**
rotate 1 steps to the right: [7,1,2,3,4,5,6]
rotate 2 steps to the right: [6,7,1,2,3,4,5]
rotate 3 steps to the right: [5,6,7,1,2,3,4]

**Example 2:**

**Input:** nums = [-1,-100,3,99], k = 2
**Output:** [3,99,-1,-100]
**Explanation:** 
rotate 1 steps to the right: [99,-1,-100,3]
rotate 2 steps to the right: [3,99,-1,-100]

**Constraints:**

- `1 <= nums.length <= 105`
- `-231 <= nums[i] <= 231 - 1`
- `0 <= k <= 105`
# 2. Intutition
With a very naive approach, we can separate an array in two parts.
The one is a set whose elements move left under boundary of index and the other is a set whose is pushed front of the array because they are over the boundary of the array.

This image depicts the case :

![](../../../../images/Pasted%20image%2020240120170624.png)

My algorithm follows below steps :
1. modular operation agains k because we can't remove unneccessary rotation whose result is exactly same with original array.
2. Store elements that is over index after rotation inside cache.
3. Move inside elements left as much as k
4. Overwrite the value of indices from index-0 until there is no value in cache.

# 3. Solution
- Time complexity : O(n)
- Space complexity : O(n)
```cpp
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        int n = nums.size();
        k %= n;

        int it = 0;
        vector<int> cache(k, 0);
        for(int i = n-k; i < n; i++) cache[it++] = nums[i];
        for(int i = n-k-1; i >= 0; i--) {
            nums[i+k] = nums[i];
        }
        
        it = 0;
        for(auto val : cache) nums[it++] = val;
    }
};
```

- Instead of using extra space, we can solve this problem in place by utilizing 'reverse' algorithm
```cpp
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        int n = nums.size();
        k %= n;
// ex ) 1 2 3 4 5 6 7 

        reverse(nums.begin()+(n-k), nums.end()); // -> 1 2 3 4 7 6 5
        reverse(nums.begin(), nums.begin()+(n-k)); // -> 4 3 2 1 7 6 5
        reverse(nums.begin(), nums.end()); // 5 6 7 1 2 3 4
    }

};
```
