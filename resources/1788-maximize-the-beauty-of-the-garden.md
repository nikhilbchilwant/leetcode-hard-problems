# 1788. Maximize the Beauty of the Garden

> Source: https://algo.monster/liteproblems/1788

## Problem Statement

You have a garden with `n` flowers arranged in a line, where each flower has an integer beauty value (which can be positive, negative, or zero). These beauty values are given in an array `flowers` where `flowers[i]` represents the beauty of the `i-th` flower.

A garden is considered **valid** if:
- It contains at least two flowers
- The first and last flowers have the same beauty value

As a gardener, you can remove any flowers (or none at all) from the garden. Your goal is to create a valid garden with the maximum possible total beauty. The beauty of a garden is the sum of all beauty values of the remaining flowers.

For example, if you have flowers with beauty values `[1, 2, 3, 1, 2]`, you could keep flowers at positions 0 and 3 (both with beauty value 1) along with any flowers between them, creating a valid garden since the first and last flowers both have beauty value 1.

The task is to find the maximum possible beauty sum of a valid garden after optimally removing flowers.

Note that when calculating the beauty of a garden, you sum up all the beauty values of the flowers that remain in the garden, including any negative values.

## Intuition

The key insight is that a valid garden must start and end with flowers having the same beauty value. So we need to find pairs of flowers with identical beauty values and maximize the sum of all flowers between them (inclusive).

When we encounter a flower with beauty value `v` at position `j`, we want to check if we've seen this value before. If we have seen it at an earlier position `i`, then flowers from position `i` to `j` can form a valid garden. The beauty of this garden would be the sum of all flower values from `i` to `j`.

However, there's a twist - we want to maximize the beauty, so between positions `i` and `j`, we should keep all positive beauty values but skip negative ones (since we're allowed to remove any flowers). This is where prefix sums become useful.

We can maintain a [prefix sum](/problems/subarray_sum) array where `s[i]` represents the sum of all non-negative beauty values from the start up to position `i-1`. When we find a matching pair at positions `i` and `j`, the sum of all positive values between them would be `s[j] - s[i+1]`. But we must also include the boundary flowers (with value `v`) at positions `i` and `j`, giving us a total of `s[j] - s[i+1] + v + v`.

The reason we only accumulate non-negative values in our [prefix sum](/problems/subarray_sum) is strategic: negative beauty values between the matching pair would decrease our total, so we simply choose not to include them (remove them from the garden). But the boundary flowers must be kept regardless of their value since they define the validity of the garden.

By tracking the first occurrence of each beauty value in a hash table and continuously updating our maximum as we find matching pairs, we can efficiently find the optimal valid garden in a single pass through the array.

