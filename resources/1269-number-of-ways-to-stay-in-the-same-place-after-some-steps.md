# 1269. Number of Ways to Stay in the Same Place After Some Steps

> Source: https://algo.monster/liteproblems/1269

## Problem Statement

You start with a pointer at index `0` in an array of size `arrLen`. At each step, you can perform one of three actions:
- Move the pointer 1 position to the left
- Move the pointer 1 position to the right  
- Stay in the same position

The pointer must always remain within the array bounds (cannot go to negative indices or indices >= `arrLen`).

Given two integers:
- `steps`: the exact number of steps you must take
- `arrLen`: the size of the array

Your task is to find the number of different ways to move the pointer such that after exactly `steps` steps, the pointer returns to index `0`.

Since the answer can be very large, return it modulo `10^9 + 7`.

For example, if `steps = 3` and `arrLen = 2`, you could:
1. Stay at 0, stay at 0, stay at 0 (one way)
2. Go right to 1, go left to 0, stay at 0 (another way)
3. Stay at 0, go right to 1, go left to 0 (another way)
4. Go right to 1, stay at 1, go left to 0 (another way)

## Intuition

The key insight is that this is a path counting problem where we need to return to the starting position after a fixed number of steps. This naturally suggests a [dynamic programming](/problems/dynamic_programming_intro) approach.

First, let's think about the constraints. With `steps` limited to 500, even if we move right at every step, we can only reach position 500. This means we don't need to consider positions beyond `min(arrLen, steps)` - any position further than `steps` away from 0 is unreachable since we wouldn't have enough steps to get there and return.

The problem has optimal substructure: the number of ways to reach position 0 in `j` steps from position `i` depends on the number of ways to reach position 0 in `j-1` steps from positions `i-1`, `i`, and `i+1` (corresponding to moving right, staying, or moving left in the current step).

We can think of this recursively: if we're at position `i` with `j` steps remaining, we can:
- Move left to position `i-1` and solve for `j-1` remaining steps
- Stay at position `i` and solve for `j-1` remaining steps  
- Move right to position `i+1` and solve for `j-1` remaining steps

The base cases are:
- If we're at position 0 with 0 steps remaining, we've found one valid way (return 1)
- If we go out of bounds or run out of steps at a non-zero position, this path is invalid (return 0)
- If the position is farther from 0 than the remaining steps, it's impossible to return (return 0)

Since the same state `(position, remaining_steps)` can be reached through different paths, we use memoization to avoid recalculating the same subproblems repeatedly. This transforms what would be an exponential time solution into a polynomial one.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a top-down [dynamic programming](/problems/dynamic_programming_intro) approach with memoization using Python's `@cache` decorator.

**Function Definition:**
The core function `dfs(i, j)` represents the number of ways to return to position 0 when currently at position `i` with `j` steps remaining.

**Base Cases:**
1. **Invalid states**: If `i > j` (position is farther than remaining steps can reach), or `i >= arrLen` (out of right boundary), or `i < 0` (out of left boundary), or `j < 0` (negative steps), return 0.
2. **Success state**: If `i == 0` and `j == 0`, we've successfully returned to position 0 with exactly the required steps, return 1.

**Recursive Transition:**
For each valid state `(i, j)`, we calculate the sum of ways from three possible moves:
```python
ans = 0
for k in range(-1, 2):  # k can be -1, 0, or 1
    ans += dfs(i + k, j - 1)
    ans %= mod
```
- `k = -1`: Move left to position `i - 1`
- `k = 0`: Stay at position `i`
- `k = 1`: Move right to position `i + 1`

Each move consumes one step, so we recursively call with `j - 1` remaining steps.

**Modulo Operation:**
Since the answer can be large, we apply modulo `10^9 + 7` after each addition to prevent integer overflow.

**Memoization:**
The `@cache` decorator automatically memoizes the results, storing computed values for each unique `(i, j)` pair. This prevents redundant calculations when the same state is reached through different paths.

**Initial Call:**
The solution starts with `dfs(0, steps)`, meaning we begin at position 0 with all `steps` available.

**Time Complexity:** `O(steps × min(arrLen, steps))` as we compute each unique state at most once.
**Space Complexity:** `O(steps × min(arrLen, steps))` for the memoization cache.

## Example Walkthrough

