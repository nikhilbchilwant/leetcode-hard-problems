# 2538. Difference Between Maximum and Minimum Price Sum

> Source: https://algo.monster/liteproblems/2538

## Problem Statement

You have an undirected [tree](/problems/tree_intro) with `n` nodes labeled from `0` to `n - 1`. The tree structure is given by an array `edges` where each `edges[i] = [ai, bi]` represents an edge between nodes `ai` and `bi`. Each node has a price value given in the array `price`, where `price[i]` is the price of node `i`.

The key concepts to understand:

1. **Path Price Sum**: For any path in the [tree](/problems/tree_intro), the price sum is the total of all node prices along that path.

2. **Rooting the [Tree](/problems/tree_intro)**: You can choose any node as the root. Once a root is chosen, all paths are considered as starting from that root and going downward through the tree.

3. **Cost of a Root**: After choosing a root node, the cost is calculated as the difference between:
   - The maximum price sum among all paths starting from the root
   - The minimum price sum among all paths starting from the root

4. **Objective**: Find the maximum possible cost across all possible choices of root nodes.

For example, if you choose node `x` as root and the paths from `x` have price sums of `[10, 15, 8, 20]`, then:
- Maximum price sum = `20`
- Minimum price sum = `8`  
- Cost for this root = `20 - 8 = 12`

You need to try all possible root nodes and return the highest cost achievable.

The solution uses DFS (Depth First Search) to explore the [tree](/problems/tree_intro). For each node, it tracks two values:
- `a`: Maximum path sum including the current node
- `b`: Maximum path sum excluding the current node (essentially the path ends at a child)

By maintaining these values and combining paths from different subtrees, the algorithm efficiently computes the maximum cost without explicitly trying every root.

## Intuition

The key insight is that we want to find the maximum difference between two paths starting from the same root. The minimum path sum from any root is always just the root itself (a path of length 0), so we're essentially looking for the maximum path sum from each potential root.

But here's the clever part: instead of trying every node as root separately (which would be inefficient), we can solve this in a single DFS traversal by thinking about paths differently.

For any node in the [tree](/problems/tree_intro), a path passing through it can be viewed as:
1. A path coming from one subtree, passing through the node, and potentially extending into another subtree
2. The combination of the node with paths from its children

This leads us to track two important values for each node:
- `a`: The maximum path sum that **includes** the current node (the path starts from somewhere below and goes up to include this node)
- `b`: The maximum path sum that **excludes** the current node (the path starts from somewhere below but doesn't include this node's price)

Why these two values? Because when we're at a node with multiple children:
- We can combine a path that includes prices from one child (`a` from child 1) with a path that excludes the current node from another child (`b` from child 2)
- Or vice versa (`b` from child 1 with `a` from child 2)

This combination gives us the maximum "spread" or difference we're looking for. The path including the node represents our maximum sum, while implicitly the minimum is just the node itself.

The beauty of this approach is that as we traverse up from the leaves, we're constantly computing the best possible paths and their differences at each node, effectively considering all possible root choices in a single pass.

For example, if we're at node `i` with children, and child `j` returns `(c, d)`:
- `c` represents the best path in child `j`'s subtree that includes `j`
- `d` represents the best path in child `j`'s subtree that excludes `j`

We can then combine these with paths from other children to find the maximum difference at node `i`, which could be a candidate for our final answer.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses DFS with a clever state tracking mechanism. Let's walk through the key components:

**Data Structure Setup:**
- Build an adjacency list `g` using `defaultdict(list)` to represent the [tree](/problems/tree_intro)
- Since the tree is undirected, add edges in both directions

**DFS Function Design:**

The `dfs(i, fa)` function processes node `i` with parent `fa` and returns two values:
- `a`: Maximum path sum that includes node `i` 
- `b`: Maximum path sum that excludes node `i`

**Base Case:**
For a leaf node (no children except parent):
- `a = price[i]` (path is just the node itself)
- `b = 0` (empty path, excluding the node)

**Recursive Case:**
For each child `j` of node `i`:
1. Recursively call `dfs(j, i)` to get `(c, d)` where:
   - `c`: Maximum path including child `j`
   - `d`: Maximum path excluding child `j`

2. Update the global answer by considering two scenarios:
   - `a + d`: Current best path including `i` combined with path excluding child `j`
   - `b + c`: Current best path excluding `i` combined with path including child `j`
 
   These combinations represent complete paths through node `i` that maximize the difference.

3. Update local values:
   - `a = max(a, price[i] + c)`: Best path including `i` is either just `i` itself or `i` plus the best path from a child
   - `b = max(b, price[i] + d)`: Best path "excluding" `i` actually means the path ends at `i` but came from below, so we add `price[i]` to the child's excluded path

**Key Insight in the Update Logic:**

