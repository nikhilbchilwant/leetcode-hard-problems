# 568. Maximum Vacation Days

> Source: https://algo.monster/liteproblems/568

## Problem Statement

You are planning a travel schedule across `n` cities over `k` weeks to maximize vacation days. You start in city 0 on Monday of the first week.

**Travel Rules:**
- Cities are numbered from 0 to n-1
- You can only fly between cities on Monday mornings (once per week)
- The `flights` matrix shows available flights: `flights[i][j] = 1` means there's a flight from city i to city j, `flights[i][j] = 0` means no flight exists
- You can choose to stay in your current city instead of flying

**Vacation Rules:**
- The `days` matrix shows vacation days available: `days[i][j]` represents the maximum vacation days you can take in city i during week j
- You can stay in a city even after using all vacation days (working on extra days)
- If you fly to a new city and take vacation that week, the vacation days count for the destination city

**Goal:** Find the maximum total vacation days possible over all k weeks.

**Example Understanding:**
If you're in city A in week 1 with 2 vacation days available, and city B has 5 vacation days available in week 2, you could:
- Stay in city A for week 1 (get 2 vacation days)
- Fly to city B on Monday of week 2 (if a flight exists)
- Take vacation in city B for week 2 (get 5 vacation days)
- Total: 7 vacation days

## Intuition

The key insight is that at any point in time, what matters is: which week we're in and which city we're currently at. Our future vacation days don't depend on how we got to this city, only on our current position and remaining weeks.

This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach. We can think of the problem as making optimal decisions week by week. For each week, we need to decide: should we stay in our current city or fly to another city?

The state can be defined as `f[k][j]` = maximum vacation days we can get by the end of week `k` if we're in city `j`.

For each week `k` and each possible destination city `j`, we have two choices:
1. Stay in city `j` from the previous week (if we were already there): `f[k-1][j]`
2. Fly from some other city `i` to city `j` (if a flight exists): `f[k-1][i]` where `flights[i][j] = 1`

We take the maximum of all these possibilities and add the vacation days available in city `j` during week `k`: `days[j][k-1]`.

The initialization is crucial: we start at city 0 with 0 vacation days before week 1 begins, so `f[0][0] = 0`. All other states start as negative infinity since they're impossible to reach initially.

After processing all `k` weeks, the answer is the maximum value among all cities in the final week, since we can end our journey in any city.

This bottom-up approach ensures we consider all possible flight paths and choose the one that maximizes total vacation days.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement a 2D [dynamic programming](/problems/dynamic_programming_intro) solution where `f[k][j]` represents the maximum vacation days achievable by the end of week `k` when ending in city `j`.

**Initialization:**
- Create a 2D array `f` with dimensions `(K+1) × n`, where `K` is the number of weeks and `n` is the number of cities
- Initialize all values to negative infinity (`-inf`) to represent impossible states
- Set `f[0][0] = 0` since we start in city 0 with 0 vacation days before any week begins

**State Transition:**
For each week `k` from 1 to `K` and each possible destination city `j`:

1. **Option 1 - Stay in the same city:**
   - If we were in city `j` last week, we can stay: `f[k][j] = f[k-1][j]`

2. **Option 2 - Fly from another city:**
   - Check all cities `i` that have flights to city `j`
   - If `flights[i][j] == 1`, we can fly from city `i` to city `j`
   - Update: `f[k][j] = max(f[k][j], f[k-1][i])`

3. **Add vacation days:**
   - After determining the best way to reach city `j` in week `k`, add the vacation days available
   - `f[k][j] += days[j][k-1]` (note: `k-1` because `days` array is 0-indexed for weeks)

**Final Answer:**
After processing all weeks, the maximum vacation days is the maximum value in the last week across all cities: `max(f[K][j])` for all `j` from 0 to `n-1`.

**Time Complexity:** `O(K × n²)` - For each of `K` weeks, we check all `n` cities, and for each city, we check all `n` possible source cities.

**Space Complexity:** `O(K × n)` - We maintain a 2D DP array of size `(K+1) × n`.

The algorithm ensures we explore all possible travel paths while making optimal decisions at each step to maximize total vacation days.

