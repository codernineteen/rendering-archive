
#failed #medium #sort #array #permutation

# 1. Problem description
A **permutation** of an array of integers is an arrangement of its members into a sequence or linear order.

- For example, for `arr = [1,2,3]`, the following are all the permutations of `arr`: `[1,2,3], [1,3,2], [2, 1, 3], [2, 3, 1], [3,1,2], [3,2,1]`.

The **next permutation** of an array of integers is the next lexicographically greater permutation of its integer. More formally, if all the permutations of the array are sorted in one container according to their lexicographical order, then the **next permutation** of that array is the permutation that follows it in the sorted container. If such arrangement is not possible, the array must be rearranged as the lowest possible order (i.e., sorted in ascending order).

- For example, the next permutation of `arr = [1,2,3]` is `[1,3,2]`.
- Similarly, the next permutation of `arr = [2,3,1]` is `[3,1,2]`.
- While the next permutation of `arr = [3,2,1]` is `[1,2,3]` because `[3,2,1]` does not have a lexicographical larger rearrangement.

Given an array of integers `nums`, _find the next permutation of_ `nums`.

The replacement must be **[in place](http://en.wikipedia.org/wiki/In-place_algorithm)** and use only constant extra memory.

# 2. Intuition
---
By starting at the end of nums vector, keep expanding search distance from 1 to nums.size()-1.
While expading the search distance, check whether it is possible to increment order lexicalgraphically by swapping pivot point with any value in the range.
If it is possible, we swap the pivot with the value and then sort rest of range using algorihtm STL.

# 3. Solution
---
```cpp
class Solution {

public:

    void nextPermutation(vector<int>& nums) {

        int pivot = nums.size()-1;

        for(int i = 1; i < nums.size(); i++) {

           int nextBig = INT_MAX;

           int ind = -1;

           for(int j = pivot-i; j < nums.size(); j++) {

               if(nums[j] > nums[pivot-i] && nums[j] < nextBig){

                    nextBig = nums[j];

                    ind = j;

                }

           }

           if(nextBig != INT_MAX) {

               int temp = nums[pivot-i];

               nums[pivot-i] = nextBig;

               nums[ind] = temp;

               sort(nums.begin()+pivot-i+1, nums.end());

               return;

           }

        }

  

        sort(nums.begin(), nums.end());

    }

};
```

# 4. Result
![](../../../images/Pasted%20image%2020240109141816.png)