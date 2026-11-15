# 685. Redundant Connection II

> Source: https://algo.monster/liteproblems/685

## Problem Statement

You are given a directed [graph](/problems/graph_intro) that was originally a rooted tree with `n` nodes (labeled from `1` to `n`), but one additional directed edge has been added to it.

A rooted tree in this problem is defined as:
- A directed [graph](/problems/graph_intro) with exactly one root node
- All other nodes are descendants of the root
- Every node has exactly one parent (except the root which has no parent)

The input is provided as a 2D array `edges`, where each element `[ui, vi]` represents a directed edge from parent `ui` to child `vi`.

Your task is to find and return the one edge that can be removed so that the resulting [graph](/problems/graph_intro) becomes a valid rooted tree again. If there are multiple valid answers, return the edge that appears last in the input array.

For example, the added edge could create scenarios like:
- A node having two parents (in-degree of 2)
- A cycle in the [graph](/problems/graph_intro)

The challenge is to identify which specific edge, when removed, will restore the tree structure while maintaining all nodes connected with the proper parent-child relationships.

## Intuition

When we add one extra edge to a valid rooted tree, we break the tree property in one of two ways:

1. **A node gets two parents** (in-degree becomes 2)
2. **A cycle is formed** in the [graph](/problems/graph_intro)

Let's think about what happens in each case:

**Case 1: Node with two parents**
If a node has two parents, one of those parent-child edges must be the redundant one. But which one? Here's the key insight: if removing one of these edges still leaves a cycle in the [graph](/problems/graph_intro), then that edge cannot be the answer - we must remove the other one.

**Case 2: Only a cycle exists (no node with in-degree 2)**
This means the extra edge created a cycle without giving any node two parents. We need to find which edge in the cycle to remove. Since the problem asks for the edge that appears last in the input when multiple answers exist, we can use Union-Find to detect when we're about to create a cycle - that edge is our answer.

