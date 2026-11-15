# 2313. Minimum Flips in Binary Tree to Get Result

> Source: https://algo.monster/liteproblems/2313

## Problem Statement

You have a binary [tree](/problems/tree_intro) where each node contains a specific value with the following meaning:

**Leaf nodes** (nodes with no children):
- Value `0` represents `false`
- Value `1` represents `true`

**Non-leaf nodes** (internal nodes) represent boolean operations:
- Value `2` represents `OR` operation
- Value `3` represents `AND` operation
- Value `4` represents `XOR` operation
- Value `5` represents `NOT` operation

Special note: `NOT` nodes have exactly one child (either left or right), while all other non-leaf nodes have exactly two children.

The [tree](/problems/tree_intro) evaluates to a boolean value following these rules:
- A leaf node evaluates to its boolean value (`0` → `false`, `1` → `true`)
- A non-leaf node evaluates by first evaluating its children, then applying its boolean operation to those child values

You can perform a "flip" operation on any leaf node, which changes:
- `0` (false) to `1` (true)
- `1` (true) to `0` (false)

Given a binary [tree](/problems/tree_intro) `root` and a target boolean value `result`, find the **minimum number of leaf flips** needed to make the entire tree evaluate to `result`.

For example, if you have an `AND` node (value `3`) with two leaf children `1` and `0`, the [tree](/problems/tree_intro) evaluates to `true AND false = false`. To make it evaluate to `true`, you would need to flip the `0` leaf to `1`, requiring 1 flip operation.

The problem guarantees that it's always possible to achieve the desired `result` through some combination of flips.

## Intuition

The key insight is that at any node in the [tree](/problems/tree_intro), we need to know two pieces of information: how many flips are needed to make this subtree evaluate to `false`, and how many flips are needed to make it evaluate to `true`. Why both? Because the parent node might need either value depending on its operation and what result it's trying to achieve.

Think about it this way: if you're at an `AND` node and want the result to be `true`, you need both children to be `true`. But if you're at an `OR` node and want `true`, you only need one child to be `true`. So each child subtree should tell us: "It costs X flips to make me `false` and Y flips to make me `true`."

For leaf nodes, this is straightforward:
- If the leaf is `0` (false), it costs 0 flips to be `false` and 1 flip to be `true`
- If the leaf is `1` (true), it costs 1 flip to be `false` and 0 flips to be `true`

For non-leaf nodes, we combine the costs from children based on the boolean operation:

- **OR node** (`2`): To get `false`, both children must be `false`. To get `true`, at least one child must be `true` (so we pick the cheapest option).

- **AND node** (`3`): To get `false`, at least one child must be `false` (pick the cheapest). To get `true`, both children must be `true`.

- **XOR node** (`4`): To get `false`, children must have the same value (both `true` or both `false`). To get `true`, children must differ.

- **NOT node** (`5`): Simply inverts the child - to get `false`, we need the child to be `true`, and vice versa.

By computing both possibilities (cost to achieve `false` and cost to achieve `true`) at each node bottom-up, we can propagate this information to the root. At the root, we simply pick the cost corresponding to our desired `result`.

This [dynamic programming](/problems/dynamic_programming_intro) approach on the [tree](/problems/tree_intro) ensures we find the minimum number of flips by always choosing the cheapest option at each decision point while traversing from leaves to root.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Binary Tree](/problems/binary_tree_min_depth) patterns.**

## Solution Approach

The implementation uses [tree](/problems/tree_intro) [dynamic programming](/problems/dynamic_programming_intro) with DFS to compute the minimum flips needed. We define a recursive function `dfs(root)` that returns a tuple `(cost_to_false, cost_to_true)` representing the minimum flips needed to make the subtree rooted at `root` evaluate to `false` and `true` respectively.

**Base Case:**
- If the node is `None`, return `(inf, inf)` since an empty [tree](/problems/tree_intro) can't be evaluated.

**Leaf Nodes (values 0 or 1):**
- For value `0` (false): return `(0, 1)` - costs 0 flips to remain `false`, 1 flip to become `true`
- For value `1` (true): return `(1, 0)` - costs 1 flip to become `false`, 0 flips to remain `true`
- This is implemented as `return (x, x ^ 1)` where `x` is the node value

**Non-Leaf Nodes:**
First, recursively compute costs for left and right children:
```python
l, r = dfs(root.left), dfs(root.right)
```

Then handle each boolean operation:

1. **OR Operation (value = 2):**
   - To get `false`: Both children must be `false` → `l[0] + r[0]`
   - To get `true`: At least one child must be `true` → `min(l[0] + r[1], l[1] + r[0], l[1] + r[1])`

