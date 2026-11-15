# 996. Number of Squareful Arrays

> Source: https://algo.monster/liteproblems/996

## Problem Statement

You are given an integer array `nums`. A permutation of this array is called **squareful** if for every pair of adjacent elements in the permutation, their sum is a perfect square.

A perfect square is a number that can be expressed as the product of an integer with itself. For example, 1, 4, 9, 16, 25 are perfect squares because they equal 1×1, 2×2, 3×3, 4×4, and 5×5 respectively.

Your task is to find how many different permutations of `nums` are squareful.

Two permutations are considered different if there exists at least one position where the elements differ. For example, `[1, 8, 17]` and `[17, 8, 1]` are different permutations.

**Example:**
- If `nums = [1, 17, 8]`, one squareful permutation would be `[1, 8, 17]` because:
  - 1 + 8 = 9 = 3²
  - 8 + 17 = 25 = 5²

The function should return the total count of all valid squareful permutations.

## Intuition

While [backtracking](/problems/backtracking) would work for this problem, we can observe that the solution actually uses a more efficient approach - [dynamic programming](/problems/dynamic_programming_intro) with bitmasks. Let's understand why this transformation happens.

Initially, we might think of using [backtracking](/problems/backtracking) to generate all permutations and check each one. However, we can make a key observation: this is essentially a path-finding problem where we want to visit all numbers exactly once, and adjacent numbers in our path must sum to a perfect square.

The insight is that we can use **state compression with [dynamic programming](/problems/dynamic_programming_intro)**. Instead of generating permutations explicitly, we can think of it as: "How many ways can we arrange a subset of numbers, ending at a specific number?"

Let's define `f[mask][j]` where:
- `mask` represents which numbers we've used (using bits - if bit `i` is 1, we've used `nums[i]`)
- `j` is the index of the last number in our current arrangement
- The value is the count of valid arrangements

The base case is simple: `f[1 << j][j] = 1` for each position `j`, meaning we can start with any single number.

For the transition, we build up from smaller masks to larger ones. For a given state `f[mask][j]`, we can extend it by adding another unused number `k` if:
1. Number at index `k` hasn't been used yet (bit `k` is 0 in the previous mask)
2. `nums[j] + nums[k]` is a perfect square

This way, `f[mask | (1 << k)][k] += f[mask][j]` accumulates all ways to reach the new state.

The final answer would be the sum of `f[(1 << n) - 1][j]` for all ending positions `j`, which represents all ways to use all numbers.

One crucial detail: since the original array might have duplicates, we need to divide by the factorial of each duplicate count to avoid counting the same permutation multiple times. For example, if we have two 1's, swapping them doesn't create a different permutation, so we divide by `2!`.

This DP approach is more efficient than [backtracking](/problems/backtracking) because it avoids regenerating the same subproblems - we compute each state only once rather than exploring the same partial permutations multiple times through different paths.

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro), [Backtracking](/problems/backtracking) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The snippets below use short names: `f` is `dp`, `i` is `mask`, `j` is `last_idx`, and `k` is `prev_idx` in the full solution.

Let's implement the [dynamic programming](/problems/dynamic_programming_intro) solution with [bitmask](/problems/bitmask_intro) state compression step by step:

**1. Initialize the DP table:**
```python
n = len(nums)
f = [[0] * n for _ in range(1 << n)]
```
We create a 2D array `f` where `f[mask][j]` represents the number of ways to arrange the numbers indicated by `mask`, with the arrangement ending at position `j`.

**2. Set base cases:**
```python
for j in range(n):
    f[1 << j][j] = 1
```
Each single number can be the start of a valid arrangement, so we set `f[1 << j][j] = 1` for all positions. The mask `1 << j` has only the `j`-th bit set, meaning only `nums[j]` is used.

**3. Build up the DP states:**
```python
for i in range(1 << n):
    for j in range(n):
        if i >> j & 1:  # Check if j-th bit is set in mask i
            for k in range(n):
                if (i >> k & 1) and k != j:  # k is used and k != j
                    s = nums[j] + nums[k]
                    t = int(sqrt(s))
                    if t * t == s:  # Check if sum is perfect square
                        f[i][j] += f[i ^ (1 << j)][k]
```

