# 3312. Sorted GCD Pair Queries

> Source: https://algo.monster/liteproblems/3312

## Problem Statement

You have an integer array `nums` of length `n` and an integer array `queries`.

The problem asks you to:

1. **Generate all possible GCD pairs**: Calculate the GCD (Greatest Common Divisor) for every possible pair `(nums[i], nums[j])` where `0 <= i < j < n`. This means you need to find the GCD for all combinations of two different elements from the array.

2. **Create a sorted GCD array**: Take all these GCD values and sort them in ascending order. This sorted array is called `gcdPairs`.

3. **Answer queries**: For each value `queries[i]`, return the element at index `queries[i]` in the `gcdPairs` array.

The function should return an array `answer` where `answer[i]` contains the value at position `queries[i]` in the sorted `gcdPairs` array.

**Example to illustrate:**
- If `nums = [2, 4, 6]`, the pairs would be: `(2,4)`, `(2,6)`, `(4,6)`
- The GCDs would be: `gcd(2,4) = 2`, `gcd(2,6) = 2`, `gcd(4,6) = 2`
- So `gcdPairs = [2, 2, 2]` after sorting
- If `queries = [0, 1, 2]`, the answer would be `[2, 2, 2]`

The key challenge is efficiently computing and storing the GCD values for all pairs, especially when dealing with large arrays, since the number of pairs grows quadratically with the size of the input array.

## Intuition

The naive approach would be to compute the GCD for all pairs, sort them, and answer queries directly. However, with `n` elements, we'd have `n*(n-1)/2` pairs, which could be extremely large and inefficient.

The key insight is to think about the problem from a different angle: instead of computing individual GCDs, we can count **how many pairs have each possible GCD value**.

Why is this helpful? Consider that:
1. The GCD of any pair must be a divisor of both numbers in the pair
2. The GCD values are limited by the maximum value in the array (at most `max(nums)` different values)
3. If we know the count of pairs for each GCD value, we can build the sorted `gcdPairs` array conceptually without storing all individual values

