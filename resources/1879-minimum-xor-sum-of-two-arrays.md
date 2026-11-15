# 1879. Minimum XOR Sum of Two Arrays

> Source: https://algo.monster/liteproblems/1879

## Problem Statement

You have two integer arrays `nums1` and `nums2`, both of length `n`.

The **XOR sum** is calculated by pairing each element from `nums1` with exactly one element from `nums2` and computing: `(nums1[0] XOR nums2[0]) + (nums1[1] XOR nums2[1]) + ... + (nums1[n-1] XOR nums2[n-1])`.

Your task is to **rearrange** the elements of `nums2` to create a new pairing that produces the **minimum possible XOR sum**.

For example, given `nums1 = [1,2,3]` and `nums2 = [3,2,1]`:
- The current pairing gives: `(1 XOR 3) + (2 XOR 2) + (3 XOR 1) = 2 + 0 + 2 = 4`
- But you could rearrange `nums2` to find a better pairing that gives a smaller sum

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with [bitmask](/problems/bitmask_intro) to track which elements from `nums2` have been used. The state `f[i][j]` represents the minimum XOR sum when considering the first `i` elements of `nums1` and the set of used elements from `nums2` represented by bitmask `j`. For each element in `nums1`, we try pairing it with each unused element in `nums2` and track the minimum sum.

The final answer is stored in `f[-1][-1]`, which represents using all elements from both arrays.

## Intuition

The key insight is that this is an assignment problem - we need to find the optimal one-to-one pairing between elements of `nums1` and `nums2` to minimize the total XOR sum.

Since we need to pair each element in `nums1` with exactly one element in `nums2`, and each element in `nums2` can only be used once, this is essentially finding the best permutation of `nums2` to match with the fixed order of `nums1`.

A brute force approach would try all `n!` permutations of `nums2`, which becomes impractical for larger arrays. Instead, we can think of this as a step-by-step decision process: for each element in `nums1`, we choose which available element from `nums2` to pair it with.

This naturally leads to [dynamic programming](/problems/dynamic_programming_intro) with [bitmask](/problems/bitmask_intro). The bitmask helps us track which elements from `nums2` have already been used in previous pairings. For a bitmask `j`, if the `k-th` bit is set to 1, it means `nums2[k]` has been used.

The recurrence relation emerges from considering: if we're at position `i` in `nums1` and have already used certain elements from `nums2` (represented by [bitmask](/problems/bitmask_intro) `j`), what's the minimum XOR sum we can achieve? We try pairing `nums1[i-1]` with each unused element from `nums2` and take the minimum.

The state transition is: `f[i][j] = min(f[i-1][j without k-th bit] + (nums1[i-1] XOR nums2[k]))` for all `k` where the `k-th` bit in `j` is set.

This approach efficiently explores all valid pairings while avoiding redundant calculations through memoization, reducing the complexity from `O(n!)` to `O(n² × 2ⁿ)`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The implementation uses [dynamic programming](/problems/dynamic_programming_intro) with [bitmask](/problems/bitmask_intro) to solve this assignment problem efficiently.

**Data Structure Setup:**
- Create a 2D DP table `f` with dimensions `(n+1) × (1 << n)` where `n` is the length of the arrays
- `f[i][j]` represents the minimum XOR sum when we've processed the first `i` elements of `nums1` and used the elements from `nums2` indicated by [bitmask](/problems/bitmask_intro) `j`
- Initialize all values to infinity (`inf`) except `f[0][0] = 0` (base case: no elements processed, no elements used)

**Algorithm Implementation:**

1. **Outer Loop - Process each element in nums1:**
   ```python
   for i, x in enumerate(nums1, 1):
   ```
   - Iterate through `nums1` with 1-based indexing for the DP table
   - `x` is the current element from `nums1` to be paired

2. **Middle Loop - Iterate through all possible bitmasks:**
   ```python
   for j in range(1 << n):
   ```
   - Each [bitmask](/problems/bitmask_intro) `j` represents a subset of used elements from `nums2`
   - For `n` elements, there are `2^n` possible subsets

3. **Inner Loop - Try pairing with each used element:**
   ```python
   for k in range(n):
       if j >> k & 1:
   ```
   - Check if the `k-th` bit is set in [bitmask](/problems/bitmask_intro) `j` using bit manipulation `j >> k & 1`
   - If set, it means `nums2[k]` is part of the current subset being considered

