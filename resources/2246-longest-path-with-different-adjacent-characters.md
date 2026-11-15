# 2246. Longest Path With Different Adjacent Characters

> Source: https://algo.monster/liteproblems/2246

## Problem Statement

You are given a [tree](/problems/tree_intro) with `n` nodes numbered from `0` to `n - 1`, where node `0` is the root. The tree structure is represented by an array `parent` of size `n`, where `parent[i]` indicates the parent of node `i`. Since node `0` is the root, `parent[0] == -1`.

Additionally, you have a string `s` of length `n`, where `s[i]` represents the character assigned to node `i`.

Your task is to find the length of the longest path in the [tree](/problems/tree_intro) such that no two adjacent nodes on the path have the same character assigned to them.

A path in a [tree](/problems/tree_intro) is a sequence of nodes where each adjacent pair of nodes in the sequence has an edge between them. The length of a path is the number of nodes in the path.

For example, if you have a [tree](/problems/tree_intro) where node 0 has character 'a', node 1 (child of 0) has character 'b', and node 2 (child of 1) has character 'a', then the path from node 0 to node 2 would have length 3, and since adjacent nodes have different characters ('a' -> 'b' -> 'a'), this would be a valid path.

The goal is to find the maximum possible length among all valid paths in the [tree](/problems/tree_intro) where adjacent nodes have different characters.

## Intuition

When we think about finding the longest path in a [tree](/problems/tree_intro) with the constraint that adjacent nodes must have different characters, we need to consider what makes a valid path and how paths can be formed in a tree.

In a [tree](/problems/tree_intro), any path can be visualized as either going straight down from an ancestor to a descendant, or as a path that goes up from one descendant, passes through a common ancestor, and then goes down to another descendant. The key insight is that every path has a "highest" node - the node closest to the root that the path passes through.

For each node, we can think of it as potentially being this "highest" node in some path. If a node is the highest point in a path, then the path consists of:
1. A downward path from this node through one of its children (if any)
2. Another downward path from this node through a different child (if any)

The total length would be the sum of these two longest valid downward paths plus the node itself.

This suggests a bottom-up approach using DFS. For each node, we want to:
- Calculate the longest valid path that starts at this node and goes downward through each child
- Keep track of the two longest such paths (since we can combine at most two branches)
- Update our global answer if combining these paths creates a longer valid path

The constraint about adjacent nodes having different characters simply means that when we extend a path from a node to its child, we can only do so if `s[node] != s[child]`.

By using DFS, we can process the [tree](/problems/tree_intro) from the leaves up to the root. For each node, we:
1. Recursively get the longest downward path from each child
2. If the characters are different, we can extend that path by including the current node
3. Track the longest path we can make by joining paths from different children through the current node
4. Return the longest single downward path from this node (to be used by its parent)

This way, we consider every possible path in the [tree](/problems/tree_intro) by considering every node as a potential "turning point" or highest node in the path.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro), [Graph](/problems/graph_intro) and [Topological Sort](/problems/topo_intro) patterns.**

## Solution Approach

The implementation uses [tree](/problems/tree_intro)-shaped dynamic programming with DFS to solve this problem efficiently.

**Step 1: Build the [Tree](/problems/tree_intro) Structure**

First, we construct an adjacency list `g` from the parent array. Since `parent[i]` tells us the parent of node `i`, we iterate through all nodes (except the root) and add each node to its parent's children list:
```python
g = defaultdict(list)
for i in range(1, len(parent)):
    g[parent[i]].append(i)
```

**Step 2: DFS Traversal**

We define a recursive DFS function that processes each node and returns the length of the longest valid path going downward from that node.

For each node `i`, the function:
1. Initializes `mx = 0` to track the longest downward path from node `i`
2. Iterates through all children `j` in `g[i]`
3. Recursively calls `dfs(j)` to get the longest path from child `j`
4. Adds 1 to include the edge from `i` to `j`: `x = dfs(j) + 1`

**Step 3: Handle the Character Constraint**

