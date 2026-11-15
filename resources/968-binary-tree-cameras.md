# 968. Binary Tree Cameras

> Source: https://algo.monster/liteproblems/968

## Problem Statement

You have a binary [tree](/problems/tree_intro) and need to install security cameras on some of its nodes. Each camera placed on a node can monitor:
- The node itself
- Its parent node (if it exists)
- Its immediate children nodes (if they exist)

Your task is to find the minimum number of cameras needed to monitor every single node in the [tree](/problems/tree_intro).

For example, if you place a camera on a node, it will cover that node, the node directly above it, and the nodes directly below it. The goal is to strategically place cameras so that all nodes are monitored while using as few cameras as possible.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with three states for each node:
- State `a`: The node has a camera installed on it
- State `b`: The node doesn't have a camera but is monitored by at least one of its children
- State `c`: The node doesn't have a camera and is not monitored by its children

The algorithm recursively calculates the minimum cameras needed for each state at every node, considering:
- If a node has a camera (`a`), both its children are automatically monitored, so we can choose any state for them
- If a node is monitored by its children (`b`), at least one child must have a camera
- If a node is not monitored by its children (`c`), both children must be monitored by their own children

The final answer is the minimum between placing a camera at the root or having the root monitored by its children.

## Intuition

The key insight is to think about this problem from the bottom up - starting from the leaves and working our way to the root. Why? Because a camera's coverage extends both upward and downward, creating dependencies between parent and child nodes.

