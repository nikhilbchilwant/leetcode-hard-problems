# 1373. Maximum Sum BST in Binary Tree

> Source: https://algo.monster/liteproblems/1373

## Problem Statement

You are given the root of a binary [tree](/problems/tree_intro). Your task is to find the maximum sum of all node values in any subtree that is also a valid [Binary Search Tree](/problems/bst_intro) (BST).

A Binary Search [Tree](/problems/tree_intro) must satisfy these properties:
- Every node in the left subtree has a value **less than** the current node's value
- Every node in the right subtree has a value **greater than** the current node's value  
- Both left and right subtrees must themselves be valid BSTs

The problem asks you to examine all possible subtrees within the given binary [tree](/problems/tree_intro), identify which ones are valid BSTs, calculate the sum of node values for each valid BST, and return the maximum sum found.

For example, if you have a [tree](/problems/tree_intro) where only certain subtrees are valid BSTs, you would:
1. Check each subtree to determine if it's a valid BST
2. For each valid BST found, calculate the sum of all its node values
3. Track the maximum sum encountered
4. Return this maximum value

Note that:
- A single node by itself is considered a valid BST with sum equal to its value
- An empty subtree is considered a valid BST with sum `0`
- The entire [tree](/problems/tree_intro) might not be a BST, but some of its subtrees could be
- You need to find the maximum sum among all valid BST subtrees

## Intuition

To find the maximum sum of any BST subtree, we need to check every possible subtree in the given binary [tree](/problems/tree_intro). The key insight is that we can't just check if a subtree is a BST in isolation - we need information from the children to make this determination.

Think about what makes a [tree](/problems/tree_intro) a BST: the root value must be greater than all values in its left subtree and less than all values in its right subtree. This means we need to know the **maximum value** from the left subtree and the **minimum value** from the right subtree to validate the BST property at any node.

This naturally leads to a bottom-up approach using DFS. Starting from the leaves and working our way up allows us to gather the necessary information from children before processing the parent.

For each node, we need to track four pieces of information:
1. **Is this subtree a valid BST?** - We can only form a BST at the current node if both children are BSTs
2. **What's the minimum value in this subtree?** - Parent nodes need this to check BST validity
3. **What's the maximum value in this subtree?** - Parent nodes need this to check BST validity  
4. **What's the sum of all nodes in this subtree?** - We need this to track the maximum sum

The beauty of this approach is that when we're at a node and both its subtrees are valid BSTs, we just need to check if `left_max < node.val < right_min`. If this condition holds, the current subtree is also a BST, and we can calculate its sum as `left_sum + right_sum + node.val`.

For empty nodes (base case), we return values that won't interfere with BST validation: `(True, +∞, -∞, 0)`. The infinity values ensure that any comparison with actual node values will satisfy the BST conditions.

As we traverse the [tree](/problems/tree_intro), we keep updating a global maximum whenever we find a valid BST, ensuring we capture the largest sum among all valid BST subtrees.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro), [Binary Search Tree](/problems/bst_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Binary Tree](/problems/binary_tree_min_depth) patterns.**

## Solution Approach

The solution implements a DFS traversal that returns a quadruple `(bst, mi, mx, s)` for each subtree, where:
- `bst`: Binary flag (1 if the subtree is a BST, 0 otherwise)
- `mi`: Minimum value in the subtree
- `mx`: Maximum value in the subtree
- `s`: Sum of all nodes in the subtree

**Base Case:**
When we encounter an empty node (`root is None`), we return `(1, inf, -inf, 0)`. The infinity values are chosen strategically - any real node value will be less than `inf` and greater than `-inf`, ensuring the BST property checks pass for leaf nodes.

**Recursive Case:**
For each node, we:
1. Recursively call `dfs` on the left child, getting `(lbst, lmi, lmx, ls)`
2. Recursively call `dfs` on the right child, getting `(rbst, rmi, rmx, rs)`

**BST Validation:**
A subtree rooted at the current node is a BST if and only if:
- Both left and right subtrees are BSTs: `lbst == 1` and `rbst == 1`
- The current node's value is greater than the maximum value in the left subtree: `lmx < root.val`
- The current node's value is less than the minimum value in the right subtree: `root.val < rmi`

**When the subtree is a valid BST:**
- Calculate the sum: `s = ls + rs + root.val`
- Update the global maximum: `ans = max(ans, s)`
- Return the tuple representing this valid BST:
  - BST flag: `1`
  - Minimum value: `min(lmi, root.val)` (could be from left subtree or current node)
  - Maximum value: `max(rmx, root.val)` (could be from right subtree or current node)
  - Sum: `s`

