# 2569. Handling Sum Queries After Update

> Source: https://algo.monster/liteproblems/2569

## Problem Statement

You are given two arrays `nums1` and `nums2` (both 0-indexed) and need to process a series of queries. Each query can be one of three types:

**Type 1 Query:** `queries[i] = [1, l, r]`
- Flip all values in `nums1` from index `l` to index `r` (inclusive)
- This means changing all `0`s to `1`s and all `1`s to `0`s in that range

**Type 2 Query:** `queries[i] = [2, p, 0]`
- Update every element in `nums2` using the formula: `nums2[i] = nums2[i] + nums1[i] * p`
- This adds `p` times the corresponding `nums1` value to each element of `nums2`

**Type 3 Query:** `queries[i] = [3, 0, 0]`
- Calculate and return the sum of all elements in `nums2`

Your task is to process all queries in order and return an array containing the results of all Type 3 queries.

## Intuition

Let's think about what each operation really does:

For Type 3 queries, we need to return the sum of `nums2`. Instead of recalculating this sum every time, we can maintain a running total `s` that represents `sum(nums2)`.

For Type 2 queries, notice that when we perform `nums2[i] = nums2[i] + nums1[i] * p` for all indices, we're essentially adding `p * sum(nums1)` to our running total `s`. This is because:
- `new_sum(nums2) = sum(nums2[i] + nums1[i] * p)`
- `= sum(nums2[i]) + sum(nums1[i] * p)`
- `= sum(nums2) + p * sum(nums1)`

So we don't need to update individual elements of `nums2` - we just need to know the sum of `nums1` at any given moment.

For Type 1 queries, we flip bits in a range of `nums1`. When a bit flips from `0` to `1`, it contributes `+1` to the sum, and when it flips from `1` to `0`, it contributes `-1`. For a range `[l, r]`:
- The new sum in that range = `(r - l + 1) - old_sum_in_range`
- This is because each `1` becomes `0` and each `0` becomes `1`

Since Type 1 involves range updates (flipping) and we need to efficiently query the sum of `nums1`, this naturally leads us to use a [segment tree](/problems/segment_tree_intro) with lazy propagation. The segment tree allows us to:
- Flip a range of values in `O(log n)` time
- Query the total sum in `O(log n)` time
- Use lazy propagation to defer updates until necessary, making range updates efficient

The lazy tag in each node indicates whether the range represented by that node needs to be flipped. When we propagate the lazy tag down, we flip the child nodes and toggle their lazy tags.

**Learn more about [Segment Tree](/problems/segment_tree_intro) patterns.**

## Solution Approach

The solution uses a **[Segment Tree](/problems/segment_tree_intro) with Lazy Propagation** to efficiently handle range updates and queries on `nums1`.

### Node Structure
Each node in the [segment tree](/problems/segment_tree_intro) contains:
- `l`, `r`: The left and right endpoints of the range (1-indexed)
- `s`: The sum of elements in this range
- `lazy`: A flag indicating if this range needs to be flipped

### Building the [Segment Tree](/problems/segment_tree_intro)
The `build(u, l, r)` function constructs the tree recursively:
- If `l == r`, it's a leaf node representing a single element from `nums1`
- Otherwise, split the range at midpoint `mid = (l + r) >> 1`
- Build left child at index `u << 1` for range `[l, mid]`
- Build right child at index `u << 1 | 1` for range `[mid + 1, r]`
- Update parent's sum using `pushup(u)`

### Range Modification (Flipping)
The `modify(u, l, r)` function flips values in range `[l, r]`:
- If current node's range is completely within `[l, r]`:
  - Toggle the lazy flag: `lazy ^= 1`
  - Update the sum: `s = (r - l + 1) - s` (since all 0s become 1s and vice versa)
- Otherwise, push down lazy tag and recursively modify children

### Lazy Propagation
The `pushdown(u)` function propagates lazy tags to children:
- If current node has a lazy tag:
  - Flip left child's sum: `left.s = (mid - l + 1) - left.s`
  - Toggle left child's lazy: `left.lazy ^= 1`
  - Do the same for right child
  - Clear current node's lazy tag

### Range Query
The `query(u, l, r)` function returns the sum of range `[l, r]`:
- If current node's range is completely within `[l, r]`, return its sum
- Otherwise, push down lazy tag and recursively query children
- Combine results from left and right children

