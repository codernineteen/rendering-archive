
- #solved #medium #binary-tree #graph
- took 16min to solve

# 1. Problem description

Given the `root` of a binary tree, the value of a target node `target`, and an integer `k`, return _an array of the values of all nodes that have a distance_ `k` _from the target node._

You can return the answer in **any order**.

**Example 1:**

![](https://s3-lc-upload.s3.amazonaws.com/uploads/2018/06/28/sketch0.png)

**Input:** root = [3,5,1,6,2,0,8,null,null,7,4], target = 5, k = 2
**Output:** [7,4,1]
Explanation: The nodes that are a distance 2 from the target node (with value 5) have values 7, 4, and 1.

**Example 2:**

**Input:** root = [1], target = 1, k = 3
**Output:** []

**Constraints:**

- The number of nodes in the tree is in the range `[1, 500]`.
- `0 <= Node.val <= 500`
- All the values `Node.val` are **unique**.
- `target` is the value of one of the nodes in the tree.
- `0 <= k <= 1000`

# 2. Intuition

In my opinion, the difficult part was to find nodes places in k-distance above current node, not in the subtrees of current node.

My idea is to convert this binary tree into bi-directional graph so that we can find nodes placed at k-distance from target without difficulties in traversing nodes.

1. Traverse binary tree and generate a graph by following below rule
	If there is a left child or right child, generate a bi-directional edge between the child and current root.
2. Create a queue and do BFS by keeping the visited nodes.
	A element in the queue is pair of node value and its distance from target
3. If a distance of current node is equal to k, push back it into result vector.

# 3. Solution

```cpp
class Solution {
public:
    void generateGraph(unordered_map<int, vector<int>>& graph, TreeNode* root) {
		if(root->left) {
			graph[root->left->val].push_back(root->val);
			graph[root->val].push_back(root->left->val);
			generateGraph(graph, root->left);
		}
		if(root->right) {
			graph[root->right->val].push_back(root->val);
			graph[root->val].push_back(root->right->val);
			generateGraph(graph, root->right);
		}
    }

    vector<int> distanceK(TreeNode* root, TreeNode* target, int k) {
        vector<int> res;

        unordered_map<int, vector<int>> graph;
        generateGraph(graph, root);

        vector<bool> visited(501, false);
        queue<pair<int, int>> nexts;
        nexts.push({target->val, 0});
        visited[target->val] = true;

        while(!nexts.empty()) {
            auto p = nexts.front();
            nexts.pop();

            if(p.second == k) {
                res.push_back(p.first);
                continue;
            }

            for(auto node : graph[p.first]) {
                if(visited[node] == false) {
                    visited[node] = true;
                    nexts.push({node, p.second+1});
                }
            }
        }

        return res;
    }
};
```