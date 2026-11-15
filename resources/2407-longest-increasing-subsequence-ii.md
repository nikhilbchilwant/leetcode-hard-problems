# 2407. Longest Increasing Subsequence II

> Source: https://algo.monster/liteproblems/2407

## Problem Statement

You are given an integer array `nums` and an integer `k`.

Your task is to find the longest subsequence from `nums` that satisfies two conditions:
1. The subsequence must be **strictly increasing** (each element is greater than the previous one)
2. The difference between any two adjacent elements in the subsequence must be **at most** `k` (if we have adjacent elements `a` and `b` where `b` comes after `a`, then `b - a ≤ k`)

Return the length of the longest subsequence that meets both requirements.

A **subsequence** is a sequence that can be derived from the original array by deleting some or no elements without changing the relative order of the remaining elements. For example, `[1, 3, 5]` is a subsequence of `[1, 2, 3, 4, 5]`.

The key challenge here is that while we want the subsequence to be increasing, we also need to ensure that consecutive elements in our chosen subsequence don't differ by more than `k`. This means if we select an element `v`, the next element we can select must be greater than `v` but not greater than `v + k`.

## Intuition

Let's think about this problem step by step. For each element in the array, we need to decide whether to include it in our subsequence or not. If we include it, we want to know: what's the longest valid subsequence that can end at this element?

This naturally leads us to [dynamic programming](/problems/dynamic_programming_intro). We can define `f[v]` as the length of the longest increasing subsequence that ends with value `v`. When we encounter a new element `v`, we need to find the best subsequence we can extend.

Since our subsequence must be strictly increasing with adjacent differences at most `k`, if we want to end our subsequence at value `v`, the previous element in the subsequence must be:
- Less than `v` (for strict increasing property)
- At least `v - k` (to satisfy the difference constraint)

So we need to look at all values in the range `[v-k, v-1]` and find which one gives us the longest subsequence to extend. The recurrence relation becomes: `f[v] = max(f[x]) + 1` where `x ∈ [v-k, v-1]`.

The challenge now is efficiency. For each element, we need to query the maximum value in a range and update a single point. If we use a simple array and iterate through the range each time, it would be too slow for large inputs.

This is where the [segment tree](/problems/segment_tree_intro) comes in. It's perfect for this scenario because:
- We need range maximum queries (find the best subsequence length in range `[v-k, v-1]`)
- We need point updates (update `f[v]` after processing element `v`)
- Both operations can be done in `O(log n)` time with a segment tree

The [segment tree](/problems/segment_tree_intro) maintains the maximum subsequence length for any range of values efficiently, allowing us to solve the problem in `O(n log m)` time where `n` is the length of the array and `m` is the maximum value in the array.

**Learn more about [Segment Tree](/problems/segment_tree_intro), [Queue](/problems/queue_intro), [Divide and Conquer](/problems/divide_and_conquer_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Monotonic Queue](/problems/mono_stack_intro) patterns.**

## Solution Approach

The solution uses a [segment tree](/problems/segment_tree_intro) to efficiently maintain and query the maximum subsequence length for value ranges. Let's walk through the implementation:

### [Segment Tree](/problems/segment_tree_intro) Structure

The [segment tree](/problems/segment_tree_intro) is built to handle values from `1` to `max(nums)`. Each node in the tree stores:
- `l`: left boundary of the interval
- `r`: right boundary of the interval  
- `v`: maximum subsequence length in this interval

The tree is initialized with `4 * n` nodes (a standard size for segment trees) where `n` is the maximum value in `nums`.

### Building the Tree

The `build` function recursively constructs the tree:
- Each node represents an interval `[l, r]`
- Leaf nodes represent single values `[x, x]`
- Internal nodes split their range at the midpoint: `mid = (l + r) >> 1`
- Left child covers `[l, mid]`, right child covers `[mid + 1, r]`

### Core Operations

**1. Query Operation (`query`)**
- Finds the maximum subsequence length in range `[l, r]`
- If current node's interval is completely within query range, return its value
- Otherwise, recursively query relevant children
- Combines results from left and right subtrees using `max`

**2. Modify Operation (`modify`)**  
- Updates the subsequence length for a specific value `x` to `v`
- Navigates to the leaf node representing value `x`
- Updates the value and propagates changes up using `pushup`