### Main Algorithm
1. Initialize [segment tree](/problems/segment_tree_intro) with `nums1`
2. Calculate initial sum `s = sum(nums2)`
3. Process each query:
   - **Type 1** `[1, l, r]`: Call `modify(1, l+1, r+1)` to flip range (converting to 1-indexed)
   - **Type 2** `[2, p, 0]`: Update `s += p * query(1, 1, n)` where `query` returns current sum of `nums1`
   - **Type 3** `[3, 0, 0]`: Append current value of `s` to answer array

The time complexity is `O(q * log n)` where `q` is the number of queries and `n` is the length of `nums1`. The space complexity is `O(n)` for the [segment tree](/problems/segment_tree_intro).

## Example Walkthrough

Let's walk through a small example to illustrate how the solution works.

**Input:**
- `nums1 = [1, 0, 1]`
- `nums2 = [2, 3, 4]`
- `queries = [[1, 0, 1], [2, 2, 0], [3, 0, 0], [1, 1, 2], [3, 0, 0]]`

**Initial Setup:**
1. Build segment tree for `nums1 = [1, 0, 1]`:
   - Root node (range [1,3]): sum = 2
   - Left child (range [1,2]): sum = 1
   - Right child (range [3,3]): sum = 1
   - Leaf nodes: [1], [0], [1]

2. Calculate initial sum: `s = sum(nums2) = 2 + 3 + 4 = 9`

**Processing Queries:**

**Query 1: [1, 0, 1]** - Flip `nums1[0:1]` (indices 0 and 1)
- Convert to 1-indexed: flip range [1, 2]
- Before flip: `nums1 = [1, 0, 1]`, segment tree sum = 2
- After flip: `nums1 = [0, 1, 1]`, segment tree sum = 2
- The segment tree handles this by:
  - Node [1,2] gets lazy flag toggled
  - Its sum changes from 1 to (2-1) = 1 (but different elements are 1s now)

**Query 2: [2, 2, 0]** - Add `2 * nums1[i]` to each `nums2[i]`
- Current sum of `nums1` = query(1, 1, 3) = 2
- Update running total: `s = 9 + 2 * 2 = 13`
- This represents: `nums2 = [2+0*2, 3+1*2, 4+1*2] = [2, 5, 6]`

**Query 3: [3, 0, 0]** - Return sum of `nums2`
- Return current `s = 13`
- Add 13 to answer array: `ans = [13]`

**Query 4: [1, 1, 2]** - Flip `nums1[1:2]` (indices 1 and 2)
- Convert to 1-indexed: flip range [2, 3]
- Before flip: `nums1 = [0, 1, 1]`, segment tree sum = 2
- After flip: `nums1 = [0, 0, 0]`, segment tree sum = 0
- The segment tree handles this by:
  - Nodes [2,2] and [3,3] get updated
  - Total sum changes from 2 to 0

**Query 5: [3, 0, 0]** - Return sum of `nums2`
- Return current `s = 13` (unchanged since last Type 2 query)
- Add 13 to answer array: `ans = [13, 13]`

**Final Answer:** `[13, 13]`

**Key Insights from Example:**
1. We never actually modify the `nums2` array - we just track its sum in variable `s`
2. Type 1 queries efficiently update ranges in the segment tree without iterating through elements
3. Type 2 queries only need the total sum of `nums1`, which the segment tree provides in O(log n) time
4. The lazy propagation ensures we don't update nodes until necessary, making range flips efficient

## Python Solution

