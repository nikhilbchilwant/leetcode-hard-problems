# 315. Count of Smaller Numbers After Self

> Source: https://algo.monster/liteproblems/315

## Problem Statement

You are given an integer array `nums`. For each element at position `i` in the array, you need to count how many elements to its right (positions `i+1` to the end) are smaller than `nums[i]`.

Return an array `counts` where `counts[i]` represents the number of elements smaller than `nums[i]` that appear after position `i`.

For example, if `nums = [5, 2, 6, 1]`:
- For `nums[0] = 5`: elements to the right are `[2, 6, 1]`, and `2` and `1` are smaller than `5`, so `counts[0] = 2`
- For `nums[1] = 2`: elements to the right are `[6, 1]`, and only `1` is smaller than `2`, so `counts[1] = 1`  
- For `nums[2] = 6`: elements to the right are `[1]`, and `1` is smaller than `6`, so `counts[2] = 1`
- For `nums[3] = 1`: no elements to the right, so `counts[3] = 0`

The result would be `counts = [2, 1, 1, 0]`.

The solution uses a Binary Indexed Tree (Fenwick Tree) data structure combined with coordinate compression. The approach works by:

1. **Coordinate Compression**: First, all unique values in `nums` are sorted and mapped to indices starting from 1. This compression reduces the range of values we need to handle.

2. **Reverse Processing**: The array is processed from right to left. For each element:
   - Query the Binary Indexed Tree to count how many elements smaller than the current element have been seen so far
   - Update the tree by adding the current element
 
3. **Binary Indexed Tree Operations**:
   - `update(x, delta)`: Adds `delta` to position `x` in the tree
   - `query(x)`: Returns the sum of all elements from position 1 to `x`
   - `lowbit(x)`: Helper function that returns the rightmost set bit, used for tree navigation

Since we process elements from right to left and build our answer array in reverse, the final step reverses the answer array to get the correct order.

## Intuition

The naive approach would be to check every pair of indices `(i, j)` where `j > i` and count how many times `nums[j] < nums[i]`. This would take `O(n²)` time, which is inefficient for large arrays.

The key insight is to process the array from right to left. When we're at position `i`, we've already seen all elements that come after it. If we can efficiently query "how many elements smaller than `nums[i]` have we seen so far?", we solve the problem.

This naturally leads us to think about data structures that support:
1. Efficient insertion of elements as we see them
2. Efficient counting of elements less than a given value

A Binary Indexed Tree (Fenwick Tree) is perfect for this because it can handle both operations in `O(log n)` time. However, there's a challenge: Binary Indexed Trees work with array indices, not arbitrary values.

This is where coordinate compression comes in. If `nums = [5, 2, 6, 1]`, we notice that the actual values don't matter - only their relative ordering does. We can map these to smaller indices: `1 → 1`, `2 → 2`, `5 → 3`, `6 → 4`. This mapping preserves the relative order while giving us a compact range `[1, 4]` to work with.

Walking through the process backwards:
- Start from the last element (nothing to its right, count = 0)
- Move to the second-last element, check how many smaller elements we've seen
- Continue this pattern, building up our "seen" elements in the Binary Indexed Tree

The Binary Indexed Tree acts like a frequency array where `tree[x]` conceptually represents how many times we've seen value `x`. When we query for values less than `x`, we sum up frequencies from `1` to `x-1`, giving us the count of smaller elements.

By processing right-to-left and using the tree to track what we've seen, we transform a quadratic problem into one that runs in `O(n log n)` time.

## Solution Approach

The implementation consists of two main components: the Binary Indexed Tree class and the main solution logic.

**Binary Indexed Tree Implementation:**

The `BinaryIndexedTree` class maintains an array `c` of size `n+1` (1-indexed for easier calculations). The core operations are:

1. **`lowbit(x)`**: Returns `x & -x`, which isolates the rightmost set bit. For example, `lowbit(6) = 2` because `6` in binary is `110`, and the rightmost set bit represents `2`.

2. **`update(x, delta)`**: Adds `delta` to position `x` and propagates the update upward in the tree structure. Starting from index `x`, it moves to the next responsible node by adding `lowbit(x)` until it exceeds `n`.

3. **`query(x)`**: Computes the prefix sum from index `1` to `x`. It accumulates values by moving downward in the tree, subtracting `lowbit(x)` at each step until reaching `0`.

