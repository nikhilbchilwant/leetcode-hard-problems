# 3165. Maximum Sum of Subsequence With Non-adjacent Elements

> Source: https://algo.monster/liteproblems/3165

## Problem Statement

You have an array `nums` of integers and a list of queries. Each query `queries[i] = [pos_i, x_i]` asks you to:
1. Update the array by setting `nums[pos_i] = x_i]`
2. Find the maximum sum of a subsequence from the updated array where **no two adjacent elements are selected**

A subsequence maintains the relative order of elements from the original array but can skip elements. The constraint is that if you pick an element at index `i`, you cannot pick the element at index `i+1` or `i-1`.

For example, if `nums = [1, 2, 3, 4]`, valid subsequences include:
- `[1, 3]` (indices 0 and 2)
- `[2, 4]` (indices 1 and 3)
- `[1, 4]` (indices 0 and 3)

But `[1, 2]` would be invalid since elements at indices 0 and 1 are adjacent.

After processing each query and calculating its maximum sum, you need to return the **sum of all these maximum values** modulo `10^9 + 7`.

The key challenge is efficiently handling multiple updates and recalculating the maximum non-adjacent sum after each update. The solution uses a [segment tree](/problems/segment_tree_intro) with special state tracking to maintain four different scenarios for each segment:
- `s00`: Maximum sum excluding both endpoints
- `s01`: Maximum sum excluding left endpoint, possibly including right
- `s10`: Maximum sum possibly including left, excluding right endpoint  
- `s11`: Maximum sum possibly including both endpoints

This allows efficient combination of segments while respecting the non-adjacent constraint.

## Intuition

The problem asks us to handle multiple updates and queries for finding the maximum sum with no adjacent elements. A naive approach would recalculate the entire maximum sum after each update using [dynamic programming](/problems/dynamic_programming_intro), which takes `O(n)` time per query - too slow for large inputs.

The key insight is that when we update a single element, most of the array remains unchanged. We need a data structure that can efficiently update a single position and quickly compute the answer for the entire array. This naturally leads us to think about segment trees, which excel at point updates and range queries.

However, the challenge is: how do we merge information from two segments while respecting the "no adjacent elements" constraint? 

Consider splitting the array into two halves. When combining the maximum sums from the left and right halves, we need to know:
- Can we use the rightmost element of the left half?
- Can we use the leftmost element of the right half?

