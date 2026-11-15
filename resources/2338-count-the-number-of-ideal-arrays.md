# 2338. Count the Number of Ideal Arrays

> Source: https://algo.monster/liteproblems/2338

## Problem Statement

You need to count how many distinct "ideal arrays" of length `n` can be formed, where each element is between 1 and `maxValue`.

An array is considered "ideal" if it satisfies two conditions:
1. Every element must be a value from 1 to `maxValue` (inclusive)
2. Each element (except the first) must be divisible by the element immediately before it - meaning `arr[i]` is divisible by `arr[i-1]` for all positions `i > 0`

For example, `[1, 2, 6]` is an ideal array because 2 is divisible by 1, and 6 is divisible by 2. Similarly, `[3, 6, 6]` is ideal because 6 is divisible by 3, and 6 is divisible by 6.

The key insight is that in an ideal array, the values form a non-decreasing sequence where each element is a multiple of the previous one. This means if we start with value `a`, the next value must be `a*k` for some integer `k ≥ 1`.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) where `f[i][j]` represents the number of distinct sequences ending with value `i` and having exactly `j` distinct values. Since we have an array of length `n` but only `j` distinct values, we need to distribute these `j` values across `n` positions. This is done using combinations - specifically `C(n-1, j-1)`, which represents choosing `j-1` positions from `n-1` possible positions to place the transitions between different values.

The algorithm:
1. Preprocesses combination values using Pascal's triangle
2. Computes `f[i][j]` by considering all multiples of each value
3. Combines the results by multiplying the number of distinct sequences with the ways to distribute them across the array length

Since the answer can be very large, return it modulo `10^9 + 7`.

## Intuition

The key observation is that an ideal array has a special structure: each element must be a multiple of the previous one. This means if we look at the sequence of distinct values in the array, they form a divisibility chain like `[a, a*k₁, a*k₁*k₂, ...]`.

Let's think about this problem in two parts:
1. **What distinct values can appear in the array?** Since each value must divide the next, we're essentially building chains of divisors. For example, starting from 2, we could have chains like `[2, 4, 8]` or `[2, 6, 12]`.

2. **How can we arrange these distinct values in an array of length `n`?** Since the array must be non-decreasing (due to the divisibility constraint), we can have repetitions. For instance, with distinct values `[2, 4]` and array length 4, we could have `[2, 2, 4, 4]` or `[2, 4, 4, 4]`.

This leads us to think about the problem as:
- First, find all possible chains of distinct values where each divides the next
- Then, figure out how to distribute these values across `n` positions

For the first part, we can use [dynamic programming](/problems/dynamic_programming_intro). Let `f[i][j]` represent the number of ways to form a chain of exactly `j` distinct values ending with value `i`. We can build longer chains by considering all multiples of `i`: if we have a chain ending at `i` with `j` values, we can extend it to any multiple `k*i` (where `k*i ≤ maxValue`) to get a chain with `j+1` values.

For the second part, imagine we have `j` distinct values to place in `n` positions. Since the array must be non-decreasing, this is equivalent to dividing `n` positions into `j` groups (where each group contains copies of the same value). This is a classic "stars and bars" problem - we need to place `j-1` dividers among `n-1` possible positions, giving us `C(n-1, j-1)` ways.

The maximum number of distinct values in any chain is limited because each value must be at least double the previous one (smallest multiplier is 2). So the longest possible chain has length approximately `log₂(maxValue)`, which is why we only need to consider up to about 15 distinct values.

By combining these two insights - counting the chains of distinct values and then distributing them across the array - we arrive at the final answer: sum over all possible ending values `i` and chain lengths `j` of `f[i][j] * C(n-1, j-1)`.

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The implementation consists of three main parts: preprocessing combinations, computing the DP table for distinct value chains, and calculating the final answer.

**Step 1: Precompute Combinations using Pascal's Triangle**

We build a 2D array `c[i][j]` to store `C(i, j)` values using the recurrence relation:
- `c[i][j] = c[i-1][j] + c[i-1][j-1]`
- Base case: `c[i][0] = 1` for all `i`

