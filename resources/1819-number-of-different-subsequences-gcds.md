# 1819. Number of Different Subsequences GCDs

> Source: https://algo.monster/liteproblems/1819

## Problem Statement

You have an array `nums` containing positive integers. Your task is to find how many different GCD values can be obtained from all possible non-empty subsequences of this array.

The **GCD (Greatest Common Divisor)** of a sequence is the largest integer that divides all numbers in that sequence evenly. For instance, the GCD of `[4, 6, 16]` is `2` because 2 is the largest number that divides 4, 6, and 16 without remainder.

A **subsequence** is formed by selecting some (or all) elements from the original array while maintaining their relative order. Elements don't need to be consecutive. For example, from the array `[1, 2, 1, 2, 4, 1, 5, 10]`, we can form the subsequence `[2, 5, 10]` by selecting the 2nd, 7th, and 8th elements.

The problem asks you to:
1. Consider all possible non-empty subsequences of `nums`
2. Calculate the GCD for each subsequence
3. Count how many distinct GCD values exist

For example, if `nums = [6, 10, 3]`, some possible subsequences and their GCDs would be:
- `[6]` → GCD = 6
- `[10]` → GCD = 10  
- `[3]` → GCD = 3
- `[6, 3]` → GCD = 3
- `[6, 10]` → GCD = 2
- `[10, 3]` → GCD = 1
- `[6, 10, 3]` → GCD = 1

The distinct GCD values are: 1, 2, 3, 6, 10, giving us an answer of 5.

## Intuition

Instead of generating all possible subsequences (which would be exponentially many), we can think about this problem from a different angle: what are all the possible GCD values that could exist?

The key insight is that any GCD of a subsequence must be between 1 and the maximum value in the array. Why? Because the GCD of any set of numbers cannot exceed the smallest number in that set, and the largest possible GCD would be when we pick just the maximum element by itself.

So we can flip our approach: instead of finding subsequences and computing their GCDs, we can check each potential GCD value `x` from 1 to `max(nums)` and determine if there exists some subsequence with GCD equal to `x`.

How do we check if a GCD value `x` is achievable? Here's the mathematical insight: if `x` is the GCD of some subsequence, then all elements in that subsequence must be multiples of `x`. This is because `x` divides all elements in the subsequence evenly.

Therefore, for each candidate GCD value `x`, we only need to look at elements in `nums` that are multiples of `x`. We can do this efficiently by checking values `x, 2x, 3x, 4x, ...` up to the maximum value in the array.

For the multiples of `x` that exist in `nums`, we compute their GCD. If this GCD equals `x`, then we've found a valid subsequence with GCD `x`. The reason this works is that we're essentially finding the "tightest" common divisor among all multiples of `x` in the array. If their GCD is exactly `x`, it means we can form a subsequence with that GCD.

This approach is much more efficient than brute force because we're iterating through potential GCD values (at most `max(nums)` values) and for each one, checking its multiples, rather than generating all `2^n` possible subsequences.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation follows our intuition by enumerating each possible GCD value and checking if it's achievable:

1. **Find the maximum value**: First, we find `mx = max(nums)` to establish the upper bound for possible GCD values.

2. **Create a set for O(1) lookup**: We convert `nums` to a set `vis = set(nums)` to enable constant-time checks for whether a number exists in the original array.

3. **Enumerate potential GCD values**: We iterate through each potential GCD value `x` from 1 to `mx`.

4. **Check multiples of x**: For each candidate GCD `x`, we check all its multiples `y` that could exist in the array:
   - We iterate `y` from `x` to `mx` in steps of `x` (i.e., `x, 2x, 3x, ...`)
   - This efficiently generates all multiples of `x` up to the maximum value

5. **Calculate GCD of found multiples**: 
   - We maintain a running GCD variable `g`, initially set to 0
   - For each multiple `y` that exists in `vis`, we update `g = gcd(g, y)`
   - The property `gcd(0, n) = n` for any number `n` helps us start the GCD calculation

6. **Check if the GCD equals x**:
   - If at any point `g == x`, we know that `x` is achievable as a GCD of some subsequence
   - We increment our answer counter and break early (optimization)
   - Breaking early is valid because once `g == x`, adding more multiples won't change the fact that the GCD is `x`

7. **Return the count**: The final answer is the total count of distinct GCD values found.

