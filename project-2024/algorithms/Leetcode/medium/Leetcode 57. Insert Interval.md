
- #solved #medium #stack 
- took 52 min to solve


# 1. Problem description
You are given an array of non-overlapping intervals `intervals` where `intervals[i] = [starti, endi]` represent the start and the end of the `ith` interval and `intervals` is sorted in ascending order by `starti`. You are also given an interval `newInterval = [start, end]` that represents the start and end of another interval.

Insert `newInterval` into `intervals` such that `intervals` is still sorted in ascending order by `starti` and `intervals` still does not have any overlapping intervals (merge overlapping intervals if necessary).

Return `intervals` _after the insertion_.

**Example 1:**

**Input:** intervals = [[1,3],[6,9]], newInterval = [2,5]
**Output:** [[1,5],[6,9]]

**Example 2:**

**Input:** intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval = [4,8]
**Output:** [[1,2],[3,10],[12,16]]
**Explanation:** Because the new interval [4,8] overlaps with [3,5],[6,7],[8,10].

**Constraints:**

- `0 <= intervals.length <= 104`
- `intervals[i].length == 2`
- `0 <= starti <= endi <= 105`
- `intervals` is sorted by `starti` in **ascending** order.
- `newInterval.length == 2`
- `0 <= start <= end <= 105`

# 2. Intuition
I thought it was difficult to solve in-place memory when the range overlapping occurs and need to merge them.
That's why i pushed all the ranges into stack in reverse way and use another vector container for return.

My idea follows below steps :
- Generate a stack based on ranges in `intervals` in reverse direction.
- If we don't find a start point yet, keep pushing back current range into the container.
	- We should only check the range less than current end because the latter part will be checked in next iteration if we fail to find it.
- If we find a start point, Check whether it is overlapped or not (if overlapped, update start boundary of new interval)
	- search range of end point is also same
	- If we didn't find a end point yet, ignoring current range
	- If we found a end point, check whether it is overlapped or not. 
		- if overalpped, update end boundary of new interval and ignoring current range
		- If not overlapped, push back new interval into the container and current range either.
	- After inserting new interval, push back all the rest of ranges into the container.
![](../../../../images/Pasted%20image%2020240130134932.png)

# 3. Solution

- stack implementation

```cpp
class Solution {
public:
    vector<vector<int>> insert(vector<vector<int>>& intervals, vector<int>& newInterval) {
        vector<vector<int>> res;
        stack<vector<int>> stk;
        for(int i = intervals.size()-1; i>=0; --i) stk.push(intervals[i]);
        
        // edge case 1. empty intervals
        if(stk.empty()) {
            res.push_back(newInterval);
            return res;
        }
        
        int newStart = newInterval[0];
        int newEnd = newInterval[1];
        bool startFound = false;
        bool inserted =  false;

        while(!stk.empty()) {
            auto cur = stk.top();
            stk.pop();

            if(!inserted && !startFound) {
                if(newStart < cur[0]) startFound = true;
                else if(cur[0] <= newStart && newStart <= cur[1]) {
                    startFound = true;
                    newInterval[0] = cur[0];
                }
            }
            
            if(!inserted && startFound) {
                if(newEnd < cur[0]) {
                    res.push_back(newInterval);
                    startFound = false;
                    inserted = true;
                    // continue;
                }
                else if(cur[0] <= newEnd && newEnd <= cur[1]) {
                    newInterval[1] = cur[1];
                    res.push_back(newInterval);
                    startFound = false;
                    inserted = true;
                    continue;
                }
            }

            if(!startFound || inserted) res.push_back(cur);
            if(stk.empty() && !inserted) res.push_back(newInterval);  
        }

        return res;
    }
};
```

- More concise version
```cpp
class Solution {
public:
    vector<vector<int>> insert(vector<vector<int>>& intervals, vector<int>& newInterval) {
        int n = intervals.size();
        vector<vector<int>> res;

        // edge case 1. empty intervals
        if(intervals.size() == 0) {
            res.push_back(newInterval);
            return res;
        }
        
        int newStart = newInterval[0];
        int newEnd = newInterval[1];

        int i = 0;
        while(i < n && intervals[i][1] < newStart) {res.push_back(intervals[i]); i++;}
        
        if(i < n && intervals[i][0] <= newStart && newStart <= intervals[i][1]) newInterval[0] = intervals[i][0];

        while(i < n && intervals[i][1] < newEnd) i++;
        
        if(i < n && intervals[i][0] <= newEnd && newEnd <= intervals[i][1]) {newInterval[1] = intervals[i][1]; i++;}
        res.push_back(newInterval);

        while(i < n) {res.push_back(intervals[i]); i++;};

        return res;
    }
};
```