The algorithm iterates through all possible masks `i` (subsets of numbers). For each mask and ending position `j`:
- We check if position `j` is included in the current mask (`i >> j & 1`)
- We look for a previous position `k` that could have been the last element before `j`
- The previous mask would be `i ^ (1 << j)` (current mask without position `j`)
- We verify that `nums[j] + nums[k]` forms a perfect square
- If valid, we add the count from the previous state to the current state

**4. Calculate the final answer:**
```python
ans = sum(f[(1 << n) - 1][j] for j in range(n))
```
The mask `(1 << n) - 1` represents all bits set (all numbers used). We sum up the counts for all possible ending positions.

**5. Handle duplicates:**
```python
for v in Counter(nums).values():
    ans //= factorial(v)
```
Since duplicate numbers in different positions create identical permutations, we need to divide by the factorial of each duplicate count. For example, if a number appears 3 times, those 3 positions can be arranged in `3! = 6` ways, but they all represent the same permutation.

**Time Complexity:** `O(2^n × n^2)` where `n` is the length of the array. We iterate through `2^n` masks, and for each mask and position, we check `n` previous positions.

**Space Complexity:** `O(2^n × n)` for the DP table.

This approach efficiently counts all valid squareful permutations by building them incrementally using [dynamic programming](/problems/dynamic_programming_intro), avoiding the redundant computation that would occur with pure [backtracking](/problems/backtracking).

## Example Walkthrough

Let's walk through the solution with `nums = [2, 2, 7]`.

**Step 1: Setup and Initialization**
- We have n = 3 numbers
- Create DP table `f[mask][j]` where mask uses 3 bits (000 to 111) and j ranges from 0 to 2
- Initialize base cases: `f[001][0] = 1`, `f[010][1] = 1`, `f[100][2] = 1`
  - These represent starting with just nums[0]=2, nums[1]=2, or nums[2]=7 respectively

**Step 2: Check Perfect Square Sums**
Let's identify which pairs sum to perfect squares:
- nums[0] + nums[1] = 2 + 2 = 4 = 2² ✓
- nums[0] + nums[2] = 2 + 7 = 9 = 3² ✓  
- nums[1] + nums[2] = 2 + 7 = 9 = 3² ✓

**Step 3: Build DP States**

For mask = 011 (using positions 0 and 1, both containing 2):
- Ending at position 0: Can we reach here from position 1?
  - Previous mask would be 010 (just position 1)
  - Check: nums[0] + nums[1] = 4 is perfect square ✓
  - So `f[011][0] = f[010][1] = 1`
- Ending at position 1: Can we reach here from position 0?
  - Previous mask would be 001 (just position 0)
  - Check: nums[1] + nums[0] = 4 is perfect square ✓
  - So `f[011][1] = f[001][0] = 1`

For mask = 101 (using positions 0 and 2, values 2 and 7):
- Ending at position 0: Can we reach here from position 2?
  - Previous mask would be 100 (just position 2)
  - Check: nums[0] + nums[2] = 9 is perfect square ✓
  - So `f[101][0] = f[100][2] = 1`
- Ending at position 2: Can we reach here from position 0?
  - Previous mask would be 001 (just position 0)
  - Check: nums[2] + nums[0] = 9 is perfect square ✓
  - So `f[101][2] = f[001][0] = 1`

For mask = 110 (using positions 1 and 2, values 2 and 7):
- Similar analysis gives us `f[110][1] = 1` and `f[110][2] = 1`

For mask = 111 (using all three numbers):
- Ending at position 0 (value 2):
  - From position 1 with mask 110: nums[0] + nums[1] = 4 ✓
    - `f[111][0] += f[110][1] = 1`
  - From position 2 with mask 110: nums[0] + nums[2] = 9 ✓
    - `f[111][0] += f[110][2] = 1`
  - Total: `f[111][0] = 2`
