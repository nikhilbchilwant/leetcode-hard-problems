# 3187. Peaks in Array

> Source: https://algo.monster/liteproblems/3187

## Problem Statement

You are given an integer array `nums` and need to identify **peaks** in it. A peak is an element that is strictly greater than both its previous and next elements.

You need to process a list of `queries`, where each query can be one of two types:

1. **Type 1 Query** `[1, l, r]`: Count how many peaks exist in the subarray `nums[l..r]` (inclusive).

2. **Type 2 Query** `[2, index, val]`: Update the element at position `index` to value `val`, i.e., set `nums[index] = val`.

**Important constraints:**
- The first element (`nums[0]`) and last element (`nums[n-1]`) of the array can never be considered peaks
- Similarly, within any subarray query, the first and last elements of that subarray cannot be peaks

Your task is to return an array containing the results of all Type 1 queries in the order they appear.

**Example of a peak:**
If `nums = [1, 3, 2, 5, 4]`, then:
- `nums[1] = 3` is a peak because `1 < 3 > 2`
- `nums[3] = 5` is a peak because `2 < 5 > 4`

## Intuition

The key insight is to transform the peak detection problem into a binary problem. Instead of storing the actual array values, we can think of each position as either being a peak (1) or not a peak (0). This transforms our problem into:
- **Type 1 queries** become: "Count the number of 1s in a given range"
- **Type 2 queries** become: "Update a value and recalculate which positions are 1s or 0s"

For range sum queries with updates, a Binary Indexed Tree (Fenwick Tree) is a natural choice as it provides `O(log n)` time complexity for both operations.

The clever observation is that when we update a single element at position `idx`, it doesn't affect the entire array's peak status. A value change at position `idx` can only affect the peak status of:
1. Position `idx - 1` (because its right neighbor changed)
2. Position `idx` itself (because its value changed)
3. Position `idx + 1` (because its left neighbor changed)

This locality of impact makes the update operation efficient. Instead of rechecking the entire array, we only need to:
1. Remove the contribution of these three positions (set them to 0 in our BIT)
2. Update the actual value in the array
3. Recheck these three positions to see if they're peaks and add them back if needed

For range queries `[l, r]`, since the first and last elements of a subarray cannot be peaks, we actually query the range `[l+1, r-1]`. The Binary Indexed Tree efficiently gives us the sum (count of peaks) in this adjusted range.

This approach elegantly combines the binary representation of peaks with the efficiency of Binary Indexed Trees to handle both query types in `O(log n)` time per operation.

**Learn more about [Segment Tree](/problems/segment_tree_intro) patterns.**

## Solution Approach

The solution uses a **Binary Indexed Tree (Fenwick Tree)** to efficiently track and query peak elements in the array.

### Binary Indexed Tree Implementation

The `BinaryIndexedTree` class maintains an array `c` of size `n+1` where:
- `update(x, delta)`: Adds `delta` to position `x` in `O(log n)` time
- `query(x)`: Returns the prefix sum from position 1 to `x` in `O(log n)` time

### Main Solution Logic

