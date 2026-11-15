# 3193. Count the Number of Inversions

> Source: https://algo.monster/liteproblems/3193

## Problem Statement

You are given an integer `n` and a 2D array `requirements`. Each element `requirements[i] = [end_i, cnt_i]` specifies that the subarray from index 0 to index `end_i` must have exactly `cnt_i` inversions.

An **inversion** in an array `nums` is a pair of indices `(i, j)` where:
- `i < j` 
- `nums[i] > nums[j]`

Your task is to find how many permutations of `[0, 1, 2, ..., n - 1]` satisfy all the given requirements. For each requirement, the subarray `perm[0..end_i]` must contain exactly `cnt_i` inversions.

For example, if you have a permutation `[2, 0, 1, 3]` and you want to count inversions in `perm[0..2]` (which is `[2, 0, 1]`), the inversions would be:
- `(0, 1)` because `2 > 0`
- `(0, 2)` because `2 > 1`

So this subarray has 2 inversions.

The answer should be returned modulo `10^9 + 7` since it can be very large.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) where `f[i][j]` represents the number of permutations of elements `[0..i]` that have exactly `j` inversions. The key insight is that when placing a number at position `i`, if it's smaller than `k` numbers before it, it creates `k` new inversions. The transition formula is:

`f[i][j] = Σ(k=0 to min(i,j)) f[i-1][j-k]`

## Intuition

Let's think about how inversions are created when building a permutation step by step. When we place a number at position `i`, we need to understand how it affects the total inversion count.

Imagine we're building the permutation from left to right. At position `i`, we have already placed `i` numbers (at positions `0` to `i-1`). Now we need to place one more number. The key observation is: **the number of new inversions created depends on how many of the previously placed numbers are larger than the current number**.

For example, if we have placed `[2, 0, 1]` and want to add another number at position 3:
- If we add `3`, it's larger than all previous numbers, so it creates 0 new inversions
- If we add a number that's smaller than 2 of the previous numbers, it creates 2 new inversions

This leads us to think about the problem in terms of **relative ordering**. When working with permutations of `[0, 1, 2, ..., n-1]`, we can think of each position as choosing how many numbers to the left should be greater than the current number.

The [dynamic programming](/problems/dynamic_programming_intro) approach emerges naturally from this observation:
- `f[i][j]` = number of ways to arrange the first `i+1` positions with exactly `j` inversions
- When moving from position `i-1` to position `i`, we can choose to create `k` new inversions (where `0 ≤ k ≤ i`)
- If we had `j-k` inversions up to position `i-1`, and we create `k` new inversions at position `i`, we get `j` total inversions

The constraint that certain prefixes must have exact inversion counts simply means that at those positions, we only consider states with the required number of inversions. This naturally filters out invalid permutations as we build them.

The formula `f[i][j] = Σ(k=0 to min(i,j)) f[i-1][j-k]` captures this: we sum over all possible ways to reach `j` inversions by adding different amounts of new inversions at position `i`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation follows the [dynamic programming](/problems/dynamic_programming_intro) approach we established, with careful handling of the requirements constraints.

**Step 1: Process Requirements**
```python
req = [-1] * n
for end, cnt in requirements:
    req[end] = cnt
```
We create an array `req` where `req[i]` stores the required number of inversions for position `i`. We initialize with `-1` to indicate positions without requirements.

**Step 2: Handle Base Case**
```python
if req[0] > 0:
    return 0
req[0] = 0
```
Position 0 can never have inversions (there are no elements before it), so if any requirement asks for inversions at position 0, we return 0. Otherwise, we set `req[0] = 0`.

**Step 3: Initialize DP Table**
```python
m = max(req)
f = [[0] * (m + 1) for _ in range(n)]
f[0][0] = 1
```
We create a 2D DP table where `f[i][j]` represents the number of permutations of `[0..i]` with exactly `j` inversions. We only need to track up to `m` inversions (the maximum required). The base case is `f[0][0] = 1` - one way to arrange a single element with 0 inversions.

**Step 4: Fill DP Table**
```python
for i in range(1, n):
    l, r = 0, m
    if req[i] >= 0:
        l = r = req[i]
    for j in range(l, r + 1):
        for k in range(min(i, j) + 1):
            f[i][j] = (f[i][j] + f[i - 1][j - k]) % mod
```

For each position `i`:
- If there's a requirement at position `i` (i.e., `req[i] >= 0`), we only compute `f[i][req[i]]` by setting `l = r = req[i]`
- Otherwise, we compute all possible inversion counts from 0 to `m`
- For each target inversion count `j`, we sum contributions from all valid previous states

