# 272. Closest Binary Search Tree Value II

> Source: https://algo.monster/liteproblems/272

## Problem Statement

You are given a binary search tree (BST), a target value, and an integer `k`. Your task is to find the `k` values in the BST that are closest to the target value.

The problem requires you to:
- Traverse the BST and examine all node values
- Calculate the distance (absolute difference) between each node value and the target
- Return exactly `k` values that have the smallest distances to the target
- The values can be returned in any order

Key points about the problem:
- The tree is a valid BST, meaning left subtree values are less than the root, and right subtree values are greater than the root
- The target can be any floating-point number (not necessarily in the tree)
- You are guaranteed that there is exactly one unique set of `k` closest values (no ties at the k-th position)

For example, if you have a BST with values `[1, 2, 3, 4, 5]`, target = `3.7`, and `k = 2`, the two closest values would be `[3, 4]` since they have the smallest absolute differences from 3.7 (distances of 0.7 and 0.3 respectively).

## Intuition

The key insight comes from understanding two important properties: the BST structure and the nature of finding "k closest" values.

In a BST, an in-order traversal gives us values in sorted ascending order. This is crucial because when values are sorted, the closest values to any target must be contiguous in this sorted sequence. Think about it - if you have sorted values `[1, 2, 3, 4, 5]` and target `3.2`, the closest values will form a continuous window around the target, not scattered randomly.

This leads us to a sliding window approach. As we traverse the tree in-order:
1. We maintain a window of size `k` containing our current best candidates
2. For each new value we encounter, we compare distances to decide if it should replace an existing value in our window

The clever part is using a deque (double-ended queue) to maintain this window. Why? Because as we traverse in ascending order:
- Early in the traversal, all values are smaller than or equal to the target
- Later in the traversal, values become larger than the target
- The leftmost value in our deque (the smallest) will always be the farthest from the target among our current candidates when we encounter larger values

This means we only need to compare the new value's distance with the leftmost value's distance. If the new value is closer, we slide the window by removing from the left and adding to the right.

An optimization emerges: once the incoming values become so large that they're farther from the target than our leftmost value, all subsequent values (being even larger in the sorted order) will also be too far. This allows us to terminate the traversal early rather than visiting the entire tree.

This approach elegantly combines the BST property with a sliding window technique to solve the problem in `O(n)` time with `O(k)` extra space for the deque.

## Solution Approach

The implementation uses DFS with in-order traversal combined with a deque to maintain the k closest values:

**Data Structures Used:**
- `deque`: A double-ended queue to maintain a sliding window of k values
- Recursive call stack for DFS traversal

**Algorithm Steps:**

1. **Initialize the deque**: Create an empty deque `q` that will store up to `k` values.

2. **Perform in-order DFS traversal**: The `dfs` function recursively traverses the tree:
   ```python
   dfs(root.left)  # Visit left subtree first
   # Process current node
   dfs(root.right) # Visit right subtree last
   ```

3. **Process each node during traversal**:
   - **If deque has less than k elements**: Simply append the current value
     ```python
     if len(q) < k:
         q.append(root.val)
     ```

   - **If deque is full (has k elements)**:
     - Compare the distance of the current value with the leftmost value in deque
     - Since we're traversing in ascending order, the leftmost value is the smallest
     - If `abs(root.val - target) >= abs(q[0] - target)`, we can stop early because:
       - The current value is farther from target than our worst candidate
       - All subsequent values (being larger) will be even farther
     - Otherwise, slide the window:
       ```python
       q.popleft()      # Remove the farthest value (leftmost)
       q.append(root.val)  # Add the closer value
       ```

4. **Early termination optimization**: When we encounter a value that's farther from the target than our worst candidate in the deque, we return immediately. This prevents unnecessary traversal of the remaining larger values in the tree.

5. **Return the result**: Convert the deque to a list and return.

**Time Complexity**: `O(n)` in worst case where we traverse all nodes, but often better due to early termination.

