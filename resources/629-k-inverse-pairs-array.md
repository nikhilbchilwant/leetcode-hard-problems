# 629. K Inverse Pairs Array

> Source: https://algo.monster/liteproblems/629

## Problem Statement

Given two integers `n` and `k`, you need to count how many different arrays of length `n` containing numbers from `1` to `n` have exactly `k` inverse pairs.

An **inverse pair** in an array is defined as a pair of indices `[i, j]` where:
- `0 <= i < j < array.length`
- `array[i] > array[j]`

In other words, an inverse pair occurs when a larger number appears before a smaller number in the array.

For example:
- Array `[1, 3, 2]` has 1 inverse pair: `[1, 2]` (since `3 > 2`)
- Array `[3, 2, 1]` has 3 inverse pairs: `[0, 1]`, `[0, 2]`, and `[1, 2]`
- Array `[1, 2, 3]` has 0 inverse pairs

The task is to find the total number of permutations of numbers from `1` to `n` that result in exactly `k` inverse pairs. Since the answer can be very large, return the result modulo `10^9 + 7`.

## Intuition

Let's think about how to build arrays with a specific number of inverse pairs step by step.

Start with a smaller problem: if we have an array of length `i-1` with some number of inverse pairs, what happens when we add the number `i` to it?

The key insight is that when we insert the largest number `i` into different positions of an existing array, we can control exactly how many new inverse pairs we create:

- If we place `i` at the end (rightmost position), it creates `0` new inverse pairs (since `i` is larger than all previous numbers)
- If we place `i` at the second-to-last position, it creates `1` new inverse pair (with the number that was originally at the last position)
- If we place `i` at the third-to-last position, it creates `2` new inverse pairs
- ...
- If we place `i` at the beginning (leftmost position), it creates `i-1` new inverse pairs (with all other numbers)

This means if we know the number of ways to create arrays of length `i-1` with `j` inverse pairs, we can compute the number of ways to create arrays of length `i` with `k` inverse pairs by considering all valid placements of the number `i`.

For `f[i][k]` (arrays of length `i` with `k` inverse pairs), we sum up:
- Ways to have `k` inverse pairs by placing `i` at the end (need `k` inverse pairs from the first `i-1` numbers)
- Ways to have `k` inverse pairs by placing `i` second-to-last (need `k-1` inverse pairs from the first `i-1` numbers)
- ...
- Ways to have `k` inverse pairs by placing `i` at the beginning (need `k-(i-1)` inverse pairs from the first `i-1` numbers)

This gives us the recurrence: `f[i][k] = f[i-1][k] + f[i-1][k-1] + ... + f[i-1][k-(i-1)]`

The sum pattern suggests we can optimize using prefix sums to avoid recalculating overlapping ranges, and since each state only depends on the previous row, we can optimize space by using a rolling array.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement the solution using [dynamic programming](/problems/dynamic_programming_intro) with space optimization and prefix sum optimization.

**[Dynamic Programming](/problems/dynamic_programming_intro) Setup:**

Define `f[j]` as the number of arrays of the current length with `j` inverse pairs. We iterate through lengths from `1` to `n`, updating this array for each length.

Initially, `f[0] = 1` (one way to have 0 inverse pairs with any length), and all other `f[j] = 0`.

**Prefix Sum Optimization:**

Since calculating `f[i][j]` requires summing `f[i-1][j] + f[i-1][j-1] + ... + f[i-1][max(0, j-(i-1))]`, we use a prefix sum array `s` to efficiently compute these range sums.

The prefix sum array `s[j]` stores the cumulative sum: `s[j] = f[0] + f[1] + ... + f[j-1]`

**Implementation Steps:**

1. **Initialize arrays:**
   - `f = [1] + [0] * k`: Start with `f[0] = 1`
   - `s = [0] * (k + 2)`: Prefix sum array with extra space for boundary handling

2. **For each array length `i` from `1` to `n`:**
   - **Update `f[j]` for each `j` from `1` to `k`:**
     - The number of inverse pairs `j` can be formed by placing number `i` at different positions
     - We need the sum of `f[i-1][j-p]` where `p` ranges from `0` to `min(j, i-1)`
     - Using prefix sums: `f[j] = s[j+1] - s[max(0, j-(i-1))]`
     - Apply modulo to keep numbers manageable

   - **Rebuild prefix sum array for next iteration:**
     - `s[j] = s[j-1] + f[j-1]` for each position
     - This prepares the prefix sums for the next length

3. **Return `f[k]`:** The final answer for arrays of length `n` with exactly `k` inverse pairs

**Space Optimization:**

Instead of maintaining a 2D array `f[i][j]`, we use a 1D array that gets updated in place. This works because:
- Each `f[i][j]` only depends on values from `f[i-1][...]`
- We process in order, so by the time we calculate `f[j]`, we've already used the old `f[j]` value