When processing each child `j`, we check if `s[i] != s[j]` (parent and child have different characters):
- If true, the path from `i` through `j` is valid
- We can potentially combine this path with another valid path from `i` through a different child
- Update the global answer: `ans = max(ans, mx + x)` where `mx` is the longest path seen so far from `i`, and `x` is the current path through `j`
- Update `mx = max(mx, x)` to track the longest single downward path from `i`

**Step 4: Path Combination Logic**

The key insight is in how we combine paths:
- `mx` represents the longest valid downward path from node `i` seen so far
- `x` represents the current valid downward path through child `j`
- `mx + x` represents a complete path that goes: 
  - Up from some descendant through a previous child to node `i` (length `mx`)
  - Then down from node `i` through child `j` to another descendant (length `x`)

**Step 5: Return the Result**

The function returns `mx`, which is the longest single downward path from the current node. This value is used by the parent node in its calculations.

Finally, we add 1 to the answer (`ans + 1`) because:
- `ans` tracks the sum of path lengths (edges) from two branches
- We need to add 1 for the connecting node itself to get the total number of nodes in the path

The algorithm efficiently considers all possible paths by treating each node as a potential "turning point" where two downward paths meet, ensuring we find the globally longest valid path.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a tree with 4 nodes and the following structure:
- `parent = [-1, 0, 0, 1]` 
- `s = "abac"`

This gives us the tree:
```
    0 (a)
   / \
  1(b) 2(a)
  |
  3(c)
```

**Step 1: Build adjacency list**
- `g[0] = [1, 2]` (node 0 has children 1 and 2)
- `g[1] = [3]` (node 1 has child 3)
- `g[2] = []` (node 2 is a leaf)
- `g[3] = []` (node 3 is a leaf)

**Step 2: DFS from root (node 0)**

Starting `dfs(0)`:
- Initialize `mx = 0`, `ans = 0`
- Process child 1:
  - Call `dfs(1)`:
    - Initialize `mx = 0` for node 1
    - Process child 3:
      - Call `dfs(3)`: returns 0 (leaf node)
      - `x = 0 + 1 = 1`
      - Check `s[1] != s[3]`: 'b' != 'c' ✓ (valid)
      - Update `ans = max(0, 0 + 1) = 1`
      - Update `mx = max(0, 1) = 1`
    - Return `mx = 1`
  - Back in `dfs(0)`: `x = 1 + 1 = 2`
  - Check `s[0] != s[1]`: 'a' != 'b' ✓ (valid)
  - Update `ans = max(1, 0 + 2) = 2`
  - Update `mx = max(0, 2) = 2`

- Process child 2:
  - Call `dfs(2)`: returns 0 (leaf node)
  - `x = 0 + 1 = 1`
  - Check `s[0] != s[2]`: 'a' != 'a' ✗ (invalid - same character)
  - Don't update `ans` or `mx`

- Return `mx = 2`

**Step 3: Final answer**
- `ans = 2` (sum of edges in longest path)
- Return `ans + 1 = 3` (number of nodes)

The longest valid path is 3 → 1 → 0, which has 3 nodes with characters 'c' → 'b' → 'a', where all adjacent pairs have different characters.

The algorithm correctly identified this by:
1. Finding the path from node 1 down to node 3 (length 1)
2. Extending it through node 1 to node 0 (total length 2)
3. Converting from edge count (2) to node count (3)

## Python Solution