The inner loop iterates `k` from 0 to `min(i, j)`:
- `k` represents the number of new inversions created at position `i`
- We can create at most `i` inversions (if the new element is smaller than all previous ones)
- We can't create more than `j` inversions total

The transition formula `f[i][j] = f[i][j] + f[i-1][j-k]` accumulates all ways to reach `j` inversions at position `i` by having `j-k` inversions at position `i-1` and creating `k` new ones.

**Step 5: Return Result**
```python
return f[n - 1][req[n - 1]]
```
The answer is the number of permutations of length `n` with exactly `req[n-1]` inversions. Since we enforced all intermediate requirements during the DP computation, this value represents only valid permutations.

The modulo operation `% mod` is applied throughout to prevent integer overflow, where `mod = 10^9 + 7`.

## Example Walkthrough

Let's walk through a small example with `n = 4` and `requirements = [[2, 2], [3, 0]]`.

This means:
- The subarray `perm[0..2]` must have exactly 2 inversions
- The subarray `perm[0..3]` (entire array) must have exactly 0 inversions

**Step 1: Process Requirements**
```
req = [-1, -1, 2, 0]
```
Position 2 requires 2 inversions, position 3 requires 0 inversions.

**Step 2: Initialize DP Table**
```
f[0][0] = 1  (one way to place first element with 0 inversions)
```

**Step 3: Build Position 1**
At position 1, we can create 0 or 1 inversions:
- `f[1][0] = f[0][0] = 1` (place smaller number, no new inversions)
- `f[1][1] = f[0][0] = 1` (place larger number, one new inversion)

