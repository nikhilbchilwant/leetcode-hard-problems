# 956. Tallest Billboard

> Source: https://algo.monster/liteproblems/956

## Problem Statement

You need to build a billboard with two steel supports of equal height, one on each side. You're given a collection of `rods` with different lengths that can be welded together to create these supports.

The goal is to make the tallest possible billboard by:
- Dividing the rods into two groups (one for each support)
- Each group's rods are welded together to form a support
- Both supports must have exactly the same height
- You want to maximize this height

For example, if you have rods of lengths `[1, 2, 3]`:
- You could weld rod 3 for one support (height = 3)
- Weld rods 1 and 2 for the other support (height = 1 + 2 = 3)
- This gives you a billboard height of 3

Not all rods need to be used - some can be left out if it helps achieve equal heights for both supports.

Return the maximum possible height of the billboard. If it's impossible to create two supports of equal height, return `0`.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization. The `dfs(i, j)` function tracks:
- `i`: the current rod index being considered
- `j`: the difference in height between the two supports

For each rod, there are three choices:
1. Skip the rod (don't use it)
2. Add it to the taller support (increasing the difference `j`)
3. Add it to the shorter support (decreasing the difference `j`)

The function returns the maximum height achievable when both supports are equal (when `j = 0`).

## Intuition

The key insight is that we need to track the difference between the two support heights rather than tracking each support's height separately. This transforms the problem into finding a way to make this difference equal to zero while maximizing the height of the supports.

Think of it this way: imagine you're building the two supports simultaneously. At any point, one support might be taller than the other. We can represent this state with just the difference in their heights. Our goal is to end with a difference of 0 (equal heights).

For each rod, we have three decisions:
1. **Don't use it** - the difference stays the same
2. **Add it to the taller support** - this increases the difference
3. **Add it to the shorter support** - this reduces the difference

Why track the difference instead of individual heights? Because if we know the difference and how much we've added to get there, we can reconstruct the actual heights. More importantly, this reduces our state space significantly - instead of tracking two potentially large values, we track just one.

The brilliant part is in the third choice: when we add a rod to the shorter support, we're actually contributing to the final billboard height. If the rod is longer than the current difference, it makes the previously shorter support become the taller one, and we flip which support is "ahead". The amount we can count toward our final answer is `min(j, rods[i])` - either we close the entire gap `j`, or we use the entire rod length.

This [dynamic programming](/problems/dynamic_programming_intro) approach with memoization ensures we explore all possible ways to distribute the rods while keeping track of the maximum achievable height when both supports are equal.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization through Python's `@cache` decorator. Let's break down the implementation:

**State Definition:**
- `dfs(i, j)` represents the maximum height achievable when:
  - We've considered rods from index `i` to the end
  - The current difference between the two supports is `j`

**Base Case:**
- When `i >= len(rods)` (all rods considered):
  - If `j == 0`: both supports are equal, return `0` (no more height can be added)
  - If `j != 0`: supports are unequal, return `-inf` (invalid state)

**Recursive Transitions:**
For each rod at index `i`, we have three choices:

1. **Skip the rod:** 
   - `dfs(i + 1, j)` - move to next rod without changing the difference

2. **Add to the taller support:**
   - `dfs(i + 1, j + rods[i])` - increases the difference by `rods[i]`

3. **Add to the shorter support:**
   - `dfs(i + 1, abs(rods[i] - j)) + min(j, rods[i])`
   - The new difference becomes `abs(rods[i] - j)`
   - We add `min(j, rods[i])` to our answer because:
     - If `rods[i] <= j`: we close gap by `rods[i]`, contributing that much to final height
     - If `rods[i] > j`: we close the entire gap `j` and overshoot, contributing `j` to final height

**Why this works:**
- The function returns the maximum additional height we can achieve from the current state
- When we add a rod to the shorter support, we're "banking" height that counts toward our final answer
- The `@cache` decorator memoizes results, preventing redundant calculations

**Time Complexity:** `O(n * S)` where `n` is the number of rods and `S` is the sum of all rod lengths (maximum possible difference)

**Space Complexity:** `O(n * S)` for the memoization cache

The initial call `dfs(0, 0)` starts with the first rod and zero difference, returning the maximum achievable billboard height.

## Example Walkthrough

Let's walk through the solution with rods = `[1, 2, 3, 4]`.

In the code the function is `dp(index, height_diff)`; below it is written as `dfs(i, j)`, with `i` the rod index and `j` the height difference. Each step adds a "contribution" equal to how much the shorter support's height rises toward the taller one, and the answer is the largest total contribution over all paths that end with `j = 0`.

We start with `dfs(0, 0)`, considering the rod at index 0 (length 1) with difference 0.

**Step 1: Rod 1 (length 1)**
From `dfs(0, 0)`, we have three choices:
- Skip it: `dfs(1, 0)`
- Add to taller support: `dfs(1, 1)` (difference becomes 0 + 1 = 1)
- Add to shorter support: since both are equal (`j = 0`), this also gives `dfs(1, 1)` and contributes `min(0, 1) = 0`

Follow the path where rod 1 goes on support A: A = 1, B = 0, state `dfs(1, 1)`.

**Step 2: Rod 2 (length 2)**
From `dfs(1, 1)`:
- Skip it: `dfs(2, 1)`
- Add to taller support: `dfs(2, 3)` (difference becomes 1 + 2 = 3)
- Add to shorter support: `dfs(2, |2 - 1|) = dfs(2, 1)` plus `min(1, 2) = 1`

Follow the shorter-support choice: rod 2 goes on B, so A = 1, B = 2. B is now taller by 1, and the contribution so far is 1 (the shorter support, A, has height 1).

**Step 3: Rod 3 (length 3)**
From `dfs(2, 1)`:
- Skip it: `dfs(3, 1)`
- Add to taller support: `dfs(3, 4)` (difference becomes 1 + 3 = 4)
- Add to shorter support: `dfs(3, |3 - 1|) = dfs(3, 2)` plus `min(1, 3) = 1`

Follow the taller-support choice: rod 3 goes on B, so A = 1, B = 5, state `dfs(3, 4)`. No contribution is added.

**Step 4: Rod 4 (length 4)**
From `dfs(3, 4)`:
- Skip it: `dfs(4, 4)` returns `-inf` (unequal supports at the end)
- Add to taller support: `dfs(4, 8)` returns `-inf`
- Add to shorter support: `dfs(4, |4 - 4|) = dfs(4, 0)` plus `min(4, 4) = 4`. The base case returns 0, so this choice gives 4.

Rod 4 goes on A, so A = 1 + 4 = 5 and B = 2 + 3 = 5.

**Result:**
The total contribution along this path is 1 (Step 2) + 4 (Step 4) = 5, which equals the common height of the two supports `{1, 4}` and `{2, 3}`. The memoized recursion compares this with every other path and returns the maximum, so `dfs(0, 0) = 5`.

## Python Solution

```python
class Solution:
    def tallestBillboard(self, rods: List[int]) -> int:
        """
        Find the tallest billboard that can be built using two supports of equal height.
      
        Args:
            rods: List of available rod lengths
          
        Returns:
            Maximum height of billboard (height of one support)
        """
        from functools import cache
        from math import inf
      
        @cache
        def dp(index: int, height_diff: int) -> int:
            """
            Dynamic programming function to find maximum height of shorter support.
          
            Args:
                index: Current rod index being considered
                height_diff: Absolute difference between two support heights
              
            Returns:
                Maximum achievable height of the shorter support from this state
            """
            # Base case: no more rods to consider
            if index >= len(rods):
                # Valid only if both supports have equal height
                return 0 if height_diff == 0 else -inf
          
            current_rod = rods[index]
          
            # Option 1: Skip current rod
            result = dp(index + 1, height_diff)
          
            # Option 2: Add current rod to the taller support
            result = max(result, dp(index + 1, height_diff + current_rod))
          
            # Option 3: Add current rod to the shorter support
            # The shorter support gains min(height_diff, current_rod) in effective height
            height_gain = min(height_diff, current_rod)
            new_diff = abs(current_rod - height_diff)
            result = max(result, dp(index + 1, new_diff) + height_gain)
          
            return result
      
        return dp(0, 0)
```

## Time and Space Complexity

**Time Complexity:** `O(n * S)` where `n` is the length of the rods array and `S` is the sum of all rod lengths.

The analysis follows from the memoized recursive solution:
- The function `dp(index, height_diff)` has two parameters: `index` ranging from `0` to `n`, and the difference `height_diff`, which is always non-negative and at most `S = sum(rods)`.
- Due to the `@cache` decorator, each unique state `(i, j)` is computed only once.
- Total number of unique states: `(n + 1) * (S + 1) = O(n * S)`
- Each state performs constant time operations (three recursive calls with memoization).
- Therefore, total time complexity is `O(n * S)`.

**Space Complexity:** `O(n * S)`

The space complexity comes from:
- The memoization cache stores up to `O(n * S)` states.
- The recursion stack depth is at most `n + 1` (with `n <= 20`, well within Python's recursion limit), contributing `O(n)` to space.
- Since `O(n * S) > O(n)`, the overall space complexity is `O(n * S)`.

Note: In practice, not all states `(i, j)` will be reached, so the actual complexity might be lower, but the worst-case remains `O(n * S)` for both time and space.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding the State Representation**

**Pitfall:** Many developers incorrectly interpret `j` (height_diff) as the actual heights of the two supports, leading to confusion about what the function returns.

**Issue:** The state `dfs(i, j)` doesn't track individual support heights but rather their difference. This abstraction is crucial for reducing the state space from O(n * S * S) to O(n * S).

**Solution:** Think of `j` as always representing |height_of_taller - height_of_shorter|. The function returns the additional height that can be added to the shorter support while maintaining or achieving equality.

### 2. **Incorrect Calculation When Adding to Shorter Support**

**Pitfall:** Writing `dfs(i + 1, abs(rods[i] - j)) + rods[i]` instead of `dfs(i + 1, abs(rods[i] - j)) + min(j, rods[i])`.

**Issue:** When adding a rod to the shorter support, only the portion that contributes to equalizing the heights counts toward the final answer. If the rod is longer than the gap, only the gap amount contributes to the billboard height.

**Solution:** Always use `min(j, rods[i])` to represent the actual contribution to the final height:
```python
# Correct: only count the height that helps equalize
result = max(result, dfs(i + 1, abs(rods[i] - j)) + min(j, rods[i]))
```

### 3. **Not Handling Negative Infinity Properly**

**Pitfall:** Using a small negative number like `-1` or `-10000` instead of negative infinity for invalid states.

**Issue:** An invalid path still collects height gains before it reaches the base case. If the sentinel is not small enough, those gains can outweigh it. For `rods = [2, 5]`, no equal split exists, so the answer is 0. With `-1` as the sentinel, the path "2 on one side, 5 on the other" gains `min(2, 5) = 2` and ends at difference 3, giving `2 + (-1) = 1`, so the function returns 1. A sentinel works only if it is below minus the largest possible total gain (at most `sum(rods) / 2`, which is 2500 under the constraints), so `-10000` happens to be safe here, while `-1` is not.

**Solution:** Use `-inf` from the math module, which is safe for any input:
```python
from math import inf
# Base case for invalid state
if index >= len(rods):
    return 0 if height_diff == 0 else -inf
```

### 4. **Forgetting to Consider the "Skip" Option**

**Pitfall:** Only considering adding rods to either support, forgetting that skipping a rod is also valid.

**Issue:** The optimal solution might require leaving some rods unused. For example, with rods `[1, 2, 3, 6]`, the best split is `{1, 2, 3}` and `{6}` for height 6, which uses every rod, but with rods `[1, 2, 3, 7]` no split uses rod 7, and the answer 3 comes from `{1, 2}` and `{3}` with rod 7 skipped. Without the skip option, the recursion is forced to place rod 7 on some support, no path ends with equal heights, and it returns `-inf` instead of 3.

**Solution:** Always include the skip option as the first choice:
```python
# Option 1: Skip current rod (always consider this first)
result = dp(index + 1, height_diff)
```

### 5. **Cache Key Collision with Negative Differences**

**Pitfall:** Not using absolute values for height differences, allowing both positive and negative differences in the cache.

**Issue:** States like `(i, 3)` and `(i, -3)` represent the same configuration (one support is 3 units taller) but would be cached separately, wasting memory and computation.

**Solution:** Always maintain `height_diff` as a non-negative value using `abs()`:
```python
# When adding to shorter support, ensure new difference is absolute
new_diff = abs(current_rod - height_diff)
result = max(result, dp(index + 1, new_diff) + height_gain)
```

