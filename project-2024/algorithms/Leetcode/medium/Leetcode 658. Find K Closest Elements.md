
- #solved #medium #two-pointer #binary-search 
- took 25min to solve

# 1. Problem description
Given a **sorted** integer array `arr`, two integers `k` and `x`, return the `k` closest integers to `x` in the array. The result should also be sorted in ascending order.

An integer `a` is closer to `x` than an integer `b` if:

- `|a - x| < |b - x|`, or
- `|a - x| == |b - x|` and `a < b`

**Example 1:**

**Input:** arr = [1,2,3,4,5], k = 4, x = 3
**Output:** [1,2,3,4]

**Example 2:**

**Input:** arr = [1,2,3,4,5], k = 4, x = -1
**Output:** [1,2,3,4]

**Constraints:**

- `1 <= k <= arr.length`
- `1 <= arr.length <= 104`
- `arr` is sorted in **ascending** order.
- `-104 <= arr[i], x <= 104`

# 2. Intuition

I thought the core of this problem is to find a initial location of starting point.

First, i made up all the possible cases that can be a starting point.
1. when a starting point is equal to 'x'
	In this case, we need to add the number first and set 'left' to 'start-1' and 'right' to 'start+1'
2. when a starting point is intermediate value between left and right
	In this case, we set left to 'start-1' and right to 'start'
3. When starting point is located on one of boundaries of the array.
	In implementation, we just need to care about right boundary case because we can generalize left-boundary with case.2.

After we set a starting point and left/right pointers appropriately, What we need to do is only to follow the given formulas in the problem description.
The one edge case is that one of left and right pointers reaches a boundary of the array when the other is still at the middle of array.
In this case, we just need to add all the numbers toward possible direction until we exhaust available 'k'.

![](../../../../images/Pasted%20image%2020240131121100.png)

Can we make the logic simpler?
By refering to few solutions , i got a clear sense for it.
We can think of this problem to find a range which satisfy the closest elements condition.
Regardless of the position of x, we need to decrease right if the current left is closer to 'x' than current right.
On the other hand, we need to increment left when current right is closer to 'x' than current left.
We can keep this rule because a given array was sorted in ascending order.
![](../../../../images/Pasted%20image%2020240131123315.png)
# 3. Solution

- Without optimization
```cpp
class Solution {
public:
	// return a left number closer to x than right
    inline bool isLeftCloser(int& a, int& b, int& x) {
        if(abs(a-x) < abs(b-x)) return true;
        else if(abs(a-x) > abs(b-x)) return false;
        else {
            if(a < b) return true;
            else return false;
        }
    }

    vector<int> findClosestElements(vector<int>& arr, int k, int x) {
        int n = arr.size();
        vector<int> res;
        int left, right;
// find a starting point
        for(int i = 0; i < n; i++) {
            if(x < arr[i]) {
                left = i-1;
                right = i;
                break;
            }   
            else if(arr[i] == x) {
                res.push_back(x);
                k--;
                left = i-1;
                right = i+1;
                break;
            }
            
            if(i == n-1) {
                left = n-1;
                right = n;
            }
        }
// add numbers to vector
        while(k > 0) {
            if(left >= 0 && right <= n-1) {
                if(isLeftCloser(arr[left], arr[right], x)) {
                    res.insert(res.begin(), arr[left]);
                    k--;
                    left--;
                }
                else {
                    res.push_back(arr[right]);
                    k--;
                    right++;
                }
            }
            else if(left < 0) {
                while(k > 0) {
                    res.push_back(arr[right]);
                    k--;
                    right++;
                }
            }
            else {
                while(k > 0) {
                    res.insert(res.begin(), arr[left]);
                    k--;
                    left--;
                }
            }
        }

        return res;
    }
};
```


- Using binary search, i could optimize the process of finding a start point further
```cpp
class Solution {
public:
    inline bool isLeftCloser(int& a, int& b, int& x) {
        if(abs(a-x) < abs(b-x)) return true;
        else if(abs(a-x) > abs(b-x)) return false;
        else {
            if(a < b) return true;
            else return false;
        }
    }

    vector<int> findClosestElements(vector<int>& arr, int k, int x) {
        int n = arr.size();
        vector<int> res;
        int left, right;

        auto it = upper_bound(arr.begin(), arr.end(), x);
        if(it != arr.end()) {
            int ind = it-arr.begin();
            if(arr[ind] == x) {
                res.push_back(x);
                k--;
                left = ind-1;
                right = ind+1;
            }
            else {
                left = ind-1;
                right = ind;
            }
        }
        else {
            left = n-1;
            right = n;
        }

        while(k > 0) {
            if(left >= 0 && right <= n-1) {
                if(isLeftCloser(arr[left], arr[right], x)) {
                    res.insert(res.begin(), arr[left]);
                    k--;
                    left--;
                }
                else {
                    res.push_back(arr[right]);
                    k--;
                    right++;
                }
            }
            else if(left < 0) {
                while(k > 0) {
                    res.push_back(arr[right]);
                    k--;
                    right++;
                }
            }
            else {
                while(k > 0) {
                    res.insert(res.begin(), arr[left]);
                    k--;
                    left--;
                }
            }
        }

        return res;
    }
};
```

- range-based solution
```cpp
class Solution {

public:
    vector<int> findClosestElements(vector<int>& arr, int k, int x) {
        int n = arr.size();
        int left = 0, right = n-1;
        while(right - left >= k) {
            if(abs(arr[left] - x) > abs(arr[right] - x)) left++;
            else right--;
        }
        return vector<int>(arr.begin()+left, arr.begin()+right+1);
    }

};
```