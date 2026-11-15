# 834. Sum of Distances in Tree

> Source: https://algo.monster/liteproblems/834

## Problem Statement

You have an undirected connected [tree](/problems/tree_intro) with `n` nodes labeled from `0` to `n - 1`, connected by `n - 1` edges.

Given:
- An integer `n` representing the number of nodes
- An array `edges` where each `edges[i] = [ai, bi]` represents an edge between nodes `ai` and `bi`

Your task is to return an array `answer` of length `n`, where `answer[i]` represents the sum of distances from node `i` to all other nodes in the [tree](/problems/tree_intro).

For example, if node `i` has distances of 1, 2, and 3 to three other nodes respectively, then `answer[i] = 1 + 2 + 3 = 6`.

The solution uses a two-pass DFS approach with re-rooting technique:

1. **First DFS (`dfs1`)**: Starting from node 0, calculate:
   - The sum of distances from node 0 to all other nodes (stored in `ans[0]`)
   - The size of each node's subtree (stored in `size` array)

2. **Second DFS (`dfs2`)**: Use the result from node 0 to calculate answers for all other nodes. When moving from parent node `i` to child node `j`, the sum of distances changes by the formula: `t - size[j] + n - size[j]`
   - Nodes in `j`'s subtree become closer by 1 (total decrease: `size[j]`)
   - All other nodes become farther by 1 (total increase: `n - size[j]`)

This re-rooting technique allows us to compute all answers in `O(n)` time by reusing calculations rather than running a separate DFS from each node.

## Intuition

The naive approach would be to run a DFS/BFS from each node to calculate the sum of distances to all other nodes. This would take `O(n²)` time, which is inefficient for large trees.

The key insight is that when we move from one node to an adjacent node as the new "root", the distances don't change dramatically - they follow a predictable pattern. Let's think about what happens when we shift the root from node `i` to its child `j`:

1. **Nodes in j's subtree**: All nodes in the subtree rooted at `j` get 1 step closer to the new root. If there are `size[j]` nodes in this subtree, the total distance decreases by `size[j]`.

2. **All other nodes**: The remaining `n - size[j]` nodes (including node `i` and nodes in other subtrees) become 1 step farther from the new root. The total distance increases by `n - size[j]`.

Therefore, if we know the answer for node `i` is `t`, the answer for its child `j` becomes:
`answer[j] = t - size[j] + (n - size[j]) = t - 2*size[j] + n`

This observation allows us to compute all answers with just two DFS passes:
- **First pass**: Pick any node (say node 0) as root, calculate its answer and record each node's subtree size
- **Second pass**: Use the formula above to propagate the answer from each parent to its children

