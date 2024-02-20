
# 1. Problem Description

Given two strings `s` and `t`, return `true` _if_ `t` _is an anagram of_ `s`_, and_ `false` _otherwise_.

An **Anagram** is a word or phrase formed by rearranging the letters of a different word or phrase, typically using all the original letters exactly once.

**Example 1:**

**Input:** s = "anagram", t = "nagaram"
**Output:** true

**Example 2:**

**Input:** s = "rat", t = "car"
**Output:** false

**Constraints:**

- `1 <= s.length, t.length <= 5 * 104`
- `s` and `t` consist of lowercase English letters.

# 2. Intuition

Given description mentioning "An **Anagram** is a word or phrase formed by rearranging the letters of a different word or phrase, typically using all the original letters exactly once."

That means there is no need to apply check algorithm if the lengths of two input strings are different. -> return false right away.

When the lengths are same, I followed this algorithm :
- store all characters of 's' in hash map
- while iterating every character of t, 
	- if there is no character in map, return false,
	- if any decrement of key makes negativa value, return false

If we escape the while loop without return, we can return true eventually.


# 3. Solution
```cpp
class Solution {
public:
    bool isAnagram(string s, string t) {
        if(s.length() != t.length()) return false;
        else {
            unordered_map<char, int> um;
            for(auto c : s) um[c]++;
            for(auto c : t) {
                if(um.find(c) == um.end()) return false;
                else {
                    if(--um[c] < 0) return false;
                }
            }

            return true;
        }
    }
};
```