When we compute `ans = max(ans, a + d, b + c)`, we're finding paths that:
- Start from one subtree
- Pass through node `i` 
- End at node `i` or continue to another subtree

The difference between including/excluding helps us track whether we're counting the node's price once (as an intermediate node) or as an endpoint.

**Final Answer:**
Start DFS from any node (here node 0) with parent -1, and return the maximum difference found across all nodes during the traversal.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a tree with 4 nodes and prices:
- Nodes: 0, 1, 2, 3
- Edges: [[0,1], [1,2], [1,3]]
- Prices: [2, 3, 5, 4]

The tree structure looks like:
```
      0 (price=2)
      |
      1 (price=3)
     / \
    2   3
  (5)   (4)
```

**Step 1: Build adjacency list**
```
g[0] = [1]
g[1] = [0, 2, 3]
g[2] = [1]
g[3] = [1]
```

**Step 2: Start DFS from node 0**

Call `dfs(0, -1)`:
- Node 0 has one child: node 1
- Need to process `dfs(1, 0)` first

**Step 3: Process dfs(1, 0)**

Node 1 has children 2 and 3 (excluding parent 0).

First, process `dfs(2, 1)`:
- Node 2 is a leaf (no children except parent 1)
- Returns `(a=5, b=0)`
- Meaning: max path including node 2 is 5, excluding is 0

Next, process `dfs(3, 1)`:
- Node 3 is a leaf
- Returns `(a=4, b=0)`

Now at node 1:
- Initialize `a = price[1] = 3`, `b = 0`

Process child 2 with `(c=5, d=0)`:
- Update answer: `max(ans, a+d, b+c) = max(0, 3+0, 0+5) = 5`
- Update `a = max(3, 3+5) = 8` (path: 2→1)
- Update `b = max(0, 3+0) = 3`

Process child 3 with `(c=4, d=0)`:
- Update answer: `max(ans, a+d, b+c) = max(5, 8+0, 3+4) = 8`
- Update `a = max(8, 3+4) = 8` (still path: 2→1)
- Update `b = max(3, 3+0) = 3`

Node 1 returns `(a=8, b=3)` to node 0.

**Step 4: Complete dfs(0, -1)**

At node 0:
- Initialize `a = price[0] = 2`, `b = 0`
- Process child 1 with `(c=8, d=3)`:
  - Update answer: `max(ans, a+d, b+c) = max(8, 2+3, 0+8) = 8`
  - Update `a = max(2, 2+8) = 10` (path: 2→1→0)
  - Update `b = max(0, 2+3) = 5`

Returns `(a=10, b=5)`

**Final Answer: 8**

The maximum cost is 8, which represents:
- Choosing node 1 as root
- Maximum path sum: 8 (path from node 2 to node 1)
- Minimum path sum: 0 (empty path, just considering paths going down from root)
- Cost = 8 - 0 = 8

The algorithm found this optimal value when processing node 1, considering the path from node 2 (price 5) through node 1 (price 3), giving us the maximum difference of 8.

## Python Solution

```python
class Solution:
    def maxOutput(self, n: int, edges: List[List[int]], price: List[int]) -> int:
        from collections import defaultdict
        from typing import List
      
        def dfs(node: int, parent: int) -> tuple[int, int]:
            """
            DFS to calculate maximum path sums.
          
            Returns:
                - max_path_with_node: Maximum path sum starting from current node (including its price)
                - max_path_without_node: Maximum path sum passing through current node (excluding its price)
            """
            # Initialize: path including current node starts with its price, path excluding it starts at 0
            max_path_with_node = price[node]
            max_path_without_node = 0
          
            # Traverse all adjacent nodes (children in the tree)
            for neighbor in adjacency_list[node]:
                if neighbor != parent:  # Skip the parent to avoid revisiting
                    # Recursively get the maximum paths from the child
                    child_with, child_without = dfs(neighbor, node)
                  
                    # Update global answer by combining paths:
                    # 1. Current path with node + child path without its starting node
                    # 2. Current path without node + child path with its starting node
                    nonlocal max_result
                    max_result = max(max_result, max_path_with_node + child_without, max_path_without_node + child_with)
                  
                    # Update local maximums:
                    # Path with current node = max of existing or current node price + child's path with child
                    max_path_with_node = max(max_path_with_node, price[node] + child_with)
                    # Path without current node = max of existing or current node price + child's path without child
                    max_path_without_node = max(max_path_without_node, price[node] + child_without)
          
            return max_path_with_node, max_path_without_node
      
        # Build adjacency list representation of the tree
        adjacency_list = defaultdict(list)
        for node_a, node_b in edges:
            adjacency_list[node_a].append(node_b)
            adjacency_list[node_b].append(node_a)
      
        # Initialize the maximum result
        max_result = 0
      
        # Start DFS from node 0 with -1 as parent (no parent for root)
        dfs(0, -1)
      
        return max_result
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm performs a depth-first search (DFS) traversal of the tree. Each node is visited exactly once during the DFS traversal. At each node, the algorithm:
- Iterates through all adjacent nodes (children in the tree)
- Performs constant time operations for updating variables `a`, `b`, and `ans`

Since this is a tree with `n` nodes and `n-1` edges, the total number of edges traversed is `O(n)`. Therefore, the overall time complexity is `O(n)`.

**Space Complexity:** `O(n)`

The space complexity consists of:
- **Graph representation:** The adjacency list `g` stores all edges. Since there are `n-1` edges in a tree and each edge is stored twice (bidirectional), this requires `O(n)` space.
- **Recursion call stack:** In the worst case (when the tree is a linear chain), the DFS recursion can go up to depth `n`, requiring `O(n)` stack space.
- **Local variables:** The variables `a`, `b`, `c`, `d`, and `ans` use `O(1)` space at each recursion level.

The dominant factors are the graph storage and recursion stack, both being `O(n)`, resulting in an overall space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Misunderstanding the Return Values

**The Problem:**
A common mistake is misinterpreting what `max_path_without_node` represents. Despite its name suggesting "without node", it actually includes the current node's price (`price[node] + child_without`). This confuses many developers who expect it to completely exclude the current node.

**Why It Happens:**
The variable naming is counterintuitive. When we say "without node", we're actually referring to paths that don't start from this node but pass through it as an intermediate point.

**The Fix:**
Rename variables to better reflect their actual meaning:
```python
def dfs(node: int, parent: int) -> tuple[int, int]:
    # Better naming
    max_path_starting_here = price[node]  # Path starts from this node
    max_path_through_here = 0  # Path passes through but doesn't start here
