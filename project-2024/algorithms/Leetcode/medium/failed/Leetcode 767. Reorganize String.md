
- #failed #medium #greedy
- tried 54min

# 1. Problem description
Given a string `s`, rearrange the characters of `s` so that any two adjacent characters are not the same.

Return _any possible rearrangement of_ `s` _or return_ `""` _if not possible_.

**Example 1:**

**Input:** s = "aab"
**Output:** "aba"

**Example 2:**

**Input:** s = "aaab"
**Output:** ""

**Constraints:**

- `1 <= s.length <= 500`
- `s` consists of lowercase English letters.

# 2. Intuition

My first trial was inserting a character into a map by recording the number of their occurences in given string.
Then , i sorted the map by value in descending order and rearranged string by moving two pointers with proper conditions.

But there were too many edge cases that is difficult to handle with explicit conditions, so i needed to more general approach.

By refering to [this solution](https://leetcode.com/problems/reorganize-string/solutions/335312/c-beat-100/) , i got to know that we can solve this problem in greedy way.

Using the distance between first alphabet('a') and current character as index of a vector, we can find the most frequent character in given string.

If the count of frequent character is smaller than $\frac{lengthOfString+1}{2}$  , we can rearrange the string so that there is no adjacency agains same character.

A good example here )
```
11111222333
1 1 1 1 1   // step1
1 1 1 1 1 2 // step2
12121 1 1 2 // step3
12121313132 // finished
```

# 3. Solution
```cpp
class Solution {

public:

    string reorganizeString(string s) {

        vector<int> cnt(26);

        int mostFreq = 0, i = 0;

        for(char c : s)

            if(++cnt[c-'a'] > cnt[mostFreq])

                mostFreq = (c-'a');

        if(2*cnt[mostFreq]-1 > s.size()) return "";

  

        while(cnt[mostFreq]) {

            s[i] = static_cast<char>('a' + mostFreq);

            i += 2;

            cnt[mostFreq]--;

        }

  

        for(int j=0; j < 26; j++) {

            while(cnt[j]) { // > 0

                if(i >= s.size()) i = 1;

                s[i] = ('a' + j);

                cnt[j]--;

                i += 2;

            }

        }

        return s;

    }

};
```