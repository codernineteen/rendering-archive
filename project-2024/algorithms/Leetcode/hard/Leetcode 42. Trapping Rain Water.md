
- #solved #hard #twopointer 
- took 35min to solve

# 1. Problem Description
Given `n` non-negative integers representing an elevation map where the width of each bar is `1`, compute how much water it can trap after raining.

**Example 1:**

![](https://assets.leetcode.com/uploads/2018/10/22/rainwatertrap.png)

**Input:** height = `[0,1,0,2,1,0,1,3,2,1,2,1]`
**Output:** 6
**Explanation:** The above elevation map (black section) is represented by array `[0,1,0,2,1,0,1,3,2,1,2,1]`. In this case, 6 units of rain water (blue section) are being trapped.

**Example 2:**

**Input:** height = `[4,2,0,3,2,5]`
**Output:** 9

**Constraints:**
- `n == height.length`
- `1 <= n <= 2 * 104`
- `0 <= height[i] <= 105`
# 2. Intuition

The very naive approach is a brute force approach.
- we find a max height.
- traverse height vector and find positions of heights that have same value with current height.
- After the traversal, we need to compute how much water we can contain between two discrete heights.
- Repeat this steps until we encounter zero-valued height.

But as we can see in the constraints, we need 2,000,000,000 steps at worst case and this will definitely exceeds time limitation.

I had to find a way to solve this problem in O(n) time complexity and experimented 'two pointers' approach.
Let' see how we can compute water in sub-range.
With a given pointer 'left', we can compute the amount of water we can contain whenever we increment right pointer.
Meanwhile, If we encounter a height that is greater or equal to height of left, we can't contain the water anymore because it generates a barrier between two ranges.
Therefore, we need to store the temporary water into total amount at this moment.
![](../../../../images/Pasted%20image%2020240123141920%201.png)

But there is an edge case.
What if we end up the traversal without storing temporary water into total like below?
For this case, we can simply resolve this issue by taking same steps in a reverse way.
The new end is equal to position of left in previous traversal.
![](../../../../images/Pasted%20image%2020240123142452%201.png)

By refering to a solution in leetcode, i realized that we can optimize this further.
Instead of traversing in a reverse way, we can change the direction whenver height of left is less larger than the height of right so that we can store water toward higher height.
This generalization avoids the edge case i mentioned before.

# 3. Solution

- initial implementation
```cpp
class Solution {

public:

    int trap(vector<int>& height) {
        int water = 0;
        int l=0, r=0;

        // increment left, right pointer until we encounter non-zero height.
        while(height[l] == 0) {
            l++; r++;
            if(l >= height.size()) return water;
        }
        r++;

        int tempContainer = 0;
        while(r < height.size()) {
            if(height[l] > height[r]) {
                // hold water in temp container
                tempContainer += height[l] - height[r];
                r++;
            }
            else {
                // If the height of right pointer is greater or equal to left's, store the amount of temp container in water.
                water += tempContainer;
                tempContainer = 0;
                l = r;
                r++;
            }
        }


        tempContainer = 0;
        // if left ends up with the value that is less than right,
        // it means that we couldn't find a height that is greater or equal to left before the traversal ends
        // Hence, we should do the same steps backward at this time.
        if(l < r) {
            int end = l;
            r = height.size()-1;
            l = height.size()-2;

            while(l >= end) {
                if(height[l] < height[r]) {
                    tempContainer += height[r] - height[l];
                    l--;
                }
                else {
                    water += tempContainer;
                    tempContainer = 0;
                    r = l;
                    l--;
                }
            }
        }
  
        return water;
    }
};
```


- optimized version
	ref : https://leetcode.com/problems/trapping-rain-water/solutions/3401992/100-detailed-explaination-with-pictures-in-c-java-python-two-pointers/
```cpp
int trap(vector<int>& height) {

       int n = height.size();
       int leftMax = INT_MIN;
       int rightMax = INT_MIN;
       int left = 0;
       int right = n-1;
       int ans = 0;
       while(left<=right){
           if(height[left]<=height[right]){
               leftMax = max(height[left], leftMax);
               ans+= (leftMax - height[left]);
               left++;
           }else{
               rightMax = max(height[right], rightMax);
               ans+= (rightMax - height[right]);
               right--;
           }
	  }
       return ans;
   }
```