**3. Pushup Operation**
- Updates parent node's value as the maximum of its children
- Ensures the tree maintains correct maximum values after modifications

### Main Algorithm

```python
def lengthOfLIS(self, nums: List[int], k: int) -> int:
    tree = SegmentTree(max(nums))
    ans = 1
    for v in nums:
        t = tree.query(1, v - k, v - 1) + 1
        ans = max(ans, t)
        tree.modify(1, v, t)
    return ans
```

For each element `v` in `nums`:
1. **Query** the maximum subsequence length in range `[v-k, v-1]` using `tree.query(1, v - k, v - 1)`
2. **Calculate** the new subsequence length ending at `v`: `t = query_result + 1`
3. **Update** the answer with the maximum length found so far
4. **Modify** the tree to record that the longest subsequence ending at value `v` has length `t`

### Time Complexity

- Building the tree: `O(m)` where `m = max(nums)`
- Each query and modify operation: `O(log m)`
- Total: `O(m + n log m)` where `n` is the length of `nums`

The [segment tree](/problems/segment_tree_intro) efficiently handles the range maximum queries and point updates needed for the [dynamic programming](/problems/dynamic_programming_intro) solution, making it possible to solve the problem for large inputs.

## Example Walkthrough

Let's walk through a concrete example to understand how the solution works.

**Input:** `nums = [4, 2, 1, 4, 3, 5]`, `k = 2`

We'll build a segment tree for values 1 to 5 (max of nums). The tree will store `f[v]` - the length of the longest increasing subsequence ending at value `v`.

**Initial State:**
- Segment tree initialized with all values as 0
- `f[1] = 0, f[2] = 0, f[3] = 0, f[4] = 0, f[5] = 0`

**Processing nums[0] = 4:**
- Query range `[4-2, 4-1] = [2, 3]`: max(f[2], f[3]) = max(0, 0) = 0
- New length ending at 4: `t = 0 + 1 = 1`
- Update: `f[4] = 1`
- Current answer: `ans = 1`

**Processing nums[1] = 2:**
- Query range `[2-2, 2-1] = [0, 1]`: max(f[1]) = 0 (0 is out of bounds)
- New length ending at 2: `t = 0 + 1 = 1`
- Update: `f[2] = 1`
- Current answer: `ans = 1`

**Processing nums[2] = 1:**
- Query range `[1-2, 1-1] = [-1, 0]`: no valid values in range, returns 0
- New length ending at 1: `t = 0 + 1 = 1`
- Update: `f[1] = 1`
- Current answer: `ans = 1`

**Processing nums[3] = 4:**
- Query range `[4-2, 4-1] = [2, 3]`: max(f[2], f[3]) = max(1, 0) = 1
- New length ending at 4: `t = 1 + 1 = 2`
- Update: `f[4] = 2` (we found subsequence [2, 4])
- Current answer: `ans = 2`

**Processing nums[4] = 3:**
- Query range `[3-2, 3-1] = [1, 2]`: max(f[1], f[2]) = max(1, 1) = 1
- New length ending at 3: `t = 1 + 1 = 2`
- Update: `f[3] = 2` (we can have [1, 3] or [2, 3])
- Current answer: `ans = 2`

**Processing nums[5] = 5:**
- Query range `[5-2, 5-1] = [3, 4]`: max(f[3], f[4]) = max(2, 2) = 2
- New length ending at 5: `t = 2 + 1 = 3`
- Update: `f[5] = 3` (we found subsequence [2, 4, 5] or [2, 3, 5])
- Current answer: `ans = 3`

**Final Result:** 3

The longest valid subsequence is `[2, 4, 5]` or `[2, 3, 5]`, both with length 3. Notice how:
- Each subsequence is strictly increasing
- Adjacent differences are at most k=2 (4-2=2, 5-4=1 for first; 3-2=1, 5-3=2 for second)
- The segment tree efficiently tracked the best subsequence length ending at each value

## Python Solution

