# 3287. Find the Maximum Sequence Value of Array

> Source: https://algo.monster/liteproblems/3287

## Problem Statement

You are given an integer array `nums` and a positive integer `k`.

The problem asks you to find a subsequence of `nums` with exactly `2 * k` elements, then split this subsequence into two equal halves. The value of this subsequence is calculated by:
1. Taking the OR of all elements in the first half (first `k` elements)
2. Taking the OR of all elements in the second half (last `k` elements)  
3. Computing the XOR of these two OR values

More formally, for a sequence `seq` of size `2 * x`, its value is:
`(seq[0] OR seq[1] OR ... OR seq[x - 1]) XOR (seq[x] OR seq[x + 1] OR ... OR seq[2 * x - 1])`

Your task is to find the maximum possible value among all subsequences of `nums` that have exactly `2 * k` elements.

For example, if you have a subsequence `[a, b, c, d]` where `k = 2`, the value would be `(a OR b) XOR (c OR d)`.

Note that a subsequence maintains the relative order of elements from the original array but doesn't need to be contiguous.

## Intuition

The key insight is that we need to select `2k` elements from the array and split them into two groups of `k` elements each. The challenge is that there are exponentially many ways to choose these elements, making brute force infeasible.

However, notice that the value computation only depends on the OR values of each half. Since we're dealing with integers that fit in a small range (the solution uses `m = 1 << 7 = 128`, suggesting values are at most 7 bits), there are only 128 possible OR values for any subset.

This leads us to think about [dynamic programming](/problems/dynamic_programming_intro) where we track which OR values are achievable rather than tracking specific subsets. The question becomes: can we achieve OR value `x` using `k` elements from some portion of the array?

The crucial observation is that we can split the problem at any position `i` in the array. Elements before position `i` can contribute to the first group (with OR value `x`), and elements from position `i` onwards can contribute to the second group (with OR value `y`). The final answer would be `x XOR y`.

To implement this, we use:
- **Forward DP (`f`)**: `f[i][j][x]` tells us if we can select `j` elements from the first `i` elements to achieve OR value `x`
- **Backward DP (`g`)**: `g[i][j][y]` tells us if we can select `j` elements starting from position `i` to achieve OR value `y`

By computing both forward and backward states, we can enumerate all possible split points `i` and check if we can form `k` elements with OR value `x` before position `i`, and `k` elements with OR value `y` from position `i` onwards. The maximum `x XOR y` among all valid combinations gives us our answer.

This approach is efficient because instead of tracking specific element combinations (which would be exponential), we only track achievable OR values (at most 128 possibilities), making the solution polynomial in time.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with prefix and suffix decomposition to efficiently find all possible OR values.

**Forward DP Construction:**

We build a 3D DP table `f[i][j][x]` where:
- `i` represents considering the first `i` elements (0 to n)
- `j` represents the number of elements selected (0 to k)
- `x` represents the OR value achieved (0 to 127)
- `f[i][j][x] = True` if we can select `j` elements from the first `i` elements with OR value `x`

The base case is `f[0][0][0] = True` (selecting 0 elements gives OR value 0).

For each element at position `i`, we have two choices:
1. **Skip it**: `f[i + 1][j][x] |= f[i][j][x]` - the states remain the same
2. **Take it**: `f[i + 1][j + 1][x | nums[i]] |= f[i][j][x]` - we increment the count and update the OR value

**Backward DP Construction:**

Similarly, we build `g[i][j][y]` where:
- `i` represents considering elements starting from position `i` 
- `j` represents the number of elements selected
- `y` represents the OR value achieved
- `g[i][j][y] = True` if we can select `j` elements starting from position `i` with OR value `y`

The base case is `g[n][0][0] = True`.

We iterate backwards from position `n` to 0:
1. **Skip element at i-1**: `g[i - 1][j][y] |= g[i][j][y]`
2. **Take element at i-1**: `g[i - 1][j + 1][y | nums[i - 1]] |= g[i][j][y]`

