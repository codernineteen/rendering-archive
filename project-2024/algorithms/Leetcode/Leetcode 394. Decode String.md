
- #solved #medium #string #stack #recursion
- took 26min to solve.

# 1. Problem description

Given an encoded string, return its decoded string.

The encoding rule is: `k[encoded_string]`, where the `encoded_string` inside the square brackets is being repeated exactly `k` times. Note that `k` is guaranteed to be a positive integer.

You may assume that the input string is always valid; there are no extra white spaces, square brackets are well-formed, etc. Furthermore, you may assume that the original data does not contain any digits and that digits are only for those repeat numbers, `k`. For example, there will not be input like `3a` or `2[4]`.

The test cases are generated so that the length of the output will never exceed `105`.

**Example 1:**

**Input:** s = "3[a]2[bc]"
**Output:** "aaabcbc"

**Example 2:**

**Input:** s = "3[a2[c]]"
**Output:** "accaccacc"

**Example 3:**

**Input:** s = "2[abc]3[cd]ef"
**Output:** "abcabccdcdcdef"

# 2. Intuition

My idea is to decode nested array recursively until there is no nested string anymore whenver we encounter a character `'['`.
 When we parse nested substring, we pass reference of index integer so that it increment the value in reference and we can continue the process with the incremented index in outer function.

Let's take one of given examples in description.
a string input :  `3[a2[c]]`
To decode this string in a right way, we need to repeat `a2[c]` 3 times and `c`  two times for each of `a2[c]`.

To make this simple, Let's substitute each nested substring with some of variables
- `A = a2[c]`
- `B = 2[c]`
Now what we need to decode is `3A=3(aB)` and `B`.
For that, we need to decode what the `B` is first and this is exact;y why i took an recursive approach in this problem.

In other words, whenever we encounter a nested character `[`, we should stop adding a character to decoded string and start to find what the nested decode string is first.
# 3. Solution

Note that it runs as fast as stack approach
- Recursion
```cpp
class Solution {

public:

    string solve(string s, int& idx) {
        string res = "";
        string numStr = "";
        int multiplier = 0;
        
        for(; idx < s.length(); idx++) {
            if(s[idx] >= '0' && s[idx] <= '9') numStr += s[idx];
            else if(s[idx] == '[') {
                int multiplier = stoi(numStr);
                numStr = "";
                idx++;
                string subStr = solve(s, idx); // pass string and
                for(int i = 1; i <= multiplier; i++) res += subStr;
            }
            else if(s[idx] == ']') return res;
            else res += s[idx];
        }

        return res;
    }

    string decodeString(string s) {
        int start = 0;
        return solve(s, start);
    }
};
```

- Stack
The other approach is to use stack (https://leetcode.com/problems/decode-string/solutions/472087/0ms-c-solution-using-one-stack/)
```cpp
class Solution {

public:
    string decodeString(string s) {
        stack<char> stk;
        string res = "";
  
        for(auto c : s) {
            if(c != ']') {
                stk.push(c);
            }
            else {
                string cur = "";
                while(stk.top() != '[') {
                    cur = stk.top() + cur;
                    stk.pop();
                }
                stk.pop(); // pop '['
                
                //compute number
                string num = "";
                while(!stk.empty() && stk.top() >= '0' && stk.top() <= '9') {
                    num = stk.top() + num;
                    stk.pop();
                }
                int times = stoi(num);
                while(times--) {
                    for(auto c : cur) stk.push(c);
                }
            }
        }

        while(!stk.empty()) {
            res = stk.top() + res;
            stk.pop();
        }

        return res;
    }

};
```