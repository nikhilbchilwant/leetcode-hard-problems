# 1489. Find Critical and Pseudo-Critical Edges in Minimum Spanning Tree

> Source: https://algo.monster/liteproblems/1489

## Problem Statement

You are given a weighted undirected connected [graph](/problems/graph_intro) with `n` vertices (numbered from `0` to `n-1`) and an array `edges`. Each element `edges[i] = [ai, bi, weighti]` represents a bidirectional weighted edge between nodes `ai` and `bi` with weight `weighti`.

A **[Minimum Spanning Tree](/problems/mst_forest) (MST)** is a subset of the [graph](/problems/graph_intro)'s edges that:
- Connects all vertices together
- Contains no cycles
- Has the minimum possible total edge weight

Your task is to identify two types of edges in relation to the MST:

1. **Critical Edges**: These are edges that appear in every possible MST of the [graph](/problems/graph_intro). If you remove a critical edge from the graph, the weight of the MST would increase (or it would become impossible to form an MST).

2. **Pseudo-Critical Edges**: These are edges that appear in at least one MST but not in all MSTs. They can be part of some valid MSTs but are not essential.

You need to return two lists:
- The first list contains the indices of all critical edges
- The second list contains the indices of all pseudo-critical edges

The indices refer to the original positions of edges in the input `edges` array, and you can return them in any order within each list.

For example, if an edge at index `2` in the original `edges` array is critical, then `2` should appear in the first returned list. If an edge at index `5` can appear in some MSTs but not all, then `5` should appear in the second returned list.

## Intuition

To understand which edges are critical or pseudo-critical, we need to think about what makes an edge special in an MST.

First, let's establish a baseline - we need to know the weight of the actual MST. We can find this using a standard MST algorithm like Kruskal's ([sorting](/problems/sorting_summary) edges by weight and using Union-Find to avoid cycles).

Now, how do we identify critical edges? A critical edge is one that **must** be in every MST. If we remove it from the [graph](/problems/graph_intro) entirely and try to build an MST without it, one of two things will happen:
1. We can't connect all vertices anymore (the graph becomes disconnected)
2. We can connect all vertices, but the total weight increases

This gives us our first test: for each edge, remove it from the [graph](/problems/graph_intro) and try to build an MST. If we can't build one with the same weight as the original MST, that edge is critical.

For pseudo-critical edges, we need a different approach. A pseudo-critical edge is one that **can** be part of an MST but isn't required. How do we test if an edge can be part of an MST? We can force it to be included! 

Here's the key insight: if we force an edge to be in our spanning tree from the start (by including it first before running the MST algorithm), and the resulting total weight equals the original MST weight, then this edge can be part of at least one valid MST.

The strategy becomes:
1. Calculate the original MST weight `v`
2. For each edge:
   - Test if it's critical by excluding it and checking if we get a higher weight or disconnected [graph](/problems/graph_intro)
   - If it's not critical, test if it's pseudo-critical by forcing its inclusion and checking if we still get weight `v`