Here's the clever observation: if we want to count pairs with GCD equal to `g`, we need to:
- First, count all pairs where both numbers are divisible by `g` (these pairs have GCD that's at least `g`)
- Then subtract pairs whose GCD is actually `2g, 3g, 4g, ...` (multiples of `g`)

For example, if we want pairs with GCD = 2:
- Count all pairs where both numbers are divisible by 2
- Subtract pairs with GCD = 4, 6, 8, ... (which we've already computed)

This suggests a bottom-up approach: start from the largest possible GCD and work downwards. For each value `i`:
1. Count how many numbers in `nums` are multiples of `i`. If there are `v` such numbers, they can form `v*(v-1)/2` pairs
2. These pairs have GCD that's a multiple of `i`, but some might have larger GCDs
3. Subtract the counts of pairs with GCD equal to multiples of `i` (which we've already calculated)

Once we have counts for each GCD value, we can use prefix sums to quickly determine which GCD value corresponds to each query index, using [binary search](/problems/binary_search_intro) to find the answer efficiently.

This transforms a potentially quadratic problem into one that's linear in the range of values, making it much more efficient.

**Learn more about [Math](/problems/math-basics), [Binary Search](/problems/binary_search_intro), [Combinatorics](/problems/backtracking) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**Step 1: Initialize data structures**
- Find `mx = max(nums)` - the maximum value in the array
- Create `cnt = Counter(nums)` to store the frequency of each number
- Initialize `cnt_g = [0] * (mx + 1)` to store the count of pairs with each GCD value

**Step 2: Calculate GCD counts (traverse from large to small)**

We iterate `i` from `mx` down to 1. For each value `i`:

1. **Count multiples of `i`:**
   ```python
   v = 0
   for j in range(i, mx + 1, i):
       v += cnt[j]
   ```
   This counts how many numbers in `nums` are multiples of `i` (i.e., `i, 2i, 3i, ...`)

2. **Calculate pairs with GCD being a multiple of `i`:**
   ```python
   cnt_g[i] += v * (v - 1) // 2
   ```
   If there are `v` numbers that are multiples of `i`, they can form `v*(v-1)/2` pairs, all having GCD that's at least `i`.

3. **Subtract over-counted pairs:**
   ```python
   for j in range(i, mx + 1, i):
       cnt_g[i] -= cnt_g[j]
   ```
   We need to subtract pairs whose GCD is actually `2i, 3i, 4i, ...` (not exactly `i`). Since we're iterating from large to small, these values have already been computed.

**Step 3: Build [prefix sum](/problems/subarray_sum) array**
```python
s = list(accumulate(cnt_g))
```
The prefix sum array `s[i]` tells us how many pairs have GCD less than or equal to `i`. This essentially gives us the sorted `gcdPairs` array in compressed form.

**Step 4: Answer queries using [binary search](/problems/binary_search_intro)**

For each query `q`, we find the smallest index `i` where `s[i] > q` using the standard template:
```python
def upper_bound(s, target):
    left, right = 0, len(s) - 1
    first_true_index = len(s)  # Default: not found

    while left <= right:
        mid = (left + right) // 2
        if s[mid] > target:  # Feasible: found a position > target
            first_true_index = mid
            right = mid - 1  # Look for earlier
        else:
            left = mid + 1

    return first_true_index
```
This index `i` is the GCD value at position `q` in the conceptual sorted `gcdPairs` array.

**Example walkthrough:**
If `nums = [4, 6, 8]` and `mx = 8`:
- For `i = 8`: One number (8) is divisible by 8, so 0 pairs
- For `i = 4`: Two numbers (4, 8) divisible by 4, giving 1 pair with GCD ≥ 4
- For `i = 2`: All three numbers divisible by 2, giving 3 pairs, minus 1 pair with GCD=4, equals 2 pairs with GCD=2
- For `i = 1`: Would count all pairs minus those already counted

The [prefix sum](/problems/subarray_sum) array then allows us to map query indices to their corresponding GCD values efficiently.

## Example Walkthrough

Let's walk through a concrete example with `nums = [4, 6, 8]` and `queries = [0, 1, 2]`.

**Step 1: Setup**
- Maximum value: `mx = 8`
- Frequency counter: `cnt = {4: 1, 6: 1, 8: 1}`
- Initialize `cnt_g = [0, 0, 0, 0, 0, 0, 0, 0, 0]` (indices 0 to 8)

**Step 2: Calculate GCD counts (from largest to smallest)**

*For i = 8:*
- Multiples of 8 in nums: just 8 → `v = 1`
- Pairs with GCD ≥ 8: `1 * (1-1) / 2 = 0`
- No multiples to subtract
- `cnt_g[8] = 0`

*For i = 7:*
- No multiples of 7 in nums → `v = 0`
- `cnt_g[7] = 0`

*For i = 6:*
- Multiples of 6: just 6 → `v = 1`
- Pairs with GCD ≥ 6: `1 * 0 / 2 = 0`
- `cnt_g[6] = 0`

*For i = 5:*
- No multiples of 5 → `cnt_g[5] = 0`

*For i = 4:*
- Multiples of 4: 4 and 8 → `v = 2`
- Pairs with GCD ≥ 4: `2 * 1 / 2 = 1`
- Subtract cnt_g[8] = 0 (since 8 is a multiple of 4)
- `cnt_g[4] = 1 - 0 = 1`
- This represents the pair (4, 8) with GCD = 4

*For i = 3:*
- Multiples of 3: just 6 → `v = 1`
- `cnt_g[3] = 0`

*For i = 2:*
- Multiples of 2: 4, 6, and 8 → `v = 3`
- Pairs with GCD ≥ 2: `3 * 2 / 2 = 3`
- Subtract: cnt_g[4] = 1, cnt_g[6] = 0, cnt_g[8] = 0
- `cnt_g[2] = 3 - 1 = 2`
- This represents pairs (4, 6) and (6, 8), both with GCD = 2

*For i = 1:*
- All numbers are multiples of 1 → `v = 3`
- Pairs: `3 * 2 / 2 = 3`
- Subtract all other GCDs: 0 + 2 + 0 + 1 + 0 + 0 + 0 + 0 = 3
- `cnt_g[1] = 3 - 3 = 0`

**Step 3: Build prefix sum array**
- `cnt_g = [0, 0, 2, 0, 1, 0, 0, 0, 0]`
- Prefix sums: `s = [0, 0, 2, 2, 3, 3, 3, 3, 3]`

This means:
- Indices 0-1 in gcdPairs have GCD = 2 (2 pairs)
- Index 2 in gcdPairs has GCD = 4 (1 pair)

**Step 4: Answer queries**
- `queries[0] = 0`: bisect_right(s, 0) = 2 → answer is 2
- `queries[1] = 1`: bisect_right(s, 1) = 2 → answer is 2
- `queries[2] = 2`: bisect_right(s, 2) = 4 → answer is 4

**Verification:**
The actual pairs and their GCDs:
- gcd(4, 6) = 2
- gcd(4, 8) = 4
- gcd(6, 8) = 2

Sorted gcdPairs = [2, 2, 4]

So queries[0] = 2, queries[1] = 2, queries[2] = 4 ✓

The algorithm efficiently computes these values without explicitly generating all pairs, using the counting technique to handle large inputs effectively.

## Python Solution

```python
from typing import List
from collections import Counter
from itertools import accumulate
from bisect import bisect_right

class Solution:
    def gcdValues(self, nums: List[int], queries: List[int]) -> List[int]:
        # Find the maximum value in nums to determine the range
        max_value = max(nums)

        # Count frequency of each number in nums
        frequency_count = Counter(nums)

        # Array to store count of pairs with GCD equal to index
        gcd_pair_count = [0] * (max_value + 1)

        # Calculate GCD counts from largest to smallest using inclusion-exclusion principle
        for gcd in range(max_value, 0, -1):
            # Count numbers that are multiples of current gcd
            multiples_count = 0

            # Iterate through all multiples of current gcd
            for multiple in range(gcd, max_value + 1, gcd):
                multiples_count += frequency_count[multiple]
                # Subtract pairs already counted with larger GCDs (inclusion-exclusion)
                gcd_pair_count[gcd] -= gcd_pair_count[multiple]

            # Add number of pairs that can be formed from multiples
            # Using combination formula: C(n, 2) = n * (n - 1) / 2
            gcd_pair_count[gcd] += multiples_count * (multiples_count - 1) // 2

        # Create prefix sum array for binary search
        prefix_sum = list(accumulate(gcd_pair_count))

        # For each query, find the GCD value using binary search on prefix sum
        return [bisect_right(prefix_sum, query) for query in queries]
```

## Time and Space Complexity

**Time Complexity:** `O(n + M * log M + q * log M)`

- Creating the Counter from nums takes `O(n)` time
- The nested loop iterates through all divisors:
  - The outer loop runs from `mx` down to 1, which is `O(M)` iterations
  - For each value `i`, the inner loop iterates through multiples of `i` up to `mx`, which takes `O(M/i)` time
  - The total time for all iterations is `O(M/1 + M/2 + M/3 + ... + M/M) = O(M * log M)` (harmonic series)
- Building the prefix sum array with accumulate takes `O(M)` time
- For each of the `q` queries, binary search on the prefix sum array takes `O(log M)` time
- Total: `O(n + M * log M + q * log M)` which simplifies to `O(n + (M + q) * log M)`

**Space Complexity:** `O(M)`

- The Counter object stores at most `n` elements but uses keys up to value `M`, effectively `O(M)` in worst case
- The `cnt_g` array has size `M + 1`, which is `O(M)`
- The prefix sum array `s` also has size `M + 1`, which is `O(M)`
- Total space complexity is `O(M)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Order of Subtraction in Inclusion-Exclusion**

**The Pitfall:**
A common mistake is subtracting the over-counted pairs BEFORE calculating the total pairs with GCD being a multiple of `i`. This leads to incorrect results because you're subtracting from a value that hasn't been computed yet.

**Incorrect Implementation:**
```python
# WRONG: Subtracting before calculating total pairs
for gcd in range(max_value, 0, -1):
    multiples_count = 0

    # Subtract first (WRONG!)
    for multiple in range(gcd, max_value + 1, gcd):
        gcd_pair_count[gcd] -= gcd_pair_count[multiple]

    # Then calculate pairs
    for multiple in range(gcd, max_value + 1, gcd):
        multiples_count += frequency_count[multiple]

    gcd_pair_count[gcd] += multiples_count * (multiples_count - 1) // 2
```

**Correct Implementation:**
```python
# CORRECT: Calculate total pairs first, then subtract over-counted
for gcd in range(max_value, 0, -1):
    multiples_count = 0

    # First, count all multiples
    for multiple in range(gcd, max_value + 1, gcd):
        multiples_count += frequency_count[multiple]

    # Calculate total pairs with GCD being a multiple of current value
    gcd_pair_count[gcd] = multiples_count * (multiples_count - 1) // 2

    # Then subtract pairs with GCD that is a proper multiple (2*gcd, 3*gcd, etc.)
    for multiple in range(2 * gcd, max_value + 1, gcd):
        gcd_pair_count[gcd] -= gcd_pair_count[multiple]
```

### 2. **Starting Subtraction from Wrong Multiple**

**The Pitfall:**
Another subtle error is starting the subtraction loop from `gcd` instead of `2 * gcd`. This causes the algorithm to subtract `gcd_pair_count[gcd]` from itself, which zeros out the value incorrectly.

**Incorrect Implementation:**
```python
# WRONG: Starting from gcd itself
for multiple in range(gcd, max_value + 1, gcd):  # Starts at gcd
    gcd_pair_count[gcd] -= gcd_pair_count[multiple]
```

**Correct Implementation:**
```python
# CORRECT: Starting from 2*gcd to avoid self-subtraction
for multiple in range(2 * gcd, max_value + 1, gcd):  # Starts at 2*gcd
    gcd_pair_count[gcd] -= gcd_pair_count[multiple]
```

### 3. **Off-by-One Error with Query Indices**

**The Pitfall:**
Using `bisect_left` instead of `bisect_right` (or vice versa) without proper adjustment can lead to incorrect query answers, especially at boundary cases.

**Why it matters:**
- `bisect_right(s, q)` returns the smallest index where all values to the left are `≤ q`
- This directly gives us the GCD value at position `q` in our conceptual sorted array
- Using `bisect_left` would require additional logic to handle edge cases

**Example to illustrate the difference:**
If `prefix_sum = [0, 2, 5, 8]` and `query = 2`:
- `bisect_right([0, 2, 5, 8], 2)` returns `2` (correct GCD value)
- `bisect_left([0, 2, 5, 8], 2)` returns `1` (would need adjustment)

