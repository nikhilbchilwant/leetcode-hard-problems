# 2361. Minimum Costs Using the Train Line

> Source: https://algo.monster/liteproblems/2361

## Problem Statement

You are traveling on a train line that has two routes through a city: a regular route and an express route. Both routes pass through the same `n + 1` stops (labeled from `0` to `n`). You start at stop `0` on the regular route.

You're given two arrays that describe the travel costs:
- `regular[i]`: the cost to travel from stop `i-1` to stop `i` using the regular route
- `express[i]`: the cost to travel from stop `i-1` to stop `i` using the express route

Both arrays are 1-indexed and have length `n`.

There's also an `expressCost` value, which is the fee you pay to switch from the regular route to the express route.

The switching rules are:
- Switching from regular to express costs `expressCost` (you pay this fee every time you make this switch)
- Switching from express back to regular is free
- Staying on the express route has no additional cost

Your task is to find the minimum cost to reach each stop from stop `0`. Return a 1-indexed array `costs` of length `n`, where `costs[i]` represents the minimum cost to reach stop `i` from stop `0`.

The key insight is that at any stop, you can arrive via either route, and you want to track the cheapest way to get there. The [dynamic programming](/problems/dynamic_programming_intro) approach maintains two states:
- `f[i]`: minimum cost to reach stop `i` via the regular route
- `g[i]`: minimum cost to reach stop `i` via the express route

For each stop, you calculate both possibilities and take the minimum of the two as your answer for that stop.

## Intuition

The key observation is that at each stop, we need to track which route we're currently on because the cost of future transitions depends on our current route. If we're on the regular route, we can either continue on it for free or pay `expressCost` to switch to express. If we're on the express route, we can continue on it or switch back to regular for free.

This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach where we maintain two separate states for each stop - one for arriving via regular route and one for arriving via express route. Why two states? Because the minimum cost to reach a stop might differ based on which route we use to arrive there, and this affects our future choices.

Think of it like this: even if arriving at stop `i` via express is more expensive than via regular, it might still be worth it if the express route offers significant savings for stops `i+1`, `i+2`, etc. We can't just greedily pick the cheapest option at each stop - we need to consider both possibilities.

For each stop `i`, we calculate:
- The minimum cost to arrive via regular: either we were already on regular (cost is `f[i-1] + regular[i]`) or we were on express and stayed on regular when moving to this stop (cost is `g[i-1] + regular[i]`)
- The minimum cost to arrive via express: either we switch from regular to express (cost is `f[i-1] + expressCost + express[i]`) or we were already on express (cost is `g[i-1] + express[i]`)

The answer for each stop is simply the minimum of these two values, since we can reach any stop from either route. This builds up our solution incrementally from stop `0` to stop `n`, ensuring we always have the optimal cost for each stop considering all possible route combinations.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement a [dynamic programming](/problems/dynamic_programming_intro) solution using two arrays to track the minimum costs for each route:

1. **Initialize the DP arrays:**
   - `f[i]` represents the minimum cost to reach stop `i` via the regular route
   - `g[i]` represents the minimum cost to reach stop `i` via the express route
   - Set `f[0] = 0` (we start on regular route at stop 0)
   - Set `g[0] = inf` (impossible to start on express route)

2. **Build the solution iteratively:**
   For each stop `i` from 1 to n, we calculate both `f[i]` and `g[i]`:

   **For arriving via regular route:**
   ```
   f[i] = min(f[i-1] + regular[i], g[i-1] + regular[i])
   ```
   This means we take the minimum of:
   - Coming from stop `i-1` on regular and continuing on regular: `f[i-1] + regular[i]`
   - Coming from stop `i-1` on express and switching to regular (free): `g[i-1] + regular[i]`

   **For arriving via express route:**
   ```
   g[i] = min(f[i-1] + expressCost + express[i], g[i-1] + express[i])
   ```
   This means we take the minimum of:
   - Coming from stop `i-1` on regular and switching to express: `f[i-1] + expressCost + express[i]`
   - Coming from stop `i-1` on express and continuing on express: `g[i-1] + express[i]`

