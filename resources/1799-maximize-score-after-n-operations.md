# 1799. Maximize Score After N Operations

> Source: https://algo.monster/liteproblems/1799

## Problem Statement

You have an array `nums` containing `2 * n` positive integers. You need to perform exactly `n` operations on this array to maximize your total score.

For each operation `i` (where operations are numbered starting from 1):
- Select any two elements `x` and `y` from the remaining elements in `nums`
- Calculate the score for this operation as `i * gcd(x, y)`, where `gcd(x, y)` is the greatest common divisor of `x` and `y`
- Remove both `x` and `y` from the array
- Add the calculated score to your total score

Since you perform `n` operations and remove 2 elements each time, all elements will be removed from the array by the end.

The goal is to find the maximum possible total score you can achieve by choosing the optimal pairing of elements and the order in which you perform the operations.

For example, if `nums = [1, 2, 3, 4]`, you need to perform 2 operations:
- You might choose to pair (3, 4) in operation 1, getting score `1 * gcd(3, 4) = 1 * 1 = 1`
- Then pair (1, 2) in operation 2, getting score `2 * gcd(1, 2) = 2 * 1 = 2`
- Total score would be `1 + 2 = 3`

The challenge is to determine which elements to pair together and in what order to maximize the total score, considering that later operations have higher multipliers but you have fewer choices of remaining elements.

## Intuition

When we first look at this problem, the [backtracking](/problems/backtracking) approach tells us we need to try all possible ways to pair elements and order operations. But naive backtracking would be inefficient because we'd repeatedly solve the same subproblems.

Let's think about what really matters in this problem. After performing some operations, what do we need to know to make optimal decisions for the remaining operations? We need:
1. Which elements have already been used (removed from the array)
2. How many operations we've performed so far (to know the multiplier for the next operation)

The key insight is that once we know which elements are still available, the order in which we previously removed elements doesn't matter - only the current state matters. This suggests we can use [dynamic programming](/problems/dynamic_programming_intro) with states.

We can represent "which elements have been used" as a [bitmask](/problems/bitmask_intro). If we have `m` elements, we need `m` bits where bit `i` being 1 means element at index `i` is still available. With `m` elements, we have `2^m` possible states.

For each state `k`, we can determine how many operations have been performed by counting how many pairs have been removed. If `cnt` bits are set to 1 in state `k`, it means `cnt` elements are still available, so we've performed `(m - cnt) / 2` operations. The next operation would be operation number `(m - cnt) / 2 + 1`.

Wait, there's a cleaner way to think about this: instead of tracking which elements are available, let's track which elements have been used. If `cnt` bits are set to 1 in state `k`, and `cnt` is even, then we've performed `cnt / 2` operations. The current operation number is `cnt / 2`.

For each valid state (even number of elements used), we try all possible pairs of used elements `(i, j)` and calculate the score we'd get: `operation_number * gcd(nums[i], nums[j])`. We update our DP table with the maximum score possible for reaching this state.

Since calculating GCD is expensive and we'll need it multiple times, we precompute `g[i][j] = gcd(nums[i], nums[j])` for all pairs.

The answer will be `f[2^m - 1]`, representing the maximum score when all elements have been used (all bits set to 1).

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro), [Backtracking](/problems/backtracking) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The solution uses state compression with [dynamic programming](/problems/dynamic_programming_intro) to efficiently explore all possible pairing combinations.

**Step 1: Precompute GCD Values**

First, we precompute the GCD for all pairs of elements to avoid redundant calculations:
```python
g = [[0] * m for _ in range(m)]
for i in range(m):
    for j in range(i + 1, m):
        g[i][j] = gcd(nums[i], nums[j])
```
This creates a 2D array `g` where `g[i][j]` stores `gcd(nums[i], nums[j])`.

**Step 2: Initialize DP Array**

We create a DP array `f` of size `2^m` where `m` is the length of `nums`:
```python
f = [0] * (1 << m)
```
Here, `f[k]` represents the maximum score achievable when the state is `k`. The state `k` is a [bitmask](/problems/bitmask_intro) where bit `i` being 1 means element at index `i` has been used.

**Step 3: State Transition**

We iterate through all possible states from 0 to `2^m - 1`:
```python
for k in range(1 << m):
    if (cnt := k.bit_count()) % 2 == 0:
```

For each state `k`, we first check if the number of used elements (`cnt`) is even. This is necessary because we can only form complete pairs.

**Step 4: Try All Valid Pairs**

For each valid state, we enumerate all pairs of elements that are marked as "used" in this state:
```python
for i in range(m):
    if k >> i & 1:  # Check if bit i is set
        for j in range(i + 1, m):
            if k >> j & 1:  # Check if bit j is set
```