This approach cleverly uses the property that if an edge is neither critical nor pseudo-critical, forcing its inclusion would result in a spanning tree with weight greater than the MST weight (because we're forcing a suboptimal edge into the solution).

**Learn more about [Union Find](/problems/dsu_intro), [Graph](/problems/graph_intro), [Minimum Spanning Tree](/problems/mst_forest) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation uses Kruskal's algorithm with Union-Find to build MSTs and test edges for criticality.

**Step 1: Prepare the edges**
```python
for i, e in enumerate(edges):
    e.append(i)
edges.sort(key=lambda x: x[2])
```
We append the original index to each edge (so we can track which edge is which after [sorting](/problems/sorting_summary)), then sort all edges by weight. This prepares us for Kruskal's algorithm.

**Step 2: Calculate the original MST weight**
```python
uf = UnionFind(n)
v = sum(w for f, t, w, _ in edges if uf.union(f, t))
```
Using Union-Find, we build the MST by iterating through sorted edges. The `union` method returns `True` if the two nodes weren't already connected (edge is added to MST), `False` if they were (edge would create a cycle). We sum up weights of edges that were successfully added to get the MST weight `v`.

**Step 3: Test each edge**
For each edge `(f, t, w, i)` where `i` is the original index:

**Testing for Critical Edge:**
```python
uf = UnionFind(n)
k = sum(z for x, y, z, j in edges if j != i and uf.union(x, y))
if uf.n > 1 or (uf.n == 1 and k > v):
    ans[0].append(i)
    continue
```
- Build an MST excluding edge `i` (condition `j != i`)
- Check if the [graph](/problems/graph_intro) is still connected (`uf.n == 1` means all nodes are in one component)
- If disconnected (`uf.n > 1`) or if the new MST weight `k` is greater than `v`, the edge is critical

**Testing for Pseudo-Critical Edge:**
```python
uf = UnionFind(n)
uf.union(f, t)
k = w + sum(z for x, y, z, j in edges if j != i and uf.union(x, y))
if k == v:
    ans[1].append(i)
```
- Force the edge into the MST by calling `uf.union(f, t)` first
- Add its weight `w` to the total
- Build the rest of the MST excluding this edge from the iteration
- If the final weight equals `v`, this edge can be part of an MST (pseudo-critical)

**Union-Find Data Structure:**
The `UnionFind` class maintains:
- `p`: parent array for each node
- `n`: number of connected components (starts at `n`, decreases with each union)

The `find` method uses path compression for efficiency:
```python
def find(self, x):
    if self.p[x] != x:
        self.p[x] = self.find(self.p[x])
    return self.p[x]
```

The `union` method connects two components and returns whether they were previously disconnected:
```python
def union(self, a, b):
    if self.find(a) == self.find(b):
        return False
    self.p[self.find(a)] = self.find(b)
    self.n -= 1
    return True
```

This approach efficiently identifies both critical and pseudo-critical edges in `O(E² × α(V))` time, where `E` is the number of edges and `α` is the inverse Ackermann function (practically constant).

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example Graph:**
- `n = 4` (vertices: 0, 1, 2, 3)
- `edges = [[0,1,1], [1,2,1], [2,3,2], [0,3,2], [1,3,3]]`

**Step 1: Prepare edges with indices and sort by weight**
After appending original indices and sorting:
```
edges = [[0,1,1,0], [1,2,1,1], [2,3,2,2], [0,3,2,3], [1,3,3,4]]
         (from, to, weight, original_index)
```

**Step 2: Calculate original MST weight**
Using Kruskal's algorithm with Union-Find:
- Edge [0,1,1]: Connect 0-1, add to MST (weight = 1)
- Edge [1,2,1]: Connect 1-2, add to MST (weight = 1) 
- Edge [2,3,2]: Connect 2-3, add to MST (weight = 2)
- Edge [0,3,2]: Skip (would create cycle 0-1-2-3-0)
- Edge [1,3,3]: Skip (would create cycle)

MST weight `v = 1 + 1 + 2 = 4`

**Step 3: Test each edge**

**Testing edge 0 [0,1,1]:**
- Exclude it and build MST: Use edges [1,2,1], [2,3,2], [0,3,2] → weight = 5
- Since 5 > 4, edge 0 is **critical**

**Testing edge 1 [1,2,1]:**
- Exclude it and build MST: Use edges [0,1,1], [2,3,2], [0,3,2] → weight = 5
- Since 5 > 4, edge 1 is **critical**

**Testing edge 2 [2,3,2]:**
- Exclude it and build MST: Use edges [0,1,1], [1,2,1], [0,3,2], [1,3,3] → weight = 5
- Since 5 > 4, edge 2 is **critical**

**Testing edge 3 [0,3,2]:**
- Exclude it and build MST: Use edges [0,1,1], [1,2,1], [2,3,2] → weight = 4
- Not critical (same weight without it)
- Force include it: Start with [0,3,2], then add [0,1,1], [1,2,1] → weight = 4
- Since weight equals v, edge 3 is **pseudo-critical**

**Testing edge 4 [1,3,3]:**
- Exclude it and build MST: Use edges [0,1,1], [1,2,1], [2,3,2] → weight = 4
- Not critical
- Force include it: Start with [1,3,3], then add [0,1,1], [1,2,1] → weight = 5
- Since 5 > 4, edge 4 is neither critical nor pseudo-critical

**Result:**
- Critical edges: [0, 1, 2]
- Pseudo-critical edges: [3]

This example demonstrates how the algorithm identifies edges that must appear in every MST (critical) versus edges that can appear in some MSTs but not all (pseudo-critical).

## Python Solution

```python
from typing import List


class UnionFind:
    """Union-Find (Disjoint Set Union) data structure with path compression."""
  
    def __init__(self, n: int) -> None:
        """Initialize n disjoint sets, each containing one element."""
        self.parent = list(range(n))  # Each node is initially its own parent
        self.num_components = n  # Number of connected components
  
    def union(self, node_a: int, node_b: int) -> bool:
        """
        Union two sets containing node_a and node_b.
        Returns True if they were in different sets, False otherwise.
        """
        root_a = self.find(node_a)
        root_b = self.find(node_b)
      
        if root_a == root_b:
            return False  # Already in the same set
      
        # Merge the two sets by making one root point to the other
        self.parent[root_a] = root_b
        self.num_components -= 1
        return True
  
    def find(self, node: int) -> int:
        """
        Find the root of the set containing node.
        Applies path compression for optimization.
        """
        if self.parent[node] != node:
            # Path compression: make all nodes point directly to root
            self.parent[node] = self.find(self.parent[node])
        return self.parent[node]


class Solution:
    def findCriticalAndPseudoCriticalEdges(
        self, n: int, edges: List[List[int]]
    ) -> List[List[int]]:
        """
        Find critical and pseudo-critical edges in an MST.
      
        Critical edge: MST weight increases if this edge is removed
        Pseudo-critical edge: Can be in some MST but not critical
      
        Args:
            n: Number of vertices
            edges: List of edges [from, to, weight]
      
        Returns:
            List containing two lists: [critical_edges, pseudo_critical_edges]
        """
        # Add original index to each edge to track them after sorting
        for edge_idx, edge in enumerate(edges):
            edge.append(edge_idx)
      
        # Sort edges by weight for Kruskal's algorithm
        edges.sort(key=lambda edge: edge[2])
      
        # Find MST weight using standard Kruskal's algorithm
        uf_mst = UnionFind(n)
        mst_weight = 0
        for from_node, to_node, weight, _ in edges:
            if uf_mst.union(from_node, to_node):
                mst_weight += weight
      
        critical_edges = []
        pseudo_critical_edges = []
      
        # Check each edge to determine if it's critical or pseudo-critical
        for from_node, to_node, weight, original_idx in edges:
            # Test 1: Try building MST without this edge
            uf_without = UnionFind(n)
            weight_without = 0
          
            for curr_from, curr_to, curr_weight, curr_idx in edges:
                if curr_idx != original_idx:  # Skip current edge
                    if uf_without.union(curr_from, curr_to):
                        weight_without += curr_weight
          
            # If graph is disconnected or weight increases, edge is critical
            if uf_without.num_components > 1 or weight_without > mst_weight:
                critical_edges.append(original_idx)
                continue
          
            # Test 2: Force include this edge and build MST
            uf_with = UnionFind(n)
            uf_with.union(from_node, to_node)
            weight_with = weight  # Start with current edge weight
          
            for curr_from, curr_to, curr_weight, curr_idx in edges:
                if curr_idx != original_idx:  # Skip current edge (already included)
                    if uf_with.union(curr_from, curr_to):
                        weight_with += curr_weight
          
            # If MST weight remains same, edge is pseudo-critical
            if weight_with == mst_weight:
                pseudo_critical_edges.append(original_idx)
      
        return [critical_edges, pseudo_critical_edges]
```

## Time and Space Complexity

**Time Complexity:** `O(E² * α(V))` where `E` is the number of edges, `V` is the number of vertices, and `α` is the inverse Ackermann function.

- Sorting edges takes `O(E log E)`
- Finding the MST weight initially requires `O(E * α(V))` operations
- For each edge (total `E` edges), we perform:
  - First check (excluding edge): iterate through all edges performing union-find operations, which takes `O(E * α(V))`
  - Second check (forcing edge): iterate through all edges performing union-find operations, which takes `O(E * α(V))`
- Overall: `O(E log E) + O(E * α(V)) + E * O(E * α(V))` = `O(E² * α(V))`

Since `α(V)` grows extremely slowly and is effectively constant for practical purposes, the time complexity can be approximated as `O(E²)`.

**Space Complexity:** `O(E + V)`

- Modified edges list with indices: `O(E)` (adding one index per edge)
- UnionFind data structure: `O(V)` for the parent array
- The algorithm creates new UnionFind instances for each edge check, but only one exists at a time
- Result lists store at most `E` indices: `O(E)`
- Overall space complexity: `O(E + V)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Track Original Edge Indices**
One of the most common mistakes is sorting the edges by weight without preserving the original indices. Since the problem asks for the indices from the original input array, losing track of these indices after sorting will produce incorrect results.

**Incorrect approach:**
```python
edges.sort(key=lambda x: x[2])  # Original indices are lost!
# Later trying to return edge positions won't match original input
```

**Correct approach:**
```python
# Append original index before sorting
for i, edge in enumerate(edges):
    edge.append(i)
edges.sort(key=lambda x: x[2])
# Now edge[3] contains the original index
```

### 2. **Incorrectly Determining Graph Connectivity**
When testing if an edge is critical by excluding it, developers often check only the MST weight but forget to verify if the graph remains connected. An edge is critical if removing it either disconnects the graph OR increases the MST weight.

**Incorrect approach:**
```python
# Only checking weight difference
if weight_without > mst_weight:
    critical_edges.append(i)
```

**Correct approach:**
```python
# Check both disconnection AND weight increase
if uf_without.num_components > 1 or weight_without > mst_weight:
    critical_edges.append(i)
```

### 3. **Not Properly Forcing Edge Inclusion for Pseudo-Critical Test**
When testing if an edge is pseudo-critical, you must force it into the MST first, then build the rest. A common mistake is to just prioritize it during the normal MST construction process, which doesn't guarantee its inclusion.

**Incorrect approach:**
```python
# Just trying to use the edge first doesn't guarantee inclusion
uf_with = UnionFind(n)
weight_with = 0
# Process current edge first
if uf_with.union(from_node, to_node):
    weight_with += weight
# Then process others... but the edge might already be redundant!
```

**Correct approach:**
```python
# Force the edge inclusion first
uf_with = UnionFind(n)
uf_with.union(from_node, to_node)  # Force union
weight_with = weight  # Always add its weight
# Then build the rest of MST
for curr_from, curr_to, curr_weight, curr_idx in edges:
    if curr_idx != original_idx:
        if uf_with.union(curr_from, curr_to):
            weight_with += curr_weight
```

### 4. **Edge Classification Logic Error**
Some implementations mistakenly classify edges as both critical and pseudo-critical, or miss the fact that these are mutually exclusive categories. Once an edge is identified as critical, it should not be tested for pseudo-critical status.

**Incorrect approach:**
```python
for edge in edges:
    if is_critical(edge):
        critical_edges.append(edge)
    if is_pseudo_critical(edge):  # Wrong! Should not test if already critical
        pseudo_critical_edges.append(edge)
```

**Correct approach:**
```python
for edge in edges:
    if is_critical(edge):
        critical_edges.append(edge)
        continue  # Skip pseudo-critical test
    if is_pseudo_critical(edge):
        pseudo_critical_edges.append(edge)
```

### 5. **Union-Find Path Compression Implementation Error**
A subtle but impactful mistake in the Union-Find implementation is forgetting to update the parent during path compression, leading to inefficient operations.

**Incorrect approach:**
```python
def find(self, x):
    if self.parent[x] != x:
        return self.find(self.parent[x])  # No path compression!
    return self.parent[x]
```

**Correct approach:**
```python
def find(self, x):
    if self.parent[x] != x:
        self.parent[x] = self.find(self.parent[x])  # Path compression
    return self.parent[x]
```

These pitfalls can lead to incorrect classification of edges, time limit exceeded errors, or runtime errors. Always ensure proper index tracking, complete connectivity checks, and correct edge forcing logic when implementing this algorithm.

