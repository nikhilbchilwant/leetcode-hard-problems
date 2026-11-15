# 2458. Height of Binary Tree After Subtree Removal Queries

> Source: https://algo.monster/liteproblems/2458

## Problem Statement

You have a binary [tree](/problems/tree_intro) with `n` nodes, where each node has a unique value from `1` to `n`. You're given an array `queries` of size `m`.

For each query `queries[i]`, you need to:
- Remove the subtree rooted at the node with value `queries[i]` from the [tree](/problems/tree_intro)
- Calculate the height of the remaining tree
- Note that `queries[i]` will never be the root node

The key point is that each query is **independent** - the [tree](/problems/tree_intro) returns to its original state before processing the next query.

The height of a [tree](/problems/tree_intro) is defined as the number of edges in the longest path from the root to any leaf node.

Your task is to return an array `answer` where `answer[i]` represents the height of the [tree](/problems/tree_intro) after removing the subtree specified by `queries[i]`.

For example, if you have a [tree](/problems/tree_intro) and remove a subtree rooted at node `x`, you need to find the maximum depth among all remaining paths from the root. This becomes the new height of the tree for that particular query.

The solution uses two DFS traversals:
1. First DFS (`f` function): Calculates the depth of each subtree and stores it in dictionary `d`, where `d[node]` represents the height of the subtree rooted at that node
2. Second DFS (`dfs` function): For each node, calculates what the [tree](/problems/tree_intro) height would be if that node's subtree were removed, considering the maximum between:
   - The current path depth up to that node's parent
   - The depth through the sibling subtree (if it exists)

The `res` array stores the final height for each node value when that node's subtree is removed, which can then be used to answer all queries efficiently.

## Intuition

When we remove a subtree from the [tree](/problems/tree_intro), we're essentially cutting off an entire branch. The key insight is that the new tree height will be determined by the **longest remaining path** from the root.

Think about what happens when we remove a subtree rooted at node `x`:
- All nodes in `x`'s subtree disappear
- The rest of the [tree](/problems/tree_intro) remains intact
- The new height is the maximum depth among all remaining paths

The naive approach would be to actually remove each queried subtree and recalculate the height, but this would be inefficient for multiple queries. Instead, we can **precompute** what the height would be if any node were removed.

Here's the clever observation: When we remove a node's subtree, the new [tree](/problems/tree_intro) height is the maximum of:
1. The depth of paths that don't go through that node at all
2. The depth from root to that node's parent (since we stop there after removal)

To efficiently calculate this, we need two pieces of information:
1. The height of each subtree (calculated in the first DFS)
2. For each node, what would be the maximum depth if we removed it

During the second DFS traversal, as we visit each node, we maintain `rest` - the maximum height achievable without using the current node's subtree. This includes:
- Paths we've already explored (from ancestors and their other children)
- The sibling subtree's contribution (if current node has a sibling)

For example, when we're at a left child, `rest` would consider:
- The current depth to this node's parent
- The maximum depth achievable through the right sibling subtree

This way, we precompute for every possible removal in just two [tree](/problems/tree_intro) traversals, making each query answerable in O(1) time by simply looking up the precomputed value.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro) and [Binary Tree](/problems/binary_tree_min_depth) patterns.**

## Solution Approach

The solution uses two DFS traversals to efficiently precompute the answer for all possible queries.

**First DFS - Calculate Subtree Heights:**

The function `f(root)` performs a post-order traversal to calculate the height of each subtree:
- Base case: If the node is `None`, return height 0
- Recursively calculate heights of left (`l`) and right (`r`) subtrees
- The current node's subtree height is `1 + max(l, r)`
- Store this in dictionary `d` where `d[node]` = height of subtree rooted at `node`

**Second DFS - Calculate Heights After Removal:**

The function `dfs(root, depth, rest)` precomputes what the [tree](/problems/tree_intro) height would be if each node's subtree were removed:

Parameters:
- `root`: Current node being processed
- `depth`: Current depth from the original root (starts at -1, incremented to 0 at root)
- `rest`: Maximum height achievable without using the current node's subtree

