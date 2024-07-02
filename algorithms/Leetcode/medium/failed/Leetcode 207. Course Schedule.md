
- #failed #medium #topological-sort
- tried 1h 33min

# 1. Problem description

There are a total of `numCourses` courses you have to take, labeled from `0` to `numCourses - 1`. You are given an array `prerequisites` where `prerequisites[i] = [ai, bi]` indicates that you **must** take course `bi` first if you want to take course `ai`.

- For example, the pair `[0, 1]`, indicates that to take course `0` you have to first take course `1`.

Return `true` if you can finish all courses. Otherwise, return `false`.

**Example 1:**

**Input:** numCourses = 2, prerequisites = [[1,0]]
**Output:** true
**Explanation:** There are a total of 2 courses to take. 
To take course 1 you should have finished course 0. So it is possible.

**Example 2:**

**Input:** numCourses = 2, prerequisites = [[1,0],[0,1]]
**Output:** false
**Explanation:** There are a total of 2 courses to take. 
To take course 1 you should have finished course 0, and to take course 0 you should also have finished course 1. So it is impossible.

# 2. intuition

I quickly got to know that we can conver this problem into a problem to check whether there is a cycle inside a graph or not.
But what i failed to implement is to find a cycle. 
Because i could only check the latter adjacent parent of a certain node, i couldn't take the same process on another parents of current node. 
In other words, there is an edge case when a node's indegreee can be larger than 1.

Based on solution, i realized that we can solve this problem with 'topological sort'.
The steps of topological sort is :
1. Build a graph
2. Find nodes whose indegree is zero. They will be entry points.
3. Remove entry point and its adjacent nodes (We use a queue data structure for this in general)
	Also when we remove adjacent nodes, we should decrement the indegree of each of the nodes.
4. If a indegree of one of adjacent nodes become zero after decrement, we push it into a queue for next traversal.

# 3. Solution

- Topological sort solution

``` cpp
class Solution {
public:
    bool canFinish(int numCourses, vector<vector<int>>& pre) {
        vector<vector<int>> adj(numCourses, vector<int>());
        vector<int> indegree(numCourses, 0);

        for(auto req : pre) {
            adj[req[1]].push_back(req[0]);
            indegree[req[0]]++;
        }
        queue<int> q;
        for(int i = 0; i < numCourses; i++)
            if(indegree[i] == 0) q.push(i);
        while(!q.empty()) {
            int curr = q.front();
            q.pop();
            numCourses--;
            for(auto next : adj[curr])
                if(--indegree[next] == 0) q.push(next);
        }

        return numCourses == 0;
    }
};
```


- DFS approach
```cpp
class Solution {
public:
    bool canFinish(int numCourses, vector<pair<int, int>>& prerequisites) {
        graph g = buildGraph(numCourses, prerequisites);
        vector<bool> todo(numCourses, false), done(numCourses, false);
        for (int i = 0; i < numCourses; i++) {
            if (!done[i] && !acyclic(g, todo, done, i)) {
                return false;
            }
        }
        return true;
    }
private:
    typedef vector<vector<int>> graph;
    
    graph buildGraph(int numCourses, vector<pair<int, int>>& prerequisites) {
        graph g(numCourses);
        for (auto p : prerequisites) {
            g[p.second].push_back(p.first);
        }
        return g;
    }
    
    bool acyclic(graph& g, vector<bool>& todo, vector<bool>& done, int node) {
        if (todo[node]) {
            return false;
        }
        if (done[node]) {
            return true;
        }
        todo[node] = done[node] = true;
        for (int v : g[node]) {
            if (!acyclic(g, todo, done, v)) {
                return false;
            }
        }
        todo[node] = false;
        return true;
    }
};
```