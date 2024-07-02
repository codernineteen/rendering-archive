
- #solved #medium #divide-and-conquer
- took 50min to solve.

# 1. Problem description

Given a `n * n` matrix `grid` of `0's` and `1's` only. We want to represent `grid` with a Quad-Tree.

Return _the root of the Quad-Tree representing_ `grid`.

A Quad-Tree is a tree data structure in which each internal node has exactly four children. Besides, each node has two attributes:

- `val`: True if the node represents a grid of 1's or False if the node represents a grid of 0's. Notice that you can assign the `val` to True or False when `isLeaf` is False, and both are accepted in the answer.
- `isLeaf`: True if the node is a leaf node on the tree or False if the node has four children.

class Node {
    public boolean val;
    public boolean isLeaf;
    public Node topLeft;
    public Node topRight;
    public Node bottomLeft;
    public Node bottomRight;
}

We can construct a Quad-Tree from a two-dimensional area using the following steps:

1. If the current grid has the same value (i.e all `1's` or all `0's`) set `isLeaf` True and set `val` to the value of the grid and set the four children to Null and stop.
2. If the current grid has different values, set `isLeaf` to False and set `val` to any value and divide the current grid into four sub-grids as shown in the photo.
3. Recurse for each of the children with the proper sub-grid.

![](https://assets.leetcode.com/uploads/2020/02/11/new_top.png)

If you want to know more about the Quad-Tree, you can refer to the [wiki](https://en.wikipedia.org/wiki/Quadtree).

ex)
![](../../../../../images/Pasted%20image%2020240226142926.png)

# Intuition

This is one of typical examples of divide & conquer algorithm.

The basic algorithm follows :
if a search range is not pure (it means that there are two different elements in grid.),
we need to cosntruct a quad tree of the grid.
If not, we can return itself.

When we need to construct a quad tree, we only need to care the direct children of root by using recursive function because the children in deeper depth can be handled in direct children as sub-roots.

# Solution
- time complexity : construct a quad is $2^x \times 2^ x = (2^2)^\alpha, \alpha=x$ . Hence, $O(x)$ and every time the different value is placed at last element it requires $O(len^2)$ 
	- Eventually $O(len^2 \times x)$
- space complexity : $O(x)$ 
![](../../../../../images/Pasted%20image%2020240226143324.png)
```cpp
/*
// Definition for a QuadTree node.
class Node {
public:
    bool val;
    bool isLeaf;
    Node* topLeft;
    Node* topRight;
    Node* bottomLeft;
    Node* bottomRight;
    
    Node() {
        val = false;
        isLeaf = false;
        topLeft = NULL;
        topRight = NULL;
        bottomLeft = NULL;
        bottomRight = NULL;
    }
    
    Node(bool _val, bool _isLeaf) {
        val = _val;
        isLeaf = _isLeaf;
        topLeft = NULL;
        topRight = NULL;
        bottomLeft = NULL;
        bottomRight = NULL;
    }
    
    Node(bool _val, bool _isLeaf, Node* _topLeft, Node* _topRight, Node* _bottomLeft, Node* _bottomRight) {
        val = _val;
        isLeaf = _isLeaf;
        topLeft = _topLeft;
        topRight = _topRight;
        bottomLeft = _bottomLeft;
        bottomRight = _bottomRight;
    }
};
*/

class Solution {
public:
    inline bool isPure(vector<vector<int>>& grid, int sRow, int sCol, int len) {
        int prev = grid[sRow][sCol];
        for(int row = sRow, rowIt = 0; rowIt < len; row++, rowIt++) {
            for(int col = sCol, colIt = 0; colIt < len; col++, colIt++) {
                if(prev != grid[row][col]) return false;
                prev = grid[row][col];
            }
        }

        return true;
    }

    Node* solve(vector<vector<int>>& grid, int sRow, int sCol, int len) {
        auto root = new Node(grid[sRow][sCol], true);

        if(len != 1 && !isPure(grid, sRow, sCol, len)) {
            int subLen = len/2;
            root->isLeaf = false;
            root->val = 0;
            root->topLeft = solve(grid, sRow, sCol, subLen);
            root->topRight = solve(grid, sRow, sCol + subLen, subLen);
            root->bottomLeft = solve(grid, sRow + subLen, sCol, subLen);
            root->bottomRight = solve(grid, sRow + subLen, sCol + subLen, subLen);
        }

        return root;
    }

    Node* construct(vector<vector<int>>& grid) {
        return solve(grid, 0, 0, grid.size());
    }
};
```