Let's walk through a small example with `steps = 4` and `arrLen = 3`.

We need to find the number of ways to start at index 0, take exactly 4 steps, and return to index 0.

Starting with `dfs(0, 4)` (at position 0 with 4 steps remaining):

**Step 1: From dfs(0, 4)**
We can make three moves:
- Move left: `dfs(-1, 3)` → returns 0 (out of bounds)
- Stay: `dfs(0, 3)` → need to compute
- Move right: `dfs(1, 3)` → need to compute

**Step 2: Computing dfs(0, 3)**
From position 0 with 3 steps:
- Move left: `dfs(-1, 2)` → returns 0 (out of bounds)
- Stay: `dfs(0, 2)` → need to compute
- Move right: `dfs(1, 2)` → need to compute

**Step 3: Computing dfs(1, 3)**
From position 1 with 3 steps:
- Move left: `dfs(0, 2)` → need to compute (or use memoized value)
- Stay: `dfs(1, 2)` → need to compute
- Move right: `dfs(2, 2)` → need to compute

**Step 4: Computing dfs(0, 2)**
From position 0 with 2 steps:
- Move left: `dfs(-1, 1)` → returns 0 (out of bounds)
- Stay: `dfs(0, 1)` → need to compute
- Move right: `dfs(1, 1)` → need to compute

**Step 5: Computing dfs(1, 2)**
From position 1 with 2 steps:
- Move left: `dfs(0, 1)` → need to compute (or use memoized value)
- Stay: `dfs(1, 1)` → need to compute
- Move right: `dfs(2, 1)` → returns 0 (position 2 > remaining steps 1, impossible to return)

**Step 6: Computing base cases with 1 step remaining**
- `dfs(0, 1)`: Can only stay → `dfs(0, 0)` = 1 ✓
- `dfs(1, 1)`: Can only move left → `dfs(0, 0)` = 1 ✓

**Step 7: Backtracking with computed values**
- `dfs(0, 2)` = 0 + 1 + 1 = 2 (from steps: left=0, stay=1, right=1)
- `dfs(1, 2)` = 1 + 1 + 0 = 2 (from steps: left=1, stay=1, right=0)
- `dfs(2, 2)` = 1 + 0 + 0 = 1 (can only move left then left again)

Continuing this process:
- `dfs(0, 3)` = 0 + 2 + 2 = 4
- `dfs(1, 3)` = 2 + 2 + 1 = 5

Finally:
- `dfs(0, 4)` = 0 + 4 + 5 = 9

So there are **9 different ways** to return to position 0 after exactly 4 steps.

Some example paths:
1. Stay → Stay → Stay → Stay
2. Right → Left → Stay → Stay
3. Right → Stay → Left → Stay
4. Right → Right → Left → Left
5. Stay → Right → Left → Stay
(and 4 more...)

## Python Solution

```python
class Solution:
    def numWays(self, steps: int, arrLen: int) -> int:
        """
        Count the number of ways to stay at position 0 after 'steps' moves.
        At each step, we can move left (-1), stay (0), or move right (+1).
      
        Args:
            steps: Total number of steps to take
            arrLen: Length of the array (positions are 0 to arrLen-1)
      
        Returns:
            Number of ways to end at position 0, modulo 10^9 + 7
        """
        from functools import cache
      
        MOD = 10**9 + 7
      
        @cache
        def dfs(position: int, remaining_steps: int) -> int:
            """
            Dynamic programming function to count valid paths.
          
            Args:
                position: Current position in the array
                remaining_steps: Number of steps left to take
          
            Returns:
                Number of ways to reach position 0 with exactly 0 steps remaining
            """
            # Base case: out of bounds or negative steps
            if position < 0 or position >= arrLen or remaining_steps < 0:
                return 0
          
            # Base case: too far from origin to return in remaining steps
            if position > remaining_steps:
                return 0
          
            # Success case: at position 0 with no steps remaining
            if position == 0 and remaining_steps == 0:
                return 1
          
            # Try all three possible moves: left, stay, right
            ways = 0
            for move in [-1, 0, 1]:
                ways += dfs(position + move, remaining_steps - 1)
                ways %= MOD
          
            return ways
      
        # Start at position 0 with 'steps' moves available
        return dfs(0, steps)
```

## Time and Space Complexity

