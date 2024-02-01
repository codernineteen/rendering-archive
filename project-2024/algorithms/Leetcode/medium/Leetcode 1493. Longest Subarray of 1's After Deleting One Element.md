
- #solved #medium #subsequence
- took 10min to solve

# 1. Problem description

Given a binary array `nums`, you should delete one element from it.

Return _the size of the longest non-empty subarray containing only_ `1`_'s in the resulting array_. Return `0` if there is no such subarray.

**Example 1:**

**Input:** nums = [1,1,0,1]
**Output:** 3
**Explanation:** After deleting the number in position 2, [1,1,1] contains 3 numbers with value of 1's.

**Example 2:**

**Input:** nums = [0,1,1,1,0,1,1,0,1]
**Output:** 5
**Explanation:** After deleting the number in position 4, [0,1,1,1,1,1,0,1] longest subarray with value of 1's is [1,1,1,1,1].

**Example 3:**

**Input:** nums = [1,1,1]
**Output:** 2
**Explanation:** You must delete one element.

**Constraints:**

- `1 <= nums.length <= 105`
- `nums[i]` is either `0` or `1`.

# 2. Intuition

I started with very intuitive approach.
The longest subsequence can be made when the two subsequences of 1 are placed around a zero-valued element.

To compute the subsequences easier, i compressed the subsequences of 1 first.
Then, i computed the sum of the length of adjacent subsequences after removing zero elemnt in-between them.
![](../../../../images/Pasted%20image%2020240201121933.png)

But this approach requires two times traversal , which leads to higher time complexity boundary agains optimized version.

If we add proper conditions in the compression step, we can solve this problem in a traversal.
- Update 'now' if current element is 1
- If current element has 0 value,
	- If next element exist and its value is 1
		- If we have pre computed subsequence , update result using 'max' stl
		update pre-computed as 'now' and initialize 'now' as 0 again.
	- else
		update result using 'max' stl and initialize both of 'prev' and 'now' as 0

# 3. Solution

- Intuitive approach (beats 8.31 %)
	- runtime : O(n) with two traversals
	- memory : O(n)
```cpp
class Solution {
public:
    int longestSubarray(vector<int>& nums) {
        vector<int> compressed;
        int count = 0;
        for(int i=0; i < nums.size(); i++) {
            if(nums[i] == 1) count++;
            else {
                if(count > 0) compressed.push_back(count);
                count = 0;
                compressed.push_back(count);
            }

            if(i == nums.size()-1 && count > 0) compressed.push_back(count); 
        }
        for(auto n : compressed) cout << n << " ";

        count = 0;
        if(compressed.size() == 1 && compressed[0] > 0) return compressed[0]-1;
        else {
            for(int i = 0; i < compressed.size(); i++) {
                if(compressed[i] != 0) {
                    if(i+2 < compressed.size()) {
                        count = max(count, compressed[i]+compressed[i+2]);
                    }
                    else count = max(count, compressed[i]);
                }
            }

            return count;
        }
    }
};
```

- Optimization (beats 99% )
	- runtime : O(n) with one traversal
	- memory : in-place , O(1)
```cpp
class Solution {
public:
    int longestSubarray(vector<int>& nums) {
        int prev = 0;
        int now = 0;
        int res = 0;
        int n = nums.size();

        bool zeroExist = false;

        for(int i=0; i < n; i++) {
            if(nums[i] == 1) now++;
            else {
                if(nums[i] == 0 && i+1 < n && nums[i+1]==1) {
                    if(prev > 0) res = max(res, prev+now);
                    prev = now;
                    now = 0;
                }
                else {
                    res = max(res, prev+now);
                    prev = now = 0;
                }
                zeroExist = true;
            }
        }

        if(prev != 0 || now != 0) res = max(res, prev+now);

        return zeroExist ? res : res-1;
    }
};
```