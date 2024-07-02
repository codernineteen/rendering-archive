- #solved #medium #bits #integer
- took 10min to solve

# Intuition

- convert given integer to string
- While reversing the converted string, follow the equation
	`res = res * 10 + (character - '0')`
- If `res > INT_MAX/10` , it means overflow will be happened
	- return 0
- If last character is a sign, not a number
	- multiply -1 to res.

# Solution

```cpp
class Solution {

public:

    int reverse(int x) {

        auto x_str = to_string(x);

        int res = 0;

        for(int i = x_str.length()-1; i > -1; i--) {

            if(i == x_str.length()-1 && x_str[i] == '0') continue;

            if(i == 0 && x_str[i] == '-') {

                res *= -1;

                continue;

            }

            if(res > INT_MAX/10) return 0;

            res *= 10;

            res += x_str[i] - '0';

        }

  

        return res;

    }

};
```