
- #solved #medium #stack 
- Took 1h 20min to solve

# 1. Problem description

Given a string `s` which represents an expression, _evaluate this expression and return its value_. 

The integer division should truncate toward zero.

You may assume that the given expression is always valid. All intermediate results will be in the range of `[-231, 231 - 1]`.

**Note:** You are not allowed to use any built-in function which evaluates strings as mathematical expressions, such as `eval()`.

**Example 1:**

**Input:** s = "3+2*2"
**Output:** 7

**Example 2:**

**Input:** s = " 3/2 "
**Output:** 1

**Example 3:**

**Input:** s = " 3+5 / 2 "
**Output:** 5

**Constraints:**

- `1 <= s.length <= 3 * 105`
- `s` consists of integers and operators `('+', '-', '*', '/')` separated by some number of spaces.
- `s` represents **a valid expression**.
- All the integers in the expression are non-negative integers in the range `[0, 231 - 1]`.
- The answer is **guaranteed** to fit in a **32-bit integer**.

# 2. Intuition

The most important rule in arithemetic operations is priority of the operations. 
Because Multiplication and Division have higher priority in mathematical expression, we need to compute them to get a right result. (there is no parethesis , so we don't need to think about this case.)

If we look carefully the given examples, it seems there are spaces between charaters.
To parse given expression easily, we need to remove spaces first.
- First thing to do  : remove spaces

Then my idea is so simple :
- compute all adjacent sub expressions first against multiplication and division operation
- After shrink the expression by computing sub expressions, we can just compute the expression that only have addition and subtraction now in forward order.

But we still have an edge case can be happened.
What if there are a set of incoming numbers in a row ?
For example , we have an expression "30+5".
Then we iterate the given expression character by character.
After '3' , we are still on '0' which is not a operator.

For solving this problem, i determined to convert equation to a vector with int by separating all operands based on in-between operator.
For example,
```
"30+5" -> [30, -1 , 5]
```
where, negative numbers are corresponding to certain operator.
# 3. Solution

- erase spaces
```cpp
s.erase(remove_if(s.begin(), s.end(), ::isspace), s.end());
```

- parse equation
```cpp
int it = 0;
        vector<int> equation;
        while(it < s.length()) {
            int operand = 0;
            while(isInt(s[it])) {
                operand = operand * 10 + (s[it] - '0');
                it++;
            }

            equation.push_back(operand);
            switch(s[it]) {
                case '+':
                    equation.push_back(-1);
                    break;
                case '-':
                    equation.push_back(-2);
                    break;
                case '*':
                    equation.push_back(-3);
                    break;
                case '/':
                    equation.push_back(-4);
                    break;
            }
            it++;
        }
```

- compute expression
```cpp
it = 0;
        vector<int> computedEq;
        while(it < equation.size()) {
            int now = equation[it];
            it++;
            while(it < equation.size()-1 && (equation[it] == -3 || equation[it] == -4)) {
                if(equation[it] == -3) now *= equation[it+1];
                else now /= equation[it+1];
                it+=2; 
            }
            computedEq.push_back(now);
        }

        int res = computedEq[0];
        it = 1;
        while(it < computedEq.size()-1) {
            if(computedEq[it] == -1) res += computedEq[it+1];
            else res -= computedEq[it+1];
            it+=2;
        }
        return res;
```

- whole code
```cpp
class Solution {
public:
    inline bool isInt(char c) {
        return '0' <= c && c <= '9';
    }

    int calculate(string s) {
        // -1 : +
        // -2 : -
        // -3 : *
        // -4 : /
        s.erase(remove_if(s.begin(), s.end(), ::isspace), s.end());

        int it = 0;
        vector<int> equation;
        while(it < s.length()) {
            int operand = 0;
            while(isInt(s[it])) {
                operand = operand * 10 + (s[it] - '0');
                it++;
            }

            equation.push_back(operand);
            switch(s[it]) {
                case '+':
                    equation.push_back(-1);
                    break;
                case '-':
                    equation.push_back(-2);
                    break;
                case '*':
                    equation.push_back(-3);
                    break;
                case '/':
                    equation.push_back(-4);
                    break;
            }
            it++;
        }

        if(equation.size() == 1) return equation[0];

        it = 0;
        vector<int> computedEq;
        while(it < equation.size()) {
            int now = equation[it];
            it++;
            while(it < equation.size()-1 && (equation[it] == -3 || equation[it] == -4)) {
                if(equation[it] == -3) now *= equation[it+1];
                else now /= equation[it+1];
                it+=2; 
            }
            computedEq.push_back(now);
        }

        int res = computedEq[0];
        it = 1;
        while(it < computedEq.size()-1) {
            if(computedEq[it] == -1) res += computedEq[it+1];
            else res -= computedEq[it+1];
            it+=2;
        }
        return res;
    }
};
```

In my solution, i traversed three times of expression and there also additional memory usage which can be considere as inefficieny.

By refering to some solutions in leetcode, i got to know that we can solve this in a more clear way using 'stack' data structure.

```cpp
class Solution {
public:
    int calculate(string s) {
        stack<int> stk;
        char sign = '+';
        int res = 0;
        int temp = 0;

        for(int i=0; i < s.length(); i++)
        {
            while(i < s.length() && isdigit(s[i])) {
                temp = temp * 10 + (s[i] - '0');
                i++;
            }

            if(!isdigit(s[i]) && !isspace(s[i]) || i == s.size()-1) {
                if(sign == '-') stk.push(-temp);
                else if(sign == '+') stk.push(temp);
                else {
                    int num;
                    if(sign == '*') num = stk.top()*temp;
                    else num = stk.top()/temp;
                    stk.pop();
                    stk.push(num);
                }
                sign = s[i];
                temp = 0;
            }
        }

        while(!stk.empty()) {
            res += stk.top();
            stk.pop();
        }

        return res;
    }
};
```
