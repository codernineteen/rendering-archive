
- #failed #medium #linked-list #implementation #hashamp
- tried 1h 54min

# 1. Problem description
Design a data structure that follows the constraints of a **[Least Recently Used (LRU) cache](https://en.wikipedia.org/wiki/Cache_replacement_policies#LRU)**.

Implement the `LRUCache` class:

- `LRUCache(int capacity)` Initialize the LRU cache with **positive** size `capacity`.
- `int get(int key)` Return the value of the `key` if the key exists, otherwise return `-1`.
- `void put(int key, int value)` Update the value of the `key` if the `key` exists. Otherwise, add the `key-value` pair to the cache. If the number of keys exceeds the `capacity` from this operation, **evict** the least recently used key.

The functions `get` and `put` must each run in `O(1)` average time complexity.

**Example 1:**

**Input**
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
**Output**
[null, null, null, 1, null, -1, null, -1, 3, 4]

**Explanation**
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // cache is {1=1}
lRUCache.put(2, 2); // cache is {1=1, 2=2}
lRUCache.get(1);    // return 1
lRUCache.put(3, 3); // LRU key was 2, evicts key 2, cache is {1=1, 3=3}
lRUCache.get(2);    // returns -1 (not found)
lRUCache.put(4, 4); // LRU key was 1, evicts key 1, cache is {4=4, 3=3}
lRUCache.get(1);    // return -1 (not found)
lRUCache.get(3);    // return 3
lRUCache.get(4);    // return 4

**Constraints:**

- `1 <= capacity <= 3000`
- `0 <= key <= 104`
- `0 <= value <= 105`
- At most `2 * 105` calls will be made to `get` and `put`.
# 2. intuition

Because we need to implement `get` and `put` method to work in $O(1)$ average time complextiy, we can think of some data structures that has efficient lookup time.

1. hashmap
	A hash map is key-value paired data strucure and it provided us theoritically constant time complexity of lookup data agains a certain key.

Because we also need to add a new pair into hashmap in O(1) time, we can consider Doubly Linked list data structure in this case.
By keeping the very first node as most recent node and the last node as the oldest node in cache, we can implement addition or removal node from LRU cache in constant time.

# 3. Solution
```cpp
class KeyVal {
    public :
        int key;
        int val;
        KeyVal(int key, int val) : key(key), val(val) {}
};

  

class LRUCache {
    unordered_map<int, list<KeyVal>::iterator> um;
    list<KeyVal> dll;
    int cap;

public:
    LRUCache(int capacity) : cap(capacity) {}
    
    int get(int key) {
        auto it = um.find(key);
        if(it != um.end()) {
            auto ret = *(it->second);
            dll.erase(it->second);
            dll.push_front(ret);
            um[ret.key] = dll.begin();
            return ret.val;
        }
        else return -1;
    }
    
    void put(int key, int value) {
        auto it = um.find(key);
        if(it != um.end()) {
            auto ret = *(it->second);
            ret.val = value;
            dll.erase(it->second);
            dll.push_front(ret);
            um[ret.key] = dll.begin();
        }
        else {
            if(cap > 0) cap--;
            else {
                auto lru = prev(dll.end());
                int key = lru->key;
                um.erase(key);
                dll.erase(lru);
            }

            KeyVal newPair(key, value);
            dll.push_front(newPair);
            um[key] = dll.begin();
        }
    }

};
```