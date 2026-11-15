# 887. Super Egg Drop

> Source: https://algo.monster/liteproblems/887

## Problem Statement

You have `k` identical eggs and a building with `n` floors (numbered from 1 to n). There exists a critical floor `f` (where `0 ≤ f ≤ n`) with the following properties:
- Any egg dropped from a floor **higher** than `f` will **break**
- Any egg dropped from floor `f` or **below** will **not break**

In each move, you can:
1. Take an unbroken egg and drop it from any floor `x` (where `1 ≤ x ≤ n`)
2. If the egg breaks, you cannot use it again
3. If the egg doesn't break, you can reuse it in future moves

Your goal is to find the **minimum number of moves** required to determine the exact value of `f` **with certainty** (in the worst case).

For example, if you have 2 eggs and 10 floors:
- You need to find a strategy that guarantees finding the critical floor `f`
- The strategy should minimize the maximum number of drops needed in the worst-case scenario
- You must be able to determine `f` exactly, even if all but one egg breaks

The challenge is to develop an optimal strategy that balances:
- Using eggs efficiently (since broken eggs cannot be reused)
- Minimizing the total number of drops needed in the worst case
- Ensuring you can always determine `f` regardless of where it actually is

## Intuition

Think about this problem as a decision tree where each node represents dropping an egg from a certain floor. When we drop an egg from floor `x`:
- If it breaks, we know `f` must be below `x`, and we lose one egg
- If it doesn't break, we know `f` is at or above `x`, and we keep all eggs

The key insight is that this is a **minimax problem**. We want to minimize the maximum number of moves needed in the worst case. At each step, we need to consider: what's the worst that could happen after dropping from floor `x`?

Let's define `dfs(i, j)` as the minimum number of moves needed to determine `f` with certainty when we have `i` floors to check and `j` eggs available.

When we drop an egg from floor `mid` (where `mid` is between 1 and `i`):
- If egg breaks: We have `mid - 1` floors below to check with `j - 1` eggs
- If egg doesn't break: We have `i - mid` floors above to check with `j` eggs (same number of eggs)

Since we don't know which case will happen, we must prepare for the worst: `max(dfs(mid - 1, j - 1), dfs(i - mid, j)) + 1`