```python
from typing import List


class Node:
    """Represents a node in the segment tree."""
  
    def __init__(self):
        self.left_bound = 0   # Left boundary of the segment
        self.right_bound = 0  # Right boundary of the segment
        self.value = 0        # Maximum value in this segment


class SegmentTree:
    """
    Segment tree for range maximum queries and point updates.
    Used to efficiently find maximum values in ranges.
    """
  
    def __init__(self, n: int):
        """
        Initialize segment tree with size n.
      
        Args:
            n: Maximum value in the range [1, n]
        """
        # Allocate 4*n nodes for the segment tree
        self.tree = [Node() for _ in range(4 * n)]
        self.build(1, 1, n)
  
    def build(self, node_idx: int, left: int, right: int) -> None:
        """
        Build the segment tree recursively.
      
        Args:
            node_idx: Current node index in the tree array
            left: Left boundary of current segment
            right: Right boundary of current segment
        """
        self.tree[node_idx].left_bound = left
        self.tree[node_idx].right_bound = right
      
        # Base case: leaf node
        if left == right:
            return
      
        # Recursively build left and right subtrees
        mid = (left + right) >> 1  # Equivalent to // 2
        self.build(node_idx << 1, left, mid)           # Left child at index 2*node_idx
        self.build(node_idx << 1 | 1, mid + 1, right)  # Right child at index 2*node_idx + 1
  
    def modify(self, node_idx: int, position: int, value: int) -> None:
        """
        Update the value at a specific position.
      
        Args:
            node_idx: Current node index in the tree array
            position: Position to update
            value: New value to set
        """
        current_node = self.tree[node_idx]
      
        # Base case: reached the target leaf node
        if current_node.left_bound == position and current_node.right_bound == position:
            current_node.value = value
            return
      
        # Recursively update the appropriate child
        mid = (current_node.left_bound + current_node.right_bound) >> 1
        if position <= mid:
            self.modify(node_idx << 1, position, value)
        else:
            self.modify(node_idx << 1 | 1, position, value)
      
        # Update current node's value based on children
        self.pushup(node_idx)
  
    def pushup(self, node_idx: int) -> None:
        """
        Update parent node value based on its children's values.
      
        Args:
            node_idx: Current node index to update
        """
        left_child_value = self.tree[node_idx << 1].value
        right_child_value = self.tree[node_idx << 1 | 1].value
        self.tree[node_idx].value = max(left_child_value, right_child_value)
  
    def query(self, node_idx: int, query_left: int, query_right: int) -> int:
        """
        Query the maximum value in range [query_left, query_right].
      
        Args:
            node_idx: Current node index in the tree array
            query_left: Left boundary of query range
            query_right: Right boundary of query range
          
        Returns:
            Maximum value in the specified range
        """
        current_node = self.tree[node_idx]
      
        # Current segment is completely within query range
        if current_node.left_bound >= query_left and current_node.right_bound <= query_right:
            return current_node.value
      
        # Query both children if they overlap with query range
        mid = (current_node.left_bound + current_node.right_bound) >> 1
        max_value = 0
      
        # Check if left child overlaps with query range
        if query_left <= mid:
            max_value = self.query(node_idx << 1, query_left, query_right)
      
        # Check if right child overlaps with query range
        if query_right > mid:
            max_value = max(max_value, self.query(node_idx << 1 | 1, query_left, query_right))
      
        return max_value


class Solution:
    def lengthOfLIS(self, nums: List[int], k: int) -> int:
        """
        Find length of longest increasing subsequence with constraint that
        difference between consecutive elements is at most k.
      
        Args:
            nums: Input array of integers
            k: Maximum allowed difference between consecutive elements
          
        Returns:
            Length of the longest increasing subsequence with the given constraint
        """
        # Initialize segment tree with range up to maximum value in nums
        segment_tree = SegmentTree(max(nums))
        max_length = 1
      
        for current_value in nums:
            # Query maximum length ending with values in range [current_value - k, current_value - 1]
            # This ensures the difference constraint is satisfied
            prev_max_length = segment_tree.query(1, current_value - k, current_value - 1)
            current_length = prev_max_length + 1
          
            # Update the answer if we found a longer subsequence
            max_length = max(max_length, current_length)
          
            # Update the segment tree with the length of subsequence ending at current_value
            segment_tree.modify(1, current_value, current_length)
      
        return max_length
```

## Time and Space Complexity

**Time Complexity:** `O(n × log m)`, where `n` is the length of the array `nums` and `m` is the maximum value in `nums`.

- The segment tree initialization takes `O(m)` time to build a tree with `m` leaf nodes
- For each of the `n` elements in `nums`:
  - The `query` operation traverses from root to leaf, taking `O(log m)` time
  - The `modify` operation also traverses from root to leaf and calls `pushup` back up, taking `O(log m)` time