**Finding the Maximum Value:**

After building both DP tables, we enumerate all possible split points `i` from `k` to `n - k` (ensuring we have at least `k` elements on each side).

For each split point `i`:
- Check all possible OR values `x` where `f[i][k][x] = True` (we can get OR value `x` using `k` elements from the first `i` elements)
- Check all possible OR values `y` where `g[i][k][y] = True` (we can get OR value `y` using `k` elements from position `i` onwards)
- If both are true, update the answer: `ans = max(ans, x ^ y)`

The time complexity is `O(n * k * m^2)` where `n` is the array length, `k` is the parameter, and `m = 128` is the range of OR values. The space complexity is `O(n * k * m)` for the DP tables.

## Example Walkthrough

Let's walk through a small example with `nums = [3, 5, 1, 6]` and `k = 1`.

We need to select a subsequence of exactly `2 * k = 2` elements, split them into two halves (1 element each), and maximize `(first_element) XOR (second_element)`.

**Step 1: Build Forward DP Table `f[i][j][x]`**

We track whether we can select `j` elements from the first `i` elements to achieve OR value `x`.

- Initial: `f[0][0][0] = True` (0 elements selected, OR value = 0)

- Process element 3 (index 0):
  - Skip: `f[1][0][0] = True` (from `f[0][0][0]`)
  - Take: `f[1][1][3] = True` (from `f[0][0][0]`, OR value becomes 0|3 = 3)

- Process element 5 (index 1):
  - Skip: `f[2][0][0] = True`, `f[2][1][3] = True`
  - Take: `f[2][1][5] = True` (from `f[1][0][0]`, OR value becomes 0|5 = 5)

- Process element 1 (index 2):
  - Skip: `f[3][0][0] = True`, `f[3][1][3] = True`, `f[3][1][5] = True`
  - Take: `f[3][1][1] = True` (from `f[2][0][0]`, OR value becomes 0|1 = 1)

- Process element 6 (index 3):
  - Skip: `f[4][0][0] = True`, `f[4][1][3] = True`, `f[4][1][5] = True`, `f[4][1][1] = True`
  - Take: `f[4][1][6] = True` (from `f[3][0][0]`, OR value becomes 0|6 = 6)

**Step 2: Build Backward DP Table `g[i][j][y]`**

We track whether we can select `j` elements starting from position `i` to achieve OR value `y`.

- Initial: `g[4][0][0] = True` (0 elements selected from position 4 onwards)

- Process from position 4 to 3 (element 6):
  - Skip: `g[3][0][0] = True`
  - Take: `g[3][1][6] = True` (from `g[4][0][0]`, OR value becomes 0|6 = 6)

- Process from position 3 to 2 (element 1):
  - Skip: `g[2][0][0] = True`, `g[2][1][6] = True`
  - Take: `g[2][1][1] = True` (from `g[3][0][0]`, OR value becomes 0|1 = 1)

- Process from position 2 to 1 (element 5):
  - Skip: `g[1][0][0] = True`, `g[1][1][6] = True`, `g[1][1][1] = True`
  - Take: `g[1][1][5] = True` (from `g[2][0][0]`, OR value becomes 0|5 = 5)

- Process from position 1 to 0 (element 3):
  - Skip: `g[0][0][0] = True`, `g[0][1][6] = True`, `g[0][1][1] = True`, `g[0][1][5] = True`
  - Take: `g[0][1][3] = True` (from `g[1][0][0]`, OR value becomes 0|3 = 3)

**Step 3: Find Maximum XOR Value**

We check split points from `i = k = 1` to `i = n - k = 3`:

- **Split at i = 1:**
  - `f[1][1][3] = True` (can select element 3 from first part)
  - `g[1][1][5] = True` (can select element 5 from second part)
  - Value: 3 XOR 5 = 6

