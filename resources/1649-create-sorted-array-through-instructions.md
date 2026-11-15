# 1649. Create Sorted Array through Instructions

> Source: https://algo.monster/liteproblems/1649

## Problem Statement

You need to build a sorted array by inserting elements one by one from a given integer array `instructions`. Starting with an empty array `nums`, you process each element from `instructions` in order from left to right.

For each element you insert, there's a cost associated with the insertion. The cost is calculated as the **minimum** of:
- The count of elements in `nums` that are strictly less than the current element
- The count of elements in `nums` that are strictly greater than the current element

For example, if you're inserting `3` into `nums = [1,2,3,5]`:
- Elements less than `3`: `1` and `2` (count = 2)
- Elements greater than `3`: `5` (count = 1)
- Cost = `min(2, 1) = 1`
- After insertion: `nums = [1,2,3,3,5]`

The goal is to calculate the total cost of inserting all elements from `instructions` into `nums`. Since the final answer can be very large, return it modulo `10^9 + 7`.

The solution uses a Binary Indexed Tree (Fenwick Tree) to efficiently track and query the count of elements. The BIT maintains frequencies of values, allowing us to:
- Query how many elements are less than a value `x` using `tree.query(x - 1)`
- Calculate elements greater than `x` as `i - tree.query(x)` where `i` is the total number of elements inserted so far
- Update the frequency count when inserting a new element using `tree.update(x, 1)`

## Intuition

The key insight is recognizing that we need to efficiently count elements less than and greater than a given value as we build our array. If we were to use a simple sorted list and [binary search](/problems/binary_search_intro) for each insertion, we'd face `O(n)` insertion time, leading to `O(n^2)` overall complexity.

We observe that:
1. We don't actually need to maintain the sorted array itself - we only need to track how many of each value we've seen
2. For any value `x`, we need two pieces of information:
   - Count of all values strictly less than `x`
   - Count of all values strictly greater than `x`

This naturally leads us to think about frequency counting. If we maintain frequencies, then:
- Count of elements less than `x` = sum of frequencies from `1` to `x-1`
- Count of elements greater than `x` = total elements inserted - count of elements less than or equal to `x`

The challenge becomes: how do we efficiently update frequencies and calculate prefix sums? A naive approach would require `O(m)` time for each prefix sum query where `m` is the range of values.

This is where the Binary Indexed Tree (Fenwick Tree) becomes the perfect data structure. It allows us to:
- Update a frequency in `O(log m)` time
- Query a prefix sum (count of elements ≤ some value) in `O(log m)` time

The BIT works by storing partial sums in a clever way using binary representation. Each index in the BIT is responsible for a range of elements determined by the lowest set bit in its binary representation. This structure enables both updates and queries to traverse only `O(log m)` nodes.

By using the BIT to track frequencies and compute range sums efficiently, we reduce the overall complexity from `O(n^2)` to `O(n log m)`, making the solution scalable for large inputs.

**Learn more about [Segment Tree](/problems/segment_tree_intro), [Binary Search](/problems/binary_search_intro), [Divide and Conquer](/problems/divide_and_conquer_intro) and [Merge Sort](/problems/merge_k_sorted_lists) patterns.**

## Solution Approach

The solution implements a Binary Indexed Tree (Fenwick Tree) to efficiently track element frequencies and compute range queries.

**Binary Indexed Tree Implementation:**

The `BinaryIndexedTree` class maintains an array `c` of size `n+1` where:
- `c[x]` stores partial sums based on the binary representation of index `x`
- Each index is responsible for a range determined by its lowest set bit

**Update Operation** (`update(x, v)`):
```python
while x <= self.n:
    self.c[x] += v
    x += x & -x
```
- Adds value `v` to position `x`
- `x & -x` extracts the lowest set bit (e.g., `12 & -12 = 4`)
- Updates all nodes that include position `x` in their range
- Traverses up the tree by adding the lowest set bit

