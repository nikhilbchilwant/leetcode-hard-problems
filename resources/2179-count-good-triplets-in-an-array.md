# 2179. Count Good Triplets in an Array

> Source: https://algo.monster/liteproblems/2179

## Problem Statement

You are given two arrays `nums1` and `nums2`, both of length `n`. Each array is a permutation of `[0, 1, ..., n - 1]`, meaning they contain all numbers from 0 to n-1 exactly once but in different orders.

A **good triplet** is a set of three distinct values `(x, y, z)` that satisfy both of these conditions:
1. In `nums1`, value `x` appears before value `y`, and value `y` appears before value `z`
2. In `nums2`, value `x` appears before value `y`, and value `y` appears before value `z`

In other words, if we denote:
- `pos1[v]` = index of value `v` in `nums1`
- `pos2[v]` = index of value `v` in `nums2`

Then for a good triplet `(x, y, z)`, we need:
- `pos1[x] < pos1[y] < pos1[z]` (x comes before y, y comes before z in nums1)
- `pos2[x] < pos2[y] < pos2[z]` (x comes before y, y comes before z in nums2)

The task is to count the total number of such good triplets.

For example, if `nums1 = [2, 0, 1]` and `nums2 = [0, 1, 2]`:
- The triplet `(0, 1, 2)` is NOT good because in `nums1`, the order is 2→0→1, so 2 comes before 0 and 1, which violates the required ordering
- No good triplets exist in this example

## Intuition

Instead of finding all possible triplets directly (which would be O(n³)), let's think about this problem differently. For each element, we can consider it as the **middle element** of a potential good triplet.

For a value `y` to be the middle element of a good triplet `(x, y, z)`:
- Value `x` must appear before `y` in both arrays
- Value `z` must appear after `y` in both arrays

The key insight is that as we traverse `nums1` from left to right, we're processing elements in their `nums1` order. For the current element `y`:
- Any element we've already seen comes before `y` in `nums1` (by definition of our traversal)
- Any element we haven't seen yet comes after `y` in `nums1`

Now we need to check the `nums2` constraint. Among the elements we've already seen (which are before `y` in `nums1`), how many are also before `y` in `nums2`? These can serve as `x` in our triplet. Similarly, among the elements we haven't seen yet (which are after `y` in `nums1`), how many are also after `y` in `nums2`? These can serve as `z`.

If there are `left` valid choices for `x` and `right` valid choices for `z`, then the number of good triplets with `y` as the middle element is `left × right`.

To efficiently track which positions in `nums2` have been "seen" and quickly count how many seen elements are before a given position, we use a Binary Indexed Tree (Fenwick Tree). As we process each element:
1. Find its position `p` in `nums2`
2. Count how many already-processed elements have positions less than `p` in `nums2` (these are valid `x` values)
3. Count how many unprocessed elements have positions greater than `p` in `nums2` (these are valid `z` values)
4. Multiply these counts to get triplets with current element as middle
5. Mark position `p` as processed in our Binary Indexed Tree

This reduces the complexity from O(n³) to O(n log n).

**Learn more about [Segment Tree](/problems/segment_tree_intro), [Binary Search](/problems/binary_search_intro), [Divide and Conquer](/problems/divide_and_conquer_intro) and [Merge Sort](/problems/merge_k_sorted_lists) patterns.**

## Solution Approach

The solution uses a **Binary Indexed Tree (Fenwick Tree)** to efficiently track and query the positions of processed elements in `nums2`.

### Step-by-step Implementation:

1. **Build position mapping**: First, create a dictionary `pos` that maps each value to its 1-indexed position in `nums2`:
   ```python
   pos = {v: i for i, v in enumerate(nums2, 1)}
   ```
   We use 1-indexed positions because Binary Indexed Trees typically work with 1-based indexing.

2. **Initialize the Binary Indexed Tree**: Create a tree of size `n` to track which positions in `nums2` have been processed:
   ```python
   tree = BinaryIndexedTree(n)
   ```
   Initially, all positions have value 0 (unprocessed).

