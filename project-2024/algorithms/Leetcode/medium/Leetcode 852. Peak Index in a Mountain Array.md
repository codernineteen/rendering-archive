
- #solved #medium #binary-search 
- Took 12min to solve

# 1. Problem description

An array `arr` is a **mountain** if the following properties hold:

- `arr.length >= 3`
- There exists some `i` with `0 < i < arr.length - 1` such that:
    - `arr[0] < arr[1] < ... < arr[i - 1] < arr[i]`
    - `arr[i] > arr[i + 1] > ... > arr[arr.length - 1]`

Given a mountain array `arr`, return the index `i` such that `arr[0] < arr[1] < ... < arr[i - 1] < arr[i] > arr[i + 1] > ... > arr[arr.length - 1]`.

You must solve it in `O(log(arr.length))` time complexity.

**Example 1:**

**Input:** arr = [0,1,0]
**Output:** 1

**Example 2:**

**Input:** arr = [0,2,1,0]
**Output:** 1

**Example 3:**

**Input:** arr = [0,10,5,2]
**Output:** 1

**Constraints:**

- `3 <= arr.length <= 105`
- `0 <= arr[i] <= 106`
- `arr` is **guaranteed** to be a mountain array.


# 2. Intuition

First, Let's understand problem description by visualizing it.
![](../../../../images/Pasted%20image%2020240207151414.png)

Now we know that we should find i-th element which is bigger than its adjacent elements.
In the problem description, it gives us a huge hint "solve in O(log(arr.length-1))".
In most cases, O(log n) algorithm follows binary search.
So I tried making an binary search application idea first.
By modifying the common binary search a bit, we can solve this problem easily.
1. If the mid element is bigger than (mid-1) and (mid+1), that is the exactly peak of given mountain and we return the index of the peak.
2. If left side of mid is smaller than 'mid' and right side is bigger than 'mid', It means that we are on left slope of mountain.  So we should shrink the search range from mid to right
3. In opposite case, we are on right-side slope of mountain, so we need to set the range from left to mid.

**A mistake in below image** was updating left and right as previous / next index of middle index. It will discard the crucial search point when the answer is located on right after mid index or right before mid index.
So we should update left and right 
```cpp
left = mid
...
right = mid
```
, not
```cpp
left = mid + 1;
...
right = mid - 1;
```
![](../../../../images/Pasted%20image%2020240207151409.png)


# 3. Solution

96% beats
- time complexity : O(log n)
- space complexity : O(1)

```cpp
class Solution {

public:

    inline int modifiedBinarySearch(vector<int>& arr, int low, int high) {
        int ind = -1;
        int mid = (low + high + 1) / 2;

        while(low < high) {
            if(arr[mid-1] < arr[mid] && arr[mid] > arr[mid+1]) return mid;
            else if(arr[mid-1] < arr[mid] && arr[mid] < arr[mid+1]) low = mid;
            else high = mid;
            mid = (low+high+1)/2;
        }

        return ind;
    }

  

    int peakIndexInMountainArray(vector<int>& arr) {
        return modifiedBinarySearch(arr, 0, arr.size()-1);
    }
};
```
