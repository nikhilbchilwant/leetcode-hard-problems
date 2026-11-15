# 3351. Sum of Good Subsequences

> Source: https://algo.monster/liteproblems/3351

## Problem Statement

You are given an integer array `nums`. Your task is to find all possible **good subsequences** and return their sum.

A **good subsequence** is defined as a subsequence where the absolute difference between any two consecutive elements is exactly 1. In other words, if you pick elements from the array to form a subsequence, each adjacent pair in your subsequence must differ by exactly 1.

For example:
- If `nums = [1, 2, 1]`, the good subsequences would be: `[1]`, `[2]`, `[1]` (the individual elements), `[1,2]`, `[2,1]`, and `[1,2,1]`
- The subsequence `[1,2]` is good because `|2-1| = 1`
- The subsequence `[1,1]` would NOT be good because `|1-1| = 0`

Important points:
- A subsequence maintains the relative order of elements from the original array (you can skip elements but cannot rearrange them)
- Single elements are considered good subsequences by definition
- You need to return the **sum of all elements** across all possible good subsequences
- Since the answer can be very large, return it modulo `10^9 + 7`

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with two tracking dictionaries:
- `f[x]`: tracks the sum of all good subsequences ending at value `x`
- `g[x]`: tracks the count of all good subsequences ending at value `x`

For each element in the array, the algorithm updates these values by considering:
1. The element itself as a new subsequence
2. Extending subsequences ending at `x-1` (one less than current)
3. Extending subsequences ending at `x+1` (one more than current)

## Intuition

The key insight is that we need to track good subsequences as we build them, but we don't need to store the actual subsequences - just their sums and counts.

Think about what happens when we encounter a value `x` in the array. This value can:
1. Start a new good subsequence by itself
2. Extend any good subsequence that ends with `x-1` (since `|x - (x-1)| = 1`)
3. Extend any good subsequence that ends with `x+1` (since `|x - (x+1)| = 1`)

This leads us to a [dynamic programming](/problems/dynamic_programming_intro) approach where we maintain information about subsequences ending at each value.

Why do we need two tracking variables?
- **Count (`g[x]`)**: We need to know how many good subsequences end with value `x` because when we see another `x` later, we can extend all of them
- **Sum (`f[x]`)**: We need the sum of all good subsequences ending with `x` to build our final answer

When we process a new element `x`:
- We add `x` itself as a new subsequence (count increases by 1, sum increases by `x`)
- For all subsequences ending at `x-1`, we can append `x` to create new subsequences. If there are `g[x-1]` such subsequences, we're adding `x` to each of them, contributing `g[x-1] * x` to the sum
- Similarly for subsequences ending at `x+1`, we add `g[x+1] * x` to the sum

The beauty of this approach is that we process each element once and update our tracking dictionaries on the fly. We don't need to generate all subsequences explicitly - we just keep track of the cumulative sums and counts grouped by the ending value.

At the end, the sum of all values in `f` gives us the total sum of all good subsequences, because `f[v]` contains the sum of all good subsequences ending with value `v`, and every good subsequence must end with some value.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses [dynamic programming](/problems/dynamic_programming_intro) with two hash maps (dictionaries) to efficiently compute the sum of all good subsequences.

**Data Structures:**
- `f`: A dictionary where `f[x]` stores the sum of all good subsequences ending with value `x`
- `g`: A dictionary where `g[x]` stores the count of all good subsequences ending with value `x`

**Algorithm Steps:**

1. **Initialize** empty dictionaries `f` and `g` using `defaultdict(int)` to handle missing keys automatically.

