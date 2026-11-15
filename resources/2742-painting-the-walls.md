# 2742. Painting the Walls

> Source: https://algo.monster/liteproblems/2742

## Problem Statement

You have `n` walls that need to be painted. You're given two arrays:
- `cost[i]`: the money required for a paid painter to paint wall `i`
- `time[i]`: the time units needed for a paid painter to paint wall `i`

You have two types of painters available:

1. **Paid Painter**: Takes `time[i]` units to paint wall `i` and charges `cost[i]` money
2. **Free Painter**: Can paint any wall in exactly 1 time unit for free, but can only work when the paid painter is busy

The key insight is that while the paid painter is working on one wall (taking `time[i]` units), the free painter can paint `time[i]` other walls during that same period.

For example, if the paid painter takes 3 time units to paint wall 0, during those 3 time units, the free painter can paint 3 other walls.

Your goal is to find the minimum total cost to paint all `n` walls.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization. The function `dfs(i, j)` represents:
- `i`: the current wall index being considered
- `j`: the number of walls the free painter can still paint (accumulated from previous paid painter jobs)

At each wall, you have two choices:
1. Use the paid painter: pay `cost[i]` and gain `time[i]` free painter slots
2. Use the free painter: pay nothing but consume 1 free painter slot (so `j` decreases by 1)

The base cases are:
- If remaining walls `(n - i)` ≤ available free painter time `j`, all remaining walls can be painted for free
- If `i ≥ n` and we still need to paint walls, return infinity (invalid state)

## Intuition

The key realization is that when a paid painter works on a wall for `time[i]` units, we're essentially "buying" `time[i]` units of free painter time. This transforms the problem into a resource allocation problem.

Think of it this way: every wall must be painted by someone. When we choose to use the paid painter on wall `i`, we're making a trade-off:
- We pay `cost[i]` money
- We occupy the paid painter for `time[i]` units
- During those `time[i]` units, the free painter can paint `time[i]` walls

This means using the paid painter on one wall can potentially cover multiple walls through the free painter's work during that time period.

The problem becomes: which walls should we assign to the paid painter such that:
1. The free painter has enough time to paint all remaining walls
2. The total cost is minimized

We can model this as a decision problem where at each wall, we decide whether to:
- Use the paid painter (gaining free painter time)
- Use the free painter (consuming previously gained time)

The state we need to track is:
- Which wall we're currently considering (`i`)
- How much "credit" of free painter time we've accumulated (`j`)

When `j` represents our accumulated free painter time, if we have `j` time units available and only `n - i` walls left to paint, and `j ≥ n - i`, then all remaining walls can be painted for free.

This naturally leads to a recursive solution with memoization, where we explore both choices at each wall and pick the minimum cost path. The beauty of this approach is that it automatically handles the constraint that the free painter can only work when the paid painter is busy - we're tracking this implicitly through the accumulated time `j`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization to solve this optimization problem. Let's break down the implementation:

**Function Definition:**
We define `dfs(i, j)` where:
- `i` represents the current wall index we're considering (0 to n-1)
- `j` represents the accumulated free painter time available

**Base Cases:**

1. **All remaining walls can be painted for free:** If `n - i <= j`, meaning the number of remaining walls is less than or equal to the available free painter time, we return `0` since all remaining walls can be painted by the free painter at no cost.

