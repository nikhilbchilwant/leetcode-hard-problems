# 265. Paint House II

> Source: https://algo.monster/liteproblems/265

## Problem Statement

You have `n` houses arranged in a row that need to be painted. Each house can be painted with one of `k` different colors, and the cost to paint each house with each color varies.

The problem provides a 2D cost matrix `costs` where:
- `costs[i][j]` represents the cost of painting house `i` with color `j`
- The matrix has dimensions `n x k` (n houses, k colors)

**Constraint**: No two adjacent houses can have the same color.

**Goal**: Find the minimum total cost to paint all houses while satisfying the color constraint.

For example, if you have 3 houses and 3 colors with costs:
- House 0: Color 0 costs $1, Color 1 costs $5, Color 2 costs $3
- House 1: Color 0 costs $2, Color 1 costs $9, Color 2 costs $4
- House 2: Color 0 costs $5, Color 1 costs $1, Color 2 costs $2

You need to choose one color for each house such that consecutive houses have different colors and the sum of all chosen costs is minimized.

## Intuition

When faced with this problem, we need to make a series of decisions - which color to paint each house. The key insight is that our decision for the current house depends on what color we painted the previous house, since adjacent houses can't have the same color.

This dependency between consecutive decisions suggests a dynamic programming approach. We can think of it as: "If I know the minimum cost to paint all houses up to house `i-1` with each possible color, I can determine the minimum cost for house `i`."

For each house, we need to track the minimum cost for each color choice. Why? Because we don't know which color will lead to the global minimum until we've processed all houses.

The recurrence relation becomes clear:
- To paint house `i` with color `j`, we need to find the minimum cost from the previous house where we used any color except `j`
- The total cost would be: `cost[i][j] + min(previous costs for all colors except j)`

Instead of using a 2D DP array `dp[n][k]`, we can optimize space by only keeping track of the costs for the current and previous house. This works because we only ever look back one house when making decisions.

The algorithm maintains an array `f` that stores the minimum cost to paint houses up to the current position with each color. For each new house, we calculate a new array `g` where `g[j]` represents the minimum cost if we paint the current house with color `j`. After processing all houses, the minimum value in our final array gives us the answer.

This bottom-up approach builds the solution incrementally, making optimal local decisions that lead to the global optimum.

## Solution Approach

The solution uses dynamic programming with space optimization. Let's walk through the implementation step by step:

**Initialization:**
```python
n, k = len(costs), len(costs[0])
f = costs[0][:]
```
- Extract dimensions: `n` houses and `k` colors
- Initialize `f` as a copy of the first row of costs, representing the cost to paint house 0 with each color

**Main DP Loop:**
```python
for i in range(1, n):
    g = costs[i][:]
    for j in range(k):
        t = min(f[h] for h in range(k) if h != j)
        g[j] += t
    f = g
```

For each house `i` from 1 to n-1:
1. Create `g` as a copy of the current house's base costs (`costs[i]`)
2. For each color `j` that we could paint house `i`:
   - Find the minimum cost from the previous house (`f`) among all colors except `j`
   - This ensures no adjacent houses have the same color
   - Add this minimum to the base cost: `g[j] += t`
3. Update `f = g` to prepare for the next iteration

**Key Pattern - State Compression:**
Instead of maintaining a 2D DP table `dp[n][k]`, we use two 1D arrays:
- `f`: stores the minimum costs for the previous house
- `g`: calculates the minimum costs for the current house

**Time Complexity:** `O(n * k²)`
- We iterate through `n` houses
- For each house, we consider `k` colors
- For each color, we find the minimum among `k-1` other colors

**Space Complexity:** `O(k)`
- We only maintain arrays of size `k` instead of the full `n x k` table

**Final Result:**
```python
return min(f)
```
After processing all houses, `f` contains the minimum costs to paint all houses ending with each color. The answer is the minimum among all these values.

The elegance of this solution lies in its space optimization - we only track what we need (previous house costs) rather than the entire history.

## Example Walkthrough

Let's walk through a small example with 3 houses and 3 colors:

```
costs = [[1, 5, 3],
         [2, 9, 4],
         [5, 1, 2]]
```

**Step 1: Initialization**
- `n = 3` houses, `k = 3` colors
- `f = [1, 5, 3]` (costs for house 0)

**Step 2: Process House 1** (i = 1)
- Start with `g = [2, 9, 4]` (base costs for house 1)
- For color 0 (j = 0):
  - Find min from previous house excluding color 0: `min(f[1], f[2]) = min(5, 3) = 3`
  - `g[0] = 2 + 3 = 5`
- For color 1 (j = 1):
  - Find min from previous house excluding color 1: `min(f[0], f[2]) = min(1, 3) = 1`
  - `g[1] = 9 + 1 = 10`
- For color 2 (j = 2):
  - Find min from previous house excluding color 2: `min(f[0], f[1]) = min(1, 5) = 1`
  - `g[2] = 4 + 1 = 5`
- Update: `f = [5, 10, 5]`

**Step 3: Process House 2** (i = 2)
- Start with `g = [5, 1, 2]` (base costs for house 2)
- For color 0 (j = 0):
  - Find min from previous house excluding color 0: `min(f[1], f[2]) = min(10, 5) = 5`
  - `g[0] = 5 + 5 = 10`
- For color 1 (j = 1):
  - Find min from previous house excluding color 1: `min(f[0], f[2]) = min(5, 5) = 5`
  - `g[1] = 1 + 5 = 6`
- For color 2 (j = 2):
  - Find min from previous house excluding color 2: `min(f[0], f[1]) = min(5, 10) = 5`
  - `g[2] = 2 + 5 = 7`
- Update: `f = [10, 6, 7]`

