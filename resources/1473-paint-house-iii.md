# 1473. Paint House III

> Source: https://algo.monster/liteproblems/1473

## Problem Statement

You have a row of `m` houses that need to be painted with one of `n` colors (labeled from `1` to `n`). Some houses are already painted and should not be repainted.

A **neighborhood** is defined as a maximal group of continuous houses painted with the same color. For example, if `houses = [1,2,2,3,3,2,1,1]`, there are 5 neighborhoods: `[{1}, {2,2}, {3,3}, {2}, {1,1}]`.

You are given:
- An array `houses` where `houses[i]` represents:
  - The color of house `i` (if already painted)
  - `0` if the house is not painted yet
- An `m x n` matrix `cost` where `cost[i][j]` is the cost to paint house `i` with color `j + 1`
- An integer `target` representing the desired number of neighborhoods

Your task is to find the **minimum cost** to paint all unpainted houses such that the total number of neighborhoods equals exactly `target`. If it's impossible to achieve exactly `target` neighborhoods, return `-1`.

The key constraints are:
- Already painted houses cannot be repainted
- You must create exactly `target` neighborhoods
- Each unpainted house must be assigned one of the `n` available colors
- The cost of painting each house with each color is given in the cost matrix

## Intuition

The problem asks us to find the minimum cost to paint houses while maintaining exactly `target` neighborhoods. This is a classic optimization problem with constraints, which suggests [dynamic programming](/problems/dynamic_programming_intro) as a potential approach.

Let's think about what information we need to track as we process each house:
1. **Which house we're currently at** - we need to make decisions for each house sequentially
2. **What color the previous house was painted** - this determines whether we're continuing a neighborhood or starting a new one
3. **How many neighborhoods we've formed so far** - we need exactly `target` neighborhoods at the end

This naturally leads to a 3-dimensional DP state: `f[i][j][k]` representing the minimum cost to paint houses from index `0` to `i`, where house `i` is painted with color `j`, and we've formed exactly `k` neighborhoods.

The key insight is how neighborhoods are formed:
- If we paint the current house the **same color** as the previous house, we're extending the current neighborhood (no new neighborhood is created)
- If we paint the current house a **different color** than the previous house, we're starting a new neighborhood (neighborhood count increases by 1)

For each house, we have two scenarios:
1. **House is already painted**: We can't change its color, so we only need to calculate the cost based on whether it forms a new neighborhood or not (cost is 0 since no painting is needed)
2. **House is not painted**: We try all possible colors and add the painting cost, considering whether each color choice creates a new neighborhood

The transition between states depends on comparing the current house's color with the previous house's color:
- If `j == j0` (same color as previous): `f[i][j][k] = f[i-1][j0][k] + cost` (if needed)
- If `j != j0` (different color): `f[i][j][k] = f[i-1][j0][k-1] + cost` (if needed)

By building up the solution house by house, tracking all possible color and neighborhood combinations, we can find the minimum cost to achieve exactly `target` neighborhoods. The final answer is the minimum among all possible colors for the last house with exactly `target` neighborhoods.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement a 3D [dynamic programming](/problems/dynamic_programming_intro) solution where `f[i][j][k]` represents the minimum cost to paint houses from index `0` to `i`, with house `i` painted in color `j`, forming exactly `k` neighborhoods.

**Initialization:**
- Create a 3D array `f` with dimensions `[m][n+1][target+1]`, initialized to infinity
- For the first house (index 0):
  - If unpainted: `f[0][j][1] = cost[0][j-1]` for each color `j`
  - If already painted: `f[0][houses[0]][1] = 0`

**Main DP Loop:**
For each house `i` from 1 to `m-1`:

1. **If house is unpainted** (`houses[i] == 0`):
   - Try all possible colors `j` from 1 to `n`
   - For each valid neighborhood count `k` from 1 to `min(target, i+1)`:
     - For each possible previous color `j0`:
       - If `j == j0` (same color, extend neighborhood):
         ```
         f[i][j][k] = min(f[i][j][k], f[i-1][j][k] + cost[i][j-1])
         ```
       - If `j != j0` (different color, new neighborhood):
         ```
         f[i][j][k] = min(f[i][j][k], f[i-1][j0][k-1] + cost[i][j-1])
         ```

2. **If house is already painted** (`houses[i] != 0`):
   - Set `j = houses[i]` (fixed color)
   - For each valid neighborhood count `k`:
     - For each possible previous color `j0`:
       - If `j == j0` (same color, extend neighborhood):
         ```
         f[i][j][k] = min(f[i][j][k], f[i-1][j][k])
         ```
       - If `j != j0` (different color, new neighborhood):
         ```
         f[i][j][k] = min(f[i][j][k], f[i-1][j0][k-1])
         ```
   - No painting cost is added since the house is already painted

**Finding the Answer:**
- After processing all houses, check `f[m-1][j][target]` for all colors `j`
- Return the minimum value found
- If all values are infinity, return -1 (impossible to achieve target neighborhoods)

**Key Optimizations:**
- The neighborhood count `k` is bounded by `min(target+1, i+2)` since we can't have more neighborhoods than houses processed so far
- We use 1-indexed colors to match the problem's color labeling (1 to n)
- The cost array uses 0-indexing, so we access `cost[i][j-1]` for color `j`

The time complexity is `O(m * n^2 * target)` and space complexity is `O(m * n * target)`.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Input:**
- `houses = [0, 2, 0]` (3 houses, middle one is already painted with color 2)
- `cost = [[1, 10], [10, 1], [1, 10]]` (costs to paint each house with colors 1 or 2)
- `target = 2` (we want exactly 2 neighborhoods)
- `n = 2` (2 colors available)

**Goal:** Paint the unpainted houses (indices 0 and 2) to form exactly 2 neighborhoods with minimum cost.

**DP Setup:**
We create `f[i][j][k]` where:
- `i` = house index (0 to 2)
- `j` = color (1 or 2)
- `k` = number of neighborhoods formed (1 or 2)

**Step 1: Initialize for house 0**
House 0 is unpainted, so we can paint it with either color:
- `f[0][1][1] = cost[0][0] = 1` (paint house 0 with color 1, forming 1 neighborhood)
- `f[0][2][1] = cost[0][1] = 10` (paint house 0 with color 2, forming 1 neighborhood)

**Step 2: Process house 1**
House 1 is already painted with color 2 (no cost to add):

For `k = 1` (maintaining 1 neighborhood):
- `f[1][2][1] = f[0][2][1] + 0 = 10` (previous house was also color 2, same neighborhood)

For `k = 2` (forming 2 neighborhoods):
- `f[1][2][2] = f[0][1][1] + 0 = 1` (previous house was color 1, different from 2, new neighborhood)

**Step 3: Process house 2**
House 2 is unpainted, so we try both colors:

For color 1, `k = 2`:
- From previous color 2 (different): `f[2][1][2] = f[1][2][1] + cost[2][0] = 10 + 1 = 11`

For color 1, `k = 3`:
- From previous color 2 (different): `f[2][1][3] = f[1][2][2] + cost[2][0] = 1 + 1 = 2`
- But wait! We only want `target = 2` neighborhoods, so we don't consider `k = 3`.

For color 2, `k = 1`:
- From previous color 2 (same): `f[2][2][1] = f[1][2][1] + cost[2][1] = 10 + 10 = 20`

For color 2, `k = 2`:
- From previous color 2 (same): `f[2][2][2] = f[1][2][2] + cost[2][1] = 1 + 10 = 11`

**Step 4: Find the answer**
We look at `f[2][j][2]` for all colors `j`:
- `f[2][1][2] = 11` (end with color 1, 2 neighborhoods)
- `f[2][2][2] = 11` (end with color 2, 2 neighborhoods)