**When the subtree is NOT a valid BST:**
Return `(0, 0, 0, 0)`. The actual values for min, max, and sum don't matter since the BST flag is 0, indicating this subtree won't be considered by parent nodes.

**Global Answer Tracking:**
The variable `ans` is maintained globally (using `nonlocal` in Python) to track the maximum sum encountered across all valid BST subtrees. It's initialized to 0 and updated whenever we find a valid BST.

The algorithm visits each node exactly once, making it O(n) in time complexity, where n is the number of nodes in the [tree](/problems/tree_intro).

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this binary tree:
```
        4
       / \
      2   5
     / \
    1   3
```

We'll trace through the DFS traversal to see how it identifies valid BSTs and finds the maximum sum.

**Step 1: DFS reaches leaf node 1**
- Since node 1 has no children, both left and right return `(1, inf, -inf, 0)`
- Check BST validity: `-inf < 1 < inf` ✓ (both children are BSTs)
- This subtree is a BST with sum = 0 + 0 + 1 = 1
- Update global max: ans = max(0, 1) = 1
- Return: `(1, 1, 1, 1)`

**Step 2: DFS reaches leaf node 3**
- Similar to node 1, returns `(1, 3, 3, 3)`
- Update global max: ans = max(1, 3) = 3

**Step 3: DFS processes node 2**
- Left child (node 1) returned: `(1, 1, 1, 1)`
- Right child (node 3) returned: `(1, 3, 3, 3)`
- Check BST validity:
  - Both children are BSTs ✓
  - Left max (1) < current (2) ✓
  - Current (2) < right min (3) ✓
- This subtree is a BST with sum = 1 + 3 + 2 = 6
- Update global max: ans = max(3, 6) = 6
- Return: `(1, min(1,2), max(3,2), 6)` = `(1, 1, 3, 6)`

**Step 4: DFS reaches leaf node 5**
- Returns `(1, 5, 5, 5)`
- Update global max: ans = max(6, 5) = 6

**Step 5: DFS processes root node 4**
- Left child (node 2) returned: `(1, 1, 3, 6)`
- Right child (node 5) returned: `(1, 5, 5, 5)`
- Check BST validity:
  - Both children are BSTs ✓
  - Left max (3) < current (4) ✓
  - Current (4) < right min (5) ✓
- This entire tree is a BST with sum = 6 + 5 + 4 = 15
- Update global max: ans = max(6, 15) = 15
- Return: `(1, 1, 5, 15)`

**Result:** The maximum sum BST is the entire tree with sum = 15

Now let's consider a tree where not all subtrees are BSTs:
```
        5
       / \
      3   8
     /
    7
```

**Processing node 7:** Valid BST, sum = 7, ans = 7

**Processing node 3:**
- Left child (node 7) returned: `(1, 7, 7, 7)`
- Right child (null) returned: `(1, inf, -inf, 0)`
- Check BST validity:
  - Both children are BSTs ✓
  - Left max (7) < current (3) ✗ (FAILS!)
- This subtree is NOT a BST
- Return: `(0, 0, 0, 0)`

**Processing node 8:** Valid BST, sum = 8, ans = max(7, 8) = 8

**Processing root node 5:**
- Left child (node 3) returned: `(0, 0, 0, 0)` (not a BST)
- Right child (node 8) returned: `(1, 8, 8, 8)`
- Check BST validity:
  - Left child is not a BST ✗ (FAILS!)
- This subtree is NOT a BST
- Return: `(0, 0, 0, 0)`

**Result:** The maximum sum BST is the single node 8 with sum = 8

This walkthrough demonstrates how the algorithm:
1. Uses bottom-up traversal to gather information from children first
2. Validates BST properties using min/max values from subtrees
3. Tracks the maximum sum across all valid BST subtrees
4. Handles cases where parent trees aren't BSTs even if some children are

## Python Solution

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right

