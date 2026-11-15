# 1697. Checking Existence of Edge Length Limited Paths

> Source: https://algo.monster/liteproblems/1697

## Problem Statement

You are given an undirected [graph](/problems/graph_intro) with `n` nodes and a list of edges `edgeList`. Each edge is represented as `edgeList[i] = [ui, vi, disi]`, which means there is an edge between nodes `ui` and `vi` with a distance (or weight) of `disi`. Note that there can be multiple edges between the same pair of nodes.

You are also given an array of queries `queries`, where each query is represented as `queries[j] = [pj, qj, limitj]`. For each query, you need to determine if there exists a path between nodes `pj` and `qj` such that **every edge** on that path has a distance **strictly less than** `limitj`.

Your task is to return a boolean array `answer` where:
- `answer.length == queries.length`
- `answer[j]` is `true` if there exists a valid path for `queries[j]` (a path from `pj` to `qj` where all edges have distances < `limitj`)
- `answer[j]` is `false` otherwise

## Intuition

The challenge here is that we need to answer multiple queries about path existence with different edge weight limits. A naive approach would be to build a new [graph](/problems/graph_intro) for each query containing only edges below the limit, then check connectivity - but this would be inefficient.

The key observation is that if we have a path between two nodes using edges with weights less than some limit `L1`, then we automatically have a path for any larger limit `L2 > L1`. This monotonic property suggests we can process queries in increasing order of their limits.

Think of it like gradually "activating" edges as we increase the limit threshold. Start with no edges (limit = 0), then progressively add edges as the limit increases. For a query with limit `L`, we only need edges with weight strictly less than `L`.

This leads us to the following approach:
1. Sort edges by weight in ascending order
2. Sort queries by limit in ascending order (but remember their original positions for the answer)
3. Process queries one by one, and for each query:
   - Add all edges with weight less than the current limit to our [graph](/problems/graph_intro)
   - Check if the two query nodes are connected

For connectivity checking, Union-Find is perfect because:
- We only need to add edges (union operation)
- We need to check if two nodes are in the same connected component (find operation)
- Both operations are nearly O(1) with path compression

By processing queries offline (not in their original order) and [sorting](/problems/sorting_summary) them alongside edges, we ensure each edge is added at most once, making the solution efficient. The answer array is filled according to the original query indices to maintain the correct output order.