But which floor should we choose to drop from? We want to minimize this worst-case scenario. The brilliant observation is that:
- `dfs(mid - 1, j - 1)` increases as `mid` increases (more floors to check below if egg breaks)
- `dfs(i - mid, j)` decreases as `mid` increases (fewer floors to check above if egg doesn't break)

Since one function increases and the other decreases, their maximum is minimized when they're approximately equal! This is why we use [binary search](/problems/binary_search_intro) to find the optimal dropping floor - we're looking for the point where these two values are closest to each other.

The [binary search](/problems/binary_search_intro) finds the floor where `dfs(mid - 1, j - 1) ≤ dfs(i - mid, j)` transitions to `dfs(mid - 1, j - 1) > dfs(i - mid, j)`, which gives us the optimal balance between the two worst-case scenarios.

Base cases are intuitive:
- If `i < 1`: No floors to check, return 0
- If `j = 1`: Only one egg, must check linearly from bottom, return `i`

**Learn more about [Math](/problems/math-basics), [Binary Search](/problems/binary_search_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a [dynamic programming](/problems/dynamic_programming_intro) approach with memoization using a recursive function `dfs(i, j)` that represents the minimum number of moves needed to determine the critical floor with `i` floors to check and `j` eggs available.

**Implementation Details:**

1. **Main Function Setup:**
   - The function `dfs(n, k)` is called where `n` is the number of floors and `k` is the number of eggs
   - The `@cache` decorator is used for memoization to avoid recalculating the same subproblems

2. **Base Cases:**
   ```python
   if i < 1:
       return 0  # No floors to check
   if j == 1:
       return i  # Only one egg left, must check linearly
   ```

3. **[Binary Search](/problems/binary_search_intro) for Optimal Floor:**
   ```python
   l, r = 1, i
   while l < r:
       mid = (l + r + 1) >> 1  # Equivalent to (l + r + 1) // 2
       a = dfs(mid - 1, j - 1)  # Egg breaks case
       b = dfs(i - mid, j)      # Egg doesn't break case
       if a <= b:
           l = mid
       else:
           r = mid - 1
   ```

   The binary search finds the floor `l` where:
   - `dfs(l - 1, j - 1) ≤ dfs(i - l, j)` but
   - `dfs(l, j - 1) > dfs(i - l - 1, j)`

   This is the point where the two worst-case scenarios are most balanced.

4. **Computing the Result:**
   ```python
   return max(dfs(l - 1, j - 1), dfs(i - l, j)) + 1
   ```

   After finding the optimal floor `l`, we compute:
   - The worst case between egg breaking (`l - 1` floors with `j - 1` eggs) and not breaking (`i - l` floors with `j` eggs)
   - Add 1 for the current drop

**Why [Binary Search](/problems/binary_search_intro) Works:**

The key insight is that as we increase the dropping floor from 1 to `i`:
- `dfs(mid - 1, j - 1)` monotonically increases (more floors to check if egg breaks)
- `dfs(i - mid, j)` monotonically decreases (fewer floors to check if egg doesn't break)

The optimal floor is where these two values are closest, minimizing their maximum. The [binary search](/problems/binary_search_intro) efficiently finds this crossover point.

**Time Complexity:** `O(k * n * log n)` where we have at most `O(k * n)` unique states, and each state requires `O(log n)` time for [binary search](/problems/binary_search_intro).

**Space Complexity:** `O(k * n)` for the memoization cache storing all possible states.

## Example Walkthrough

Let's walk through a small example with **2 eggs and 6 floors** to illustrate the solution approach.

**Problem Setup:** We need to find the minimum number of moves to determine the critical floor `f` in the worst case.

**Step 1: Initial Call**
- Call `dfs(6, 2)` - we have 6 floors to check and 2 eggs available

**Step 2: Binary Search for Optimal Floor**
- We need to find the best floor to drop from among floors 1-6
- Initialize: `l = 1, r = 6`

**First Binary Search Iteration:**
- `mid = (1 + 6 + 1) / 2 = 4`
- Calculate:
  - `a = dfs(3, 1)` (if egg breaks at floor 4: check floors 1-3 with 1 egg)
  - `b = dfs(2, 2)` (if egg doesn't break: check floors 5-6 with 2 eggs)

Let's evaluate these recursive calls:
- `dfs(3, 1) = 3` (with 1 egg, must check linearly: worst case is 3 drops)
- `dfs(2, 2)` needs further calculation...

**Computing dfs(2, 2):**
- Binary search: `l = 1, r = 2`
- `mid = (1 + 2 + 1) / 2 = 2`
- `a = dfs(1, 1) = 1` (1 floor with 1 egg)
- `b = dfs(0, 2) = 0` (0 floors to check)
- Since `a > b`, set `r = 1`
- Final: drop from floor 1
- Result: `max(dfs(0, 1), dfs(1, 2)) + 1 = max(0, 1) + 1 = 2`

**Back to Main Calculation:**
- Now we know: `a = dfs(3, 1) = 3` and `b = dfs(2, 2) = 2`
- Since `a > b`, set `r = 3`

**Second Binary Search Iteration:**
- `l = 1, r = 3`
- `mid = (1 + 3 + 1) / 2 = 2`
- `a = dfs(1, 1) = 1` (if egg breaks at floor 2)
- `b = dfs(4, 2)` (if egg doesn't break: check floors 3-6)

**Computing dfs(4, 2):**
- Through similar binary search process: optimal is floor 2
- Result: `max(dfs(1, 1), dfs(2, 2)) + 1 = max(1, 2) + 1 = 3`

**Back to Main:**
- `a = 1`, `b = 3`
- Since `a ≤ b`, set `l = 2`

**Third Binary Search Iteration:**
- `l = 2, r = 3`
- `mid = (2 + 3 + 1) / 2 = 3`
- `a = dfs(2, 1) = 2`
- `b = dfs(3, 2) = 2`
- Since `a ≤ b`, set `l = 3`
- Now `l = r = 3`, binary search ends

**Step 3: Final Result**
- Optimal floor to drop from is 3
- Result: `max(dfs(2, 1), dfs(3, 2)) + 1 = max(2, 2) + 1 = 3`

**Answer: 3 moves minimum in the worst case**

**Verification of Strategy:**
The optimal strategy is:
1. First drop from floor 3
   - If breaks: check floors 1-2 with 1 egg (max 2 more drops)
   - If doesn't break: check floors 4-6 with 2 eggs (max 2 more drops)
2. Total worst case: 1 + 2 = 3 drops

This demonstrates how the binary search finds the floor (3) where the two worst-case scenarios are balanced (both requiring 2 additional drops), minimizing the overall worst case.

## Python Solution

```python
class Solution:
    def superEggDrop(self, k: int, n: int) -> int:
        from functools import cache

        @cache
        def dp(floors: int, eggs: int) -> int:
            """
            Calculate minimum number of moves needed in worst case
            to find critical floor with given number of floors and eggs.

            Args:
                floors: Number of floors to check
                eggs: Number of eggs available

            Returns:
                Minimum number of moves needed in worst case
            """
            # Base case: No floors to check
            if floors < 1:
                return 0

            # Base case: Only 1 egg left, must check linearly from bottom
            if eggs == 1:
                return floors

            # Binary search to find optimal floor to drop egg from
            # We're looking for the floor that minimizes the worst case
            left, right = 1, floors

            while left < right:
                # Calculate middle point (ceiling division)
                mid = (left + right + 1) >> 1

                # Case 1: Egg breaks at floor mid
                # Check floors below with one less egg
                egg_breaks = dp(mid - 1, eggs - 1)

                # Case 2: Egg doesn't break at floor mid
                # Check floors above with same number of eggs
                egg_survives = dp(floors - mid, eggs)

                # Binary search to find the floor where the two cases balance
                # This minimizes the maximum of the two cases
                if egg_breaks <= egg_survives:
                    left = mid
                else:
                    right = mid - 1

            # Return worst case scenario (max of two cases) plus current move
            return max(dp(left - 1, eggs - 1), dp(floors - left, eggs)) + 1

        # Start with n floors and k eggs
        return dp(n, k)
```

## Time and Space Complexity

**Time Complexity:** `O(k * n * log n)`

The analysis breaks down as follows:
- The recursive function `dfs(i, j)` has at most `n * k` unique states since `i` ranges from 0 to `n` and `j` ranges from 0 to `k`
- Due to memoization with `@cache`, each state is computed only once
- For each state `(i, j)`, the function performs a binary search from 1 to `i`, which takes `O(log i)` iterations, where `i ≤ n`
- In each iteration of the binary search, we make recursive calls to `dfs(mid - 1, j - 1)` and `dfs(i - mid, j)`, but these are memoized lookups with `O(1)` access time after initial computation
- Therefore, the total time complexity is `O(k * n * log n)`

**Space Complexity:** `O(k * n)`

The space complexity consists of:
- The memoization cache stores at most `O(k * n)` entries (one for each unique state)
- Each nested call has either one fewer egg or fewer floors, so the recursion stack can never be deeper than the number of states. In practice it is shallow, because the first probe of each binary search is near the middle floor (for `k = 2, n = 10^4` the depth is 14)
- The stack therefore does not change the overall space complexity of `O(k * n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Binary Search Boundary Logic**

One of the most common mistakes is implementing the binary search incorrectly when finding the optimal floor to drop from. The binary search is looking for a specific crossover point where the two scenarios (egg breaks vs. egg doesn't break) are most balanced.

**Pitfall Example:**
```python
# WRONG: floor division combined with left = mid
while left < right:
    mid = (left + right) // 2  # Rounds down
    egg_breaks = dp(mid - 1, eggs - 1)
    egg_survives = dp(floors - mid, eggs)

    if egg_breaks <= egg_survives:
        left = mid  # When right == left + 1, mid == left, so nothing changes
    else:
        right = mid - 1
```

**Why it's wrong:** When `right == left + 1`, floor division gives `mid == left`. If the condition holds, `left = mid` leaves the range unchanged and the loop never ends. For example, `dp(2, 2)` starts with `left = 1, right = 2`, gets `mid = 1`, finds `dp(0, 1) = 0 <= dp(1, 2) = 1`, and repeats forever.

Other correct forms exist. For example, `while left <= right` with `mid = (left + right) // 2`, moving `left = mid + 1` when `egg_breaks < egg_survives` and `right = mid - 1` otherwise, ends with `left` at the first floor where `egg_breaks >= egg_survives`, which also gives the optimal answer. The important part is that every branch shrinks the range.

**Solution:**
```python
# CORRECT: Use ceiling division and proper boundary conditions
while left < right:  # Stop when left == right
    mid = (left + right + 1) >> 1  # Ceiling division to avoid infinite loop
    egg_breaks = dp(mid - 1, eggs - 1)
    egg_survives = dp(floors - mid, eggs)

    if egg_breaks <= egg_survives:  # Use <= to find the rightmost valid position
        left = mid  # Don't skip mid
    else:
        right = mid - 1
```

### 2. **Off-by-One Errors in Recursive Calls**

Another frequent mistake involves incorrect floor calculations when making recursive calls, especially after an egg breaks or doesn't break.

**Pitfall Example:**
```python
# WRONG: Incorrect floor count calculations
egg_breaks = dp(mid, eggs - 1)      # Should be mid - 1
egg_survives = dp(floors - mid + 1, eggs)  # Should be floors - mid
```

**Why it's wrong:**
- If we drop from floor `mid` and the egg breaks, we know the critical floor is in `[1, mid-1]`, not `[1, mid]`
- If the egg doesn't break at floor `mid`, we need to check floors `[mid+1, floors]`, which is `floors - mid` floors, not `floors - mid + 1`

**Solution:**
```python
# CORRECT: Accurate floor calculations
egg_breaks = dp(mid - 1, eggs - 1)    # Check floors 1 to mid-1
egg_survives = dp(floors - mid, eggs)  # Check floors mid+1 to floors
```

### 3. **Forgetting to Add 1 for the Current Move**

A subtle but critical error is forgetting to account for the current drop when calculating the total number of moves.

**Pitfall Example:**
```python
# WRONG: Forgetting to add the current move
return max(dp(left - 1, eggs - 1), dp(floors - left, eggs))
```

**Solution:**
```python
# CORRECT: Add 1 for the current drop
return max(dp(left - 1, eggs - 1), dp(floors - left, eggs)) + 1
```

### 4. **Inefficient Linear Search Instead of Binary Search**

Some implementations might use a linear search to find the optimal floor, which significantly increases time complexity.

**Pitfall Example:**
```python
# Correct but slow: linear search through all possible floors
min_moves = float('inf')
for floor in range(1, floors + 1):
    egg_breaks = dp(floor - 1, eggs - 1)
    egg_survives = dp(floors - floor, eggs)
    worst_case = max(egg_breaks, egg_survives) + 1
    min_moves = min(min_moves, worst_case)
return min_moves
```

**Why it's a problem:** This returns correct answers, but it has O(n) complexity per state instead of O(log n), making the overall complexity O(k * n²) instead of O(k * n * log n).

**Solution:** Use the binary search approach as shown in the correct implementation to efficiently find the optimal floor in O(log n) time.