2. **AND Operation (value = 3):**
   - To get `false`: At least one child must be `false` → `min(l[0] + r[0], l[0] + r[1], l[1] + r[0])`
   - To get `true`: Both children must be `true` → `l[1] + r[1]`

3. **XOR Operation (value = 4):**
   - To get `false`: Children must have same value → `min(l[0] + r[0], l[1] + r[1])`
   - To get `true`: Children must have different values → `min(l[0] + r[1], l[1] + r[0])`

4. **NOT Operation (value = 5):**
   - To get `false`: Child must be `true` → `min(l[1], r[1])` (only one child exists)
   - To get `true`: Child must be `false` → `min(l[0], r[0])`

**Final Result:**
After computing the costs for the root, we return `dfs(root)[int(result)]`:
- If `result` is `False`, we need `dfs(root)[0]`
- If `result` is `True`, we need `dfs(root)[1]`

The time complexity is `O(n)` where `n` is the number of nodes, as we visit each node once. The space complexity is `O(h)` where `h` is the height of the [tree](/problems/tree_intro), due to the recursion stack.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this binary tree where we want the result to be `True`:

```
       AND (3)
       /      \
    OR (2)    true (1)
    /    \
false(0) true(1)
```

We'll compute the costs bottom-up using DFS.

**Step 1: Process the leftmost leaf `false (0)`**
- Cost to make it false: 0 flips (already false)
- Cost to make it true: 1 flip
- Returns: `(0, 1)`

**Step 2: Process the middle leaf `true (1)`**
- Cost to make it false: 1 flip
- Cost to make it true: 0 flips (already true)
- Returns: `(1, 0)`

**Step 3: Process the OR node (2)**
- Left child costs: `(0, 1)`
- Right child costs: `(1, 0)`
- For OR to be false: both children must be false → `0 + 1 = 1` flip
- For OR to be true: at least one child must be true → `min(0+0, 1+1, 1+0) = 0` flips
- Returns: `(1, 0)`

**Step 4: Process the rightmost leaf `true (1)`**
- Cost to make it false: 1 flip
- Cost to make it true: 0 flips
- Returns: `(1, 0)`

**Step 5: Process the root AND node (3)**
- Left child (OR) costs: `(1, 0)`
- Right child costs: `(1, 0)`
- For AND to be false: at least one child must be false → `min(1+1, 1+0, 0+1) = 1` flip
- For AND to be true: both children must be true → `0 + 0 = 0` flips
- Returns: `(1, 0)`

**Final Result:**
Since we want the tree to evaluate to `True`, we look at index 1 of the root's result: `0` flips needed.

This makes sense: the OR node already evaluates to true (false OR true = true), and the rightmost leaf is already true, so true AND true = true with no flips needed!

If we wanted the result to be `False` instead, we'd need 1 flip (either flip the middle leaf from true to false, making the OR evaluate to false, or flip the rightmost leaf from true to false).

## Python Solution

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right

from typing import Optional, Tuple
from math import inf

class Solution:
    def minimumFlips(self, root: Optional[TreeNode], result: bool) -> int:
        def dfs(node: Optional[TreeNode]) -> Tuple[int, int]:
            """
            Returns a tuple (cost_to_make_false, cost_to_make_true) for the subtree.

            Args:
                node: Current tree node

            Returns:
                Tuple containing minimum flips needed to make subtree evaluate to:
                - [0]: False
                - [1]: True
            """
            # Base case: empty node
            if node is None:
                return inf, inf

            node_value = node.val

            # Leaf nodes: value is 0 or 1
            if node_value in (0, 1):
                # Cost to keep current value is 0, cost to flip is 1
                return node_value, node_value ^ 1

            # Recursively calculate costs for left and right subtrees
            left_costs, right_costs = dfs(node.left), dfs(node.right)

            # OR operation (value = 2)
            if node_value == 2:
                # To get False: both children must be False
                cost_to_false = left_costs[0] + right_costs[0]
                # To get True: at least one child must be True
                cost_to_true = min(
                    left_costs[0] + right_costs[1],  # Left False, Right True
                    left_costs[1] + right_costs[0],  # Left True, Right False
                    left_costs[1] + right_costs[1]   # Both True
                )
                return cost_to_false, cost_to_true

            # AND operation (value = 3)
            if node_value == 3:
                # To get False: at least one child must be False
                cost_to_false = min(
                    left_costs[0] + right_costs[0],  # Both False
                    left_costs[0] + right_costs[1],  # Left False, Right True
                    left_costs[1] + right_costs[0]   # Left True, Right False
                )
                # To get True: both children must be True
                cost_to_true = left_costs[1] + right_costs[1]
                return cost_to_false, cost_to_true

            # XOR operation (value = 4)
            if node_value == 4:
                # To get False: both children must have same value
                cost_to_false = min(
                    left_costs[0] + right_costs[0],  # Both False
                    left_costs[1] + right_costs[1]   # Both True
                )
                # To get True: children must have different values
                cost_to_true = min(
                    left_costs[0] + right_costs[1],  # Left False, Right True
                    left_costs[1] + right_costs[0]   # Left True, Right False
                )
                return cost_to_false, cost_to_true

            # NOT operation (value = 5)
            # Only has one child (could be left or right)
            # To get False: child must be True
            cost_to_false = min(left_costs[1], right_costs[1])
            # To get True: child must be False
            cost_to_true = min(left_costs[0], right_costs[0])
            return cost_to_false, cost_to_true

        # Get the minimum flips for the entire tree
        # Index 0 for False, 1 for True
        return dfs(root)[int(result)]
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm performs a depth-first search (DFS) traversal of the binary tree. Each node is visited exactly once during the traversal. At each node, the function performs constant time operations:
- For leaf nodes (values 0 or 1), it returns values in `O(1)` time
- For internal nodes (values 2, 3, 4, or 5), it performs constant time calculations using the results from child nodes

