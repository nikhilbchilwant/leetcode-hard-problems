# 2143. Choose Numbers From Two Arrays in Range

> Source: https://algo.monster/liteproblems/2143

## Problem Statement

You have two arrays `nums1` and `nums2`, both of length `n` and 0-indexed.

A range `[l, r]` (where `0 <= l <= r < n`) is called **balanced** if:
- For each index `i` in the range `[l, r]`, you select either `nums1[i]` or `nums2[i]`
- The sum of all values selected from `nums1` equals the sum of all values selected from `nums2`
- If no values are selected from an array, its sum is considered 0

Two balanced ranges are considered **different** if:
- They have different start points (`l1 != l2`), OR
- They have different end points (`r1 != r2`), OR
- For at least one index `i`, one range picks from `nums1[i]` while the other picks from `nums2[i]` (or vice versa)

The task is to count the total number of different balanced ranges possible. Since this number can be very large, return the result modulo `10^9 + 7`.

For example, if you have a range `[0, 2]` with `nums1 = [1, 2, 3]` and `nums2 = [3, 2, 1]`, you could:
- Pick `nums1[0] = 1`, `nums2[1] = 2`, and `nums1[2] = 3`, giving sums of 4 and 2 (not balanced)
- Pick `nums2[0] = 3`, `nums1[1] = 2`, and `nums2[2] = 1`, giving sums of 2 and 4 (not balanced)
- Pick `nums1[0] = 1`, `nums1[1] = 2`, and `nums2[2] = 1`, giving sums of 3 and 1 (not balanced)
- And so on...

The challenge is to efficiently count all possible balanced configurations across all possible ranges.

## Intuition

The key insight is to transform this problem into tracking the **difference** between sums rather than tracking two separate sums. When we pick `nums1[i]`, we can think of it as adding `nums1[i]` to our difference. When we pick `nums2[i]`, we can think of it as subtracting `nums2[i]` from our difference. A balanced range occurs when this difference equals 0.

Let's think about this step by step:
- If we pick `nums1[i]`, the difference increases by `nums1[i]`
- If we pick `nums2[i]`, the difference decreases by `nums2[i]`
- We want to count all ways to achieve a difference of 0

This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach where we track all possible difference values we can achieve ending at each position.

We use `f[i][j]` to represent the number of ways to achieve a difference of `j - s2` when considering subarrays ending at position `i`. We offset by `s2` (sum of `nums2`) to handle negative differences, since array indices must be non-negative.

For each position `i`:
- We can start a new subarray here by picking either `nums1[i]` (difference = `nums1[i]`) or `nums2[i]` (difference = `-nums2[i]`)
- We can extend previous subarrays ending at `i-1` by:
  - Picking `nums1[i]`: if the previous difference was `j - nums1[i]`, it becomes `j`
  - Picking `nums2[i]`: if the previous difference was `j + nums2[i]`, it becomes `j`

The answer accumulates by counting all ways to achieve a difference of 0 (stored at index `s2` after offset) at each ending position. This counts all balanced ranges because each different way of achieving 0 difference represents a different balanced configuration.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses [dynamic programming](/problems/dynamic_programming_intro) with a 2D array to track all possible difference values at each position.

**Data Structure Setup:**
- Create a 2D DP array `f` of size `n × (s1 + s2 + 1)` where `s1 = sum(nums1)` and `s2 = sum(nums2)`
- The second dimension size `s1 + s2 + 1` ensures we can handle all possible differences from `-s2` to `s1`
- We use `s2` as an offset to handle negative indices (difference of 0 maps to index `s2`)

**Algorithm Steps:**

1. **Initialize for each position `i`:**
   - `f[i][a + s2] += 1`: Start a new subarray by picking `nums1[i]`, creating difference `a`
   - `f[i][-b + s2] += 1`: Start a new subarray by picking `nums2[i]`, creating difference `-b`

2. **Extend from previous position (when `i > 0`):**
   - For each possible difference value `j`:
     - If we pick `nums1[i]`: Add ways from `f[i-1][j-a]` to `f[i][j]`
       - This means: previous difference was `j-a`, adding `a` makes it `j`
     - If we pick `nums2[i]`: Add ways from `f[i-1][j+b]` to `f[i][j]`
       - This means: previous difference was `j+b`, subtracting `b` makes it `j`

3. **Accumulate answer:**
   - After processing position `i`, add `f[i][s2]` to the answer
   - `f[i][s2]` represents all ways to achieve difference 0 for subarrays ending at position `i`

**Boundary Checks:**
- Before accessing `f[i-1][j-a]`, ensure `j >= a` to avoid underflow
- Before accessing `f[i-1][j+b]`, ensure `j+b < s1+s2+1` to avoid overflow