```python
from typing import List


class Node:
    """Represents a node in the segment tree."""
  
    def __init__(self):
        self.left = 0          # Left boundary of the segment
        self.right = 0         # Right boundary of the segment
        self.sum = 0           # Sum of 1s in the segment
        self.lazy = 0          # Lazy propagation flag for flip operations


class SegmentTree:
    """
    Segment tree implementation with lazy propagation for range updates.
    Supports range flip operations and range sum queries.
    """
  
    def __init__(self, nums: List[int]):
        """
        Initialize the segment tree with the given array.
      
        Args:
            nums: Initial array of 0s and 1s
        """
        self.nums = nums
        n = len(nums)
        # Create array of nodes with size 4*n for the segment tree
        self.tree = [Node() for _ in range(n << 2)]
        self.build(1, 1, n)
  
    def build(self, node_idx: int, left: int, right: int) -> None:
        """
        Build the segment tree recursively.
      
        Args:
            node_idx: Current node index in the tree
            left: Left boundary of current segment
            right: Right boundary of current segment
        """
        self.tree[node_idx].left = left
        self.tree[node_idx].right = right
      
        # Leaf node: directly assign the value
        if left == right:
            self.tree[node_idx].sum = self.nums[left - 1]  # Convert to 0-indexed
            return
      
        # Build left and right subtrees
        mid = (left + right) >> 1
        self.build(node_idx << 1, left, mid)
        self.build(node_idx << 1 | 1, mid + 1, right)
      
        # Update current node's sum based on children
        self.pushup(node_idx)
  
    def modify(self, node_idx: int, left: int, right: int) -> None:
        """
        Flip all bits in the range [left, right].
      
        Args:
            node_idx: Current node index in the tree
            left: Left boundary of the range to flip
            right: Right boundary of the range to flip
        """
        # Current segment is completely within the update range
        if self.tree[node_idx].left >= left and self.tree[node_idx].right <= right:
            # Toggle lazy flag
            self.tree[node_idx].lazy ^= 1
            # Flip all bits: new sum = total elements - old sum
            segment_size = self.tree[node_idx].right - self.tree[node_idx].left + 1
            self.tree[node_idx].sum = segment_size - self.tree[node_idx].sum
            return
      
        # Push down lazy updates before proceeding
        self.pushdown(node_idx)
      
        # Recursively update overlapping segments
        mid = (self.tree[node_idx].left + self.tree[node_idx].right) >> 1
        if left <= mid:
            self.modify(node_idx << 1, left, right)
        if right > mid:
            self.modify(node_idx << 1 | 1, left, right)
      
        # Update current node's sum after modifying children
        self.pushup(node_idx)
  
    def query(self, node_idx: int, left: int, right: int) -> int:
        """
        Query the sum of elements in range [left, right].
      
        Args:
            node_idx: Current node index in the tree
            left: Left boundary of the query range
            right: Right boundary of the query range
          
        Returns:
            Sum of 1s in the specified range
        """
        # Current segment is completely within the query range
        if self.tree[node_idx].left >= left and self.tree[node_idx].right <= right:
            return self.tree[node_idx].sum
      
        # Push down lazy updates before querying
        self.pushdown(node_idx)
      
        # Query overlapping segments
        mid = (self.tree[node_idx].left + self.tree[node_idx].right) >> 1
        result = 0
        if left <= mid:
            result += self.query(node_idx << 1, left, right)
        if right > mid:
            result += self.query(node_idx << 1 | 1, left, right)
      
        return result
  
    def pushup(self, node_idx: int) -> None:
        """
        Update parent node's sum based on its children.
      
        Args:
            node_idx: Current node index to update
        """
        left_child = node_idx << 1
        right_child = node_idx << 1 | 1
        self.tree[node_idx].sum = self.tree[left_child].sum + self.tree[right_child].sum
  
    def pushdown(self, node_idx: int) -> None:
        """
        Push lazy updates to children nodes.
      
        Args:
            node_idx: Current node index with lazy updates
        """
        if self.tree[node_idx].lazy:
            mid = (self.tree[node_idx].left + self.tree[node_idx].right) >> 1
            left_child = node_idx << 1
            right_child = node_idx << 1 | 1
          
            # Update left child
            left_size = mid - self.tree[node_idx].left + 1
            self.tree[left_child].sum = left_size - self.tree[left_child].sum
            self.tree[left_child].lazy ^= 1
          
            # Update right child
            right_size = self.tree[node_idx].right - mid
            self.tree[right_child].sum = right_size - self.tree[right_child].sum
            self.tree[right_child].lazy ^= 1
          
            # Clear parent's lazy flag
            self.tree[node_idx].lazy = 0


class Solution:
    """Solution for handling three types of queries on two arrays."""
  
    def handleQuery(
        self, nums1: List[int], nums2: List[int], queries: List[List[int]]
    ) -> List[int]:
        """
        Process queries on two arrays.
      
        Query types:
        1. Flip bits in nums1 from index a to b
        2. Add a * sum(nums1) to all elements in nums2
        3. Return sum of nums2
      
        Args:
            nums1: Binary array (0s and 1s)
            nums2: Integer array
            queries: List of [operation, a, b] queries
          
        Returns:
            List of results for type 3 queries
        """
        # Initialize segment tree for nums1
        tree = SegmentTree(nums1)
      
        # Track running sum of nums2
        current_sum = sum(nums2)
      
        # Store results for type 3 queries
        results = []
      
        for operation, param_a, param_b in queries:
            if operation == 1:
                # Flip bits in range [param_a, param_b]
                # Convert to 1-indexed for segment tree
                tree.modify(1, param_a + 1, param_b + 1)
            elif operation == 2:
                # Add param_a * sum(nums1) to all elements in nums2
                # This increases total sum by param_a * sum(nums1) * len(nums2)
                ones_count = tree.query(1, 1, len(nums1))
                current_sum += param_a * ones_count
            else:  # operation == 3
                # Return current sum of nums2
                results.append(current_sum)
      
        return results
```

