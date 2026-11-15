# 1553. Minimum Number of Days to Eat N Oranges

> Source: https://algo.monster/liteproblems/1553

## Problem Statement

You have `n` oranges and want to eat all of them in the minimum number of days. Each day, you can choose exactly one of the following actions:

1. **Eat one orange** - Reduce the number of oranges by 1
2. **Eat half of the oranges** - If the current number of oranges `n` is divisible by 2, you can eat `n/2` oranges (leaving `n/2` oranges remaining)
3. **Eat two-thirds of the oranges** - If the current number of oranges `n` is divisible by 3, you can eat `2*(n/3)` oranges (leaving `n/3` oranges remaining)

Given an integer `n` representing the initial number of oranges, return the minimum number of days needed to eat all `n` oranges.

For example, if you have 10 oranges:
- Day 1: Eat 1 orange, leaving 9 oranges
- Day 2: Since 9 is divisible by 3, eat 6 oranges (2×3), leaving 3 oranges  
- Day 3: Since 3 is divisible by 3, eat 2 oranges, leaving 1 orange
- Day 4: Eat the last orange

This would take 4 days total.

The challenge is to find the optimal sequence of actions that minimizes the total number of days needed to consume all oranges.

## Intuition

The key insight is that eating oranges one by one is inefficient when we have large numbers. Instead, we should use the division operations (dividing by 2 or 3) as much as possible since they allow us to reduce the number of oranges dramatically in a single day.

Consider if we have 100 oranges. Eating them one by one would take 100 days. But if we first eat 1 orange to get to 99, then eat `2*(99/3) = 66` oranges to get to 33, we've eliminated 67 oranges in just 2 days!

This suggests a greedy strategy: we want to reach numbers divisible by 2 or 3 as quickly as possible, then use the division operations. For any number `n`:
- To make it divisible by 2, we need `n % 2` days of eating one orange
- To make it divisible by 3, we need `n % 3` days of eating one orange

Once we reach a number divisible by 2 or 3, we can perform the division operation in 1 day, reducing our problem to a smaller instance: `n/2` or `n/3` oranges.

This naturally leads to a recursive approach where for each `n`, we consider:
1. Eat `n % 2` oranges one by one to make it divisible by 2, then divide by 2 (total: `n % 2 + 1` days plus the days needed for `n/2`)
2. Eat `n % 3` oranges one by one to make it divisible by 3, then divide by 3 (total: `n % 3 + 1` days plus the days needed for `n/3`)

We choose whichever path gives us the minimum total days. The base cases are when `n < 2`, where we simply need `n` days to eat the remaining oranges.

Since the same values of `n` might be reached through different paths, [memoization](/problems/memoization_intro) helps avoid recalculating the same subproblems, making the solution efficient.

