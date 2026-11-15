# 3108. Minimum Cost Walk in Weighted Graph

> Source: https://algo.monster/liteproblems/3108

## Problem Statement

You have an undirected weighted [graph](/problems/graph_intro) with `n` vertices labeled from `0` to `n - 1`.

The [graph](/problems/graph_intro) is defined by:
- An integer `n` representing the number of vertices
- An array `edges` where each element `edges[i] = [ui, vi, wi]` represents an edge between vertices `ui` and `vi` with weight `wi`

A **walk** on the [graph](/problems/graph_intro) is a sequence of vertices and edges where:
- The walk starts and ends with a vertex
- Each edge connects consecutive vertices in the sequence
- The same edge or vertex can be visited multiple times

The **cost** of a walk from node `u` to node `v` is calculated as the bitwise AND of all edge weights traversed during the walk. For example, if the edge weights encountered are `w0, w1, w2, ..., wk`, then the cost = `w0 & w1 & w2 & ... & wk`.

You're given a 2D array `query` where each `query[i] = [si, ti]` asks for the minimum cost of any walk from vertex `si` to vertex `ti`.

For each query, you need to find:
- The minimum cost of a walk from `si` to `ti` if such a walk exists
- `-1` if no walk exists between `si` and `ti`

Return an array `answer` where `answer[i]` contains the result for `query[i]`.

**Key Insights:**
- Since we can revisit edges and vertices, if two nodes are in the same connected component, we can traverse all edges in that component
- The bitwise AND operation only decreases or stays the same when applied to positive integers
- Therefore, the minimum cost for any walk between two connected nodes would be the AND of all edge weights in their connected component
- If `si = ti`, the cost is `0` (no edges traversed)
- If `si` and `ti` are in different connected components, return `-1`

## Intuition

Let's think about what happens when we take a walk between two nodes. The key observation is that we can revisit edges and vertices as many times as we want. This means if nodes `s` and `t` are connected, we can potentially use ANY edge in their connected component as part of our walk.

Consider the bitwise AND operation. When we AND multiple numbers together, the result can only get smaller or stay the same - it never increases. For example, `5 & 3 = 1`, and if we continue `1 & 7 = 1`. Each bit position in the result can only be `1` if that bit is `1` in ALL the numbers being ANDed together.

Since we want to minimize the cost and we can use any edge in the connected component, we should use ALL edges in the component. Why? Because using more edges in the AND operation can only make the result smaller or keep it the same.

This leads us to a crucial insight: the minimum cost for any walk between two connected nodes is simply the AND of ALL edge weights in their connected component. It doesn't matter what specific path we take - as long as we can include all edges in the component (which we can, since we can revisit edges), we'll get the minimum possible cost.

Now the problem transforms into:
1. Finding which nodes belong to the same connected component (this is where Union-Find comes in)
2. Computing the AND of all edge weights within each component
3. For each query, checking if the two nodes are in the same component and returning the precomputed AND value

Special cases:
- If `s = t`, we don't need to traverse any edges, so the cost is `0`
- If `s` and `t` are in different components, there's no walk between them, so we return `-1`

