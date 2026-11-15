# 2440. Create Components With Same Value

> Source: https://algo.monster/liteproblems/2440

## Problem Statement

You have an undirected [tree](/problems/tree_intro) with `n` nodes labeled from `0` to `n - 1`. Each node `i` has a value given by `nums[i]`, where `nums` is a 0-indexed array of length `n`. The tree structure is defined by an array `edges` of length `n - 1`, where each `edges[i] = [ai, bi]` represents an edge between nodes `ai` and `bi`.

Your task is to delete some edges from the [tree](/problems/tree_intro), which will split it into multiple connected components. The value of each component is defined as the sum of all `nums[i]` values for nodes `i` within that component.

The goal is to find the **maximum** number of edges you can delete such that all resulting connected components have the **same value**.

For example, if you delete `k` edges, you'll create `k + 1` connected components. All these components must have equal sums of their node values.

The problem asks you to return the maximum number of edges that can be deleted while maintaining this equal-value constraint across all components.

Key points to understand:
- The [tree](/problems/tree_intro) initially has `n` nodes and `n - 1` edges (as it's a tree)
- Deleting edges creates multiple subtrees (connected components)
- Each component's value is the sum of its nodes' values
- All components must have exactly the same total value
- You want to maximize the number of deleted edges (which maximizes the number of components)

## Intuition

Let's think about what happens when we delete edges. If we delete `k-1` edges, we create exactly `k` connected components. For all components to have equal value, each component must have value `total_sum / k`.

The key insight is that `total_sum / k` must be an integer for a valid partition to exist. This immediately limits our choices for `k` - we can only consider values of `k` that evenly divide the total sum.

Now, how do we check if we can actually create `k` components with equal values? We need to verify that we can partition the [tree](/problems/tree_intro) into subtrees where each has exactly value `target = total_sum / k`.

Here's where DFS becomes crucial. Starting from any node (say node 0), we can traverse the [tree](/problems/tree_intro) and calculate subtree sums. When we encounter a subtree whose sum equals our target value, we know we can "cut" it off from its parent - this represents deleting an edge. We mark this by returning 0 to the parent, indicating this subtree has been separated.

But what if a subtree's sum exceeds the target? This means our current choice of `k` is invalid - we can't create equal-valued components with this target. We return -1 to signal failure.

To maximize the number of edges deleted (which means maximizing `k`), we try values of `k` from large to small. The first valid `k` we find gives us the maximum number of components possible, and thus the maximum number of edges we can delete is `k-1`.

There's also an optimization: the maximum possible `k` is bounded by `min(n, total_sum / max_value)`, because each component must contain at least one node, and the smallest possible component value is at least the maximum node value in the [tree](/problems/tree_intro).

The beauty of this approach is that DFS naturally handles the [tree](/problems/tree_intro) structure and allows us to make greedy decisions - whenever we find a valid subtree of the target size, we can immediately "cut" it, knowing this won't prevent us from finding other valid partitions in the remaining tree.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro) and [Math](/problems/math-basics) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**1. Graph Representation**

First, we build an adjacency list representation of the [tree](/problems/tree_intro) using a `defaultdict(list)`. For each edge `[a, b]`, we add `b` to `g[a]` and `a` to `g[b]` since it's an undirected tree.

**2. Enumeration of k (Number of Components)**

We calculate:
- `s = sum(nums)`: Total sum of all node values
- `mx = max(nums)`: Maximum value among all nodes
- Starting `k = min(n, s // mx)`: The maximum possible number of components

We iterate `k` from this maximum value down to 2 (since `k=1` means no edges deleted). For each `k`, we check if `s % k == 0`. If not, we skip this `k` since we can't divide the sum evenly.

**3. DFS Function for Validation**

For each valid `k`, we set `t = s // k` as our target component value and use DFS to check if we can partition the [tree](/problems/tree_intro):

```python
def dfs(i, fa):
    x = nums[i]  # Start with current node's value
    for j in g[i]:
        if j != fa:  # Avoid going back to parent
            y = dfs(j, i)  # Get subtree sum from child
            if y == -1:
                return -1  # Propagate failure
            x += y  # Add child's contribution
    if x > t:
        return -1  # Subtree sum exceeds target - invalid
    return x if x < t else 0  # Return sum or 0 if we found a complete component
```

The DFS logic:
- Start with the current node's value
- Recursively calculate sums from all children (excluding parent to avoid cycles)
- If any child returns `-1`, propagate the failure upward
- Add valid child contributions to current subtree sum
- If subtree sum equals target `t`, return `0` (indicating this subtree can be cut off)
- If subtree sum is less than `t`, return the sum (this subtree needs to be combined with others)
- If subtree sum exceeds `t`, return `-1` (invalid partition)

**4. Finding the Answer**

We start DFS from node 0 with parent `-1`. If `dfs(0, -1)` returns `0`, it means we successfully partitioned the entire [tree](/problems/tree_intro) into `k` components of equal value. We return `k - 1` as the number of edges deleted.

If no valid partition is found for any `k`, we return `0` (no edges can be deleted while maintaining equal component values).

**Time Complexity**: `O(n * d)` where `d` is the number of divisors of `s`. In the worst case, this could be `O(n²)`.

**Space Complexity**: `O(n)` for the adjacency list and recursion stack.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Example:** 
- `nums = [6, 2, 2, 2, 6]` (5 nodes with these values)
- `edges = [[0,1], [1,2], [1,3], [3,4]]`

The tree structure looks like:
```
    0(6)
     |
    1(2)
   /   \
  2(2)  3(2)
         |
        4(6)
```

**Step 1: Calculate total sum and maximum value**
- Total sum `s = 6 + 2 + 2 + 2 + 6 = 18`
- Maximum value `mx = 6`
- Maximum possible components `k = min(5, 18//6) = min(5, 3) = 3`

**Step 2: Try k = 3 (attempting to create 3 components)**
- Check if `18 % 3 == 0` ✓ (Yes, it divides evenly)
- Target value per component: `t = 18/3 = 6`

**Step 3: Run DFS from node 0**

Let's trace through the DFS:

1. Start at node 0 (value = 6)
   - Visit child 1
 
2. At node 1 (value = 2)
   - Visit child 2
     - Node 2 is a leaf with value 2
     - Returns 2 (less than target 6)
   - Visit child 3
     - At node 3 (value = 2)
       - Visit child 4
         - Node 4 is a leaf with value 6
         - Returns 0 (equals target, can be cut off!)
       - Node 3: `x = 2 + 0 = 2`
       - Returns 2 (less than target)
   - Node 1: `x = 2 + 2 + 2 = 6`
   - Returns 0 (equals target, can be cut off!)
 
3. Back at node 0: `x = 6 + 0 = 6`
   - Returns 0 (equals target)

**Result:** DFS returns 0, meaning we successfully created 3 components:
- Component 1: Node 0 alone (sum = 6)
- Component 2: Nodes 1, 2, 3 (sum = 2+2+2 = 6)  
- Component 3: Node 4 alone (sum = 6)

We deleted 2 edges to create these 3 components.

**Step 4: Check if we can do better with k = 2**
- Target would be `18/2 = 9`
- Running DFS would fail because no valid partition exists where each component sums to 9

**Final Answer:** Maximum edges we can delete = 2

The algorithm efficiently found that we can partition the tree into 3 equal-valued components by removing the edges between nodes 0-1 and nodes 3-4.

## Python Solution

```python
from typing import List
from collections import defaultdict

class Solution:
    def componentValue(self, nums: List[int], edges: List[List[int]]) -> int:
        """
        Find the maximum number of edges that can be deleted from the tree
        such that all resulting components have equal sum.
      
        Args:
            nums: Node values
            edges: Tree edges connecting nodes
          
        Returns:
            Maximum number of edges that can be deleted
        """
      
        def dfs(node: int, parent: int) -> int:
            """
            Perform DFS to check if we can partition the subtree rooted at 'node'
            into components of size 'target_sum'.
          
            Args:
                node: Current node
                parent: Parent of current node
              
            Returns:
                Remainder sum after forming complete components, or -1 if impossible
            """
            # Start with the value of current node
            subtree_sum = nums[node]
          
            # Process all children
            for neighbor in adjacency_list[node]:
                if neighbor != parent:
                    # Get the remainder from child's subtree
                    child_remainder = dfs(neighbor, node)
                  
                    # If child subtree cannot be partitioned properly, propagate failure
                    if child_remainder == -1:
                        return -1
                  
                    # Add child's remainder to current subtree sum
                    subtree_sum += child_remainder
          
            # If subtree sum exceeds target, partitioning is impossible
            if subtree_sum > target_sum:
                return -1
          
            # If subtree sum equals target, we can cut here (return 0 remainder)
            # Otherwise, return the current sum as remainder for parent
            return 0 if subtree_sum == target_sum else subtree_sum
      
        # Build adjacency list for the tree
        num_nodes = len(nums)
        adjacency_list = defaultdict(list)
        for node_a, node_b in edges:
            adjacency_list[node_a].append(node_b)
            adjacency_list[node_b].append(node_a)
      
        # Calculate total sum and maximum value
        total_sum = sum(nums)
        max_value = max(nums)
      
        # Try different numbers of components (from maximum possible down to 2)
        # Each component must have sum = total_sum / num_components
        max_components = min(num_nodes, total_sum // max_value)
      
        for num_components in range(max_components, 1, -1):
            # Check if total sum can be evenly divided into num_components parts
            if total_sum % num_components == 0:
                target_sum = total_sum // num_components
              
                # Try to partition the tree with this target sum
                if dfs(0, -1) == 0:
                    # If successful, return number of cuts (components - 1)
                    return num_components - 1
      
        # No valid partition found, return 0 (no edges can be deleted)
        return 0
```

## Time and Space Complexity

**Time Complexity:** `O(n × √s)`

The algorithm iterates through possible values of `k` from `min(n, s // mx)` down to `2`. The key insight is that we're looking for divisors of `s` (the sum of all node values). The number of divisors of `s` is bounded by `O(√s)` in the average case, though it can be `O(s^(1/3))` in the worst case for highly composite numbers. However, the practical bound used here is `O(√s)`.

For each valid divisor (when `s % k == 0`), we perform a DFS traversal of the tree which takes `O(n)` time, where `n` is the number of nodes. The DFS visits each node exactly once and processes each edge twice (once from each direction).

Therefore, the total time complexity is `O(number of divisors × DFS cost) = O(√s × n) = O(n × √s)`.

**Space Complexity:** `O(n)`

The space complexity consists of:
- The adjacency list `g` which stores all edges: `O(n)` space (since a tree with `n` nodes has `n-1` edges)
- The recursive call stack for DFS which can go up to depth `O(n)` in the worst case (when the tree is a linear chain)
- Other variables like `s`, `mx`, `t`, `k` which use `O(1)` space

The dominant factor is the recursion stack and adjacency list, both of which are `O(n)`, giving us a total space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Component Detection in DFS**

A common mistake is misunderstanding when a subtree forms a complete component. Consider this incorrect implementation:

```python
def dfs(node, parent):
    subtree_sum = nums[node]
    for neighbor in adjacency_list[node]:
        if neighbor != parent:
            child_sum = dfs(neighbor, node)
            if child_sum == -1:
                return -1
            subtree_sum += child_sum
  
    # WRONG: Only checking if sum equals target
    if subtree_sum == target_sum:
        return 0
    elif subtree_sum < target_sum:
        return subtree_sum
    else:
        return -1
```

**Problem**: This doesn't properly handle the case where we need to accumulate values from multiple subtrees before reaching the target sum.

**Solution**: The correct approach returns `0` when a complete component is found (allowing the parent to "cut" this subtree), returns the accumulated sum when it's less than target (to be combined with sibling subtrees), and returns `-1` for invalid cases.

### 2. **Off-by-One Error in Return Value**

Many solutions incorrectly return `num_components` instead of `num_components - 1`:

```python
# WRONG
if dfs(0, -1) == 0:
    return num_components  # This returns number of components, not edges deleted!

# CORRECT
if dfs(0, -1) == 0:
    return num_components - 1  # Number of edges deleted = components - 1
```

**Problem**: The question asks for the number of edges deleted, not the number of components created.

**Solution**: Remember that deleting `k` edges creates `k + 1` components, so if we want `num_components` components, we need to delete `num_components - 1` edges.

### 3. **Inefficient Component Count Enumeration**

Starting from `k = 2` and going up is inefficient:

```python
# INEFFICIENT
for num_components in range(2, num_nodes + 1):
    if total_sum % num_components == 0:
        target_sum = total_sum // num_components
        if dfs(0, -1) == 0:
            return num_components - 1
```

**Problem**: This approach finds the minimum number of edges to delete rather than the maximum. Also, it doesn't consider the constraint that each component must have at least one node.

**Solution**: Start from the maximum possible number of components and work downward. The maximum is bounded by `min(n, total_sum // max_value)` since each component needs at least the value of the largest node.

### 4. **Missing Edge Case: Single Node Components**

Not considering that the maximum component value constrains the minimum component sum:

```python
# WRONG: Not considering node value constraints
max_components = num_nodes

# CORRECT: Each component must contain at least the max node value
max_components = min(num_nodes, total_sum // max(nums))
```

**Problem**: If we have a node with value 10 and total sum is 30, we can have at most 3 components (each with sum 10), not more.

**Solution**: Calculate the upper bound on components as `min(n, total_sum // max(nums))` to ensure each component can accommodate at least the largest node value.

### 5. **Graph Construction Error with Self-Loops**

Be careful not to create duplicate edges or self-loops when building the adjacency list:

```python
# POTENTIAL ISSUE if edges contain duplicates or self-loops
for node_a, node_b in edges:
    adjacency_list[node_a].append(node_b)
    adjacency_list[node_b].append(node_a)
```

**Problem**: While the problem guarantees a valid tree (no cycles, n-1 edges for n nodes), some implementations might not handle edge cases properly during testing.

**Solution**: The current implementation is correct for valid tree input. However, for robustness, you could add validation or use a set-based adjacency list if duplicate edges were possible.

