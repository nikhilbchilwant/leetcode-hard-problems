# 668. Kth Smallest Number in Multiplication Table

> Source: https://algo.monster/liteproblems/668

## Problem Statement

You're given a multiplication table of size `m x n`, where each cell at position `[i][j]` contains the product `i * j` (using 1-based indexing). For example, a 3x3 multiplication table would look like:

```
1  2  3
2  4  6
3  6  9
```

The task is to find the `k`th smallest element when all values in this multiplication table are considered in sorted order.

**Input:**
- `m`: number of rows in the multiplication table
- `n`: number of columns in the multiplication table
- `k`: the position of the element to find (1-indexed)

**Output:**
- The `k`th smallest value in the multiplication table

**Example:**
For a 3x3 table with `k = 5`, the values in sorted order would be: `[1, 2, 2, 3, 3, 4, 6, 6, 9]`. The 5th smallest element is `3`.

## Intuition

The key insight is that we don't need to generate the entire multiplication table to find the `k`th smallest element. Instead, we can use [binary search](/problems/binary_search_intro) on the possible values.

Think about it this way: the smallest possible value in the table is `1` (at position `[1][1]`), and the largest is `m * n` (at position `[m][n]`). The answer must be somewhere in this range.

For any given value `x`, we can efficiently count how many elements in the multiplication table are less than or equal to `x` without generating the table. How? For each row `i`, the elements are `i, 2i, 3i, ..., n*i`. The number of elements ≤ `x` in row `i` is simply `min(x // i, n)` - we divide `x` by `i` to see how many multiples of `i` are ≤ `x`, but cap it at `n` since each row has only `n` elements.

This counting ability enables a [binary search](/problems/binary_search_intro) strategy:
- If we pick a value `mid` and find that there are at least `k` elements ≤ `mid`, then our answer is at most `mid`
- If there are fewer than `k` elements ≤ `mid`, then our answer must be greater than `mid`

The beauty of this approach is that [binary search](/problems/binary_search_intro) will naturally converge to a value that actually exists in the table. Even though we're searching through all integers in the range `[1, m*n]`, the final answer will always be a valid product that appears in the multiplication table because we're looking for the smallest value where the count reaches `k`.

