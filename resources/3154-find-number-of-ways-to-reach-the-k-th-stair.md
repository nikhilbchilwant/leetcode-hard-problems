# 3154. Find Number of Ways to Reach the K-th Stair

> Source: https://algo.monster/liteproblems/3154

## Problem Statement

You have an infinite staircase where stairs are numbered starting from 0. Alice starts on stair 1 and wants to reach stair `k`.

Alice has a variable called `jump` that starts at 0. From any stair `i`, she can perform one of two operations:

1. **Go down one stair** to stair `i - 1`. This operation has restrictions:
   - Cannot be used on stair 0 (since there's no stair -1)
   - Cannot be used twice in a row (consecutively)

2. **Jump up** to stair `i + 2^jump`. After using this operation, `jump` increases by 1.
   - First jump: move up by `2^0 = 1` stair, then `jump` becomes 1
   - Second jump: move up by `2^1 = 2` stairs, then `jump` becomes 2
   - Third jump: move up by `2^2 = 4` stairs, then `jump` becomes 3
   - And so on...

The task is to find the **total number of different ways** Alice can reach stair `k` using any sequence of these operations.

Important notes:
- Alice can reach stair `k` multiple times in the same sequence, and each time counts toward the total
- She can continue performing operations even after reaching stair `k` and potentially return to it again
- The input `k` is a non-negative integer

For example, if `k = 1`, Alice is already at stair 1, so that's one way. She could also jump up (to stair 2), go down (to stair 1), achieving another way to reach stair 1.

## Intuition

The key insight is that Alice's movement follows a very specific pattern due to the constraints. Since she can't go down twice in a row, every downward move must be followed by an upward jump. This creates a decision tree where at each position, we need to track:

1. **Current position** `i` - where Alice is on the staircase
2. **Last operation type** `j` - whether the last move was down (1) or up (0)
3. **Jump counter** `jump` - how many upward jumps have been made

The reason we need to track the last operation is crucial: if we just went down (`j = 1`), we cannot go down again immediately. This constraint shapes our entire exploration strategy.

Notice that the upward jumps grow exponentially (`2^0, 2^1, 2^2, ...`). This means Alice can quickly reach very high stairs. However, there's an important boundary to consider: if Alice is already at a position `i > k + 1`, she cannot reach stair `k` again. Why? Because:
- She can only go down by 1 at most (to `i - 1`)
- Even if at `i = k + 2`, going down gives us `k + 1`, not `k`
- From `k + 1`, she must jump up (can't go down twice), which takes her even further from `k`

This gives us a natural pruning condition: stop exploring when `i > k + 1`.

The recursive structure emerges naturally:
- From any valid position, we can always jump up (adding `2^jump` to our position)
- We can only go down if we're not at stair 0 and our last move wasn't already down

Each time we land exactly on stair `k`, we count it as one valid way. But we don't stop there - Alice can continue moving and potentially return to `k` again through a different sequence of moves.

[Memoization](/problems/memoization_intro) is essential here because the same state `(i, j, jump)` can be reached through different sequences of operations. By caching these results, we avoid recalculating the number of ways from the same state multiple times.

**Learn more about [Memoization](/problems/memoization_intro), [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The solution uses **[memoization](/problems/memoization_intro) search ([dynamic programming](/problems/dynamic_programming_intro) with recursion)** to explore all possible paths Alice can take to reach stair `k`.

We implement a recursive function `dfs(i, j, jump)` where:
- `i` represents the current stair position
- `j` indicates whether the last operation was going down (1) or not (0)
- `jump` tracks how many upward jumps have been performed

**Base Case and Pruning:**
The function first checks if `i > k + 1`. If true, it returns 0 because it's impossible to reach stair `k` from this position (as explained in the intuition).

**Counting Valid Paths:**
When we're at position `i`, we check if `i == k`. If yes, we've found one way to reach stair `k`, so we initialize `ans = 1`. Otherwise, `ans = 0`.

**Recursive Exploration:**
From the current position, we explore two possible operations:

1. **Going Down:** If `i > 0` (not at stair 0) and `j == 0` (last operation wasn't going down), we can go down one stair. We recursively call `dfs(i - 1, 1, jump)` where:
   - New position is `i - 1`
   - Set `j = 1` to mark that we just went down
   - Keep `jump` unchanged

2. **Jumping Up:** We can always jump up (no restrictions). We recursively call `dfs(i + (1 << jump), 0, jump + 1)` where:
   - New position is `i + 2^jump` (using bit shift `1 << jump` for efficiency)
   - Set `j = 0` to mark that we just jumped up
   - Increment `jump` by 1

**[Memoization](/problems/memoization_intro):**
The `@cache` decorator automatically memoizes the function results. When `dfs(i, j, jump)` is called with the same parameters again, it returns the cached result instead of recalculating.

**Starting Point:**
The solution begins by calling `dfs(1, 0, 0)`:
- Start at stair 1
- No previous down operation (`j = 0`)
- No jumps performed yet (`jump = 0`)

The function accumulates all possible ways by adding up the results from both recursive branches, ultimately returning the total number of ways to reach stair `k`.

## Example Walkthrough

Let's trace through the solution for `k = 2`.

Starting at stair 1 with `jump = 0` and no previous down operation:

**Initial call: `dfs(1, 0, 0)`**
- Currently at stair 1, which is not equal to k=2, so `ans = 0`
- We can explore two operations:
  1. Jump up: go to stair 1 + 2^0 = 2, calling `dfs(2, 0, 1)`
  2. Go down: go to stair 0, calling `dfs(0, 1, 0)`

**Branch 1: `dfs(2, 0, 1)`** (jumped to stair 2)
- Currently at stair 2, which equals k=2, so `ans = 1` (found one way!)
- We can explore two operations:
  1. Jump up: go to stair 2 + 2^1 = 4, calling `dfs(4, 0, 2)`
  2. Go down: go to stair 1, calling `dfs(1, 1, 1)`

**Branch 1.1: `dfs(4, 0, 2)`** (at stair 4)
- Position 4 > k+1 (which is 3), so return 0 immediately

**Branch 1.2: `dfs(1, 1, 1)`** (went down to stair 1)
- Currently at stair 1, not equal to k=2, so `ans = 0`
- Last operation was down (j=1), so we can't go down again
- Can only jump up: go to stair 1 + 2^1 = 3, calling `dfs(3, 0, 2)`

**Branch 1.2.1: `dfs(3, 0, 2)`** (at stair 3)
- Currently at stair 3, not equal to k=2, so `ans = 0`
- Can go down to stair 2, calling `dfs(2, 1, 2)`
- Can jump up to stair 3 + 2^2 = 7, calling `dfs(7, 0, 3)`

**Branch 1.2.1.1: `dfs(2, 1, 2)`** (went down to stair 2)
- Currently at stair 2, which equals k=2, so `ans = 1` (found another way!)
- Last operation was down, so can only jump up
- Jump to stair 2 + 2^2 = 6, calling `dfs(6, 0, 3)`
- This returns 0 (position > k+1)
- Total from this branch: 1

**Branch 1.2.1.2: `dfs(7, 0, 3)`**
- Position 7 > k+1, return 0

Going back up the call stack:
- `dfs(3, 0, 2)` returns 1 (from going down to stair 2)
- `dfs(1, 1, 1)` returns 1
- `dfs(2, 0, 1)` returns 1 + 0 + 1 = 2

**Branch 2: `dfs(0, 1, 0)`** (went down to stair 0)
- Currently at stair 0, not equal to k=2, so `ans = 0`
- Can't go down from stair 0 (would be negative)
- Can only jump up: go to stair 0 + 2^0 = 1, calling `dfs(1, 0, 1)`

This `dfs(1, 0, 1)` call explores similar paths but with jump counter at 1, finding additional ways to reach k=2.

The final answer accumulates all these different paths where Alice lands on stair 2.

## Python Solution

```python
class Solution:
    def waysToReachStair(self, k: int) -> int:
        from functools import cache
      
        @cache
        def dfs(current_stair: int, can_go_down: int, jump_power: int) -> int:
            """
            Dynamic programming function to count ways to reach stair k.
          
            Args:
                current_stair: Current position on the staircase
                can_go_down: Flag indicating if we can move down (0 = can move down, 1 = cannot)
                jump_power: Current power for the jump operation (jump size = 2^jump_power)
          
            Returns:
                Number of ways to reach stair k from current position
            """
            # Pruning: if current position is too far from target, no valid paths exist
            if current_stair > k + 1:
                return 0
          
            # Initialize answer: add 1 if we're currently at the target stair
            ways_count = int(current_stair == k)
          
            # Option 1: Move down by 1 stair (only if we're above stair 0 and allowed to go down)
            if current_stair > 0 and can_go_down == 0:
                # After moving down, set flag to 1 (cannot move down consecutively)
                ways_count += dfs(current_stair - 1, 1, jump_power)
          
            # Option 2: Jump up by 2^jump_power stairs
            # After jumping up, reset flag to 0 (can move down again) and increment jump power
            ways_count += dfs(current_stair + (1 << jump_power), 0, jump_power + 1)
          
            return ways_count
      
        # Start from stair 1, can move down initially, with jump power 0
        return dfs(1, 0, 0)
```

## Time and Space Complexity

The time complexity is `O(log^2 k)` and the space complexity is `O(log^2 k)`.

**Time Complexity Analysis:**
- The function uses memoization with `@cache` decorator to avoid redundant calculations
- The state space is determined by three parameters: `(i, j, jump)`
- The value of `i` is bounded by `k + 1` due to the pruning condition `if i > k + 1: return 0`
- The value of `j` can only be 0 or 1
- The key insight is that `jump` determines how we reach position `i`. Since we add `1 << jump` (which is `2^jump`) to position, to reach a position around `k`, we need at most `O(log k)` jumps
- For each jump value from 0 to `log k`, we can reach at most `O(log k)` different positions through combinations of operations
- The total number of unique states is `O(log k) × 2 × O(log k) = O(log^2 k)`
- Each state is computed once due to memoization, resulting in `O(log^2 k)` time complexity

**Space Complexity Analysis:**
- The space is primarily used by the memoization cache and the recursion call stack
- The memoization cache stores all unique states visited, which is `O(log^2 k)` as analyzed above
- The maximum recursion depth is `O(log k)` since we can have at most `O(log k)` jump operations in a path
- Therefore, the total space complexity is `O(log^2 k)` dominated by the cache storage

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Pruning Condition**
A critical pitfall is using an incorrect or overly aggressive pruning condition. Many might think to prune when `current_stair > k`, but this would miss valid paths where Alice goes above stair `k` and then comes back down.

**Why `k + 1` is the correct bound:**
- From position `k + 1`, Alice can reach `k` by going down once
- From position `k + 2` or higher, Alice cannot reach `k` using only down operations (limited to one consecutive down move)
- The minimum jump size keeps increasing (1, 2, 4, 8...), so once we're at `k + 2` or higher, we can never land exactly on `k` again

**Incorrect approach:**
```python
if current_stair > k:  # Too aggressive - misses valid paths!
    return 0
```

**Correct approach:**
```python
if current_stair > k + 1:  # Allows reaching k from k+1 by going down
    return 0
```

### 2. **Misunderstanding the "Can Go Down" Flag**
The flag parameter can be confusing. Some might interpret `can_go_down = 1` as "allowed to go down", but it actually means the opposite.

**Common mistake:**
```python
# Wrong interpretation
if current_stair > 0 and can_go_down == 1:  # Incorrect logic!
    ways_count += dfs(current_stair - 1, 0, jump_power)
```

**Correct implementation:**
```python
# can_go_down: 0 = allowed to go down, 1 = not allowed (just went down)
if current_stair > 0 and can_go_down == 0:
    ways_count += dfs(current_stair - 1, 1, jump_power)  # Set to 1 after going down
```

### 3. **Forgetting to Count Current Position**
A subtle pitfall is forgetting to count the current position when it equals `k`. Each time we reach stair `k`, it should be counted as one valid way, even if we continue exploring from there.

**Incorrect:**
```python
ways_count = 0
# Forgot to check if current_stair == k!
```

**Correct:**
```python
ways_count = int(current_stair == k)  # Count this visit if we're at k
```

### 4. **Integer Overflow with Large Jump Powers**
For large values of `jump_power`, calculating `2^jump_power` could theoretically cause issues. While Python handles big integers well, in other languages this could overflow.

**Potential issue in other languages:**
```python
# In languages with fixed integer sizes, this might overflow
current_stair + (1 << jump_power)
```

**Solution for other languages:**
- Add overflow checks or use appropriate data types (e.g., `long long` in C++)
- Consider early termination when jump size exceeds reasonable bounds

### 5. **Not Using Memoization**
Without memoization, the solution would have exponential time complexity due to recalculating the same states multiple times.

**Without memoization (inefficient):**
```python
def dfs(current_stair, can_go_down, jump_power):
    # No caching - recalculates same states repeatedly
    ...
```

**With memoization (efficient):**
```python
@cache  # Essential for avoiding timeout!
def dfs(current_stair, can_go_down, jump_power):
    ...
```