**Time Complexity:** `O(n * k)` - we iterate through `n` lengths and `k` inverse pairs for each  
**Space Complexity:** `O(k)` - we only maintain arrays of size proportional to `k`

## Example Walkthrough

Let's trace through finding the number of permutations of `[1, 2, 3]` (n=3) with exactly k=2 inverse pairs.

**Initial Setup:**
- `f = [1, 0, 0]` (f[j] represents ways to form j inverse pairs)
- `s = [0, 0, 0, 0]` (prefix sum array)

**Iteration 1: Adding number 1 (i=1)**
- Arrays of length 1 can only have 0 inverse pairs: `[1]`
- No updates needed since k > 0
- Update prefix sums: `s = [0, 1, 1, 1]`

**Iteration 2: Adding number 2 (i=2)**
- We can place 2 in two positions in arrays containing just 1:
  - `[1, 2]` - placing 2 at end creates 0 new inverse pairs (total: 0)
  - `[2, 1]` - placing 2 at start creates 1 new inverse pair (total: 1)

- For j=1: `f[1] = s[2] - s[max(0, 1-1)] = s[2] - s[0] = 1 - 0 = 1`
  - This counts: `[2, 1]` (1 way)
- For j=2: `f[2] = s[3] - s[max(0, 2-1)] = s[3] - s[1] = 1 - 1 = 0`
  - No arrays of length 2 can have 2 inverse pairs

- Result: `f = [1, 1, 0]`
- Update prefix sums: `s = [0, 1, 2, 2]`

**Iteration 3: Adding number 3 (i=3)**
- We can place 3 in three positions in existing arrays:
  - At end (position 2): creates 0 new inverse pairs
  - At middle (position 1): creates 1 new inverse pair
  - At start (position 0): creates 2 new inverse pairs

- For j=1: `f[1] = s[2] - s[max(0, 1-2)] = s[2] - s[0] = 2 - 0 = 2`
  - From `[1, 2]` (0 inversions): place 3 at position 1 → `[1, 3, 2]` (1 inversion)
  - From `[2, 1]` (1 inversion): place 3 at end → `[2, 1, 3]` (1 inversion)

- For j=2: `f[2] = s[3] - s[max(0, 2-2)] = s[3] - s[0] = 2 - 0 = 2`
  - From `[1, 2]` (0 inversions): place 3 at position 0 → `[3, 1, 2]` (2 inversions)
  - From `[2, 1]` (1 inversion): place 3 at position 1 → `[2, 3, 1]` (2 inversions)

**Final Answer:** `f[2] = 2`

The two permutations of `[1, 2, 3]` with exactly 2 inverse pairs are:
1. `[3, 1, 2]` - inverse pairs: (0,1) and (0,2)
2. `[2, 3, 1]` - inverse pairs: (0,2) and (1,2)

## Python Solution

```python
class Solution:
    def kInversePairs(self, n: int, k: int) -> int:
        """
        Find the number of different arrays consisting of numbers from 1 to n 
        such that there are exactly k inverse pairs.
      
        An inverse pair is a pair (i, j) where i < j but arr[i] > arr[j].
      
        Args:
            n: The range of numbers [1, n] to form the array
            k: The target number of inverse pairs
          
        Returns:
            The count of valid arrays modulo 10^9 + 7
        """
        MOD = 10**9 + 7
      
        # dp[j] represents the number of permutations with exactly j inverse pairs
        # Initially, with 0 numbers, there's 1 way to have 0 inverse pairs (empty permutation)
        dp = [1] + [0] * k
      
        # prefix_sum[j] stores the cumulative sum of dp[0] to dp[j-1]
        # Used for range sum queries to optimize the DP transition
        prefix_sum = [0] * (k + 2)
      
        # Build up permutations by adding numbers from 1 to n
        for current_number in range(1, n + 1):
            # Calculate new dp values for current_number elements
            for inverse_pairs in range(1, k + 1):
                # When inserting current_number into a permutation of (current_number - 1) elements,
                # we can place it at positions creating 0 to min(inverse_pairs, current_number - 1) new inverse pairs
                # 
                # The range of valid previous inverse pairs is:
                # [max(0, inverse_pairs - (current_number - 1)), inverse_pairs]
                min_prev_pairs = max(0, inverse_pairs - (current_number - 1))
              
                # Sum dp values in the valid range using prefix sums
                dp[inverse_pairs] = (prefix_sum[inverse_pairs + 1] - prefix_sum[min_prev_pairs]) % MOD
          
            # Update prefix sum array for the next iteration
            for j in range(1, k + 2):
                prefix_sum[j] = (prefix_sum[j - 1] + dp[j - 1]) % MOD
      
        return dp[k]
```

## Time and Space Complexity

