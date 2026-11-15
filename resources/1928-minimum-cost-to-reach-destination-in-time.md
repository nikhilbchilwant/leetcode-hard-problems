# 1928. Minimum Cost to Reach Destination in Time

> Source: https://algo.monster/liteproblems/1928

## Problem Statement

You have a country with `n` cities numbered from `0` to `n - 1`. All cities are connected by bi-directional roads. The road network is given as a 2D array `edges` where `edges[i] = [xi, yi, timei]` represents a road between cities `xi` and `yi` that takes `timei` minutes to travel. There can be multiple roads with different travel times between the same two cities, but no road connects a city to itself.

Each city has a passing fee that you must pay whenever you pass through it (including your starting and ending cities). These fees are given in the array `passingFees` where `passingFees[j]` is the cost in dollars to pass through city `j`.

You start at city `0` and need to reach city `n - 1` within `maxTime` minutes or less. The total cost of your journey is the sum of all passing fees for every city you pass through during your trip.

Your task is to find the minimum cost to complete the journey from city `0` to city `n - 1` within the time limit. If it's impossible to reach the destination within `maxTime` minutes, return `-1`.

For example, if you travel from city `0` → city `2` → city `3`, you would pay `passingFees[0] + passingFees[2] + passingFees[3]` as your total cost.

## Intuition

The key insight is that we need to find the minimum cost path while considering both the time constraint and the passing fees. This is a constrained shortest path problem where we have two dimensions to optimize: time and cost.

We can think of this problem as exploring different states where each state represents "being at a certain city after spending a certain amount of time." Since we want to minimize cost while respecting the time limit, we need to track the minimum cost to reach each city at different time points.

The natural approach is [dynamic programming](/problems/dynamic_programming_intro) because:
1. We have overlapping subproblems - we might reach the same city at the same time through different paths
2. We need to make optimal decisions - among all ways to reach a city within a time limit, we want the cheapest one

The state can be defined as `f[i][j]` = minimum cost to reach city `j` using exactly `i` minutes. Why track exact time instead of "within i minutes"? Because a path that reaches a city earlier might enable us to take a different route later that leads to a better overall solution.

For each time step from `1` to `maxTime`, we try all possible edges. For an edge `(x, y, t)`, if we have `t ≤ i` minutes available, we can:
- Reach city `x` by first getting to city `y` in `i - t` minutes, then taking this edge (adding `passingFees[x]`)
- Reach city `y` by first getting to city `x` in `i - t` minutes, then taking this edge (adding `passingFees[y]`)

This builds up all possible ways to reach each city at each time point, keeping only the minimum cost for each state. Finally, we check all time points from `0` to `maxTime` and find the minimum cost to reach city `n - 1`.