```

### Pitfall 2: Incorrect Answer Updates

**The Problem:**
Developers often update the answer incorrectly by trying combinations like:
```python
max_result = max(max_result, child_with + child_without)  # Wrong!
```
This doesn't properly account for the current node's contribution to the path.

**Why It Happens:**
The logic for combining paths from different subtrees through the current node is subtle. We need to ensure we're creating valid paths that include the current node exactly once.

**The Fix:**
Always combine one path that includes the current node with one that doesn't:
```python
# Correct: Combine paths properly through current node
max_result = max(max_result, 
                max_path_with_node + child_without,  # Path from one side including current
                max_path_without_node + child_with)  # Path from other side including current
```

### Pitfall 3: Edge Case with Single Node

**The Problem:**
When the tree has only one node, the DFS never updates `max_result` because there are no children to process, leading to a return value of 0 instead of the expected 0 (since max path = min path = node price).

**Why It Happens:**
The answer is only updated when processing children. With no edges, the loop never executes.

**The Fix:**
Handle the single node case explicitly:
```python
def maxOutput(self, n: int, edges: List[List[int]], price: List[int]) -> int:
    if n == 1:
        return 0  # Single node: max_path = min_path = price[0], difference = 0
  
    # Rest of the implementation...
```

### Pitfall 4: Forgetting to Use Nonlocal

**The Problem:**
Forgetting to declare `nonlocal max_result` inside the DFS function causes either an error or creates a local variable that shadows the outer one:
```python
def dfs(node, parent):
    max_result = max(max_result, ...)  # UnboundLocalError without nonlocal
```

**Why It Happens:**
Python's scoping rules require explicit declaration when modifying variables from outer scopes.

**The Fix:**
Always declare nonlocal before modifying:
```python
def dfs(node, parent):
    nonlocal max_result  # Declare before use
    # ... rest of function
```

### Complete Corrected Solution:

```python
class Solution:
    def maxOutput(self, n: int, edges: List[List[int]], price: List[int]) -> int:
        # Handle edge case
        if n == 1:
            return 0
          
        from collections import defaultdict
      
        def dfs(node: int, parent: int) -> tuple[int, int]:
            # Clearer variable names
            path_from_here = price[node]  # Max path starting from this node
            path_through_here = 0         # Max path passing through (not starting from) this node
          
            for child in adjacency_list[node]:
                if child != parent:
                    child_from, child_through = dfs(child, node)
                  
                    # Update answer with valid path combinations
                    nonlocal max_cost
                    max_cost = max(max_cost,
                                  path_from_here + child_through,
                                  path_through_here + child_from)
                  
                    # Update local maximums
                    path_from_here = max(path_from_here, price[node] + child_from)
                    path_through_here = max(path_through_here, price[node] + child_through)
          
            return path_from_here, path_through_here
      
        adjacency_list = defaultdict(list)
        for a, b in edges:
            adjacency_list[a].append(b)
            adjacency_list[b].append(a)
      
        max_cost = 0
        dfs(0, -1)
        return max_cost
```