- **Split at i = 2:**
  - `f[2][1][3] = True` (can select element 3 from first part)
  - `g[2][1][6] = True` (can select element 6 from second part)
  - Value: 3 XOR 6 = 5
  - `f[2][1][5] = True` (can select element 5 from first part)
  - `g[2][1][6] = True` (can select element 6 from second part)
  - Value: 5 XOR 6 = 3
  - `f[2][1][5] = True` (can select element 5 from first part)
  - `g[2][1][1] = True` (can select element 1 from second part)
  - Value: 5 XOR 1 = 4

- **Split at i = 3:**
  - `f[3][1][3] = True` (can select element 3 from first part)
  - `g[3][1][6] = True` (can select element 6 from second part)
  - Value: 3 XOR 6 = 5
  - `f[3][1][5] = True` (can select element 5 from first part)
  - `g[3][1][6] = True` (can select element 6 from second part)
  - Value: 5 XOR 6 = 3
  - `f[3][1][1] = True` (can select element 1 from first part)
  - `g[3][1][6] = True` (can select element 6 from second part)
  - Value: 1 XOR 6 = 7

**Maximum value = 7** (achieved by selecting subsequence [1, 6])

## Python Solution

```python
class Solution:
    def maxValue(self, nums: List[int], k: int) -> int:
        MAX_VALUE = 1 << 7  # Maximum possible value (128) for bitwise operations
        n = len(nums)
      
        # dp_left[i][j][mask] = True if we can select j elements from first i elements
        # such that their OR equals mask
        dp_left = [[[False] * MAX_VALUE for _ in range(k + 2)] for _ in range(n + 1)]
        dp_left[0][0][0] = True  # Base case: 0 elements selected, OR value is 0
      
        # Build DP table for left side selections
        for i in range(n):
            for j in range(k + 1):
                for mask in range(MAX_VALUE):
                    if dp_left[i][j][mask]:
                        # Option 1: Don't include nums[i]
                        dp_left[i + 1][j][mask] = True
                        # Option 2: Include nums[i] in the selection
                        dp_left[i + 1][j + 1][mask | nums[i]] = True
      
        # dp_right[i][j][mask] = True if we can select j elements from nums[i:n]
        # such that their OR equals mask
        dp_right = [[[False] * MAX_VALUE for _ in range(k + 2)] for _ in range(n + 1)]
        dp_right[n][0][0] = True  # Base case: 0 elements selected from empty suffix
      
        # Build DP table for right side selections (traverse backwards)
        for i in range(n, 0, -1):
            for j in range(k + 1):
                for mask in range(MAX_VALUE):
                    if dp_right[i][j][mask]:
                        # Option 1: Don't include nums[i-1]
                        dp_right[i - 1][j][mask] = True
                        # Option 2: Include nums[i-1] in the selection
                        dp_right[i - 1][j + 1][mask | nums[i - 1]] = True
      
        max_xor = 0
      
        # Try all possible split points where we take k elements from left
        # and k elements from right
        for split_point in range(k, n - k + 1):
            # Check all possible OR values from left side
            for left_or in range(MAX_VALUE):
                if dp_left[split_point][k][left_or]:
                    # Check all possible OR values from right side
                    for right_or in range(MAX_VALUE):
                        if dp_right[split_point][k][right_or]:
                            # Calculate XOR and update maximum
                            max_xor = max(max_xor, left_or ^ right_or)
      
        return max_xor
```

## Time and Space Complexity

**Time Complexity:** `O(n × m × k)`

The algorithm consists of three main parts:

1. **Forward DP computation**: The first nested loop iterates through `n` elements, `k+1` possible subset sizes, and `m` possible OR values. For each state `f[i][j][x]`, we perform two operations (skip element or include element). This gives us `O(n × k × m)` operations.

2. **Backward DP computation**: Similarly, the second nested loop iterates from position `n` to `0`, with `k+1` subset sizes and `m` possible OR values. This also results in `O(n × k × m)` operations.

