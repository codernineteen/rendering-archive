
- #failed #medium #array #two-pointer 
- troied 53 min to solve.

# 1. Problem description
Given an array `nums` of `n` integers, return _an array of all the **unique** quadruplets_ `[nums[a], nums[b], nums[c], nums[d]]` such that:

- `0 <= a, b, c, d < n`
- `a`, `b`, `c`, and `d` are **distinct**.
- `nums[a] + nums[b] + nums[c] + nums[d] == target`

You may return the answer in **any order**.

**Example 1:**

**Input:** nums = [1,0,-1,0,-2,2], target = 0
**Output:** [[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]]

**Example 2:**

**Input:** nums = [2,2,2,2,2], target = 8
**Output:** [[2,2,2,2]]

**Constraints:**

- `1 <= nums.length <= 200`
- `-109 <= nums[i] <= 109`
- `-109 <= target <= 109`

# 2. Solution

I failed to solve this problem because i forgot how to implement the solution of '3Sum' problem.
The solutions of 'N-sum' problems are similar to each other.
In 4Sum problem, what we need to do is just setting a pivot and leaning on the solution of 3Sum.
Hence, we can generalize Nsum problem as recursive way.

Let's see the solution of '4sum' first.
```cpp
class Solution {
    vector<vector<int>> res;
public:
    vector<vector<int>> threeSum(vector<int>& nums, long long int target) {
        vector<vector<int>> res;

        for(int i = 0; i < nums.size(); i++) {
            long long int targetSum = target-nums[i];
            int l = i+1;
            int r = nums.size()-1;
            while(l < r) {
                long long int sum = nums[l] + nums[r];

                if(sum < targetSum) {
                    l++;
                }
                else if(sum > targetSum) {
                    r--;
                }
                else {
                    vector<int> triplet = {nums[i], nums[l], nums[r]};
                    res.push_back(triplet);

                    while(l < r && nums[l] == triplet[1]) l++;
                    while(l < r && nums[r] == triplet[2]) r--;
                }
            }

            while(i+1 < nums.size() && nums[i+1] == nums[i]) i++;
        }

        return res;
    }

    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        if(nums.size() < 4) return res;

        // sort ascending order. 
        sort(nums.begin(), nums.end());

        int pivot;
        for(pivot = 0; pivot < nums.size(); pivot++) {
            vector<int> sub_vec(nums.begin()+pivot+1, nums.end());
            auto temp = threeSum(sub_vec, target-nums[pivot]);

            for(auto t : temp) {
                t.push_back(nums[pivot]);
                res.push_back(t);
            }

            while(pivot+1 < nums.size() && nums[pivot] == nums[pivot+1]) pivot++;
        }

        return res;
    }
};
```

To generalize this problem, we can separate the solution into two parts.

First,
With pivot current location 'i', find combinations with (n-1) elements whose sum satisfy (target - num of i) and decrement 'N'
For example, 
- current location : 3rd element (5)
- target sum : 10
- we need to find combinations of two numbers whose sum is equal to (10 - 5) in range between i+1 and last element in vector.

Second, If N is two , Find all the possible pairs and return the pairs

If we follow above steps, we will get the pairs at phase of N == 2.
Then The function returns the pairs , the caller of the function will get the returned vector and it appened current number to the vector.

- code
```cpp

vector<vector<int>> NSum(vector<int>& nums, int pivot, int left, int right, int N, long long int target) {
	vector<vector<int>> res;

	if(N == 2) {
		while(left < right) {
			long long int sum = nums[left] + nums[right];

			if(sum < target) left++;
			else if(sum > target) right--;
			else {
				vector<int> comb = {nums[left], nums[right]};
				res.push_back(comb);

				while(left < right && nums[left] == comb[0]) left++;
				while(left < right && nums[right] == comb[1]) right--;
			}
		}
	}
	else if(N > 2) {
		for(int i = pivot; i < nums.size(); i++) {
			long long int targetSum = target-nums[i];
	
			vector<vector<int>> combs = NSum(nums, i+1, i+1, right, N-1, targetSum);
			for(auto comb : combs) {
				comb.push_back(nums[i]);
				res.push_back(comb);
			}

			while(i+1 < nums.size() && nums[i+1] == nums[i]) i++;
		}
	}

	return res;
}

vector<vector<int>> fourSum(vector<int>& nums, int target) {
	if(nums.size() < 4) return res;

	// sort ascending order. 
	sort(nums.begin(), nums.end());

	res = NSum(nums, 0, 1, nums.size()-1, 4, target);

	return res;
}
```