The time complexity is `O(steps × min(steps, arrLen))`, and the space complexity is `O(steps × min(steps, arrLen))`.

The analysis can be refined to `O(steps²)` for both time and space complexity when considering the practical constraints. Here's why:

For the recursive function `dfs(i, j)`:
- Parameter `j` ranges from `0` to `steps`
- Parameter `i` represents the position in the array, ranging from `0` to `arrLen - 1`
- However, due to the nature of the problem, we can only move at most `steps` distance from the starting position `0`, so the effective range of `i` is bounded by `min(steps, arrLen - 1)`

Since we start at position `0` and have `steps` moves, with each move allowing us to go left, right, or stay, the maximum distance we can reach from origin is `steps`. Therefore:
- The number of unique states is `O(steps × min(steps, arrLen))`
- When `arrLen > steps`, the effective positions we can reach are limited to `[0, steps]`, making it `O(steps²)`
- When `arrLen ≤ steps`, we have `O(steps × arrLen)` states

The reference answer simplifies this to `O(steps × steps)` because in most practical scenarios, the positions we can actually reach are limited by the number of steps, effectively making the complexity `O(steps²)` for both time and space.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Optimizing the Search Space**
A critical optimization that's easy to miss is that we don't need to consider positions beyond `min(arrLen, steps // 2 + 1)`. Since we start at position 0 and must return to 0, the maximum distance we can travel is `steps // 2` (go out and come back). Many solutions fail to include this optimization, leading to unnecessary computation and potential timeout for large inputs.

**Problem Code:**
```python
@cache
def dfs(position: int, remaining_steps: int) -> int:
    if position < 0 or position >= arrLen or remaining_steps < 0:
        return 0
    # ... rest of the code
```

**Solution:**
```python
# Optimize the array length to avoid unnecessary states
max_position = min(arrLen, steps // 2 + 1)

@cache
def dfs(position: int, remaining_steps: int) -> int:
    if position < 0 or position >= max_position or remaining_steps < 0:
        return 0
    # ... rest of the code
```

### 2. **Forgetting to Apply Modulo Consistently**
Another common mistake is forgetting to apply the modulo operation after accumulating results, which can cause integer overflow in languages with fixed integer sizes or incorrect results.

**Problem Code:**
```python
ways = 0
for move in [-1, 0, 1]:
    ways += dfs(position + move, remaining_steps - 1)
return ways  # Forgot to apply modulo!
```

**Solution:**
```python
ways = 0
for move in [-1, 0, 1]:
    ways += dfs(position + move, remaining_steps - 1)
    ways %= MOD  # Apply modulo after each addition
return ways
```

### 3. **Incorrect Base Case Ordering**
Checking base cases in the wrong order can lead to incorrect results. The "too far to return" check (`position > remaining_steps`) should come after boundary checks to avoid index errors.

**Problem Code:**
```python
def dfs(position: int, remaining_steps: int) -> int:
    # Checking distance before boundaries - could cause issues
    if position > remaining_steps:
        return 0
    if position < 0 or position >= arrLen:
        return 0
```

**Solution:**
```python
def dfs(position: int, remaining_steps: int) -> int:
    # Check boundaries first
    if position < 0 or position >= arrLen or remaining_steps < 0:
        return 0
    # Then check if too far to return
    if position > remaining_steps:
        return 0
```

### 4. **Memory Limit Issues with Large Inputs**
For very large values of `steps` and `arrLen`, the memoization cache can grow too large. Using the optimization from pitfall #1 is crucial to prevent this.

**Complete Optimized Solution:**
```python
class Solution:
    def numWays(self, steps: int, arrLen: int) -> int:
        from functools import cache
      
        MOD = 10**9 + 7
        # Key optimization: limit search space
        max_position = min(arrLen, steps // 2 + 1)
      
        @cache
        def dfs(position: int, remaining_steps: int) -> int:
            # Boundary and validity checks
            if position < 0 or position >= max_position or remaining_steps < 0:
                return 0
          
            # Optimization: impossible to return to 0
            if position > remaining_steps:
                return 0
          
            # Success case
            if position == 0 and remaining_steps == 0:
                return 1
          
            # Try all three moves
            ways = 0
            for move in [-1, 0, 1]:
                ways += dfs(position + move, remaining_steps - 1)
                ways %= MOD
          
            return ways
      
        return dfs(0, steps)
```

