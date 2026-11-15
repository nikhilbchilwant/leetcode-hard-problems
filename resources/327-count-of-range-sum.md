# 327. Count of Range Sum

> Source: https://algo.monster/liteproblems/327

## Problem Statement

You are given an integer array `nums` and two integers `lower` and `upper`. Your task is to count how many range sums fall within the interval `[lower, upper]` (inclusive).

A range sum `S(i, j)` represents the sum of all elements in `nums` from index `i` to index `j` (inclusive), where `i ≤ j`.

For example, if `nums = [1, 2, 3]`, the possible range sums are:
- `S(0, 0) = 1` (sum of elements from index 0 to 0)
- `S(0, 1) = 3` (sum of elements from index 0 to 1)
- `S(0, 2) = 6` (sum of elements from index 0 to 2)
- `S(1, 1) = 2` (sum of elements from index 1 to 1)
- `S(1, 2) = 5` (sum of elements from index 1 to 2)
- `S(2, 2) = 3` (sum of elements from index 2 to 2)

You need to count how many of these range sums have values between `lower` and `upper` (inclusive).

## Intuition

The naive approach would be to check every possible range sum `S(i, j)` by iterating through all pairs of indices, but this would take `O(n²)` time just to generate the sums, making it inefficient for large arrays.

The key insight is to transform the problem using prefix sums. If we compute the prefix sum array where `prefix[i]` represents the sum of elements from index 0 to i-1, then any range sum `S(i, j)` can be expressed as `prefix[j+1] - prefix[i]`.

Now the problem becomes: for each ending position `j`, count how many starting positions `i` (where `i ≤ j`) satisfy:
`lower ≤ prefix[j+1] - prefix[i] ≤ upper`

Rearranging this inequality:
`prefix[j+1] - upper ≤ prefix[i] ≤ prefix[j+1] - lower`

This means for each prefix sum `prefix[j+1]`, we need to count how many previous prefix sums fall within the range `[prefix[j+1] - upper, prefix[j+1] - lower]`.

This is a classic "count elements in range" problem that can be solved efficiently using a data structure like Binary Indexed Tree (BIT). As we process each prefix sum:
1. Query the BIT to count how many previously seen prefix sums fall in our target range
2. Add the current prefix sum to the BIT for future queries

To handle arbitrary values efficiently, we use coordinate compression - mapping all relevant values (prefix sums and their boundaries) to a smaller range of indices that can be used with the BIT. This allows us to work with the relative ordering of values rather than their absolute magnitudes.

The time complexity becomes `O(n log n)` due to sorting for coordinate compression and BIT operations, which is much better than the naive `O(n²)` approach.

## Solution Approach

The solution implements a Binary Indexed Tree (Fenwick Tree) with coordinate compression to efficiently count range sums within the given bounds.

**Step 1: Build Prefix Sum Array**
```python
s = list(accumulate(nums, initial=0))
```
We create a prefix sum array where `s[i]` represents the sum of elements from index 0 to i-1. The `initial=0` parameter ensures `s[0] = 0`, representing an empty prefix.

**Step 2: Coordinate Compression**
```python
arr = sorted(set(v for x in s for v in (x, x - lower, x - upper)))
```
For each prefix sum `x`, we need to query ranges `[x - upper, x - lower]`. We collect all unique values that will be used (the prefix sums themselves and their range boundaries) and sort them. This creates a mapping from actual values to compressed indices (0 to len(arr)-1).

**Step 3: Initialize Binary Indexed Tree**
```python
tree = BinaryIndexedTree(len(arr))
```
The BIT is initialized with size equal to the number of unique values. It supports:
- `update(x, v)`: Add value `v` at position `x`
- `query(x)`: Get sum of values from position 1 to `x`

**Step 4: Process Each Prefix Sum**
```python
for x in s:
    l = bisect_left(arr, x - upper) + 1
    r = bisect_left(arr, x - lower) + 1
    ans += tree.query(r) - tree.query(l - 1)
    tree.update(bisect_left(arr, x) + 1, 1)
```