**Query Operation** (`query(x)`):
```python
s = 0
while x:
    s += self.c[x]
    x -= x & -x
return s
```
- Computes the prefix sum from positions `1` to `x`
- Traverses down by removing the lowest set bit each iteration
- Accumulates partial sums from responsible nodes

**Main Algorithm:**

1. **Initialize BIT**: Create a tree with size equal to the maximum value in `instructions`
   ```python
   m = max(instructions)
   tree = BinaryIndexedTree(m)
   ```

2. **Process each instruction**: For each element `x` at position `i`:

   a. **Calculate insertion cost**:
   ```python
   cost = min(tree.query(x - 1), i - tree.query(x))
   ```
   - `tree.query(x - 1)`: Count of elements strictly less than `x`
   - `tree.query(x)`: Count of elements less than or equal to `x`
   - `i - tree.query(x)`: Count of elements strictly greater than `x`
   - The cost is the minimum of these two counts

   b. **Update the tree**:
   ```python
   tree.update(x, 1)
   ```
   - Increment the frequency of value `x` by 1

   c. **Accumulate total cost**:
   ```python
   ans += cost
   ```

3. **Return result modulo `10^9 + 7`**:
   ```python
   return ans % mod
   ```

**Time Complexity**: `O(n log m)` where `n` is the length of instructions and `m` is the maximum value
- Each update and query operation takes `O(log m)`
- We perform `n` iterations

**Space Complexity**: `O(m)` for the BIT array

The elegance of this solution lies in using the BIT to maintain a dynamic frequency array that supports efficient range sum queries, avoiding the need to maintain an actual sorted array.

## Example Walkthrough

Let's walk through the solution with `instructions = [1, 5, 6, 2]`.

**Initial Setup:**
- Maximum value `m = 6`, so we create a BIT of size 6
- BIT array `c = [0, 0, 0, 0, 0, 0, 0]` (index 0 unused)
- Total cost = 0

**Step 1: Insert 1 (i = 0)**
- Query elements < 1: `tree.query(0) = 0`
- Query elements ≤ 1: `tree.query(1) = 0`
- Elements > 1: `0 - 0 = 0`
- Cost = `min(0, 0) = 0`
- Update BIT with value 1: `tree.update(1, 1)`
  - Update index 1: `c[1] += 1` → `c = [0, 1, 0, 0, 0, 0, 0]`
  - Next index: `1 + (1 & -1) = 2`
  - Update index 2: `c[2] += 1` → `c = [0, 1, 1, 0, 0, 0, 0]`
  - Next index: `2 + (2 & -2) = 4`
  - Update index 4: `c[4] += 1` → `c = [0, 1, 1, 0, 1, 0, 0]`
- Total cost = 0

**Step 2: Insert 5 (i = 1)**
- Query elements < 5: `tree.query(4)`
  - At index 4: accumulate `c[4] = 1`
  - Total = 1
- Elements > 5: `1 - tree.query(5) = 1 - 1 = 0`
- Cost = `min(1, 0) = 0`
- Update BIT with value 5: `tree.update(5, 1)`
  - Update index 5: `c[5] += 1` → `c = [0, 1, 1, 0, 1, 1, 0]`
  - Next index: `5 + (5 & -5) = 6`
  - Update index 6: `c[6] += 1` → `c = [0, 1, 1, 0, 1, 1, 1]`
- Total cost = 0

**Step 3: Insert 6 (i = 2)**
- Query elements < 6: `tree.query(5)`
  - At index 5: accumulate `c[5] = 1`
  - Next: `5 - (5 & -5) = 4`
  - At index 4: accumulate `c[4] = 1`
  - Total = 2
- Elements > 6: `2 - tree.query(6) = 2 - 2 = 0`
- Cost = `min(2, 0) = 0`
- Update BIT with value 6: `tree.update(6, 1)`
  - Update index 6: `c[6] += 1` → `c = [0, 1, 1, 0, 1, 1, 2]`
