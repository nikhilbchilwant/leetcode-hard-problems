# 1714. Sum Of Special Evenly-Spaced Elements In Array

> Source: https://algo.monster/liteproblems/1714

## Problem Statement

You have an array `nums` of non-negative integers (0-indexed) and a list of queries. Each query is represented as `[xi, yi]`.

For each query `[xi, yi]`, you need to:
1. Start from index `xi` in the array
2. Sum up elements at positions `xi`, `xi + yi`, `xi + 2*yi`, `xi + 3*yi`, and so on
3. In other words, sum all `nums[j]` where `j >= xi` and `(j - xi)` is divisible by `yi`

The answer to each query should be returned modulo `10^9 + 7`.

For example, if `nums = [1, 2, 3, 4, 5]` and query is `[1, 2]`:
- Start at index 1 (value 2)
- Next valid index: 1 + 2 = 3 (value 4)
- Next valid index: 3 + 2 = 5 (out of bounds)
- Sum = 2 + 4 = 6

The solution uses block decomposition to optimize performance:
- For queries with small step sizes (≤ √n), precompute suffix sums for each starting position and step size
- For queries with large step sizes (> √n), calculate the sum directly since there won't be many elements to sum

The preprocessing creates a 2D array `suf[i][j]` that stores the sum of elements starting from position `j` with step size `i`. This allows O(1) lookup for small step queries while keeping the overall complexity manageable.

## Intuition

The naive approach would be to iterate through the array for each query, jumping by step size `y` starting from index `x`. This works but can be inefficient when we have many queries.

The key insight is recognizing that the efficiency depends on the step size `y`:

1. **Small step sizes** (like 1, 2, 3): If `y` is small, we'll visit many elements. For example, with step size 1, we visit every element from `x` to the end. With step size 2, we visit half the remaining elements. These patterns repeat across different queries with the same step size.

2. **Large step sizes** (like n/2, n-1): If `y` is large, we only visit a few elements. For example, with step size `n/2` in an array of size `n`, we visit at most 2 elements.

This observation leads to a trade-off strategy:
- For small step sizes that appear frequently and visit many elements, we should **precompute** the results to avoid redundant calculations
- For large step sizes that visit few elements, direct calculation is already efficient

The threshold for "small" vs "large" is chosen as `√n` because:
- There are at most `√n` different small step sizes (1 to `√n`)
- For each small step size, we need `O(n)` space to store precomputed sums
- Total preprocessing space: `O(√n * n)` which is manageable
- Each large step query visits at most `n/√n = √n` elements, making direct calculation `O(√n)`

This balances preprocessing time/space with query efficiency, achieving `O(√n)` per query regardless of step size.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements block decomposition with a threshold of `√n` to distinguish between small and large step sizes.

**Preprocessing Phase:**

1. Calculate the threshold `m = int(sqrt(n))` to separate small and large step sizes.

2. Create a 2D array `suf` with dimensions `(m+1) × (n+1)`:
   - `suf[i][j]` stores the sum of elements starting from position `j` with step size `i`
   - We only precompute for step sizes from 1 to `m`

3. Build suffix sums in reverse order:
   ```python
   for i in range(1, m + 1):  # For each small step size
       for j in range(n - 1, -1, -1):  # Process positions from right to left
           suf[i][j] = suf[i][min(n, j + i)] + nums[j]
   ```
   - Processing from right to left allows us to build suffix sums incrementally
   - `suf[i][j] = nums[j] + suf[i][j+i]` means: the sum starting at `j` equals the current element plus the sum starting at the next position `j+i`
   - `min(n, j + i)` handles boundary cases when `j + i` exceeds array length

**Query Processing Phase:**

For each query `[x, y]`:

1. **If `y <= m`** (small step size):
   - Directly return the precomputed value: `suf[y][x] % mod`
   - This is O(1) lookup time

2. **If `y > m`** (large step size):
   - Calculate the sum directly using Python's slice notation: `sum(nums[x::y])`
   - `nums[x::y]` creates a slice starting at index `x` with step `y`
   - This visits at most `n/y < n/√n = √n` elements

**Time Complexity:**
- Preprocessing: `O(m × n) = O(√n × n)`
- Per query: `O(1)` for small steps, `O(√n)` for large steps
- Total: `O(n√n + q√n)` where `q` is the number of queries

**Space Complexity:** `O(n√n)` for storing the suffix sum array

## Example Walkthrough

Let's walk through the solution with a concrete example to illustrate how the block decomposition approach works.

**Given:**
- `nums = [1, 2, 3, 4, 5, 6]` (n = 6)
- Queries: `[[0, 2], [1, 2], [2, 3], [0, 5]]`