The minimum cost is **11**.

**Verification:**
One optimal solution: Paint house 0 with color 1 (cost 1), house 1 is already color 2, paint house 2 with color 2 (cost 10). This gives us colors `[1, 2, 2]` with 2 neighborhoods: `[{1}, {2,2}]`. Total cost = 1 + 10 = 11.

## Python Solution

```python
class Solution:
    def minCost(
        self, houses: List[int], cost: List[List[int]], m: int, n: int, target: int
    ) -> int:
        # Initialize DP table: dp[house_idx][color][num_neighborhoods]
        # dp[i][j][k] = minimum cost to paint houses 0..i where house i has color j 
        # and forms exactly k neighborhoods
        INF = float('inf')
        dp = [[[INF] * (target + 1) for _ in range(n + 1)] for _ in range(m)]
      
        # Base case: handle the first house
        if houses[0] == 0:
            # First house is unpainted, try all possible colors
            for color_idx, paint_cost in enumerate(cost[0], 1):
                dp[0][color_idx][1] = paint_cost
        else:
            # First house is already painted, no cost needed
            dp[0][houses[0]][1] = 0
      
        # Fill the DP table for remaining houses
        for house_idx in range(1, m):
            if houses[house_idx] == 0:
                # Current house is unpainted, try all possible colors
                for current_color in range(1, n + 1):
                    # Try different numbers of neighborhoods (up to house_idx + 1)
                    for num_neighborhoods in range(1, min(target + 1, house_idx + 2)):
                        # Consider all possible colors of the previous house
                        for prev_color in range(1, n + 1):
                            if current_color == prev_color:
                                # Same color as previous house, no new neighborhood
                                dp[house_idx][current_color][num_neighborhoods] = min(
                                    dp[house_idx][current_color][num_neighborhoods],
                                    dp[house_idx - 1][prev_color][num_neighborhoods] + cost[house_idx][current_color - 1]
                                )
                            else:
                                # Different color from previous house, creates new neighborhood
                                dp[house_idx][current_color][num_neighborhoods] = min(
                                    dp[house_idx][current_color][num_neighborhoods],
                                    dp[house_idx - 1][prev_color][num_neighborhoods - 1] + cost[house_idx][current_color - 1]
                                )
            else:
                # Current house is already painted with a specific color
                current_color = houses[house_idx]
                # Try different numbers of neighborhoods
                for num_neighborhoods in range(1, min(target + 1, house_idx + 2)):
                    # Consider all possible colors of the previous house
                    for prev_color in range(1, n + 1):
                        if current_color == prev_color:
                            # Same color as previous house, no new neighborhood
                            dp[house_idx][current_color][num_neighborhoods] = min(
                                dp[house_idx][current_color][num_neighborhoods],
                                dp[house_idx - 1][prev_color][num_neighborhoods]
                            )
                        else:
                            # Different color from previous house, creates new neighborhood
                            dp[house_idx][current_color][num_neighborhoods] = min(
                                dp[house_idx][current_color][num_neighborhoods],
                                dp[house_idx - 1][prev_color][num_neighborhoods - 1]
                            )
      
        # Find the minimum cost among all possible colors for the last house
        # with exactly target neighborhoods
        min_cost = min(dp[-1][color][target] for color in range(1, n + 1))
      
        # Return -1 if impossible, otherwise return the minimum cost
        return -1 if min_cost >= INF else min_cost
```

## Time and Space Complexity

**Time Complexity:** `O(m × n² × target)`

The analysis breaks down as follows:
- The outer loop iterates through `m` houses (from index 1 to m-1)
- For each house, we iterate through up to `n` possible colors for the current house (variable `j`)
- For each color, we iterate through up to `target` possible neighborhood counts (variable `k`)
- For each neighborhood count, we iterate through `n` possible colors of the previous house (variable `j0`)
- The initialization phase for the first house takes `O(n)` time