**Learn more about [Union Find](/problems/dsu_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The solution uses **Union-Find (Disjoint Set Union)** data structure to efficiently manage connected components and compute the minimum cost for each component.

**Step 1: Build Connected Components**

First, we initialize a Union-Find structure with `n` nodes. The Union-Find helps us:
- Identify which nodes belong to the same connected component
- Merge components when we process edges

For each edge `(u, v, w)`, we union nodes `u` and `v` to put them in the same connected component. The union operation uses union by size optimization to keep the tree balanced.

**Step 2: Compute Component-wise AND Values**

We create an array `g` where `g[i]` will store the bitwise AND of all edge weights in the component whose root is `i`. Initially, all values are set to `-1` (all bits set to 1).

Then we traverse each edge `(u, v, w)` again:
- Find the root of the component containing `u` using `uf.find(u)`
- Update `g[root] &= w` to accumulate the AND of all edge weights in this component

Note that we only need to find the root of one endpoint since both endpoints are already in the same component after Step 1.

**Step 3: Process Queries**

For each query `(s, t)`:
- If `s == t`, return `0` (no edges traversed)
- Find the roots of `s` and `t` using `uf.find()`
- If they have the same root (same component), return `g[root]` (the precomputed AND value)
- If they have different roots (different components), return `-1` (no path exists)

**Time Complexity:**
- Building Union-Find: `O(E × α(n))` where `E` is the number of edges and `α` is the inverse Ackermann function
- Computing AND values: `O(E × α(n))`
- Processing queries: `O(Q × α(n))` where `Q` is the number of queries
- Overall: `O((E + Q) × α(n))`, which is nearly linear

**Space Complexity:** `O(n)` for the Union-Find structure and the `g` array

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- `n = 4` vertices (labeled 0, 1, 2, 3)
- `edges = [[0, 1, 7], [1, 2, 5], [0, 3, 3]]`
- `query = [[0, 2], [1, 3], [2, 3]]`

**Step 1: Build Connected Components using Union-Find**

Initial state: Each vertex is its own component
```
Component 0: {0}
Component 1: {1}
Component 2: {2}
Component 3: {3}
```

Process edges:
- Edge `[0, 1, 7]`: Union vertices 0 and 1
  ```
  Component 0-1: {0, 1}
  Component 2: {2}
  Component 3: {3}
  ```

- Edge `[1, 2, 5]`: Union vertices 1 and 2
  ```
  Component 0-1-2: {0, 1, 2}
  Component 3: {3}
  ```

- Edge `[0, 3, 3]`: Union vertices 0 and 3
  ```
  Component 0-1-2-3: {0, 1, 2, 3}
  ```

All vertices are now in the same connected component!

**Step 2: Compute Component-wise AND Values**

Initialize `g` array with -1 (all bits set): `g = [-1, -1, -1, -1]`

Process each edge and update the AND value for its component's root:
- Edge `[0, 1, 7]`: 
  - Find root of vertex 0 → root is 0
  - Update: `g[0] = -1 & 7 = 7` (binary: 111)

- Edge `[1, 2, 5]`:
  - Find root of vertex 1 → root is 0
  - Update: `g[0] = 7 & 5 = 5` (binary: 111 & 101 = 101)

- Edge `[0, 3, 3]`:
  - Find root of vertex 0 → root is 0
  - Update: `g[0] = 5 & 3 = 1` (binary: 101 & 011 = 001)

Final AND value for the component: `g[0] = 1`

**Step 3: Process Queries**

Query 1: `[0, 2]`
- s = 0, t = 2
- s ≠ t, so check if they're connected
- Find root of 0 → 0
- Find root of 2 → 0
- Same root! Return `g[0] = 1`

Query 2: `[1, 3]`
- s = 1, t = 3
- s ≠ t, so check if they're connected
- Find root of 1 → 0
- Find root of 3 → 0
- Same root! Return `g[0] = 1`

Query 3: `[2, 3]`
- s = 2, t = 3
- s ≠ t, so check if they're connected
- Find root of 2 → 0
- Find root of 3 → 0
- Same root! Return `g[0] = 1`

**Final Answer:** `[1, 1, 1]`

**Why this works:**
Since all vertices are in the same component, any walk between any two vertices can potentially traverse all three edges. The minimum possible cost is achieved by including all edges in the walk (which we can do by revisiting them), giving us `7 & 5 & 3 = 1`. This is the minimum cost for any walk between any two vertices in this component.

## Python Solution

```python
class UnionFind:
    def __init__(self, n: int):
        """Initialize Union-Find data structure with n elements."""
        # Each element is initially its own parent
        self.parent = list(range(n))
        # Track size of each component for union by rank optimization
        self.component_size = [1] * n

    def find(self, x: int) -> int:
        """Find the root of element x with path compression."""
        if self.parent[x] != x:
            # Path compression: make all nodes point directly to root
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, node_a: int, node_b: int) -> bool:
        """
        Unite the components containing node_a and node_b.
        Returns True if they were in different components, False otherwise.
        """
        root_a = self.find(node_a)
        root_b = self.find(node_b)
      
        # Already in same component
        if root_a == root_b:
            return False
      
        # Union by size: attach smaller tree under larger tree
        if self.component_size[root_a] > self.component_size[root_b]:
            self.parent[root_b] = root_a
            self.component_size[root_a] += self.component_size[root_b]
        else:
            self.parent[root_a] = root_b
            self.component_size[root_b] += self.component_size[root_a]
      
        return True


class Solution:
    def minimumCost(
        self, n: int, edges: List[List[int]], query: List[List[int]]
    ) -> List[int]:
        """
        Find minimum cost for each query where cost is the bitwise AND 
        of all edge weights in any path between two nodes.
      
        Args:
            n: Number of nodes in the graph
            edges: List of [u, v, weight] representing edges
            query: List of [start, end] pairs to find minimum cost between
      
        Returns:
            List of minimum costs for each query
        """
        # Initialize component weights array (start with all bits set)
        component_weights = [-1] * n
      
        # Build connected components using Union-Find
        union_find = UnionFind(n)
        for source_node, dest_node, _ in edges:
            union_find.union(source_node, dest_node)
      
        # Calculate the bitwise AND of all edge weights for each component
        for source_node, _, edge_weight in edges:
            component_root = union_find.find(source_node)
            # Accumulate weights using bitwise AND
            component_weights[component_root] &= edge_weight
      
        def calculate_query_cost(start_node: int, end_node: int) -> int:
            """
            Calculate minimum cost between two nodes.
          
            Returns:
                0 if nodes are the same
                Component weight if nodes are in same component
                -1 if nodes are in different components (no path exists)
            """
            # Same node - no edges needed
            if start_node == end_node:
                return 0
          
            # Find roots of both nodes
            start_root = union_find.find(start_node)
            end_root = union_find.find(end_node)
          
            # If in same component, return the AND of all edges in component
            # Otherwise return -1 (no path exists)
            return component_weights[start_root] if start_root == end_root else -1
      
        # Process all queries and return results
        return [calculate_query_cost(start, end) for start, end in query]
```

## Time and Space Complexity

**Time Complexity:** `O((n + m + q) × α(n))`

The time complexity breaks down as follows:
- **Initialization of UnionFind:** `O(n)` to create parent and size arrays
- **Processing edges (first loop):** `O(m × α(n))` where each `union` operation takes `O(α(n))` amortized time due to path compression in `find`
- **Processing edges (second loop):** `O(m × α(n))` where each edge requires a `find` operation to get the root and perform bitwise AND
- **Processing queries:** `O(q × α(n))` where each query calls function `f` which performs at most two `find` operations

The total time complexity is `O(n + m × α(n) + m × α(n) + q × α(n)) = O((n + m + q) × α(n))`, where `α(n)` is the inverse Ackermann function, which grows extremely slowly and is effectively constant for all practical values of `n`.

**Space Complexity:** `O(n)`

The space complexity consists of:
- **UnionFind data structure:** `O(n)` for storing parent array `p` and size array `size`
- **Array `g`:** `O(n)` for storing the bitwise AND results for each component root
- **Auxiliary space:** `O(1)` for variables and function call stack (with path compression, the recursion depth is bounded by `α(n)`)

The total space complexity is `O(n) + O(n) + O(1) = O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Component Weight Initialization**

A critical pitfall is initializing the component weights array with `0` instead of `-1`. This mistake completely breaks the algorithm's logic.

**Why it's wrong:**
- Bitwise AND with `0` always results in `0`, regardless of the actual edge weights
- If initialized with `0`, all queries would incorrectly return `0` for connected components

**Incorrect Implementation:**
```python
# WRONG: Starting with 0
component_weights = [0] * n  # This will make all results 0!

# Later when accumulating:
component_weights[root] &= edge_weight  # Always results in 0
```

**Correct Implementation:**
```python
# CORRECT: Starting with -1 (all bits set to 1)
component_weights = [-1] * n

# Later when accumulating:
component_weights[root] &= edge_weight  # Properly accumulates AND values
```

**Why `-1` works:**
- In binary, `-1` is represented as all bits set to `1` (in two's complement)
- When we AND with `-1`, we get the original value: `x & -1 = x`
- This allows us to properly accumulate the bitwise AND of all edge weights

### 2. **Processing Component Weights Before Union-Find is Complete**

Another common mistake is trying to calculate component weights while building the Union-Find structure, rather than in two separate passes.

**Incorrect Approach:**
```python
# WRONG: Trying to do both at once
for source, dest, weight in edges:
    union_find.union(source, dest)
    root = union_find.find(source)
    component_weights[root] &= weight  # Root might change after more unions!
```

**Problem:** The root of a component can change as more edges are processed and components merge. This leads to weights being accumulated in the wrong indices.

**Correct Approach:**
```python
# CORRECT: Two separate passes
# First pass: Build all connections
for source, dest, _ in edges:
    union_find.union(source, dest)

# Second pass: Calculate weights after structure is stable
for source, _, weight in edges:
    root = union_find.find(source)
    component_weights[root] &= weight
```

### 3. **Forgetting the Self-Loop Case**

Failing to handle the case where `start_node == end_node` is a subtle but important oversight.

**Why it matters:**
- When start and end are the same node, no edges are traversed
- The bitwise AND of an empty set of edges should be considered as having no effect on the result
- The problem specifies this should return `0`

**Correct handling:**
```python
if start_node == end_node:
    return 0  # No edges traversed, cost is 0
```

Without this check, the code would incorrectly return the component's weight even for self-loops, which violates the problem's requirements.