The time complexity is `O(mx * log(mx))` where `mx` is the maximum value in `nums`. For each potential GCD value `x`, we check approximately `mx/x` multiples, and the total work across all values is bounded by the harmonic series. The `gcd` operation takes `O(log(mx))` time.

The space complexity is `O(n)` for storing the set of numbers from the input array.

## Example Walkthrough

Let's walk through the solution with `nums = [6, 10, 3]`.

**Step 1: Setup**
- Find maximum: `mx = 10`
- Create set: `vis = {3, 6, 10}`
- Initialize answer counter: `ans = 0`

**Step 2: Check each potential GCD from 1 to 10**

**x = 1:**
- Check multiples: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
- Found in array: 3, 6, 10
- Calculate GCD: `g = gcd(0, 3) = 3`, then `g = gcd(3, 6) = 3`, then `g = gcd(3, 10) = 1`
- Since `g = 1 == x`, GCD of 1 is achievable! `ans = 1`

**x = 2:**
- Check multiples: 2, 4, 6, 8, 10
- Found in array: 6, 10
- Calculate GCD: `g = gcd(0, 6) = 6`, then `g = gcd(6, 10) = 2`
- Since `g = 2 == x`, GCD of 2 is achievable! `ans = 2`

**x = 3:**
- Check multiples: 3, 6, 9
- Found in array: 3, 6
- Calculate GCD: `g = gcd(0, 3) = 3`, then `g = gcd(3, 6) = 3`
- Since `g = 3 == x`, GCD of 3 is achievable! `ans = 3`

**x = 4:**
- Check multiples: 4, 8
- Found in array: none
- No multiples found, skip

**x = 5:**
- Check multiples: 5, 10
- Found in array: 10
- Calculate GCD: `g = gcd(0, 10) = 10`
- Since `g = 10 ≠ 5`, GCD of 5 is not achievable

**x = 6:**
- Check multiples: 6
- Found in array: 6
- Calculate GCD: `g = gcd(0, 6) = 6`
- Since `g = 6 == x`, GCD of 6 is achievable! `ans = 4`

**x = 7:**
- Check multiples: 7
- Found in array: none
- No multiples found, skip

**x = 8:**
- Check multiples: 8
- Found in array: none
- No multiples found, skip

**x = 9:**
- Check multiples: 9
- Found in array: none
- No multiples found, skip

**x = 10:**
- Check multiples: 10
- Found in array: 10
- Calculate GCD: `g = gcd(0, 10) = 10`
- Since `g = 10 == x`, GCD of 10 is achievable! `ans = 5`

**Final Answer: 5**

The distinct achievable GCD values are {1, 2, 3, 6, 10}, which corresponds to these subsequences:
- GCD = 1: [6, 10, 3] or [10, 3]
- GCD = 2: [6, 10]
- GCD = 3: [6, 3] or [3]
- GCD = 6: [6]
- GCD = 10: [10]

## Python Solution

```python
from typing import List
from math import gcd

class Solution:
    def countDifferentSubsequenceGCDs(self, nums: List[int]) -> int:
        # Find the maximum value in the array to set our search boundary
        max_value = max(nums)
      
        # Convert nums to a set for O(1) membership checking
        num_set = set(nums)
      
        # Counter for distinct GCD values
        distinct_gcd_count = 0
      
        # Try each possible GCD value from 1 to max_value
        for potential_gcd in range(1, max_value + 1):
            # Current GCD of all multiples of potential_gcd found in num_set
            current_gcd = 0
          
            # Check all multiples of potential_gcd up to max_value
            for multiple in range(potential_gcd, max_value + 1, potential_gcd):
                # If this multiple exists in our original array
                if multiple in num_set:
                    # Update the GCD with this multiple
                    current_gcd = gcd(current_gcd, multiple)
                  
                    # If we've found that potential_gcd is achievable as a GCD
                    if current_gcd == potential_gcd:
                        distinct_gcd_count += 1
                        break  # No need to check more multiples
      
        return distinct_gcd_count
```

## Time and Space Complexity

**Time Complexity:** `O(n + M × log M)`