This re-rooting technique transforms an `O(n²)` problem into an `O(n)` solution by cleverly reusing computations instead of starting fresh for each node.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro), [Graph](/problems/graph_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements the re-rooting technique using two DFS passes:

The snippets below use short names. In the Python solution, `g` is `graph`, `ans` is `distance_sums`, `size` is `subtree_sizes`, `dfs1` is `calculate_initial_distances`, and `dfs2` is `reroot_tree`.

**Data Structures:**
- `g`: An adjacency list (defaultdict) to represent the [tree](/problems/tree_intro) structure
- `ans`: Array to store the sum of distances for each node
- `size`: Array to store the subtree size for each node

**Implementation Steps:**

1. **Build the [Graph](/problems/graph_intro):**
   ```python
   g = defaultdict(list)
   for a, b in edges:
       g[a].append(b)
       g[b].append(a)
   ```
   Since the [tree](/problems/tree_intro) is undirected, we add edges in both directions.

2. **First DFS (`dfs1`) - Calculate from Root 0:**
   ```python
   def dfs1(i: int, fa: int, d: int):
       ans[0] += d
       size[i] = 1
       for j in g[i]:
           if j != fa:
               dfs1(j, i, d + 1)
               size[i] += size[j]
   ```
   - `i`: Current node
   - `fa`: Parent node (to avoid revisiting)
   - `d`: Current depth (distance from root 0)
 
   This DFS calculates:
   - `ans[0]`: Sum of distances from node 0 to all other nodes (by accumulating depth `d`)
   - `size[i]`: Size of subtree rooted at node `i` (including itself)

3. **Second DFS (`dfs2`) - Re-root to All Nodes:**
   ```python
   def dfs2(i: int, fa: int, t: int):
       ans[i] = t
       for j in g[i]:
           if j != fa:
               dfs2(j, i, t - size[j] + n - size[j])
   ```
   - `t`: The sum of distances for the current root `i`
   - When moving from parent `i` to child `j`, the new sum becomes:
     - `t - size[j]`: Nodes in `j`'s subtree get 1 step closer
     - `+ (n - size[j])`: All other nodes get 1 step farther
     - Simplified: `t - 2*size[j] + n`

4. **Execute the Algorithm:**
   ```python
   dfs1(0, -1, 0)  # Start from node 0 with no parent (-1) at depth 0
   dfs2(0, -1, ans[0])  # Propagate from node 0 with its calculated answer
   ```

The elegance of this solution lies in avoiding redundant calculations. Instead of computing distances from scratch for each node, we use the relationship between adjacent nodes to transform one answer into another in constant time.

## Example Walkthrough

Let's walk through a small example with 4 nodes to illustrate the solution approach.

**Example Tree:**
```
    0
   / \
  1   2
      |
      3
```
- n = 4
- edges = [[0,1], [0,2], [2,3]]

**Step 1: Build the adjacency list**
```
g[0] = [1, 2]
g[1] = [0]
g[2] = [0, 3]
g[3] = [2]
```

**Step 2: First DFS (dfs1) - Calculate from root 0**

Starting from node 0 with depth 0:

- Visit node 0 (depth=0):
  - ans[0] += 0 (total: 0)
  - size[0] = 1
  - Visit child 1 (depth=1):
    - ans[0] += 1 (total: 1)
    - size[1] = 1
    - No unvisited children
  - size[0] += size[1] = 2
  - Visit child 2 (depth=1):
    - ans[0] += 1 (total: 2)
    - size[2] = 1
    - Visit child 3 (depth=2):
      - ans[0] += 2 (total: 4)
      - size[3] = 1
      - No unvisited children
    - size[2] += size[3] = 2
  - size[0] += size[2] = 4

After first DFS:
- ans[0] = 4 (distances: 0→1=1, 0→2=1, 0→3=2, sum=4)
- size = [4, 1, 2, 1]

**Step 3: Second DFS (dfs2) - Re-root to all nodes**

Starting from node 0 with ans[0] = 4:

- Visit node 0 (t=4):
  - ans[0] = 4
  - Visit child 1:
    - New t = 4 - size[1] + (n - size[1]) = 4 - 1 + 3 = 6
    - ans[1] = 6 (distances: 1→0=1, 1→2=2, 1→3=3, sum=6)
    - No unvisited children from node 1
  - Visit child 2:
    - New t = 4 - size[2] + (n - size[2]) = 4 - 2 + 2 = 4
    - ans[2] = 4 (distances: 2→0=1, 2→1=2, 2→3=1, sum=4)
    - Visit child 3 from node 2:
      - New t = 4 - size[3] + (n - size[3]) = 4 - 1 + 3 = 6
      - ans[3] = 6 (distances: 3→0=2, 3→1=3, 3→2=1, sum=6)

**Final Result:**
ans = [4, 6, 4, 6]

**Understanding the Re-rooting Formula:**

When moving from node 0 to node 2:
- Node 2's subtree (nodes 2 and 3) get 1 step closer: -2
- Other nodes (nodes 0 and 1) get 1 step farther: +2
- Net change: -2 + 2 = 0, so ans[2] = 4

When moving from node 0 to node 1:
- Node 1's subtree (just node 1) gets 1 step closer: -1
- Other nodes (nodes 0, 2, 3) get 1 step farther: +3
- Net change: -1 + 3 = +2, so ans[1] = 6

This demonstrates how the re-rooting technique efficiently computes all answers by transforming the parent's result rather than recalculating from scratch.

## Python Solution

```python
import sys

# The DFS can be up to n = 30000 calls deep, above Python's default limit of 1000
sys.setrecursionlimit(100000)

from collections import defaultdict
from typing import List

class Solution:
    def sumOfDistancesInTree(self, n: int, edges: List[List[int]]) -> List[int]:
        # Build adjacency list representation of the tree
        graph = defaultdict(list)
        for node_a, node_b in edges:
            graph[node_a].append(node_b)
            graph[node_b].append(node_a)
      
        # Initialize arrays to store results and subtree sizes
        distance_sums = [0] * n  # distance_sums[i] = sum of distances from node i to all other nodes
        subtree_sizes = [0] * n  # subtree_sizes[i] = number of nodes in subtree rooted at i
      
        def calculate_initial_distances(node: int, parent: int, depth: int) -> None:
            """
            First DFS: Calculate the sum of distances from root (node 0) to all other nodes
            and compute the size of each subtree.
          
            Args:
                node: Current node being visited
                parent: Parent of current node (-1 for root)
                depth: Current depth from root
            """
            # Add current depth to the total sum for root node
            distance_sums[0] += depth
          
            # Initialize subtree size for current node
            subtree_sizes[node] = 1
          
            # Visit all neighbors except parent
            for neighbor in graph[node]:
                if neighbor != parent:
                    calculate_initial_distances(neighbor, node, depth + 1)
                    # Add child's subtree size to current node's subtree
                    subtree_sizes[node] += subtree_sizes[neighbor]
      
        def reroot_tree(node: int, parent: int, parent_distance_sum: int) -> None:
            """
            Second DFS: Calculate sum of distances for all nodes using rerooting technique.
            When moving from parent to child, the sum changes by:
            - Nodes in child's subtree get 1 unit closer: -subtree_sizes[child]
            - Nodes outside child's subtree get 1 unit farther: +(n - subtree_sizes[child])
          
            Args:
                node: Current node being visited
                parent: Parent of current node (-1 for root)
                parent_distance_sum: Sum of distances for the parent node
            """
            # Set the distance sum for current node
            distance_sums[node] = parent_distance_sum
          
            # Visit all neighbors except parent
            for neighbor in graph[node]:
                if neighbor != parent:
                    # Calculate new sum when rerooting from current node to neighbor
                    # Moving from node to neighbor: 
                    # - subtree_sizes[neighbor] nodes get closer by 1
                    # - (n - subtree_sizes[neighbor]) nodes get farther by 1
                    new_sum = parent_distance_sum - subtree_sizes[neighbor] + (n - subtree_sizes[neighbor])
                    reroot_tree(neighbor, node, new_sum)
      
        # First pass: Calculate distances from root and subtree sizes
        calculate_initial_distances(0, -1, 0)
      
        # Second pass: Calculate distances for all other nodes using rerooting
        reroot_tree(0, -1, distance_sums[0])
      
        return distance_sums
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm consists of two depth-first search (DFS) traversals:
- `calculate_initial_distances` (`dfs1`): Visits each node exactly once to calculate the sum of distances from node 0 and the size of each subtree. This takes `O(n)` time.
- `reroot_tree` (`dfs2`): Visits each node exactly once to compute the sum of distances for all other nodes using the rerooting technique. This also takes `O(n)` time.

Building the adjacency list from edges takes `O(n-1) = O(n)` time since a tree with `n` nodes has exactly `n-1` edges.

Overall time complexity: `O(n) + O(n) + O(n) = O(n)`

**Space Complexity:** `O(n)`

The space usage includes:
- Adjacency list `graph`: Stores all edges twice (bidirectional), requiring `O(2(n-1)) = O(n)` space
- `distance_sums` array: Stores results for all nodes, requiring `O(n)` space
- `subtree_sizes` array: Stores subtree sizes for all nodes, requiring `O(n)` space
- Recursion stack: In the worst case (skewed tree), the DFS can go up to depth `n`, requiring `O(n)` stack space

Overall space complexity: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Re-rooting Formula**
One of the most common mistakes is getting the re-rooting formula wrong when transitioning from parent to child node.

**Incorrect implementations:**
```python
# Wrong: Only considering nodes getting closer
new_sum = parent_distance_sum - subtree_sizes[neighbor]

# Wrong: Using wrong signs
new_sum = parent_distance_sum + subtree_sizes[neighbor] - (n - subtree_sizes[neighbor])

# Wrong: Not doubling the subtree size effect
new_sum = parent_distance_sum - subtree_sizes[neighbor] + n
```

**Correct implementation:**
```python
new_sum = parent_distance_sum - subtree_sizes[neighbor] + (n - subtree_sizes[neighbor])
# Or simplified:
new_sum = parent_distance_sum - 2 * subtree_sizes[neighbor] + n
```

### 2. **Forgetting to Handle Undirected Edges**
Since the tree is undirected, forgetting to add edges in both directions will cause the algorithm to fail.

**Wrong:**
```python
for a, b in edges:
    graph[a].append(b)  # Only one direction!
```

**Correct:**
```python
for a, b in edges:
    graph[a].append(b)
    graph[b].append(a)
```

### 3. **Not Tracking Parent to Avoid Cycles**
In a tree represented as an undirected graph, not tracking the parent node will cause infinite recursion.

**Wrong:**
```python
def dfs(node):
    for neighbor in graph[node]:
        dfs(neighbor)  # Will revisit parent and cause infinite loop!
```

**Correct:**
```python
def dfs(node, parent):
    for neighbor in graph[node]:
        if neighbor != parent:  # Skip the parent
            dfs(neighbor, node)
```

### 4. **Initializing Subtree Sizes Incorrectly**
Forgetting that each node counts as 1 in its own subtree size.

**Wrong:**
```python
def calculate_initial_distances(node, parent, depth):
    subtree_sizes[node] = 0  # Wrong: should start at 1
    for neighbor in graph[node]:
        if neighbor != parent:
            calculate_initial_distances(neighbor, node, depth + 1)
            subtree_sizes[node] += subtree_sizes[neighbor]
```

**Correct:**
```python
def calculate_initial_distances(node, parent, depth):
    subtree_sizes[node] = 1  # Include the node itself
    for neighbor in graph[node]:
        if neighbor != parent:
            calculate_initial_distances(neighbor, node, depth + 1)
            subtree_sizes[node] += subtree_sizes[neighbor]
```

### 5. **Using Wrong Initial Values for DFS Calls**
Starting the first DFS at depth 1 instead of 0 adds 1 for every node, so `distance_sums[0]` is too large by `n`, and every answer derived from it is too large by `n` as well.

**Wrong:**
```python
calculate_initial_distances(0, -1, 1)  # Wrong: the root is at depth 0
```

The parent argument for the root should be a value that is not a real neighbor of node 0. `-1` is the clearest choice. (Passing `0` happens to work here because node 0 is never its own neighbor, but it hides the intent.)

**Correct:**
```python
calculate_initial_distances(0, -1, 0)  # Root has no parent (-1), starts at depth 0
reroot_tree(0, -1, distance_sums[0])
```

### 6. **Accumulating Distance in Wrong Array Element**
During the first DFS, distances should be accumulated in `distance_sums[0]` (for root), not `distance_sums[node]`.

**Wrong:**
```python
def calculate_initial_distances(node, parent, depth):
    distance_sums[node] += depth  # Wrong: should accumulate in distance_sums[0]
```

**Correct:**
```python
def calculate_initial_distances(node, parent, depth):
    distance_sums[0] += depth  # Accumulate all distances to root (node 0)
```

### 7. **Recursion Depth in Python**
Both DFS functions are recursive, and a tree shaped like a path makes the recursion `n` levels deep. With `n` up to `3 × 10^4`, this exceeds Python's default recursion limit of 1000, so a plain recursive version raises `RecursionError`. The solution above calls `sys.setrecursionlimit(100000)` for this reason. Rewriting the two passes iteratively with an explicit stack also works.

