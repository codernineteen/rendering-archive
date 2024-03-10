
- #failed #medium #binary-search 
- Tried 1h 13min

# 1. Problem description

A peak element is an element that is strictly greater than its neighbors.

Given a **0-indexed** integer array `nums`, find a peak element, and return its index. If the array contains multiple peaks, return the index to **any of the peaks**.

You may imagine that `nums[-1] = nums[n] = -∞`. In other words, an element is always considered to be strictly greater than a neighbor that is outside the array.

You must write an algorithm that runs in `O(log n)` time.

**Example 1:**

**Input:** nums = [1,2,3,1]
**Output:** 2
**Explanation:** 3 is a peak element and your function should return the index number 2.

**Example 2:**

**Input:** nums = [1,2,1,3,5,6,4]
**Output:** 5
**Explanation:** Your function can return either index number 1 where the peak element is 2, or index number 5 where the peak element is 6.

**Constraints:**

- `1 <= nums.length <= 1000`
- `-231 <= nums[i] <= 231 - 1`
- `nums[i] != nums[i + 1]` for all valid `i`.

# 2. Intuition
There are three crucial conditions to solve this problem.
1. return the index to **any of the peaks**.
2. `nums[-1] = nums[n] = -infinity`
3. `nums[i] != nums[i+1]` for all valid i.

Definitely there can be multiple peaks, but we still can focus on just a peak in the array.
In other words, we can abstract this problem as a problem to find a single peak in the array for simplicity.

Basically, there are three possible visualization.
1. montonically increasing slope
2. intermediat peak
3. montonically decreasing slope
![](../../../../../images/Pasted%20image%2020240307131309.png)

We can detect whether it is a peak or not for case 1, 3 with conditional statements right away.
So we just need to implement an algorithm to find a peak the case when a peak is placed at middle of the array.

This was the exact point that i failed to implement because i had no idea to update index of middle in a algorithmatically perfect way.

By referring to a solution, i could get a good intuition about the solution.
With given low and high indices, we can determine the middle index.
If we are lucky enough, then the middle index will be exact peak position.
If not, we have two different cases.
- the place where slope is montonically decreasing
- the place where slop is montonically increasing.
For the decreasing case, we need to update high index as middle.
For the increasing case, we need to update low index as middle.
As a result, thanks to nature of binary search , our middle position will find a proper peak position by moving the slope zig-zag way.
![](../../../../../images/Pasted%20image%2020240307131859.png)

# 3. Solution
- time complexity : $O(\log{n})$
- space complexity : $O(1)$
```cpp
class Solution {
public:
    int binSearch(vector<int>& nums, int low, int high)
    {
        int mid = low+(high-low)/2;

        while(low < high)
        {
            mid = low+(high-low)/2;
            if(nums[mid-1] < nums[mid] && nums[mid] > nums[mid+1]) return mid;
            else if(nums[mid-1] > nums[mid]) high = mid-1;
            else if(nums[mid+1] > nums[mid]) low = mid+1;
        }

        return low;
    }

    int findPeakElement(vector<int>& nums) {
        int n = nums.size();
        if(n == 1) return 0;
        if(nums[1] < nums[0]) return 0;
        if(nums[n-2] < nums[n-1]) return n-1;

        return binSearch(nums, 1, n-2);
    }
};
```