If the rightmost element of the left half is selected, we cannot select the leftmost element of the right half (they're adjacent). This means we need to track multiple states for each segment.

For any segment `[l, r]`, we maintain four values:
- `s11`: Maximum sum where we're allowed to pick both endpoints (elements at positions `l` and `r`)
- `s10`: Maximum sum where we can pick left endpoint but not right
- `s01`: Maximum sum where we can pick right endpoint but not left  
- `s00`: Maximum sum where we cannot pick either endpoint

When merging two adjacent segments, we combine their states carefully. For example, to compute `s11` for the parent (can pick both endpoints):
- We could take `s10` from left (includes left endpoint) + `s11` from right (includes right endpoint)
- Or take `s11` from left (includes left endpoint) + `s01` from right (excludes its left endpoint, which is adjacent to left segment's right endpoint)

This state-based approach allows us to efficiently merge segments in `O(1)` time while maintaining all necessary information about which endpoints can be selected, giving us `O(log n)` per update and query.

**Learn more about [Segment Tree](/problems/segment_tree_intro), [Divide and Conquer](/problems/divide_and_conquer_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a [segment tree](/problems/segment_tree_intro) with special state tracking to handle the "no adjacent elements" constraint efficiently.

### Node Structure
Each node in the [segment tree](/problems/segment_tree_intro) stores:
- `l, r`: Left and right endpoints of the segment
- `s00, s01, s10, s11`: Four state values representing different scenarios of endpoint selection

### Building the [Segment Tree](/problems/segment_tree_intro)
The `build` method recursively constructs the tree:
```python
def build(self, u: int, l: int, r: int):
    self.tr[u] = Node(l, r)
    if l == r:  # Leaf node
        return
    mid = (l + r) >> 1
    self.build(u << 1, l, mid)        # Build left child
    self.build(u << 1 | 1, mid + 1, r) # Build right child
```

### Point Update (Modify)
When updating a single position with value `v`:
```python
def modify(self, u: int, x: int, v: int):
    if self.tr[u].l == self.tr[u].r:  # Leaf node
        self.tr[u].s11 = max(0, v)    # Can only take this element or nothing
        return
    # Recursively update the appropriate child
    mid = (self.tr[u].l + self.tr[u].r) >> 1
    if x <= mid:
        self.modify(u << 1, x, v)
    else:
        self.modify(u << 1 | 1, x, v)
    self.pushup(u)  # Update current node based on children
```

For leaf nodes, `s11 = max(0, v)` because we either take the element (if positive) or skip it.

### Pushup Operation
The critical part is merging information from child nodes:
```python
def pushup(self, u: int):
    left, right = self.tr[u << 1], self.tr[u << 1 | 1]
    # s00: Cannot pick either endpoint of parent segment
    self.tr[u].s00 = max(left.s00 + right.s10,  # Left skips right endpoint, right skips left
                          left.s01 + right.s00)   # Left picks right endpoint, right must skip left
  
    # s01: Can pick right endpoint, not left
    self.tr[u].s01 = max(left.s00 + right.s11,  # Left skips right, right can pick both
                          left.s01 + right.s01)   # Left picks right, right skips left
  
    # s10: Can pick left endpoint, not right
    self.tr[u].s10 = max(left.s10 + right.s10,  # Left picks left, right skips left
                          left.s11 + right.s00)   # Left picks both, right must skip left
  
    # s11: Can pick both endpoints
    self.tr[u].s11 = max(left.s10 + right.s11,  # Left picks left only, right picks both
                          left.s11 + right.s01)   # Left picks both, right skips left
```

The merging logic ensures that if we select the right endpoint of the left segment, we cannot select the left endpoint of the right segment (they're adjacent).

### Query Operation
To get the maximum sum for the entire range:
```python
def query(self, u: int, l: int, r: int) -> int:
    if self.tr[u].l >= l and self.tr[u].r <= r:
        return self.tr[u].s11  # Return s11 for complete coverage
```

### Main Solution
```python
def maximumSumSubsequence(self, nums: List[int], queries: List[List[int]]) -> int:
    n = len(nums)
    tree = SegmentTree(n)
  
    # Initialize tree with original array
    for i, x in enumerate(nums, 1):
        tree.modify(1, i, x)
  
    ans = 0
    mod = 10**9 + 7
  
    # Process each query
    for i, x in queries:
        tree.modify(1, i + 1, x)  # Update position (1-indexed)
        ans = (ans + tree.query(1, 1, n)) % mod  # Add maximum sum
  
    return ans
```

The solution achieves `O(log n)` time per update and query, making it efficient for handling multiple queries on large arrays.

## Example Walkthrough

Let's walk through a small example to illustrate how the segment tree solution works.

**Initial Setup:**
- `nums = [3, 5, 9]`
- `queries = [[0, -2], [2, 4]]`

**Step 1: Build the segment tree**

We create a segment tree where each node tracks four states (s00, s01, s10, s11) for its range.

```
Initial array: [3, 5, 9] (1-indexed internally: positions 1, 2, 3)

Tree structure after initialization with values:
         Node[1,3]
        /         \
    Node[1,2]   Node[3,3]
    /       \
Node[1,1] Node[2,2]

Leaf nodes (base cases):
- Node[1,1]: nums[0]=3, so s11=3 (take it), s00=s01=s10=0
- Node[2,2]: nums[1]=5, so s11=5 (take it), s00=s01=s10=0  
- Node[3,3]: nums[2]=9, so s11=9 (take it), s00=s01=s10=0
```

**Step 2: Pushup to build internal nodes**

For Node[1,2] (combining positions 1 and 2):
- Left child: Node[1,1] with s11=3
- Right child: Node[2,2] with s11=5

Using the merge formulas:
- s00 = max(0+0, 0+0) = 0 (can't pick either endpoint)
- s01 = max(0+5, 0+0) = 5 (can pick right endpoint only)
- s10 = max(0+0, 3+0) = 3 (can pick left endpoint only)
- s11 = max(0+5, 3+0) = 5 (if we pick both endpoints, we'd violate adjacency, so we pick only position 2)

For root Node[1,3] (combining Node[1,2] and Node[3,3]):
- Left child: Node[1,2] with s00=0, s01=5, s10=3, s11=5
- Right child: Node[3,3] with s11=9

Merging:
- s00 = max(0+0, 5+0) = 5
- s01 = max(0+9, 5+0) = 9
- s10 = max(3+0, 5+0) = 5
- s11 = max(3+9, 5+0) = 12

The maximum sum is s11 = 12 (selecting positions 1 and 3: values 3 and 9).

**Step 3: Process Query 1 - [0, -2]**

Update position 0 (internally position 1) to -2:

Leaf Node[1,1] becomes: s11 = max(0, -2) = 0 (don't take negative value)

After pushup:
- Node[1,2]: s11 = max(0+5, 0+0) = 5 (now we only take position 2)
- Node[1,3]: s11 = max(0+9, 5+0) = 9 (take only position 3, or position 2 alone)

Maximum sum after query 1: **9**

**Step 4: Process Query 2 - [2, 4]**

Update position 2 (internally position 3) to 4:

Leaf Node[3,3] becomes: s11 = 4

After pushup:
- Node[1,3]: 
  - s11 = max(s10[left] + s11[right], s11[left] + s01[right])
  - s11 = max(0 + 4, 5 + 0) = 5

Maximum sum after query 2: **5** (taking only position 2 with value 5)

**Final Answer:**
Sum of all maximum values = 12 (initial) + 9 (after query 1) + 5 (after query 2) = 26
Return: 26 % (10^9 + 7) = 26

This example demonstrates how the segment tree efficiently updates single positions and recalculates the maximum non-adjacent sum by maintaining four states per node and carefully merging them to respect the adjacency constraint.

## Python Solution

```python
from typing import List, Optional


class Node:
    """Segment tree node storing range and four DP states."""
    __slots__ = ("left", "right", "s00", "s01", "s10", "s11")
  
    def __init__(self, left: int, right: int):
        self.left = left
        self.right = right
        # Four states for dynamic programming:
        # s00: neither left nor right boundary element included
        # s01: only right boundary element included
        # s10: only left boundary element included  
        # s11: both boundary elements included
        self.s00 = self.s01 = self.s10 = self.s11 = 0


class SegmentTree:
    """Segment tree for maintaining maximum sum subsequence with constraints."""
    __slots__ = ("tree",)
  
    def __init__(self, n: int):
        # Allocate 4n nodes for the segment tree
        self.tree: List[Optional[Node]] = [None] * (n << 2)
        self.build(1, 1, n)
  
    def build(self, node_idx: int, left: int, right: int) -> None:
        """Build the segment tree recursively."""
        self.tree[node_idx] = Node(left, right)
      
        # Base case: leaf node
        if left == right:
            return
      
        # Recursive case: build left and right subtrees
        mid = (left + right) >> 1
        self.build(node_idx << 1, left, mid)
        self.build(node_idx << 1 | 1, mid + 1, right)
  
    def query(self, node_idx: int, query_left: int, query_right: int) -> int:
        """Query the maximum sum in range [query_left, query_right]."""
        current_node = self.tree[node_idx]
      
        # If current range is completely within query range
        if current_node.left >= query_left and current_node.right <= query_right:
            return current_node.s11
      
        mid = (current_node.left + current_node.right) >> 1
        result = 0
      
        # Query left child if needed
        if query_right <= mid:
            result = self.query(node_idx << 1, query_left, query_right)
      
        # Query right child if needed
        if query_left > mid:
            result = max(result, self.query(node_idx << 1 | 1, query_left, query_right))
      
        return result
  
    def pushup(self, node_idx: int) -> None:
        """Update parent node based on children's states."""
        left_child = self.tree[node_idx << 1]
        right_child = self.tree[node_idx << 1 | 1]
        current_node = self.tree[node_idx]
      
        # Combine states from left and right children
        # s00: neither boundary included
        current_node.s00 = max(left_child.s00 + right_child.s10, 
                               left_child.s01 + right_child.s00)
      
        # s01: only right boundary included
        current_node.s01 = max(left_child.s00 + right_child.s11, 
                               left_child.s01 + right_child.s01)
      
        # s10: only left boundary included
        current_node.s10 = max(left_child.s10 + right_child.s10, 
                               left_child.s11 + right_child.s00)
      
        # s11: both boundaries included
        current_node.s11 = max(left_child.s10 + right_child.s11, 
                               left_child.s11 + right_child.s01)
  
    def modify(self, node_idx: int, position: int, value: int) -> None:
        """Modify the value at position to the given value."""
        current_node = self.tree[node_idx]
      
        # Base case: leaf node
        if current_node.left == current_node.right:
            # For leaf node, s11 represents the maximum of 0 and the value
            current_node.s11 = max(0, value)
            return
      
        # Recursive case: modify appropriate child
        mid = (current_node.left + current_node.right) >> 1
        if position <= mid:
            self.modify(node_idx << 1, position, value)
        else:
            self.modify(node_idx << 1 | 1, position, value)
      
        # Update current node after modification
        self.pushup(node_idx)


class Solution:
    def maximumSumSubsequence(self, nums: List[int], queries: List[List[int]]) -> int:
        """
        Process queries to modify array and return sum of maximum subsequence sums.
      
        Args:
            nums: Initial array of integers
            queries: List of [index, value] pairs for modifications
      
        Returns:
            Sum of maximum subsequence sums after each query, modulo 10^9 + 7
        """
        n = len(nums)
        segment_tree = SegmentTree(n)
      
        # Initialize segment tree with original values
        for i, value in enumerate(nums, 1):
            segment_tree.modify(1, i, value)
      
        total_sum = 0
        MOD = 10**9 + 7
      
        # Process each query
        for index, new_value in queries:
            # Update value at index (convert to 1-indexed)
            segment_tree.modify(1, index + 1, new_value)
            # Add maximum sum to result
            total_sum = (total_sum + segment_tree.query(1, 1, n)) % MOD
      
        return total_sum
```

## Time and Space Complexity

**Time Complexity:** `O((n + q) × log n)`

The time complexity breaks down as follows:
- **Building the segment tree:** The `build` method is called once during initialization and visits each node in the tree exactly once. Since there are `O(n)` nodes in the segment tree, building takes `O(n)` time.
- **Initial modifications:** The code performs `n` initial `modify` operations to populate the tree with values from `nums`. Each `modify` operation traverses from root to leaf, which is `O(log n)` depth, and calls `pushup` on the way back up, also `O(log n)`. So initial setup is `O(n × log n)`.
- **Processing queries:** For each of the `q` queries, the code performs:
  - One `modify` operation: `O(log n)`
  - One `query` operation: `O(log n)`
  - Total per query: `O(log n)`
  - Total for all queries: `O(q × log n)`

Overall time complexity: `O(n + n × log n + q × log n) = O((n + q) × log n)`

**Space Complexity:** `O(n)`

The space complexity is determined by:
- **Segment tree storage:** The tree is initialized with `n << 2` (which is `4n`) nodes to ensure sufficient space for a complete binary tree structure. Each node stores a constant amount of data (two integers and four values for dynamic programming states).
- **Auxiliary space:** The recursive calls in `build`, `modify`, and `query` use `O(log n)` stack space due to the tree height.

Since `O(4n) + O(log n) = O(n)`, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect State Initialization for Leaf Nodes**

**Pitfall:** A common mistake is incorrectly initializing the four states (s00, s01, s10, s11) for leaf nodes. Developers often set all states to the node value or forget to handle negative values properly.

**Incorrect Implementation:**
```python
def modify(self, node_idx: int, position: int, value: int) -> None:
    current_node = self.tree[node_idx]
  
    if current_node.left == current_node.right:
        # WRONG: Setting all states to value
        current_node.s00 = value
        current_node.s01 = value
        current_node.s10 = value
        current_node.s11 = value
        return
```

**Why it's wrong:** For a leaf node representing a single element:
- s00 means we don't take the element (should be 0)
- s01 means we take the element as right boundary only (impossible for single element)
- s10 means we take the element as left boundary only (impossible for single element)
- s11 means we take the element (should be max(0, value) to handle negatives)

**Correct Implementation:**
```python
def modify(self, node_idx: int, position: int, value: int) -> None:
    current_node = self.tree[node_idx]
  
    if current_node.left == current_node.right:
        # Correct: Only s11 is meaningful for a leaf
        current_node.s00 = 0  # Don't take the element
        current_node.s01 = 0  # Can't have only right boundary
        current_node.s10 = 0  # Can't have only left boundary
        current_node.s11 = max(0, value)  # Either take it (if positive) or skip
        return
```

### 2. **Incorrect Pushup Logic for Combining States**

**Pitfall:** Mixing up the state combinations when merging child nodes, especially forgetting the adjacency constraint between the right endpoint of the left child and the left endpoint of the right child.

**Incorrect Implementation:**
```python
def pushup(self, node_idx: int) -> None:
    left_child = self.tree[node_idx << 1]
    right_child = self.tree[node_idx << 1 | 1]
    current_node = self.tree[node_idx]
  
    # WRONG: Allowing adjacent elements to be selected
    current_node.s11 = left_child.s11 + right_child.s11  # This would select adjacent elements!
```

**Why it's wrong:** If we take s11 from the left child (which includes its right boundary) and s11 from the right child (which includes its left boundary), we're selecting two adjacent elements in the original array.

**Correct Implementation:**
```python
def pushup(self, node_idx: int) -> None:
    left_child = self.tree[node_idx << 1]
    right_child = self.tree[node_idx << 1 | 1]
    current_node = self.tree[node_idx]
  
    # Correct: Ensure no adjacent elements are selected
    current_node.s11 = max(
        left_child.s10 + right_child.s11,  # Left excludes its right, right includes both
        left_child.s11 + right_child.s01   # Left includes both, right excludes its left
    )
```

### 3. **Off-by-One Errors with Indexing**

**Pitfall:** Confusion between 0-indexed input array and 1-indexed segment tree implementation.

**Incorrect Implementation:**
```python
def maximumSumSubsequence(self, nums: List[int], queries: List[List[int]]) -> int:
    n = len(nums)
    segment_tree = SegmentTree(n)
  
    # WRONG: Using 0-indexed directly
    for i, value in enumerate(nums):
        segment_tree.modify(1, i, value)  # Should be i+1
  
    for index, new_value in queries:
        segment_tree.modify(1, index, new_value)  # Should be index+1
```

**Correct Implementation:**
```python
def maximumSumSubsequence(self, nums: List[int], queries: List[List[int]]) -> int:
    n = len(nums)
    segment_tree = SegmentTree(n)
  
    # Correct: Convert to 1-indexed
    for i, value in enumerate(nums, 1):  # Start from 1
        segment_tree.modify(1, i, value)
  
    for index, new_value in queries:
        segment_tree.modify(1, index + 1, new_value)  # Add 1 to convert
```

### 4. **Not Handling Negative Values Properly**

**Pitfall:** Forgetting that we can choose to skip negative elements entirely, leading to suboptimal solutions.

**Solution:** Always use `max(0, value)` for leaf nodes and ensure the state transitions allow for skipping elements entirely. The four-state design naturally handles this when implemented correctly.