```python
c = [[0] * 16 for _ in range(n)]
for i in range(n):
    for j in range(min(16, i + 1)):
        c[i][j] = 1 if j == 0 else (c[i - 1][j] + c[i - 1][j - 1]) % mod
```

We only need combinations up to 16 columns because the maximum chain length is bounded by `log₂(maxValue) + 1 ≈ 15`.

**Step 2: Build DP Table for Distinct Value Chains**

Initialize `f[i][j]` where:
- `i` represents the ending value (1 to `maxValue`)
- `j` represents the number of distinct values in the chain
- `f[i][j]` = number of ways to form a chain of `j` distinct values ending at `i`

Base case: `f[i][1] = 1` for all `i` (single element chains).

```python
f = [[0] * 16 for _ in range(maxValue + 1)]
for i in range(1, maxValue + 1):
    f[i][1] = 1
```

For each chain length `j` from 1 to 14, and each value `i`, we extend the chain to all its multiples:

```python
for j in range(1, 15):
    for i in range(1, maxValue + 1):
        k = 2
        while k * i <= maxValue:
            f[k * i][j + 1] = (f[k * i][j + 1] + f[i][j]) % mod
            k += 1
```

This builds chains by considering: if we have a chain of length `j` ending at `i`, we can extend it to `k*i` (for `k ≥ 2`) to create a chain of length `j+1`.

**Step 3: Calculate Final Answer**

For each possible ending value `i` and chain length `j`, multiply:
- `f[i][j]`: number of distinct value chains
- `c[n-1][j-1]`: ways to distribute `j` distinct values across `n` positions

```python
ans = 0
for i in range(1, maxValue + 1):
    for j in range(1, 16):
        ans = (ans + f[i][j] * c[-1][j - 1]) % mod
```

The formula `c[n-1][j-1]` represents choosing `j-1` positions from `n-1` possible spots to place dividers, effectively distributing `j` distinct values into `n` array positions while maintaining non-decreasing order.

**Time Complexity**: `O(maxValue × log² maxValue)` - For each value up to `maxValue`, we consider chains up to length `log maxValue`, and for each chain, we check multiples up to `maxValue`.

**Space Complexity**: `O(n × log maxValue + maxValue × log maxValue)` - For storing the combination table and DP table.

## Example Walkthrough

Let's walk through a small example with `n = 3` and `maxValue = 5` to illustrate the solution approach.

**Step 1: Precompute Combinations**

We need combinations `C(i,j)` for distributing distinct values. For `n=3`, we build:
```
c[0][0] = 1
c[1][0] = 1, c[1][1] = 1
c[2][0] = 1, c[2][1] = 2, c[2][2] = 1
```
So `C(2,0) = 1`, `C(2,1) = 2`, `C(2,2) = 1`.

**Step 2: Build DP Table for Distinct Value Chains**

Initialize `f[i][1] = 1` for all values 1 to 5 (single-element chains):
- `f[1][1] = 1`, `f[2][1] = 1`, `f[3][1] = 1`, `f[4][1] = 1`, `f[5][1] = 1`

Now build chains of length 2 by considering multiples:
- From value 1: multiples are 2,3,4,5 → `f[2][2] = 1`, `f[3][2] = 1`, `f[4][2] = 1`, `f[5][2] = 1`
- From value 2: multiple is 4 → `f[4][2] += 1` (now `f[4][2] = 2`)

After processing all values for length 2:
- `f[2][2] = 1` (chain: 1→2)
- `f[3][2] = 1` (chain: 1→3)
- `f[4][2] = 2` (chains: 1→4 and 2→4)
- `f[5][2] = 1` (chain: 1→5)

Build chains of length 3:
- From `f[2][2] = 1`: multiple is 4 → `f[4][3] = 1` (chain: 1→2→4)

**Step 3: Calculate Final Answer**

Now we combine the chains with ways to distribute them:

For chains of length 1 (need to fill 3 positions with 1 distinct value):
- Ways to distribute: `C(2,0) = 1`
- Contribution: `(f[1][1] + f[2][1] + f[3][1] + f[4][1] + f[5][1]) × 1 = 5 × 1 = 5`
- Examples: [1,1,1], [2,2,2], [3,3,3], [4,4,4], [5,5,5]