Since we visit all `n` nodes exactly once and perform `O(1)` work at each node, the total time complexity is `O(n)`.

**Space Complexity:** `O(n)`

The space complexity comes from the recursive call stack used during the DFS traversal. In the worst case, when the tree is completely unbalanced (essentially a linked list), the recursion depth can be `O(n)`. In the best case of a balanced tree, the recursion depth would be `O(log n)`. However, considering the worst-case scenario, the space complexity is `O(n)`.

Additionally, each recursive call stores a constant amount of data (the return values and local variables), which doesn't change the overall space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of NOT Operation's Single Child**

**Pitfall:** A common mistake is assuming that NOT nodes always have their child in a specific position (either left or right). Developers often write code like:
```python
# WRONG: Assumes child is always on the left
if node_value == 5:
    cost_to_false = left_costs[1]
    cost_to_true = left_costs[0]
```

**Problem:** If the NOT node's only child is on the right side, `left_costs` would be `(inf, inf)`, leading to incorrect results.

**Solution:** Use `min()` to handle both possibilities:
```python
# CORRECT: Handles child on either side
if node_value == 5:
    cost_to_false = min(left_costs[1], right_costs[1])
    cost_to_true = min(left_costs[0], right_costs[0])
```

### 2. **Forgetting to Consider All Combinations for OR/AND Operations**

**Pitfall:** When calculating the minimum cost for OR to evaluate to true (or AND to evaluate to false), developers might forget some valid combinations:
```python
# WRONG: Missing the case where both children are true
if node_value == 2:  # OR
    cost_to_true = min(
        left_costs[0] + right_costs[1],
        left_costs[1] + right_costs[0]
    )  # Missing: left_costs[1] + right_costs[1]
```

**Problem:** This misses valid scenarios and may return a suboptimal answer.

**Solution:** Include all valid combinations:
```python
# CORRECT: Includes all cases where OR evaluates to true
cost_to_true = min(
    left_costs[0] + right_costs[1],  # F OR T = T
    left_costs[1] + right_costs[0],  # T OR F = T
    left_costs[1] + right_costs[1]   # T OR T = T
)
```

### 3. **Confusing Leaf Node Return Values**

**Pitfall:** Incorrectly calculating the flip costs for leaf nodes:
```python
# WRONG: Reversed logic
if node_value in (0, 1):
    return 1 - node_value, node_value  # Incorrect!
```

**Problem:** This reverses the costs - a node with value 0 would return (1, 0) instead of (0, 1).

**Solution:** Remember that the tuple represents `(cost_to_make_false, cost_to_make_true)`:
```python
# CORRECT: For a leaf with value x
# - Cost to make it false: x (flip if it's 1, don't flip if it's 0)
# - Cost to make it true: x^1 (flip if it's 0, don't flip if it's 1)
return node_value, node_value ^ 1
```

### 4. **Not Handling Empty/None Nodes Properly**

**Pitfall:** Returning incorrect values for None nodes or not handling them at all:
```python
# WRONG: Returns 0 costs for None nodes
if node is None:
    return 0, 0
```

**Problem:** This would incorrectly allow operations on non-existent children, producing wrong results.

**Solution:** Return infinity to indicate impossibility:
```python
# CORRECT: None nodes can't be evaluated
if node is None:
    return inf, inf
```
This ensures that when `min()` is used with NOT operations, only the existing child's cost is considered.

