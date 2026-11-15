# 2065. Maximum Path Quality of a Graph

> Source: https://algo.monster/liteproblems/2065

## Problem Statement

You have an undirected [graph](/problems/graph_intro) with `n` nodes numbered from `0` to `n - 1`. Each node has a value given by `values[i]` for node `i`. The graph's edges are defined by a 2D array `edges`, where each element `edges[j] = [u_j, v_j, time_j]` represents an undirected edge between nodes `u_j` and `v_j` that takes `time_j` seconds to traverse.

Your task is to find a valid path that:
- Starts at node `0`
- Ends at node `0` (returns to the starting point)
- Takes at most `maxTime` seconds to complete
- Can visit the same node multiple times

The quality of a path is calculated as the sum of values of all unique nodes visited during the path. This means if you visit a node multiple times, its value is only counted once in the quality calculation.

You need to return the maximum possible quality among all valid paths.

**Key constraints:**
- Each node has at most 4 edges connected to it
- You must start and end at node `0`
- The total time of the path cannot exceed `maxTime`
- Node values are only counted once in the quality sum, regardless of how many times you visit them

**Example:**
If you have a path that visits nodes `0 → 1 → 2 → 1 → 0`, and the values are `[5, 10, 15, ...]`, the quality would be `5 + 10 + 15 = 30` (node 1's value is only counted once despite being visited twice).

## Intuition

The key insight comes from examining the constraints. With at most 4 edges per node and a maximum time limit, we realize the search space is actually quite small. Even in the worst case where each edge takes minimum time (10 seconds), we can only traverse about 10 edges within `maxTime = 100`. This means our paths won't be very long.

Since we need to find the maximum quality among all valid paths from node 0 back to node 0, and the search space is manageable, we can afford to explore every possible path. This naturally leads us to a [backtracking](/problems/backtracking) approach.

The tricky part is handling the quality calculation - we want to count each node's value only once, even if we visit it multiple times. This suggests we need to track which nodes we've visited in our current path. When we visit a node for the first time, we add its value to our quality sum. If we visit it again, we don't add its value again.

Here's the thought process:
1. Start at node 0 with its value already counted
2. For each neighbor, check if we have enough time to visit it
3. If we visit a new node, mark it as visited and add its value
4. If we revisit a node, don't add its value again
5. When we return to node 0, we have a complete path - update our answer if this path has better quality
6. Backtrack by unmarking nodes we visited (so other paths can count them fresh)

The beauty of this approach is that the [backtracking](/problems/backtracking) naturally handles the exploration of all paths, while the visited array ensures we correctly calculate the quality by counting each node's value at most once per path. The DFS recursion stack keeps track of our current path, and we only need to maintain the cumulative time and quality as we go.

**Learn more about [Graph](/problems/graph_intro) and [Backtracking](/problems/backtracking) patterns.**

## Solution Approach

The solution uses DFS with [backtracking](/problems/backtracking) to explore all valid paths. Let's walk through the implementation step by step:

**1. [Graph](/problems/graph_intro) Representation**
First, we build an adjacency list `g` to represent the graph. For each edge `[u, v, t]`, we add `(v, t)` to `g[u]` and `(u, t)` to `g[v]` since the graph is undirected.

```python
g = [[] for _ in range(n)]
for u, v, t in edges:
    g[u].append((v, t))
    g[v].append((u, t))
```

**2. Visited Tracking**
We maintain a boolean array `vis` of length `n` to track which nodes are currently in our path. Initially, we mark node 0 as visited since we start from there.

```python
vis = [False] * n
vis[0] = True
```

**3. DFS Function**
The core logic is in the `dfs(u, cost, value)` function where:
- `u`: current node we're at
- `cost`: total time spent so far
- `value`: total quality (sum of unique node values) collected so far

**4. Base Case**
When we reach node 0 (which means we've completed a cycle back to start), we update our answer:
```python
if u == 0:
    ans = max(ans, value)
```

**5. Exploring Neighbors**
For each neighbor `v` of current node `u` with edge time `t`:
- First check if we have enough time: `cost + t <= maxTime`
- If `vis[v]` is `True` (node already visited in current path):
  - Recursively call `dfs(v, cost + t, value)` without adding the node's value again
- If `vis[v]` is `False` (first time visiting this node):
  - Mark it as visited: `vis[v] = True`
  - Recursively call with added value: `dfs(v, cost + t, value + values[v])`
  - Backtrack by unmarking: `vis[v] = False`

**6. Starting the Search**
We initiate the DFS from node 0 with:
- Starting cost: 0
- Starting value: `values[0]` (node 0's value)

The algorithm cleverly handles the requirement that each node's value should be counted at most once per path. By maintaining the `vis` array and only adding a node's value when we first visit it in the current path, we ensure correct quality calculation. The [backtracking](/problems/backtracking) (setting `vis[v] = False`) allows other paths to count that node's value independently.

The time complexity is bounded by the small search space - with at most 4 edges per node and limited path length, the algorithm efficiently explores all possibilities despite being exponential in nature.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example Graph:**
- Nodes: 4 nodes (0, 1, 2, 3)
- Values: [10, 5, 15, 20]
- Edges: [[0,1,10], [0,2,15], [1,3,20], [2,3,10]]
- maxTime: 30

**Graph Visualization:**
```
    0(10)---10---1(5)
      |           |
     15          20
      |           |
    2(15)---10---3(20)
```

**DFS Exploration:**

1. **Start at node 0**
   - vis = [True, False, False, False]
   - cost = 0, value = 10
   - Check neighbors: nodes 1 and 2

2. **Path: 0 → 1**
   - cost = 10, value = 10 + 5 = 15
   - vis = [True, True, False, False]
   - From node 1, can go to: node 0 (cost=20) or node 3 (cost=30)
 
3. **Path: 0 → 1 → 0**
   - Back at node 0, cost = 20, value = 15
   - Update answer: ans = 15
   - Continue exploring from node 1...

4. **Path: 0 → 1 → 3**
   - cost = 30, value = 15 + 20 = 35
   - vis = [True, True, False, True]
   - From node 3, time budget exhausted (any move would exceed 30)
   - Backtrack: vis[3] = False

5. **Backtrack to node 0, explore second neighbor**
   - vis = [True, False, False, False]
   - Path: 0 → 2
   - cost = 15, value = 10 + 15 = 25
   - vis = [True, False, True, False]

6. **Path: 0 → 2 → 3**
   - cost = 25, value = 25 + 20 = 45
   - vis = [True, False, True, True]
   - From node 3, can only afford to go back to node 2 (cost=35 > 30 for node 1)
   - Backtrack: vis[3] = False

7. **Path: 0 → 2 → 0**
   - Back at node 0, cost = 30, value = 25
   - Update answer: ans = max(15, 25) = 25

**Key Observations:**
- When we visit node 1 twice in path 0→1→0→1, we don't add value[1] again
- The vis array tracks nodes in current path, preventing double-counting
- Backtracking (vis[v] = False) allows other paths to count that node's value
- We only update the answer when we return to node 0
- The algorithm explores all feasible paths within the time limit

**Final Answer:** 25 (from path 0→2→0 with nodes {0, 2} visited)

## Python Solution

```python
class Solution:
    def maximalPathQuality(
        self, values: List[int], edges: List[List[int]], maxTime: int
    ) -> int:
        def dfs(current_node: int, time_spent: int, current_value: int) -> None:
            """
            Depth-first search to explore all possible paths.
          
            Args:
                current_node: Current node in the traversal
                time_spent: Total time spent so far in the path
                current_value: Total value collected so far in the path
            """
            # If we're back at node 0, update the maximum answer
            if current_node == 0:
                nonlocal max_quality
                max_quality = max(max_quality, current_value)
          
            # Explore all neighboring nodes
            for neighbor, travel_time in adjacency_list[current_node]:
                # Only proceed if we have enough time remaining
                if time_spent + travel_time <= maxTime:
                    if visited[neighbor]:
                        # If neighbor is already visited, don't add its value again
                        dfs(neighbor, time_spent + travel_time, current_value)
                    else:
                        # Mark as visited, add value, explore, then backtrack
                        visited[neighbor] = True
                        dfs(neighbor, time_spent + travel_time, current_value + values[neighbor])
                        visited[neighbor] = False  # Backtrack: unmark visited

        # Initialize graph data structures
        num_nodes = len(values)
        adjacency_list = [[] for _ in range(num_nodes)]
      
        # Build adjacency list representation of the graph
        for node_u, node_v, travel_time in edges:
            adjacency_list[node_u].append((node_v, travel_time))
            adjacency_list[node_v].append((node_u, travel_time))
      
        # Track visited nodes to avoid counting their values multiple times
        visited = [False] * num_nodes
        visited[0] = True  # Start from node 0
      
        # Initialize the maximum quality answer
        max_quality = 0
      
        # Start DFS from node 0 with initial time 0 and initial value of node 0
        dfs(0, 0, values[0])
      
        return max_quality
```

## Time and Space Complexity

**Time Complexity:** `O(n + m + 4^(maxTime/min(time_j)))`

The time complexity consists of:
- `O(n + m)` for building the adjacency list from the edges
- The DFS exploration complexity, which in the worst case can be `O(4^(maxTime/min(time_j)))`

The exponential part arises because:
- From each node, we can potentially explore multiple paths (up to degree of the node)
- We can revisit nodes multiple times as long as the time constraint allows
- The maximum depth of recursion is bounded by `maxTime/min(time_j)` where `min(time_j)` is the minimum edge weight
- At each level, we might branch to multiple neighbors, and since we can revisit nodes, the branching factor can be up to 4 in dense graphs (considering bidirectional edges)

**Space Complexity:** `O(n + m + maxTime/min(time_j))`

The space complexity consists of:
- `O(n + m)` for storing the graph adjacency list
- `O(n)` for the visited array
- `O(maxTime/min(time_j))` for the recursion stack depth in the worst case, which represents the maximum number of edges we can traverse within the time limit

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrectly Handling the Starting Node's Value

**The Problem:**
A common mistake is forgetting to include node 0's value in the initial quality calculation or accidentally double-counting it. Some implementations might start with `current_value = 0` instead of `current_value = values[0]`, which would miss counting the starting node's value entirely.

**Why It Happens:**
Since we mark `visited[0] = True` before starting the DFS, and we only add values for unvisited nodes during traversal, node 0's value would never be added if we don't initialize it properly.

**The Fix:**
Always initialize the DFS with `values[0]` as the starting quality:
```python
# Correct initialization
dfs(0, 0, values[0])  # Start with node 0's value included

# Incorrect initialization that misses node 0's value
# dfs(0, 0, 0)  # This would miss counting values[0]
```

### Pitfall 2: Not Properly Backtracking the Visited State

**The Problem:**
Forgetting to reset `visited[neighbor] = False` after exploring a path leads to incorrect results. Without proper backtracking, once a node is visited in any path, it would remain marked as visited for all subsequent path explorations, preventing other valid paths from counting that node's value.

**Why It Happens:**
The backtracking step can be easily overlooked, especially when focusing on the forward exploration logic. The algorithm explores multiple paths that might share common nodes, and each path should independently track which nodes it has visited.

**The Fix:**
Always reset the visited state after exploring a branch:
```python
if not visited[neighbor]:
    visited[neighbor] = True
    dfs(neighbor, time_spent + travel_time, current_value + values[neighbor])
    visited[neighbor] = False  # Critical: Must backtrack!
```

**Example of the Bug:**
Consider a graph where node 0 connects to nodes 1 and 2, and both 1 and 2 connect back to 0. Without proper backtracking:
- Path 1: `0 → 1 → 0` would mark node 1 as visited
- Path 2: `0 → 2 → 1 → 0` would not be able to count node 1's value because it's still marked as visited from Path 1

This would result in a lower quality score than the correct implementation.