The Union-Find approach works perfectly here because:
- As we process edges one by one, we can detect the exact moment a cycle would be formed (when two nodes we're trying to connect already belong to the same component)
- It naturally gives us the "last occurring" edge that creates the problem

The algorithm strategy becomes:
1. First, check if any node has in-degree 2
2. If yes, identify the two edges pointing to that node. Try removing the second one - if the remaining [graph](/problems/graph_intro) is valid (no cycles), that's our answer. Otherwise, it's the first edge.
3. If no node has in-degree 2, just find the edge that creates a cycle using Union-Find

This approach elegantly handles all three possible scenarios that can occur when adding an edge to a rooted tree.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro), [Union Find](/problems/dsu_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The implementation follows our intuition by handling the two main scenarios: nodes with in-degree 2 and cycle detection.

**Step 1: Calculate In-degrees**
First, we count the in-degree of each node by iterating through all edges:
```python
ind = [0] * n
for _, v in edges:
    ind[v - 1] += 1
```

**Step 2: Find Edges Pointing to Node with In-degree 2**
We identify all edges that point to nodes with in-degree 2. These edges are our candidates for removal:
```python
dup = [i for i, (_, v) in enumerate(edges) if ind[v - 1] == 2]
```
The list `dup` will contain the indices of the two edges if a node has two parents.

**Step 3: Initialize Union-Find Structure**
We create a parent array for Union-Find, where each node initially points to itself:
```python
p = list(range(n))
```

**Step 4: Handle Case with In-degree 2**
If we found a node with two parents (`dup` is not empty):
- We skip the second edge (`dup[1]`) and try to build the [graph](/problems/graph_intro) with remaining edges
- While building, we check for cycles using Union-Find
- If we find a cycle (two nodes already in same component), it means removing `dup[1]` doesn't fix the problem, so `dup[0]` must be removed
- If no cycle is found, then `dup[1]` is the redundant edge

```python
if dup:
    for i, (u, v) in enumerate(edges):
        if i == dup[1]:  # Skip the second edge
            continue
        pu, pv = find(u - 1), find(v - 1)
        if pu == pv:  # Cycle detected
            return edges[dup[0]]
        p[pu] = pv  # Union operation
    return edges[dup[1]]
```

**Step 5: Handle Case with Only Cycle (No In-degree 2)**
If no node has in-degree 2, we simply find which edge creates a cycle:
```python
for i, (u, v) in enumerate(edges):
    pu, pv = find(u - 1), find(v - 1)
    if pu == pv:  # Found the edge creating cycle
        return edges[i]
    p[pu] = pv  # Union operation
```

**The Find Function**
The find function with path compression optimizes the Union-Find structure:
```python
def find(x: int) -> int:
    if p[x] != x:
        p[x] = find(p[x])  # Path compression
    return p[x]
```

This approach handles all scenarios in `O(n log n)` time, where `n` is the number of edges (see the complexity section for details).

## Example Walkthrough

Let's walk through a concrete example to understand how the solution works.

**Example 1: Node with Two Parents**

Consider the graph with edges: `[[1,2], [1,3], [2,3]]`

```
Initial tree should be:
    1
   / \
  2   3

But with the extra edge [2,3], node 3 has two parents:
    1
   / \
  2→→3
```

**Step 1: Calculate in-degrees**
- Node 1: in-degree = 0
- Node 2: in-degree = 1 (from edge [1,2])
- Node 3: in-degree = 2 (from edges [1,3] and [2,3])

**Step 2: Find candidate edges**
Node 3 has in-degree 2, so we identify the two edges pointing to it:
- `dup[0] = 1` (edge [1,3] at index 1)
- `dup[1] = 2` (edge [2,3] at index 2)

**Step 3: Try removing the second edge [2,3]**
Build the graph without edge at index 2:
- Process edge [1,2]: Union nodes 1 and 2 (no cycle)
- Process edge [1,3]: Union nodes 1 and 3 (no cycle)
- Result: Valid tree formed!

Since no cycle was detected, the answer is edge [2,3].

**Example 2: Cycle Without Two Parents**

Consider the graph with edges: `[[1,2], [2,3], [3,1]]`

```
This creates a cycle:
1 → 2
↑   ↓
3 ← 
```

**Step 1: Calculate in-degrees**
- All nodes have in-degree = 1
- No node has two parents

**Step 2: No edges in `dup` list**

**Step 3: Detect cycle using Union-Find**
- Process edge [1,2]: Union nodes 1 and 2 (parent[1] = 2)
- Process edge [2,3]: Union nodes 2 and 3 (parent[2] = 3)
- Process edge [3,1]: 
  - Find(3) = 3, Find(1) = 3 (following parent pointers: 1→2→3)
  - They're already in the same component! Cycle detected.

The answer is edge [3,1] - the edge that would create the cycle.

**Example 3: Two Parents with Hidden Cycle**

Consider edges: `[[2,1], [3,1], [4,2], [1,4]]`

```
Node 1 has two parents (2 and 3), and 1 → 4 → 2 → 1 is a cycle:
    3
    ↓
    1 → 4
    ↑   ↓
    2 ←─┘
```

**Step 1: In-degrees**
- Node 1: in-degree = 2 (from [2,1] and [3,1])

**Step 2: Candidate edges**
- `dup[0] = 0` (edge [2,1])
- `dup[1] = 1` (edge [3,1])

**Step 3: Try removing [3,1]**
Build the graph without the edge at index 1 (Find results are shown as node labels):
- Process [2,1]: Find(2) = 2, Find(1) = 1 → different, union them (node 2 now points to node 1)
- Process [4,2]: Find(4) = 4, Find(2) = 1 → different, union them (node 4 now points to node 1)
- Process [1,4]: Find(1) = 1, Find(4) = 1 → same component, **cycle detected**

Removing [3,1] leaves the cycle 1 → 4 → 2 → 1 in place, so [3,1] cannot be the answer. The algorithm returns `edges[dup[0]] = [2,1]`. After removing [2,1], the edges [3,1], [1,4], [4,2] form the valid tree 3 → 1 → 4 → 2.

These examples demonstrate how the algorithm handles different scenarios: simple two-parent cases, pure cycles, and complex situations where both conditions exist.

## Python Solution

```python
class Solution:
    def findRedundantDirectedConnection(self, edges: List[List[int]]) -> List[int]:
        """
        Find the redundant edge in a directed graph that should be removed.
        The graph should form a rooted tree after removal.
      
        Args:
            edges: List of directed edges [parent, child]
      
        Returns:
            The redundant edge to remove
        """
      
        def find_root(node: int) -> int:
            """Find the root of the node using path compression."""
            if parent[node] != node:
                parent[node] = find_root(parent[node])  # Path compression
            return parent[node]
      
        n = len(edges)
      
        # Count in-degrees for each node (to find nodes with 2 parents)
        in_degree = [0] * n
        for parent_node, child_node in edges:
            in_degree[child_node - 1] += 1
      
        # Find edges where the child has 2 parents (in-degree = 2)
        duplicate_parent_edges = [
            i for i, (parent_node, child_node) in enumerate(edges) 
            if in_degree[child_node - 1] == 2
        ]
      
        # Initialize Union-Find parent array
        parent = list(range(n))
      
        if duplicate_parent_edges:
            # Case 1: A node has two parents
            # Try removing the second edge first
            for i, (u, v) in enumerate(edges):
                if i == duplicate_parent_edges[1]:
                    continue  # Skip the second duplicate edge
              
                root_u, root_v = find_root(u - 1), find_root(v - 1)
              
                if root_u == root_v:
                    # Found a cycle without the second edge
                    # So the first duplicate edge must be removed
                    return edges[duplicate_parent_edges[0]]
              
                # Union the components
                parent[root_u] = root_v
          
            # No cycle found after removing second edge
            # So the second edge should be removed
            return edges[duplicate_parent_edges[1]]
      
        # Case 2: No node has two parents, just find the cycle
        for i, (u, v) in enumerate(edges):
            root_u, root_v = find_root(u - 1), find_root(v - 1)
          
            if root_u == root_v:
                # Found the edge that creates a cycle
                return edges[i]
          
            # Union the components
            parent[root_u] = root_v
      
        # This line should never be reached given valid input
        return []
```

## Time and Space Complexity

The time complexity is `O(n log n)`, where `n` is the number of edges (which equals the number of nodes). Counting in-degrees and building the candidate list take `O(n)`. The union-find pass processes each edge once with two `find` calls and one union. The `find` function uses path compression, but the union step always links `root_u` under `root_v` without union by rank or size, and path compression alone gives an amortized `O(log n)` per operation, so the pass costs `O(n log n)`. Adding union by rank or size would lower this to `O(n × α(n))`, where `α(n)` is the inverse Ackermann function.

The space complexity is `O(n)`. The algorithm uses:
- Array `in_degree` of size `n` to track in-degrees
- List `duplicate_parent_edges`, which stores at most 2 indices (constant space)
- Array `parent` of size `n` for the union-find parent pointers
- The recursive call stack for `find_root`, which can reach `O(n)` depth in the worst case before path compression flattens a long chain

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Handling the Two-Parent Case Priority**

**Pitfall:** When a node has two parents, many solutions incorrectly assume that simply removing the later of the two edges will work. If removing that edge still leaves a cycle, then the other edge must be the one to remove.

**Example scenario:**
```
Edges: [[2,1], [3,1], [4,2], [1,4]]
Node 1 has two parents (2 and 3)
```
If we remove the later edge [3,1], the edges 2→1, 4→2, 1→4 still form the cycle 1→4→2→1, so the result is not a tree.
If we remove the earlier edge [2,1], we get 3→1→4→2, a valid rooted tree, so the answer is [2,1].
(When both removals give a valid tree, as in `[[1,2], [1,3], [2,3]]`, the later edge [2,3] is returned.)

**Solution:** Always test by removing the second (later) edge first. Only if this creates a cycle should we conclude the first edge needs removal:

```python
# Correct approach - try removing dup[1] first
if duplicate_parent_edges:
    for i, (u, v) in enumerate(edges):
        if i == duplicate_parent_edges[1]:  # Skip second edge
            continue
        root_u, root_v = find_root(u - 1), find_root(v - 1)
        if root_u == root_v:  # Cycle found
            return edges[duplicate_parent_edges[0]]  # Remove first edge instead
    return edges[duplicate_parent_edges[1]]  # No cycle, remove second edge
```

### 2. **Off-by-One Errors with Node Indexing**

**Pitfall:** The problem states nodes are labeled from 1 to n, but arrays in most programming languages are 0-indexed. Forgetting to adjust indices leads to incorrect parent tracking or array out-of-bounds errors.

**Incorrect code:**
```python
# Wrong - treating 1-indexed nodes as 0-indexed
in_degree[child_node] += 1  # Should be child_node - 1
root_u = find_root(u)  # Should be u - 1
```

**Solution:** Consistently convert between 1-indexed nodes and 0-indexed arrays:

```python
# Correct indexing
in_degree[child_node - 1] += 1
root_u, root_v = find_root(u - 1), find_root(v - 1)
```

### 3. **Not Using Path Compression in Union-Find**

**Pitfall:** Implementing find() without path compression can lead to O(n) time complexity per find operation in worst case, making the overall solution O(n²) instead of O(n log n).

**Inefficient implementation:**
```python
def find_root(node):
    while parent[node] != node:
        node = parent[node]
    return node
```

**Solution:** Always implement path compression to maintain near-constant time complexity:

```python
def find_root(node: int) -> int:
    if parent[node] != node:
        parent[node] = find_root(parent[node])  # Path compression
    return parent[node]
```

### 4. **Returning Wrong Edge Format**

**Pitfall:** Returning the edge index instead of the actual edge, or modifying the edge before returning it.

**Wrong:**
```python
return duplicate_parent_edges[0]  # Returns index, not edge
return [edges[i][0] - 1, edges[i][1] - 1]  # Modifies values
```

**Solution:** Always return the original edge from the input array:

```python
return edges[duplicate_parent_edges[0]]  # Correct - returns actual edge
```