3. **Process each element in nums1**: For each number in `nums1`, we calculate how many good triplets have this number as the middle element:
   ```python
   for num in nums1:
       p = pos[num]  # Get position of num in nums2
   ```

4. **Count valid left elements**: Query how many processed elements appear before position `p` in `nums2`:
   ```python
   left = tree.query(p)
   ```
   These are elements that:
   - Appear before `num` in `nums1` (already processed)
   - Appear before `num` in `nums2` (position < `p`)

5. **Count valid right elements**: Calculate how many unprocessed elements appear after position `p` in `nums2`:
   ```python
   right = n - p - (tree.query(n) - tree.query(p))
   ```
   Breaking this down:
   - `n - p`: Total elements after position `p`
   - `tree.query(n) - tree.query(p)`: Processed elements after position `p`
   - The difference gives us unprocessed elements after `p`

6. **Calculate triplets with current element as middle**:
   ```python
   ans += left * right
   ```
   Each valid left element can pair with each valid right element.

7. **Mark current position as processed**:
   ```python
   tree.update(p, 1)
   ```
   Set position `p` in the tree to 1, indicating this position is now processed.

### Binary Indexed Tree Operations:

The Binary Indexed Tree supports two key operations in O(log n) time:

- **`query(x)`**: Returns the sum of elements from position 1 to `x`. This tells us how many positions ≤ `x` have been processed.

- **`update(x, delta)`**: Adds `delta` to the value at position `x`. We use `delta = 1` to mark a position as processed.

The `lowbit(x)` function (`x & -x`) is used internally to navigate the tree structure efficiently.

### Example Walkthrough:

Consider `nums1 = [4,0,1,3,2]` and `nums2 = [4,1,0,2,3]`:

- Process `4` (position 1 in nums2): left=0, right=4, contributes 0 triplets
- Process `0` (position 3 in nums2): left=1, right=2, contributes 2 triplets
- Process `1` (position 2 in nums2): left=1, right=2, contributes 2 triplets
- And so on...

The time complexity is O(n log n) due to the Binary Indexed Tree operations, and space complexity is O(n) for the tree and position mapping.

## Example Walkthrough

Let's walk through a small example with `nums1 = [0, 3, 1, 2]` and `nums2 = [1, 0, 3, 2]`.

**Step 1: Build position mapping for nums2**
- `pos = {1: 1, 0: 2, 3: 3, 2: 4}` (1-indexed positions)

**Step 2: Initialize Binary Indexed Tree**
- Tree starts with all zeros: `[0, 0, 0, 0]`

**Step 3: Process each element in nums1**

**Processing element 0 (first in nums1):**
- Position in nums2: `p = pos[0] = 2`
- Left count: `tree.query(2) = 0` (no elements processed yet before position 2)
- Right count: `4 - 2 - (0 - 0) = 2` (elements at positions 3 and 4 are unprocessed and after position 2)
- Triplets with 0 as middle: `0 × 2 = 0`
- Update tree at position 2: Tree becomes `[0, 1, 0, 0]` (conceptually)
- Running total: 0

**Processing element 3 (second in nums1):**
- Position in nums2: `p = pos[3] = 3`
- Left count: `tree.query(3) = 1` (element 0 at position 2 is processed and before position 3)
- Right count: `4 - 3 - (1 - 1) = 1` (element at position 4 is unprocessed and after position 3)
- Triplets with 3 as middle: `1 × 1 = 1` (the triplet is (0, 3, 2))
- Update tree at position 3: Tree marks position 3 as processed
- Running total: 1

**Processing element 1 (third in nums1):**
- Position in nums2: `p = pos[1] = 1`
- Left count: `tree.query(1) = 0` (no processed elements before position 1)
- Right count: `4 - 1 - (2 - 0) = 1` (element 2 at position 4 is unprocessed and after position 1)
- Triplets with 1 as middle: `0 × 1 = 0`
- Update tree at position 1: Tree marks position 1 as processed
- Running total: 1