**Main Solution Logic:**

1. **Coordinate Compression**: 
   ```python
   alls = sorted(set(nums))
   m = {v: i for i, v in enumerate(alls, 1)}
   ```
   - Extract unique values from `nums` and sort them
   - Create a mapping where each unique value maps to an index starting from `1`
   - This compresses the value range to `[1, len(unique_values)]`

2. **Initialize the Tree**:
   ```python
   tree = BinaryIndexedTree(len(m))
   ```
   Create a Binary Indexed Tree with size equal to the number of unique values.

3. **Process Array in Reverse**:
   ```python
   for v in nums[::-1]:
       x = m[v]
       tree.update(x, 1)
       ans.append(tree.query(x - 1))
   ```
   - Iterate through `nums` from right to left
   - For each value `v`, get its compressed index `x`
   - Update the tree at position `x` (increment frequency by `1`)
   - Query the tree for sum of positions `[1, x-1]` to count smaller elements
   - Append this count to the answer list

4. **Return Reversed Result**:
   ```python
   return ans[::-1]
   ```
   Since we processed the array backwards, reverse the answer to get the correct order.

**Example Walkthrough:**

For `nums = [5, 2, 6, 1]`:
- Compression: `{1:1, 2:2, 5:3, 6:4}`
- Process `1`: update position `1`, query `[1,0]` = `0`
- Process `6`: update position `4`, query `[1,3]` = `1` (only `1` is smaller)
- Process `2`: update position `2`, query `[1,1]` = `1` (only `1` is smaller)
- Process `5`: update position `3`, query `[1,2]` = `2` (both `1` and `2` are smaller)
- Reverse to get `[2, 1, 1, 0]`

The time complexity is `O(n log n)` for sorting and `O(n log n)` for the tree operations, giving overall `O(n log n)`. Space complexity is `O(n)` for the tree and mapping structures.

## Example Walkthrough

Let's walk through the solution with `nums = [5, 2, 6, 1]` to see how the Binary Indexed Tree approach works.

**Step 1: Coordinate Compression**
- Extract unique values: `{5, 2, 6, 1}`
- Sort them: `[1, 2, 5, 6]`
- Create mapping: `{1:1, 2:2, 5:3, 6:4}`
- This maps each value to a compressed index (1-indexed)

**Step 2: Initialize Binary Indexed Tree**
- Create a tree of size 4 (number of unique values)
- Initially, all frequencies are 0: `tree = [0, 0, 0, 0, 0]` (index 0 unused)

**Step 3: Process Array from Right to Left**

**Processing `nums[3] = 1`:**
- Compressed index: `x = 1`
- Query tree for positions [1, 0]: count = 0 (no elements smaller than 1)
- Update tree at position 1 (add 1)
- Tree state: `[0, 1, 1, 0, 1]` (after update propagation)
- Answer so far: `[0]`

**Processing `nums[2] = 6`:**
- Compressed index: `x = 4`
- Query tree for positions [1, 3]: count = 1 (element 1 is smaller)
- Update tree at position 4 (add 1)
- Tree state: `[0, 1, 1, 0, 2]` (after update propagation)
- Answer so far: `[0, 1]`

**Processing `nums[1] = 2`:**
- Compressed index: `x = 2`
- Query tree for positions [1, 1]: count = 1 (element 1 is smaller)
- Update tree at position 2 (add 1)
- Tree state: `[0, 1, 2, 0, 3]` (after update propagation)
- Answer so far: `[0, 1, 1]`

**Processing `nums[0] = 5`:**
- Compressed index: `x = 3`
- Query tree for positions [1, 2]: count = 2 (elements 1 and 2 are smaller)
- Update tree at position 3 (add 1)
- Tree state: `[0, 1, 2, 1, 4]` (after update propagation)
- Answer so far: `[0, 1, 1, 2]`

**Step 4: Reverse the Answer**
- We built the answer backwards: `[0, 1, 1, 2]`
- Reverse to get final result: `[2, 1, 1, 0]`

This matches our expected output where:
- `nums[0] = 5` has 2 smaller elements to its right (2 and 1)
- `nums[1] = 2` has 1 smaller element to its right (1)
- `nums[2] = 6` has 1 smaller element to its right (1)
- `nums[3] = 1` has 0 smaller elements to its right

