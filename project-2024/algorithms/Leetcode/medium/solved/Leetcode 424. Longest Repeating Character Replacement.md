
- #failed #medium #sliding-window 
- tried 1h to solve

# 1. Problem description
You are given a string `s` and an integer `k`. You can choose any character of the string and change it to any other uppercase English character. You can perform this operation at most `k` times.

Return _the length of the longest substring containing the same letter you can get after performing the above operations_.

**Example 1:**

**Input:** s = "ABAB", k = 2
**Output:** 4
**Explanation:** Replace the two 'A's with two 'B's or vice versa.

**Example 2:**

**Input:** s = "AABABBA", k = 1
**Output:** 4
**Explanation:** Replace the one 'A' in the middle with 'B' and form "AABBBBA".
The substring "BBBB" has the longest repeating letters, which is 4.
There may exists other ways to achieve this answer too.

**Constraints:**

- `1 <= s.length <= 105`
- `s` consists of only uppercase English letters.
- `0 <= k <= s.length`
# 2. Solution

```cpp
class Solution {

public:

    int characterReplacement(string s, int k) {
        unordered_map<char, int> um;
        int n = s.size();
        // left : left pointer of window
        // right : right pointer of window
        // maximum : mamximum character of current substring.
        int left = 0, right = 0, maximum = 0;

        int ans = -1;
        while(r < n)
        {
            um[s[r]]++;
            maximum = max(maximum, um[s[r]]);
// If (the length of current window - maximum) > k, it is impossible to happen.
// so we increment left pointer of window and decrement the character at the position that the left pointer pointed to before.
            if((r-l+1) - maximum > k)
            {
                um[s[l]]--;
                l++;
            }
            // compute answer every iteration
            ans = max(ans, (r-l+1));
            r++;
        }
        
        return ans;
    }

};
```