**Step 4: Build Position 2**
Position 2 has requirement `req[2] = 2`, so we only compute `f[2][2]`:
- To get 2 total inversions, we can:
  - Have 2 inversions from before, add 0 new: `f[1][2]` (doesn't exist)
  - Have 1 inversion from before, add 1 new: `f[1][1] = 1`
  - Have 0 inversions from before, add 2 new: `f[1][0] = 1`
- `f[2][2] = 0 + 1 + 1 = 2`

**Step 5: Build Position 3**
Position 3 has requirement `req[3] = 0`, so we only compute `f[3][0]`:
- To get 0 total inversions at position 3, we must have had 0 inversions at position 2 and add 0 new inversions
- But `req[2] = 2`, so `f[2][0] = 0`
- Therefore, `f[3][0] = 0`

**Result:** The answer is 0. There are no permutations that satisfy both requirements.

This makes sense intuitively: if we already have 2 inversions by position 2, we cannot reduce that count to 0 by position 3. Once inversions are created, they cannot be "undone" by adding more elements.

Let's verify with a different example where a solution exists: `n = 3` and `requirements = [[2, 1]]`.

**DP Table Evolution:**
- `f[0][0] = 1`
- Position 1: `f[1][0] = 1`, `f[1][1] = 1`
- Position 2 (requirement: exactly 1 inversion):
  - `f[2][1] = f[1][1] + f[1][0] = 1 + 1 = 2`

**Result:** 2 valid permutations exist.

These are `[0, 2, 1]` and `[1, 0, 2]`:
- `[0, 2, 1]`: The subarray `[0, 2, 1]` has 1 inversion: (1,2) since 2 > 1
- `[1, 0, 2]`: The subarray `[1, 0, 2]` has 1 inversion: (0,1) since 1 > 0

## Python Solution

```python
class Solution:
    def numberOfPermutations(self, n: int, requirements: List[List[int]]) -> int:
        # Initialize array to store required inversion counts at each position
        # -1 means no requirement at that position
        required_inversions = [-1] * n
      
        # Populate requirements: at position 'end', we need exactly 'cnt' inversions
        for end_position, inversion_count in requirements:
            required_inversions[end_position] = inversion_count
      
        # Position 0 must have 0 inversions (base case)
        if required_inversions[0] > 0:
            return 0
        required_inversions[0] = 0
      
        # Constants for modulo arithmetic and maximum inversions needed
        MOD = 10**9 + 7
        max_inversions = max(required_inversions)
      
        # DP table: dp[i][j] = number of ways to arrange first i+1 elements 
        # with exactly j inversions
        dp = [[0] * (max_inversions + 1) for _ in range(n)]
      
        # Base case: one way to arrange first element with 0 inversions
        dp[0][0] = 1
      
        # Fill DP table for each position
        for position in range(1, n):
            # Determine the range of valid inversion counts for this position
            min_inversions, max_inversions_at_pos = 0, max_inversions
          
            # If there's a requirement at this position, fix the range
            if required_inversions[position] >= 0:
                min_inversions = max_inversions_at_pos = required_inversions[position]
          
            # Calculate number of ways for each valid inversion count
            for current_inversions in range(min_inversions, max_inversions_at_pos + 1):
                # Try adding element at position with k new inversions
                # k can be at most min(position, current_inversions)
                for new_inversions in range(min(position, current_inversions) + 1):
                    dp[position][current_inversions] = (
                        dp[position][current_inversions] + 
                        dp[position - 1][current_inversions - new_inversions]
                    ) % MOD
      
        # Return the number of valid permutations with required inversions at last position
        return dp[n - 1][required_inversions[n - 1]]
```

## Time and Space Complexity

**Time Complexity:** `O(n × m × min(n, m))`

The code uses three nested loops:
- The outer loop iterates through `i` from `1` to `n-1`, contributing `O(n)`
- The middle loop iterates through `j` from `l` to `r+1`, where in the worst case this ranges from `0` to `m`, contributing `O(m)`
- The inner loop iterates through `k` from `0` to `min(i, j)`, which is bounded by `min(n, m)`, contributing `O(min(n, m))`

Therefore, the overall time complexity is `O(n × m × min(n, m))`, where `n` is the input size and `m` is the maximum number of inversions (which is at most 400 in this problem).

**Space Complexity:** `O(n × m)`

The code allocates:
- A `req` array of size `n`: `O(n)`
- A 2D DP table `f` with dimensions `n × (m + 1)`: `O(n × m)`

The dominant space usage comes from the DP table, giving us a total space complexity of `O(n × m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Positions Without Requirements**

**Pitfall:** A common mistake is not properly handling positions that don't have explicit requirements. The code might incorrectly assume these positions need 0 inversions or fail to consider all possible inversion counts.

**Issue in Code:**
```python
# Wrong approach - forcing unrequired positions to have specific inversions
for i in range(1, n):
    if req[i] >= 0:
        # Only compute for required inversions
        for j in range(req[i], req[i] + 1):
            # ...
    # Missing else case - not computing other possibilities!
```

**Solution:** When a position has no requirement (`req[i] == -1`), compute all possible inversion counts from 0 to the maximum:
```python
for i in range(1, n):
    l, r = 0, max_inversions
    if req[i] >= 0:
        l = r = req[i]  # Fix range only if requirement exists
    for j in range(l, r + 1):
        # Compute for entire range
```

### 2. **Overflow in Modulo Arithmetic**

**Pitfall:** Forgetting to apply modulo operation during accumulation can cause integer overflow, especially in languages with fixed integer sizes.

**Issue in Code:**
```python
# Wrong - might overflow before taking modulo
f[i][j] = f[i][j] + f[i - 1][j - k]
result = result % MOD  # Too late!
```

**Solution:** Apply modulo immediately during each addition:
```python
f[i][j] = (f[i][j] + f[i - 1][j - k]) % MOD
```

### 3. **Incorrect Maximum Inversions Calculation**

**Pitfall:** Using the wrong upper bound for the DP table size. Some might use `n*(n-1)/2` (theoretical maximum) which wastes memory, or might underestimate the needed size.

**Issue in Code:**
```python
# Wrong - using theoretical maximum wastes memory
m = n * (n - 1) // 2
dp = [[0] * (m + 1) for _ in range(n)]  # Too large!
```

**Solution:** Use only the maximum from actual requirements:
```python
m = max(req)  # Only allocate what's needed
dp = [[0] * (m + 1) for _ in range(n)]
```

### 4. **Off-by-One Error in Loop Bounds**

**Pitfall:** Incorrectly setting the upper bound for the number of new inversions `k`. Remember that at position `i`, you can create at most `i` new inversions (not `i+1` or `i-1`).

**Issue in Code:**
```python
# Wrong - can create too many inversions
for k in range(min(i + 1, j) + 1):  # Should be i, not i+1
    f[i][j] = (f[i][j] + f[i - 1][j - k]) % MOD
```

**Solution:** Use the correct bound:
```python
for k in range(min(i, j) + 1):  # k can be 0 to min(i, j) inclusive
    f[i][j] = (f[i][j] + f[i - 1][j - k]) % MOD
```

### 5. **Not Validating Impossible Requirements Early**

**Pitfall:** Not checking if requirements are theoretically possible before running the DP, leading to unnecessary computation or incorrect results.

**Issue in Code:**
```python
# Missing validation - what if req[i] > i*(i+1)/2?
for end, cnt in requirements:
    req[end] = cnt  # Should validate first!
```

**Solution:** Add early validation:
```python
for end, cnt in requirements:
    # Maximum possible inversions for subarray [0..end]
    max_possible = end * (end + 1) // 2
    if cnt > max_possible or cnt < 0:
        return 0
    req[end] = cnt
```