4. **State Transition:**
   ```python
   f[i][j] = min(f[i][j], f[i - 1][j ^ (1 << k)] + (x ^ nums2[k]))
   ```
   - `j ^ (1 << k)` removes the `k-th` bit from `j`, representing the state before using `nums2[k]`
   - Add the XOR value `(x ^ nums2[k])` to the previous state's minimum sum
   - Take the minimum across all possible pairings for position `i`

5. **Return the Result:**
   ```python
   return f[-1][-1]
   ```
   - `f[-1][-1]` or `f[n][(1 << n) - 1]` contains the minimum XOR sum when all `n` elements from `nums1` are processed and all elements from `nums2` are used
   - The [bitmask](/problems/bitmask_intro) `(1 << n) - 1` has all `n` bits set, representing that all elements are used

The algorithm systematically builds up the solution by considering all valid ways to assign elements from `nums2` to elements in `nums1`, ensuring each element is used exactly once while tracking the minimum possible XOR sum.

## Example Walkthrough

Let's trace through a small example with `nums1 = [1, 2]` and `nums2 = [3, 4]`.

**Setup:**
- n = 2
- Create DP table `f` with dimensions 3 × 4 (since we have 2 elements and 2^2 = 4 possible bitmasks)
- Initialize `f[0][0] = 0`, all others to infinity

**Bitmask representations:**
- `00` (0): no elements from nums2 used
- `01` (1): nums2[0] = 3 used
- `10` (2): nums2[1] = 4 used  
- `11` (3): both nums2[0] and nums2[1] used

**Step 1: Process nums1[0] = 1 (i = 1)**

For bitmask `01` (j = 1, using nums2[0] = 3):
- k = 0: bit is set, so we can pair nums1[0] with nums2[0]
- Previous state: `f[0][0]` (no elements used before)
- `f[1][1] = f[0][0] + (1 XOR 3) = 0 + 2 = 2`

For bitmask `10` (j = 2, using nums2[1] = 4):
- k = 1: bit is set, so we can pair nums1[0] with nums2[1]
- Previous state: `f[0][0]`
- `f[1][2] = f[0][0] + (1 XOR 4) = 0 + 5 = 5`

For bitmask `11` (j = 3, using both elements):
- k = 0: `f[1][3] = min(inf, f[0][2] + (1 XOR 3))` = inf (f[0][2] is inf)
- k = 1: `f[1][3] = min(inf, f[0][1] + (1 XOR 4))` = inf (f[0][1] is inf)

**Step 2: Process nums1[1] = 2 (i = 2)**

For bitmask `11` (j = 3, both elements used):
- k = 0: bit is set
  - Previous state: `f[1][2]` (had used nums2[1], now adding nums2[0])
  - `f[2][3] = min(inf, f[1][2] + (2 XOR 3)) = 5 + 1 = 6`
- k = 1: bit is set
  - Previous state: `f[1][1]` (had used nums2[0], now adding nums2[1])
  - `f[2][3] = min(6, f[1][1] + (2 XOR 4)) = min(6, 2 + 6) = 6`

**Result:**
`f[2][3] = 6`, which represents the minimum XOR sum when all elements are paired.

This corresponds to:
- Pairing nums1[0]=1 with nums2[0]=3: 1 XOR 3 = 2
- Pairing nums1[1]=2 with nums2[1]=4: 2 XOR 4 = 6
- Total: 2 + 6 = 8

Or alternatively:
- Pairing nums1[0]=1 with nums2[1]=4: 1 XOR 4 = 5  
- Pairing nums1[1]=2 with nums2[0]=3: 2 XOR 3 = 1
- Total: 5 + 1 = 6 (minimum)

The algorithm correctly finds that rearranging nums2 to [4, 3] gives the minimum XOR sum of 6.

## Python Solution

```python
class Solution:
    def minimumXORSum(self, nums1: List[int], nums2: List[int]) -> int:
        n = len(nums2)
      
        # dp[i][mask] represents the minimum XOR sum when considering 
        # the first i elements from nums1 and using elements from nums2
        # indicated by the bitmask
        dp = [[float('inf')] * (1 << n) for _ in range(n + 1)]
      
        # Base case: no elements from nums1, no elements from nums2 used
        dp[0][0] = 0
      
        # Iterate through each element in nums1
        for i, num1_element in enumerate(nums1, 1):
            # Try all possible masks (subsets of nums2)
            for mask in range(1 << n):
                # Try pairing current nums1[i-1] with each available element in nums2
                for k in range(n):
                    # Check if k-th element of nums2 is used in current mask
                    if (mask >> k) & 1:
                        # Previous mask without k-th element
                        prev_mask = mask ^ (1 << k)
                        # Calculate XOR of current pair
                        xor_value = num1_element ^ nums2[k]
                        # Update minimum XOR sum
                        dp[i][mask] = min(dp[i][mask], 
                                         dp[i - 1][prev_mask] + xor_value)
      
        # Return minimum XOR sum using all elements from both arrays
        # Last row, last column (all bits set in mask)
        return dp[n][(1 << n) - 1]
```