```python
from typing import List
from collections import defaultdict

class Solution:
    def longestPath(self, parent: List[int], s: str) -> int:
        # Build adjacency list representation of the tree
        children = defaultdict(list)
        for node_id in range(1, len(parent)):
            parent_node = parent[node_id]
            children[parent_node].append(node_id)
      
        # Variable to track the maximum path length found
        self.max_path_length = 0
      
        def dfs(current_node: int) -> int:
            """
            Depth-first search to find the longest path from current_node downward.
          
            Returns the longest valid path length starting from current_node
            and going down to any descendant.
            """
            # Track the longest valid path from this node downward
            longest_chain = 0
          
            # Explore all children of the current node
            for child_node in children[current_node]:
                # Get the longest path from the child node
                child_chain_length = dfs(child_node) + 1
              
                # Only consider this path if characters are different
                if s[current_node] != s[child_node]:
                    # Update global maximum by considering path through current node
                    # connecting two branches (longest_chain + child_chain_length)
                    self.max_path_length = max(self.max_path_length, 
                                               longest_chain + child_chain_length)
                    # Update the longest single chain from current node
                    longest_chain = max(longest_chain, child_chain_length)
          
            return longest_chain
      
        # Start DFS from the root node (node 0)
        dfs(0)
      
        # Return the result (add 1 to convert from edge count to node count)
        return self.max_path_length + 1
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm performs a depth-first search (DFS) traversal of the tree. Each node is visited exactly once during the DFS traversal. For each node `i`, we iterate through all its children in `g[i]`. Since this is a tree structure with `n` nodes and `n-1` edges, the total number of parent-child relationships processed across all nodes is `n-1`. Therefore, the overall time complexity is `O(n)`, where `n` is the number of nodes in the tree.

**Space Complexity:** `O(n)`

The space complexity consists of several components:
- The adjacency list `g` stores all parent-child relationships. In a tree with `n` nodes, there are `n-1` edges, so the adjacency list uses `O(n)` space.
- The recursive DFS call stack can go as deep as the height of the tree. In the worst case (a skewed tree), the height could be `O(n)`, requiring `O(n)` space for the call stack.
- Other variables (`ans`, `mx`, loop variables) use `O(1)` space.

Therefore, the overall space complexity is `O(n)`, where `n` is the number of nodes in the tree.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Forgetting to Handle Isolated Nodes
**Issue**: When a node has no valid children (either no children at all or all children have the same character), the code might not properly account for the node itself as a path of length 1.

**Example**: Consider a tree where the root has character 'a' and all its children also have character 'a'. The longest path should still be 1 (just the root or any single node).

**Solution**: The current implementation handles this correctly by returning `longest_chain` (which remains 0 when no valid children exist) and adding 1 at the final return. However, developers often forget this edge case when implementing from scratch.

### Pitfall 2: Incorrect Path Combination Logic
**Issue**: A common mistake is to combine the two longest paths from children without checking if both paths are valid (i.e., both children have different characters from the parent).

**Incorrect approach**:
```python
# Wrong: combining any two longest paths
first_longest = 0
second_longest = 0
for child in children[node]:
    child_length = dfs(child) + 1
    if child_length > first_longest:
        second_longest = first_longest
        first_longest = child_length
    elif child_length > second_longest:
        second_longest = child_length
# This ignores the character constraint!
```

**Solution**: Always check the character constraint before considering a path:
```python
for child_node in children[current_node]:
    child_chain_length = dfs(child_node) + 1
    # Only process if characters are different
    if s[current_node] != s[child_node]:
        self.max_path_length = max(self.max_path_length, 
                                   longest_chain + child_chain_length)
        longest_chain = max(longest_chain, child_chain_length)
```

### Pitfall 3: Confusing Edge Count with Node Count
**Issue**: The DFS function tracks the number of edges in paths, but the problem asks for the number of nodes. Forgetting to add 1 at the end will give an off-by-one error.

**Example**: A path with 3 nodes (A -> B -> C) has 2 edges. If the DFS returns 2, the final answer should be 3.

**Solution**: Always remember to add 1 when converting from edge count to node count:
```python
return self.max_path_length + 1  # Convert edges to nodes
```

### Pitfall 4: Not Initializing the Global Maximum Properly
**Issue**: If no valid paths exist (all adjacent nodes have the same character), the global maximum might remain 0, leading to returning 1 when it should return 1 anyway (for any single node).

**Solution**: The current implementation handles this correctly, but ensure that:
- The global maximum starts at 0
- The final return adds 1, ensuring at least 1 is returned (representing a single node)