**Space Complexity**: `O(k)` for the deque plus `O(h)` for the recursion stack, where h is the height of the tree.

The elegance of this solution lies in using the sorted nature of in-order traversal to maintain a sliding window efficiently, requiring only one comparison per node after the deque is full.

## Example Walkthrough

Let's walk through finding the k=2 closest values to target=3.8 in this BST:

```
       4
      / \
     2   5
    / \
   1   3
```

**Initial Setup:**
- Target = 3.8
- k = 2
- deque q = []

**In-order Traversal (visits nodes in ascending order: 1→2→3→4→5):**

**Step 1: Visit node 1**
- q is empty (len < k), so append 1
- q = [1]

**Step 2: Visit node 2**
- q has 1 element (len < k), so append 2
- q = [1, 2]
- Now q is full with k=2 elements

**Step 3: Visit node 3**
- q is full, need to compare distances:
  - Distance of new value 3 from target: |3 - 3.8| = 0.8
  - Distance of leftmost value 1 from target: |1 - 3.8| = 2.8
  - Since 0.8 < 2.8, value 3 is closer than value 1
- Slide the window: remove 1 from left, add 3 to right
- q = [2, 3]

**Step 4: Visit node 4**
- q is full, compare distances:
  - Distance of new value 4 from target: |4 - 3.8| = 0.2
  - Distance of leftmost value 2 from target: |2 - 3.8| = 1.8
  - Since 0.2 < 1.8, value 4 is closer than value 2
- Slide the window: remove 2 from left, add 4 to right
- q = [3, 4]

**Step 5: Visit node 5**
- q is full, compare distances:
  - Distance of new value 5 from target: |5 - 3.8| = 1.2
  - Distance of leftmost value 3 from target: |3 - 3.8| = 0.8
  - Since 1.2 > 0.8, value 5 is NOT closer than value 3
- **Early termination triggered!** Since we're traversing in ascending order and value 5 is already farther than our worst candidate (3), all subsequent values would be even farther.
- Return immediately with q = [3, 4]

**Result:** [3, 4] are the 2 closest values to 3.8

The key insight is that the deque maintains a sliding window of the k closest values seen so far, and the sorted nature of in-order traversal allows us to stop early once values become too far from the target.

## Python Solution

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right

from collections import deque
from typing import List, Optional

class Solution:
    def closestKValues(self, root: Optional[TreeNode], target: float, k: int) -> List[int]:
        """
        Find k values in a BST that are closest to the target value.

        Uses in-order traversal to visit nodes in sorted order and maintains
        a sliding window of k closest values using a deque.

        Args:
            root: Root of the binary search tree
            target: Target value to find closest values to
            k: Number of closest values to return

        Returns:
            List of k values closest to the target
        """

        def inorder_traverse(node: Optional[TreeNode]) -> None:
            """
            Perform in-order traversal of the BST while maintaining
            a window of k closest values to the target.
            """
            if node is None:
                return

            # Traverse left subtree first (smaller values in BST)
            inorder_traverse(node.left)

            # Process current node
            if len(closest_values_queue) < k:
                # Queue not full yet, add current value
                closest_values_queue.append(node.val)
            else:
                # Queue is full, check if current value is closer than the first value in queue
                # Since we're doing in-order traversal, values are processed in ascending order
                # The first element in queue is the smallest and potentially furthest from target
                if abs(node.val - target) >= abs(closest_values_queue[0] - target):
                    # Current value is not closer than the furthest value in queue
                    # All subsequent values will be even further (due to BST property)
                    return

                # Current value is closer, remove the furthest and add current
                closest_values_queue.popleft()
                closest_values_queue.append(node.val)

            # Traverse right subtree (larger values in BST)
            inorder_traverse(node.right)

        # Initialize deque to store k closest values
        closest_values_queue = deque()

        # Start in-order traversal from root
        inorder_traverse(root)

        # Convert deque to list and return
        return list(closest_values_queue)
