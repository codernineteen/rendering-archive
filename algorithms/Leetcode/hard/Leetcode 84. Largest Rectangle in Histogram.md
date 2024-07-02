
- #failed #Hard #stack #dynammic-programming
- tried 1hour

# 1. Problem description
Given an array of integers `heights` representing the histogram's bar height where the width of each bar is `1`, return _the area of the largest rectangle in the histogram_.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/01/04/histogram.jpg)

**Input:** heights = [2,1,5,6,2,3]
**Output:** 10
**Explanation:** The above is a histogram where width of each bar is 1.
The largest rectangle is shown in the red area, which has an area = 10 units.

**Example 2:**

![](https://assets.leetcode.com/uploads/2021/01/04/histogram-1.jpg)

**Input:** heights = [2,4]
**Output:** 4

**Constraints:**

- `1 <= heights.length <= 105`
- `0 <= heights[i] <= 104`

# 2. Idea
For any given index 'i', we can find a rectangle that has maximum area by finding leftmost edge and rightmost edge.
The leftmost boundary and rightmost boundary should satisfy that there is no height smaller than 'height of index i' in range `[leftmost, rightmost]`.

After finding the boundaries, we can compute the area following the equaiton
`area = max(area, rightmost_index - leftmost_index - 1) * height_of_i`

Then how to find the boundaries?
Let's see how we can find leftmost boundary.
```
// With given index i
int pointer = i-1;
while(pointer >= 0 && heights[pointer] >= heights[i]) {
	p--;
}
leftOfCurrent[i] = pointer;
```
As you can see, we can traverse left part of heights vector based on current index 'i' and check whether the value pointer points to is greater or equal to heights of i.

This works fine, but the above way has redundancy when you try same thing on next index. For example, after index i, you will take same steps for index i+1. If height of i and i+1 is same, there is a redundancy.

To remove this problem, we can jump to another index by using 'leftOfCurrent' vector itself.
For example, Let's say that we have a vector like :
`[10, 9, 8, 7, 6]`
When we are on 5th element (the value is 6), we can reuse pre computed 'leftOfCurrent' by check what the result of 4th element was.
1. 7 is bigger than 6 -> Ok what's next?
2. 7 already knows that 10 is the leftmost boundary. 

By this intuition, we can optimize code this way :
```
// With given index i
int pointer = i-1;
while(pointer >= 0 && heights[pointer] >= heights[i]) {
	pointer = leftOfCurrent[pointer]; // next pointer will be the leftmost boundary of current pointer.
}
leftOfCurrent[i] = pointer;
```

# 3. Solution
```cpp
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        int n = heights.size();

        if(n == 0) return 0;
  
        vector<int> lessFromLeft(n, 0);
        vector<int> lessFromRight(n, 0);
        lessFromRight[n-1] = n;
        lessFromLeft[0] = -1;

  

        for(int i = 1; i < n; i++) {
            int p = i - 1;

            while(p >= 0 && heights[p] >= heights[i]) {
                p = lessFromLeft[p];
            }
            lessFromLeft[i] = p;
        }

  

        for(int i = n-2; i > -1; i--) {
            int p = i + 1;

            while(p < n && heights[p] >= heights[i]) {
                p = lessFromRight[p];
            }
            lessFromRight[i] = p;

        }
        
        int area = 0;

        for(int i = 0; i < n; i++) {
            area = max(area, (lessFromRight[i] - lessFromLeft[i] - 1) * heights[i]);
        }
        return area;
    }

};
```

# 4. Review
This is one of the most difficult algorihtm problem that i've ever tried so far.
Even with solutions, it was quite difficult to understand how it works for an hour.

After understanding the solution, i realized that I should have followed this process,
1. How to find a maximum area based on current position?
2. How we can do it faster if it possible?
3. What is the utmost area among the maximums for each position?
If i decomposed the problem in several parts, it would be more easier than trying to solve it globally.