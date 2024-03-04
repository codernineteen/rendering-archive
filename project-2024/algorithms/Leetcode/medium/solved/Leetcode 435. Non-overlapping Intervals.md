
- #solved #medium #greedy #sort 
- took 17min to solve

# 1. Problem description

Given an array of intervals `intervals` where `intervals[i] = [starti, endi]`, return _the minimum number of intervals you need to remove to make the rest of the intervals non-overlapping_.

**Example 1:**

**Input:** intervals = [[1,2],[2,3],[3,4],[1,3]]
**Output:** 1
**Explanation:** [1,3] can be removed and the rest of the intervals are non-overlapping.

**Example 2:**

**Input:** intervals = [[1,2],[1,2],[1,2]]
**Output:** 2
**Explanation:** You need to remove two [1,2] to make the rest of the intervals non-overlapping.

**Example 3:**

**Input:** intervals = [[1,2],[2,3]]
**Output:** 0
**Explanation:** You don't need to remove any of the intervals since they're already non-overlapping.

**Constraints:**

- `1 <= intervals.length <= 105`
- `intervals[i].length == 2`
- `-5 * 104 <= starti < endi <= 5 * 104`

# 2. Intuition

Let's suppose that we are on the middle of intervals and we have some candidate intervals at current position that we're on.

What is the best interval to minimize future overlappings?
We can say that the shortest interval is the best right away.
The shortest interval will reduce the possibilities that it overlaps with other intervals regardless of where the intervals start.

If you can't understand, let's take an extreme case whose start and end index are same.
In this case, the interval is just a bonus! It will not be overlapped any other intervals surround it by problem description.

Now we got the core principle to solve this problem. But How we can keep the cosistency of this rule?
Sorting intervals by start index will helps us to achieve this consistency.
Let's see a visualization.
1. If the shorter interval comes in, we need to update last endpoint as end index of shorter one.
	If not, we keep the last endpoint.
![](../../../../../images/Pasted%20image%2020240304193831.png)
2. We sorted intervals by start index. Therefore (2,2) interval never happen once we have already moved on start index after 3.
![](../../../../../images/Pasted%20image%2020240304194019.png)


# 3. Solution
- Time complexity : $O(n \log{n})$
- Space complexity : $O(1)$

```cpp
class Solution {
public:
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        int res = 0;
        sort(intervals.begin(), intervals.end(), [&](const vector<int>& a, const vector<int>& b) {
            return a[0] < b[0];
        });

        int pEnd = -50001;
        for(const auto& interval : intervals) 
        {
            if(interval[0] < pEnd) {
                res++;
                if(interval[1] < pEnd) pEnd = interval[1];
            }
            else pEnd = interval[1];
        }

        return res;
    }
};
```