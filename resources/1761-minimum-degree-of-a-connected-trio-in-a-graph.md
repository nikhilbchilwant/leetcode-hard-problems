# 1761. Minimum Degree of a Connected Trio in a Graph

> Source: https://algo.monster/liteproblems/1761

## Problem Statement

You have an undirected [graph](/problems/graph_intro) with `n` nodes (numbered from 1 to n) and a list of edges connecting pairs of nodes. Your task is to find the minimum degree of a "connected trio" in this graph.

A **connected trio** is a set of exactly three nodes where every pair of nodes in the set is directly connected by an edge. In other words, these three nodes form a triangle in the [graph](/problems/graph_intro).

The **degree of a connected trio** is the total number of edges that connect any node in the trio to nodes outside the trio. This counts all edges that have one endpoint inside the trio and the other endpoint outside of it.

For example, if nodes A, B, and C form a connected trio:
- Count all edges from A to nodes other than B and C
- Count all edges from B to nodes other than A and C  
- Count all edges from C to nodes other than A and B
- The sum of these counts is the degree of this trio

Your goal is to:
1. Find all connected trios in the [graph](/problems/graph_intro)
2. Calculate the degree of each trio
3. Return the minimum degree among all trios
4. If no connected trio exists in the graph, return `-1`

The input consists of:
- `n`: the number of nodes in the [graph](/problems/graph_intro)
- `edges`: an array where each element `[u, v]` represents an undirected edge between nodes `u` and `v`

## Intuition

To find connected trios, we need to identify sets of three nodes where each node is connected to the other two. The straightforward way to do this is to check all possible combinations of three nodes and verify if they form a triangle.

For efficient edge checking, we can use an adjacency matrix `g[i][j]` that tells us in constant time whether nodes `i` and `j` are connected. This is faster than repeatedly searching through the edge list.

When calculating the degree of a trio, we need to count edges going outside the trio. If we have nodes `i`, `j`, and `k` forming a trio, we need to count:
- Edges from `i` to nodes other than `j` and `k`
- Edges from `j` to nodes other than `i` and `k`  
- Edges from `k` to nodes other than `i` and `j`

A key observation here is that we can use the total degree of each node and subtract the internal connections. Each node in the trio has exactly 2 edges connecting to the other nodes in the trio. So if `deg[i]` is the total degree of node `i`, then `deg[i] - 2` gives us the edges from `i` going outside the trio.

Therefore, for a trio `(i, j, k)`, the total degree would be:
- `(deg[i] - 2) + (deg[j] - 2) + (deg[k] - 2)`
- Which simplifies to: `deg[i] + deg[j] + deg[k] - 6`

This formula allows us to quickly calculate the trio degree without having to iterate through all edges again. We simply enumerate all possible triplets, check if they form a connected trio using our adjacency matrix, and if they do, calculate their degree using this formula and track the minimum.