3. **Final answer computation**: The last part iterates through at most `n-2k+1` positions (from `k` to `n-k`), and for each position checks all `m` possible values for the first subset and all `m` possible values for the second subset. This gives us `O(n × m²)` operations.

Since `m = 2^7 = 128` is a constant and `k ≤ n`, the dominant term is `O(n × m × k)` when considering `m` as a parameter. The total time complexity is `O(n × m × k) + O(n × m × k) + O(n × m²) = O(n × m × (k + m))`. Given that `m = 128` is constant and typically `k` is the varying parameter, this simplifies to `O(n × m × k)`.

**Space Complexity:** `O(n × m × k)`

The algorithm uses two 3D boolean arrays:
- Array `f` with dimensions `[n+1][k+2][m]`
- Array `g` with dimensions `[n+1][k+2][m]`

Both arrays require `O((n+1) × (k+2) × m) = O(n × k × m)` space. The total space complexity is therefore `O(n × m × k)`, where `n` is the length of the array, `k` is the parameter for subset size, and `m = 2^7 = 128`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Split Point Iteration Range**

One of the most common mistakes is incorrectly defining the range for split points when combining left and right subsequences.

**Pitfall Code:**
```python
# Wrong: This assumes we need exactly k elements before the split point
for split_point in range(k, n - k + 1):
    for left_or in range(MAX_VALUE):
        if dp_left[split_point][k][left_or]:  # Requires exactly k from first split_point elements
            for right_or in range(MAX_VALUE):
                if dp_right[split_point][k][right_or]:  # Requires exactly k from position split_point onwards
```

**Issue:** The split point doesn't represent where we must have selected exactly k elements. Instead, it represents a position in the array where we can select k elements from before it and k elements from after (or including) it. The current logic restricts valid subsequences unnecessarily.

**Solution:**
```python
# Correct: Try all positions as potential split points
for split_point in range(n + 1):
    for left_or in range(MAX_VALUE):
        if dp_left[split_point][k][left_or]:  # Can select k from first split_point elements
            for right_or in range(MAX_VALUE):
                if dp_right[split_point][k][right_or]:  # Can select k from position split_point onwards
                    max_xor = max(max_xor, left_or ^ right_or)
```

### 2. **Memory Optimization Oversight**

**Pitfall:** Using full 3D arrays when only adjacent layers are needed, leading to unnecessary memory usage.

**Issue:** The DP only depends on the previous layer (i-1 to i), so maintaining all n+1 layers wastes memory.

**Solution:**
```python
# Use rolling arrays to reduce space complexity from O(n*k*m) to O(k*m)
dp_curr = [[False] * MAX_VALUE for _ in range(k + 2)]
dp_prev = [[False] * MAX_VALUE for _ in range(k + 2)]
dp_prev[0][0] = True

for i in range(n):
    # Reset current layer
    dp_curr = [[False] * MAX_VALUE for _ in range(k + 2)]
  
    for j in range(k + 1):
        for mask in range(MAX_VALUE):
            if dp_prev[j][mask]:
                dp_curr[j][mask] = True  # Don't take element i
                dp_curr[j + 1][mask | nums[i]] = True  # Take element i
  
    # Store results needed for final answer before swapping
    if i >= k - 1:  # Can potentially have k elements selected
        # Store dp_curr[k] values for later use
        pass
  
    dp_prev, dp_curr = dp_curr, dp_prev
```

### 3. **Off-by-One Errors in DP Table Dimensions**

**Pitfall Code:**
```python
# Wrong: Using k+1 instead of k+2 for the j dimension
dp_left = [[[False] * MAX_VALUE for _ in range(k + 1)] for _ in range(n + 1)]
```

**Issue:** When we select j elements and then include one more, we access index j+1. If j can be k, then j+1 = k+1, requiring dimension k+2.

**Solution:** Always allocate k+2 for the selection count dimension to avoid index out of bounds errors when j+1 is accessed with j=k.