- Ending at position 1 (value 2):
  - From position 0 with mask 101: nums[1] + nums[0] = 4 ✓
    - `f[111][1] += f[101][0] = 1`
  - From position 2 with mask 101: nums[1] + nums[2] = 9 ✓
    - `f[111][1] += f[101][2] = 1`
  - Total: `f[111][1] = 2`
- Ending at position 2 (value 7):
  - From position 0 with mask 011: nums[2] + nums[0] = 9 ✓
    - `f[111][2] += f[011][0] = 1`
  - From position 1 with mask 011: nums[2] + nums[1] = 9 ✓
    - `f[111][2] += f[011][1] = 1`
  - Total: `f[111][2] = 2`

**Step 4: Calculate Raw Answer**
Sum all ways to use all numbers: `f[111][0] + f[111][1] + f[111][2] = 2 + 2 + 2 = 6`

**Step 5: Handle Duplicates**
Since we have two 2's (positions 0 and 1), we need to divide by 2! = 2:
- Final answer = 6 / 2 = **3**

The three distinct squareful permutations are:
1. [2, 2, 7]: 2+2=4 (2²), 2+7=9 (3²)
2. [2, 7, 2]: 2+7=9 (3²), 7+2=9 (3²)
3. [7, 2, 2]: 7+2=9 (3²), 2+2=4 (2²)

Each of these appears twice in the raw count of 6, once for each way to assign the two 2's to positions 0 and 1, which is why we divide by 2!.

## Python Solution

```python
from typing import List
from math import sqrt, factorial
from collections import Counter

class Solution:
    def numSquarefulPerms(self, nums: List[int]) -> int:
        """
        Count the number of permutations of nums that are squareful.
        A permutation is squareful if every pair of adjacent elements sums to a perfect square.
      
        Uses dynamic programming with bitmask to track which numbers have been used,
        and handles duplicate numbers by dividing by factorial of their frequencies.
        """
        n = len(nums)
      
        # dp[mask][last_idx] = number of ways to arrange numbers in 'mask' ending at index 'last_idx'
        # mask: bitmask representing which indices have been used (1 = used, 0 = unused)
        # last_idx: the index of the last number in the current arrangement
        dp = [[0] * n for _ in range(1 << n)]
      
        # Base case: single element arrangements
        # Each number by itself is a valid arrangement of length 1
        for idx in range(n):
            dp[1 << idx][idx] = 1
      
        # Iterate through all possible subsets of indices (represented by bitmask)
        for mask in range(1 << n):
            # Try each possible last element in the current arrangement
            for last_idx in range(n):
                # Check if last_idx is included in the current mask
                if mask >> last_idx & 1:
                    # Try to extend from each possible previous element
                    for prev_idx in range(n):
                        # Check if prev_idx is in mask and is different from last_idx
                        if (mask >> prev_idx & 1) and prev_idx != last_idx:
                            # Check if the sum of the two adjacent numbers is a perfect square
                            sum_value = nums[last_idx] + nums[prev_idx]
                            sqrt_value = int(sqrt(sum_value))
                          
                            if sqrt_value * sqrt_value == sum_value:
                                # If valid, add the number of ways to reach prev_idx
                                # in the state without last_idx
                                prev_mask = mask ^ (1 << last_idx)
                                dp[mask][last_idx] += dp[prev_mask][prev_idx]
      
        # Sum up all valid complete arrangements (using all numbers)
        full_mask = (1 << n) - 1  # All bits set to 1
        total_arrangements = sum(dp[full_mask][last_idx] for last_idx in range(n))
      
        # Adjust for duplicate numbers by dividing by factorial of their frequencies
        # This removes overcounting due to identical elements being treated as distinct
        frequency_map = Counter(nums)
        for frequency in frequency_map.values():
            total_arrangements //= factorial(frequency)
      
        return total_arrangements
```

## Time and Space Complexity

**Time Complexity:** `O(2^n * n^2)`