**Step 4: Find Result**
- `min(f) = min(10, 6, 7) = 6`

**Optimal Solution:** Paint house 0 with color 0 (cost 1), house 1 with color 2 (cost 4), and house 2 with color 1 (cost 1). Total cost = 1 + 4 + 1 = 6.

The array `f` at each step represents the minimum cost to paint all houses up to that point, ending with each specific color.

## Python Solution

```python
class Solution:
    def minCostII(self, costs: List[List[int]]) -> int:
        # Get dimensions: n houses and k colors
        num_houses = len(costs)
        num_colors = len(costs[0])

        # Initialize dp array with first house costs
        # dp[j] represents minimum cost to paint houses 0..i with house i painted with color j
        dp_prev = costs[0][:]

        # Process each subsequent house
        for house_idx in range(1, num_houses):
            # Create new dp array for current house
            dp_curr = costs[house_idx][:]

            # For each color option for current house
            for curr_color in range(num_colors):
                # Find minimum cost from previous house with different color
                min_prev_cost = min(dp_prev[prev_color]
                                   for prev_color in range(num_colors)
                                   if prev_color != curr_color)

                # Add minimum previous cost to current color cost
                dp_curr[curr_color] += min_prev_cost

            # Update dp array for next iteration
            dp_prev = dp_curr

        # Return minimum cost among all color options for last house
        return min(dp_prev)
```

## Time and Space Complexity

**Time Complexity:** `O(n * k^2)`

The algorithm iterates through `n` houses (from index 1 to n-1). For each house, it iterates through all `k` colors. For each color `j`, it finds the minimum cost from the previous house by checking all `k` colors except color `j`, which takes `O(k)` time. Therefore, the total time complexity is `O(n * k * k) = O(n * k^2)`.

**Space Complexity:** `O(k)`

The algorithm uses two arrays `f` and `g`, each of size `k`, to store the minimum costs for the current and previous houses. Since we only maintain the costs for at most two houses at any given time (current and previous), the space complexity is `O(k) + O(k) = O(k)`. The input array `costs` is not counted as extra space since it's given as input.

## Common Pitfalls

### 1. **Inefficient Minimum Finding - O(k²) Inner Loop**

The current solution has a nested loop structure where for each color `j`, we iterate through all `k` colors to find the minimum excluding `j`. This results in O(k²) operations per house.

**The Problem:**
```python
# Current approach - inefficient
for curr_color in range(num_colors):
    min_prev_cost = min(dp_prev[prev_color]
                       for prev_color in range(num_colors)
                       if prev_color != curr_color)
```

**Optimized Solution - O(k) per house:**
```python
def minCostII(self, costs: List[List[int]]) -> int:
    num_houses = len(costs)
    num_colors = len(costs[0])

    # Track the two smallest costs and their colors from previous house
    min1_cost, min1_color = 0, -1
    min2_cost, min2_color = 0, -1

    for house_idx in range(num_houses):
        # Calculate new minimums for current house
        new_min1_cost, new_min1_color = float('inf'), -1
        new_min2_cost, new_min2_color = float('inf'), -1

        for color in range(num_colors):
            # Calculate cost for this color
            if color == min1_color:
                # If same as previous min color, use second min
                cost = costs[house_idx][color] + min2_cost
            else:
                # Otherwise use the minimum
                cost = costs[house_idx][color] + min1_cost

            # Update the two minimums
            if cost < new_min1_cost:
                new_min2_cost, new_min2_color = new_min1_cost, new_min1_color
                new_min1_cost, new_min1_color = cost, color
            elif cost < new_min2_cost:
                new_min2_cost, new_min2_color = cost, color

        min1_cost, min1_color = new_min1_cost, new_min1_color
        min2_cost, min2_color = new_min2_cost, new_min2_color

    return min1_cost
```

### 2. **Edge Case: Single Color (k=1)**

When there's only one color available, it's impossible to paint adjacent houses with different colors (for n > 1). LeetCode's constraints guarantee `k >= 2`, so the solution never hits this case there, but code reused elsewhere should guard against it.

**The Problem:**
```python
# Will fail or give incorrect result when k=1 and n>1
min_prev_cost = min(dp_prev[prev_color]
                   for prev_color in range(num_colors)
                   if prev_color != curr_color)
# Generator will be empty when k=1 and curr_color=0
```

**Solution:**
```python
def minCostII(self, costs: List[List[int]]) -> int:
    num_houses = len(costs)
    num_colors = len(costs[0])

    # Handle edge case
    if num_colors == 1:
        return costs[0][0] if num_houses == 1 else -1  # or float('inf')

    # Continue with regular solution...
```

### 3. **Memory Allocation Overhead**

Creating new lists in each iteration can be inefficient for large inputs.

**The Problem:**
```python
# Creates new list copies in each iteration
dp_curr = costs[house_idx][:]  # Unnecessary copying
```

**Solution:**
```python
def minCostII(self, costs: List[List[int]]) -> int:
    num_houses = len(costs)
    num_colors = len(costs[0])

    # Pre-allocate arrays and swap references
    dp_prev = costs[0][:]
    dp_curr = [0] * num_colors  # Reuse this array

    for house_idx in range(1, num_houses):
        for curr_color in range(num_colors):
            min_prev_cost = min(dp_prev[prev_color]
                               for prev_color in range(num_colors)
                               if prev_color != curr_color)
            dp_curr[curr_color] = costs[house_idx][curr_color] + min_prev_cost

        # Swap references instead of creating new arrays
        dp_prev, dp_curr = dp_curr, dp_prev

    return min(dp_prev)
```

The most significant optimization is addressing pitfall #1, which reduces time complexity from O(n×k²) to O(n×k), making the solution much more efficient for large values of k.