## Time and Space Complexity

**Time Complexity:** `O(n² * 2^n)`

The algorithm uses dynamic programming with bitmask. Breaking down the nested loops:
- The outer loop iterates through `nums1`, which runs `n` times
- The middle loop iterates through all possible bitmask states, which is `2^n` iterations
- The inner loop iterates through each bit position to check which elements of `nums2` are used, which runs `n` times

Therefore, the total time complexity is `O(n * 2^n * n) = O(n² * 2^n)`

**Space Complexity:** `O(n * 2^n)`

The space is dominated by the 2D DP array `f`:
- The first dimension has size `n + 1` (for each element in `nums1` plus initial state)
- The second dimension has size `2^n` (for all possible bitmask states representing which elements of `nums2` have been used)

Thus, the space complexity is `O((n + 1) * 2^n) = O(n * 2^n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Bitmask Interpretation**
A common mistake is misunderstanding what the bitmask represents. Developers often confuse whether bit `i` being set means element `i` has been used or is available to use.

**Wrong interpretation:**
```python
# Incorrectly checking if element is available (not used)
if not (mask >> k) & 1:  # Wrong!
    prev_mask = mask | (1 << k)  # Adding instead of removing
```

**Correct approach:**
```python
# Bit set means element IS used in current state
if (mask >> k) & 1:
    prev_mask = mask ^ (1 << k)  # Remove k-th bit to get previous state
```

### 2. **Inefficient State Iteration Order**
Processing states in the wrong order can lead to accessing uninitialized values or incorrect results.

**Problematic approach:**
```python
# Wrong: iterating through masks that don't match the number of elements processed
for i in range(1, n + 1):
    for mask in range(1 << n):
        if bin(mask).count('1') != i:  # Unnecessary check, wastes time
            continue
```

**Optimized solution:**
```python
# Better: only iterate through masks with exactly i bits set
for i in range(1, n + 1):
    for mask in range(1 << n):
        if bin(mask).count('1') == i:  # Only process relevant masks
            # Process state
```

### 3. **Memory Overflow with Large Arrays**
The space complexity is O(n × 2^n), which becomes problematic for large n (typically n > 20).

**Memory-intensive approach:**
```python
dp = [[float('inf')] * (1 << n) for _ in range(n + 1)]  # Uses O(n × 2^n) space
```

**Space-optimized solution using rolling array:**
```python
# Only keep current and previous row
prev = [float('inf')] * (1 << n)
prev[0] = 0

for i, num1_element in enumerate(nums1):
    curr = [float('inf')] * (1 << n)
    for mask in range(1 << n):
        if bin(mask).count('1') == i + 1:
            for k in range(n):
                if (mask >> k) & 1:
                    prev_mask = mask ^ (1 << k)
                    curr[mask] = min(curr[mask], 
                                    prev[prev_mask] + (num1_element ^ nums2[k]))
    prev = curr

return prev[(1 << n) - 1]
```

### 4. **Bit Operation Precedence Errors**
Forgetting operator precedence can lead to incorrect bit manipulation.

**Common mistake:**
```python
# Wrong: & has lower precedence than >>
if mask >> k & 1:  # This works but can be confusing
  
# Clearer with parentheses:
if (mask >> k) & 1:  # More explicit about operation order

# Another common error:
prev_mask = mask - 1 << k  # Wrong! Subtraction happens before shift
prev_mask = mask ^ (1 << k)  # Correct: use XOR to toggle bit
```

### 5. **Off-by-One Errors in Indexing**
Mixing 0-based and 1-based indexing can cause subtle bugs.

**Error-prone code:**
```python
for i in range(n):  # 0-based
    for mask in range(1 << n):
        # Accessing dp[i+1] but nums1[i]
        dp[i+1][mask] = min(dp[i+1][mask], 
                           dp[i][prev_mask] + (nums1[i+1] ^ nums2[k]))  # Wrong index!
```

**Consistent indexing:**
```python
# Use enumerate with start=1 for clarity
for i, num1_element in enumerate(nums1, 1):
    # Now i matches dp table indexing
    # num1_element is the actual value, no index confusion
    dp[i][mask] = min(dp[i][mask], 
                     dp[i-1][prev_mask] + (num1_element ^ nums2[k]))
```

