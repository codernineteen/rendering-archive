
- #solved #medium #implementation 

- Took 37 min to solve


# 1. Problem description
Implement the `RandomizedSet` class:

- `RandomizedSet()` Initializes the `RandomizedSet` object.
- `bool insert(int val)` Inserts an item `val` into the set if not present. Returns `true` if the item was not present, `false` otherwise.
- `bool remove(int val)` Removes an item `val` from the set if present. Returns `true` if the item was present, `false` otherwise.
- `int getRandom()` Returns a random element from the current set of elements (it's guaranteed that at least one element exists when this method is called). Each element must have the **same probability** of being returned.

You must implement the functions of the class such that each function works in **average** `O(1)` time complexity.

**Example 1:**

**Input**
["RandomizedSet", "insert", "remove", "insert", "getRandom", "remove", "insert", "getRandom"]
[[], [1], [2], [2], [], [1], [2], []]
**Output**
[null, true, false, true, 2, true, false, 2]

**Explanation**
RandomizedSet randomizedSet = new RandomizedSet();
randomizedSet.insert(1); // Inserts 1 to the set. Returns true as 1 was inserted successfully.
randomizedSet.remove(2); // Returns false as 2 does not exist in the set.
randomizedSet.insert(2); // Inserts 2 to the set, returns true. Set now contains [1,2].
randomizedSet.getRandom(); // getRandom() should return either 1 or 2 randomly.
randomizedSet.remove(1); // Removes 1 from the set, returns true. Set now contains [2].
randomizedSet.insert(2); // 2 was already in the set, so return false.
randomizedSet.getRandom(); // Since 2 is the only number in the set, getRandom() will always return 2.

**Constraints:**

- `-231 <= val <= 231 - 1`
- At most `2 *` `105` calls will be made to `insert`, `remove`, and `getRandom`.
- There will be **at least one** element in the data structure when `getRandom` is called.

# 2. Intuition

A key point to solve this problem is to implement 'getRandom' in an appropriate way under the constraint 'average O(1) runtime'.

First, we can't use vector because the possible range of value is between $-2^{31}$ and $2^{31}$, so it will definitely exceed memory constraints in this problem.

To implement insertion and removal in O(1) time complexity, we can choose set or hash map data structure to achieve it.

Then how we can get a unformly distributed random number from a set?
In this case, we can utilize a standary library 'random' in cpp language.
Whenever we want to get a random value from the set, we can generate a random distance between 0 and (size of set - 1).
Then we can get a random value by advancing iterator as much as generated random distance.

![](../../../../../images/Pasted%20image%2020240213154925.png)


# 3. Solution

```cpp
#include <random>
#include <iterator>

class RandomizedSet {
    std::mt19937 gen;
    unordered_set<int> elements;
public:

    RandomizedSet() : gen(1) {}
    bool insert(int val) {
        if(elements.find(val) != elements.end()) return false;
        else {
            elements.insert(val);
            return true;
        }
    }

    bool remove(int val) {
        if(elements.find(val) != elements.end()) {
            elements.erase(val);
            return true;
        }
        else return false;
	}

    int getRandom() {
        uniform_int_distribution<int> distribution(0, elements.size()-1);
        int distance = distribution(gen);
        auto it = elements.begin();
        std::advance (it, distance);
        int random_num = *it;
        return random_num;
    }
};
```