class Solution:
    def maxSumBST(self, root: Optional[TreeNode]) -> int:
        """
        Find the maximum sum of all keys of any sub-tree which is also a Binary Search Tree (BST).
      
        Args:
            root: Root node of the binary tree
          
        Returns:
            Maximum sum of any BST subtree in the tree
        """
      
        def dfs(node: Optional[TreeNode]) -> tuple[int, int, int, int]:
            """
            Perform DFS to check if subtree is BST and calculate sum.
          
            Args:
                node: Current node being processed
              
            Returns:
                Tuple of (is_bst, min_value, max_value, sum_of_subtree)
                - is_bst: 1 if subtree is valid BST, 0 otherwise
                - min_value: Minimum value in the subtree
                - max_value: Maximum value in the subtree
                - sum_of_subtree: Sum of all nodes in the subtree
            """
            # Base case: empty node is a valid BST
            if node is None:
                return 1, float('inf'), float('-inf'), 0
          
            # Recursively process left and right subtrees
            left_is_bst, left_min, left_max, left_sum = dfs(node.left)
            right_is_bst, right_min, right_max, right_sum = dfs(node.right)
          
            # Check if current subtree forms a valid BST
            # Conditions: both subtrees are BSTs and current node value is in valid range
            if left_is_bst and right_is_bst and left_max < node.val < right_min:
                # Calculate sum of current BST subtree
                current_sum = left_sum + right_sum + node.val
              
                # Update global maximum sum
                nonlocal max_sum
                max_sum = max(max_sum, current_sum)
              
                # Return BST properties for current subtree
                return (1, 
                        min(left_min, node.val),  # Minimum value in subtree
                        max(right_max, node.val),  # Maximum value in subtree
                        current_sum)               # Sum of subtree
          
            # Current subtree is not a valid BST
            return 0, 0, 0, 0
      
        # Initialize maximum sum to 0 (empty BST has sum 0)
        max_sum = 0
      
        # Start DFS traversal from root
        dfs(root)
      
        return max_sum
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm performs a depth-first search (DFS) traversal of the binary tree. Each node is visited exactly once during the traversal. At each node, the algorithm performs constant time operations:
- Comparing values (`lmx < root.val < rmi`)
- Computing the sum (`ls + rs + root.val`)
- Finding minimum and maximum values (`min(lmi, root.val)`, `max(rmx, root.val)`)
- Updating the answer (`max(ans, s)`)

Since we visit each of the `n` nodes exactly once and perform `O(1)` operations at each node, the overall time complexity is `O(n)`.

**Space Complexity:** `O(n)`

The space complexity comes from two sources:
1. **Recursion call stack:** In the worst case (skewed tree), the recursion depth can be `O(n)`, requiring `O(n)` space on the call stack.
2. **Return values:** Each recursive call returns a tuple of 4 values, but these are `O(1)` space per call and don't accumulate.

In the best case (balanced tree), the recursion depth would be `O(log n)`, but we must consider the worst case. Therefore, the space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Infinity Values for Base Case**

**Pitfall:** Using the wrong infinity values in the base case can break the BST validation logic. For example, if you return `(1, -inf, inf, 0)` for a null node instead of `(1, inf, -inf, 0)`, the BST property checks will fail.

**Why it happens:** The logic requires that:
- Any node value should be greater than the maximum of its left subtree
- Any node value should be less than the minimum of its right subtree

When a node has no left child, we need `left_max = -inf` so that `left_max < node.val` always passes. Similarly, when there's no right child, we need `right_min = inf` so that `node.val < right_min` always passes.

**Solution:**
```python
# Correct base case
if node is None:
    return 1, float('inf'), float('-inf'), 0
  
# NOT this:
# return 1, float('-inf'), float('inf'), 0  # Wrong!
```

### 2. **Forgetting to Handle Single Node BSTs**

**Pitfall:** Not updating the maximum sum when processing leaf nodes or single-node subtrees, which are valid BSTs by definition.

**Why it happens:** Developers might focus on complex subtrees and forget that a single node is also a valid BST that should be considered for the maximum sum.

**Solution:** The current implementation handles this correctly by updating `max_sum` for every valid BST, including single nodes. The key is placing the update inside the BST validation block:

```python
if left_is_bst and right_is_bst and left_max < node.val < right_min:
    current_sum = left_sum + right_sum + node.val
    max_sum = max(max_sum, current_sum)  # Updates for ALL valid BSTs, including leaves
```

### 3. **Using Weak BST Validation (≤ instead of <)**

**Pitfall:** Using `<=` or `>=` in BST validation instead of strict inequalities, which would allow duplicate values and violate the BST property as defined in this problem.

**Why it happens:** Different BST definitions exist - some allow duplicates on one side. However, this problem specifically requires strict inequalities.

**Solution:**
```python
# Correct: strict inequalities
if left_is_bst and right_is_bst and left_max < node.val < right_min:
  
# Wrong: would allow duplicates
# if left_is_bst and right_is_bst and left_max <= node.val <= right_min:
```

### 4. **Not Considering Empty Tree or Negative Values**

**Pitfall:** Initializing `max_sum` to a negative value or not handling trees with all negative values correctly.

**Why it happens:** Developers might assume all node values are positive or that there's always at least one valid BST with positive sum.

**Solution:** Initialize `max_sum = 0` because:
- An empty subtree is a valid BST with sum 0
- If all BST subtrees have negative sums, we should return 0 (the empty BST)
- This handles the edge case where the entire tree has negative values

```python
# Correct initialization
max_sum = 0

# Not this:
# max_sum = float('-inf')  # Would return negative sum even when empty BST (sum=0) is better
```