**Learn more about [Greedy](/problems/greedy_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

We implement the solution using a hash table and [prefix sum](/problems/subarray_sum) array to efficiently find the maximum beauty valid garden.

**Data Structures:**
- Hash table `d`: Records the first occurrence index of each beauty value
- [Prefix sum](/problems/subarray_sum) array `s`: Stores cumulative sum of non-negative beauty values
- Variable `ans`: Tracks the maximum beauty found, initialized to negative infinity

**Algorithm Steps:**

1. **Initialize structures**: Create a [prefix sum](/problems/subarray_sum) array `s` of size `n+1` (one extra for easier indexing), an empty hash table `d`, and set `ans = -inf`.

2. **Single pass through flowers**: For each flower at position `i` with beauty value `v`:

   a. **Check for matching pair**: If `v` already exists in hash table `d`:
      - We found a valid garden from position `d[v]` to `i`
      - Calculate beauty: `s[i] - s[d[v] + 1] + v * 2`
        - `s[i] - s[d[v] + 1]` gives sum of positive values between the pair (exclusive)
        - `v * 2` adds the two boundary flowers
      - Update `ans` with the maximum value

   b. **First occurrence**: If `v` is not in `d`:
      - Record this position: `d[v] = i`

   c. **Update [prefix sum](/problems/subarray_sum)**:
      - `s[i + 1] = s[i] + max(v, 0)`
      - We only add non-negative values to skip negative beauty values in the middle

3. **Return result**: After processing all flowers, `ans` contains the maximum possible beauty.

**Key Insight**: The formula `s[i] - s[d[v] + 1] + v * 2` works because:
- `s[i]` contains sum of all positive values from start to position `i-1`
- `s[d[v] + 1]` contains sum of all positive values from start to position `d[v]`
- Their difference gives us positive values between `d[v]+1` and `i-1`
- Adding `v * 2` includes the mandatory boundary flowers at positions `d[v]` and `i`

This approach runs in O(n) time with O(n) space complexity, processing each flower exactly once while maintaining the necessary information to find optimal valid gardens.

## Example Walkthrough

Let's walk through the solution with `flowers = [1, -2, 3, 1, 5]`.

**Initialization:**
- `s = [0, 0, 0, 0, 0, 0]` (prefix sum array of size n+1)
- `d = {}` (empty hash table)
- `ans = -inf`

**Processing each flower:**

**i = 0, v = 1:**
- Check if 1 is in `d`: No
- Add to hash table: `d[1] = 0`
- Update prefix sum: `s[1] = s[0] + max(1, 0) = 0 + 1 = 1`
- Current state: `d = {1: 0}`, `s = [0, 1, 0, 0, 0, 0]`, `ans = -inf`

**i = 1, v = -2:**
- Check if -2 is in `d`: No
- Add to hash table: `d[-2] = 1`
- Update prefix sum: `s[2] = s[1] + max(-2, 0) = 1 + 0 = 1`
- Current state: `d = {1: 0, -2: 1}`, `s = [0, 1, 1, 0, 0, 0]`, `ans = -inf`

**i = 2, v = 3:**
- Check if 3 is in `d`: No
- Add to hash table: `d[3] = 2`
- Update prefix sum: `s[3] = s[2] + max(3, 0) = 1 + 3 = 4`
- Current state: `d = {1: 0, -2: 1, 3: 2}`, `s = [0, 1, 1, 4, 0, 0]`, `ans = -inf`

**i = 3, v = 1:**
- Check if 1 is in `d`: Yes! Found at position 0
- Calculate beauty of garden from position 0 to 3:
  - `beauty = s[3] - s[0 + 1] + 1 * 2`
  - `beauty = 4 - 1 + 2 = 5`
  - This represents keeping flowers [1, 3, 1] (skipping the -2)
- Update `ans = max(-inf, 5) = 5`
- Update prefix sum: `s[4] = s[3] + max(1, 0) = 4 + 1 = 5`
- Current state: `d = {1: 0, -2: 1, 3: 2}`, `s = [0, 1, 1, 4, 5, 0]`, `ans = 5`

**i = 4, v = 5:**
- Check if 5 is in `d`: No
- Add to hash table: `d[5] = 4`
- Update prefix sum: `s[5] = s[4] + max(5, 0) = 5 + 5 = 10`
- Final state: `d = {1: 0, -2: 1, 3: 2, 5: 4}`, `s = [0, 1, 1, 4, 5, 10]`, `ans = 5`

**Result:** The maximum beauty is 5, achieved by keeping flowers at positions 0, 2, and 3 with values [1, 3, 1].

## Python Solution

```python
from typing import List
from math import inf

class Solution:
    def maximumBeauty(self, flowers: List[int]) -> int:
        # prefix_sum[i] stores the sum of all positive values from index 0 to i-1
        prefix_sum = [0] * (len(flowers) + 1)

        # first_occurrence stores the first occurrence index of each flower value
        first_occurrence = {}

        # Initialize maximum beauty to negative infinity
        max_beauty = -inf

        # Iterate through each flower
        for index, flower_value in enumerate(flowers):
            # If we've seen this flower value before, we can form a valid subarray
            if flower_value in first_occurrence:
                # Calculate beauty:
                # - Add the flower value twice (for start and end positions)
                # - Add sum of positive values between the two occurrences
                first_index = first_occurrence[flower_value]
                positive_sum_between = prefix_sum[index] - prefix_sum[first_index + 1]
                current_beauty = positive_sum_between + flower_value * 2
                max_beauty = max(max_beauty, current_beauty)
            else:
                # Record the first occurrence of this flower value
                first_occurrence[flower_value] = index

            # Update prefix sum: only add positive values
            prefix_sum[index + 1] = prefix_sum[index] + max(flower_value, 0)

        return max_beauty
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm iterates through the flowers array exactly once using a single for loop. Within each iteration, all operations are constant time:
- Dictionary lookup (`v in d`) - `O(1)` average case
- Dictionary insertion (`d[v] = i`) - `O(1)` average case
- Array access and update (`s[i + 1] = s[i] + max(v, 0)`) - `O(1)`
- Max comparisons - `O(1)`

Since we perform `n` iterations with `O(1)` operations per iteration, the overall time complexity is `O(n)`.

**Space Complexity:** `O(n)`

The algorithm uses additional space for:
- Array `s` of size `len(flowers) + 1` - `O(n)`
- Dictionary `d` which in the worst case stores all unique flower values - `O(n)`
- Variables `ans`, `i`, `v` - `O(1)`

The dominant space usage comes from the array `s` and dictionary `d`, both of which can grow proportionally to the input size `n`. Therefore, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Including Negative Values in the Middle**
**Pitfall**: A common mistake is trying to include all values between the matching flowers, including negative ones. Developers might write:
```python
# WRONG: This includes negative values unnecessarily
current_beauty = sum(flowers[first_index:index+1])
```

**Why it's wrong**: Including negative values between the matching flowers reduces the total beauty unnecessarily. We should skip them since we're allowed to remove any flowers from the middle.

**Solution**: Use a prefix sum array that only accumulates non-negative values:
```python
# Correct: Only sum positive values between the boundaries
prefix_sum[index + 1] = prefix_sum[index] + max(flower_value, 0)
positive_sum_between = prefix_sum[index] - prefix_sum[first_index + 1]
```

### 2. **Off-by-One Errors in Index Calculation**
**Pitfall**: Incorrectly calculating the sum between two positions:
```python
# WRONG: This includes the value at first_index
positive_sum_between = prefix_sum[index] - prefix_sum[first_index]
```

**Why it's wrong**: The prefix sum at `prefix_sum[first_index]` contains values from 0 to `first_index-1`. To get values strictly between `first_index` and `index`, we need `prefix_sum[index] - prefix_sum[first_index + 1]`.

**Solution**: Use `first_index + 1` to exclude the boundary flower from the middle sum:
```python
# Correct: Excludes both boundary flowers from the middle sum
positive_sum_between = prefix_sum[index] - prefix_sum[first_index + 1]
current_beauty = positive_sum_between + flower_value * 2
```

### 3. **Updating First Occurrence After Finding a Match**
**Pitfall**: Updating the first occurrence of a flower value after finding a match:
```python
# WRONG: Don't update first_occurrence after finding a match
if flower_value in first_occurrence:
    # ... calculate beauty ...
    first_occurrence[flower_value] = index  # WRONG!
```

**Why it's wrong**: We always want to use the leftmost occurrence to potentially create the longest valid garden. Updating it would miss better solutions.

**Solution**: Only update `first_occurrence` when encountering a value for the first time:
```python
# Correct: Only set first occurrence once
if flower_value in first_occurrence:
    # Calculate beauty using the original first occurrence
    # ...
else:
    first_occurrence[flower_value] = index  # Only set on first encounter
```

### 4. **Forgetting to Handle Negative Boundary Values**
**Pitfall**: Assuming boundary flowers are always positive and forgetting to add them when they're negative:
```python
# WRONG: Only works correctly for positive boundary values
if flower_value > 0:
    current_beauty = positive_sum_between + flower_value * 2
```

**Why it's wrong**: Even if the boundary flowers have negative beauty values, they must be included in a valid garden. The algorithm should still consider them as they might lead to the maximum beauty when combined with large positive values in between.

**Solution**: Always include the boundary flowers regardless of their sign:
```python
# Correct: Always add boundary flowers, even if negative
current_beauty = positive_sum_between + flower_value * 2
```