1. **Initialization**: 
   - Create a BIT of size `n-1` (since first and last elements can't be peaks)
   - Iterate through positions `[1, n-2]` and mark each peak position with value 1 in the BIT

2. **Helper Function `update(i, val)`**:
   - Checks if position `i` is valid (not first or last element)
   - If `nums[i-1] < nums[i] > nums[i+1]`, updates the BIT at position `i` with value `val`

3. **Processing Queries**:

   **Type 1 Query `[1, l, r]`** - Count peaks in subarray:
   - Adjust range to `[l+1, r-1]` since endpoints can't be peaks
   - If `l+1 > r-1`, return 0 (no valid range)
   - Otherwise, compute `tree.query(r-1) - tree.query(l)` to get the count of 1s in the range

   **Type 2 Query `[2, idx, val]`** - Update element:
   - **Remove affected peaks**: For positions `[idx-1, idx, idx+1]`, call `update(i, -1)` to remove their current peak status from the BIT
   - **Update the array**: Set `nums[idx] = val`
   - **Recalculate peaks**: For the same three positions, call `update(i, 1)` to add back any positions that are now peaks

### Time Complexity
- Initialization: `O(n log n)` to build the BIT
- Type 1 Query: `O(log n)` for range sum query
- Type 2 Query: `O(log n)` for updating at most 3 positions

### Space Complexity
- `O(n)` for the Binary Indexed Tree

The elegance of this solution lies in treating peaks as binary values (1 for peak, 0 for non-peak) and using the BIT's efficient range sum capabilities to count peaks, while using the local nature of updates to maintain the structure efficiently.

## Example Walkthrough

Let's walk through a concrete example to illustrate how the solution works.

**Initial Setup:**
- `nums = [1, 3, 2, 5, 4]`
- `queries = [[1, 0, 4], [2, 3, 6], [1, 0, 4]]`

**Step 1: Initialize the Binary Indexed Tree**

First, identify which positions are peaks:
- Position 0: Cannot be a peak (first element)
- Position 1: Is `1 < 3 > 2`? Yes! → Peak
- Position 2: Is `3 < 2 > 5`? No
- Position 3: Is `2 < 5 > 4`? Yes! → Peak  
- Position 4: Cannot be a peak (last element)

Create BIT with size 4 (n-1) and mark peaks:
- BIT position 1 (array index 1): Set to 1
- BIT position 2 (array index 2): Set to 0
- BIT position 3 (array index 3): Set to 1

BIT state: `[0, 1, 0, 1]` (representing peaks at positions 1 and 3)

**Step 2: Process Query [1, 0, 4] - Count peaks in nums[0..4]**

Adjust range for peak counting: `[0+1, 4-1]` = `[1, 3]`
- Calculate: `tree.query(3) - tree.query(0)`
- `tree.query(3)` = sum of positions 1 to 3 = 1 + 0 + 1 = 2
- `tree.query(0)` = 0
- Result: 2 peaks

**Step 3: Process Query [2, 3, 6] - Update nums[3] to 6**

Before update: `nums = [1, 3, 2, 5, 4]`

Remove affected peaks (positions 2, 3, 4):
- Position 2: Not currently a peak, no change
- Position 3: Currently a peak, remove it (BIT update position 3 with -1)
- Position 4: Cannot be a peak (last element)

Update array: `nums[3] = 6` → `nums = [1, 3, 2, 6, 4]`

Recalculate affected positions:
- Position 2: Is `3 < 2 > 6`? No, still not a peak
- Position 3: Is `2 < 6 > 4`? Yes! Add it back (BIT update position 3 with +1)
- Position 4: Still cannot be a peak

BIT state remains: `[0, 1, 0, 1]`

**Step 4: Process Query [1, 0, 4] - Count peaks again**

Adjust range: `[0+1, 4-1]` = `[1, 3]`
- Calculate: `tree.query(3) - tree.query(0)` 
- Result: 2 peaks (positions 1 and 3 are still peaks)

**Final Answer:** `[2, 2]`

The key insight demonstrated here is that the update at position 3 only required checking positions 2, 3, and 4 for their peak status, making the update operation efficient at O(log n) rather than having to recheck the entire array.

## Python Solution

```python
from typing import List


class BinaryIndexedTree:
    """Fenwick Tree for efficient range sum queries and point updates."""
  
    __slots__ = ("size", "tree")
  
    def __init__(self, size: int) -> None:
        """
        Initialize Binary Indexed Tree with given size.
      
        Args:
            size: Maximum index that can be stored (1-indexed)
        """
        self.size = size
        self.tree = [0] * (size + 1)  # 1-indexed array
  
    def update(self, index: int, delta: int) -> None:
        """
        Add delta to element at given index and update tree.
      
        Args:
            index: Position to update (1-indexed)
            delta: Value to add at the position
        """
        while index <= self.size:
            self.tree[index] += delta
            # Move to next index affected by this update
            # Adding the least significant bit moves to parent
            index += index & -index
  
    def query(self, index: int) -> int:
        """
        Calculate prefix sum from index 1 to given index.
      
        Args:
            index: End position for prefix sum (1-indexed)
      
        Returns:
            Sum of elements from 1 to index
        """
        prefix_sum = 0
        while index > 0:
            prefix_sum += self.tree[index]
            # Move to previous range by removing least significant bit
            index -= index & -index
        return prefix_sum


class Solution:
    def countOfPeaks(self, nums: List[int], queries: List[List[int]]) -> List[int]:
        """
        Process queries on array to count peaks in ranges or update values.
        A peak is an element greater than both its neighbors.
      
        Args:
            nums: Input array of integers
            queries: List of queries where:
                     - [1, left, right]: Count peaks in range [left, right]
                     - [2, index, value]: Update nums[index] to value
      
        Returns:
            List of results for type 1 queries
        """
      
        def update_peak_status(position: int, delta: int) -> None:
            """
            Update Binary Indexed Tree if position is a peak.
          
            Args:
                position: Index to check for peak status
                delta: 1 to add peak, -1 to remove peak
            """
            # Skip boundary elements (cannot be peaks)
            if position <= 0 or position >= array_length - 1:
                return
          
            # Check if current position is a peak
            if nums[position - 1] < nums[position] and nums[position] > nums[position + 1]:
                fenwick_tree.update(position, delta)
      
        array_length = len(nums)
        # Initialize BIT to track peaks (excluding first and last elements)
        fenwick_tree = BinaryIndexedTree(array_length - 1)
      
        # Initially mark all peaks in the array
        for i in range(1, array_length - 1):
            update_peak_status(i, 1)
      
        results = []
      
        for query in queries:
            if query[0] == 1:
                # Type 1: Count peaks in range query
                # Adjust range to exclude boundaries (they can't be peaks)
                left_bound = query[1] + 1
                right_bound = query[2] - 1
              
                if left_bound > right_bound:
                    # No valid peaks possible in this range
                    results.append(0)
                else:
                    # Calculate peaks in range using prefix sums
                    peaks_count = fenwick_tree.query(right_bound) - fenwick_tree.query(left_bound - 1)
                    results.append(peaks_count)
          
            else:
                # Type 2: Update value at index
                update_index, new_value = query[1], query[2]
              
                # Remove peak status for affected positions before update
                # Check positions: update_index-1, update_index, update_index+1
                for i in range(update_index - 1, update_index + 2):
                    update_peak_status(i, -1)
              
                # Update the value in array
                nums[update_index] = new_value
              
                # Re-evaluate peak status for affected positions after update
                for i in range(update_index - 1, update_index + 2):
                    update_peak_status(i, 1)
      
        return results
```

## Time and Space Complexity

**Time Complexity:** `O((n + q) × log n)`

The time complexity breaks down as follows:
- **Initialization of BinaryIndexedTree:** Creating the tree takes `O(n)` time to allocate the array.
- **Initial peak detection:** The loop from `1` to `n-1` calls `update(i, 1)` for each position. Each `update` call to the tree takes `O(log n)` time due to the Binary Indexed Tree operations. This gives us `O(n × log n)` for initialization.
- **Processing queries:** For each of the `q` queries:
  - **Type 1 queries (range query):** Calls `tree.query()` twice, each taking `O(log n)` time.
  - **Type 2 queries (update):** 
    - Calls `update(i, -1)` for up to 3 positions (from `idx-1` to `idx+1`), each potentially calling `tree.update()` which takes `O(log n)`.
    - Updates the value in `nums`.
    - Calls `update(i, 1)` for up to 3 positions again, each potentially calling `tree.update()` which takes `O(log n)`.
    - Total per Type 2 query: `O(log n)`
  - Overall for all queries: `O(q × log n)`
- **Total time complexity:** `O(n × log n) + O(q × log n) = O((n + q) × log n)`

**Space Complexity:** `O(n)`

The space complexity consists of:
- **BinaryIndexedTree storage:** The array `c` of size `n+1` requires `O(n)` space.
- **Answer array:** Stores at most `q` results, but this is typically not counted as part of auxiliary space since it's the output.
- **Other variables:** Constant space for loop variables and temporary values.
- **Total space complexity:** `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Errors in BIT Indexing**
The Binary Indexed Tree uses 1-based indexing while the original array uses 0-based indexing. This creates multiple opportunities for indexing errors:

**Pitfall Example:**
```python
# WRONG: Directly using array index for BIT
fenwick_tree.update(i, 1)  # i is 0-based array index

# CORRECT: BIT expects 1-based index for internal positions
fenwick_tree.update(i, 1)  # where i is already adjusted (1 to n-2)
```

**Solution:** 
The code correctly handles this by:
- Creating a BIT of size `n-1` (excluding first and last elements)
- Using positions 1 through n-2 in the BIT to represent array indices 1 through n-2
- In range queries, using `query(right_bound) - query(left_bound - 1)` with proper adjustments

### 2. **Incorrect Range Adjustment for Peak Queries**
When querying peaks in range [l, r], the endpoints cannot be peaks, so the actual searchable range is [l+1, r-1].

**Pitfall Example:**
```python
# WRONG: Querying the full range
if query[0] == 1:
    left, right = query[1], query[2]
    count = fenwick_tree.query(right) - fenwick_tree.query(left - 1)

# CORRECT: Adjust range to exclude boundaries
if query[0] == 1:
    left_bound = query[1] + 1
    right_bound = query[2] - 1
    if left_bound > right_bound:
        results.append(0)
    else:
        count = fenwick_tree.query(right_bound) - fenwick_tree.query(left_bound - 1)
```

### 3. **Incomplete Update of Affected Peaks**
When updating an element, it affects the peak status of three positions: the updated position and its two neighbors. Forgetting to update all three positions is a common mistake.

**Pitfall Example:**
```python
# WRONG: Only updating the changed position
update_peak_status(update_index, -1)
nums[update_index] = new_value
update_peak_status(update_index, 1)

# CORRECT: Update all affected positions
for i in range(update_index - 1, update_index + 2):
    update_peak_status(i, -1)
nums[update_index] = new_value
for i in range(update_index - 1, update_index + 2):
    update_peak_status(i, 1)
```

### 4. **Not Removing Old Peak Status Before Updates**
Failing to remove the old peak status before recalculating can lead to double-counting peaks in the BIT.

**Pitfall Example:**
```python
# WRONG: Directly adding new peak status without removing old
nums[update_index] = new_value
for i in range(update_index - 1, update_index + 2):
    if is_peak(i):
        fenwick_tree.update(i, 1)  # May double-count if already a peak

# CORRECT: Remove old status first, then add new
for i in range(update_index - 1, update_index + 2):
    update_peak_status(i, -1)  # Remove old status
nums[update_index] = new_value
for i in range(update_index - 1, update_index + 2):
    update_peak_status(i, 1)  # Add new status if peak
```

### 5. **Boundary Check Failures in Peak Detection**
The `update_peak_status` function must verify that a position is not at the array boundaries before checking peak conditions.

**Pitfall Example:**
```python
# WRONG: Checking peak condition without boundary validation
def update_peak_status(position: int, delta: int) -> None:
    if nums[position - 1] < nums[position] > nums[position + 1]:  # IndexError!
        fenwick_tree.update(position, delta)

# CORRECT: Check boundaries first
def update_peak_status(position: int, delta: int) -> None:
    if position <= 0 or position >= array_length - 1:
        return
    if nums[position - 1] < nums[position] and nums[position] > nums[position + 1]:
        fenwick_tree.update(position, delta)
```

These pitfalls highlight the importance of careful index management, proper range adjustments, and complete handling of all affected positions during updates when working with Binary Indexed Trees and peak detection algorithms.