Consider what happens at each node. We have three possible situations:
1. Place a camera at this node
2. Don't place a camera, but get monitored by a child's camera
3. Don't place a camera and don't get monitored by children (rely on parent's camera)

The greedy approach of always placing cameras as high as possible doesn't work because we might miss optimal placements. Instead, we need to consider all possibilities and choose the best one.

Here's the critical observation: if we know the optimal camera placement for all subtrees, we can determine the optimal placement for the current node. This naturally leads to a recursive solution where we solve smaller subproblems first.

For each node, we need to track different states because the parent's decision depends on what happened with the children. If a child has a camera, it monitors its parent. If a child is unmonitored, the parent must either have a camera or be monitored by its other child.

The clever part is recognizing that we need exactly three states:
- State `a`: Having a camera here covers this node, its parent, and its children
- State `b`: Being monitored by children means at least one child must have a camera
- State `c`: Not being monitored by children means we're counting on the parent to monitor us

By calculating the minimum cameras needed for each state at every node and propagating these values up the [tree](/problems/tree_intro), we can find the global minimum. The final answer is the minimum between placing a camera at the root (state `a`) or having the root monitored by its children (state `b`). We can't leave the root unmonitored (state `c`) since it has no parent to monitor it.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Binary Tree](/problems/binary_tree_min_depth) patterns.**

## Solution Approach

The implementation uses [dynamic programming](/problems/dynamic_programming_intro) with DFS to solve this problem. Let's walk through how the solution works:

**Function Structure:**
We define a recursive `dfs(root)` function that returns three values for each node:
- `a`: Minimum cameras needed if we place a camera at this node
- `b`: Minimum cameras needed if this node is monitored by its children
- `c`: Minimum cameras needed if this node is not monitored by its children

**Base Case:**
When we reach a null node, we return `[inf, 0, 0]`. The `inf` for state `a` indicates it's impossible to place a camera on a null node. The zeros for states `b` and `c` mean no cameras are needed for non-existent nodes.

**Recursive Case:**
For each non-null node, we first recursively solve for the left and right subtrees:
- `la, lb, lc = dfs(root.left)` 
- `ra, rb, rc = dfs(root.right)`

Then we calculate the three states for the current node:

1. **State `a` (camera at current node):** 
   - Formula: `a = min(la, lb, lc) + min(ra, rb, rc) + 1`
   - If we place a camera here, both children are automatically monitored
   - We can choose any state for the children (hence the `min`), plus 1 for the current camera

2. **State `b` (monitored by children):**
   - Formula: `b = min(la + rb, lb + ra, la + ra)`
   - At least one child must have a camera to monitor the parent
   - We consider three scenarios: left has camera, right has camera, or both have cameras
   - We pick the minimum among these options

3. **State `c` (not monitored by children):**
   - Formula: `c = lb + rb`
   - Neither child has a camera pointing at this node
   - Both children must be monitored by their own children (state `b`)

**Final Answer:**
After computing the states for the root, we return `min(a, b)`. We exclude state `c` because the root has no parent to monitor it, so it must either have a camera or be monitored by its children.

The algorithm runs in `O(n)` time since we visit each node once, and uses `O(h)` space for the recursion stack, where `h` is the height of the [tree](/problems/tree_intro).

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach. Consider this binary tree:

```
       1
      / \
     2   3
    /
   4
```

We'll compute the three states (a, b, c) for each node, working from the leaves up.

**Step 1: Process leaf node 4**
- Node 4 has no children, so `dfs(null)` returns `[inf, 0, 0]` for both left and right
- State a: `min(inf, 0, 0) + min(inf, 0, 0) + 1 = 0 + 0 + 1 = 1` (camera at node 4)
- State b: `min(inf + 0, 0 + inf, inf + inf) = min(inf, inf, inf) = inf` (can't be monitored by non-existent children)
- State c: `0 + 0 = 0` (no children to monitor)
- Node 4 returns: `[1, inf, 0]`

**Step 2: Process leaf node 3**
- Node 3 has no children, similar to node 4
- Node 3 returns: `[1, inf, 0]`

**Step 3: Process node 2**
- Left child (node 4) returns `[1, inf, 0]`, right child (null) returns `[inf, 0, 0]`
- State a: `min(1, inf, 0) + min(inf, 0, 0) + 1 = 0 + 0 + 1 = 1` (camera at node 2)
- State b: `min(1 + 0, inf + inf, 1 + inf) = min(1, inf, inf) = 1` (node 4 has camera, monitoring node 2)
- State c: `inf + 0 = inf` (node 4 must be in state b, which is impossible)
- Node 2 returns: `[1, 1, inf]`

**Step 4: Process root node 1**
- Left child (node 2) returns `[1, 1, inf]`, right child (node 3) returns `[1, inf, 0]`
- State a: `min(1, 1, inf) + min(1, inf, 0) + 1 = 1 + 0 + 1 = 2` (camera at node 1, optimal states for children)
- State b: `min(1 + inf, 1 + 1, 1 + 1) = min(inf, 2, 2) = 2` (either child has camera)
- State c: `1 + inf = inf` (impossible since node 3 can't be in state b)
- Node 1 returns: `[2, 2, inf]`

**Final Answer:**
`min(a, b) = min(2, 2) = 2`

The optimal solution uses 2 cameras. One valid placement is:
- Camera at node 2 (monitors nodes 1, 2, and 4)
- Camera at node 3 (monitors node 3)

This example shows how the algorithm considers all possibilities at each node and propagates the optimal solutions upward, ultimately finding the minimum number of cameras needed.

## Python Solution

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right

from typing import Optional
from math import inf
import sys

# A skewed tree with 1000 nodes recurses deeper than the default limit of 1000
sys.setrecursionlimit(10000)

class Solution:
    def minCameraCover(self, root: Optional[TreeNode]) -> int:
        """
        Find minimum number of cameras needed to monitor all nodes in binary tree.
        Each camera can monitor its parent, itself, and its immediate children.
        """
      
        def dfs(node: Optional[TreeNode]) -> tuple[float, int, int]:
            """
            Returns tuple of three values for each node:
            - camera_here: min cameras if we place a camera at this node
            - covered_no_camera: min cameras if this node is covered but has no camera
            - not_covered: min cameras if this node is not covered
          
            For null nodes, we return (inf, 0, 0) because:
            - inf: cannot place camera on null node
            - 0: null node is considered covered without needing cameras
            - 0: null node doesn't need coverage
            """
          
            # Base case: null node
            if node is None:
                return inf, 0, 0
          
            # Recursively solve for left and right subtrees
            left_camera, left_covered, left_not_covered = dfs(node.left)
            right_camera, right_covered, right_not_covered = dfs(node.right)
          
            # Case 1: Place camera at current node
            # Camera covers current node and both children
            # Children can be in any state since they're covered by this camera
            camera_here = min(left_camera, left_covered, left_not_covered) + \
                         min(right_camera, right_covered, right_not_covered) + 1
          
            # Case 2: Current node is covered but has no camera
            # At least one child must have a camera to cover current node
            covered_no_camera = min(
                left_camera + right_covered,  # left child has camera
                left_covered + right_camera,  # right child has camera
                left_camera + right_camera    # both children have cameras
            )
          
            # Case 3: Current node is not covered
            # Both children must be covered (but don't have cameras)
            # This forces parent to have a camera
            not_covered = left_covered + right_covered
          
            return camera_here, covered_no_camera, not_covered
      
        # Get results for root node
        camera_at_root, root_covered, _ = dfs(root)
      
        # Root must be covered, so return minimum of:
        # - placing camera at root
        # - root being covered by its children
        return min(camera_at_root, root_covered)
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm performs a depth-first search (DFS) traversal of the binary tree. Each node is visited exactly once during the traversal. At each node, the function performs constant-time operations:
- Recursive calls to left and right children
- Computing values `a`, `b`, and `c` using simple arithmetic operations and `min()` comparisons

Since we visit all `n` nodes exactly once and perform `O(1)` work at each node, the overall time complexity is `O(n)`.

**Space Complexity:** `O(n)`

The space complexity is dominated by the recursive call stack. In the worst case, when the binary tree is completely unbalanced (forming a linked list), the recursion depth can reach `n` levels, requiring `O(n)` space on the call stack. In the best case of a perfectly balanced tree, the recursion depth would be `O(log n)`, but we consider the worst-case scenario for space complexity analysis.

Additionally, the function uses only a constant amount of extra space for variables (`la`, `lb`, `lc`, `ra`, `rb`, `rc`, `a`, `b`, `c`) at each recursive level, which doesn't change the overall space complexity.

Therefore, the space complexity is `O(n)`. Because a skewed tree with 1000 nodes needs more than 1000 nested calls (including the calls on null children), the Python solution raises the recursion limit with `sys.setrecursionlimit`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Handling the Root Node's Final State**

**Pitfall:** A common mistake is returning `min(a, b, c)` or just `a` for the final answer, forgetting that the root node has no parent to monitor it.

**Why it's wrong:** State `c` represents a node that is NOT monitored. Since the root has no parent, if we choose state `c`, the root would remain unmonitored forever, violating the problem's requirement.

**Solution:** Always return `min(a, b)` for the root, excluding state `c`:
```python
camera_at_root, root_covered, root_not_covered = dfs(root)
# Correct: Root must be monitored
return min(camera_at_root, root_covered)
# Wrong: Would leave root potentially unmonitored
# return min(camera_at_root, root_covered, root_not_covered)
```

### 2. **Incorrect Base Case for Null Nodes**

**Pitfall:** Returning `[0, 0, 0]` for null nodes.

**Why it's wrong:** A first value of 0 says a free camera sits on the missing child. A leaf then reports state `b` (covered by a child's camera) at cost 0, so a single-node tree returns 0 instead of 1. Using `inf` makes a camera on a null node impossible. (A first value of 1 happens to give the same answers, because a phantom camera costing 1 is never cheaper than a real camera on the leaf, but `inf` states the meaning directly.)

**Solution:** Always return `[inf, 0, 0]` for null nodes:
```python
if node is None:
    return inf, 0, 0  # Correct
    # return 0, 0, 0  # Wrong: suggests we can place a camera on null
```

### 3. **Misunderstanding State `b` Calculation**

**Pitfall:** Calculating state `b` as simply `la + ra` or `min(la, ra)`, forgetting that at least one child MUST have a camera to monitor the parent.

**Why it's wrong:** State `b` means the node is monitored by its children. This requires at least one child to have a camera pointing upward.

**Solution:** Consider all valid combinations where at least one child has a camera:
```python
# Correct: At least one child must have a camera
covered_no_camera = min(
    left_camera + right_covered,   # left has camera
    left_covered + right_camera,   # right has camera  
    left_camera + right_camera     # both have cameras
)
# Wrong: Doesn't ensure a child has a camera
# covered_no_camera = left_covered + right_covered
```

### 4. **Forgetting to Add 1 When Placing a Camera**

**Pitfall:** When calculating state `a`, forgetting to add 1 for the camera being placed at the current node.

**Why it's wrong:** Each camera costs 1, so we must account for it in our count.

**Solution:** Always add 1 when placing a camera:
```python
# Correct
camera_here = min(la, lb, lc) + min(ra, rb, rc) + 1
# Wrong: Forgets to count the current camera
# camera_here = min(la, lb, lc) + min(ra, rb, rc)
```

### 5. **Using Global Variables Incorrectly**

**Pitfall:** Some might try to use a counter stored outside the call (a module-level variable, a mutable default argument, or an instance attribute set in `__init__`) to track cameras. If the counter is not reset at the start of each `minCameraCover` call, counts from earlier test cases carry over and the results are wrong.

**Solution:** Use the return value approach with tuples to maintain state properly:
```python
# Correct: Return states as tuple
def dfs(node):
    # ... calculations ...
    return camera_here, covered_no_camera, not_covered

# Wrong: a counter that is never reset between calls
# cameras = 0  # module-level, keeps growing across test cases
```

### 6. **Exceeding Python's Recursion Limit**

**Pitfall:** The tree can have up to 1000 nodes. When it is a single chain, `dfs` nests about 1001 calls (1000 nodes plus the null child at the bottom), which exceeds Python's default recursion limit of 1000 and raises `RecursionError`.

**Solution:** Raise the limit before running the DFS:
```python
import sys
sys.setrecursionlimit(10000)
```

