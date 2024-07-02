
- #solved #medium #sort #array 
- Took 14min to solve
 
# 1. Problem description
Given a string `s`, sort it in **decreasing order** based on the **frequency** of the characters. The **frequency** of a character is the number of times it appears in the string.

Return _the sorted string_. If there are multiple answers, return _any of them_.

**Example 1:**

**Input:** s = "tree"
**Output:** "eert"
**Explanation:** 'e' appears twice while 'r' and 't' both appear once.
So 'e' must appear before both 'r' and 't'. Therefore "eetr" is also a valid answer.

**Example 2:**

**Input:** s = "cccaaa"
**Output:** "aaaccc"
**Explanation:** Both 'c' and 'a' appear three times, so both "cccaaa" and "aaaccc" are valid answers.
Note that "cacaca" is incorrect, as the same characters must be together.

**Example 3:**

**Input:** s = "Aabb"
**Output:** "bbAa"
**Explanation:** "bbaA" is also a valid answer, but "Aabb" is incorrect.
Note that 'A' and 'a' are treated as two different characters.

**Constraints:**

- `1 <= s.length <= 5 * 105`
- `s` consists of uppercase and lowercase English letters and digits.

# 2. Intuition

This problem is straightforward.
The sorting target is a length of each characters, not the character order in dictionary.

In conclusion, What we need to do is :
- count common characters
- sort character map by count value against each of keyrs
- build new string

# 3. Solution

- 33ms took for this initial implementation
```cpp
class Solution {

public:

    string frequencySort(string s) {
    // 1. count common characters into map.
        unordered_map<char, int> chars;
        for(auto c : s) chars[c]++;

	// 2. build a pair to sort them
        vector<pair<char, int>> temp;
        for(auto it : chars) temp.push_back({it.first, it.second});

	// 3. custom comparer
        auto comp = [&temp](const pair<char, int>& a, const pair<char, int>& b) {
            return a.second > b.second;
        };
   //4. sort
        sort(temp.begin(), temp.end(), comp);
   //5. build new string
        s = "";
        int i = 0;
        while(i < temp.size()) {
            s += temp[i].first;
            temp[i].second--;
            if(temp[i].second == 0) i++;
        }
        return s;
    }
};
```

- optimized version
	- time complexity : O(nlogn)
	- space complexity : O(1)
	If we use ascii vector with size of 128, we can reduce some redundancy in the intial implementation. (by subtracting each character by '0' to get its position.)
```cpp
class Solution {
public:
   string frequencySort(string s) {
        vector<string> charMap(128, "");
        for(auto c : s) charMap[c - '0'] += c;

        auto comp = [&charMap](const string& a, const string& b) {
            return a.length() > b.length();
        };
        sort(charMap.begin(), charMap.end(), comp);


        s = "";
        for(auto& t : charMap) {
            if(t == "") break;
            s += t;
        }
        return s;
    }
};
```