**Learn more about [Math](/problems/math-basics) and [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

We use the standard [binary search](/problems/binary_search_intro) template with `first_true_index` to find the kth smallest element.

**Binary Search Setup:**
- `left = 1`: minimum possible value in the table
- `right = m * n`: maximum possible value in the table
- `first_true_index = -1`: tracks the first value where count >= k

**Feasible Condition:**
For any value `mid`, we count how many elements in the multiplication table are ≤ `mid`. If this count is >= k, then `mid` could be our answer (or the answer is smaller). This creates the monotonic pattern: `false, false, ..., true, true, true`.

**Counting Elements ≤ mid:**
For each row `i` (from 1 to m), the elements are `i, 2i, 3i, ..., n*i`. The count of elements ≤ `mid` in row `i` is `min(mid // i, n)`:
- `mid // i` gives how many multiples of `i` are ≤ `mid`
- We cap at `n` because each row has at most `n` elements

**Binary Search Template:**
```python
while left <= right:
    mid = (left + right) // 2
    count = sum(min(mid // row, n) for row in range(1, m + 1))

    if count >= k:  # Feasible: kth element is at most mid
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

return first_true_index
```

**Why the Result Exists in the Table:**
The [binary search](/problems/binary_search_intro) finds the smallest value where count >= k. This value is guaranteed to exist in the multiplication table because:
- If `mid` is in the table and count(mid) >= k while count(mid-1) < k, then `mid` is exactly the kth smallest
- The template converges to this exact boundary

The time complexity is `O(m * log(m * n))` - `O(log(m * n))` iterations with `O(m)` counting each. Space complexity is `O(1)`.

## Example Walkthrough

Let's trace through the binary search template with `m = 3`, `n = 3`, `k = 5`.

**Setup:**
- Multiplication table:
```
1  2  3
2  4  6
3  6  9
```
- Sorted values: [1, 2, 2, 3, 3, 4, 6, 6, 9]
- Looking for 5th element (should be 3)

**Initial state:**
- `left = 1`, `right = 9`, `first_true_index = -1`
- Feasible condition: count of elements ≤ mid is >= 5

**Iteration 1:**
- `mid = (1 + 9) // 2 = 5`
- Count elements ≤ 5:
  - Row 1: min(5/1, 3) = 3 elements (1, 2, 3)
  - Row 2: min(5/2, 3) = 2 elements (2, 4)
  - Row 3: min(5/3, 3) = 1 element (3)
  - Total = 6
- Is 6 >= 5? Yes → Feasible
- Update: `first_true_index = 5`, `right = 4`

**Iteration 2:**
- `left = 1`, `right = 4`
- `mid = (1 + 4) // 2 = 2`
- Count elements ≤ 2:
  - Row 1: min(2/1, 3) = 2 elements (1, 2)
  - Row 2: min(2/2, 3) = 1 element (2)
  - Row 3: min(2/3, 3) = 0 elements
  - Total = 3
- Is 3 >= 5? No → Not feasible
- Update: `left = 3`

**Iteration 3:**
- `left = 3`, `right = 4`
- `mid = (3 + 4) // 2 = 3`
- Count elements ≤ 3:
  - Row 1: min(3/1, 3) = 3 elements
  - Row 2: min(3/2, 3) = 1 element
  - Row 3: min(3/3, 3) = 1 element
  - Total = 5
- Is 5 >= 5? Yes → Feasible
- Update: `first_true_index = 3`, `right = 2`

**Iteration 4:**
- `left = 3`, `right = 2`
- Loop ends: `left > right`

**Result:**
- `first_true_index = 3`
- The 5th smallest element is 3

The template correctly finds the boundary where count first reaches k.

## Python Solution

```python
class Solution:
    def findKthNumber(self, m: int, n: int, k: int) -> int:
        """
        Find the kth smallest number in a multiplication table of size m x n.
        Uses binary search template with first_true_index.
        Feasible condition: count of elements <= mid is >= k
        """
        left, right = 1, m * n
        first_true_index = -1

        while left <= right:
            mid = (left + right) // 2

            # Count how many numbers in the multiplication table are <= mid
            count = 0
            for row in range(1, m + 1):
                # For each row i, elements are: i, 2i, 3i, ..., n*i
                # Number of elements <= mid in row i is min(mid // i, n)
                count += min(mid // row, n)

            # Feasible: count >= k means the kth number is at most mid
            if count >= k:
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1

        return first_true_index
```

## Time and Space Complexity

**Time Complexity:** `O(m * log(m * n))`

The binary search runs from `1` to `m * n`, which means the search space has size `m * n`. Binary search takes `O(log(m * n))` iterations to converge.

For each iteration of the binary search, we need to count how many elements in the multiplication table are less than or equal to `mid`. This counting process involves a loop that runs `m` times (from `1` to `m`), and for each iteration `i`, we calculate `min(mid // i, n)` which takes `O(1)` time.

Therefore, the total time complexity is `O(m * log(m * n))`.

**Space Complexity:** `O(1)`

The algorithm only uses a constant amount of extra space for variables `left`, `right`, `mid`, `cnt`, and the loop variable `i`. No additional data structures that scale with input size are used.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Mixing Up Binary Search Template Variants

**The Pitfall:** Two loop styles are common: `while left < right` with `right = mid`, and `while left <= right` with `right = mid - 1`. Each is correct on its own, but mixing parts of them (for example `while left <= right` with `right = mid`) causes an infinite loop once `left == right` and the condition is true.

**Alternative (also correct) variant:**
```python
while left < right:
    mid = (left + right) // 2
    if count >= k:
        right = mid
    else:
        left = mid + 1
return left
```

**Correct (standard template):**
```python
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if count >= k:
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
return first_true_index
```

Both versions return the same answer for this problem. The `first_true_index` version records the answer explicitly, which makes it easier to see what the loop returns.

### 2. Incorrect Counting Logic

**The Pitfall:** Not considering the table boundaries when counting:

**Incorrect:**
```python
count += mid // row  # Missing the min() with n
```

**Why This Fails:**
Each row has exactly `n` elements. Without `min(mid // row, n)`, we count more elements than exist. If `mid = 100`, `row = 1`, and `n = 5`, we'd count 100 elements instead of 5.

**Correct:**
```python
count += min(mid // row, n)
```

### 3. Using 0-Based Indexing for Rows

**Incorrect:**
```python
for row in range(m):  # Wrong: starts from 0
    count += min(mid // row, n)
```

**Why This Fails:**
The multiplication table uses 1-based indexing. Starting from row 0 makes the first iteration compute `mid // 0`, which raises `ZeroDivisionError`. (Writing `mid // (row + 1)` with `range(m)` is also correct.)

**Correct:**
```python
for row in range(1, m + 1):  # Start from 1, go up to m inclusive
    count += min(mid // row, n)
```

### 4. Integer Overflow in Large Inputs

In languages with fixed integer sizes, `left + right` can overflow when the search range is large. Under this problem's constraints (`m, n <= 3 * 10^4`), `m * n <= 9 * 10^8` and `left + right <= 1.8 * 10^9`, which still fits in a 32-bit signed int (max about `2.147 * 10^9`). The safer form below costs nothing and avoids the issue if the bounds grow.

**Safe Approach:**
```python
mid = left + (right - left) // 2  # Prevents overflow
```

