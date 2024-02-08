
- #failed #medium #dynammic-programming 
- Tried 1 hour .

# 1. Problem Description
You are given an array `prices` where `prices[i]` is the price of a given stock on the `ith` day.

Find the maximum profit you can achieve. You may complete as many transactions as you like (i.e., buy one and sell one share of the stock multiple times) with the following restrictions:

- After you sell your stock, you cannot buy stock on the next day (i.e., cooldown one day).

**Note:** You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

**Example 1:**

**Input:** prices = [1,2,3,0,2]
**Output:** 3
**Explanation:** transactions = [buy, sell, cooldown, buy, sell]

**Example 2:**

**Input:** prices = [1]
**Output:** 0

**Constraints:**

- `1 <= prices.length <= 5000`
- `0 <= prices[i] <= 1000`

# 2. Intuition

My First idea was brute force and memoized to prune impossible branches.
As you can see in below image, we can get the sense of states on each day depending on the chocies of previous day.

For example, 
- If we bought a stock yesterday, we should cool down always
- If we cool downed yesterday, we can choose to buy stock or not.
- If we bought a stock yesterday, we can sell it or not.

Based on the above three possible choices, i drew a tree to visualize it.
However, this is why i failed to solve this problem because there are exponential possible cases whenever the depth of tree increments.
![](../../../../../Image-1%20(1).jpg)

Wait, Did i say 'states on each day depending on the chocies of previous day.' ?
If we are the current day 'i', we can choose what we need to do depending on what we did on the day 'i-1'.
By refering to few solutions, i quickly got to know we can divide up a unique solution into solutions of sub-problems.

Now we have three states and their transitions.
Then, we can set up some recursions depending on the current state while we take a possible max profit on the state.
![](../../../../../Image-1.jpg)

# 3. Solution


- failed case. (208 / 210)
```cpp
class Solution {
    int profit = 0;
public:
    void trading(const vector<int>& prices, /*index of prices*/int day, int state, int budget) {
        if(day == prices.size()-1) {
            if(state == 2) budget += prices[day]; // sell before end.
            profit = max(profit, budget); // update profit
            return;
        }

        if(state == 0) {
            if(prices[day] == 0) { // if cost zero, always buy
                trading(prices, day+1, state+2, budget);
            }
            else {
                trading(prices, day+1, state+2, budget-prices[day]); // buy
                trading(prices, day+1, state, budget); // don't buy
            }
        }
        else if(state == 1) {
            // need to cool down -> go to next day right away
            trading(prices, day+1, state-1, budget); // need to buy
        }
        else {
            // need to sell -> sell or stay
            trading(prices, day+1, state-1, budget+prices[day]); // sell
            trading(prices, day+1, state, budget); // doesn't sell
        }
    }

    int maxProfit(vector<int>& prices) {
        if(prices.size() == 1) return profit;

        trading(prices, 0, 0, 0);

        return profit;
    }
};
```

- 100% beats case
	- Time complexity : O(n)
	- Space complexity : O(n)
```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if(prices.size() == 1) return 0;
        
        // records
        vector<int> s0(prices.size(), 0);
        vector<int> s1(prices.size(), 0);
        vector<int> s2(prices.size(), 0);

        // base cases
        s0[0] = -prices[0];
        s1[0] = 0;
        s2[0] = INT_MIN;

        // recursion
        for(int i = 1; i < prices.size(); i++) {
            s0[i] = max(s0[i-1], s1[i-1]-prices[i]);
            s1[i] = max(s2[i-1], s1[i-1]);
            s2[i] = s0[i-1] + prices[i];
        }

        return max(s1[prices.size()-1], s2[prices.size()-1]);
    }
};
```