The Binary Indexed Tree efficiently tracks the frequency of each compressed value we've seen, allowing us to query "how many values less than X have we encountered?" in O(log n) time.

## Python Solution

```python
from typing import List


class BinaryIndexedTree:
    """
    Binary Indexed Tree (Fenwick Tree) for efficient prefix sum queries and updates.
    Supports O(log n) updates and range sum queries.
    """
  
    def __init__(self, n: int) -> None:
        """
        Initialize a Binary Indexed Tree with size n.
      
        Args:
            n: The size of the tree (1-indexed)
        """
        self.size = n
        self.tree = [0] * (n + 1)  # 1-indexed array for BIT
  
    @staticmethod
    def lowbit(x: int) -> int:
        """
        Get the lowest set bit of x using bit manipulation.
        This determines the range of responsibility for each node.
      
        Args:
            x: The input number
          
        Returns:
            The value of the lowest set bit
        """
        return x & -x
  
    def update(self, index: int, delta: int) -> None:
        """
        Add delta to the element at position index.
        Updates all affected nodes in the tree.
      
        Args:
            index: The position to update (1-indexed)
            delta: The value to add
        """
        while index <= self.size:
            self.tree[index] += delta
            index += BinaryIndexedTree.lowbit(index)
  
    def query(self, index: int) -> int:
        """
        Calculate the prefix sum from position 1 to index.
      
        Args:
            index: The end position of the prefix sum (1-indexed)
          
        Returns:
            The sum of elements from position 1 to index
        """
        prefix_sum = 0
        while index > 0:
            prefix_sum += self.tree[index]
            index -= BinaryIndexedTree.lowbit(index)
        return prefix_sum


class Solution:
    def countSmaller(self, nums: List[int]) -> List[int]:
        """
        Count the number of smaller elements to the right of each element.
      
        Uses coordinate compression and a Binary Indexed Tree to efficiently
        count inversions while processing the array from right to left.
      
        Args:
            nums: List of integers
          
        Returns:
            List where result[i] is the count of smaller elements after nums[i]
        """
        # Step 1: Coordinate compression - map values to ranks
        # Sort unique values and create a mapping to compressed indices
        unique_sorted_values = sorted(set(nums))
        value_to_rank = {value: rank for rank, value in enumerate(unique_sorted_values, 1)}
      
        # Step 2: Initialize Binary Indexed Tree with compressed range
        fenwick_tree = BinaryIndexedTree(len(value_to_rank))
      
        # Step 3: Process array from right to left
        result = []
        for value in reversed(nums):
            # Get the compressed rank of current value
            rank = value_to_rank[value]
          
            # Update the tree to mark this value as seen
            fenwick_tree.update(rank, 1)
          
            # Query how many values smaller than current value we've seen
            # (rank - 1) gives us all ranks strictly less than current rank
            smaller_count = fenwick_tree.query(rank - 1)
            result.append(smaller_count)
      
        # Step 4: Reverse result since we processed array backwards
        return result[::-1]
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)`

The time complexity breaks down as follows:
- Creating the sorted set of unique values: `O(n log n)` for sorting
- Creating the mapping dictionary: `O(n)` 
- Processing each element in the reversed array: `O(n)` iterations
  - Each `update` operation: `O(log n)` - walks up the tree following lowbit jumps, at most `log n` levels
  - Each `query` operation: `O(log n)` - walks down the tree following lowbit jumps, at most `log n` levels
- Reversing the final answer: `O(n)`

Total: `O(n log n) + O(n) + O(n) * (O(log n) + O(log n)) + O(n) = O(n log n)`

**Space Complexity:** `O(n)`

The space complexity consists of:
- `alls` sorted array: `O(k)` where `k ≤ n` is the number of unique elements
- Mapping dictionary `m`: `O(k)` where `k ≤ n`
- Binary Indexed Tree array `c`: `O(k + 1) = O(k)` where `k ≤ n`
- Answer array `ans`: `O(n)`

Total: `O(k) + O(k) + O(k) + O(n) = O(n)` since `k ≤ n`

## Common Pitfalls

### 1. **Off-by-One Errors in Binary Indexed Tree Indexing**