For each prefix sum `x`:
1. Find the compressed indices for the range `[x - upper, x - lower]`:
   - `l` is the 1-indexed position of `x - upper` in the compressed array
   - `r` is the 1-indexed position of `x - lower` in the compressed array

2. Query the BIT to count how many previous prefix sums fall in this range:
   - `tree.query(r)` gives count of all values ≤ `x - lower`
   - `tree.query(l - 1)` gives count of all values < `x - upper`
   - The difference gives count in range `[x - upper, x - lower]`

3. Add the current prefix sum to the BIT:
   - Find its compressed index using `bisect_left(arr, x) + 1`
   - Update the BIT to record this prefix sum for future queries

**Binary Indexed Tree Implementation**
- `update(x, v)`: Adds `v` to position `x` and propagates updates using `x += x & -x` (adds the least significant bit)
- `query(x)`: Sums values from 1 to `x` by traversing using `x -= x & -x` (removes the least significant bit)

The algorithm processes each prefix sum once with `O(log n)` BIT operations, resulting in overall `O(n log n)` time complexity.

## Example Walkthrough

Let's walk through the solution with a concrete example:
- `nums = [-2, 5, -1]`
- `lower = -2`
- `upper = 2`

**Step 1: Build Prefix Sum Array**
```
s = [0, -2, 3, 2]
```
- `s[0] = 0` (empty prefix)
- `s[1] = 0 + (-2) = -2`
- `s[2] = -2 + 5 = 3`
- `s[3] = 3 + (-1) = 2`

**Step 2: Coordinate Compression**
For each prefix sum `x` in `s`, we need values: `x`, `x - lower`, and `x - upper`:
- For `x = 0`: values are `0`, `0 - (-2) = 2`, `0 - 2 = -2`
- For `x = -2`: values are `-2`, `-2 - (-2) = 0`, `-2 - 2 = -4`
- For `x = 3`: values are `3`, `3 - (-2) = 5`, `3 - 2 = 1`
- For `x = 2`: values are `2`, `2 - (-2) = 4`, `2 - 2 = 0`

Sorted unique values: `arr = [-4, -2, 0, 1, 2, 3, 4, 5]`

**Step 3: Initialize BIT**
Create a BIT of size 8 (all initially 0).

**Step 4: Process Each Prefix Sum**

**Processing `s[0] = 0`:**
- Range to query: `[0 - 2, 0 - (-2)]` = `[-2, 2]`
- Compressed indices: `l = 2`, `r = 5` (1-indexed)
- Query BIT: `tree.query(5) - tree.query(1) = 0 - 0 = 0`
- No previous prefix sums exist, so count = 0
- Update BIT at position 3 (compressed index of 0)
- BIT state: position 3 has value 1

**Processing `s[1] = -2`:**
- Range to query: `[-2 - 2, -2 - (-2)]` = `[-4, 0]`
- Compressed indices: `l = 1`, `r = 3`
- Query BIT: `tree.query(3) - tree.query(0) = 1 - 0 = 1`
- Found 1 valid range: `S(0,0) = -2 - 0 = -2` ✓ (within [-2, 2])
- Update BIT at position 2 (compressed index of -2)
- Running count: 1

**Processing `s[2] = 3`:**
- Range to query: `[3 - 2, 3 - (-2)]` = `[1, 5]`
- Compressed indices: `l = 4`, `r = 8`
- Query BIT: `tree.query(8) - tree.query(3) = 2 - 2 = 0` (both stored prefix sums, 0 and -2, are below 1)
- No valid ranges ending at index 1
- Update BIT at position 6 (compressed index of 3)
- Running count: 1

**Processing `s[3] = 2`:**
- Range to query: `[2 - 2, 2 - (-2)]` = `[0, 4]`
- Compressed indices: `l = 3`, `r = 7`
- Query BIT: `tree.query(7) - tree.query(2) = 3 - 1 = 2`
- The stored prefix sums in `[0, 4]` are `0` and `3`, giving two valid ranges:
  - `S(0,2) = 2 - 0 = 2` ✓
  - `S(2,2) = 2 - 3 = -1` ✓