**Learn more about [Union Find](/problems/dsu_intro), [Graph](/problems/graph_intro), [Two Pointers](/problems/two_pointers_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The solution implements the offline query processing technique with Union-Find data structure. Here's how the implementation works:

**1. Union-Find Setup:**
- Initialize a parent array `p` where `p[i] = i` for all nodes from `0` to `n-1`
- Implement `find(x)` function with path compression to find the root of node `x`

**2. Sort Operations:**
- Sort `edgeList` by edge weight in ascending order using `edgeList.sort(key=lambda x: x[2])`
- Sort queries by limit value while preserving original indices using `sorted(enumerate(queries), key=lambda x: x[1][2])`
  - The enumeration keeps track of original query index `i`
  - [Sorting](/problems/sorting_summary) is done based on `x[1][2]` which is the limit value

**3. Process Queries in Order:**
- Initialize answer array `ans` with `False` values
- Use pointer `j` to track the current position in sorted `edgeList`
- For each sorted query `(i, (a, b, limit))`:

  **Add eligible edges:**
  ```python
  while j < len(edgeList) and edgeList[j][2] < limit:
      u, v, _ = edgeList[j]
      p[find(u)] = find(v)  # Union operation
      j += 1
  ```
  - Add all edges with weight strictly less than current `limit`
  - Union the endpoints of each added edge

  **Check connectivity:**
  ```python
  ans[i] = find(a) == find(b)
  ```
  - Check if nodes `a` and `b` belong to the same component
  - Store result at original index `i` in answer array

**4. Return Results:**
- Return the `ans` array with results in the original query order

The time complexity is `O(E log E + Q log Q + (E + Q) × α(n))` where `E` is the number of edges, `Q` is the number of queries, and `α` is the inverse Ackermann function (practically constant). The space complexity is `O(n)` for the Union-Find parent array.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- `n = 4` nodes (labeled 0, 1, 2, 3)
- `edgeList = [[0,1,2], [1,2,4], [2,3,1]]` 
  - Edge 0↔1 with weight 2
  - Edge 1↔2 with weight 4  
  - Edge 2↔3 with weight 1
- `queries = [[0,2,3], [2,3,5], [0,3,2]]`
  - Query 0: Can we go from 0 to 2 using only edges with weight < 3?
  - Query 1: Can we go from 2 to 3 using only edges with weight < 5?
  - Query 2: Can we go from 0 to 3 using only edges with weight < 2?

**Step 1: Initialize Union-Find**
```
parent = [0, 1, 2, 3]  (each node is its own parent initially)
```

**Step 2: Sort edges by weight**
```
sortedEdges = [[2,3,1], [0,1,2], [1,2,4]]
              (weight 1)  (weight 2)  (weight 4)
```

**Step 3: Sort queries by limit (keeping original indices)**
```
sortedQueries = [(2, [0,3,2]), (0, [0,2,3]), (1, [2,3,5])]
                 limit=2        limit=3        limit=5
                 (original index 2, 0, 1)
```

**Step 4: Process queries in sorted order**

**Processing Query index 2: [0,3,2] with limit 2**
- Add edges with weight < 2: Only edge [2,3,1] qualifies
- Union(2,3): parent becomes [0, 1, 3, 3]
- Check find(0) == find(3)? → 0 ≠ 3 → FALSE
- `ans[2] = False`

**Processing Query index 0: [0,2,3] with limit 3**
- Add edges with weight < 3: Edge [0,1,2] qualifies (edge [2,3,1] already added)
- Union(0,1): parent becomes [1, 1, 3, 3]
- Check find(0) == find(2)? → 1 ≠ 3 → FALSE
- `ans[0] = False`

**Processing Query index 1: [2,3,5] with limit 5**
- Add edges with weight < 5: Edge [1,2,4] qualifies (others already added)
- Union(1,2): parent becomes [1, 1, 1, 3] → then [1, 1, 1, 1] after path compression
- Check find(2) == find(3)? → 1 == 1 → TRUE
- `ans[1] = True`

**Step 5: Return answer in original order**
```
ans = [False, True, False]
```

This means:
- Query 0: No path from 0 to 2 using only edges < 3 (would need edge with weight 4)
- Query 1: Path exists from 2 to 3 using edges < 5 (all edges qualify)
- Query 2: No path from 0 to 3 using only edges < 2 (would need at least edge with weight 2)

## Python Solution

```python
from typing import List

class Solution:
    def distanceLimitedPathsExist(
        self, n: int, edgeList: List[List[int]], queries: List[List[int]]
    ) -> List[bool]:
        """
        Determines if there exists a path between two nodes with all edge weights
        less than a given limit for each query.
      
        Args:
            n: Number of nodes in the graph (0 to n-1)
            edgeList: List of edges [u, v, distance]
            queries: List of queries [node_a, node_b, limit]
      
        Returns:
            List of booleans indicating if a valid path exists for each query
        """
      
        def find(node: int) -> int:
            """
            Find operation for Union-Find with path compression.
            Returns the root parent of the given node.
            """
            if parent[node] != node:
                # Path compression: make node point directly to root
                parent[node] = find(parent[node])
            return parent[node]
      
        # Initialize Union-Find parent array where each node is its own parent
        parent = list(range(n))
      
        # Sort edges by weight in ascending order
        edgeList.sort(key=lambda edge: edge[2])
      
        # Initialize result array
        result = [False] * len(queries)
      
        # Process queries in ascending order of limit while preserving original indices
        # enumerate(queries) gives (index, query), then sort by query's limit (index 2)
        sorted_queries = sorted(enumerate(queries), key=lambda x: x[1][2])
      
        edge_index = 0
      
        for query_index, (node_a, node_b, distance_limit) in sorted_queries:
            # Add all edges with weight less than current query's limit to Union-Find
            while edge_index < len(edgeList) and edgeList[edge_index][2] < distance_limit:
                u, v, _ = edgeList[edge_index]
                # Union operation: connect the two components
                parent[find(u)] = find(v)
                edge_index += 1
          
            # Check if node_a and node_b are in the same connected component
            result[query_index] = find(node_a) == find(node_b)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(m × log m + q × log q + (m + q) × α(n))`

Breaking down the time complexity:
- Sorting `edgeList` by weight takes `O(m × log m)` where `m` is the number of edges
- Sorting queries by their limit value takes `O(q × log q)` where `q` is the number of queries  
- The main loop iterates through all `q` queries
- The inner while loop processes each edge at most once across all iterations, contributing `O(m)` operations
- Each union-find operation (find and union) takes `O(α(n))` amortized time, where `α` is the inverse Ackermann function
- Total union operations: at most `m` (one per edge)
- Total find operations: at most `2m + 2q` (two finds per union, two finds per query check)

Since `α(n)` grows extremely slowly and is effectively constant for practical values of `n`, the time complexity can be simplified to `O(m × log m + q × log q)` as stated in the reference answer.

**Space Complexity:** `O(n + m + q)`

Breaking down the space complexity:
- The parent array `p` uses `O(n)` space
- Storing the sorted `edgeList` uses `O(m)` space (though this could be considered as modifying input in-place)
- The sorted queries with indices use `O(q)` space for the enumeration
- The answer array `ans` uses `O(q)` space
- The recursion stack for path compression in the worst case uses `O(n)` space before compression occurs

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Comparison Operator in Edge Addition**

**Pitfall:** Using `<=` instead of `<` when comparing edge weights with the query limit.

```python
# WRONG: This includes edges with weight equal to limit
while edge_index < len(edgeList) and edgeList[edge_index][2] <= distance_limit:
    u, v, _ = edgeList[edge_index]
    parent[find(u)] = find(v)
    edge_index += 1
```

**Why it's wrong:** The problem specifically states that every edge on the path must have distance **strictly less than** `limitj`. Including edges with weight equal to the limit violates this constraint.

**Solution:** Always use strict inequality `<`:
```python
# CORRECT: Only includes edges with weight strictly less than limit
while edge_index < len(edgeList) and edgeList[edge_index][2] < distance_limit:
    u, v, _ = edgeList[edge_index]
    parent[find(u)] = find(v)
    edge_index += 1
```

### 2. **Forgetting to Preserve Original Query Order**

**Pitfall:** Sorting queries without tracking their original indices, leading to results in wrong positions.

```python
# WRONG: Loses track of original query positions
sorted_queries = sorted(queries, key=lambda x: x[2])
result = []
for node_a, node_b, distance_limit in sorted_queries:
    # Process query...
    result.append(find(node_a) == find(node_b))  # Wrong order!
```

**Solution:** Use `enumerate()` to track original indices:
```python
# CORRECT: Preserves original query indices
sorted_queries = sorted(enumerate(queries), key=lambda x: x[1][2])
result = [False] * len(queries)
for query_index, (node_a, node_b, distance_limit) in sorted_queries:
    # Process query...
    result[query_index] = find(node_a) == find(node_b)  # Correct position!
```

### 3. **Resetting Union-Find Between Queries**

**Pitfall:** Reinitializing the Union-Find structure for each query, which defeats the purpose of the offline processing optimization.

```python
# WRONG: Inefficient approach that resets Union-Find for each query
for query in queries:
    parent = list(range(n))  # Reset Union-Find - WRONG!
    for edge in edgeList:
        if edge[2] < query[2]:
            # Add edge...
```

**Solution:** Maintain a single Union-Find structure throughout and process queries in sorted order:
```python
# CORRECT: Initialize Union-Find once and reuse it
parent = list(range(n))  # Initialize only once
edge_index = 0
for query_index, (node_a, node_b, distance_limit) in sorted_queries:
    # Incrementally add edges without resetting
    while edge_index < len(edgeList) and edgeList[edge_index][2] < distance_limit:
        # Add edge...
        edge_index += 1
```

### 4. **Incorrect Union Operation**

**Pitfall:** Directly modifying parent array without using find operation, breaking the Union-Find structure.

```python
# WRONG: Direct assignment without finding roots
parent[u] = v  # or parent[u] = parent[v]
```

**Solution:** Always union the roots of the components:
```python
# CORRECT: Union the roots of both components
parent[find(u)] = find(v)
```

