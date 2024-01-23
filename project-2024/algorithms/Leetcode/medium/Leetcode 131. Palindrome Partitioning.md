
- #solved #medium #string/array 
- took 43min to solve

# 1. Problem description
Given a string `s`, partition `s` such that every 

substring

 of the partition is a 

**palindrome**

. Return _all possible palindrome partitioning of_ `s`.

**Example 1:**

**Input:** s = "aab"
**Output:** [["a","a","b"],["aa","b"]]

**Example 2:**

**Input:** s = "a"
**Output:** [["a"]]

**Constraints:**

- `1 <= s.length <= 16`
- `s` contains only lowercase English letters.

# 2. Intuition

As always, i started with brute force approach.
First i drew all possible partitioning by using parse tree.
Once we determined a prefix, we just need to solve the problem against the rest part of string.
In conclusion, the solutions of sub-problem is combined together into a solution of total problem.
![](../../../../images/Pasted%20image%2020240117141722.png)

Because i had no idea to optimize further at the moment, i used recursion & memoization techniques to avoid a process that checks if a string is palindrome again when it has already been computed before.

Also when we move on next partition pivot, we need to remove previous prefix from temporary vector.
Hence, Backtracking comes in handy in this case.

# 3. Solution
By using reference parameters, i can optimize the runtime up to 104ms, which beats 62.01%.

```cpp
class Solution {
    vector<vector<string>> res;
    // memoization for palindrome and non-palindrome strings
    unordered_set<string> palindromes;
    unordered_set<string> npalindromes;

public:
// a helper function to check palindrome check
    bool isPalindrome(string& s) {
        int l = 0, r = s.length()-1;
        while(l <= r) {
            if(s[l] != s[r]) return false;
            l++;
            r--;
        }
        return true;
    }
// recursion 
// s :  a substring parameter passed by caller
// tmp : a container to store intermediate partitions
    void solve(string s, vector<string>& tmp) {
        if(s.length() == 1) {
            tmp.push_back(s);
            res.push_back(tmp);
            tmp.pop_back();
            return;
        }

// If we found that this string is palindrome, we add it to result
        if(palindromes.find(s) != palindromes.end()) {
            tmp.push_back(s);
            res.push_back(tmp);
            tmp.pop_back();
        }
        else if(isPalindrome(s)) {
            palindromes.insert(s);
            tmp.push_back(s);
            res.push_back(tmp);
            tmp.pop_back();
        }

  
// changing partition pivot in for-loop (the pivot is i)
        for(int i=0; i < s.length()-1; i++) {
            string left = s.substr(0, i+1);
            
            if(npalindromes.find(left) != npalindromes.end()) continue;
            else if(!isPalindrome(left)) {
                npalindromes.insert(left);
                continue;
            }
            string right = s.substr(i+1, s.length()-i);
            tmp.push_back(left);
            solve(, tmp);
            tmp.pop_back();
        }
    }

  

    vector<vector<string>> partition(string s) {
	    vector<string> tmp{};
        solve(s, tmp);
        return res;
    }

};
```