**Processing element 2 (fourth in nums1):**
- Position in nums2: `p = pos[2] = 4`
- Left count: `tree.query(4) = 3` (elements 1, 0, and 3 at positions 1, 2, 3 are all processed and before position 4)
- Right count: `4 - 4 - (3 - 3) = 0` (no elements after position 4)
- Triplets with 2 as middle: `3 × 0 = 0`
- Update tree at position 4: Tree marks position 4 as processed
- Running total: 1

**Final Answer: 1 good triplet**

The single good triplet is (0, 3, 2):
- In nums1: 0 appears at index 0, 3 at index 1, 2 at index 3 (0 < 1 < 3) ✓
- In nums2: 0 appears at index 1, 3 at index 2, 2 at index 3 (1 < 2 < 3) ✓

## Python Solution

```python
from typing import List


class BinaryIndexedTree:
    """
    Binary Indexed Tree (Fenwick Tree) for efficient prefix sum queries and updates.
    Supports O(log n) update and query operations.
    """

    def __init__(self, n: int) -> None:
        """
        Initialize a Binary Indexed Tree with size n.

        Args:
            n: The size of the tree (1-indexed)
        """
        self.size = n
        # Tree array with 1-based indexing (index 0 is unused)
        self.tree = [0] * (n + 1)

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

    def update(self, x: int, delta: int) -> None:
        """
        Add delta to the value at position x and update all affected nodes.

        Args:
            x: The position to update (1-indexed)
            delta: The value to add at position x
        """
        # Traverse up the tree updating all affected nodes
        while x <= self.size:
            self.tree[x] += delta
            # Move to the next node that this update affects
            x += BinaryIndexedTree.lowbit(x)

    def query(self, x: int) -> int:
        """
        Calculate the prefix sum from index 1 to x (inclusive).

        Args:
            x: The ending position for the prefix sum (1-indexed)

        Returns:
            The sum of elements from index 1 to x
        """
        prefix_sum = 0
        # Traverse down the tree accumulating partial sums
        while x > 0:
            prefix_sum += self.tree[x]
            # Move to the previous range
            x -= BinaryIndexedTree.lowbit(x)
        return prefix_sum


class Solution:
    def goodTriplets(self, nums1: List[int], nums2: List[int]) -> int:
        """
        Count the number of good triplets (i, j, k) where i < j < k and
        the relative order of nums1[i], nums1[j], nums1[k] is the same
        in both arrays.

        Args:
            nums1: First array of distinct integers
            nums2: Second array containing the same integers as nums1

        Returns:
            The count of good triplets
        """
        # Map each value to its position in nums2 (1-indexed for BIT)
        value_to_position = {value: index for index, value in enumerate(nums2, 1)}

        triplet_count = 0
        array_length = len(nums1)

        # Initialize Binary Indexed Tree to track processed elements
        fenwick_tree = BinaryIndexedTree(array_length)

        # Process each element in nums1
        for current_value in nums1:
            # Get the position of current value in nums2
            position_in_nums2 = value_to_position[current_value]

            # Count elements that appear before current position in nums2
            # and have already been processed (appear before in nums1)
            elements_before = fenwick_tree.query(position_in_nums2)

            # Count elements that appear after current position in nums2
            # and haven't been processed yet (will appear after in nums1)
            total_processed = fenwick_tree.query(array_length)
            elements_after_processed = total_processed - fenwick_tree.query(position_in_nums2)
            elements_after = array_length - position_in_nums2 - elements_after_processed

            # Each combination of (element before, current element, element after)
            # forms a valid triplet
            triplet_count += elements_before * elements_after

            # Mark current position as processed in the tree
            fenwick_tree.update(position_in_nums2, 1)

        return triplet_count
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)`

