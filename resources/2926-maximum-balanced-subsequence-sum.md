# 2926. Maximum Balanced Subsequence Sum

> Source: https://algo.monster/liteproblems/2926

## Problem Statement

You are given a 0-indexed integer array `nums`.

A subsequence of `nums` with length `k` and consisting of indices `i₀ < i₁ < ... < i_{k-1}` is called **balanced** if it satisfies the following condition:

For every `j` in the range `[1, k-1]`: `nums[i_j] - nums[i_{j-1}] >= i_j - i_{j-1}`

In other words, the difference between consecutive values in the subsequence must be at least as large as the difference between their indices.

A subsequence of length 1 is automatically considered balanced.

Your task is to find the **maximum possible sum** of elements in any balanced subsequence of `nums`.

**Key Points:**
- A subsequence maintains the relative order of elements from the original array but doesn't need to be contiguous
- The balance condition requires that the value increase between consecutive elements is at least as much as the index increase
- You need to maximize the sum of the selected elements while maintaining the balance property

**Example Understanding:**
If you select indices `2` and `5` from the array, then:
- The values are `nums[2]` and `nums[5]`
- The balance condition requires: `nums[5] - nums[2] >= 5 - 2 = 3`
- These two indices can form a balanced subsequence only if the value at index 5 is at least 3 more than the value at index 2

## Intuition

Let's start by understanding what the balance condition really means. We have `nums[i_j] - nums[i_{j-1}] >= i_j - i_{j-1}` for consecutive elements in our subsequence.

If we rearrange this inequality, we get:
`nums[i_j] - i_j >= nums[i_{j-1}] - i_{j-1}`

This is a key insight! If we create a new array `arr` where `arr[i] = nums[i] - i`, then the balance condition simply becomes: for any two consecutive indices in our subsequence, the corresponding values in `arr` must be non-decreasing.

In other words, a balanced subsequence in the original array corresponds to selecting indices where the `arr` values form a non-decreasing sequence.

Now our problem transforms into: given the array `arr`, select a subset of indices such that:
1. The `arr` values at these indices are non-decreasing
2. The sum of the corresponding `nums` values is maximized

This looks like a [dynamic programming](/problems/dynamic_programming_intro) problem where for each index `i`, we need to find the best subsequence ending at `i`. We can define `f[i]` as the maximum sum achievable by a balanced subsequence ending at index `i`.

For each position `i`, we look at all previous positions `j` where `arr[j] <= arr[i]` (meaning we can extend the subsequence from `j` to `i`), and choose the one that gives us the maximum sum. The recurrence is:
`f[i] = max(f[j] for all j < i where arr[j] <= arr[i]) + nums[i]`

However, finding all such `j` values for each `i` would be O(n²). To optimize this, we need a data structure that can efficiently:
1. Query the maximum `f[j]` value among all `j` where `arr[j] <= arr[i]`
2. Update the value at a given position

A Binary Indexed Tree (Fenwick Tree) is perfect for this! We can use coordinate compression to map the `arr` values to a smaller range, then use the BIT to maintain prefix maximums. This reduces our complexity to O(n log n).