3. **Construct the answer:**
   For each stop `i`, the minimum cost is:
   ```
   cost[i-1] = min(f[i], g[i])
   ```
   We use `i-1` because the output array is 1-indexed while our iteration uses 0-indexed stops.

The algorithm processes each stop exactly once, computing two values (regular and express costs) per stop. The space complexity is O(n) for storing the DP arrays, and the time complexity is O(n) since we iterate through all stops once.

The implementation uses `zip(regular, express)` with `enumerate(_, 1)` to elegantly iterate through both arrays simultaneously while maintaining the correct 1-based indexing for stops.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Input:**
- `regular = [1, 6, 9, 5]` (costs to travel via regular route)
- `express = [5, 2, 3, 10]` (costs to travel via express route)
- `expressCost = 3` (fee to switch from regular to express)

We need to find the minimum cost to reach stops 1, 2, 3, and 4 from stop 0.

**Initial Setup:**
- `f[0] = 0` (start at stop 0 on regular route)
- `g[0] = inf` (cannot start on express route)

**Stop 1:**
- To arrive via regular: `f[1] = min(f[0] + regular[1], g[0] + regular[1]) = min(0 + 1, inf + 1) = 1`
- To arrive via express: `g[1] = min(f[0] + expressCost + express[1], g[0] + express[1]) = min(0 + 3 + 5, inf + 5) = 8`
- Minimum cost to reach stop 1: `min(1, 8) = 1`

**Stop 2:**
- To arrive via regular: `f[2] = min(f[1] + regular[2], g[1] + regular[2]) = min(1 + 6, 8 + 6) = 7`
- To arrive via express: `g[2] = min(f[1] + expressCost + express[2], g[1] + express[2]) = min(1 + 3 + 2, 8 + 2) = 6`
- Minimum cost to reach stop 2: `min(7, 6) = 6`

**Stop 3:**
- To arrive via regular: `f[3] = min(f[2] + regular[3], g[2] + regular[3]) = min(7 + 9, 6 + 9) = 15`
- To arrive via express: `g[3] = min(f[2] + expressCost + express[3], g[2] + express[3]) = min(7 + 3 + 3, 6 + 3) = 9`
- Minimum cost to reach stop 3: `min(15, 9) = 9`

**Stop 4:**
- To arrive via regular: `f[4] = min(f[3] + regular[4], g[3] + regular[4]) = min(15 + 5, 9 + 5) = 14`
- To arrive via express: `g[4] = min(f[3] + expressCost + express[4], g[3] + express[4]) = min(15 + 3 + 10, 9 + 10) = 19`
- Minimum cost to reach stop 4: `min(14, 19) = 14`

**Final Answer:** `[1, 6, 9, 14]`

The optimal path in this example:
- Stop 0 → Stop 1: Use regular route (cost: 1)
- Stop 1 → Stop 2: Switch to express (cost: 3 + 2 = 5, total: 6)
- Stop 2 → Stop 3: Stay on express (cost: 3, total: 9)
- Stop 3 → Stop 4: Switch back to regular for free (cost: 5, total: 14)

## Python Solution