The time complexity consists of two main parts:
- `O(n)` for creating the set `vis` from the input array `nums` and finding the maximum value
- `O(M × log M)` for the nested loops:
  - The outer loop runs from `1` to `mx`, contributing `O(M)` iterations
  - For each value `x` in the outer loop, the inner loop iterates through multiples of `x` up to `mx`, which takes `O(M/x)` iterations
  - The total number of inner loop iterations across all outer loop iterations is `M/1 + M/2 + M/3 + ... + M/M = M × (1/1 + 1/2 + 1/3 + ... + 1/M)`, which equals `O(M × log M)` (harmonic series)
  - Each iteration involves a GCD operation `gcd(g, y)` which takes `O(log M)` time, but since we break early when `g == x`, the amortized cost is absorbed in the `O(M × log M)` bound

**Space Complexity:** `O(M)`

The space complexity is determined by:
- The set `vis` which stores at most `n` unique elements from `nums`, but in the worst case when all elements are distinct and equal to `M`, this is `O(M)`
- Other variables (`mx`, `ans`, `g`, loop variables) use `O(1)` space

Where `n` is the length of the array `nums` and `M` is the maximum value in the array `nums`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Attempting to Generate All Subsequences Explicitly**

Many developers initially try to solve this problem by generating all possible subsequences and calculating their GCDs:

```python
# WRONG APPROACH - Time Limit Exceeded
def countDifferentSubsequenceGCDs(nums):
    gcd_values = set()
    n = len(nums)
  
    # Generate all 2^n subsequences
    for mask in range(1, 1 << n):
        subsequence = []
        for i in range(n):
            if mask & (1 << i):
                subsequence.append(nums[i])
      
        # Calculate GCD of subsequence
        g = subsequence[0]
        for num in subsequence[1:]:
            g = gcd(g, num)
        gcd_values.add(g)
  
    return len(gcd_values)
```

**Why it fails**: This approach has O(2^n) time complexity, which is exponential. For arrays with even moderate sizes (n > 20), this becomes computationally infeasible.

**Solution**: Instead of generating subsequences, iterate through potential GCD values and check if each is achievable by finding its multiples in the array.

### 2. **Incorrect GCD Calculation Logic**

A subtle bug can occur when calculating the running GCD:

```python
# WRONG - Incorrect initialization
for potential_gcd in range(1, max_value + 1):
    current_gcd = potential_gcd  # Wrong initialization!
  
    for multiple in range(potential_gcd, max_value + 1, potential_gcd):
        if multiple in num_set:
            current_gcd = gcd(current_gcd, multiple)
```

**Why it fails**: Initializing `current_gcd` to `potential_gcd` assumes we've already found one element, which may not exist in the array. This can lead to counting GCD values that aren't actually achievable.

**Solution**: Initialize `current_gcd` to 0. The mathematical property `gcd(0, n) = n` ensures correct behavior when we find the first multiple.

### 3. **Forgetting Edge Cases with Single Elements**

Some implementations forget that single-element subsequences are valid:

```python
# WRONG - Missing single elements
for potential_gcd in range(1, max_value + 1):
    current_gcd = 0
    found_count = 0
  
    for multiple in range(potential_gcd, max_value + 1, potential_gcd):
        if multiple in num_set:
            found_count += 1
            current_gcd = gcd(current_gcd, multiple)
  
    # Wrong condition - requires at least 2 elements
    if found_count >= 2 and current_gcd == potential_gcd:
        distinct_gcd_count += 1
```

**Why it fails**: Every single element in the array forms a valid subsequence with GCD equal to itself. Requiring multiple elements misses these cases.

**Solution**: Remove the constraint on the number of elements found. Any non-zero number of multiples that produce the target GCD is valid.

### 4. **Not Using Early Termination Optimization**

While not incorrect, failing to break early can impact performance:

```python
# CORRECT but less efficient
for potential_gcd in range(1, max_value + 1):
    current_gcd = 0
  
    for multiple in range(potential_gcd, max_value + 1, potential_gcd):
        if multiple in num_set:
            current_gcd = gcd(current_gcd, multiple)
  
    # Only check at the end - misses optimization opportunity
    if current_gcd == potential_gcd:
        distinct_gcd_count += 1
```

**Why it's suboptimal**: Once `current_gcd` equals `potential_gcd`, checking additional multiples won't change this fact (the GCD can only stay the same or decrease).

**Solution**: Check if `current_gcd == potential_gcd` after each update and break immediately when true. This can significantly reduce unnecessary iterations.

