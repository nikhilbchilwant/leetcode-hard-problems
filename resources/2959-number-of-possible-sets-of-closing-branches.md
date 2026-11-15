# 2959. Number of Possible Sets of Closing Branches

> Source: https://algo.monster/liteproblems/2959

## Problem Statement

You have a company with `n` branches (numbered from 0 to n-1) connected by roads. Initially, all branches can reach each other through these roads.

The company wants to reduce travel time by potentially closing some branches. After closing branches, they need to ensure that among the remaining open branches, the maximum distance between any two branches doesn't exceed `maxDistance`.

Given:
- `n`: the number of branches
- `maxDistance`: the maximum allowed distance between any two remaining branches
- `roads`: a 2D array where `roads[i] = [ui, vi, wi]` represents an undirected road between branch `ui` and branch `vi` with length `wi`

The distance between two branches is defined as the minimum total length needed to travel from one to the other.

When a branch is closed, all roads connected to it become inaccessible.

Your task is to find the total number of different sets of branches that can remain open (including the possibility of keeping all branches open or closing all branches) such that the distance between any two open branches is at most `maxDistance`.

Note that there can be multiple roads between the same pair of branches.

For example, if you have 3 branches and decide to keep branches 0 and 2 open while closing branch 1, then you need to check if the distance between branches 0 and 2 (using only roads that don't involve branch 1) is at most `maxDistance`.

## Intuition

The key observation is that `n ≤ 10`, which means we have at most 10 branches. This small constraint immediately suggests that we can try all possible combinations of which branches to keep open.

Since we have `n` branches, and each branch can either be open or closed, we have `2^n` possible configurations. With `n ≤ 10`, this gives us at most `2^10 = 1024` possibilities, which is computationally feasible to check all of them.

For each possible subset of branches that we keep open, we need to verify if the maximum distance between any two open branches is within `maxDistance`. To find the shortest distances between all pairs of branches, we can use the Floyd-Warshall algorithm.

The approach becomes:
1. Use binary enumeration to generate all possible subsets of branches (represented by a bitmask from `0` to `2^n - 1`)
2. For each subset (mask), build a distance [graph](/problems/graph_intro) considering only the roads between open branches
3. Apply Floyd-Warshall to find shortest paths between all pairs of open branches
4. Check if all distances between open branches are at most `maxDistance`
5. Count how many valid subsets satisfy the condition

The binary mask representation is particularly elegant here: if bit `i` is set in the mask, it means branch `i` is open. For example, mask `101` (binary) means branches 0 and 2 are open while branch 1 is closed.

When building the [graph](/problems/graph_intro) for each subset, we only include roads where both endpoints are open branches (`mask >> u & 1` and `mask >> v & 1` both equal 1). This naturally handles the constraint that closed branches and their connected roads become inaccessible.

**Learn more about [Graph](/problems/graph_intro), [Shortest Path](/problems/shortest_path_unweight) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution uses **binary enumeration** combined with the **Floyd-Warshall algorithm** to find all valid sets of branches.

**Step 1: Binary Enumeration**
We iterate through all possible subsets using a mask from `0` to `2^n - 1`. Each bit position in the mask represents whether a branch is open (1) or closed (0).

```python
for mask in range(1 << n):
```

**Step 2: Build Distance [Graph](/problems/graph_intro)**
For each subset, we initialize a distance matrix `g` with infinity values. We then populate it with roads that connect only open branches:

```python
g = [[inf] * n for _ in range(n)]
for u, v, w in roads:
    if mask >> u & 1 and mask >> v & 1:  # Both branches are open
        g[u][v] = min(g[u][v], w)
        g[v][u] = min(g[v][u], w)
```

The condition `mask >> u & 1` checks if branch `u` is open by right-shifting the mask by `u` positions and checking the least significant bit. We use `min()` to handle multiple roads between the same branches.

**Step 3: Floyd-Warshall Algorithm**
We apply Floyd-Warshall to find shortest paths between all pairs of open branches:

```python
for k in range(n):
    if mask >> k & 1:  # Only use open branches as intermediate nodes
        g[k][k] = 0
        for i in range(n):
            for j in range(n):
                if g[i][k] + g[k][j] < g[i][j]:
                    g[i][j] = g[i][k] + g[k][j]
```

The algorithm considers each branch `k` as an intermediate point and updates `g[i][j]` if a shorter path exists through `k`. We only use open branches as intermediate nodes.

**Step 4: Validate the Subset**
After computing all shortest distances, we check if all distances between open branches are within `maxDistance`:

```python
if all(
    g[i][j] <= maxDistance
    for i in range(n)
    for j in range(n)
    if mask >> i & 1 and mask >> j & 1
):
    ans += 1
```

We only check distances between branches that are actually open in the current subset.

**Time Complexity:** `O(2^n * n^3)` where `2^n` is for enumerating all subsets and `n^3` is for Floyd-Warshall on each subset.

**Space Complexity:** `O(n^2)` for the distance matrix.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Example:** 
- `n = 3` branches (labeled 0, 1, 2)
- `maxDistance = 5`
- `roads = [[0, 1, 2], [1, 2, 3], [0, 2, 7]]`

This creates a triangle where:
- Branch 0 to 1: distance 2
- Branch 1 to 2: distance 3  
- Branch 0 to 2: distance 7

**Step 1: Enumerate all possible subsets**

We'll check all 8 possible masks (000 to 111 in binary):

**Mask = 0 (binary: 000)** - All branches closed
- No open branches, so trivially valid
- Count: 1

**Mask = 1 (binary: 001)** - Only branch 0 open
- Single branch, no pairs to check
- Count: 2

**Mask = 2 (binary: 010)** - Only branch 1 open
- Single branch, no pairs to check
- Count: 3

**Mask = 3 (binary: 011)** - Branches 0 and 1 open
- Build graph with only roads between 0 and 1
- Distance[0][1] = 2 (from road [0,1,2])
- Check: 2 ≤ 5? Yes, valid
- Count: 4

**Mask = 4 (binary: 100)** - Only branch 2 open
- Single branch, no pairs to check
- Count: 5

**Mask = 5 (binary: 101)** - Branches 0 and 2 open
- Build graph with only roads between 0 and 2
- Distance[0][2] = 7 (from road [0,2,7])
- Check: 7 ≤ 5? No, invalid
- Count: 5 (unchanged)

**Mask = 6 (binary: 110)** - Branches 1 and 2 open
- Build graph with only roads between 1 and 2
- Distance[1][2] = 3 (from road [1,2,3])
- Check: 3 ≤ 5? Yes, valid
- Count: 6

**Mask = 7 (binary: 111)** - All branches open
- Build complete graph with all roads
- Initial distances:
  - Distance[0][1] = 2
  - Distance[1][2] = 3
  - Distance[0][2] = 7
- Apply Floyd-Warshall:
  - Using branch 1 as intermediate: Distance[0][2] = min(7, 2+3) = 5
- Final distances after Floyd-Warshall:
  - Distance[0][1] = 2
  - Distance[1][2] = 3
  - Distance[0][2] = 5
- Check all pairs: max(2, 3, 5) = 5 ≤ 5? Yes, valid
- Count: 7

**Final Answer: 7**

The key insight demonstrated here is that mask 5 (branches 0 and 2 open) is invalid because the direct distance between them (7) exceeds maxDistance. However, mask 7 (all branches open) is valid because we can travel from branch 0 to 2 via branch 1, giving us a shorter path of length 5.

## Python Solution

```python
from typing import List
from math import inf


class Solution:
    def numberOfSets(self, n: int, maxDistance: int, roads: List[List[int]]) -> int:
        """
        Count the number of valid subsets of nodes where all pairwise distances
        are at most maxDistance.
      
        Args:
            n: Number of nodes in the graph
            maxDistance: Maximum allowed distance between any two nodes in a valid subset
            roads: List of edges [u, v, weight] representing undirected roads
      
        Returns:
            Number of valid subsets
        """
        valid_subset_count = 0
      
        # Iterate through all possible subsets using bit masking
        # Each bit position i represents whether node i is included in the subset
        for subset_mask in range(1 << n):
            # Initialize distance matrix with infinity
            distance_matrix = [[inf] * n for _ in range(n)]
          
            # Build adjacency matrix for nodes in current subset
            for u, v, weight in roads:
                # Check if both endpoints are in the current subset
                if (subset_mask >> u) & 1 and (subset_mask >> v) & 1:
                    # Update with minimum weight (handle multiple edges between same nodes)
                    distance_matrix[u][v] = min(distance_matrix[u][v], weight)
                    distance_matrix[v][u] = min(distance_matrix[v][u], weight)
          
            # Floyd-Warshall algorithm to find shortest paths
            for intermediate_node in range(n):
                # Only use nodes that are in the current subset as intermediate nodes
                if (subset_mask >> intermediate_node) & 1:
                    # Distance from a node to itself is 0
                    distance_matrix[intermediate_node][intermediate_node] = 0
                  
                    # Update shortest paths through intermediate_node
                    for start_node in range(n):
                        for end_node in range(n):
                            # Relax the distance if a shorter path exists through intermediate_node
                            if distance_matrix[start_node][intermediate_node] + distance_matrix[intermediate_node][end_node] < distance_matrix[start_node][end_node]:
                                distance_matrix[start_node][end_node] = distance_matrix[start_node][intermediate_node] + distance_matrix[intermediate_node][end_node]
          
            # Check if all pairwise distances in subset satisfy the maxDistance constraint
            is_valid_subset = all(
                distance_matrix[i][j] <= maxDistance
                for i in range(n)
                for j in range(n)
                if (subset_mask >> i) & 1 and (subset_mask >> j) & 1
            )
          
            if is_valid_subset:
                valid_subset_count += 1
      
        return valid_subset_count
```

## Time and Space Complexity

**Time Complexity:** `O(2^n × (n^3 + m))`

The algorithm iterates through all possible subsets of nodes using a bitmask, which gives us `2^n` iterations in the outer loop. For each subset (mask):

1. Building the adjacency matrix takes `O(m)` time, where we iterate through all roads and update the graph only if both endpoints are in the current subset.

2. Floyd-Warshall algorithm is applied to find all-pairs shortest paths, which has three nested loops each running up to `n` times, resulting in `O(n^3)` time complexity.

3. The final validation check iterates through all pairs of nodes in the subset, which in the worst case is `O(n^2)`, but this is dominated by the Floyd-Warshall step.

Therefore, the total time complexity is `O(2^n × (n^3 + m))`, where `n` is the number of nodes and `m` is the number of roads.

**Space Complexity:** `O(n^2)`

The space is dominated by the 2D adjacency matrix `g` which stores the distances between all pairs of nodes. This matrix has dimensions `n × n`, requiring `O(n^2)` space. The other variables (`ans`, `mask`, loop indices) use constant space, so the overall space complexity is `O(n^2)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Set Self-Distance to Zero**

**Pitfall:** A critical issue occurs when we don't explicitly set `distance_matrix[k][k] = 0` for nodes in the subset. Without this, the distance from a node to itself remains `inf`, causing the Floyd-Warshall algorithm to produce incorrect results and potentially rejecting valid subsets.

**Why it happens:** When initializing the distance matrix with `inf`, we might assume Floyd-Warshall will naturally handle self-distances, but it won't without explicit initialization.

**Example of the bug:**
```python
# WRONG - Missing self-distance initialization
for intermediate_node in range(n):
    if (subset_mask >> intermediate_node) & 1:
        # distance_matrix[intermediate_node][intermediate_node] = 0  # MISSING!
        for start_node in range(n):
            for end_node in range(n):
                # This will fail because self-distances are still inf
                if distance_matrix[start_node][intermediate_node] + distance_matrix[intermediate_node][end_node] < distance_matrix[start_node][end_node]:
                    distance_matrix[start_node][end_node] = ...
```

**Solution:** Always set diagonal elements to 0 for nodes in the current subset before running Floyd-Warshall:
```python
# CORRECT - Set self-distances first
for intermediate_node in range(n):
    if (subset_mask >> intermediate_node) & 1:
        distance_matrix[intermediate_node][intermediate_node] = 0  # Critical!
        for start_node in range(n):
            for end_node in range(n):
                # Now the algorithm works correctly
```

### 2. **Incorrect Bit Manipulation for Subset Checking**

**Pitfall:** Using wrong bit operations like `mask & (1 << u)` instead of `(mask >> u) & 1`, or forgetting parentheses, leading to incorrect subset membership checks.

**Example of the bug:**
```python
# WRONG - Various bit manipulation errors
if mask & u:  # Wrong: compares mask with u directly
if mask >> u & 1 == 1:  # Wrong: operator precedence issue (& binds tighter than ==)
if mask << u & 1:  # Wrong: shifts in wrong direction
```

**Solution:** Use the correct pattern with proper parentheses:
```python
# CORRECT
if (mask >> u) & 1:  # Shifts mask right by u positions, then checks LSB
```

### 3. **Not Handling Multiple Edges Between Same Nodes**

**Pitfall:** Overwriting edge weights instead of taking the minimum when multiple roads exist between the same pair of branches.

**Example of the bug:**
```python
# WRONG - Overwrites previous edge weight
for u, v, weight in roads:
    if (subset_mask >> u) & 1 and (subset_mask >> v) & 1:
        distance_matrix[u][v] = weight  # Overwrites!
        distance_matrix[v][u] = weight
```

**Solution:** Always use `min()` to keep the shortest direct road:
```python
# CORRECT - Keeps minimum weight
for u, v, weight in roads:
    if (subset_mask >> u) & 1 and (subset_mask >> v) & 1:
        distance_matrix[u][v] = min(distance_matrix[u][v], weight)
        distance_matrix[v][u] = min(distance_matrix[v][u], weight)
```

### 4. **Running Floyd-Warshall on All Nodes Instead of Subset Nodes**

**Pitfall:** Using nodes outside the current subset as intermediate nodes in Floyd-Warshall, which can lead to incorrect distance calculations since those nodes are "closed."

**Example of the bug:**
```python
# WRONG - Uses all nodes as intermediates
for k in range(n):  # Should check if k is in subset!
    for i in range(n):
        for j in range(n):
            distance_matrix[i][j] = min(distance_matrix[i][j], 
                                       distance_matrix[i][k] + distance_matrix[k][j])
```

**Solution:** Only use nodes in the current subset as intermediate nodes:
```python
# CORRECT - Only uses subset nodes as intermediates
for k in range(n):
    if (subset_mask >> k) & 1:  # Check if k is in subset
        distance_matrix[k][k] = 0
        for i in range(n):
            for j in range(n):
                # Update distances
```

