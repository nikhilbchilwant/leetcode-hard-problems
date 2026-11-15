# 2846. Minimum Edge Weight Equilibrium Queries in a Tree

> Source: https://algo.monster/liteproblems/2846

## Problem Statement

You are given an undirected [tree](/problems/tree_intro) with `n` nodes labeled from `0` to `n - 1`. The tree is represented by:
- An integer `n` (number of nodes)
- A 2D integer array `edges` of length `n - 1`, where `edges[i] = [ui, vi, wi]` indicates an edge between nodes `ui` and `vi` with weight `wi`

You are also given a 2D integer array `queries` of length `m`, where `queries[i] = [ai, bi]` represents a query asking about the path from node `ai` to node `bi`.

For each query, you need to find the **minimum number of operations** required to make all edge weights on the path from `ai` to `bi` equal. In one operation, you can:
- Choose any edge in the [tree](/problems/tree_intro)
- Change its weight to any value

**Important notes:**
- Each query is independent - the [tree](/problems/tree_intro) returns to its initial state before processing each new query
- The path from `ai` to `bi` is a sequence of distinct nodes starting at `ai` and ending at `bi`, where every two adjacent nodes in the sequence share an edge

The task is to return an array `answer` of length `m` where `answer[i]` is the minimum number of operations needed for the `i-th` query.

**Example understanding:**
If a path from node `a` to node `b` has edges with weights `[1, 2, 1, 3, 1]`, to make all weights equal, we could change them all to `1` (the most frequent weight). This would require changing the weights `2` and `3`, resulting in 2 operations.

## Intuition

To make all edge weights equal on a path, we need to change some edges to have the same weight. The key insight is that we should change all edges to the weight that appears most frequently on the path - this minimizes the number of changes needed.

For example, if a path has edges with weights `[1, 2, 1, 3, 1]`, we have:
- Weight `1` appears 3 times
- Weight `2` appears 1 time  
- Weight `3` appears 1 time

The optimal strategy is to change everything to weight `1`, requiring only 2 operations (changing the `2` and the `3`).

This leads us to the formula: **minimum operations = total edges on path - frequency of most common weight**

Now, how do we efficiently find this information for any path between two nodes?

1. **Finding the path length**: In a [tree](/problems/tree_intro), there's exactly one path between any two nodes. The path from node `u` to node `v` goes through their Lowest Common Ancestor (LCA). If we know the depths of `u`, `v`, and their LCA `x`, the path length is `depth[u] + depth[v] - 2 * depth[x]`.

2. **Counting edge weights on the path**: We can precompute for each node how many times each weight appears on the path from the root to that node. Let's call this `cnt[node][weight]`. Then, for a path from `u` to `v` through LCA `x`, the frequency of weight `w` on this path is `cnt[u][w] + cnt[v][w] - 2 * cnt[x][w]` (we subtract the LCA's count twice because the path to LCA is counted in both `u` and `v`).

3. **Finding LCA efficiently**: Since we may have many queries, we need a fast way to find the LCA. Binary lifting is perfect for this - it preprocesses the [tree](/problems/tree_intro) to answer LCA queries in `O(log n)` time by storing the `2^j`-th ancestor of each node.

The solution combines these ideas: preprocess the [tree](/problems/tree_intro) with binary lifting and weight counts, then for each query, find the LCA, calculate the path length, find the maximum weight frequency on that path, and subtract it from the path length.