## Example Walkthrough

Let's walk through a small example with 3 cities and 2 weeks:

**Input:**
- `flights = [[0,1,1], [1,0,1], [1,1,0]]` (city 0 connects to 1,2; city 1 connects to 0,2; city 2 connects to 0,1)
- `days = [[1,3], [6,0], [3,3]]` (vacation days per city per week)
- We start in city 0

**DP Table Initialization:**
Create `f[k][j]` where k=0 to 2 (weeks), j=0 to 2 (cities):
```
f[0][0] = 0 (start position)
f[0][1] = -inf (can't start here)
f[0][2] = -inf (can't start here)
```

**Week 1 Processing:**
For each city j, find the best way to reach it and add vacation days:

- **City 0:** Can stay from f[0][0]=0, or fly from city 1 (f[0][1]=-inf) or city 2 (f[0][2]=-inf)
  - Best: stay from city 0 → f[1][0] = 0 + days[0][0] = 0 + 1 = 1

- **City 1:** Can fly from city 0 (f[0][0]=0) since flights[0][1]=1, or stay from f[0][1]=-inf
  - Best: fly from city 0 → f[1][1] = 0 + days[1][0] = 0 + 6 = 6

- **City 2:** Can fly from city 0 (f[0][0]=0) since flights[0][2]=1, or city 1 (f[0][1]=-inf)
  - Best: fly from city 0 → f[1][2] = 0 + days[2][0] = 0 + 3 = 3

After Week 1: `f[1] = [1, 6, 3]`

**Week 2 Processing:**
For each city j, consider all possible origins:

- **City 0:** Can fly from city 1 (f[1][1]=6) or city 2 (f[1][2]=3), or stay (f[1][0]=1)
  - Best: fly from city 1 → f[2][0] = 6 + days[0][1] = 6 + 3 = 9

- **City 1:** Can fly from city 0 (f[1][0]=1) or city 2 (f[1][2]=3), or stay (f[1][1]=6)
  - Best: stay in city 1 → f[2][1] = 6 + days[1][1] = 6 + 0 = 6

- **City 2:** Can fly from city 0 (f[1][0]=1) or city 1 (f[1][1]=6), or stay (f[1][2]=3)
  - Best: fly from city 1 → f[2][2] = 6 + days[2][1] = 6 + 3 = 9

After Week 2: `f[2] = [9, 6, 9]`

**Final Answer:** max(f[2]) = 9 vacation days

**Optimal Path:** Start in city 0 → Fly to city 1 (take 6 vacation days in week 1) → Fly to city 0 or 2 (take 3 vacation days in week 2) = 9 total days

## Python Solution

```python
class Solution:
    def maxVacationDays(self, flights: List[List[int]], days: List[List[int]]) -> int:
        # Number of cities
        num_cities = len(flights)
        # Number of weeks
        num_weeks = len(days[0])

        # dp[week][city] = maximum vacation days possible when ending at city after week
        # Initialize with negative infinity (impossible states)
        dp = [[float('-inf')] * num_cities for _ in range(num_weeks + 1)]

        # Base case: start at city 0 in week 0 with 0 vacation days
        dp[0][0] = 0

        # Iterate through each week
        for week in range(1, num_weeks + 1):
            # Consider each destination city for this week
            for dest_city in range(num_cities):
                # Option 1: Stay in the same city as previous week
                dp[week][dest_city] = dp[week - 1][dest_city]

                # Option 2: Fly from another city
                for src_city in range(num_cities):
                    # Check if there's a flight from src_city to dest_city
                    if flights[src_city][dest_city]:
                        dp[week][dest_city] = max(dp[week][dest_city], dp[week - 1][src_city])

                # Add vacation days available in dest_city during this week
                # Note: week-1 because days array is 0-indexed for weeks
                dp[week][dest_city] += days[dest_city][week - 1]

        # Return the maximum vacation days possible ending at any city after all weeks
        return max(dp[-1][city] for city in range(num_cities))
```

## Time and Space Complexity

**Time Complexity:** `O(K * n^2)`

