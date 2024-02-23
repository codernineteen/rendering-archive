
- #solved #medium #string/array #sparse-matrix
- took 56min to solve

# 1. Problem description

The string `"PAYPALISHIRING"` is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility)

P   A   H   N
A P L S I I G
Y   I   R

And then read line by line: `"PAHNAPLSIIGYIR"`

Write the code that will take a string and make this conversion given a number of rows:

string convert(string s, int numRows);

**Example 1:**

**Input:** s = "PAYPALISHIRING", numRows = 3
**Output:** "PAHNAPLSIIGYIR"

**Example 2:**

**Input:** s = "PAYPALISHIRING", numRows = 4
**Output:** "PINALSIGYAHRPI"
**Explanation:**
P     I    N
A   L S  I G
Y A   H R
P     I

**Example 3:**

**Input:** s = "A", numRows = 1
**Output:** "A"

**Constraints:**

- `1 <= s.length <= 1000`
- `s` consists of English letters (lower-case and upper-case), `','` and `'.'`.
- `1 <= numRows <= 1000`

# 2. Intuition

At a glance, I thought it was enought to use two-dimensional array to solve this problem.
Although i didn't submit the solution with two-dimensional array eventually, i encountered many buffer overflow errors while trying to solving that way.

Then i made up some idea to implement the solution in a more compact way without suffering from boundary errors.

Here is my intuition,
The example is :
- given string : "PAYPALISHIRING"
- given rows : 4
Then we can make a zigzag conversion following description like below image.
![](../../../../../images/Pasted%20image%2020240223132819.png)

But there are a lot of holes as you can see in the image.
I thought this an example of sparse matrix , so needed to store string in memory-optimized way.
If we try compressing the two-dimensional matrix to the left-side of each rows, we can make a one-dimensional array like the right-side of the image.
And this compression doesn't harm the consistency in the solution at all!

In conclusion, Instead of making two-dimensional array and insert elements with complex logic, we just need to create a 1D vector and reverse the direction of pushing element whenever we count the boundaries of row(0 or numRows).

# 3. Solution
- time complexity : O(n)
- space complexity : O(n)
![](../../../../../images/Pasted%20image%2020240223133348.png)
```cpp
class Solution {
public:
    string convert(string s, int numRows) {
        if(numRows == 1) return s;
        else {
            vector<string> um(numRows, "");
            int n = s.length();
            int row = 0;
            int it = 0;
            bool straight = true;
            while(it < n) {
                um[row] += s[it];
                it++;
                if(straight) {
                    row++;
                    if(row == numRows-1) straight = false;
                }
                else {
                    row--;
                    if(row == 0) straight = true;
                }
            }

            string res = "";
            for(auto el : um) {
                res += el;
            }
            return res;
        }
    }
};
```