For chains of length 2 (need to fill 3 positions with 2 distinct values):
- Ways to distribute: `C(2,1) = 2` (either [a,a,b] or [a,b,b])
- Contribution: `(f[2][2] + f[3][2] + f[4][2] + f[5][2]) × 2 = (1+1+2+1) × 2 = 10`
- Examples for chain 1→2: [1,1,2] and [1,2,2]
- Examples for chain 2→4: [2,2,4] and [2,4,4]

For chains of length 3 (need to fill 3 positions with 3 distinct values):
- Ways to distribute: `C(2,2) = 1` (only [a,b,c])
- Contribution: `f[4][3] × 1 = 1 × 1 = 1`
- Example: [1,2,4]

**Total: 5 + 10 + 1 = 16 distinct ideal arrays**

Some of the 16 arrays are:
- Length-1 chains: [1,1,1], [2,2,2], [3,3,3], [4,4,4], [5,5,5]
- Length-2 chains: [1,1,2], [1,2,2], [1,1,3], [1,3,3], [1,1,4], [1,4,4], [2,2,4], [2,4,4], [1,1,5], [1,5,5]
- Length-3 chain: [1,2,4]

## Python Solution

```python
class Solution:
    def idealArrays(self, n: int, maxValue: int) -> int:
        MOD = 10**9 + 7
        MAX_LENGTH = 16  # Maximum possible length of strictly increasing subsequence
      
        # Build Pascal's triangle for combination calculations
        # combinations[i][j] represents C(i, j) = number of ways to choose j items from i items
        combinations = [[0] * MAX_LENGTH for _ in range(n)]
      
        for i in range(n):
            for j in range(min(MAX_LENGTH, i + 1)):
                if j == 0:
                    combinations[i][j] = 1  # C(i, 0) = 1
                else:
                    # Pascal's triangle formula: C(i, j) = C(i-1, j) + C(i-1, j-1)
                    combinations[i][j] = (combinations[i - 1][j] + combinations[i - 1][j - 1]) % MOD
      
        # dp[value][length] = number of strictly increasing sequences ending at 'value' with 'length' elements
        dp = [[0] * MAX_LENGTH for _ in range(maxValue + 1)]
      
        # Base case: all values can form a sequence of length 1
        for value in range(1, maxValue + 1):
            dp[value][1] = 1
      
        # Build up sequences of increasing lengths
        for length in range(1, MAX_LENGTH - 1):
            for current_value in range(1, maxValue + 1):
                # Try all multiples of current_value as next element in sequence
                multiplier = 2
                while multiplier * current_value <= maxValue:
                    next_value = multiplier * current_value
                    # Extend sequences ending at current_value by adding next_value
                    dp[next_value][length + 1] = (dp[next_value][length + 1] + dp[current_value][length]) % MOD
                    multiplier += 1
      
        # Calculate final answer by summing over all possible ending values and sequence lengths
        answer = 0
        for ending_value in range(1, maxValue + 1):
            for sequence_length in range(1, MAX_LENGTH):
                # For each strictly increasing sequence, we can arrange it in the array of size n
                # using combinations to place the increasing elements
                ways_to_arrange = combinations[n - 1][sequence_length - 1]
                answer = (answer + dp[ending_value][sequence_length] * ways_to_arrange) % MOD
      
        return answer
```

## Time and Space Complexity

**Time Complexity:** `O(n × 16 + maxValue × 16 × maxValue + maxValue × 16)` which simplifies to `O(n + maxValue²)`

The time complexity breaks down into three main parts:

1. Building the combination table `c`: The nested loops run for `n` iterations and up to `16` iterations, giving `O(n × 16) = O(n)`.

2. Computing the dynamic programming table `f`: 
   - The outer loop runs for `15` iterations (j from 1 to 14)
   - The middle loop runs for `maxValue` iterations (i from 1 to maxValue)
   - The inner while loop runs approximately `maxValue/i` times for each i
   - The total iterations for the inner loop across all values of i is approximately `maxValue × (1/1 + 1/2 + 1/3 + ... + 1/maxValue)` ≈ `maxValue × log(maxValue)`
   - However, since we're iterating through all multiples, the total work is `O(15 × maxValue × maxValue) = O(maxValue²)`

