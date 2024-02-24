
- #solved #medium #string 
- took 16min to solve

# 1. Problem description
Roman numerals are represented by seven different symbols: `I`, `V`, `X`, `L`, `C`, `D` and `M`.

**Symbol**       **Value**
I             1
V             5
X             10
L             50
C             100
D             500
M             1000

For example, `2` is written as `II` in Roman numeral, just two one's added together. `12` is written as `XII`, which is simply `X + II`. The number `27` is written as `XXVII`, which is `XX + V + II`.

Roman numerals are usually written largest to smallest from left to right. However, the numeral for four is not `IIII`. Instead, the number four is written as `IV`. Because the one is before the five we subtract it making four. The same principle applies to the number nine, which is written as `IX`. There are six instances where subtraction is used:

- `I` can be placed before `V` (5) and `X` (10) to make 4 and 9. 
- `X` can be placed before `L` (50) and `C` (100) to make 40 and 90. 
- `C` can be placed before `D` (500) and `M` (1000) to make 400 and 900.

Given an integer, convert it to a roman numeral.

**Example 1:**

**Input:** num = 3
**Output:** "III"
**Explanation:** 3 is represented as 3 ones.

**Example 2:**

**Input:** num = 58
**Output:** "LVIII"
**Explanation:** L = 50, V = 5, III = 3.

**Example 3:**

**Input:** num = 1994
**Output:** "MCMXCIV"
**Explanation:** M = 1000, CM = 900, XC = 90 and IV = 4.

**Constraints:**

- `1 <= num <= 3999`

# 2. Intuition

Here is the most important point :
"If a number can represented with a higher Roman Interger character, there is no need to represent it with lower Roman Integer character".

For example, Let's assume that we want to represent an integer '5' into Roman.
Then we have two choices :
1. 5 -> V
2. 5 -> IIIII
In those cases, the right representation is 'V'.

With this core rule, we can make a simple algorithm to solve this problem.
1.  Align all possible Roman Integers sorted by descending order
2. Divide given input number by denominator corresponded to a Roman character.
	2.1 - If the quotient of the division is greater than zero, add the Roman character to result as much as the quotient.
	2.2  - Update the input number as remainder of the division.
3. Repeat Step 2 until the input number become zero.

---

There can be an optimization by hard-coding the cases because input will be less than 3999 always.


# 3. Solution

```cpp
class Solution {
public:
    void generateVec(vector<pair<int, string>>& vec) {
        vec.push_back(make_pair(1000, "M"));
        vec.push_back(make_pair(900, "CM"));
        vec.push_back(make_pair(500, "D"));
        vec.push_back(make_pair(400, "CD"));
        vec.push_back(make_pair(100, "C"));
        vec.push_back(make_pair(90, "XC"));
        vec.push_back(make_pair(50, "L"));
        vec.push_back(make_pair(40, "XL"));
        vec.push_back(make_pair(10, "X"));
        vec.push_back(make_pair(9, "IX"));
        vec.push_back(make_pair(5, "V"));
        vec.push_back(make_pair(4, "IV"));
        vec.push_back(make_pair(1, "I"));
    }

    string intToRoman(int num) {
        string res = "";
        vector<pair<int, string>> vec;
        generateVec(vec);

        for(auto p : vec) {
            if(num == 0) break;

            auto denominator = p.first;
            auto roman = p.second;

            auto quotient = num / denominator;
            num = num % denominator;

            if(quotient > 0) {
                for(int i = 0; i < quotient; i++)
                    res += roman;
            }
        }

        return res;
    }
};
```