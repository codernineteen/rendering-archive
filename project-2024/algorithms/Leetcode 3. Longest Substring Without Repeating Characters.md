
- #Solved #Meidum 
- Took 41min
- 2024/1/6

# 1. Problem
Given a string `s`, find the length of the **longest** 

**substring**

 without repeating characters.

**Example 1:**

**Input:** s = "abcabcbb"
**Output:** 3
**Explanation:** The answer is "abc", with the length of 3.

**Example 2:**

**Input:** s = "bbbbb"
**Output:** 1
**Explanation:** The answer is "b", with the length of 1.

**Constraints:**

- `0 <= s.length <= 5 * 10^4`
- `s` consists of English letters, digits, symbols and spaces.

# 2. Intuition

Intuitively, Because maximum length of given string input can be 50000, we can take a Brute force approach whose time complexity corresponds to O(n^2) == 2,500,000,000 in worst case.
Now we got a sense we need to solve this problem in one traverse.

The critical point in this problem is to find a substring without **repeating** any characters.
Therefore, when we find the very first repetition in subsequence, we can ensure that there is no larger subsequence than now so far.

For next step, we need to find where is the leftmost character against the repeated character because the next character of the leftmost duplication will be new start point.
I think that the search time is inevitable for this procedure.

With above evidences, we can convince that 'sliding window' algorithm perfectly goes well with this problem.

# 3. Implementation
```cpp
class Solution {

public:

    int lengthOfLongestSubstring(string s) {
        int l = 0;
        int r = 0;
        int maxLen = 0;
        
        set<char> sub; // a set to check repetition

        while(r < s.size()) {
            if(sub.find(s[r]) == sub.end()) { // expand window
                sub.insert(s[r]);
                r++;
            }
            else {
                //dup -> shrink window
                maxLen = max(maxLen, r - l);
                while(s[l] != s[r]) {
                    sub.erase(s[l]);
                    l++;
                }
				//increment both
                l++;
                r++;
            }
        }
        // for the case when there is no repetition at all.
        maxLen = max(maxLen, r-l);

        return maxLen;

    }

};
```

# 4. Result
![[Pasted image 20240106172438.png]]