The algorithm uses dynamic programming with three nested loops:
- The outermost loop iterates through `K` weeks (where `K` is the number of weeks)
- The middle loop iterates through `n` cities (where `n` is the number of cities)
- The innermost loop also iterates through `n` cities to check all possible flights from city `i` to city `j`

For each week `k` and each destination city `j`, we check all `n` possible source cities `i`, resulting in `K * n * n = O(K * n^2)` operations.

**Space Complexity:** `O(K * n)`

The algorithm uses a 2D DP table `f` with dimensions `(K + 1) × n`:
- `K + 1` rows representing weeks from 0 to K
- `n` columns representing the cities

This gives us a space complexity of `O((K + 1) * n) = O(K * n)`.

Note that the input arrays `flights` (size `n × n`) and `days` (size `n × K`) are not counted in the space complexity analysis as they are given inputs rather than auxiliary space used by the algorithm.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting the Option to Stay in the Same City**
The `flights` matrix has `flights[i][i] = 0`, so a transition that only looks at `flights` never allows staying in place. The problem states that you can always stay in your current city.

**The Issue:**
```python
# WRONG: only flights are considered
for dest_city in range(num_cities):
    for src_city in range(num_cities):
        if flights[src_city][dest_city]:
            dp[week][dest_city] = max(dp[week][dest_city], dp[week - 1][src_city])
```
For `flights = [[0, 0], [0, 0]]` and `days = [[1, 1], [7, 7]]`, the correct answer is 2 (stay in city 0 both weeks). This version leaves every state at `-inf`, because no flight exists.

**The Fix:**
Start each state from the "stay" option, as the solution does with `dp[week][dest_city] = dp[week - 1][dest_city]`, or treat `src_city == dest_city` as an allowed transition:
```python
for src_city in range(num_cities):
    if src_city == dest_city or flights[src_city][dest_city]:
        dp[week][dest_city] = max(dp[week][dest_city], dp[week - 1][src_city])
```

### 2. **Incorrect Handling of Unreachable States**
Unreachable states are stored as negative infinity. In Python, `float('-inf') + days[...]` is still `-inf`, so the solution's unconditional `dp[week][dest_city] += days[dest_city][week - 1]` is safe, and `-inf` never wins the final `max`.

**The Issue:**
In languages with fixed-size integers, the sentinel is often `Integer.MIN_VALUE` or `INT_MIN`. Adding vacation days to it overflows and wraps around to a large positive number, which then wins the `max`.

**The Fix:**
Use a sentinel such as `-1` and skip unreachable states explicitly:
```python
if dp[week][dest_city] != float('-inf'):
    dp[week][dest_city] += days[dest_city][week - 1]
```

### 3. **Space Optimization Opportunity Missed**
While not a correctness issue, the solution uses O(K × n) space when it could use O(n) space since we only need the previous week's data.

**The Optimization:**
Use two 1D arrays instead of a 2D array:
```python
prev = [float('-inf')] * num_cities
prev[0] = 0

for week in range(num_weeks):
    curr = [float('-inf')] * num_cities
    for dest_city in range(num_cities):
        for src_city in range(num_cities):
            if src_city == dest_city or flights[src_city][dest_city]:
                curr[dest_city] = max(curr[dest_city], prev[src_city])
        if curr[dest_city] != float('-inf'):
            curr[dest_city] += days[dest_city][week]
    prev = curr

return max(prev)
```

### Complete Corrected Solution:
```python
class Solution:
    def maxVacationDays(self, flights: List[List[int]], days: List[List[int]]) -> int:
        num_cities = len(flights)
        num_weeks = len(days[0])

        dp = [[float('-inf')] * num_cities for _ in range(num_weeks + 1)]
        dp[0][0] = 0

        for week in range(1, num_weeks + 1):
            for dest_city in range(num_cities):
                for src_city in range(num_cities):
                    # Can stay in same city or fly if flight exists
                    if src_city == dest_city or flights[src_city][dest_city]:
                        dp[week][dest_city] = max(dp[week][dest_city], dp[week - 1][src_city])

                # Only add vacation days if city is reachable
                if dp[week][dest_city] != float('-inf'):
                    dp[week][dest_city] += days[dest_city][week - 1]

        return max(dp[-1])
```

