
- #solved #medium #array #permutation 
- took 7 min to solve.
# 1. Problem description

Given a string `s`, you can transform every letter individually to be lowercase or uppercase to create another string.

Return _a list of all possible strings we could create_. Return the output in **any order**.

**Example 1:**

**Input:** s = "a1b2"
**Output:** ["a1b2","a1B2","A1b2","A1B2"]

**Example 2:**

**Input:** s = "3z4"
**Output:** ["3z4","3Z4"]

**Constraints:**

- `1 <= s.length <= 12`
- `s` consists of lowercase English letters, uppercase English letters, and digits.


# 2. Intuition

We have two choices for every characters, which are
- convert current character to upper or lower case
- don't convert current character to the cases.
when the current character is one of alphabets.

By constraints, we can get a string whose length is 12 in worst case.
Although we take this non-deterministic approach for all characters with the input, the algorithm requires only 144 steps and this is totally fine to solve the problem.

# 3. Solution
I created several helper functions to determine whether a character is an alphabet first and then whether the character is uppercase or lowercase.

- Time complexity : O(2^n)
```cpp
class Solution {

public:
    inline bool isDigit(char c) {
        if(c >= '0' && c <= '9') return true;
        return false;
    }

    inline bool isLower(char c) {
        if(c >= 'a' && c <= 'z') return true;
        return false;
    }


    void permut(vector<string>& container, string& s, int idx) {
        if(idx >= s.size()) return;
        if(!isDigit(s[idx])) {
            permut(container, s, idx+1); // 1. no change in char
            // 2. change in char
            if(isLower(s[idx])) {
                s[idx] = toupper(s[idx]);
                container.push_back(s);
                permut(container, s, idx+1);
                s[idx] = tolower(s[idx]);
            }
            else {
                s[idx] = tolower(s[idx]);
                container.push_back(s);
                permut(container, s, idx+1);
                s[idx] = toupper(s[idx]);
            }
        }
        else {
            permut(container, s, idx+1);
        }
    }

    vector<string> letterCasePermutation(string s) {
        vector<string> cont;
        cont.push_back(s);
        permut(cont, s, 0);
        return cont;
    }
};
```