**Learn more about [Segment Tree](/problems/segment_tree_intro), [Binary Search](/problems/binary_search_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

Based on our intuition, let's implement the solution step by step:

**Step 1: Transform the Problem**

First, we create the transformed array `arr` where `arr[i] = nums[i] - i`:
```python
arr = [x - i for i, x in enumerate(nums)]
```

**Step 2: Coordinate Compression**

Since we'll use a Binary Indexed Tree indexed by the values in `arr`, we need to compress these values to a smaller range `[1, m]` where `m` is the number of unique values:
```python
s = sorted(set(arr))  # Get unique values and sort them
```

This creates a mapping where each unique value in `arr` gets an index from 1 to `len(s)`.

**Step 3: Binary Indexed Tree for Maximum Queries**

We implement a BIT that supports:
- `update(x, v)`: Update position `x` with value `v` (keeping the maximum)
- `query(x)`: Get the maximum value in the range `[1, x]`

The BIT is initialized with negative infinity values since we're looking for maximums:
```python
self.c = [-inf] * (n + 1)
```

**Step 4: [Dynamic Programming](/problems/dynamic_programming_intro) with BIT**

For each element in `nums`, we:

1. Find the compressed index of `arr[i] = nums[i] - i`:
```python
j = bisect_left(s, x - i) + 1  # +1 because BIT is 1-indexed
```

2. Query the maximum sum of all subsequences whose last element has `arr` value ≤ current `arr[i]`:
```python
v = max(tree.query(j), 0) + x
```
We take `max(..., 0)` because we can start a new subsequence from the current element.

3. Update the BIT with the maximum sum ending at current position:
```python
tree.update(j, v)
```

**Step 5: Get the Final Answer**

After processing all elements, the answer is the maximum value stored in the entire BIT:
```python
return tree.query(len(s))
```

**Time Complexity Analysis:**
- Sorting unique values: O(n log n)
- For each element, we do [binary search](/problems/binary_search_intro) and BIT operations: O(n log n)
- Overall: O(n log n)

**Space Complexity:** O(n) for the BIT and auxiliary arrays.

The key insight is that by transforming the problem and using coordinate compression with a BIT, we efficiently maintain the maximum sums for all valid prefixes, allowing us to build the optimal balanced subsequence incrementally.

## Example Walkthrough

Let's walk through a concrete example with `nums = [3, 3, 5, 6]`.

**Step 1: Transform the array**
We create `arr[i] = nums[i] - i`:
- `arr[0] = 3 - 0 = 3`
- `arr[1] = 3 - 1 = 2`
- `arr[2] = 5 - 2 = 3`
- `arr[3] = 6 - 3 = 3`

So `arr = [3, 2, 3, 3]`

**Step 2: Coordinate Compression**
Get unique sorted values: `s = [2, 3]`
This maps:
- Value 2 → compressed index 1
- Value 3 → compressed index 2

**Step 3: Initialize BIT**
Create a BIT of size 3 (len(s) + 1) initialized with `-inf`

**Step 4: Process each element**

*Processing index 0 (nums[0] = 3):*
- `arr[0] = 3` → compressed index = 2
- Query BIT for max in range [1, 2]: returns `-inf`
- Calculate: `v = max(-inf, 0) + 3 = 0 + 3 = 3`
- Update BIT at index 2 with value 3
- BIT state: `[-inf, -inf, 3]`

*Processing index 1 (nums[1] = 3):*
- `arr[1] = 2` → compressed index = 1
- Query BIT for max in range [1, 1]: returns `-inf`
- Calculate: `v = max(-inf, 0) + 3 = 0 + 3 = 3`
- Update BIT at index 1 with value 3
- BIT state: `[-inf, 3, 3]`

*Processing index 2 (nums[2] = 5):*
- `arr[2] = 3` → compressed index = 2
- Query BIT for max in range [1, 2]: returns `max(3, 3) = 3`
- Calculate: `v = max(3, 0) + 5 = 3 + 5 = 8`
- Update BIT at index 2 with value 8
- BIT state: `[-inf, 3, 8]`

*Processing index 3 (nums[3] = 6):*
- `arr[3] = 3` → compressed index = 2
- Query BIT for max in range [1, 2]: returns `max(3, 8) = 8`
- Calculate: `v = max(8, 0) + 6 = 8 + 6 = 14`
- Update BIT at index 2 with value 14
- BIT state: `[-inf, 3, 14]`

**Step 5: Get the answer**
Query the entire BIT: `tree.query(2) = 14`

**Verification:**
The optimal subsequence is indices [1, 2, 3] with values [3, 5, 6]:
- Check balance between indices 1 and 2: `5 - 3 = 2 >= 2 - 1 = 1` ✓
- Check balance between indices 2 and 3: `6 - 5 = 1 >= 3 - 2 = 1` ✓
- Sum = 3 + 5 + 6 = 14 ✓

This subsequence works because in the transformed array, we have `arr[1] = 2 ≤ arr[2] = 3 ≤ arr[3] = 3`, forming a non-decreasing sequence.

## Python Solution

```python
from typing import List
from bisect import bisect_left

class BinaryIndexedTree:
    """
    Binary Indexed Tree (Fenwick Tree) for range maximum queries.
    This implementation supports point updates and prefix maximum queries.
    """

    def __init__(self, n: int):
        """
        Initialize the Binary Indexed Tree with size n.

        Args:
            n: The size of the tree (1-indexed)
        """
        self.size = n
        # Tree array initialized with negative infinity (1-indexed)
        self.tree = [float('-inf')] * (n + 1)

    def update(self, index: int, value: int):
        """
        Update the tree at position index with maximum of current and new value.

        Args:
            index: The position to update (1-indexed)
            value: The new value to consider for maximum
        """
        while index <= self.size:
            # Update current position with maximum value
            self.tree[index] = max(self.tree[index], value)
            # Move to next position affected by this update
            # Adding the least significant bit moves to parent in update tree
            index += index & -index

    def query(self, index: int) -> int:
        """
        Query the maximum value in range [1, index].

        Args:
            index: The right boundary of the query range (1-indexed)

        Returns:
            The maximum value in the range [1, index]
        """
        maximum_value = float('-inf')
        while index > 0:
            # Consider current position's value
            maximum_value = max(maximum_value, self.tree[index])
            # Move to previous range by removing least significant bit
            index -= index & -index
        return maximum_value


class Solution:
    def maxBalancedSubsequenceSum(self, nums: List[int]) -> int:
        """
        Find the maximum sum of a balanced subsequence.
        A subsequence is balanced if for any two consecutive elements at positions i and j,
        nums[j] - nums[i] >= j - i.

        Args:
            nums: The input array of integers

        Returns:
            The maximum sum of a balanced subsequence
        """
        # Transform the constraint: nums[j] - nums[i] >= j - i
        # Rearranging: nums[j] - j >= nums[i] - i
        # Create array of transformed values
        transformed_values = [nums[i] - i for i in range(len(nums))]

        # Compress coordinates: map transformed values to ranks
        # This allows us to use them as indices in the BIT
        sorted_unique_values = sorted(set(transformed_values))

        # Initialize Binary Indexed Tree with compressed coordinate space size
        bit = BinaryIndexedTree(len(sorted_unique_values))

        # Process each element in the original array
        for i, current_num in enumerate(nums):
            # Find the compressed coordinate (rank) of the transformed value
            # Add 1 because BIT is 1-indexed
            compressed_index = bisect_left(sorted_unique_values, current_num - i) + 1

            # Query maximum sum ending at or before current transformed value
            # Take max with 0 to allow starting a new subsequence
            max_previous_sum = max(bit.query(compressed_index), 0)

            # Calculate new sum including current element
            new_sum = max_previous_sum + current_num

            # Update the BIT with the new maximum sum at this position
            bit.update(compressed_index, new_sum)

        # Return the overall maximum sum found
        return bit.query(len(sorted_unique_values))
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The time complexity is dominated by several operations:
- Creating the array `arr` with `x - i` for each element: `O(n)`
- Sorting the unique elements: `O(n × log n)` in the worst case when all elements are unique
- The main loop iterates `n` times, and for each iteration:
  - `bisect_left` performs binary search: `O(log n)`
  - `tree.query(j)` calls the BIT query operation, which loops at most `O(log n)` times (following the path down by removing the least significant bit)
  - `tree.update(j, v)` calls the BIT update operation, which loops at most `O(log n)` times (following the path up by adding the least significant bit)
- Final `tree.query(len(s))`: `O(log n)`

Overall: `O(n) + O(n × log n) + O(n × log n) = O(n × log n)`

**Space Complexity:** `O(n)`

The space complexity consists of:
- Array `arr` storing transformed values: `O(n)`
- Set and sorted array `s` storing unique values: `O(n)` in the worst case
- BinaryIndexedTree with array `c` of size `n + 1`: `O(n)`
- Additional variables and temporary storage: `O(1)`

Overall: `O(n) + O(n) + O(n) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Negative Values in BIT Initialization**

**Pitfall:** One common mistake is initializing the BIT with 0 instead of negative infinity. This causes incorrect results when all possible balanced subsequences have negative sums.

**Why it happens:** When querying `bit.query(compressed_index)`, if no valid subsequence has been found yet, the BIT should return negative infinity, not 0. If initialized with 0, the algorithm might incorrectly assume there's a valid empty subsequence with sum 0.

**Example scenario:**
```python
nums = [-10, -20, -30]  # All negative values
```
With incorrect initialization (using 0), the algorithm might return 0 instead of the correct answer -10 (taking just the first element).

**Solution:** Always initialize the BIT with `float('-inf')`:
```python
self.tree = [float('-inf')] * (n + 1)  # Correct
# NOT: self.tree = [0] * (n + 1)  # Wrong!
```

### 2. **Off-by-One Error in BIT Indexing**

**Pitfall:** Forgetting that BIT is 1-indexed while coordinate compression returns 0-indexed positions.

**Why it happens:** `bisect_left` returns a 0-indexed position, but BIT operations expect 1-indexed positions.

**Incorrect code:**
```python
compressed_index = bisect_left(sorted_unique_values, current_num - i)  # Wrong!
bit.update(compressed_index, new_sum)  # This will fail for index 0
```

**Solution:** Always add 1 when converting from bisect result to BIT index:
```python
compressed_index = bisect_left(sorted_unique_values, current_num - i) + 1  # Correct
```

### 3. **Misunderstanding the Transformation Logic**

**Pitfall:** Incorrectly transforming the balance condition, such as using `nums[i] + i` instead of `nums[i] - i`.

**Why it happens:** The balance condition `nums[j] - nums[i] >= j - i` needs careful algebraic manipulation:
- Rearranging: `nums[j] - j >= nums[i] - i`
- This means we need `arr[i] = nums[i] - i`, not `nums[i] + i`

**Incorrect transformation:**
```python
transformed_values = [nums[i] + i for i in range(len(nums))]  # Wrong!
```

**Solution:** Use the correct transformation:
```python
transformed_values = [nums[i] - i for i in range(len(nums))]  # Correct
```

### 4. **Not Handling the Case of Starting a New Subsequence**

**Pitfall:** Forgetting that a single element can form a valid balanced subsequence, leading to missing optimal solutions.

**Why it happens:** When calculating the maximum sum at position i, we need to consider both:
1. Extending a previous subsequence
2. Starting a new subsequence from the current element

**Incorrect code:**
```python
max_previous_sum = bit.query(compressed_index)  # Wrong!
new_sum = max_previous_sum + current_num  # Might use -inf
```

**Solution:** Take the maximum with 0 to allow starting fresh:
```python
max_previous_sum = max(bit.query(compressed_index), 0)  # Correct
new_sum = max_previous_sum + current_num
```

### 5. **Using Standard BIT for Sum Instead of Maximum**

**Pitfall:** Using a standard BIT implementation that calculates prefix sums instead of prefix maximums.

**Why it happens:** Most BIT tutorials and implementations focus on range sum queries, but this problem requires range maximum queries.

**Solution:** Modify the BIT operations to use `max` instead of addition:
```python
# In update method:
self.tree[index] = max(self.tree[index], value)  # Use max, not +=

# In query method:
maximum_value = max(maximum_value, self.tree[index])  # Use max, not +=
```