2. **Invalid state:** If `i >= n` (we've gone past all walls) but haven't satisfied the first condition, return infinity to indicate this path is invalid.

**Recursive Cases:**

For each wall `i`, we have two choices:

1. **Use the paid painter:** 
   - Cost: `cost[i]` 
   - Time gained for free painter: `time[i]`
   - Recursive call: `dfs(i + 1, j + time[i]) + cost[i]`
   - We move to the next wall and add `time[i]` to our free painter budget

2. **Use the free painter:**
   - Cost: `0`
   - Time consumed: `1` unit
   - Recursive call: `dfs(i + 1, j - 1)`
   - We move to the next wall and decrease our free painter budget by 1

The function returns the minimum of these two choices: `min(dfs(i + 1, j + time[i]) + cost[i], dfs(i + 1, j - 1))`

**Memoization:**
The `@cache` decorator is used to memoize the results. This prevents recalculating the same subproblems multiple times, reducing the time complexity from exponential to polynomial.

**Initial Call:**
We start with `dfs(0, 0)`, meaning we begin at wall 0 with no accumulated free painter time.

**Handling Negative Values:**
In the actual implementation for languages other than Python, since `j` can become negative (when we use more free painter time than we've accumulated), we need to add an offset of `n` to ensure array indices remain valid. Python handles negative dictionary keys naturally with the `@cache` decorator.

**Time and Space Complexity:**
- Time: `O(n²)` - we have at most `n * 2n` unique states to compute
- Space: `O(n²)` - for the memoization cache

## Example Walkthrough

Let's walk through a small example with `n = 4` walls, `cost = [1, 2, 3, 1]`, and `time = [3, 1, 1, 2]`.

We'll trace through the decision tree using `dfs(i, j)` where `i` is the current wall and `j` is accumulated free painter time.

**Starting at `dfs(0, 0)`** - Wall 0, no free time available:
- Option 1: Use paid painter on wall 0
  - Pay cost[0] = 1
  - Gain time[0] = 3 free painter slots
  - Recurse to `dfs(1, 3)` + 1
- Option 2: Use free painter (invalid since j = 0)
  - Would need `dfs(1, -1)` which returns ∞

**At `dfs(1, 3)`** - Wall 1, 3 free time units available:
- Check base case: remaining walls (4-1=3) ≤ free time (3)? Yes!
- Return 0 (all remaining walls can be painted free)

So path through wall 0 with paid painter costs: 1 + 0 = **1**

Let's verify another path to understand the trade-offs:

**Alternative: Start `dfs(0, 0)` but explore different choices:**

If we use paid painter on wall 0 (cost 1, gain 3 time), we get `dfs(1, 3)` = 0, total = 1

What if we tried using paid painter on wall 1 first?
- At `dfs(0, 0)`: must use paid (no free time)
- At `dfs(1, 3)`: 
  - Option 1: Use paid on wall 1 → `dfs(2, 4)` + 2
    - At `dfs(2, 4)`: remaining walls (2) ≤ free time (4), return 0
    - Total: 1 + 2 + 0 = 3
  - Option 2: Use free → `dfs(2, 2)` + 0
    - At `dfs(2, 2)`: remaining walls (2) ≤ free time (2), return 0
    - Total: 1 + 0 + 0 = 1

The algorithm correctly picks minimum cost = **1**.

**Key Insight from Example:**
Using the paid painter on wall 0 was optimal because:
- It has low cost (1) 
- It provides high time value (3 units)
- Those 3 time units exactly cover the 3 remaining walls

This demonstrates how the algorithm balances cost vs. time gained to find the optimal painting strategy.

## Python Solution

```python
class Solution:
    def paintWalls(self, cost: List[int], time: List[int]) -> int:
        from functools import cache
        from math import inf
      
        @cache
        def dp(wall_index: int, free_time: int) -> int:
            """
            Dynamic programming function to find minimum cost to paint walls.
          
            Args:
                wall_index: Current wall being considered (0-indexed)
                free_time: Amount of free time accumulated from paid painter
          
            Returns:
                Minimum cost to paint remaining walls
            """
            # Base case: If remaining walls can be painted with available free time
            if n - wall_index <= free_time:
                return 0
          
            # Base case: If we've considered all walls but still need to paint more
            if wall_index >= n:
                return inf
          
            # Two choices for current wall:
            # Option 1: Hire paid painter for current wall
            # - Move to next wall (wall_index + 1)
            # - Add time[wall_index] to free time (free painter can work during this time)
            # - Add cost[wall_index] to total cost
            hire_paid_painter = dp(wall_index + 1, free_time + time[wall_index]) + cost[wall_index]
          
            # Option 2: Use free painter for current wall (if free time available)
            # - Move to next wall (wall_index + 1)
            # - Decrease free time by 1 (free painter takes 1 unit of time per wall)
            use_free_painter = dp(wall_index + 1, free_time - 1)
          
            # Return minimum cost between the two options
            return min(hire_paid_painter, use_free_painter)
      
        # Initialize number of walls
        n = len(cost)
      
        # Start with wall 0 and 0 free time
        return dp(0, 0)
```

## Time and Space Complexity

The time complexity is `O(n^2)` and the space complexity is `O(n^2)`, where `n` is the length of the array.

**Time Complexity Analysis:**
- The recursive function `dfs(i, j)` has two parameters: `i` ranges from `0` to `n-1`, and `j` can range from approximately `-n` to `n` in the worst case
- Parameter `i` represents the current wall index (can take `n` different values)
- Parameter `j` represents the time balance. In the worst case, when all `time[i] = 1`, `j` increases by 1 each time we paint a wall, and decreases by 1 when we skip. This means `j` can range from about `-n` to `n`, giving us roughly `2n` possible values
- Due to memoization with `@cache`, each unique state `(i, j)` is computed only once
- Total number of unique states: `n × 2n = O(n^2)`
- Each state performs `O(1)` work (just comparing and returning minimum of two recursive calls)
- Therefore, time complexity is `O(n^2)`

**Space Complexity Analysis:**
- The `@cache` decorator stores all computed states in memory
- Maximum number of states that can be stored: `O(n^2)` (as analyzed above)
- The recursion depth in the worst case is `O(n)` when we always choose to skip walls
- Total space complexity: `O(n^2)` for the cache plus `O(n)` for the recursion stack
- Since `O(n^2) + O(n) = O(n^2)`, the overall space complexity is `O(n^2)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding the Free Painter's Working Constraint**

**Pitfall:** A common mistake is thinking the free painter can work independently at any time. Many incorrectly assume they can simply choose the cheapest walls for the paid painter and let the free painter handle the rest separately.

**Reality:** The free painter can ONLY work while the paid painter is actively painting. If the paid painter works for 3 time units on a wall, the free painter gets exactly those 3 time units to paint other walls.

**Wrong Approach Example:**
```python
# INCORRECT: Trying to greedily select walls
def paintWalls_wrong(cost, time):
    # This doesn't work because it doesn't respect the constraint
    walls_with_ratio = sorted(zip(cost, time), key=lambda x: x[0]/x[1])
    # ... attempting to greedily assign walls
```

**Correct Understanding:** The free painter's time is directly tied to and accumulated from the paid painter's work time.

### 2. **Incorrect Base Case Logic**

**Pitfall:** Writing the base case as `if i == n: return 0` without checking if all walls are actually painted.

**Problem:** This would return 0 even when we haven't accumulated enough free time to paint remaining walls.

**Wrong Implementation:**
```python
@cache
def dp(i, j):
    if i == n:  # WRONG: Doesn't check if all walls can be painted
        return 0
    # ...
```

**Correct Implementation:**
```python
@cache
def dp(i, j):
    if n - i <= j:  # Remaining walls can be painted with free time
        return 0
    if i >= n:  # Went past all walls but condition above not met
        return inf
    # ...
```

### 3. **Not Handling Negative Free Time States**

**Pitfall:** Assuming free time `j` will always be non-negative and not handling cases where we try to use more free painter time than available.

**Problem:** When we choose to use the free painter via `dp(i + 1, j - 1)`, `j` can become negative, which represents "borrowing" free time that must be "paid back" by hiring the paid painter later.

**Solution:** The algorithm naturally handles this because:
- Negative `j` values are valid states in the DP
- They represent a deficit that must be compensated
- The base case `n - i <= j` will be false for negative `j`, forcing the algorithm to hire paid painters

### 4. **Integer Overflow in Other Languages**

**Pitfall:** Using `Integer.MAX_VALUE` in languages like Java without careful handling can cause overflow when adding costs.

**Wrong (Java):**
```java
if (i >= n) return Integer.MAX_VALUE;  // Can overflow when adding cost[i]
```

**Correct (Java):**
```java
if (i >= n) return 1_000_000_000;  // Use a large but safe value
// OR
long result = dfs(i + 1, j + time[i]) + (long)cost[i];
return Math.min(result, dfs(i + 1, j - 1));
```

### 5. **Forgetting Memoization**

**Pitfall:** Implementing the recursive solution without memoization, leading to exponential time complexity.

**Impact:** Without memoization, the same subproblems are solved repeatedly, causing Time Limit Exceeded (TLE) for larger inputs.

**Solution:** Always use memoization (@cache in Python, HashMap in Java, or a 2D array if bounds are known).