- Total: `O(m) + n × O(log m) = O(n × log m)` since typically `n × log m > m`

Note: The reference answer states `O(n × log n)`, which would be accurate if the segment tree was built on indices rather than values. In this implementation, the tree is built on the value range `[1, max(nums)]`, making the complexity dependent on `m = max(nums)` rather than `n`.

**Space Complexity:** `O(m)`, where `m` is the maximum value in `nums`.

- The segment tree uses `4 × m` nodes to store the tree structure
- Each node stores three integers (`l`, `r`, `v`)
- Total space: `O(4m) = O(m)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Handling Edge Cases with Invalid Query Ranges**

One critical pitfall in this solution is when querying for values at the boundaries. When `current_value - k` is less than 1 or when `current_value - 1` is less than `current_value - k` (which happens when `current_value = 1`), the query range becomes invalid.

**The Problem:**
```python
# When current_value = 1, this becomes:
prev_max_length = segment_tree.query(1, 1 - k, 0)  # Invalid range!
```

This creates an invalid range where the left boundary is greater than the right boundary, potentially causing incorrect results or runtime errors.

**Solution:**
```python
def lengthOfLIS(self, nums: List[int], k: int) -> int:
    segment_tree = SegmentTree(max(nums))
    max_length = 1
  
    for current_value in nums:
        # Handle edge case: ensure valid query range
        if current_value > 1:
            query_left = max(1, current_value - k)
            query_right = current_value - 1
            prev_max_length = segment_tree.query(1, query_left, query_right)
            current_length = prev_max_length + 1
        else:
            # When current_value = 1, it must be the first element of any subsequence
            current_length = 1
      
        max_length = max(max_length, current_length)
        segment_tree.modify(1, current_value, current_length)
  
    return max_length
```

### 2. **Memory Optimization for Large Value Ranges**

Another significant pitfall occurs when the maximum value in `nums` is very large (e.g., 10^9), but the array itself is small. The current implementation allocates `4 * max(nums)` nodes, which can cause memory issues.

**The Problem:**
```python
# If max(nums) = 10^9 and len(nums) = 100
# We allocate 4 * 10^9 nodes unnecessarily!
segment_tree = SegmentTree(max(nums))
```

**Solution using Coordinate Compression:**
```python
def lengthOfLIS(self, nums: List[int], k: int) -> int:
    # Coordinate compression
    unique_values = sorted(set(nums))
    value_to_index = {v: i + 1 for i, v in enumerate(unique_values)}
  
    # Build segment tree with compressed size
    segment_tree = SegmentTree(len(unique_values))
    max_length = 1
  
    for current_value in nums:
        compressed_value = value_to_index[current_value]
      
        # Find range of valid previous values
        left_value = current_value - k
        right_value = current_value - 1
      
        # Binary search for compressed indices
        import bisect
        left_idx = bisect.bisect_left(unique_values, left_value)
        right_idx = bisect.bisect_right(unique_values, right_value) - 1
      
        if left_idx <= right_idx:
            prev_max_length = segment_tree.query(1, left_idx + 1, right_idx + 1)
            current_length = prev_max_length + 1
        else:
            current_length = 1
      
        max_length = max(max_length, current_length)
        segment_tree.modify(1, compressed_value, current_length)
  
    return max_length
```

### 3. **Integer Overflow in Bit Operations**

While Python handles large integers automatically, in other languages or when porting this code, bit operations might cause issues with large indices.

**The Problem:**
```python
# For very deep trees, this might overflow in other languages
self.build(node_idx << 1, left, mid)  # 2 * node_idx might overflow
```

**Solution:**
Add bounds checking or use regular multiplication for clarity:
```python
def build(self, node_idx: int, left: int, right: int) -> None:
    # Add safety check
    if node_idx >= len(self.tree):
        raise ValueError("Tree index out of bounds")
  
    self.tree[node_idx].left_bound = left
    self.tree[node_idx].right_bound = right
  
    if left == right:
        return
  
    mid = (left + right) // 2  # Use regular division for clarity
    left_child = 2 * node_idx
    right_child = 2 * node_idx + 1
  
    # Ensure children indices are valid
    if right_child < len(self.tree):
        self.build(left_child, left, mid)
        self.build(right_child, mid + 1, right)
```

These modifications make the solution more robust and handle edge cases that could cause incorrect results or runtime errors in production environments.