**Learn more about [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The implementation follows a brute force enumeration strategy with optimized data structures for efficient lookups:

**1. Initialize Data Structures:**
- Create an adjacency matrix `g` of size `n × n` initialized with `False` values. This allows O(1) edge existence checks.
- Create a degree array `deg` of size `n` to store the degree of each node.

**2. Build the [Graph](/problems/graph_intro) Representation:**
- For each edge `[u, v]` in the input:
  - Convert to 0-indexed: `u = u - 1`, `v = v - 1` (since nodes are numbered 1 to n)
  - Mark `g[u][v] = g[v][u] = True` to indicate the undirected edge
  - Increment the degree counters: `deg[u] += 1` and `deg[v] += 1`

**3. Find Connected Trios:**
- Initialize answer `ans = inf` to track the minimum degree
- Use three nested loops to enumerate all possible triplets `(i, j, k)` where `i < j < k`:
  - First loop: `i` from `0` to `n-1`
  - Second loop: `j` from `i+1` to `n-1`, and check if `g[i][j]` is true
  - Third loop: `k` from `j+1` to `n-1`
  - For each triplet, check if all three edges exist:
    - If `g[i][j]` and `g[i][k]` and `g[j][k]` are all `True`, we have a connected trio

**4. Calculate Trio Degree:**
- When a connected trio `(i, j, k)` is found:
  - Calculate its degree as `deg[i] + deg[j] + deg[k] - 6`
  - The `-6` accounts for the 6 internal edge endpoints (each of the 3 edges has 2 endpoints)
  - Update `ans = min(ans, deg[i] + deg[j] + deg[k] - 6)`

**5. Return Result:**
- After checking all possible triplets:
  - If `ans` is still `inf`, no connected trio was found, return `-1`
  - Otherwise, return `ans` as the minimum trio degree

The time complexity is O(n³) for checking all triplets, and space complexity is O(n²) for the adjacency matrix.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example Graph:**
- n = 5 nodes (numbered 1 to 5)
- edges = [[1,2], [1,3], [2,3], [2,4], [3,4], [3,5], [4,5]]

**Step 1: Build Graph Representation**

First, we create our adjacency matrix `g` (5×5) and degree array `deg`:
- Process edge [1,2]: Set g[0][1] = g[1][0] = True, deg[0]++ = 2, deg[1]++ = 2
- Process edge [1,3]: Set g[0][2] = g[2][0] = True, deg[0]++ = 2, deg[2]++ = 2
- Process edge [2,3]: Set g[1][2] = g[2][1] = True, deg[1]++ = 3, deg[2]++ = 3
- Process edge [2,4]: Set g[1][3] = g[3][1] = True, deg[1]++ = 3, deg[3]++ = 2
- Process edge [3,4]: Set g[2][3] = g[3][2] = True, deg[2]++ = 4, deg[3]++ = 3
- Process edge [3,5]: Set g[2][4] = g[4][2] = True, deg[2]++ = 4, deg[4]++ = 2
- Process edge [4,5]: Set g[3][4] = g[4][3] = True, deg[3]++ = 3, deg[4]++ = 2

Final degrees: deg = [2, 3, 4, 3, 2] (for nodes 1-5 respectively)

**Step 2: Find Connected Trios**

We enumerate all triplets (i, j, k) where i < j < k:

- Triplet (0, 1, 2) → Nodes (1, 2, 3):
  - Check: g[0][1] ✓, g[0][2] ✓, g[1][2] ✓ → This is a connected trio!
  - Calculate degree: deg[0] + deg[1] + deg[2] - 6 = 2 + 3 + 4 - 6 = 3
  - Update ans = 3

- Triplet (0, 1, 3) → Nodes (1, 2, 4):
  - Check: g[0][1] ✓, g[0][3] ✗ → Not a connected trio

- Triplet (0, 1, 4) → Nodes (1, 2, 5):
  - Check: g[0][1] ✓, g[0][4] ✗ → Not a connected trio

- Triplet (1, 2, 3) → Nodes (2, 3, 4):
  - Check: g[1][2] ✓, g[1][3] ✓, g[2][3] ✓ → This is a connected trio!
  - Calculate degree: deg[1] + deg[2] + deg[3] - 6 = 3 + 4 + 3 - 6 = 4
  - ans remains 3 (since 3 < 4)

- Triplet (2, 3, 4) → Nodes (3, 4, 5):
  - Check: g[2][3] ✓, g[2][4] ✓, g[3][4] ✓ → This is a connected trio!
  - Calculate degree: deg[2] + deg[3] + deg[4] - 6 = 4 + 3 + 2 - 6 = 3
  - ans remains 3 (since 3 = 3)

**Step 3: Return Result**

We found three connected trios:
- Trio (1,2,3) with degree 3
- Trio (2,3,4) with degree 4  
- Trio (3,4,5) with degree 3

The minimum degree is 3, so we return 3.

**Verification of Trio (1,2,3):**
- Node 1 has edges to: 2 (in trio), 3 (in trio) → 0 external edges
- Node 2 has edges to: 1 (in trio), 3 (in trio), 4 (external) → 1 external edge
- Node 3 has edges to: 1 (in trio), 2 (in trio), 4 (external), 5 (external) → 2 external edges
- Total external edges = 0 + 1 + 2 = 3 ✓

## Python Solution

```python
from typing import List
from math import inf


class Solution:
    def minTrioDegree(self, n: int, edges: List[List[int]]) -> int:
        # Create adjacency matrix to track connections between nodes
        # graph[i][j] = True if there's an edge between node i and j
        graph = [[False] * n for _ in range(n)]
      
        # Track the degree (number of connections) for each node
        degree = [0] * n
      
        # Build the graph from edges (convert to 0-indexed)
        for u, v in edges:
            u, v = u - 1, v - 1  # Convert from 1-indexed to 0-indexed
            graph[u][v] = graph[v][u] = True  # Undirected graph
            degree[u] += 1
            degree[v] += 1
      
        # Initialize minimum trio degree to infinity
        min_degree = inf
      
        # Check all possible trios (i, j, k) where i < j < k
        for i in range(n):
            for j in range(i + 1, n):
                if graph[i][j]:  # If i and j are connected
                    for k in range(j + 1, n):
                        # Check if k forms a trio with i and j
                        if graph[i][k] and graph[j][k]:
                            # Calculate trio degree: sum of degrees minus internal edges
                            # Each trio has 3 internal edges, counted twice in degree sum
                            trio_degree = degree[i] + degree[j] + degree[k] - 6
                            min_degree = min(min_degree, trio_degree)
      
        # Return -1 if no trio found, otherwise return minimum trio degree
        return -1 if min_degree == inf else min_degree
```

## Time and Space Complexity

**Time Complexity:** `O(n³)`

The algorithm uses three nested loops to find all possible trios (groups of three connected nodes):
- The outermost loop iterates through all nodes `i` from `0` to `n-1`: `O(n)` iterations
- The second loop iterates through nodes `j` from `i+1` to `n-1`: up to `O(n)` iterations  
- The innermost loop iterates through nodes `k` from `j+1` to `n-1`: up to `O(n)` iterations

For each trio `(i, j, k)`, the algorithm performs constant time operations:
- Checking if edges exist: `g[i][j]`, `g[i][k]`, `g[j][k]` - each is `O(1)`
- Computing the degree sum and minimum - `O(1)`

Additionally, the preprocessing step builds the adjacency matrix and degree array by iterating through all edges once, which takes `O(E)` time where `E` is the number of edges. Since `E ≤ n²`, this doesn't affect the overall complexity.

Therefore, the total time complexity is `O(n) × O(n) × O(n) = O(n³)`.

**Space Complexity:** `O(n²)`

The algorithm uses:
- A 2D adjacency matrix `g` of size `n × n` to store edge information: `O(n²)` space
- A degree array `deg` of size `n`: `O(n)` space
- A few scalar variables (`ans`, loop indices): `O(1)` space

The dominant term is the adjacency matrix, giving us a total space complexity of `O(n²)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Index Conversion**
One of the most common mistakes is forgetting to convert between 1-indexed nodes (in the problem) and 0-indexed arrays (in the implementation). This can lead to:
- Array index out of bounds errors
- Incorrect graph construction
- Missing or extra edges

**Example of the pitfall:**
```python
# WRONG: Forgetting to convert indices
for u, v in edges:
    graph[u][v] = graph[v][u] = True  # This will cause index errors!
    degree[u] += 1
    degree[v] += 1
```

**Solution:**
Always convert from 1-indexed to 0-indexed immediately when processing edges:
```python
# CORRECT: Convert to 0-indexed
for u, v in edges:
    u, v = u - 1, v - 1  # Convert first
    graph[u][v] = graph[v][u] = True
    degree[u] += 1
    degree[v] += 1
```

### 2. **Incorrect Trio Degree Calculation**
Another frequent error is miscalculating the trio degree by not properly accounting for internal edges. The trio degree should only count edges going outside the trio.

**Example of the pitfall:**
```python
# WRONG: Not subtracting internal edges
trio_degree = degree[i] + degree[j] + degree[k]  # This counts internal edges too!
```

**Why -6 is needed:**
- Each node in the trio connects to the other 2 nodes (internal edges)
- Total internal connections: 3 nodes × 2 connections each = 6
- These 6 connections should not count toward the trio degree

**Solution:**
```python
# CORRECT: Subtract the 6 internal edge endpoints
trio_degree = degree[i] + degree[j] + degree[k] - 6
```

### 3. **Inefficient Edge Checking**
Using an adjacency list instead of an adjacency matrix can make trio detection inefficient.

**Example of the pitfall:**
```python
# INEFFICIENT: Using adjacency list requires O(degree) lookup
adj_list = {i: set() for i in range(n)}
# Checking if edge exists: if k in adj_list[j] - O(1) but with higher constant
```

**Solution:**
Use an adjacency matrix for O(1) edge existence checks:
```python
# EFFICIENT: Direct O(1) lookup
if graph[i][k] and graph[j][k]:  # Instant check
```

### 4. **Duplicate Trio Counting**
While the nested loops with `i < j < k` prevent this, a common mistake when modifying the algorithm is to accidentally count the same trio multiple times.

**Example of the pitfall:**
```python
# WRONG: Could count same trio multiple times
for i in range(n):
    for j in range(n):  # Should be range(i+1, n)
        for k in range(n):  # Should be range(j+1, n)
```

**Solution:**
Maintain strict ordering in the loops:
```python
# CORRECT: Each trio counted exactly once
for i in range(n):
    for j in range(i + 1, n):
        for k in range(j + 1, n):
```