3. Computing the final answer: Two nested loops running `maxValue × 16` times, giving `O(maxValue × 16) = O(maxValue)`.

Overall: `O(n + maxValue² + maxValue) = O(n + maxValue²)`

**Space Complexity:** `O(n × 16 + maxValue × 16)` which simplifies to `O(n + maxValue)`

The space complexity comes from:
- Array `c` of size `n × 16` = `O(n)`
- Array `f` of size `(maxValue + 1) × 16` = `O(maxValue)`

Overall: `O(n + maxValue)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Understanding of Array Distribution**

**Pitfall**: A common mistake is misunderstanding how to distribute `j` distinct values across `n` positions. Many incorrectly assume we need `C(n, j)` combinations, which would be choosing `j` positions out of `n` for the distinct values.

**Why it's wrong**: The array must be non-decreasing, and we're not choosing which positions get which values. Instead, we're determining where transitions between different values occur.

**Correct approach**: Use `C(n-1, j-1)` which represents choosing `j-1` transition points from `n-1` possible positions between elements. This divides the array into `j` segments, each containing copies of the same value.

**Example**: For array length `n=5` with `j=3` distinct values `[a, b, c]`:
- We need 2 transition points (from `a` to `b`, and from `b` to `c`)
- These 2 points can be chosen from 4 possible positions (between elements)
- Result: `C(4, 2) = 6` ways to arrange like `[a,a,b,b,c]`, `[a,b,b,b,c]`, etc.

### 2. **Off-by-One Errors in DP Transition**

**Pitfall**: Starting the multiplier `k` from 1 instead of 2 when extending chains:
```python
# Wrong
k = 1  # This would include the same value, not creating a strictly increasing sequence
while k * i <= maxValue:
    f[k * i][j + 1] = (f[k * i][j + 1] + f[i][j]) % mod
    k += 1
```

**Why it's wrong**: Starting from `k=1` would mean `f[i][j+1]` includes transitions from `f[i][j]`, creating sequences with repeated consecutive values that aren't strictly increasing in terms of distinct values.

**Correct approach**: Start `k` from 2 to ensure each new value is strictly greater than the previous:
```python
k = 2  # Ensures k*i > i, maintaining strictly increasing distinct values
while k * i <= maxValue:
    f[k * i][j + 1] = (f[k * i][j + 1] + f[i][j]) % mod
    k += 1
```

### 3. **Insufficient Array Bounds for Chain Length**

**Pitfall**: Using a fixed small size like 10 or 12 for the maximum chain length without proper calculation:
```python
# Potentially wrong
MAX_LENGTH = 10  # Too small for some inputs
```

**Why it's wrong**: The maximum chain length depends on `maxValue`. For example, the chain `[1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024]` has 11 distinct values when `maxValue ≥ 1024`.

**Correct approach**: Calculate based on the logarithm of `maxValue`. Since each value must be at least double the previous one in the worst case, the maximum chain length is approximately `log₂(maxValue) + 1`:
```python
MAX_LENGTH = 16  # Safe for maxValue up to 10^4
# Or dynamically: MAX_LENGTH = min(16, int(math.log2(maxValue)) + 2)
```

### 4. **Forgetting Modulo Operations**

**Pitfall**: Not applying modulo consistently, especially in the combination calculation:
```python
# Wrong - can cause integer overflow
combinations[i][j] = combinations[i - 1][j] + combinations[i - 1][j - 1]
```

**Correct approach**: Apply modulo after every arithmetic operation:
```python
combinations[i][j] = (combinations[i - 1][j] + combinations[i - 1][j - 1]) % MOD
```

### 5. **Mishandling the Final Answer Calculation**

**Pitfall**: Using wrong indices for the combination lookup:
```python
# Wrong
ans = (ans + f[i][j] * c[n][j]) % mod  # Wrong indices
```

**Correct approach**: Use `c[n-1][j-1]` to properly distribute `j` distinct values across `n` positions:
```python
ans = (ans + f[i][j] * c[n-1][j-1]) % mod
```