The logic for each node:
1. Increment `depth` by 1 (current node's depth)
2. Store `rest` in `res[root.val]` - this is the [tree](/problems/tree_intro) height if we remove this node's subtree
3. Recursively process children:
   - For left child: Pass `max(rest, depth + d[root.right])` as the new `rest`
     - This considers either the existing `rest` or the path through the right sibling
   - For right child: Pass `max(rest, depth + d[root.left])` as the new `rest`
     - This considers either the existing `rest` or the path through the left sibling

**Key Insight:** When we're at a node and about to recurse into its left child, we calculate what the maximum height would be if the left subtree didn't exist. This would be either:
- The `rest` value (paths not involving this node)
- `depth + d[root.right]` (the path going through the right child instead)

**Final Step:**

After preprocessing, each query can be answered in O(1) time by looking up `res[queries[i]]`.

**Time Complexity:** O(n) for preprocessing + O(m) for answering queries = O(n + m)
**Space Complexity:** O(n) for storing the heights and results

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this binary tree:
```
        1
       / \
      2   3
     /
    4
```

**Step 1: First DFS - Calculate Subtree Heights**

We traverse post-order and calculate heights:
- Node 4: No children, height = 0
- Node 3: No children, height = 0  
- Node 2: Has left child (4), height = 1 + max(0, 0) = 1
- Node 1: Has children (2,3), height = 1 + max(1, 0) = 2

Dictionary `d` after first DFS:
- d[1] = 2 (entire tree height)
- d[2] = 1 (subtree at node 2)
- d[3] = 0 (subtree at node 3)
- d[4] = 0 (leaf node)

**Step 2: Second DFS - Calculate Heights After Removal**

Starting at root with `dfs(1, -1, 0)`:

At node 1 (depth = 0):
- res[1] = 0 (the rest value - but node 1 is never queried as it's the root)
- For left child (2): rest = max(0, 0 + d[3]) = max(0, 0) = 0
- For right child (3): rest = max(0, 0 + d[2]) = max(0, 1) = 1

At node 2 (depth = 1, rest = 0):
- res[2] = 0 (if we remove node 2's subtree, only node 1→3 path remains, height = 1)
- Wait, let me recalculate: res[2] should be max(rest, depth + sibling) = max(0, 0 + 0) = 0
- Actually res[2] = rest = 0 means the tree height without node 2's subtree
- But we need to consider the path to node 3: the height would be 1 (root to node 3)

Let me recalculate more carefully:

At node 1 (depth = 0):
- For left child (2): rest = max(0, 0 + 1 + d[3]) = max(0, 1) = 1
  (The 1 represents going from root through right child)
- For right child (3): rest = max(0, 0 + 1 + d[2]) = max(0, 2) = 2

At node 2 (depth = 1, rest = 1):
- res[2] = 1 (removing node 2 leaves path 1→3 with height 1)
- For left child (4): rest = max(1, 1 + 0) = 1

At node 3 (depth = 1, rest = 2):
- res[3] = 2 (removing node 3 leaves path 1→2→4 with height 2)

At node 4 (depth = 2, rest = 1):
- res[4] = 1 (removing node 4 leaves paths 1→2 and 1→3, max height is 1)

**Step 3: Answer Queries**

If queries = [2, 4, 3]:
- Query 2: res[2] = 1 (tree becomes just 1→3)
- Query 4: res[4] = 1 (tree becomes 1→2 and 1→3)  
- Query 3: res[3] = 2 (tree becomes 1→2→4)

Answer = [1, 1, 2]

This demonstrates how the algorithm precomputes all possible removal scenarios in just two tree traversals, allowing each query to be answered instantly.

## Python Solution

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right

from typing import Optional, List
from collections import defaultdict

class Solution:
    def treeQueries(self, root: Optional[TreeNode], queries: List[int]) -> List[int]:
        def calculate_height(node: Optional[TreeNode]) -> int:
            """
            Calculate and store the height of each subtree.
            Height is defined as the number of nodes on the longest path from node to leaf.
            """
            if node is None:
                return 0
          
            left_height = calculate_height(node.left)
            right_height = calculate_height(node.right)
          
            # Store height of current subtree (1 + max height of children)
            node_heights[node] = 1 + max(left_height, right_height)
          
            return node_heights[node]
      
        def compute_max_depth_without_subtree(node: Optional[TreeNode], current_depth: int, max_depth_without_current: int) -> None:
            """
            For each node, compute the maximum depth of the tree if that node's subtree is removed.
          
            Args:
                node: Current node being processed
                current_depth: Depth of the current node in the tree
                max_depth_without_current: Maximum depth achievable without the current subtree
            """
            if node is None:
                return
          
            # Move to the next depth level
            current_depth += 1
          
            # Store the maximum depth if this node's subtree is removed
            max_depth_after_removal[node.val] = max_depth_without_current
          
            # Process left child: if left subtree is removed, we can still reach depths through right subtree
            compute_max_depth_without_subtree(
                node.left, 
                current_depth, 
                max(max_depth_without_current, current_depth + node_heights[node.right])
            )
          
            # Process right child: if right subtree is removed, we can still reach depths through left subtree
            compute_max_depth_without_subtree(
                node.right, 
                current_depth, 
                max(max_depth_without_current, current_depth + node_heights[node.left])
            )
      
        # Dictionary to store heights of each subtree
        node_heights = defaultdict(int)
      
        # First pass: calculate heights of all subtrees
        calculate_height(root)
      
        # Array to store the result for each node value
        # Size is number of nodes + 1 to handle 1-indexed node values
        max_depth_after_removal = [0] * (len(node_heights) + 1)
      
        # Second pass: compute maximum depth after removing each node's subtree
        compute_max_depth_without_subtree(root, -1, 0)
      
        # Return results for each query
        return [max_depth_after_removal[node_val] for node_val in queries]
```

## Time and Space Complexity

**Time Complexity:** `O(n + m)`

The algorithm consists of two main phases:
1. The first phase computes heights using function `f(root)`, which visits each node exactly once in a post-order traversal. This takes `O(n)` time where `n` is the number of nodes in the tree.
2. The second phase performs a pre-order traversal using `dfs(root, depth, rest)`, visiting each node exactly once to compute the result for each node value. This also takes `O(n)` time.
3. Finally, constructing the answer array by iterating through the queries takes `O(m)` time where `m` is the number of queries.

Total time complexity: `O(n) + O(n) + O(m) = O(n + m)`

**Space Complexity:** `O(n)`

The space usage includes:
1. The dictionary `d` stores the height for each node in the tree, requiring `O(n)` space.
2. The `res` array has size `len(d) + 1`, which is `O(n)` space.
3. The recursive call stack for both `f` and `dfs` functions can go as deep as the height of the tree, which in the worst case (skewed tree) is `O(n)`.
4. The output array for queries uses `O(m)` space, but this is typically not counted as auxiliary space since it's required for the output.

Total space complexity: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Confusing Height vs Depth Definitions**

One of the most common pitfalls is mixing up the definitions of height and depth, leading to off-by-one errors.

**The Pitfall:**
- **Height**: Number of edges from a node to its furthest leaf (or sometimes nodes)
- **Depth**: Number of edges from root to a node

In the given code, the `calculate_height` function actually returns the number of nodes (not edges) in the longest path from node to leaf, which can cause confusion.

**Solution:**
Be consistent with your definition throughout the code. If the problem defines height as edges, adjust accordingly:

```python
def calculate_height(node: Optional[TreeNode]) -> int:
    if node is None:
        return -1  # Return -1 for null nodes to count edges
  
    left_height = calculate_height(node.left)
    right_height = calculate_height(node.right)
  
    # Height in terms of edges
    node_heights[node] = 1 + max(left_height, right_height)
    return node_heights[node]
```

### 2. **Incorrect Handling of None Children**

**The Pitfall:**
When calculating `max_depth_without_current` for child nodes, accessing `node_heights[None]` returns 0 (due to defaultdict), but this might not align with your height definition.

For example, in this line:
```python
max(max_depth_without_current, current_depth + node_heights[node.right])
```

If `node.right` is None, `node_heights[None]` returns 0, which might give incorrect results depending on your height definition.

**Solution:**
Handle None children explicitly:

```python
def compute_max_depth_without_subtree(node, current_depth, max_depth_without_current):
    if node is None:
        return
  
    current_depth += 1
    max_depth_after_removal[node.val] = max_depth_without_current
  
    # Handle left child
    if node.left:
        right_subtree_contribution = 0 if node.right is None else node_heights[node.right]
        compute_max_depth_without_subtree(
            node.left, 
            current_depth, 
            max(max_depth_without_current, current_depth + right_subtree_contribution)
        )
  
    # Handle right child
    if node.right:
        left_subtree_contribution = 0 if node.left is None else node_heights[node.left]
        compute_max_depth_without_subtree(
            node.right, 
            current_depth, 
            max(max_depth_without_current, current_depth + left_subtree_contribution)
        )
```

### 3. **Array Size Miscalculation**

**The Pitfall:**
The code assumes node values are from 1 to n and creates an array of size `len(node_heights) + 1`. However, `len(node_heights)` includes the entry for None if any None nodes were processed.

**Solution:**
Count actual nodes properly or use the given `n` value:

```python
# Option 1: Count non-None nodes
actual_nodes = sum(1 for k in node_heights.keys() if k is not None)
max_depth_after_removal = [0] * (actual_nodes + 1)

# Option 2: Pass n as parameter if available
def treeQueries(self, root: Optional[TreeNode], queries: List[int], n: int) -> List[int]:
    max_depth_after_removal = [0] * (n + 1)
```

### 4. **Initial Depth Value Confusion**

**The Pitfall:**
Starting `current_depth` at -1 in the initial call might seem counterintuitive and can lead to errors if not handled consistently.

**Solution:**
Use clearer initial values and add comments:

```python
# Start with depth 0 for root, which represents 0 edges from root to root
compute_max_depth_without_subtree(root, 0, 0)

# Then adjust the increment logic inside the function
def compute_max_depth_without_subtree(node, current_depth, max_depth_without_current):
    if node is None:
        return
  
    max_depth_after_removal[node.val] = max_depth_without_current
  
    # Depth for children is current_depth + 1
    child_depth = current_depth + 1
  
    # Process children with updated depth
    compute_max_depth_without_subtree(
        node.left, 
        child_depth,
        max(max_depth_without_current, current_depth + node_heights[node.right])
    )
```