## Time and Space Complexity

**Time Complexity:** `O(n + m × log n)`

The time complexity breaks down as follows:
- Building the segment tree in the constructor takes `O(n)` time, where `n` is the length of `nums1`. The `build` method recursively constructs the tree by visiting each node once, and there are `O(n)` nodes in total.
- Processing `m` queries takes `O(m × log n)` time, where `m` is the number of queries. For each query:
  - Type 1 queries (`modify`): Takes `O(log n)` time as it traverses from root to leaves, visiting at most `O(log n)` nodes due to the tree height being `O(log n)`.
  - Type 2 queries (`query`): Takes `O(log n)` time for the same reason - traversing at most `O(log n)` nodes to compute the sum.
  - Type 3 queries: Takes `O(1)` time as it just appends the current sum to the result.
- The initial sum calculation `sum(nums2)` takes `O(n)` time.

Therefore, the total time complexity is `O(n + m × log n)`.

**Space Complexity:** `O(n)`

The space complexity consists of:
- The segment tree array `self.tr` contains `n << 2` (which is `4n`) nodes, giving `O(n)` space.
- Each node stores a constant amount of data (four integers: `l`, `r`, `s`, `lazy`).
- The result array `ans` stores at most `m` values, but since we're typically concerned with the space used by the data structure itself rather than the output, the dominant factor is the segment tree.
- Other variables use `O(1)` space.

Therefore, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Index Conversion Error (0-indexed vs 1-indexed)
One of the most common pitfalls in this solution is confusion between 0-indexed arrays and 1-indexed segment tree operations. The segment tree internally uses 1-indexed positions while the input arrays `nums1` and `nums2` are 0-indexed.

**Problem Example:**
```python
# Incorrect: Forgetting to convert indices
tree.modify(1, l, r)  # Wrong! l and r are 0-indexed from query

# Correct: Converting to 1-indexed
tree.modify(1, l + 1, r + 1)  # Convert 0-indexed to 1-indexed
```

### 2. Incorrect Sum Update in Type 2 Query
A critical mistake is misunderstanding how Type 2 queries affect the total sum. The operation adds `p * nums1[i]` to each `nums2[i]`, which means the total sum increases by `p * sum(nums1)`, NOT `p * sum(nums1) * len(nums2)`.

**Problem Example:**
```python
# Incorrect: Multiplying by array length
current_sum += param_a * ones_count * len(nums2)  # Wrong!

# Correct: Just multiply by the count of ones
current_sum += param_a * ones_count  # Each 1 in nums1 adds param_a to total
```

### 3. Lazy Propagation Flag Management
Failing to properly toggle the lazy flag using XOR can lead to incorrect flip operations. Using assignment instead of XOR will break multiple consecutive flips.

**Problem Example:**
```python
# Incorrect: Using assignment
self.tree[node_idx].lazy = 1  # Wrong! Doesn't handle multiple flips

# Correct: Using XOR to toggle
self.tree[node_idx].lazy ^= 1  # Correctly handles multiple flips
```

### 4. Segment Tree Size Allocation
Allocating insufficient space for the segment tree array is a common mistake. The tree needs up to 4n nodes to handle all possible segments.

**Problem Example:**
```python
# Incorrect: Insufficient space
self.tree = [Node() for _ in range(n * 2)]  # May cause index out of bounds

# Correct: Allocate 4n space
self.tree = [Node() for _ in range(n << 2)]  # Safe allocation
```

### 5. Pushdown Before Query/Modify
Forgetting to push down lazy updates before querying or recursively modifying children will return stale data or apply updates incorrectly.

**Problem Example:**
```python
def query(self, node_idx: int, left: int, right: int) -> int:
    if self.tree[node_idx].left >= left and self.tree[node_idx].right <= right:
        return self.tree[node_idx].sum
  
    # Incorrect: Missing pushdown before recursive calls
    mid = (self.tree[node_idx].left + self.tree[node_idx].right) >> 1
    result = 0
    # ... recursive calls without pushdown
  
    # Correct: Always pushdown before accessing children
    self.pushdown(node_idx)  # Must come before recursive calls
    mid = (self.tree[node_idx].left + self.tree[node_idx].right) >> 1
    # ... then recursive calls
```

### Solution Best Practices:
1. **Always document index conventions** clearly in comments
2. **Test edge cases** with single-element arrays and full-range operations
3. **Validate lazy propagation** with consecutive flip operations on the same range
4. **Use consistent naming** for 0-indexed vs 1-indexed variables
5. **Add assertions** during development to catch index errors early