- Update BIT at position 5 (compressed index of 2)
- Running count: 3

**Final Answer: 3**
The valid range sums are:
1. `S(0,0) = -2`
2. `S(0,2) = 2`  
3. `S(2,2) = -1`

## Python Solution

```python
from typing import List
from itertools import accumulate
from bisect import bisect_left


class BinaryIndexedTree:
    """
    Binary Indexed Tree (Fenwick Tree) for efficient range sum queries and updates.
    Uses 1-based indexing internally.
    """
  
    def __init__(self, n: int) -> None:
        """
        Initialize a Binary Indexed Tree with size n.
      
        Args:
            n: The size of the tree
        """
        self.size = n
        self.tree = [0] * (n + 1)  # 1-indexed array
  
    def update(self, index: int, delta: int) -> None:
        """
        Add delta to the element at the given index.
      
        Args:
            index: The position to update (1-based)
            delta: The value to add
        """
        while index <= self.size:
            self.tree[index] += delta
            # Move to next index by adding the least significant bit
            index += index & (-index)
  
    def query(self, index: int) -> int:
        """
        Get the prefix sum from index 1 to the given index.
      
        Args:
            index: The ending position for the prefix sum (1-based)
          
        Returns:
            The sum of elements from 1 to index
        """
        result = 0
        while index > 0:
            result += self.tree[index]
            # Move to parent by removing the least significant bit
            index -= index & (-index)
        return result


class Solution:
    def countRangeSum(self, nums: List[int], lower: int, upper: int) -> int:
        """
        Count the number of range sums that lie in [lower, upper].
        A range sum S(i, j) is defined as the sum of elements from index i to j (inclusive).
      
        Args:
            nums: The input array
            lower: The lower bound of the range
            upper: The upper bound of the range
          
        Returns:
            The count of valid range sums
        """
        # Calculate prefix sums (including initial 0 for empty prefix)
        prefix_sums = list(accumulate(nums, initial=0))
      
        # Create a sorted array of all unique values we need to track
        # For each prefix sum x, we need x itself, x-lower, and x-upper
        unique_values = set()
        for prefix_sum in prefix_sums:
            unique_values.add(prefix_sum)
            unique_values.add(prefix_sum - lower)
            unique_values.add(prefix_sum - upper)
      
        # Sort unique values for coordinate compression
        sorted_values = sorted(unique_values)
      
        # Initialize Binary Indexed Tree with compressed coordinate space
        fenwick_tree = BinaryIndexedTree(len(sorted_values))
      
        count = 0
      
        # Process each prefix sum
        for current_sum in prefix_sums:
            # Find the range of valid previous prefix sums
            # We need previous_sum such that: lower <= current_sum - previous_sum <= upper
            # Which means: current_sum - upper <= previous_sum <= current_sum - lower
          
            # Get compressed coordinates (1-based for BIT)
            left_bound = bisect_left(sorted_values, current_sum - upper) + 1
            right_bound = bisect_left(sorted_values, current_sum - lower) + 1
          
            # Count how many previous prefix sums fall in this range
            count += fenwick_tree.query(right_bound) - fenwick_tree.query(left_bound - 1)
          
            # Add current prefix sum to the tree for future queries
            current_position = bisect_left(sorted_values, current_sum) + 1
            fenwick_tree.update(current_position, 1)
      
        return count
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)`

The algorithm consists of several main steps:

1. Computing prefix sums using `accumulate`: `O(n)`
2. Creating and sorting the coordinate compression array `arr`: 
   - Creating the set of values (at most `3n` elements): `O(n)`
   - Sorting: `O(n log n)`
3. Main loop iterating through prefix sums:
   - For each of `n+1` prefix sums:
     - Two binary searches using `bisect_left`: `O(log n)` each
     - Two BIT query operations: `O(log n)` each
     - One BIT update operation: `O(log n)`
   - Total for loop: `O(n log n)`

