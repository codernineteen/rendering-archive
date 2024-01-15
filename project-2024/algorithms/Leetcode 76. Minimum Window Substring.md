
- #failed #hard #sliding-window
- Tried 2 hour

# 1. Problem
**Example 1:**

**Input:** s = "ADOBECODEBANC", t = "ABC"
**Output:** "BANC"
**Explanation:** The minimum window substring "BANC" includes 'A', 'B', and 'C' from string t.

**Example 2:**

**Input:** s = "a", t = "a"
**Output:** "a"
**Explanation:** The entire string s is the minimum window.

**Example 3:**

**Input:** s = "a", t = "aa"
**Output:** ""
**Explanation:** Both 'a's from t must be included in the window.
Since the largest window of s only has one 'a', return empty string.

**Constraints:**

- `m == s.length`
- `n == t.length`
- `1 <= m, n <= 105`
- `s` and `t` consist of uppercase and lowercase English letters.

# 2. Idea

- expand right boundar of window until we find the substring to cover all characters in input 't'
- After found a substring that satisfy the condition, increment left boundary of window until we lack of a character to satisfy a substring condition.

# 3. Solution
```cpp
class Solution {

public:

    string minWindow(string s, string t) {

        vector<char> map(128, 0); // ascii

        for(auto c : t) map[c]++;

        int counter=t.size(), begin=0, end=0, d=INT_MAX, head=0;

        while(end < s.size()) {

            if(map[s[end++]]-- > 0) counter--;

            while(counter==0) {

                if(end-begin < d) d=end-(head=begin);

                if(map[s[begin++]]++ == 0) counter++;

            }

        }
        
        return d==INT_MAX? "" : s.substr(head, d);

    }

};
```

# 4. Review
I made an idea that is exactly same with solution, but failed to solve because i used a lot of conditions to solve this problem.
I think it is because i tried to handle every possible case with the conditions, not finiding a common rule.
Need to improve implementation skill more..


# Substring problem magice template
Most of substring problem is to find a substring under some restrictions.
A general way is to use a hashmap assited with two pointers.
```cpp
int findSubstring(string s){
        vector<int> map(128,0); // ASCII code to vector element.
        int counter; // check whether the substring is valid
        int begin=0, end=0; //two pointers, one point to tail and one  head
        int d; //the length of substring

        for() { /* initialize the hash map here */ }

        while(end<s.size()){

            if(map[s[end++]]-- ?){  /* modify counter here */ }

            while(/* counter condition */){ 
                 
                 /* update d here if finding minimum*/

                //increase begin to make it invalid/valid again
                
                if(map[s[begin++]]++ ?){ /*modify counter here*/ }
            }  

            /* update d here if finding maximum*/
        }
        return d;
  }
```