**Modulo Operation:**
- Apply modulo `10^9 + 7` after each addition to prevent integer overflow

The time complexity is `O(n × (s1 + s2))` where `n` is the array length and `s1`, `s2` are the sums of the arrays. The space complexity is also `O(n × (s1 + s2))` for the DP table.

## Example Walkthrough

Let's walk through a small example with `nums1 = [1, 2, 1]` and `nums2 = [2, 1, 2]`.

**Initial Setup:**
- `s1 = sum(nums1) = 1 + 2 + 1 = 4`
- `s2 = sum(nums2) = 2 + 1 + 2 = 5`
- Create DP array `f` of size `3 × 10` (since `s1 + s2 + 1 = 10`)
- Offset value is `5` (equal to `s2`)

**Processing Position 0 (nums1[0]=1, nums2[0]=2):**

Starting new subarrays:
- Pick `nums1[0]`: difference = +1, so `f[0][1+5] = f[0][6] = 1`
- Pick `nums2[0]`: difference = -2, so `f[0][-2+5] = f[0][3] = 1`

Check for balanced ranges: `f[0][5] = 0` (no way to get difference 0 with single element)
- Answer = 0

**Processing Position 1 (nums1[1]=2, nums2[1]=1):**

Starting new subarrays:
- Pick `nums1[1]`: `f[1][2+5] = f[1][7] = 1`
- Pick `nums2[1]`: `f[1][-1+5] = f[1][4] = 1`

Extending from position 0:
- From `f[0][6]=1` (previous difference was 1):
  - Pick `nums1[1]`: difference becomes 1+2=3, so `f[1][3+5] = f[1][8] += 1`
  - Pick `nums2[1]`: difference becomes 1-1=0, so `f[1][0+5] = f[1][5] += 1` ✓
- From `f[0][3]=1` (previous difference was -2):
  - Pick `nums1[1]`: difference becomes -2+2=0, so `f[1][0+5] = f[1][5] += 1` ✓
  - Pick `nums2[1]`: difference becomes -2-1=-3, so `f[1][-3+5] = f[1][2] += 1`

After position 1: `f[1][5] = 2` (two balanced ranges found)
- Answer = 0 + 2 = 2

**Processing Position 2 (nums1[2]=1, nums2[2]=2):**

Starting new subarrays:
- Pick `nums1[2]`: `f[2][1+5] = f[2][6] = 1`
- Pick `nums2[2]`: `f[2][-2+5] = f[2][3] = 1`

Extending from position 1:
- From `f[1][7]=1`: extend to `f[2][8]` and `f[2][5]`
- From `f[1][4]=1`: extend to `f[2][5]` and `f[2][2]`
- From `f[1][8]=1`: extend to `f[2][9]` and `f[2][6]`
- From `f[1][5]=2`: extend to `f[2][6]` and `f[2][3]`
- From `f[1][2]=1`: extend to `f[2][3]` and `f[2][0]`

After all extensions: `f[2][5] = 2` (two more balanced ranges found)
- Answer = 2 + 2 = 4

**Final Result:** 4 balanced ranges total

The balanced ranges are:
1. Range [1,1]: pick nums2[1]=1 (sum from nums1=0, sum from nums2=0, but we treat empty as 0)
2. Range [0,1]: pick nums1[0]=1 and nums2[1]=1 (sums: 1 and 1)
3. Range [0,1]: pick nums2[0]=2 and nums1[1]=2 (sums: 2 and 2)
4. Range [1,2]: pick nums1[1]=2 and nums2[2]=2 (sums: 2 and 2)

## Python Solution

```python
class Solution:
    def countSubranges(self, nums1: List[int], nums2: List[int]) -> int:
        n = len(nums1)
        total_sum1, total_sum2 = sum(nums1), sum(nums2)

        # DP table: dp[i][j] represents the number of ways to achieve
        # a difference of (j - total_sum2) between sum of elements from nums1
        # and sum of elements from nums2, ending at index i
        # We offset by total_sum2 to handle negative differences
        max_diff_range = total_sum1 + total_sum2 + 1
        dp = [[0] * max_diff_range for _ in range(n)]

        result = 0
        MOD = 10**9 + 7

        for i in range(n):
            current_val1, current_val2 = nums1[i], nums2[i]

            # Base case: start a new subrange at index i
            # Taking element from nums1 adds current_val1 to the difference
            dp[i][current_val1 + total_sum2] += 1
            # Taking element from nums2 subtracts current_val2 from the difference
            dp[i][-current_val2 + total_sum2] += 1

            # Transition: extend subarrays ending at i-1 to include element at i
            if i > 0:
                for diff in range(max_diff_range):
                    # Extend by taking element from nums1 (adds to difference)
                    if diff >= current_val1:
                        dp[i][diff] = (dp[i][diff] + dp[i - 1][diff - current_val1]) % MOD

                    # Extend by taking element from nums2 (subtracts from difference)
                    if diff + current_val2 < max_diff_range:
                        dp[i][diff] = (dp[i][diff] + dp[i - 1][diff + current_val2]) % MOD

            # Count subarrays ending at i where sum1 equals sum2 (difference = 0)
            # The offset position for difference 0 is total_sum2
            result = (result + dp[i][total_sum2]) % MOD

        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n * (s1 + s2))` where `n` is the length of the input arrays, `s1` is the sum of all elements in `nums1`, and `s2` is the sum of all elements in `nums2`.

