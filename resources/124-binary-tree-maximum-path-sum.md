# 124. Binary Tree Maximum Path Sum

> Source: https://algo.monster/liteproblems/124

## Problem Statement

You are given a binary tree and need to find the maximum path sum among all possible paths in the tree.

A **path** is defined as a sequence of connected nodes where:
- Each pair of adjacent nodes in the sequence has an edge connecting them
- Each node can appear in the path **at most once**
- The path does **not** need to pass through the root node

The **path sum** is simply the sum of all node values in the path.

Your task is to find the maximum possible sum among all non-empty paths in the binary tree.

For example, consider a binary tree with nodes containing both positive and negative values. You need to find the path (which could go through any nodes as long as they're connected) that gives you the highest sum when you add up all the node values in that path.

The path can:
- Start and end at any nodes in the tree
- Go through parent-child relationships
- Form a "bent" path that goes up through a parent and down to another child

Given the root of the binary tree, return the maximum path sum value as an integer.

## Intuition

When we think about finding the maximum path sum in a binary tree, we need to consider what a path actually looks like. A path can start from any node, go up through some ancestors, and then go down to any descendant. The key insight is that every path has a "highest point" - a node that serves as the peak where the path changes direction from going up to going down (or just stays at that node).

For each node in the tree, we can ask: "What if this node is the highest point of the maximum path?" If a node is the highest point, then the maximum path through it would consist of:
- The node's own value
- Potentially a path going down through its left subtree
- Potentially a path going down through its right subtree

This gives us the formula: `node.val + left_path + right_path`

But here's the catch - when we're calculating what a node can contribute to its parent, we can only choose one direction to go down (either left or right), not both. Why? Because if we used both directions, we'd create a "forked" path that can't connect properly to the parent.

So we need to track two different values:
1. The maximum path sum that uses this node as the highest point (can use both children)
2. The maximum path sum this node can contribute to its parent (can only use one child)

Another crucial insight: if a subtree's contribution is negative, we're better off not including it at all. We can simply treat negative contributions as `0` since adding a negative value would only decrease our sum.

This naturally leads to a DFS solution where:
- We recursively calculate the maximum contribution from left and right subtrees
- We update our global answer with the sum through the current node as the highest point
- We return to the parent what this node can maximally contribute (node value plus the better child path)

## Solution Approach

Following the DFS approach identified from our flowchart analysis, we implement a recursive solution that explores every node in the tree.

We design a helper function `dfs(root)` that serves two purposes:
1. It calculates the maximum path sum where the current node is the highest point
2. It returns the maximum contribution this node can provide to its parent

**The recursive function logic:**

For the base case, if `root` is `None`, we return `0` since an empty subtree contributes nothing.

For each node, we recursively calculate the maximum contributions from its left and right subtrees:
```python
left = max(0, dfs(root.left))
right = max(0, dfs(root.right))
```

Notice we use `max(0, ...)` to ensure we never take negative contributions. If a subtree's best path sum is negative, we're better off not including it (treating it as `0`).

Next, we calculate the maximum path sum with the current node as the highest point:
```python
ans = max(ans, root.val + left + right)
```

This represents a path that goes through the left subtree, up to the current node, and down through the right subtree. We update our global answer `ans` if this path sum is larger than what we've seen before.

Finally, we return what this node can contribute to its parent:
```python
return root.val + max(left, right)
```

We can only choose one direction (either left or right) when contributing to the parent, so we pick the maximum of the two.

**Initialization and final result:**

We initialize `ans = -inf` to handle cases where all node values might be negative. Then we call `dfs(root)` to explore the entire tree, and return the final answer which represents the maximum path sum found across all possible paths in the tree.

The time complexity is `O(n)` where `n` is the number of nodes, as we visit each node exactly once. The space complexity is `O(h)` where `h` is the height of the tree, due to the recursion stack.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach. Consider this binary tree:

```
      -10
      /  \
     9    20
         /  \
        15   7
```

We'll use DFS to explore each node and track the maximum path sum.

**Step 1: Start DFS from root (-10)**
- First, we recursively process the left subtree (node 9)

**Step 2: Process node 9 (leaf node)**
- Left child is None, so `left = max(0, 0) = 0`
- Right child is None, so `right = max(0, 0) = 0`
- Maximum path with 9 as highest point: `9 + 0 + 0 = 9`
- Update global answer: `ans = max(-inf, 9) = 9`
- Return to parent: `9 + max(0, 0) = 9`

**Step 3: Process right subtree of root (node 20)**
- First process its left child (node 15)

**Step 4: Process node 15 (leaf node)**
- Left and right children are None, so both contributions are 0
- Maximum path with 15 as highest point: `15 + 0 + 0 = 15`
- Update global answer: `ans = max(9, 15) = 15`
- Return to parent: `15 + max(0, 0) = 15`

**Step 5: Process node 7 (leaf node)**
- Left and right children are None, so both contributions are 0
- Maximum path with 7 as highest point: `7 + 0 + 0 = 7`
- Global answer stays: `ans = max(15, 7) = 15`
- Return to parent: `7 + max(0, 0) = 7`

**Step 6: Back to node 20**
- Left contribution from node 15: `left = max(0, 15) = 15`
- Right contribution from node 7: `right = max(0, 7) = 7`
- Maximum path with 20 as highest point: `20 + 15 + 7 = 42`
- Update global answer: `ans = max(15, 42) = 42`
- Return to parent: `20 + max(15, 7) = 35`

**Step 7: Back to root node (-10)**
- Left contribution from node 9: `left = max(0, 9) = 9`
- Right contribution from node 20: `right = max(0, 35) = 35`
- Maximum path with -10 as highest point: `-10 + 9 + 35 = 34`
- Global answer stays: `ans = max(42, 34) = 42`
- Return (not used since this is root): `-10 + max(9, 35) = 25`

**Final Answer: 42**

The maximum path sum is 42, which corresponds to the path: 15 → 20 → 7. This path uses node 20 as its highest point, going down to both children. Notice how we never included the root node (-10) in our optimal path because it would decrease the sum.

## Python Solution

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right

from typing import Optional

class Solution:
    def maxPathSum(self, root: Optional[TreeNode]) -> int:
        """
        Find the maximum path sum in a binary tree.
        A path can start and end at any nodes in the tree.
      
        Args:
            root: Root node of the binary tree
          
        Returns:
            Maximum sum of any path in the tree
        """
      
        def calculate_max_gain(node: Optional[TreeNode]) -> int:
            """
            Calculate the maximum gain that can be obtained by including this node
            in a path that goes through its parent.
          
            Args:
                node: Current node being processed
              
            Returns:
                Maximum gain from this node going upward (to parent)
            """
            # Base case: empty node contributes 0
            if node is None:
                return 0
          
            # Recursively get maximum gain from left and right subtrees
            # Use max(0, ...) to ignore negative paths (we can choose not to include them)
            left_gain = max(0, calculate_max_gain(node.left))
            right_gain = max(0, calculate_max_gain(node.right))
          
            # Calculate the maximum path sum that passes through the current node
            # This path includes: left subtree -> current node -> right subtree
            current_max_path = node.val + left_gain + right_gain
          
            # Update the global maximum path sum if current path is better
            nonlocal max_sum
            max_sum = max(max_sum, current_max_path)
          
            # Return the maximum gain when continuing the path through parent
            # We can only pick one branch (left or right) when going through parent
            return node.val + max(left_gain, right_gain)
      
        # Initialize the maximum sum to negative infinity to handle negative values
        max_sum = float('-inf')
      
        # Start the DFS traversal from root
        calculate_max_gain(root)
      
        return max_sum
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm performs a depth-first search (DFS) traversal of the binary tree. Each node is visited exactly once during the traversal. At each node, the algorithm performs constant time operations:
- Computing the maximum of left subtree result and 0: `O(1)`
- Computing the maximum of right subtree result and 0: `O(1)`
- Updating the global maximum `ans`: `O(1)`
- Returning the maximum path sum through the current node: `O(1)`

Since we visit all `n` nodes exactly once and perform `O(1)` work at each node, the total time complexity is `O(n)`.

**Space Complexity:** `O(n)`

The space complexity is determined by the recursive call stack used during DFS traversal. In the worst case, the binary tree is completely unbalanced (e.g., a linked list structure where each node has only one child), leading to a recursion depth of `n`. This would require `O(n)` space on the call stack.

In the best case of a perfectly balanced binary tree, the recursion depth would be `O(log n)`, but since we consider worst-case complexity, the space complexity is `O(n)`.

Additionally, the algorithm uses only a constant amount of extra space for variables (`ans`, `left`, `right`), which doesn't affect the overall space complexity.

## Common Pitfalls

### Pitfall 1: Not Handling Negative Values Correctly

A frequent mistake is forgetting to use `max(0, ...)` when calculating the contribution from child nodes. Without this check, negative path sums from subtrees would be forced into the calculation, potentially reducing the overall maximum.

**Incorrect Implementation:**
```python
def calculate_max_gain(node):
    if node is None:
        return 0
  
    # Wrong: Always including child contributions even if negative
    left_gain = calculate_max_gain(node.left)
    right_gain = calculate_max_gain(node.right)
  
    current_max_path = node.val + left_gain + right_gain
    max_sum = max(max_sum, current_max_path)
  
    return node.val + max(left_gain, right_gain)
```

**Why this fails:** Consider a tree where a node has value 10 but its left child subtree has a maximum contribution of -20. Without the `max(0, ...)` check, we'd be forced to include -20 in our path calculation, giving us a path sum of -10 instead of just 10.

**Correct Implementation:**
```python
# Always use max(0, ...) to optionally exclude negative contributions
left_gain = max(0, calculate_max_gain(node.left))
right_gain = max(0, calculate_max_gain(node.right))
```

### Pitfall 2: Confusing Path Sum vs. Node Contribution

Another common error is returning the wrong value from the recursive function. The function calculates two different values:
1. The maximum path sum with current node as the highest point (stored in `max_sum`)
2. The maximum contribution this node can provide to its parent (the return value)

**Incorrect Implementation:**
```python
def calculate_max_gain(node):
    if node is None:
        return 0
  
    left_gain = max(0, calculate_max_gain(node.left))
    right_gain = max(0, calculate_max_gain(node.right))
  
    current_max_path = node.val + left_gain + right_gain
    max_sum = max(max_sum, current_max_path)
  
    # Wrong: Returning the full path sum instead of contribution to parent
    return current_max_path
```

**Why this fails:** When we return `current_max_path`, we're telling the parent node that it can use a path that goes through both children of the current node, which violates the tree structure. A path through the parent can only go down one branch, not both.

**Correct Implementation:**
```python
# Return only what can be contributed upward (node value + one branch)
return node.val + max(left_gain, right_gain)
```

### Pitfall 3: Incorrect Initialization of Maximum Sum

Initializing `max_sum` to 0 instead of negative infinity can cause issues when all node values in the tree are negative.

**Incorrect Implementation:**
```python
max_sum = 0  # Wrong: assumes at least one non-negative value exists
```

**Example where this fails:** A tree with a single node containing value -5 should return -5, but with `max_sum = 0`, the algorithm would incorrectly return 0.

**Correct Implementation:**
```python
max_sum = float('-inf')  # Handles all negative values correctly
```

