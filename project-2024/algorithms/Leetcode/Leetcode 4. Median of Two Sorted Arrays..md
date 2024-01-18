
- #failed #hard #binary-search
- tried 3 hour

# 1. Problem
Given two sorted arrays `nums1` and `nums2` of size `m` and `n` respectively, return **the median** of the two sorted arrays.

The overall run time complexity should be `O(log (m+n))`.

**Example 1:**

**Input:** nums1 = [1,3], nums2 = [2]
**Output:** 2.00000
**Explanation:** merged array = [1,2,3] and median is 2.

**Example 2:**

**Input:** nums1 = [1,2], nums2 = [3,4]
**Output:** 2.50000
**Explanation:** merged array = [1,2,3,4] and median is (2 + 3) / 2 = 2.5.

**Constraints:**

- `nums1.length == m`
- `nums2.length == n`
- `0 <= m <= 1000`
- `0 <= n <= 1000`
- `1 <= m + n <= 2000`
- `-106 <= nums1[i], nums2[i] <= 106`

# 2. Solution

- Brute force Solution ($O((m+n)log(m+n))$)
	1. Merge two arrays
	2. Sort the merged array
	3. find median
```cpp
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
    // Merge
        nums1.insert(nums1.end(), nums2.begin(), nums2.end());
    // Sort
        sort(nums1.begin(), nums1.end());
    // Find mid
        int mid = nums1.size() / 2;

        return (nums1.size() % 2 == 0) ? (nums1[mid]+nums1[mid-1])/2.0 : (double)nums1[mid];
    }
};
```


- Binary search solution
```cpp
class Solution {

public:

    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {

        int x = nums1.size();

        int y = nums2.size();

        cout << x << "," << y << endl;

        if(x > y) return findMedianSortedArrays(nums2, nums1); // swap vector to make x smaller.

  

        int left = 0, right = x;

        while(left <= right) {

            // we need to satisfy an equation : partionX + partionY = (x + y + 1) / 2

            int partitionX = (left+right)/2;

            int partitionY = (x+y+1)/2 - partitionX;

  

            // edge cases - when partion is located on the boundaries of an array, we need to set it -infinity or infinity properly

            int maxLeftX = partitionX == 0 ? INT_MIN : nums1[partitionX-1];

            int minRightX = partitionX == x ? INT_MAX : nums1[partitionX];

            int maxLeftY = partitionY == 0 ? INT_MIN : nums2[partitionY-1];

            int minRightY = partitionY == y ? INT_MAX : nums2[partitionY];

  

            if(maxLeftX <= minRightY && maxLeftY <= minRightX)

                return (x+y)%2==0 ?

                    (max(maxLeftX, maxLeftY) + min(minRightX, minRightY)) / 2.0 :

                    max(maxLeftX, maxLeftY);

            else if(maxLeftX > minRightY) {

                // left is too big, need to move range to left

                right = partitionX-1;

            }

            else {

                // left is too small, need to move range to right

                left = partitionX+1;

            }

        }

  

        return -1;

    }

};
```