- Total cost = 0

**Step 4: Insert 2 (i = 3)**
- Query elements < 2: `tree.query(1)`
  - At index 1: accumulate `c[1] = 1`
  - Total = 1
- Query elements ≤ 2: `tree.query(2)`
  - At index 2: accumulate `c[2] = 1`
  - Total = 1
- Elements > 2: `3 - 1 = 2`
- Cost = `min(1, 2) = 1`
- Update BIT with value 2: `tree.update(2, 1)`
  - Update index 2: `c[2] += 1` → `c = [0, 1, 2, 0, 1, 1, 2]`
  - Next index: `2 + (2 & -2) = 4`
  - Update index 4: `c[4] += 1` → `c = [0, 1, 2, 0, 2, 1, 2]`
- Total cost = 1

**Final Result:** Total cost = 1

The sorted array would be `[1, 2, 5, 6]` with insertion costs of `[0, 0, 0, 1]`.

## Python Solution

```python
class BinaryIndexedTree:
    """Fenwick Tree (Binary Indexed Tree) for efficient prefix sum queries and updates."""

    def __init__(self, n: int) -> None:
        """
        Initialize a Binary Indexed Tree with size n.

        Args:
            n: The maximum value that can be indexed (1-indexed)
        """
        self.size = n
        self.tree = [0] * (n + 1)  # 1-indexed array for BIT

    def update(self, index: int, delta: int) -> None:
        """
        Add delta to the value at index and update the tree accordingly.

        Args:
            index: The position to update (1-indexed)
            delta: The value to add at the position
        """
        while index <= self.size:
            self.tree[index] += delta
            # Move to next index by adding the lowest set bit
            index += index & (-index)

    def query(self, index: int) -> int:
        """
        Get the prefix sum from 1 to index (inclusive).

        Args:
            index: The end position for prefix sum (1-indexed)

        Returns:
            The sum of elements from position 1 to index
        """
        prefix_sum = 0
        while index > 0:
            prefix_sum += self.tree[index]
            # Move to parent by removing the lowest set bit
            index -= index & (-index)
        return prefix_sum


class Solution:
    def createSortedArray(self, instructions: List[int]) -> int:
        """
        Calculate minimum cost to create sorted array using instructions.

        For each element, the cost is the minimum of:
        - Count of strictly smaller elements already inserted
        - Count of strictly larger elements already inserted

        Args:
            instructions: List of integers to insert in order

        Returns:
            Total minimum cost modulo 10^9 + 7
        """
        # Find maximum value to determine BIT size
        max_value = max(instructions)

        # Initialize Binary Indexed Tree to track element frequencies
        bit = BinaryIndexedTree(max_value)

        total_cost = 0
        MOD = 10**9 + 7

        for position, value in enumerate(instructions):
            # Count elements strictly less than current value
            smaller_count = bit.query(value - 1)

            # Count elements strictly greater than current value
            # Total elements so far minus elements less than or equal to current
            greater_count = position - bit.query(value)

            # Add minimum cost for this insertion
            cost = min(smaller_count, greater_count)
            total_cost += cost

            # Update BIT to include current element
            bit.update(value, 1)

        return total_cost % MOD
```

## Time and Space Complexity

**Time Complexity:** `O(n * log m)` where `n` is the length of the instructions array and `m` is the maximum value in the instructions array.

- The main loop iterates through all `n` instructions
- For each instruction, we perform:
  - Two `query` operations: `tree.query(x - 1)` and `tree.query(x)`, each taking `O(log m)` time
  - One `update` operation: `tree.update(x, 1)`, which takes `O(log m)` time
- The Binary Indexed Tree operations (`update` and `query`) have `O(log m)` complexity because:
  - In `update`, we traverse up the tree by adding `x & -x` (the least significant bit), which takes at most `log m` steps
  - In `query`, we traverse down by subtracting `x & -x`, which also takes at most `log m` steps