The algorithm iterates through each element in `nums1` once, performing the following operations for each element:
- Dictionary lookup for `pos[num]`: `O(1)`
- Binary Indexed Tree query operation `tree.query(p)`: `O(log n)` - traverses at most `log n` nodes by repeatedly removing the lowest bit
- Binary Indexed Tree query operations for calculating `right`: `O(log n)` for both `tree.query(n)` and `tree.query(p)`
- Binary Indexed Tree update operation `tree.update(p, 1)`: `O(log n)` - traverses at most `log n` nodes by repeatedly adding the lowest bit

Since we perform `O(log n)` operations for each of the `n` elements in the array, the total time complexity is `O(n log n)`.

**Space Complexity:** `O(n)`

The algorithm uses:
- Dictionary `pos` to store positions of elements from `nums2`: `O(n)`
- Binary Indexed Tree with array `c` of size `n + 1`: `O(n)`
- A few scalar variables (`ans`, `left`, `right`, etc.): `O(1)`

Therefore, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Errors with Indexing**
One of the most common mistakes is mixing 0-based and 1-based indexing. The Binary Indexed Tree traditionally uses 1-based indexing, while Python arrays use 0-based indexing.

**Pitfall Example:**
```python
# WRONG: Using 0-based indexing for BIT
pos = {v: i for i, v in enumerate(nums2)}  # 0-indexed positions
tree.update(pos[num], 1)  # BIT expects 1-indexed positions
```

**Solution:**
Always use 1-based indexing when working with the Binary Indexed Tree:
```python
# CORRECT: Convert to 1-based indexing
pos = {v: i for i, v in enumerate(nums2, 1)}  # Start enumeration from 1
```

### 2. **Incorrect Calculation of Elements After Current Position**
A subtle but critical error occurs when calculating how many unprocessed elements appear after the current position in `nums2`.

**Pitfall Example:**
```python
# WRONG: Forgetting to subtract already processed elements
right = n - p  # This counts ALL elements after p, including processed ones
```

**Solution:**
Account for already processed elements that appear after the current position:
```python
# CORRECT: Subtract processed elements after position p
right = n - p - (tree.query(n) - tree.query(p))
```

### 3. **Processing Elements in Wrong Order**
The algorithm depends on processing elements in the order they appear in `nums1`. Processing them in any other order will produce incorrect results.

**Pitfall Example:**
```python
# WRONG: Sorting or reordering nums1 before processing
for num in sorted(nums1):  # This breaks the algorithm logic
    # Process num...
```

**Solution:**
Always iterate through `nums1` in its original order:
```python
# CORRECT: Process in the exact order of nums1
for num in nums1:
    # Process num...
```

### 4. **Updating BIT Before Counting Triplets**
The order of operations matters. If you update the tree before counting, you'll include the current element in your own count.

**Pitfall Example:**
```python
# WRONG: Update before counting
for num in nums1:
    p = pos[num]
    tree.update(p, 1)  # Updated too early!
    left = tree.query(p)  # Now includes current element
    right = n - p - (tree.query(n) - tree.query(p))
    ans += left * right
```

**Solution:**
Always count first, then update:
```python
# CORRECT: Count first, then update
for num in nums1:
    p = pos[num]
    left = tree.query(p)  # Count elements before
    right = n - p - (tree.query(n) - tree.query(p))  # Count elements after
    ans += left * right
    tree.update(p, 1)  # Update after counting
```

### 5. **Misunderstanding the Problem Requirements**
A common conceptual error is misunderstanding what constitutes a "good triplet". The triplet consists of three VALUES (not indices) that must maintain the same relative order in both arrays.

**Pitfall Example:**
```python
# WRONG: Thinking about index triplets instead of value triplets
for i in range(n):
    for j in range(i+1, n):
        for k in range(j+1, n):
            # This is checking index relationships, not value relationships
```

**Solution:**
Focus on the values themselves and their relative positions in both arrays. The BIT solution correctly handles this by tracking which values have been processed and their positions in `nums2`.

