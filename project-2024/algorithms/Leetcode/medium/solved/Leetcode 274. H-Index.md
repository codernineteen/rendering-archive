
- #solved #medium #brute-force #sort 
- took 17min to solve.

# 1. Problem description

Given an array of integers `citations` where `citations[i]` is the number of citations a researcher received for their `ith` paper, return _the researcher's h-index_.

According to the [definition of h-index on Wikipedia](https://en.wikipedia.org/wiki/H-index): The h-index is defined as the maximum value of `h` such that the given researcher has published at least `h` papers that have each been cited at least `h` times.

**Example 1:**

**Input:** citations = [3,0,6,1,5]
**Output:** 3
**Explanation:** [3,0,6,1,5] means the researcher has 5 papers in total and each of them had received 3, 0, 6, 1, 5 citations respectively.
Since the researcher has 3 papers with at least 3 citations each and the remaining two with no more than 3 citations each, their h-index is 3.

**Example 2:**

**Input:** citations = [1,3,1]
**Output:** 1

**Constraints:**

- `n == citations.length`
- `1 <= n <= 5000`
- `0 <= citations[i] <= 1000`

# 2. Intuition

Based on constraints, it is enough to try $O(n^2)$ brute force approach.

The key point of my brute force approach is to clamp current h-index to lower number among number of citations and number of papers.

For example, we have an citations array `[1, 3, 1]`.
First , we can think of '0' citation.
Then there are three papers whose number of citations is higher than '0'.
In conclusion, the h-index of '0' citation is 0.

Now let's think about when the number of citations is equal to 3.
In this case, there are 1 paper whose citations number are 3.
So, h-index is 1.

By generalizing this,
- If # of citations is greater than or equal to # of papers, h-index is # of citations
- In opposite case, h-index is # of papers.

---

After i solved this problem with brute force approach, I started making out new idea to optimize my solution.

Conventionally, My optimization follow this rule :
- $O(n^2)$ -> $O(n \log n)$
- $O(n)$ -> $O(\log n)$
Therefore, in this problem, i could try $O(n \log n)$ optimization.

Let's think about we are on i-th element in citations which is sorted in descending order.
Then we can ensure that we have i-papers whose number of citations is greater or equal to `citations[i]`.

Wait, If we can ensure this fact, there is no way to generate a map.
It's just because we can check all of possible keys in our previous map with every iterations on sorted citation vector.

Finally, we can optimize this by following below algorithm.
- sort `citations` in descending order.
- apply same rule to determine h-index on every iterations.

# 3. Solution

- brute force approach
	- time complexity : $O(n^2)$
	- space complexity : $O(max(citations))$
```cpp
class Solution {
public:
    int hIndex(vector<int>& citations) {
        unordered_map<int, int> um;

        int numOfCite = 0;
        int paperCount = 0;
        while(true) {
            paperCount = 0;
            
            for(auto c : citations) {
                if(c >= numOfCite) paperCount++; 
            }

            if(paperCount == 0) break;
            else {
                um.insert(make_pair(numOfCite, paperCount));
                numOfCite++;
            }
        }

        int h = 0;
        for(auto p : um) {
            if(p.first <= p.second) h = max(h, p.first);
            else h = max(h, p.second);
        }

        return h;
    }
};
```

- optimized version
	- time complexity : $O(n \log n)$
	- space complexity : $O(1)$
```cpp
class Solution {
public:
    int hIndex(vector<int>& citations) {
        int h = 0;
        sort(citations.begin(), citations.end(), std::greater<int>());

        int papers = 1;
        for(int i = 0; i < citations.size(); i++) {
            h = max(h, min(papers, citations[i]));
            papers++;
        } 

        return h;
    }
};
```
![](../../../../../Pasted%20image%2020240219143525.png)