- The outer loop iterates through all `n` elements of the arrays
- For each iteration `i`, the inner loop runs through all possible sum states from `0` to `s1 + s2`, which is `O(s1 + s2)` operations
- The operations inside the inner loop (checking conditions, updating DP values) are all `O(1)`
- Therefore, the total time complexity is `O(n * (s1 + s2))`

**Space Complexity:** `O(n * (s1 + s2))`

- The 2D DP array `f` has dimensions `n × (s1 + s2 + 1)`, which requires `O(n * (s1 + s2))` space
- Other variables like `ans`, `mod`, `i`, `j`, `a`, `b` use `O(1)` space
- The dominant factor is the DP array, making the overall space complexity `O(n * (s1 + s2))`

Note: In the worst case where all elements have large values, `s1` and `s2` could be proportional to `n * max_value`, making both complexities potentially pseudo-polynomial rather than truly polynomial in the input size.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Index Offset Confusion**
The most common pitfall is misunderstanding how the offset works for handling negative differences. The DP table uses `total_sum2` as an offset to map difference values to valid array indices.

**Pitfall Example:**
```python
# Incorrect: Forgetting to apply offset consistently
dp[i][current_val1] += 1  # Wrong! This doesn't account for the offset
dp[i][-current_val2] += 1  # Wrong! Negative index without offset
```

**Solution:**
Always apply the offset `total_sum2` when accessing the DP table:
```python
# Correct: Apply offset for both positive and negative differences
dp[i][current_val1 + total_sum2] += 1
dp[i][-current_val2 + total_sum2] += 1
```

### 2. **Boundary Check Errors**
Another critical pitfall is not properly checking boundaries when extending subarrays, which can lead to index out of bounds errors.

**Pitfall Example:**
```python
# Incorrect: No boundary checks
for diff in range(max_diff_range):
    dp[i][diff] += dp[i-1][diff - current_val1]  # May underflow!
    dp[i][diff] += dp[i-1][diff + current_val2]  # May overflow!
```

**Solution:**
Always validate that indices stay within bounds:
```python
# Correct: Check boundaries before accessing
if diff >= current_val1:
    dp[i][diff] = (dp[i][diff] + dp[i-1][diff - current_val1]) % MOD
if diff + current_val2 < max_diff_range:
    dp[i][diff] = (dp[i][diff] + dp[i-1][diff + current_val2]) % MOD
```

### 3. **Incorrect Difference Calculation**
A subtle pitfall is confusing how picking from each array affects the difference.

**Pitfall Example:**
```python
# Incorrect: Wrong sign for difference updates
dp[i][current_val2 + total_sum2] += 1  # Wrong! nums2 should subtract
dp[i][-current_val1 + total_sum2] += 1  # Wrong! nums1 should add
```

**Solution:**
Remember the convention:
- Picking from `nums1[i]` **adds** `nums1[i]` to the difference
- Picking from `nums2[i]` **subtracts** `nums2[i]` from the difference

### 4. **Memory Optimization Missed**
While not incorrect, using a full 2D array can be memory-intensive. Since we only need the previous row to compute the current row, we can optimize space.

**Space-Optimized Solution:**
```python
# Use two 1D arrays instead of a 2D array
prev_dp = [0] * max_diff_range
curr_dp = [0] * max_diff_range

for i in range(n):
    curr_dp = [0] * max_diff_range
    # Base cases
    curr_dp[current_val1 + total_sum2] += 1
    curr_dp[-current_val2 + total_sum2] += 1

    # Transitions from previous
    if i > 0:
        for diff in range(max_diff_range):
            if diff >= current_val1:
                curr_dp[diff] = (curr_dp[diff] + prev_dp[diff - current_val1]) % MOD
            if diff + current_val2 < max_diff_range:
                curr_dp[diff] = (curr_dp[diff] + prev_dp[diff + current_val2]) % MOD

    result = (result + curr_dp[total_sum2]) % MOD
    prev_dp = curr_dp
```

This reduces space complexity from O(n × (s1 + s2)) to O(s1 + s2).

