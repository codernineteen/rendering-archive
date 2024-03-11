- #partially-failed #medium #binary-search 
- took 1h totally 

# 1. Problem description
There is an integer array `nums` sorted in non-decreasing order (not necessarily with **distinct** values).

Before being passed to your function, `nums` is **rotated** at an unknown pivot index `k` (`0 <= k < nums.length`) such that the resulting array is `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]` (**0-indexed**). For example, `[0,1,2,4,4,4,5,6,6,7]` might be rotated at pivot index `5` and become `[4,5,6,6,7,0,1,2,4,4]`.

Given the array `nums` **after** the rotation and an integer `target`, return `true` _if_ `target` _is in_ `nums`_, or_ `false` _if it is not in_ `nums`_._

You must decrease the overall operation steps as much as possible.

**Example 1:**

**Input:** nums = [2,5,6,0,0,1,2], target = 0
**Output:** true

**Example 2:**

**Input:** nums = [2,5,6,0,0,1,2], target = 3
**Output:** false

**Constraints:**

- `1 <= nums.length <= 5000`
- `-104 <= nums[i] <= 104`
- `nums` is guaranteed to be rotated at some pivot.
- `-104 <= target <= 104`

# 2. Intuition

My first intuition was to combine the solution for 'Rotated sorted array 2' and binary search algorihtm together.

- find a rotation pivot first.
- reorder the array in non-decreasing order
- do binary search

With this approach, it shows that the runtime of this solution beated 100% , but it seems that this was not optimal solution.

We can ensure that one of the two half parts of given array is sorted in order when the condition below satisfied.
- `!(nums[mid] == nums[left] == nums[right])`
	Because when the value of left, mid and right are all same with each other, we can't say which parts of array is sorted in order.

After finding a port sorted in order, we should check whether the target lives in the part.
If so, we can take an ordinary binary search.
If not, we need to convert search region to another part.

# 3. Solution

- find a pivot, then binary search
```cpp
class Solution {
public:
    bool search(vector<int>& nums, int target) {
        int n = nums.size();
        if(n == 1) return nums[0] == target;

        int mid = -1;
        for(int i = 0; i < n-1; i++) {
            if(nums[i] > nums[i+1]) { mid = i; break; }
        }

        if(mid != -1) 
        {
            reverse(nums.begin(), nums.begin()+mid+1);
            reverse(nums.begin()+mid+1, nums.end());
            reverse(nums.begin(), nums.end());
        }

        auto it = lower_bound(nums.begin(), nums.end(), target);
        
        if(it == nums.end()) return false;
        else return *it == target;
    }
};
```

- optimized version
	Although the runtime was measured slower than unoptimized one, the time complexity definitely is lower than the previous solution.
```cpp
class Solution {
public:
    bool search(vector<int>& nums, int target) {
        int low = 0, high = nums.size()-1, mid;

        while(low <= high)
        {
            mid = low + (high-low)/2;

            if(nums[mid] == target) return true;

            if(nums[low] == nums[mid] && nums[mid] == nums[high]) {low++; high--;}
            else if(nums[low] <= nums[mid]) 
            {
                if((nums[low] <= target) && (target < nums[mid])) high = mid-1;
                else low = mid+1;
            }
            else
            {
                if((nums[mid] < target) && (target <= nums[high])) low = mid+1;
                else high = mid-1;
            }
        }
        
        return false;
    }
};
```