The time complexity is `O(n × k)`, where `n` is the number of elements in the permutation and `k` is the target number of inverse pairs. This is because we have two nested loops: the outer loop iterates `n` times (from 1 to n), and for each iteration, the inner loop iterates `k` times to update the dynamic programming array `f[j]`. Additionally, there's another loop that updates the prefix sum array `s`, which also runs `k + 1` times, but this doesn't change the overall complexity as it's still `O(k)` for each outer iteration.

The space complexity is `O(k)`. The algorithm uses two arrays: array `f` of size `k + 1` to store the number of permutations with exactly `j` inverse pairs, and array `s` of size `k + 2` to store the prefix sums. Both arrays have space proportional to `k`, resulting in `O(k)` space complexity overall.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Errors in Prefix Sum Array Indexing**

One of the most common mistakes in this solution is incorrectly handling the prefix sum array indices. The prefix sum array needs careful indexing because:
- `prefix_sum[j]` stores the sum of `dp[0]` through `dp[j-1]`
- When calculating range sums, you need `prefix_sum[j+1] - prefix_sum[min_prev_pairs]`

**Incorrect Implementation:**
```python
# Wrong: Using incorrect indices for prefix sum
dp[inverse_pairs] = (prefix_sum[inverse_pairs] - prefix_sum[min_prev_pairs - 1]) % MOD
```

**Correct Implementation:**
```python
# Correct: Proper prefix sum indexing
dp[inverse_pairs] = (prefix_sum[inverse_pairs + 1] - prefix_sum[min_prev_pairs]) % MOD
```

### 2. **Forgetting to Handle Negative Numbers After Modulo**

Because the prefix sums are stored modulo `MOD`, the difference `prefix_sum[j + 1] - prefix_sum[min_prev_pairs]` can be negative even though the true range sum is positive. In Python this is safe: `%` with a positive modulus always returns a value in `[0, MOD)`, so the solution's line is correct. In Java, C++, or JavaScript, `%` keeps the sign of the left operand, and the result can be negative.

**Incorrect Implementation (Java/C++ style):**
```python
# Wrong in languages where % can return a negative value
dp[j] = (prefix_sum[j + 1] - prefix_sum[min_prev_pairs]) % MOD
```

**Portable Implementation:**
```python
diff = prefix_sum[j + 1] - prefix_sum[min_prev_pairs]
dp[j] = (diff % MOD + MOD) % MOD
```

### 3. **Not Properly Resetting the Prefix Sum Array**

The prefix sum array must be rebuilt completely after each iteration for the current number. A common mistake is trying to update it incrementally without proper initialization.

**Incorrect Implementation:**
```python
# Wrong: Not resetting prefix_sum properly
for j in range(1, k + 2):
    prefix_sum[j] += dp[j - 1]  # Accumulating without reset
```

**Correct Implementation:**
```python
# Correct: Rebuild prefix_sum from scratch each iteration
for j in range(1, k + 2):
    prefix_sum[j] = (prefix_sum[j - 1] + dp[j - 1]) % MOD
```

### 4. **Incorrect Calculation of Maximum Possible New Inverse Pairs**

When inserting number `i` into a permutation, it can create at most `i-1` new inverse pairs (when placed at the beginning). The valid range calculation often trips people up.

**Incorrect Implementation:**
```python
# Wrong: Using current_number instead of current_number - 1
min_prev_pairs = max(0, inverse_pairs - current_number)
```

**Correct Implementation:**
```python
# Correct: Maximum new inverse pairs is current_number - 1
min_prev_pairs = max(0, inverse_pairs - (current_number - 1))
```

### 5. **Mixing Rows When Updating the 1D Array**

The 1D array works because every `dp[j]` for the new length is computed only from `prefix_sum`, which still describes the previous length. The order of `j` does not matter, and recomputing `dp[0]` would give `prefix_sum[1] - prefix_sum[0] = 1`, the correct value. The mistake is updating `prefix_sum` inside the same loop, so later values of `j` read prefix sums that already include the new row.

**Incorrect Implementation:**
```python
for inverse_pairs in range(1, k + 1):
    min_prev_pairs = max(0, inverse_pairs - (current_number - 1))
    dp[inverse_pairs] = (prefix_sum[inverse_pairs + 1] - prefix_sum[min_prev_pairs]) % MOD
    # Wrong: prefix_sum now mixes the old and new rows
    prefix_sum[inverse_pairs + 1] = (prefix_sum[inverse_pairs] + dp[inverse_pairs]) % MOD
```

**Correct Implementation:**
Compute the whole new row first, then rebuild the prefix sums in a separate loop:
```python
for inverse_pairs in range(1, k + 1):
    min_prev_pairs = max(0, inverse_pairs - (current_number - 1))
    dp[inverse_pairs] = (prefix_sum[inverse_pairs + 1] - prefix_sum[min_prev_pairs]) % MOD
for j in range(1, k + 2):
    prefix_sum[j] = (prefix_sum[j - 1] + dp[j - 1]) % MOD
```