**Pitfall:** One of the most common mistakes is confusion between 0-indexed and 1-indexed systems. The Binary Indexed Tree traditionally uses 1-based indexing, but Python arrays are 0-indexed. Developers often make errors when:
- Forgetting to add 1 when mapping values to BIT indices
- Using 0 as a valid index in the BIT (which breaks the `lowbit` function)
- Incorrectly querying `tree.query(x)` instead of `tree.query(x - 1)` to count strictly smaller elements

**Example of the mistake:**
```python
# Wrong: Using 0-based indexing for coordinate compression
value_to_rank = {value: rank for rank, value in enumerate(unique_sorted_values)}  # Starts from 0!

# This causes issues because:
# - BIT expects indices starting from 1
# - lowbit(0) returns 0, causing infinite loops
```

**Solution:**
```python
# Correct: Use 1-based indexing for coordinate compression
value_to_rank = {value: rank for rank, value in enumerate(unique_sorted_values, 1)}

# When querying for strictly smaller elements, use (rank - 1)
smaller_count = fenwick_tree.query(rank - 1)
```

### 2. **Handling Duplicate Values Incorrectly**

**Pitfall:** When the array contains duplicate values, the order of processing matters. If not handled carefully, duplicate values might be counted incorrectly when determining "smaller" elements.

**Example scenario:**
```python
nums = [5, 2, 2, 1]
# When processing the second 2, should the first 2 be counted as "smaller"? No!
# The problem asks for strictly smaller elements
```

**Solution:** The current implementation handles this correctly by:
- Using coordinate compression that maps all occurrences of the same value to the same rank
- Querying `rank - 1` ensures we only count strictly smaller values, not equal ones
- Processing from right to left maintains the correct positional relationship

### 3. **Sizing the Tree by the Maximum Value**

**Pitfall:** Creating a BIT with size based on the maximum value in the array rather than the number of unique values can cause memory issues or crashes.

**Example of the mistake:**
```python
# Wrong: Using max value directly
max_val = max(nums)
fenwick_tree = BinaryIndexedTree(max_val)  # Huge for large values, and negative values (allowed down to -10^4) have no valid index
```

**Solution:**
```python
# Correct: Use coordinate compression to limit tree size
unique_sorted_values = sorted(set(nums))
value_to_rank = {value: rank for rank, value in enumerate(unique_sorted_values, 1)}
fenwick_tree = BinaryIndexedTree(len(value_to_rank))  # Size is at most n
```

### 4. **Forgetting to Reverse the Result**

**Pitfall:** Since we process the array from right to left (to count elements that appear after each position), the results are collected in reverse order. Forgetting the final reversal is a common mistake.

**Example of the mistake:**
```python
def countSmaller(self, nums: List[int]) -> List[int]:
    # ... processing logic ...
    for value in reversed(nums):
        # ... BIT operations ...
        result.append(smaller_count)
  
    return result  # Wrong! This is in reverse order
```

**Solution:**
```python
def countSmaller(self, nums: List[int]) -> List[int]:
    # ... processing logic ...
    for value in reversed(nums):
        # ... BIT operations ...
        result.append(smaller_count)
  
    return result[::-1]  # Correct: Reverse to match original array order
```

### 5. **Incorrect Update Order**

**Pitfall:** Mixing the order of update and query with the wrong query bound. If the tree is updated with the current element and then queried with `rank` instead of `rank - 1`, the count includes the current element itself (and any equal values to its right).

**Example of the mistake:**
```python
# Wrong: update first, then query rank (not rank - 1)
for value in reversed(nums):
    rank = value_to_rank[value]
    fenwick_tree.update(rank, 1)
    smaller_count = fenwick_tree.query(rank)  # counts the current element too
    result.append(smaller_count)
```

For `nums = [5, 2, 6, 1]` this returns `[3, 2, 2, 1]` instead of `[2, 1, 1, 0]`. Querying `rank - 1` before the update is equally correct, because positions with rank `>= rank` are excluded either way.

**Correct approach:**
```python
# The implementation updates first and queries (rank - 1),
# which counts only strictly smaller elements
for value in reversed(nums):
    rank = value_to_rank[value]
    fenwick_tree.update(rank, 1)  # Mark current element as seen
    smaller_count = fenwick_tree.query(rank - 1)  # Count smaller elements
    result.append(smaller_count)
```

Querying `rank - 1` is what excludes the current element and any equal values; the order of update and query does not matter as long as that bound is used.