```

## Time and Space Complexity

**Time Complexity:** `O(n)` where `n` is the number of nodes in the binary tree.

The algorithm performs an in-order traversal of the BST using DFS. In the worst case, it visits every node in the tree exactly once. Even though there's an early termination condition (`if abs(root.val - target) >= abs(q[0] - target): return`), this optimization only helps when we've found `k` values and the remaining nodes are farther from the target. In the worst case scenario where all nodes are very close to the target or when the target is at one extreme of the tree's value range, we still need to traverse all `n` nodes.

**Space Complexity:** `O(h + k)` where `h` is the height of the tree and `k` is the number of closest values to find.

The space complexity consists of two components:
- The recursive call stack for DFS traversal uses `O(h)` space, where `h` is the height of the tree. In the worst case of a skewed tree, `h = n`, making it `O(n)`. In the best case of a balanced tree, `h = log(n)`.
- The deque `q` stores at most `k` elements at any time, contributing `O(k)` space.
- The final list conversion also takes `O(k)` space, but this doesn't add to the asymptotic complexity.

Therefore, the total space complexity is `O(h + k)`.

## Common Pitfalls

### 1. **Incorrect Early Termination Logic**

**The Pitfall:**
A common mistake is using the wrong comparison for early termination. Some might incorrectly write:
```python
if abs(node.val - target) > abs(closest_values_queue[0] - target):
    return
```
Instead of using `>=`. With `>`, a node whose distance equals that of `q[0]` replaces it and the traversal continues. The problem guarantees a unique answer, so this does not change the result here, but it does extra work.

**Why It's Wrong:**
When distances are equal, we should still terminate because:
- We already have k values with distances less than or equal to the current distance
- Since we're traversing in ascending order, all subsequent values will have distances greater than or equal to the current one
- Missing the equal case means unnecessary traversal continues

**The Fix:**
Always use `>=` in the comparison to handle the edge case where distances are equal:
```python
if abs(node.val - target) >= abs(closest_values_queue[0] - target):
    return
```

### 2. **Assuming Target is an Integer**

**The Pitfall:**
Treating the target as an integer when it's actually a float, which can lead to incorrect distance calculations:
```python
# Wrong: assuming target is int
def closestKValues(self, root: Optional[TreeNode], target: int, k: int) -> List[int]:
```

**Why It's Wrong:**
The target can be any floating-point number (e.g., 3.7, 2.5), and treating it as an integer would either cause type errors or incorrect results due to implicit conversion.

**The Fix:**
Always declare target as float and handle floating-point arithmetic properly:
```python
def closestKValues(self, root: Optional[TreeNode], target: float, k: int) -> List[int]:
```

### 3. **Not Handling the Case When k = 0**

**The Pitfall:**
The code doesn't explicitly handle when k = 0, which should return an empty list:
```python
# Current code will work but isn't explicit about this edge case
if len(closest_values_queue) < k:  # When k=0, this never executes
    closest_values_queue.append(node.val)
```

**Why It Matters:**
While the current implementation technically handles k=0 correctly (returns empty list), it's not immediately obvious and could confuse maintainers.

**The Fix:**
Add an explicit check at the beginning:
```python
def closestKValues(self, root: Optional[TreeNode], target: float, k: int) -> List[int]:
    if k == 0 or root is None:
        return []

    # Rest of the implementation...
```

### 4. **Misunderstanding the Sliding Window Direction**

**The Pitfall:**
Some might think they should remove from the right side of the deque instead of the left:
```python
# Wrong approach:
closest_values_queue.pop()  # Removes from right
closest_values_queue.append(node.val)
```

**Why It's Wrong:**
In in-order traversal, we visit nodes in ascending order. The leftmost element in the deque is the smallest value, and as we encounter larger values, we need to check if they're closer to the target than the smallest (leftmost) value. Removing from the right would break this logic.

**The Fix:**
Always remove from the left when the deque is full:
```python
closest_values_queue.popleft()  # Correct: removes the smallest/leftmost value
closest_values_queue.append(node.val)
```