For each valid pair `(i, j)`, we calculate:
- The previous state: `k ^ (1 << i) ^ (1 << j)` (state without elements i and j)
- The operation number: `cnt // 2` (since cnt elements are used, we've done cnt/2 operations)
- The score for this operation: `(cnt // 2) * g[i][j]`

We update the DP value:
```python
f[k] = max(f[k], f[k ^ (1 << i) ^ (1 << j)] + cnt // 2 * g[i][j])
```

This means: the maximum score for state `k` is either its current value or the score from the previous state plus the score gained from pairing elements `i` and `j`.

**Step 5: Return the Answer**

The final answer is `f[-1]` or `f[2^m - 1]`, which represents the maximum score when all elements have been used (all bits are set to 1).

The time complexity is `O(2^m * m^2)` where `m = 2n` is the size of the input array. The space complexity is `O(2^m + m^2)` for the DP array and GCD preprocessing table.

## Example Walkthrough

Let's walk through the solution with `nums = [1, 2, 3, 4]` (so `m = 4`, `n = 2`).

**Step 1: Precompute GCD values**
```
g[0][1] = gcd(1, 2) = 1
g[0][2] = gcd(1, 3) = 1  
g[0][3] = gcd(1, 4) = 1
g[1][2] = gcd(2, 3) = 1
g[1][3] = gcd(2, 4) = 2
g[2][3] = gcd(3, 4) = 1
```

**Step 2: Initialize DP array**
```
f = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
```
We have 16 states (2^4) representing all possible combinations of used elements.

**Step 3: Process states with even number of elements used**

Let's trace through key states:

**State k = 3 (binary: 0011)** - elements 0 and 1 are used
- cnt = 2 (even), operation number = 2/2 = 1
- Valid pair: (0, 1)
- Previous state: 3 ^ (1<<0) ^ (1<<1) = 0
- Score: f[0] + 1 * g[0][1] = 0 + 1 * 1 = 1
- Update: f[3] = 1

**State k = 5 (binary: 0101)** - elements 0 and 2 are used
- cnt = 2, operation number = 1
- Valid pair: (0, 2)
- Previous state: 0
- Score: f[0] + 1 * g[0][2] = 0 + 1 * 1 = 1
- Update: f[5] = 1

**State k = 12 (binary: 1100)** - elements 2 and 3 are used
- cnt = 2, operation number = 1
- Valid pair: (2, 3)
- Previous state: 0
- Score: f[0] + 1 * g[2][3] = 0 + 1 * 1 = 1
- Update: f[12] = 1

**State k = 15 (binary: 1111)** - all elements used
- cnt = 4, operation number = 4/2 = 2
- We try all pairs from the used elements:

  Pair (0, 1) with remaining (2, 3):
  - Previous state: 15 ^ (1<<0) ^ (1<<1) = 12
  - Score: f[12] + 2 * g[0][1] = 1 + 2 * 1 = 3

  Pair (0, 2) with remaining (1, 3):
  - Previous state: 15 ^ (1<<0) ^ (1<<2) = 10 (binary: 1010)
  - f[10] was computed as 2 (pairing elements 1,3 first)
  - Score: f[10] + 2 * g[0][2] = 2 + 2 * 1 = 4

  Pair (1, 3) with remaining (0, 2):
  - Previous state: 15 ^ (1<<1) ^ (1<<3) = 5
  - Score: f[5] + 2 * g[1][3] = 1 + 2 * 2 = 5

  Maximum: f[15] = 5

**Final Answer: f[15] = 5**

This corresponds to:
- Operation 1: Pair (0, 2) with score 1 * gcd(1, 3) = 1
- Operation 2: Pair (1, 3) with score 2 * gcd(2, 4) = 4
- Total: 1 + 4 = 5

## Python Solution

```python
class Solution:
    def maxScore(self, nums: List[int]) -> int:
        """
        Calculate the maximum score by pairing all numbers in nums.
        Score for each pair is the operation_number * gcd(num1, num2).
      
        Args:
            nums: List of integers to be paired
          
        Returns:
            Maximum possible score from pairing all numbers
        """
        from math import gcd
        from typing import List
      
        n = len(nums)
      
        # dp[mask] stores the maximum score achievable using the subset of numbers
        # represented by the bitmask
        dp = [0] * (1 << n)
      
        # Precompute GCD values for all pairs to avoid redundant calculations
        # gcd_matrix[i][j] stores gcd(nums[i], nums[j]) for i < j
        gcd_matrix = [[0] * n for _ in range(n)]
        for i in range(n):
            for j in range(i + 1, n):
                gcd_matrix[i][j] = gcd(nums[i], nums[j])
      
        # Iterate through all possible states (subsets of numbers)
        for mask in range(1 << n):
            # Count how many numbers are selected in current mask
            selected_count = mask.bit_count()
          
            # We can only form pairs with even number of elements
            if selected_count % 2 == 0:
                # Try all possible pairs within the selected subset
                for i in range(n):
                    # Check if index i is selected in current mask
                    if mask >> i & 1:
                        for j in range(i + 1, n):
                            # Check if index j is selected in current mask
                            if mask >> j & 1:
                                # Previous state: current mask without indices i and j
                                prev_mask = mask ^ (1 << i) ^ (1 << j)
                              
                                # Calculate operation number (which pair this is)
                                operation_num = selected_count // 2
                              
                                # Update dp with maximum score
                                dp[mask] = max(
                                    dp[mask],
                                    dp[prev_mask] + operation_num * gcd_matrix[i][j]
                                )
      
        # Return the maximum score when all numbers are used (all bits set)
        return dp[-1]
```

## Time and Space Complexity

**Time Complexity:** `O(2^m × m^2)`

The algorithm uses dynamic programming with bitmask states. The main loop iterates through all possible states `k` from `0` to `2^m - 1`, which gives us `O(2^m)` iterations. For each state `k` where the bit count is even, we have two nested loops iterating through indices `i` and `j` where `i < j`, resulting in `O(m^2)` operations per state. Therefore, the overall time complexity is `O(2^m × m^2)`.

Additionally, the preprocessing step to compute all pairwise GCDs takes `O(m^2)` time, but this is dominated by the main computation.

**Space Complexity:** `O(2^m + m^2)`

The space is used for:
- Array `f` of size `2^m` to store the maximum score for each state: `O(2^m)`
- 2D array `g` of size `m × m` to store precomputed GCD values: `O(m^2)`

Since `2^m` grows exponentially and will dominate `m^2` for larger values of `m`, the space complexity can be simplified to `O(2^m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Operation Number Calculation**

One of the most common mistakes is incorrectly calculating which operation number we're currently on. The operation number directly affects the score multiplier, so getting this wrong will produce incorrect results.

**The Pitfall:**
```python
# WRONG: Using the number of remaining elements
operation_num = (n - selected_count) // 2

# WRONG: Starting from 0 instead of 1
operation_num = (selected_count // 2) - 1
```

**Why it happens:** 
- Confusion about whether operations are 0-indexed or 1-indexed (they're 1-indexed)
- Mixing up the count of selected elements vs remaining elements
- Not understanding that when we have selected `k` elements, we're computing the score for the `(k/2)`-th operation

**The Correct Approach:**
```python
# When selected_count elements are in the current state,
# we're calculating the score for operation number (selected_count // 2)
operation_num = selected_count // 2
```

**Example to illustrate:**
- When mask has 2 bits set (2 elements selected), it's operation 1 (score multiplier = 1)
- When mask has 4 bits set (4 elements selected), it's operation 2 (score multiplier = 2)
- When mask has 6 bits set (6 elements selected), it's operation 3 (score multiplier = 3)

### 2. **State Transition Direction Confusion**

Another subtle pitfall is building the DP in the wrong direction or misunderstanding what each state represents.

**The Pitfall:**
```python
# WRONG: Trying to build from full set to empty
for mask in range((1 << n) - 1, -1, -1):
    # Remove pairs from the mask...
  
# WRONG: Adding pairs to previous state instead of considering current state
next_mask = prev_mask | (1 << i) | (1 << j)
dp[next_mask] = max(dp[next_mask], dp[prev_mask] + score)
```

**Why it happens:**
- Confusion about whether we're building up states (adding elements) or breaking down states (removing elements from consideration)
- Not clearly understanding that `dp[mask]` represents the maximum score when exactly those bits/elements in `mask` have been selected and paired

**The Correct Approach:**
```python
# Build up from empty set to full set
for mask in range(1 << n):
    if mask.bit_count() % 2 == 0:  # Only even counts make sense
        # For current mask, try all pairs (i,j) that are IN this mask
        # Look back at the state WITHOUT these two elements
        prev_mask = mask ^ (1 << i) ^ (1 << j)
        dp[mask] = max(dp[mask], dp[prev_mask] + operation_num * gcd_matrix[i][j])
```

### 3. **GCD Matrix Bounds and Symmetry Issues**

**The Pitfall:**
```python
# WRONG: Creating full symmetric matrix unnecessarily
gcd_matrix = [[0] * n for _ in range(n)]
for i in range(n):
    for j in range(n):  # Calculating for all i,j
        if i != j:
            gcd_matrix[i][j] = gcd(nums[i], nums[j])

# WRONG: Accessing wrong indices later
score = gcd_matrix[j][i]  # Might be 0 if i < j
```

**The Correct Approach:**
```python
# Only calculate for i < j to save space and time
gcd_matrix = [[0] * n for _ in range(n)]
for i in range(n):
    for j in range(i + 1, n):
        gcd_matrix[i][j] = gcd(nums[i], nums[j])

# Always access with smaller index first
score = gcd_matrix[min(i,j)][max(i,j)]
# Or ensure i < j when iterating
```

### Prevention Tips:
1. **Test with small examples:** Walk through the algorithm with n=2 (4 elements) manually
2. **Add assertions:** Check that operation numbers range from 1 to n
3. **Validate state transitions:** Ensure prev_mask always has 2 fewer bits set than current mask
4. **Print intermediate values:** During debugging, print the operation number and selected elements for each state transition