**Learn more about [Memoization](/problems/memoization_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a **[memoization](/problems/memoization_intro) search** (also known as top-down [dynamic programming](/problems/dynamic_programming_intro)) to efficiently solve this problem.

**Implementation Details:**

1. **Main Function Structure**: We define a recursive function `dfs(n)` that returns the minimum number of days needed to eat `n` oranges.

2. **Base Case**: When `n < 2`:
   - If `n = 0`, we need 0 days (no oranges left)
   - If `n = 1`, we need 1 day (eat the last orange)
   - So we simply return `n`

3. **Recursive Case**: For `n >= 2`, we calculate:
   ```python
   1 + min(n % 2 + dfs(n // 2), n % 3 + dfs(n // 3))
   ```
 
   Breaking this down:
   - **Option 1**: `n % 2 + dfs(n // 2)`
     - First eat `n % 2` oranges one by one to make `n` divisible by 2
     - Then perform the division by 2 operation (1 additional day)
     - Recursively solve for `n // 2` oranges
 
   - **Option 2**: `n % 3 + dfs(n // 3)`
     - First eat `n % 3` oranges one by one to make `n` divisible by 3
     - Then perform the division by 3 operation (1 additional day)
     - Recursively solve for `n // 3` oranges
 
   - The `1 +` accounts for the day spent performing the division operation
   - We take the minimum of these two options

4. **[Memoization](/problems/memoization_intro)**: The `@cache` decorator (from Python's functools) automatically memoizes the function results. This means:
   - Each unique value of `n` is computed only once
   - Subsequent calls with the same `n` return the cached result immediately
   - This prevents exponential time complexity from redundant calculations

**Why This Works:**

The algorithm exploits the fact that division operations (by 2 or 3) are much more efficient than eating oranges one by one. By always choosing the path that minimizes the total days (including the days needed to reach a divisible number), we guarantee finding the optimal solution.

The [memoization](/problems/memoization_intro) ensures that even though the recursion tree might revisit the same values of `n` through different paths, we only compute each value once, giving us an efficient solution with time complexity roughly O(log²n) for the number of unique states visited.

## Example Walkthrough

Let's trace through the algorithm with **n = 10 oranges** to see how the memoized recursion finds the optimal solution.

**Starting with dfs(10):**

Since 10 ≥ 2, we calculate:
- Option 1 (divide by 2): `10 % 2 + dfs(10 // 2)` = `0 + dfs(5)`
- Option 2 (divide by 3): `10 % 3 + dfs(10 // 3)` = `1 + dfs(3)`

We need to evaluate both dfs(5) and dfs(3).

**Computing dfs(5):**
- Option 1: `5 % 2 + dfs(5 // 2)` = `1 + dfs(2)`
- Option 2: `5 % 3 + dfs(5 // 3)` = `2 + dfs(1)`

**Computing dfs(3):**
- Option 1: `3 % 2 + dfs(3 // 2)` = `1 + dfs(1)`
- Option 2: `3 % 3 + dfs(3 // 3)` = `0 + dfs(1)`

**Computing dfs(2):**
- Option 1: `2 % 2 + dfs(2 // 2)` = `0 + dfs(1)`
- Option 2: `2 % 3 + dfs(2 // 3)` = `2 + dfs(0)`

**Base cases:**
- dfs(1) = 1 (eat the last orange)
- dfs(0) = 0 (no oranges left)

**Working backwards:**
- dfs(2) = 1 + min(0 + 1, 2 + 0) = 1 + min(1, 2) = **2**
- dfs(3) = 1 + min(1 + 1, 0 + 1) = 1 + min(2, 1) = **2**
- dfs(5) = 1 + min(1 + 2, 2 + 1) = 1 + min(3, 3) = **4**
- dfs(10) = 1 + min(0 + 4, 1 + 2) = 1 + min(4, 3) = **4**

**The optimal path for 10 oranges takes 4 days:**
1. Day 1: Eat 1 orange (10 → 9)
2. Day 2: Divide by 3, eat 6 oranges (9 → 3)
3. Day 3: Divide by 3, eat 2 oranges (3 → 1)
4. Day 4: Eat the last orange (1 → 0)

Notice how memoization helps: if we later need dfs(3) or dfs(5) again in a different branch, we already have the answer cached!

## Python Solution

```python
from functools import cache
from typing import Optional


class Solution:
    def minDays(self, n: int) -> int:
        """
        Find the minimum number of days to reduce n to 0.
      
        Operations allowed:
        - Subtract 1 from n
        - Divide n by 2 (if n is even)
        - Divide n by 3 (if n is divisible by 3)
      
        Args:
            n: The starting integer value
          
        Returns:
            Minimum number of days/operations needed
        """
      
        @cache
        def dfs(current: int) -> int:
            """
            Recursively find minimum operations using dynamic programming.
          
            The key insight: Instead of subtracting 1 multiple times,
            we can group operations by calculating the remainder when
            dividing by 2 or 3, then perform the division.
          
            Args:
                current: Current value to reduce to 0
              
            Returns:
                Minimum operations needed from current state
            """
            # Base cases: 0 needs 0 operations, 1 needs 1 operation
            if current < 2:
                return current
          
            # Try two strategies:
            # Strategy 1: Reduce to nearest multiple of 2, then divide by 2
            # - current % 2: operations to make current divisible by 2
            # - 1: the division operation itself
            # - dfs(current // 2): operations for the resulting value
            strategy_divide_by_2 = current % 2 + 1 + dfs(current // 2)
          
            # Strategy 2: Reduce to nearest multiple of 3, then divide by 3
            # - current % 3: operations to make current divisible by 3
            # - 1: the division operation itself
            # - dfs(current // 3): operations for the resulting value
            strategy_divide_by_3 = current % 3 + 1 + dfs(current // 3)
          
            # Return the minimum of both strategies
            return min(strategy_divide_by_2, strategy_divide_by_3)
      
        # Start the recursive calculation from n
        return dfs(n)
```

## Time and Space Complexity

The time complexity is `O(log^2 n)`, and the space complexity is `O(log^2 n)`.

**Time Complexity Analysis:**
The algorithm uses memoization to solve the problem recursively. At each step, we either:
- Reduce `n` to `n // 2` (after handling `n % 2` operations)
- Reduce `n` to `n // 3` (after handling `n % 3` operations)

The key insight is that the problem space forms a tree where each node `n` can branch to `n // 2` and `n // 3`. The depth of this tree is `O(log n)` because we're repeatedly dividing by 2 or 3. 

Due to memoization with `@cache`, each unique value is computed only once. The number of unique subproblems is bounded by the number of distinct values we can reach by repeatedly dividing by 2 and 3. This creates approximately `O(log^2 n)` unique states, as we can reach a value through different combinations of dividing by 2 and 3.

Therefore, the time complexity is `O(log^2 n)`.

**Space Complexity Analysis:**
The space complexity consists of:
1. The recursion call stack depth: `O(log n)` in the worst case
2. The memoization cache storing all unique subproblems: `O(log^2 n)` entries

The dominant factor is the cache storage, making the overall space complexity `O(log^2 n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Naive BFS/DFS Without Memoization Leading to TLE**

**The Pitfall:**
Many developers initially attempt a straightforward BFS or DFS approach that explores all three options (eat 1, divide by 2, divide by 3) at each step without memoization. This creates an exponential search space.

```python
# WRONG: Naive recursive approach without memoization
def minDays_wrong(n):
    if n == 0:
        return 0
    if n == 1:
        return 1
  
    result = 1 + minDays_wrong(n - 1)  # Always try eating 1
    if n % 2 == 0:
        result = min(result, 1 + minDays_wrong(n // 2))
    if n % 3 == 0:
        result = min(result, 1 + minDays_wrong(n // 3))
  
    return result
```

This approach will timeout for large values of `n` because it recalculates the same subproblems multiple times and explores the "eat 1 orange" path too extensively.

**The Solution:**
Add memoization and avoid exploring the "eat 1" option directly in recursion. Instead, only eat individual oranges to reach the nearest multiple of 2 or 3:

```python
from functools import cache

@cache
def minDays(n):
    if n < 2:
        return n
    # Only consider paths through division by 2 or 3
    return 1 + min(n % 2 + minDays(n // 2), 
                   n % 3 + minDays(n // 3))
```

### 2. **Using Bottom-Up DP with Large Arrays**

**The Pitfall:**
Attempting to solve this with traditional bottom-up dynamic programming by creating a DP array of size `n+1`:

```python
# WRONG: Memory limit exceeded for large n
def minDays_wrong(n):
    if n <= 1:
        return n
  
    dp = [0] * (n + 1)  # This will cause MLE for n = 10^9
    dp[1] = 1
  
    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + 1
        if i % 2 == 0:
            dp[i] = min(dp[i], dp[i // 2] + 1)
        if i % 3 == 0:
            dp[i] = min(dp[i], dp[i // 3] + 1)
  
    return dp[n]
```

For large values of `n` (up to 2×10^9 in this problem), this approach will cause memory limit exceeded.

**The Solution:**
Use top-down memoization which only stores the states that are actually visited. The number of unique states is much smaller (approximately O(log²n)) compared to O(n).

### 3. **Incorrect Understanding of the Operations**

**The Pitfall:**
Misunderstanding what "eat half" or "eat two-thirds" means. Some might think:
- Eating half means reducing n to n/2 (incorrect)
- Eating two-thirds means reducing n to n/3 (correct, but confusing wording)

The problem states that when n is divisible by 3, you eat 2n/3 oranges, leaving n/3 oranges. This is equivalent to dividing n by 3.

**The Solution:**
Clearly understand that:
- Operation 2: n → n/2 (when n is even)
- Operation 3: n → n/3 (when n is divisible by 3)
- Operation 1: n → n-1 (always available)

### 4. **Not Handling the Modulo Operations Correctly**

**The Pitfall:**
Forgetting to account for the days needed to make n divisible by 2 or 3:

```python
# WRONG: Doesn't account for days to reach divisible numbers
@cache
def minDays_wrong(n):
    if n < 2:
        return n
    return 1 + min(minDays_wrong(n // 2), minDays_wrong(n // 3))
```

This incorrectly assumes we can always divide by 2 or 3, even when n is not divisible by these numbers.

**The Solution:**
Always include the modulo operations to count the days needed to eat oranges one by one until reaching a divisible number:

```python
return 1 + min(n % 2 + minDays(n // 2), 
               n % 3 + minDays(n // 3))
```

This ensures we count all the days correctly: the days to reach a divisible number plus one day for the division operation itself.