**Step 1: Calculate threshold**
- `m = int(sqrt(6)) = 2`
- Step sizes ≤ 2 are "small", step sizes > 2 are "large"

**Step 2: Build suffix sum array**

We create `suf[3][7]` (dimensions: (m+1) × (n+1) = 3 × 7)

For step size 1 (i=1):
- Start from right to left (j = 5 to 0)
- `suf[1][5] = nums[5] + suf[1][6] = 6 + 0 = 6`
- `suf[1][4] = nums[4] + suf[1][5] = 5 + 6 = 11`
- `suf[1][3] = nums[3] + suf[1][4] = 4 + 11 = 15`
- `suf[1][2] = nums[2] + suf[1][3] = 3 + 15 = 18`
- `suf[1][1] = nums[1] + suf[1][2] = 2 + 18 = 20`
- `suf[1][0] = nums[0] + suf[1][1] = 1 + 20 = 21`

For step size 2 (i=2):
- `suf[2][5] = nums[5] + suf[2][7] = 6 + 0 = 6`
- `suf[2][4] = nums[4] + suf[2][6] = 5 + 0 = 5`
- `suf[2][3] = nums[3] + suf[2][5] = 4 + 6 = 10`
- `suf[2][2] = nums[2] + suf[2][4] = 3 + 5 = 8`
- `suf[2][1] = nums[1] + suf[2][3] = 2 + 10 = 12`
- `suf[2][0] = nums[0] + suf[2][2] = 1 + 8 = 9`

Final `suf` array:
```
suf[1] = [21, 20, 18, 15, 11, 6, 0]  // step size 1
suf[2] = [9, 12, 8, 10, 5, 6, 0]     // step size 2
```

**Step 3: Process queries**

Query 1: `[0, 2]` (start at index 0, step size 2)
- Step size 2 ≤ m, so use precomputed value
- Answer: `suf[2][0] = 9`
- Verification: nums[0] + nums[2] + nums[4] = 1 + 3 + 5 = 9 ✓

Query 2: `[1, 2]` (start at index 1, step size 2)
- Step size 2 ≤ m, so use precomputed value
- Answer: `suf[2][1] = 12`
- Verification: nums[1] + nums[3] + nums[5] = 2 + 4 + 6 = 12 ✓

Query 3: `[2, 3]` (start at index 2, step size 3)
- Step size 3 > m, so calculate directly
- Visit indices: 2, 5
- Answer: nums[2] + nums[5] = 3 + 6 = 9

Query 4: `[0, 5]` (start at index 0, step size 5)
- Step size 5 > m, so calculate directly
- Visit indices: 0, 5
- Answer: nums[0] + nums[5] = 1 + 6 = 7

**Summary:**
- Small step queries (≤2): O(1) lookup from precomputed `suf` array
- Large step queries (>2): O(√n) direct calculation, visiting at most 2 elements
- All queries answered efficiently without redundant calculations

## Python Solution

```python
from typing import List
from math import sqrt

class Solution:
    def solve(self, nums: List[int], queries: List[List[int]]) -> List[int]:
        """
        Process queries on an array where each query asks for sum of elements
        at arithmetic progression positions starting from index x with step y.

        Args:
            nums: Input array of integers
            queries: List of [x, y] pairs where x is start index and y is step size

        Returns:
            List of sums for each query, modulo 10^9 + 7
        """
        MOD = 10**9 + 7
        n = len(nums)

        # Threshold for optimization - use precomputation for small steps
        threshold = int(sqrt(n))

        # Precompute suffix sums for small step sizes (1 to threshold)
        # suffix_sums[step][i] stores sum of nums[i], nums[i+step], nums[i+2*step], ...
        suffix_sums = [[0] * (n + 1) for _ in range(threshold + 1)]

        for step in range(1, threshold + 1):
            # Build suffix sums from right to left for this step size
            for start_idx in range(n - 1, -1, -1):
                next_idx = min(n, start_idx + step)
                suffix_sums[step][start_idx] = suffix_sums[step][next_idx] + nums[start_idx]

        result = []

        # Process each query
        for start_idx, step_size in queries:
            if step_size <= threshold:
                # Use precomputed suffix sum for small steps
                result.append(suffix_sums[step_size][start_idx] % MOD)
            else:
                # Compute sum directly for large steps (fewer elements to sum)
                query_sum = sum(nums[start_idx::step_size])
                result.append(query_sum % MOD)

        return result
```

## Time and Space Complexity

**Time Complexity:** `O((n + m) × √n)`

The time complexity breaks down into two main parts:

1. **Preprocessing phase:** Building the suffix sum array `suf`
   - The outer loop runs for `m + 1` iterations where `m = √n`, so approximately `√n` iterations
   - The inner loop runs for `n` iterations
   - Total preprocessing time: `O(√n × n) = O(n√n)`

2. **Query processing phase:** Processing `m` queries (where `m` is the number of queries)
   - For each query with `y ≤ √n`: Direct lookup in the precomputed array takes `O(1)` time
   - For each query with `y > √n`: Computing `sum(nums[x::y])` takes `O(n/y)` time. Since `y > √n`, this is at most `O(n/√n) = O(√n)` time
   - Total query processing time: `O(m × √n)`

Combined time complexity: `O(n√n + m√n) = O((n + m) × √n)`

**Space Complexity:** `O(n × √n)`

The space complexity is determined by:
- The `suf` array has dimensions `(m + 1) × (n + 1)` where `m = √n`
- This gives us approximately `√n × n = n√n` space
- Other variables use `O(1)` or `O(n)` space which doesn't affect the overall complexity

Therefore, the space complexity is `O(n√n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow in Suffix Sum Accumulation**

**Pitfall:** When building the suffix sums, the accumulated values can become very large before applying the modulo operation. In languages with fixed integer sizes (like C++ or Java), this can cause integer overflow. Even in Python, storing unnecessarily large intermediate values wastes memory and slows computation.

**Example:**
```python
# Problematic: Sums can grow extremely large before modulo
suffix_sums[step][start_idx] = suffix_sums[step][next_idx] + nums[start_idx]
```

**Solution:** Apply modulo operation during the accumulation phase:
```python
suffix_sums[step][start_idx] = (suffix_sums[step][next_idx] + nums[start_idx]) % MOD
```

### 2. **Incorrect Threshold Calculation for Edge Cases**

**Pitfall:** Using `int(sqrt(n))` as the threshold can be suboptimal for small arrays. When `n = 1` or `n = 2`, the threshold becomes 1, which means we only precompute for step size 1, missing potential optimizations for step size 2.

**Example:**
```python
# For n = 2, threshold = 1, but step size 2 could also be precomputed efficiently
threshold = int(sqrt(n))
```

**Solution:** Use a minimum threshold value or adjust based on memory constraints:
```python
threshold = max(int(sqrt(n)), min(n, 100))  # Ensure reasonable minimum
# OR
threshold = int(sqrt(n)) if n > 4 else n
```

### 3. **Memory Allocation for Very Large Arrays**

**Pitfall:** The 2D suffix sum array uses O(n√n) memory. For very large arrays (e.g., n = 10^6), this creates a matrix of size √10^6 × 10^6 ≈ 1000 × 10^6 = 10^9 elements, which can cause memory issues.

**Example:**
```python
# This can fail for large n due to memory constraints
suffix_sums = [[0] * (n + 1) for _ in range(threshold + 1)]
```

**Solution:** Implement lazy initialization or use a dictionary for sparse storage:
```python
# Option 1: Use dictionary for actually needed combinations
suffix_sums = {}
for step in range(1, threshold + 1):
    for start_idx in range(n - 1, -1, -1):
        next_idx = min(n, start_idx + step)
        key = (step, start_idx)
        suffix_sums[key] = (suffix_sums.get((step, next_idx), 0) + nums[start_idx]) % MOD

# Option 2: Cap the threshold based on available memory
MAX_MEMORY_CELLS = 10**7  # Adjust based on constraints
threshold = min(int(sqrt(n)), MAX_MEMORY_CELLS // n)
```

### 4. **Off-by-One Error in Boundary Handling**

**Pitfall:** The boundary condition `min(n, start_idx + step)` assumes that index `n` in the suffix array represents "out of bounds" with value 0. If this initialization is missed or misunderstood, it can lead to incorrect sums.

**Example:**
```python
# If suffix_sums[step][n] is not initialized to 0, this fails
next_idx = min(n, start_idx + step)
suffix_sums[step][start_idx] = suffix_sums[step][next_idx] + nums[start_idx]
```

**Solution:** Explicitly ensure the boundary initialization:
```python
# Initialize boundary values explicitly
suffix_sums = [[0] * (n + 1) for _ in range(threshold + 1)]
# The (n+1)th column is already 0, which is correct for out-of-bounds

# Alternative: Handle boundary explicitly in code
for step in range(1, threshold + 1):
    for start_idx in range(n - 1, -1, -1):
        if start_idx + step >= n:
            suffix_sums[step][start_idx] = nums[start_idx]
        else:
            suffix_sums[step][start_idx] = (suffix_sums[step][start_idx + step] + nums[start_idx]) % MOD
```