Overall time complexity: `O(n) + O(n log n) + O(n log n) = O(n log n)`

**Space Complexity:** `O(n)`

The space usage includes:
1. Prefix sum array `s`: `O(n)`
2. Coordinate compression array `arr`: `O(n)` (at most `3n` unique elements)
3. Binary Indexed Tree array `c`: `O(n)` (size proportional to compressed coordinates)
4. Temporary set for deduplication: `O(n)`

Overall space complexity: `O(n)`

## Common Pitfalls

### 1. **Incorrect Range Boundary Calculation**
One of the most common mistakes is incorrectly calculating which previous prefix sums create valid range sums. Developers often confuse the inequality direction or bounds.

**Pitfall Example:**
```python
# WRONG: Reversed inequality
left_bound = bisect_left(sorted_values, current_sum - lower) + 1  # Wrong!
right_bound = bisect_left(sorted_values, current_sum - upper) + 1  # Wrong!
```

**Why it's wrong:** For a range sum `S(i, j) = prefix[j+1] - prefix[i]` to be valid, we need:
- `lower ≤ prefix[j+1] - prefix[i] ≤ upper`
- Rearranging: `prefix[j+1] - upper ≤ prefix[i] ≤ prefix[j+1] - lower`

So when looking for valid previous prefix sums given current prefix sum `x`, we need the range `[x - upper, x - lower]`, not `[x - lower, x - upper]`.

**Correct Solution:**
```python
left_bound = bisect_left(sorted_values, current_sum - upper) + 1
right_bound = bisect_left(sorted_values, current_sum - lower) + 1
```

### 2. **Forgetting to Include Values in Coordinate Compression**
Another pitfall is not including all necessary values in the coordinate compression step, leading to incorrect index lookups.

**Pitfall Example:**
```python
# WRONG: Only compressing the prefix sums themselves
unique_values = set(prefix_sums)  # Missing boundary values!
```

**Why it's wrong:** We need to query ranges `[x - upper, x - lower]` for each prefix sum `x`. If these boundary values aren't in our compressed coordinate system, we can't properly find their positions.

**Correct Solution:**
```python
unique_values = set()
for prefix_sum in prefix_sums:
    unique_values.add(prefix_sum)
    unique_values.add(prefix_sum - lower)
    unique_values.add(prefix_sum - upper)
```

### 3. **Off-by-One Errors with BIT Indexing**
Binary Indexed Trees use 1-based indexing, while Python uses 0-based indexing. Mixing these can cause subtle bugs.

**Pitfall Example:**
```python
# WRONG: Using 0-based index directly with BIT
position = bisect_left(sorted_values, current_sum)  # 0-based
fenwick_tree.update(position, 1)  # BIT expects 1-based!
```

**Correct Solution:**
```python
position = bisect_left(sorted_values, current_sum) + 1  # Convert to 1-based
fenwick_tree.update(position, 1)
```

### 4. **Processing Order: Query Before Update**
The order of operations is crucial. We must query for valid previous prefix sums before adding the current one to avoid counting invalid ranges.

**Pitfall Example:**
```python
# WRONG: Updating before querying
for current_sum in prefix_sums:
    fenwick_tree.update(bisect_left(sorted_values, current_sum) + 1, 1)  # Wrong order!
    count += fenwick_tree.query(right_bound) - fenwick_tree.query(left_bound - 1)
```

**Why it's wrong:** If we update first, the current prefix sum is compared with itself. That pair represents an empty range with sum `x - x = 0`, so whenever `lower <= 0 <= upper` every prefix sum adds one extra, invalid count.

**Correct Solution:**
```python
for current_sum in prefix_sums:
    # Query first to find valid ranges with previous prefix sums
    count += fenwick_tree.query(right_bound) - fenwick_tree.query(left_bound - 1)
    # Then update with current prefix sum
    fenwick_tree.update(bisect_left(sorted_values, current_sum) + 1, 1)
```