2. **Process each element** `x` in `nums`:
 
   a. **Add `x` as a new subsequence:**
      - `f[x] += x` (add the value itself to the sum)
      - `g[x] += 1` (increment count by 1)
 
   b. **Extend subsequences ending at `x-1`:**
      - `f[x] += f[x - 1] + g[x - 1] * x`
        - `f[x - 1]`: sum of all subsequences ending at `x-1`
        - `g[x - 1] * x`: contribution of appending `x` to each of those subsequences
      - `g[x] += g[x - 1]` (add the count of subsequences we're extending)
 
   c. **Extend subsequences ending at `x+1`:**
      - `f[x] += f[x + 1] + g[x + 1] * x`
        - `f[x + 1]`: sum of all subsequences ending at `x+1`
        - `g[x + 1] * x`: contribution of appending `x` to each of those subsequences
      - `g[x] += g[x + 1]` (add the count of subsequences we're extending)

3. **Calculate final result:**
   - Sum all values in `f` dictionary: `sum(f.values())`
   - Apply modulo `10^9 + 7` to get the final answer

**Why this works:**
- When we process an element `x`, we're considering all ways it can participate in good subsequences
- By updating both sum and count simultaneously, we maintain complete information about subsequences ending at each value
- The order of processing ensures that when we extend subsequences from `x-1` or `x+1`, we're using the most up-to-date information
- Each element can extend multiple existing subsequences, which is why we multiply by the count when adding to the sum

**Time Complexity:** `O(n)` where `n` is the length of the input array
**Space Complexity:** `O(k)` where `k` is the number of unique values in the array

## Example Walkthrough

Let's walk through the algorithm with `nums = [2, 3, 1]`.

We'll track two dictionaries:
- `f[x]`: sum of all good subsequences ending with value `x`
- `g[x]`: count of all good subsequences ending with value `x`

**Initial state:**
- `f = {}`, `g = {}`

**Processing element 2:**
- Start new subsequence with just `2`:
  - `f[2] = 0 + 2 = 2` (sum increases by 2)
  - `g[2] = 0 + 1 = 1` (count increases by 1)
- Check for subsequences ending at `2-1=1`: None exist
- Check for subsequences ending at `2+1=3`: None exist
- **State:** `f = {2: 2}`, `g = {2: 1}`
- Good subsequences so far: `[2]`

**Processing element 3:**
- Start new subsequence with just `3`:
  - `f[3] = 0 + 3 = 3`
  - `g[3] = 0 + 1 = 1`
- Extend subsequences ending at `3-1=2`:
  - Found 1 subsequence ending at 2 with sum 2
  - `f[3] = 3 + 2 + (1 × 3) = 8`
  - `g[3] = 1 + 1 = 2`
- Check for subsequences ending at `3+1=4`: None exist
- **State:** `f = {2: 2, 3: 8}`, `g = {2: 1, 3: 2}`
- Good subsequences so far: `[2]`, `[3]`, `[2,3]`

**Processing element 1:**
- Start new subsequence with just `1`:
  - `f[1] = 0 + 1 = 1`
  - `g[1] = 0 + 1 = 1`
- Check for subsequences ending at `1-1=0`: None exist
- Extend subsequences ending at `1+1=2`:
  - Found 1 subsequence ending at 2 with sum 2
  - `f[1] = 1 + 2 + (1 × 1) = 4`
  - `g[1] = 1 + 1 = 2`
- **State:** `f = {2: 2, 3: 8, 1: 4}`, `g = {2: 1, 3: 2, 1: 2}`
- Good subsequences so far: `[2]`, `[3]`, `[2,3]`, `[1]`, `[2,1]`

**Final calculation:**
- Sum of all values in `f`: `2 + 8 + 4 = 14`

Let's verify our subsequences and their sums:
- `[2]` → sum = 2
- `[3]` → sum = 3  
- `[2,3]` → sum = 5
- `[1]` → sum = 1
- `[2,1]` → sum = 3
- Total: 2 + 3 + 5 + 1 + 3 = 14 ✓

The algorithm correctly computes the sum of all good subsequences by tracking sums and counts of subsequences ending at each value, building up the solution incrementally.

## Python Solution

```python
from collections import defaultdict
from typing import List

class Solution:
    def sumOfGoodSubsequences(self, nums: List[int]) -> int:
        MOD = 10**9 + 7
      
        # sum_ending_at[val] = sum of all good subsequences ending with value val
        sum_ending_at = defaultdict(int)
      
        # count_ending_at[val] = count of all good subsequences ending with value val
        count_ending_at = defaultdict(int)
      
        for current_value in nums:
            # Start a new subsequence with just the current value
            sum_ending_at[current_value] += current_value
            count_ending_at[current_value] += 1
          
            # Extend subsequences ending at (current_value - 1)
            # Each such subsequence gets current_value added to it
            sum_ending_at[current_value] += sum_ending_at[current_value - 1] + count_ending_at[current_value - 1] * current_value
            count_ending_at[current_value] += count_ending_at[current_value - 1]
          
            # Extend subsequences ending at (current_value + 1)
            # Each such subsequence gets current_value added to it
            sum_ending_at[current_value] += sum_ending_at[current_value + 1] + count_ending_at[current_value + 1] * current_value
            count_ending_at[current_value] += count_ending_at[current_value + 1]
      
        # Return the total sum of all good subsequences
        return sum(sum_ending_at.values()) % MOD
```

## Time and Space Complexity

**Time Complexity:** `O(n)`, where `n` is the length of the input array `nums`.

The algorithm iterates through the array exactly once. For each element, it performs a constant number of operations:
- Dictionary lookups and updates for keys `x`, `x-1`, and `x+1` in both `f` and `g` dictionaries
- Basic arithmetic operations (addition and multiplication)
- Each dictionary operation takes `O(1)` average time

After the main loop, `sum(f.values())` takes `O(k)` time where `k` is the number of unique values in `nums`, and since `k ≤ n`, this is also `O(n)`.

Therefore, the overall time complexity is `O(n)`.

**Space Complexity:** `O(k)`, where `k` is the number of unique values in the input array `nums`.

The algorithm uses two dictionaries (`f` and `g`):
- Each dictionary stores at most one entry for each unique value that appears in `nums` or is adjacent to a value in `nums` (i.e., `x-1` or `x+1` for some `x` in `nums`)
- In the worst case, this creates at most `3k` entries across both dictionaries (for each unique value `x`, we might have entries for `x-1`, `x`, and `x+1`)
- Since `3k` is still `O(k)`, the space complexity is `O(k)`

In the worst case where all elements are unique, `k = n`, making the space complexity `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Not Applying Modulo Operations During Computation

**Problem:** The code currently only applies modulo at the final return statement. When dealing with large arrays or large values, intermediate calculations can cause integer overflow, leading to incorrect results even before the final modulo operation.

**Example scenario:** If `nums` contains many elements or large values, the intermediate sums in `sum_ending_at` can exceed Python's integer limits in other languages, or cause performance issues with very large numbers.

**Solution:** Apply modulo operations after each addition to keep numbers manageable:

```python
def sumOfGoodSubsequences(self, nums: List[int]) -> int:
    MOD = 10**9 + 7
    sum_ending_at = defaultdict(int)
    count_ending_at = defaultdict(int)
  
    for current_value in nums:
        # Calculate new sum and count for current_value
        new_sum = current_value
        new_count = 1
      
        # Extend from current_value - 1
        new_sum = (new_sum + sum_ending_at[current_value - 1] + 
                   count_ending_at[current_value - 1] * current_value) % MOD
        new_count = (new_count + count_ending_at[current_value - 1]) % MOD
      
        # Extend from current_value + 1
        new_sum = (new_sum + sum_ending_at[current_value + 1] + 
                   count_ending_at[current_value + 1] * current_value) % MOD
        new_count = (new_count + count_ending_at[current_value + 1]) % MOD
      
        # Update dictionaries
        sum_ending_at[current_value] = (sum_ending_at[current_value] + new_sum) % MOD
        count_ending_at[current_value] = (count_ending_at[current_value] + new_count) % MOD
  
    return sum(sum_ending_at.values()) % MOD
```

### Pitfall 2: Incorrect Update Order Within Loop

**Problem:** The original code updates `sum_ending_at[current_value]` and `count_ending_at[current_value]` incrementally within the loop. This can lead to using partially updated values when the same value appears multiple times in the array.

**Example:** For `nums = [1, 2, 1]`, when processing the second `1`, we're adding to the already updated `sum_ending_at[1]` from the first `1`, which might not be the intended behavior.

**Solution:** Calculate the new values first, then update the dictionaries:

```python
def sumOfGoodSubsequences(self, nums: List[int]) -> int:
    MOD = 10**9 + 7
    sum_ending_at = defaultdict(int)
    count_ending_at = defaultdict(int)
  
    for current_value in nums:
        # Calculate contributions from extending existing subsequences
        extend_sum = 0
        extend_count = 0
      
        # Extensions from value - 1
        if current_value - 1 in sum_ending_at:
            extend_sum += sum_ending_at[current_value - 1] + count_ending_at[current_value - 1] * current_value
            extend_count += count_ending_at[current_value - 1]
      
        # Extensions from value + 1
        if current_value + 1 in sum_ending_at:
            extend_sum += sum_ending_at[current_value + 1] + count_ending_at[current_value + 1] * current_value
            extend_count += count_ending_at[current_value + 1]
      
        # Update with new subsequence starting at current_value plus extensions
        sum_ending_at[current_value] = (sum_ending_at[current_value] + current_value + extend_sum) % MOD
        count_ending_at[current_value] = (count_ending_at[current_value] + 1 + extend_count) % MOD
  
    return sum(sum_ending_at.values()) % MOD
```

### Pitfall 3: Memory Issues with Very Large Value Ranges

**Problem:** If the input array contains values with a very large range (e.g., `-10^9` to `10^9`), the dictionaries might consume excessive memory even with sparse data.

**Solution:** Consider value normalization or compression if the actual number of distinct values is much smaller than the range:

```python
def sumOfGoodSubsequences(self, nums: List[int]) -> int:
    MOD = 10**9 + 7
  
    # Compress values if needed
    unique_vals = sorted(set(nums))
    if len(unique_vals) < len(nums) // 2:  # Only compress if beneficial
        val_map = {v: i for i, v in enumerate(unique_vals)}
        compressed = [val_map[x] for x in nums]
        # Process with compressed values, then map back results
  
    # Continue with regular processing...
```