- Finding the maximum value initially takes `O(n)` time
- Total: `O(n) + n * O(log m) = O(n * log m)`

**Space Complexity:** `O(m)` where `m` is the maximum value in the instructions array.

- The Binary Indexed Tree uses an array `c` of size `m + 1`, requiring `O(m)` space
- The `instructions` input array is given and not counted as extra space
- Other variables (`ans`, `mod`, `i`, `x`, `cost`) use `O(1)` space
- Total: `O(m)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Error in Counting Strictly Less/Greater Elements**

A frequent mistake is incorrectly calculating the count of elements strictly less than or strictly greater than the current value.

**Incorrect Implementation:**
```python
# Wrong: This counts elements less than OR EQUAL to value
smaller_count = bit.query(value)  # Should be value - 1

# Wrong: This counts elements greater than OR EQUAL to value
greater_count = position - bit.query(value - 1)  # Should be position - bit.query(value)
```

**Correct Implementation:**
```python
# Count elements STRICTLY less than value
smaller_count = bit.query(value - 1)

# Count elements STRICTLY greater than value
greater_count = position - bit.query(value)
```

### 2. **Using 0-Indexed Values with 1-Indexed BIT**

Binary Indexed Trees typically use 1-based indexing, but the problem values might include 0 or use 0-based indexing.

**Problem Scenario:**
```python
instructions = [0, 1, 2, 3]  # Contains 0
bit = BinaryIndexedTree(max(instructions))  # Size = 3
bit.update(0, 1)  # Error: BIT doesn't handle index 0
```

**Solution:**
Shift all values by 1 when working with the BIT:
```python
class Solution:
    def createSortedArray(self, instructions: List[int]) -> int:
        # Shift values to handle 0s
        max_value = max(instructions)
        bit = BinaryIndexedTree(max_value + 1)  # Extra space for shifting

        total_cost = 0
        MOD = 10**9 + 7

        for position, value in enumerate(instructions):
            # Shift value by 1 for BIT operations
            shifted_value = value + 1

            smaller_count = bit.query(shifted_value - 1)
            greater_count = position - bit.query(shifted_value)

            total_cost += min(smaller_count, greater_count)
            bit.update(shifted_value, 1)

        return total_cost % MOD
```

### 3. **Integer Overflow Before Taking Modulo**

Forgetting to apply modulo during accumulation can cause overflow in languages with fixed integer sizes.

**Potential Issue:**
```python
total_cost = 0
for i in range(len(instructions)):
    cost = min(smaller_count, greater_count)
    total_cost += cost  # Can overflow for large inputs

return total_cost % MOD  # Modulo applied too late
```

**Better Practice:**
```python
total_cost = 0
for i in range(len(instructions)):
    cost = min(smaller_count, greater_count)
    total_cost = (total_cost + cost) % MOD  # Apply modulo during accumulation

return total_cost
```

### 4. **Incorrect BIT Size Allocation**

Not allocating enough space for the maximum value can cause out-of-bounds errors.

**Wrong:**
```python
# If instructions = [1, 5, 6, 2]
bit = BinaryIndexedTree(len(instructions))  # Size = 4, but max value is 6!
bit.update(6, 1)  # Index out of bounds
```

**Correct:**
```python
max_value = max(instructions)
bit = BinaryIndexedTree(max_value)  # Size based on max value, not array length
```

### 5. **Confusion Between Element Count and Element Value**

Mixing up the position index (number of elements inserted) with the actual values being inserted.

**Common Mistake:**
```python
for i, value in enumerate(instructions):
    # Wrong: Using value instead of i for total element count
    greater_count = value - bit.query(value)  # Should use i, not value
```

**Correct:**
```python
for i, value in enumerate(instructions):
    # i represents how many elements have been inserted so far
    greater_count = i - bit.query(value)
```