**Learn more about [Graph](/problems/graph_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement the [dynamic programming](/problems/dynamic_programming_intro) solution using a 2D table `f` where `f[i][j]` represents the minimum cost to reach city `j` from city `0` in exactly `i` minutes.

**Initialization:**
- Create a 2D array `f` of size `(maxTime + 1) × n` initialized with infinity (`inf`)
- Set `f[0][0] = passingFees[0]` since we start at city `0` at time `0` and must pay its passing fee
- All other `f[0][j]` remain infinity since we cannot reach any other city in `0` minutes

**State Transition:**
For each time `i` from `1` to `maxTime`:
- Iterate through all edges `(x, y, t)`
- If the edge time `t ≤ i` (we have enough time to use this edge):
  - Update `f[i][x] = min(f[i][x], f[i - t][y] + passingFees[x])`
    - This means: reach city `x` at time `i` by being at city `y` at time `i - t`, then taking the edge to `x`
  - Update `f[i][y] = min(f[i][y], f[i - t][x] + passingFees[y])`
    - This means: reach city `y` at time `i` by being at city `x` at time `i - t`, then taking the edge to `y`

**Finding the Answer:**
- After filling the DP table, check all possible times from `0` to `maxTime`
- Find the minimum value among `f[0][n-1], f[1][n-1], ..., f[maxTime][n-1]`
- If this minimum is still infinity, return `-1` (impossible to reach destination)
- Otherwise, return the minimum cost found

**Time Complexity:** `O(maxTime × |edges|)` since we iterate through all edges for each time step

**Space Complexity:** `O(maxTime × n)` for the DP table

The algorithm correctly handles multiple edges between cities and ensures we only consider paths that respect the time constraint while minimizing the total passing fees.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example Input:**
- `n = 4` (cities: 0, 1, 2, 3)
- `edges = [[0,1,10], [1,2,10], [2,3,10], [0,3,100]]`
- `passingFees = [5, 1, 2, 20]`
- `maxTime = 30`

We need to find the minimum cost to travel from city 0 to city 3 within 30 minutes.

**Step 1: Initialize DP Table**
Create a table `f[time][city]` where `f[i][j]` = minimum cost to reach city `j` in exactly `i` minutes.

Initially:
- `f[0][0] = 5` (start at city 0, pay its fee)
- All other entries = ∞

**Step 2: Fill DP Table**

For **time = 10**:
- Edge (0,1,10): Can reach city 1 from city 0
  - `f[10][1] = f[0][0] + passingFees[1] = 5 + 1 = 6`
- Edge (1,2,10): Cannot use (would need to be at city 1 or 2 at time 0)
- Edge (2,3,10): Cannot use
- Edge (0,3,100): Cannot use (needs 100 minutes)

State after time 10: `f[10][0]=∞, f[10][1]=6, f[10][2]=∞, f[10][3]=∞`

For **time = 20**:
- Edge (0,1,10): 
  - From city 1 at time 10 → city 0: `f[20][0] = min(∞, 6 + 5) = 11`
- Edge (1,2,10):
  - From city 1 at time 10 → city 2: `f[20][2] = min(∞, 6 + 2) = 8`
- Edge (2,3,10): Cannot use (no way to reach city 2 or 3 at time 10)
- Edge (0,3,100): Cannot use

State after time 20: `f[20][0]=11, f[20][1]=∞, f[20][2]=8, f[20][3]=∞`

For **time = 30**:
- Edge (0,1,10):
  - From city 0 at time 20 → city 1: `f[30][1] = min(∞, 11 + 1) = 12`
- Edge (1,2,10):
  - From city 2 at time 20 → city 1: `f[30][1] = min(12, 8 + 1) = 9`
- Edge (2,3,10):
  - From city 2 at time 20 → city 3: `f[30][3] = min(∞, 8 + 20) = 28`
- Edge (0,3,100): Cannot use

State after time 30: `f[30][0]=∞, f[30][1]=9, f[30][2]=∞, f[30][3]=28`

**Step 3: Find Minimum Cost to Reach City 3**
Check all times from 0 to 30 for reaching city 3:
- `f[0][3] = ∞`
- `f[10][3] = ∞`
- `f[20][3] = ∞`
- `f[30][3] = 28`

The minimum cost is **28**, achieved by the path: city 0 → city 1 → city 2 → city 3
- Total time: 10 + 10 + 10 = 30 minutes
- Total cost: 5 (city 0) + 1 (city 1) + 2 (city 2) + 20 (city 3) = 28

Note that the direct path 0 → 3 would take 100 minutes, exceeding our time limit, so it's not considered.

## Python Solution

```python
class Solution:
    def minCost(
        self, maxTime: int, edges: List[List[int]], passingFees: List[int]
    ) -> int:
        """
        Find minimum cost to reach the last city within maxTime.
      
        Args:
            maxTime: Maximum time allowed for travel
            edges: List of [city1, city2, time] representing bidirectional roads
            passingFees: List of fees for passing through each city
          
        Returns:
            Minimum cost to reach city n-1 from city 0, or -1 if impossible
        """
        max_time = maxTime
        num_cities = len(passingFees)
      
        # dp[time][city] = minimum cost to reach city at exact time
        # Initialize with infinity for all states
        dp = [[float('inf')] * num_cities for _ in range(max_time + 1)]
      
        # Base case: start at city 0 at time 0 with its passing fee
        dp[0][0] = passingFees[0]
      
        # Fill DP table for each time step
        for time in range(1, max_time + 1):
            # Process each edge to update minimum costs
            for city1, city2, travel_time in edges:
                # Check if we have enough time to use this edge
                if travel_time <= time:
                    # Update cost to reach city1 from city2
                    dp[time][city1] = min(
                        dp[time][city1], 
                        dp[time - travel_time][city2] + passingFees[city1]
                    )
                    # Update cost to reach city2 from city1 (bidirectional)
                    dp[time][city2] = min(
                        dp[time][city2], 
                        dp[time - travel_time][city1] + passingFees[city2]
                    )
      
        # Find minimum cost to reach the last city across all valid times
        min_cost = min(dp[time][num_cities - 1] for time in range(max_time + 1))
      
        # Return result: minimum cost if reachable, otherwise -1
        return min_cost if min_cost < float('inf') else -1
```

## Time and Space Complexity

**Time Complexity:** `O(maxTime × m)`, where `m` is the number of edges.

The algorithm uses dynamic programming with two nested loops:
- The outer loop runs `maxTime` iterations (from 1 to `m`)
- The inner loop iterates through all edges, which takes `O(m)` time per iteration
- Inside the inner loop, each edge results in constant time operations (comparisons and updates)
- The final step to find the minimum takes `O(maxTime)` time, which is dominated by the main loops

Therefore, the total time complexity is `O(maxTime × m)`.

**Space Complexity:** `O(maxTime × n)`, where `n` is the number of cities (nodes).

The algorithm creates a 2D DP table `f` with dimensions `(maxTime + 1) × n`:
- The table has `maxTime + 1` rows (one for each time from 0 to `maxTime`)
- Each row contains `n` elements (one for each city)
- No additional significant space is used beyond this table

Therefore, the space complexity is `O(maxTime × n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Initialize Starting City Cost**
A critical mistake is not properly initializing the starting city's cost. Some might set `dp[0][0] = 0` instead of `dp[0][0] = passingFees[0]`, forgetting that we must pay the passing fee for the starting city as well.

**Wrong:**
```python
dp[0][0] = 0  # Incorrect - misses the starting city fee
```

**Correct:**
```python
dp[0][0] = passingFees[0]  # Must pay fee for starting city
```

### 2. **Not Handling Bidirectional Edges Properly**
Since roads are bidirectional, each edge can be traversed in both directions. A common mistake is only updating one direction in the DP transition.

**Wrong:**
```python
for city1, city2, travel_time in edges:
    if travel_time <= time:
        # Only updating one direction
        dp[time][city2] = min(
            dp[time][city2], 
            dp[time - travel_time][city1] + passingFees[city2]
        )
```

**Correct:**
```python
for city1, city2, travel_time in edges:
    if travel_time <= time:
        # Update both directions
        dp[time][city1] = min(
            dp[time][city1], 
            dp[time - travel_time][city2] + passingFees[city1]
        )
        dp[time][city2] = min(
            dp[time][city2], 
            dp[time - travel_time][city1] + passingFees[city2]
        )
```

### 3. **Incorrect Time Constraint Check**
Some might check `travel_time < time` instead of `travel_time <= time`, which would miss valid paths where the travel time exactly equals the current time being considered.

**Wrong:**
```python
if travel_time < time:  # Misses edge case where travel_time == time
```

**Correct:**
```python
if travel_time <= time:  # Correctly includes all valid travel times
```

### 4. **Using Wrong DP State Definition**
A subtle but important pitfall is misunderstanding what the DP state represents. The state `dp[i][j]` should represent the minimum cost to reach city `j` in **at most** `i` time (or exactly `i` time in this implementation). Mixing up this definition can lead to incorrect transitions.

### 5. **Memory Optimization Trap**
While it might be tempting to optimize space by using only two arrays (current and previous time), this can lead to errors if not implemented carefully, as we need to ensure we don't overwrite values we still need to read.

**Risky optimization:**
```python
# Using only current and previous arrays can cause issues
prev = [float('inf')] * num_cities
curr = [float('inf')] * num_cities
# Need to be very careful about when values are updated
```

**Safe approach (as in the solution):**
```python
# Full 2D array is clearer and less error-prone
dp = [[float('inf')] * num_cities for _ in range(max_time + 1)]
```