**Learn more about [Tree](/problems/tree_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The solution uses **Binary Lifting** to efficiently find the Lowest Common Ancestor (LCA) and track edge weight frequencies along paths. Let's walk through the implementation step by step:

### 1. Data Structure Initialization

```python
m = n.bit_length()  # Maximum power of 2 needed for binary lifting
g = [[] for _ in range(n)]  # Adjacency list for the [tree](/problems/tree_intro)
f = [[0] * m for _ in range(n)]  # Binary lifting table: f[i][j] = 2^j-th ancestor of node i
p = [0] * n  # Direct parent of each node
cnt = [None] * n  # cnt[i][w] = count of weight w from root to node i
depth = [0] * n  # Depth of each node from root
```

### 2. Building the [Tree](/problems/tree_intro) Structure

First, we convert the edge list into an adjacency list representation. Since edge weights range from 1 to 26, we store them as 0-25 internally:

```python
for u, v, w in edges:
    g[u].append((v, w - 1))  # Store weight as 0-indexed
    g[v].append((u, w - 1))
```

### 3. Preprocessing with BFS

We perform a BFS from the root (node 0) to:
- Build the binary lifting table `f`
- Track the depth of each node
- Count edge weight frequencies from root to each node

```python
cnt[0] = [0] * 26  # Root has no edges above it
q = deque([0])
while q:
    i = q.popleft()
    f[i][0] = p[i]  # Direct parent
  
    # Fill binary lifting table: 2^j-th ancestor = 2^(j-1)-th ancestor of 2^(j-1)-th ancestor
    for j in range(1, m):
        f[i][j] = f[f[i][j - 1]][j - 1]
  
    # Process children
    for j, w in g[i]:
        if j != p[i]:  # Avoid going back to parent
            p[j] = i
            cnt[j] = cnt[i][:]  # Copy parent's weight counts
            cnt[j][w] += 1  # Add current edge weight
            depth[j] = depth[i] + 1
            q.append(j)
```

### 4. Processing Queries

For each query `(u, v)`, we need to find their LCA and calculate the minimum operations:

#### Finding LCA using Binary Lifting:

```python
x, y = u, v
# Step 1: Make x the deeper node
if depth[x] < depth[y]:
    x, y = y, x

# Step 2: Lift x to the same depth as y
for j in reversed(range(m)):
    if depth[x] - depth[y] >= (1 << j):
        x = f[x][j]

# Step 3: Lift both nodes simultaneously until they meet at LCA
for j in reversed(range(m)):
    if f[x][j] != f[y][j]:
        x, y = f[x][j], f[y][j]

# If x != y, their parent is the LCA
if x != y:
    x = p[x]  # x is now the LCA
```

#### Calculating Minimum Operations:

Once we have the LCA `x`, we can calculate:

1. **Path length**: `depth[u] + depth[v] - 2 * depth[x]`

2. **Maximum weight frequency on path**: For each weight `j` from 0 to 25, the frequency on the path from `u` to `v` is `cnt[u][j] + cnt[v][j] - 2 * cnt[x][j]`. We find the maximum among all weights.

3. **Minimum operations**: Path length minus maximum frequency

```python
mx = max(cnt[u][j] + cnt[v][j] - 2 * cnt[x][j] for j in range(26))
ans.append(depth[u] + depth[v] - 2 * depth[x] - mx)
```

### Time Complexity

- **Preprocessing**: `O(n log n)` for building the binary lifting table
- **Per query**: `O(log n)` for finding LCA + `O(26)` for finding maximum frequency = `O(log n)`
- **Total**: `O(n log n + q log n)` where `q` is the number of queries

### Space Complexity

- `O(n log n)` for the binary lifting table
- `O(26n)` for the weight frequency counts
- Overall: `O(n log n)`

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- Tree with 5 nodes (0-4)
- Edges: `[[0,1,2], [1,2,3], [1,3,2], [3,4,3]]`
- Query: Find minimum operations for path from node 2 to node 4

**Step 1: Build the Tree**
```
    0
    |
   (2)
    |
    1
   / \
 (3)  (2)
 /     \
2       3
        |
       (3)
        |
        4
```
Numbers in parentheses are edge weights.

**Step 2: Preprocessing with BFS from root (node 0)**

After BFS, we have:
- `depth`: [0, 1, 2, 2, 3]
- `parent`: [0, 0, 1, 1, 3]
- `cnt` (weight frequencies from root):
  - Node 0: weight 2 appears 0 times, weight 3 appears 0 times
  - Node 1: weight 2 appears 1 time (edge 0→1)
  - Node 2: weight 2 appears 1 time, weight 3 appears 1 time (edges 0→1→2)
  - Node 3: weight 2 appears 2 times (edges 0→1→3)
  - Node 4: weight 2 appears 2 times, weight 3 appears 1 time (edges 0→1→3→4)

**Step 3: Process Query (2, 4)**

Finding LCA of nodes 2 and 4:
- Node 2 is at depth 2, node 4 is at depth 3
- Make node 4 jump to same depth as node 2: node 4 → node 3 (now both at depth 2)
- Both nodes jump up simultaneously: node 2 → node 1, node 3 → node 1
- They meet at node 1, so LCA = 1

**Step 4: Calculate Path Information**

Path from 2 to 4 goes: 2 → 1 → 3 → 4

Path length = depth[2] + depth[4] - 2×depth[1] = 2 + 3 - 2×1 = 3 edges

Weight frequencies on path:
- For weight 2: cnt[2][2] + cnt[4][2] - 2×cnt[1][2] = 1 + 2 - 2×1 = 1
- For weight 3: cnt[2][3] + cnt[4][3] - 2×cnt[1][3] = 1 + 1 - 2×0 = 2

The path has edges with weights [3, 2, 3], confirming:
- Weight 2 appears 1 time
- Weight 3 appears 2 times

Maximum frequency = 2 (weight 3 appears most)

**Step 5: Calculate Answer**

Minimum operations = Path length - Maximum frequency = 3 - 2 = 1

We need to change only 1 edge (the edge with weight 2) to make all weights equal to 3.

---

### Another Query Example: (0, 2)

Path: 0 → 1 → 2 (edges with weights [2, 3])
- LCA of 0 and 2 is node 0 (since 0 is ancestor of 2)
- Path length = depth[0] + depth[2] - 2×depth[0] = 0 + 2 - 0 = 2
- Weight 2 frequency: cnt[0][2] + cnt[2][2] - 2×cnt[0][2] = 0 + 1 - 0 = 1
- Weight 3 frequency: cnt[0][3] + cnt[2][3] - 2×cnt[0][3] = 0 + 1 - 0 = 1
- Maximum frequency = 1
- Minimum operations = 2 - 1 = 1

This makes sense: we have weights [2, 3] on the path, and we need to change 1 of them to make both equal.

## Python Solution

```python
class Solution:
    def minOperationsQueries(
        self, n: int, edges: List[List[int]], queries: List[List[int]]
    ) -> List[int]:
        # Calculate maximum number of bits needed for binary lifting
        max_log = n.bit_length()
      
        # Build adjacency list for the tree
        graph = [[] for _ in range(n)]
        for u, v, weight in edges:
            # Convert weight to 0-indexed (weights are 1-26, store as 0-25)
            graph[u].append((v, weight - 1))
            graph[v].append((u, weight - 1))
      
        # Binary lifting table: ancestor[node][power] = 2^power-th ancestor of node
        ancestor = [[0] * max_log for _ in range(n)]
      
        # Parent of each node
        parent = [0] * n
      
        # Weight frequency count from root to each node
        weight_count = [None] * n
      
        # Depth of each node from root
        depth = [0] * n
      
        # Initialize root node (node 0)
        weight_count[0] = [0] * 26  # 26 possible weights
      
        # BFS to build the tree structure
        queue = deque([0])
        while queue:
            current_node = queue.popleft()
          
            # Build binary lifting table for current node
            ancestor[current_node][0] = parent[current_node]
            for power in range(1, max_log):
                ancestor[current_node][power] = ancestor[ancestor[current_node][power - 1]][power - 1]
          
            # Process neighbors
            for neighbor, edge_weight in graph[current_node]:
                if neighbor != parent[current_node]:
                    # Set parent relationship
                    parent[neighbor] = current_node
                  
                    # Copy weight counts and increment current edge weight
                    weight_count[neighbor] = weight_count[current_node][:]
                    weight_count[neighbor][edge_weight] += 1
                  
                    # Update depth
                    depth[neighbor] = depth[current_node] + 1
                  
                    # Add to queue for processing
                    queue.append(neighbor)
      
        # Process each query
        result = []
        for u, v in queries:
            # Find LCA using binary lifting
            node_x, node_y = u, v
          
            # Ensure node_x is deeper or at same level as node_y
            if depth[node_x] < depth[node_y]:
                node_x, node_y = node_y, node_x
          
            # Lift node_x to the same level as node_y
            for power in reversed(range(max_log)):
                if depth[node_x] - depth[node_y] >= (1 << power):
                    node_x = ancestor[node_x][power]
          
            # Lift both nodes until they meet at LCA
            for power in reversed(range(max_log)):
                if ancestor[node_x][power] != ancestor[node_y][power]:
                    node_x = ancestor[node_x][power]
                    node_y = ancestor[node_y][power]
          
            # If nodes are different, their parent is the LCA
            if node_x != node_y:
                node_x = parent[node_x]
          
            lca = node_x
          
            # Calculate the most frequent weight on the path from u to v
            max_weight_frequency = max(
                weight_count[u][weight_type] + weight_count[v][weight_type] - 2 * weight_count[lca][weight_type]
                for weight_type in range(26)
            )
          
            # Total path length minus most frequent weight gives minimum operations
            path_length = depth[u] + depth[v] - 2 * depth[lca]
            min_operations = path_length - max_weight_frequency
          
            result.append(min_operations)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O((n + q) × C × log n)` where:
- `n` is the number of nodes
- `q` is the number of queries
- `C = 26` (the maximum value of edge weight, as weights range from 1-26)
- `log n` comes from the binary lifting preprocessing and LCA computation

The time complexity breaks down as follows:
1. **Preprocessing phase:** `O(n × log n)`
   - Building the adjacency list: `O(n)`
   - BFS traversal to compute depth, parent relationships, and weight counts: `O(n × C)`
   - Binary lifting table construction: `O(n × log n)`
 
2. **Query phase:** `O(q × (log n + C))`
   - For each query, finding LCA using binary lifting: `O(log n)`
   - Computing the maximum frequency among all weights: `O(C)`

Overall: `O(n × C + n × log n + q × (log n + C)) = O((n + q) × C × log n)`

**Space Complexity:** `O(n × C × log n)` where:
- Binary lifting table `f`: `O(n × log n)`
- Weight count array `cnt`: `O(n × C)` where each node stores a frequency array of size 26
- Adjacency list `g`: `O(n)`
- Other auxiliary arrays (depth, parent): `O(n)`

The dominant space factor is `O(n × C)` for the count arrays, but when combined with the binary lifting table, the overall space complexity is `O(n × C × log n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Incorrect LCA Calculation When One Node is Ancestor of Another

**The Pitfall:**
A critical bug occurs in the LCA finding logic when one node is already the ancestor of another. After lifting the deeper node to the same level as the shallower one, if they become the same node (meaning one was the ancestor of the other), the code still proceeds to the binary lifting loop and incorrectly sets the LCA to the parent of this node.

**Example:**
Consider a path from node 2 to node 0 where node 0 is the root and ancestor of node 2:
- After lifting node 2 to the same depth as node 0, we get `node_x = node_y = 0`
- The code then checks `if node_x != node_y:` which is false, so it doesn't execute `node_x = parent[node_x]`
- However, this means `lca = node_x = 0`, which is correct by luck
- But if the query was reversed (0 to 2), after depth adjustment, both would be at node 0, and the LCA would correctly be 0

The issue becomes apparent in other tree structures where the ancestor relationship isn't at the root.

**The Solution:**
Add an early termination check after equalizing depths:

```python
# Lift node_x to the same level as node_y
for power in reversed(range(max_log)):
    if depth[node_x] - depth[node_y] >= (1 << power):
        node_x = ancestor[node_x][power]

# Early termination: if they're already the same, that's the LCA
if node_x == node_y:
    lca = node_x
else:
    # Lift both nodes until they meet at LCA
    for power in reversed(range(max_log)):
        if ancestor[node_x][power] != ancestor[node_y][power]:
            node_x = ancestor[node_x][power]
            node_y = ancestor[node_y][power]
  
    # Their parent is the LCA
    lca = parent[node_x]
```

### 2. Off-by-One Error in Weight Indexing

**The Pitfall:**
The problem states weights range from 1 to 26, but the code stores them as 0-25 for array indexing. Forgetting this conversion in any part of the code leads to:
- Array index out of bounds errors
- Incorrect weight frequency calculations
- Wrong answer for minimum operations

**Common mistake locations:**
- When building the graph: forgetting `weight - 1`
- When processing weight counts: using wrong array size (25 instead of 26)
- Mixing 0-indexed and 1-indexed weight values in different parts of code

**The Solution:**
Be consistent with the indexing throughout:
```python
# Always convert when reading edges
graph[u].append((v, weight - 1))  # weight - 1 for 0-indexing

# Use correct array size
weight_count[0] = [0] * 26  # 26 possible weights (1-26, stored as 0-25)

# Document the indexing scheme clearly
# Weight w (1-26) is stored at index w-1 (0-25)
```

### 3. Parent Tracking During BFS

**The Pitfall:**
When building the tree using BFS, not properly checking if a neighbor is the parent can cause:
- Creating cycles in parent relationships
- Incorrect depth calculations
- Wrong weight frequency counts

**The Solution:**
Always exclude the parent when traversing neighbors:
```python
for neighbor, edge_weight in graph[current_node]:
    if neighbor != parent[current_node]:  # Critical check
        parent[neighbor] = current_node
        # ... rest of the processing
```