The algorithm uses dynamic programming with bitmask to track which numbers have been used. The main computation involves:
- Iterating through all possible subsets of numbers: `2^n` states
- For each state, iterating through all possible last positions: `n` positions
- For each position, checking all previous positions that could connect: up to `n` positions
- Checking if two numbers form a perfect square sum: `O(1)` operation

Therefore, the nested loops give us `O(2^n * n * n) = O(2^n * n^2)` time complexity.

The final step of dividing by factorials of duplicate counts takes `O(n)` time, which doesn't affect the overall complexity.

**Space Complexity:** `O(2^n * n)`

The space is dominated by the 2D DP table `f`:
- First dimension represents all possible subsets (bitmasks): `2^n` states
- Second dimension represents the last element position: `n` positions
- Total space for DP table: `O(2^n * n)`

Additional space used includes:
- Counter for duplicate values: `O(n)`
- Variables for computation: `O(1)`

The overall space complexity is `O(2^n * n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Perfect Square Check With Floating Point**
A common worry is that floating-point square roots give wrong answers. For this problem the sums are at most `2 * 10^9`, and in that range both checks below give the same result for every value, so either one works.

**Float comparison:**
```python
import math
if math.sqrt(sum_value) == int(math.sqrt(sum_value)):
    # Process
```

**Integer comparison (used in the solution):**
```python
sqrt_value = int(sqrt(sum_value))
if sqrt_value * sqrt_value == sum_value:
    # Process
```

For much larger integers (above roughly `10^15`), floating-point square roots lose precision. `math.isqrt(sum_value) ** 2 == sum_value` is exact for any size and is the safest choice in general.

### 2. **Forgetting to Handle Duplicate Elements**
The algorithm counts arrangements where duplicate numbers in different positions are treated as distinct, leading to overcounting.

**Pitfall Example:**
```python
# WRONG: Not adjusting for duplicates
def numSquarefulPerms(self, nums: List[int]) -> int:
    # ... DP logic ...
    return sum(dp[full_mask][j] for j in range(n))  # Returns inflated count
```

**Solution:**
```python
# CORRECT: Divide by factorial of duplicate frequencies
frequency_map = Counter(nums)
for frequency in frequency_map.values():
    total_arrangements //= factorial(frequency)
```

### 3. **Incorrect Bitmask Operations**
Misunderstanding how to check or toggle bits in the mask is a frequent error.

**Pitfall Example:**
```python
# WRONG: Incorrect bit checking
if mask & j:  # This checks if mask & j is non-zero, not if j-th bit is set
    # Process

# FRAGILE: Subtraction instead of XOR
prev_mask = mask - (1 << last_idx)  # Only correct when the bit is set (true here, wrong elsewhere)
```

**Solution:**
```python
# CORRECT: Proper bit operations
if mask >> last_idx & 1:  # Check if last_idx-th bit is set
    # Process

prev_mask = mask ^ (1 << last_idx)  # XOR to toggle the bit
```

### 4. **Precomputing a Set of Perfect Squares**
Precomputing perfect squares is a valid alternative. Since sums are at most `2 * 10^9`, the set holds about 44,722 values:

```python
max_sum = 2 * max(nums)
perfect_squares = {i*i for i in range(int(sqrt(max_sum)) + 1)}
if sum_value in perfect_squares:
    # Process
```

This is correct, but building the set costs more than the at most `12 * 11 = 132` distinct pair checks the DP actually needs. Checking each sum on the fly, or caching the result for each pair of indices, is cheaper:

```python
sqrt_value = int(sqrt(sum_value))
if sqrt_value * sqrt_value == sum_value:
    # Process
```

### 5. **Off-by-One Errors in Base Cases**
Setting up incorrect initial states can propagate errors throughout the DP computation.

**Pitfall Example:**
```python
# WRONG: Incorrect base case initialization
dp = [[0] * n for _ in range(1 << n)]
dp[0][0] = 1  # Empty set with ending at index 0 makes no sense
```

**Solution:**
```python
# CORRECT: Each single element is a valid starting arrangement
for idx in range(n):
    dp[1 << idx][idx] = 1  # Single element at position idx
```