```python
from typing import List
from math import inf


class Solution:
    def minimumCosts(
        self, regular: List[int], express: List[int], expressCost: int
    ) -> List[int]:
        """
        Calculate minimum costs to reach each stop using either regular or express lane.

        Args:
            regular: Cost to travel each segment on regular lane
            express: Cost to travel each segment on express lane
            expressCost: One-time cost to switch from regular to express lane

        Returns:
            List of minimum costs to reach each stop (1-indexed positions)
        """
        n = len(regular)

        # Dynamic programming arrays
        # regular_min_cost[i]: minimum cost to reach position i using regular lane
        regular_min_cost = [0] * (n + 1)

        # express_min_cost[i]: minimum cost to reach position i using express lane
        express_min_cost = [inf] * (n + 1)

        # Result array to store minimum cost to reach each stop
        result = [0] * n

        # Process each stop
        for i in range(1, n + 1):
            regular_cost = regular[i - 1]
            express_cost = express[i - 1]

            # Calculate minimum cost to reach position i using regular lane
            # Option 1: Stay on regular lane from position i-1
            # Option 2: Switch from express lane to regular lane (no switching cost)
            regular_min_cost[i] = min(
                regular_min_cost[i - 1] + regular_cost,
                express_min_cost[i - 1] + regular_cost
            )

            # Calculate minimum cost to reach position i using express lane
            # Option 1: Switch from regular lane to express lane (pay switching cost)
            # Option 2: Stay on express lane from position i-1
            express_min_cost[i] = min(
                regular_min_cost[i - 1] + expressCost + express_cost,
                express_min_cost[i - 1] + express_cost
            )

            # Store the minimum cost between both lanes for position i
            result[i - 1] = min(regular_min_cost[i], express_min_cost[i])

        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm iterates through the input arrays exactly once using a single for loop that runs from index 1 to n. Within each iteration, the operations performed are:
- Two minimum comparisons for calculating `f[i]`
- Two minimum comparisons for calculating `g[i]`
- One minimum comparison for calculating `cost[i-1]`

All these operations take constant time `O(1)`. Since the loop runs n times and each iteration performs constant work, the overall time complexity is `O(n)`.

**Space Complexity:** `O(n)`

The algorithm uses three arrays:
- Array `f` of size `n + 1` to store the minimum cost to reach each station using the regular lane
- Array `g` of size `n + 1` to store the minimum cost to reach each station using the express lane
- Array `cost` of size `n` to store the final minimum costs

Each array requires `O(n)` space, and since we're using a constant number of such arrays, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Initialization of Express Lane Cost**

One of the most common mistakes is initializing `express_min_cost[0]` with 0 instead of infinity. This would incorrectly assume that you can start on the express lane at no cost, when in reality you must start on the regular lane.

**Incorrect:**
```python
express_min_cost = [0] * (n + 1)  # Wrong! Can't start on express lane
```

**Correct:**
```python
express_min_cost = [inf] * (n + 1)  # Correct - impossible to start on express
```

### 2. **Forgetting the Express Switch Cost**

Another frequent error is forgetting to add `expressCost` when switching from regular to express lane. This leads to underestimating the total cost.

**Incorrect:**
```python
express_min_cost[i] = min(
    regular_min_cost[i - 1] + express_cost,  # Missing expressCost!
    express_min_cost[i - 1] + express_cost
)
```

**Correct:**
```python
express_min_cost[i] = min(
    regular_min_cost[i - 1] + expressCost + express_cost,  # Include switch cost
    express_min_cost[i - 1] + express_cost
)
```

### 3. **Index Confusion Between 0-based and 1-based Arrays**

The problem uses 1-indexed arrays for `regular` and `express`, but the DP arrays are 0-indexed. Mixing these up can cause off-by-one errors or array out-of-bounds exceptions.

**Incorrect:**
```python
for i in range(n):  # Wrong iteration range
    regular_cost = regular[i]  # Wrong index
    result[i] = min(regular_min_cost[i], express_min_cost[i])  # Wrong result index
```

**Correct:**
```python
for i in range(1, n + 1):  # Iterate from 1 to n
    regular_cost = regular[i - 1]  # Access 1-indexed array correctly
    result[i - 1] = min(regular_min_cost[i], express_min_cost[i])  # Store in 0-indexed result
```

### 4. **Not Considering Both Routes for Final Answer**

Some implementations might only return the cost of reaching each stop via one route, forgetting to take the minimum of both routes.

**Incorrect:**
```python
result[i - 1] = regular_min_cost[i]  # Only considering regular route
```

**Correct:**
```python
result[i - 1] = min(regular_min_cost[i], express_min_cost[i])  # Consider both routes
```

### 5. **Space Optimization Pitfall**

While it's tempting to optimize space by using only the previous state, this problem requires careful handling since each state depends on both previous states. Attempting to use single variables instead of arrays without proper care can lead to incorrect results.

**Problematic optimization:**
```python
# Trying to use only two variables instead of arrays
prev_regular = 0
prev_express = inf
# This approach requires careful variable management to avoid overwriting needed values
```

**Better approach:**
Keep the full arrays as shown in the solution, or if space optimization is needed, use a rolling array technique with proper temporary variables to avoid overwriting values before they're used.