Therefore, the overall time complexity is `O(m × n × target × n) = O(m × n² × target)`

**Space Complexity:** `O(m × n × target)`

The space complexity is determined by the 3D DP array `f`:
- First dimension: `m` houses
- Second dimension: `n + 1` colors (indexed from 0 to n)
- Third dimension: `target + 1` neighborhood counts (indexed from 0 to target)

The array `f` has dimensions `[m][n+1][target+1]`, resulting in a space complexity of `O(m × n × target)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Color Indexing Between Houses and Cost Arrays**

One of the most common mistakes is confusion between the color indexing systems:
- The `houses` array uses 1-based indexing for colors (1 to n), with 0 indicating unpainted
- The `cost` matrix uses 0-based indexing where `cost[i][j]` represents the cost to paint house `i` with color `j+1`

**Pitfall Example:**
```python
# WRONG: Using color directly as index in cost array
if houses[house_idx] == 0:
    for current_color in range(1, n + 1):
        # This will cause IndexError or use wrong cost
        painting_cost = cost[house_idx][current_color]  # ❌ Wrong!
```

**Solution:**
```python
# CORRECT: Subtract 1 when accessing cost array
if houses[house_idx] == 0:
    for current_color in range(1, n + 1):
        painting_cost = cost[house_idx][current_color - 1]  # ✓ Correct!
```

### 2. **Forgetting to Skip Painting Cost for Already Painted Houses**

When a house is already painted, adding its painting cost is incorrect since we cannot repaint it.

**Pitfall Example:**
```python
# WRONG: Adding cost even for already painted houses
if houses[house_idx] != 0:
    current_color = houses[house_idx]
    # Adding cost here is wrong!
    dp[house_idx][current_color][num_neighborhoods] = min(
        dp[house_idx][current_color][num_neighborhoods],
        dp[house_idx - 1][prev_color][num_neighborhoods] + cost[house_idx][current_color - 1]  # ❌
    )
```

**Solution:**
```python
# CORRECT: No cost added for already painted houses
if houses[house_idx] != 0:
    current_color = houses[house_idx]
    dp[house_idx][current_color][num_neighborhoods] = min(
        dp[house_idx][current_color][num_neighborhoods],
        dp[house_idx - 1][prev_color][num_neighborhoods]  # ✓ No cost added
    )
```

### 3. **Incorrect Neighborhood Count Bounds**

The maximum number of neighborhoods at house `i` cannot exceed `i+1` (one neighborhood per house). Using `target+1` as the upper bound for all houses wastes computation and can lead to logic errors.

**Pitfall Example:**
```python
# WRONG: Always iterating up to target+1 neighborhoods
for num_neighborhoods in range(1, target + 1):  # ❌ Inefficient and potentially wrong
    # This allows invalid states where neighborhoods > houses
```

**Solution:**
```python
# CORRECT: Limit neighborhoods to min(target+1, house_idx+2)
for num_neighborhoods in range(1, min(target + 1, house_idx + 2)):  # ✓
    # This ensures we never have more neighborhoods than houses
```

### 4. **Not Handling the Base Case Correctly**

The first house requires special handling since it always forms exactly 1 neighborhood, and there's no previous house to consider.

**Pitfall Example:**
```python
# WRONG: Trying to access previous house for index 0
for house_idx in range(m):  # Starting from 0
    for prev_color in range(1, n + 1):
        # This will fail when house_idx = 0
        dp[house_idx][current_color][k] = dp[house_idx - 1][prev_color][k]  # ❌ Index -1!
```

**Solution:**
```python
# CORRECT: Handle first house separately
if houses[0] == 0:
    for color_idx, paint_cost in enumerate(cost[0], 1):
        dp[0][color_idx][1] = paint_cost
else:
    dp[0][houses[0]][1] = 0

# Then process remaining houses starting from index 1
for house_idx in range(1, m):  # ✓ Start from 1
    # ... rest of the logic
```

