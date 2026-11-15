# 2188. Minimum Time to Finish the Race

> Source: https://algo.monster/liteproblems/2188

## Problem Statement

You're managing a race car that needs to complete a certain number of laps. You have access to different types of tires, and each tire type has specific performance characteristics.

For each tire type, you're given two values:
- `f`: the time (in seconds) it takes to complete the first lap with this tire
- `r`: a degradation factor that makes each subsequent lap slower

The time to complete lap `x` with a tire is calculated as: `f * r^(x-1)` seconds.

For example, if a tire has `f = 3` and `r = 2`:
- Lap 1: `3 * 2^0 = 3` seconds
- Lap 2: `3 * 2^1 = 6` seconds  
- Lap 3: `3 * 2^2 = 12` seconds
- And so on...

You can change tires between laps, but changing tires costs `changeTime` seconds. You have unlimited supplies of each tire type, and you can even change to the same tire type (essentially getting a fresh tire).

Given:
- A 2D array `tires` where `tires[i] = [f_i, r_i]` represents the characteristics of the i-th tire type
- `changeTime`: the time penalty for changing tires
- `numLaps`: the total number of laps to complete

Find the minimum total time needed to complete all `numLaps` laps. You can start with any tire and strategically change tires during the race to minimize the total time.

## Intuition

The key observation is that as we use a tire continuously, each lap becomes exponentially more expensive due to the degradation factor `r`. At some point, it becomes more efficient to change to a fresh tire (paying the `changeTime` penalty) rather than continuing with the degraded one.

First, let's think about when we should stop using a tire. If the next lap with the current tire takes longer than `changeTime + f` (the cost of changing plus one lap with a fresh tire), then we should definitely change. This gives us a natural upper bound on how many consecutive laps we should consider with any single tire.

Since the lap times grow exponentially with factor `r`, and the minimum `r` is 2, the number of consecutive laps we'd ever want to use a single tire is quite limited. In practice, we'll rarely want to use the same tire for more than 17-18 consecutive laps before the exponential growth makes it inefficient.

This leads us to a [dynamic programming](/problems/dynamic_programming_intro) approach:
1. **Precompute the minimum time** to complete exactly `j` consecutive laps using any single tire (trying all tire types and taking the minimum). We only need to compute this for small values of `j` (up to about 18) because beyond that, the exponential growth makes it always better to change tires.

2. **Build up the solution** using [dynamic programming](/problems/dynamic_programming_intro): For each number of laps from 1 to `numLaps`, we consider all possible ways to complete those laps. We could:
   - Use a single tire for all laps (if the number is small enough)
   - Complete some laps with one tire, change, then complete the rest (recursively using our previous computations)

The formula becomes: `f[i] = min(f[i-j] + cost[j] + changeTime)` for all valid `j`, where:
- `f[i]` is the minimum time to complete `i` laps
- `cost[j]` is the precomputed minimum time to do `j` consecutive laps with a single tire
- We add `changeTime` for each tire change

By building up from smaller subproblems to larger ones, we ensure we always have the optimal solution for completing any number of laps up to `numLaps`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a two-phase [dynamic programming](/problems/dynamic_programming_intro) approach:

**Phase 1: Precompute minimum costs for consecutive laps**

```python
cost = [inf] * 18
for f, r in tires:
    i, s, t = 1, 0, f
    while t <= changeTime + f:
        s += t
        cost[i] = min(cost[i], s)
        t *= r
        i += 1
```

For each tire type, we calculate the total time to complete consecutive laps:
- `t` tracks the time for the current lap (starts at `f`, then multiplies by `r` each iteration)
- `s` accumulates the total time for all laps so far
- We continue while `t <= changeTime + f`, meaning the current lap is still faster than changing to a fresh tire
- `cost[i]` stores the minimum time across all tire types to complete exactly `i` consecutive laps

The array size of 18 is sufficient because with the minimum `r = 2`, lap times grow exponentially and quickly exceed the threshold.

**Phase 2: [Dynamic programming](/problems/dynamic_programming_intro) to find minimum time for numLaps**

```python
f = [inf] * (numLaps + 1)
f[0] = -changeTime
for i in range(1, numLaps + 1):
    for j in range(1, min(18, i + 1)):
        f[i] = min(f[i], f[i - j] + cost[j])
    f[i] += changeTime
```

Here we build up the solution:
- `f[i]` represents the minimum time to complete `i` laps
- `f[0] = -changeTime` is a clever initialization that cancels out the first tire change (since we start with a fresh tire, there's no initial change time)
- For each number of laps `i`, we try all possible ways to complete them:
  - Complete the last `j` laps with a single tire (using `cost[j]`)
  - The remaining `i - j` laps were completed optimally before (using `f[i - j]`)
- After finding the minimum, we add `changeTime` to account for the tire change before these laps

The recurrence relation is: `f[i] = min(f[i - j] + cost[j] + changeTime)` for all valid `j`.

The final answer `f[numLaps]` gives us the minimum time to complete all required laps, optimally balancing between tire degradation and change time penalties.

## Example Walkthrough

Let's walk through a concrete example to understand the solution approach.

**Input:**
- `tires = [[2, 3], [3, 4]]` (two tire types)
- `changeTime = 5`
- `numLaps = 4`

**Phase 1: Precompute minimum costs for consecutive laps**

For tire type [2, 3]:
- Lap 1: time = 2 × 3^0 = 2, cumulative = 2
- Lap 2: time = 2 × 3^1 = 6, cumulative = 8
- Lap 3: time = 2 × 3^2 = 18, cumulative = 26
- Stop here since 18 > changeTime + f (5 + 2 = 7)

For tire type [3, 4]:
- Lap 1: time = 3 × 4^0 = 3, cumulative = 3
- Lap 2: time = 3 × 4^1 = 12, cumulative = 15
- Stop here since 12 > changeTime + f (5 + 3 = 8)

Our `cost` array becomes:
- `cost[1] = min(2, 3) = 2` (best single lap)
- `cost[2] = min(8, 15) = 8` (best two consecutive laps)
- `cost[3] = 26` (only first tire can do 3 consecutive)
- `cost[4+] = infinity` (no tire can efficiently do 4+ consecutive laps)

**Phase 2: Build up solution using dynamic programming**

Initialize: `f[0] = -5` (negates first change time)

For `i = 1` (complete 1 lap):
- Option: use last 1 lap with single tire: `f[0] + cost[1] = -5 + 2 = -3`
- Add change time: `f[1] = -3 + 5 = 2`

For `i = 2` (complete 2 laps):
- Option 1: use last 1 lap: `f[1] + cost[1] = 2 + 2 = 4`
- Option 2: use last 2 laps: `f[0] + cost[2] = -5 + 8 = 3`
- Minimum: 3, add change time: `f[2] = 3 + 5 = 8`

For `i = 3` (complete 3 laps):
- Option 1: use last 1 lap: `f[2] + cost[1] = 8 + 2 = 10`
- Option 2: use last 2 laps: `f[1] + cost[2] = 2 + 8 = 10`
- Option 3: use last 3 laps: `f[0] + cost[3] = -5 + 26 = 21`
- Minimum: 10, add change time: `f[3] = 10 + 5 = 15`

For `i = 4` (complete 4 laps):
- Option 1: use last 1 lap: `f[3] + cost[1] = 15 + 2 = 17`
- Option 2: use last 2 laps: `f[2] + cost[2] = 8 + 8 = 16`
- Option 3: use last 3 laps: `f[1] + cost[3] = 2 + 26 = 28`
- Minimum: 16, add change time: `f[4] = 16 + 5 = 21`

**Result:** The minimum time to complete 4 laps is 21 seconds.

This corresponds to the strategy: Use first tire for 2 laps (8 seconds), change tires (5 seconds), use first tire for 2 more laps (8 seconds) = 21 total seconds.

## Python Solution

```python
class Solution:
    def minimumFinishTime(
        self, tires: List[List[int]], changeTime: int, numLaps: int
    ) -> int:
        # Maximum consecutive laps to consider with same tire (18 is sufficient 
        # because lap time grows exponentially and will exceed changeTime + first_lap)
        MAX_CONSECUTIVE_LAPS = 18
      
        # min_cost_consecutive[i] = minimum time to complete i consecutive laps with any single tire
        min_cost_consecutive = [float('inf')] * MAX_CONSECUTIVE_LAPS
      
        # Calculate minimum cost for each possible number of consecutive laps
        for first_lap_time, ratio in tires:
            consecutive_laps = 1
            total_time = 0
            current_lap_time = first_lap_time
          
            # Keep adding laps until it's no longer beneficial to continue with same tire
            # (when current lap time exceeds the cost of changing tire + starting fresh)
            while current_lap_time <= changeTime + first_lap_time:
                total_time += current_lap_time
                min_cost_consecutive[consecutive_laps] = min(
                    min_cost_consecutive[consecutive_laps], 
                    total_time
                )
                current_lap_time *= ratio
                consecutive_laps += 1
      
        # dp[i] = minimum time to complete i laps (including tire changes)
        dp = [float('inf')] * (numLaps + 1)
        dp[0] = -changeTime  # Base case: compensate for the extra changeTime added later
      
        # Dynamic programming to find minimum time for each lap count
        for lap in range(1, numLaps + 1):
            # Try all possible ways to complete the last segment of consecutive laps
            for last_segment_laps in range(1, min(MAX_CONSECUTIVE_LAPS, lap + 1)):
                dp[lap] = min(
                    dp[lap], 
                    dp[lap - last_segment_laps] + min_cost_consecutive[last_segment_laps]
                )
            # Add tire change time (for transitioning to this segment)
            dp[lap] += changeTime
      
        return dp[numLaps]
```

## Time and Space Complexity

**Time Complexity:** `O(n + m²)` where `n` is the number of tires and `m` is the number of laps (`numLaps`).

- The first loop iterates through each tire and for each tire, it runs a while loop. The while loop is bounded by the condition `t <= changeTime + f`. Since `t` grows exponentially (`t *= r`), this loop runs at most 17 times (hence the array size of 18). So this part is `O(17n) = O(n)`.

- The second part uses dynamic programming with nested loops:
  - Outer loop runs `numLaps` times
  - Inner loop runs at most `min(18, i+1)` times, which is bounded by 17
  - However, when considering all iterations, the inner loop effectively runs `O(numLaps)` times in total across all outer loop iterations

  This gives us `O(numLaps × min(17, numLaps)) = O(numLaps²)` for large values of `numLaps`, or `O(17 × numLaps) = O(numLaps)` when `numLaps` is small.

Overall: `O(n + m²)` where `m = numLaps`.

**Space Complexity:** `O(m)` where `m` is the number of laps.

- The `cost` array has a fixed size of 18, so it takes `O(1)` space.
- The `f` array has size `numLaps + 1`, taking `O(numLaps)` space.

Overall space complexity: `O(numLaps)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

**1. Incorrect Boundary for Consecutive Laps Calculation**

One of the most common mistakes is using the wrong condition to determine when to stop using the same tire. Many developers incorrectly compare the current lap time with just `changeTime`:

```python
# WRONG: Missing the first lap time in comparison
while current_lap_time <= changeTime:
    # ... rest of the code
```

The correct condition should be `current_lap_time <= changeTime + first_lap_time`. This is because when deciding whether to continue with a degraded tire or change to a fresh one, we need to compare:
- Continuing with degraded tire: `current_lap_time` seconds
- Changing to fresh tire: `changeTime + first_lap_time` seconds (change time plus the time for a fresh lap)

**2. Overflow Issues with Exponential Growth**

When calculating consecutive lap times, the values can grow exponentially and cause integer overflow:

```python
# POTENTIAL ISSUE: No bounds checking
for first_lap_time, ratio in tires:
    current_lap_time = first_lap_time
    for i in range(1, 20):  # Arbitrary large number
        total_time += current_lap_time
        current_lap_time *= ratio  # Can overflow!
```

**Solution:** Always include the boundary check `current_lap_time <= changeTime + first_lap_time` before multiplication to prevent overflow and unnecessary calculations.

**3. Off-by-One Errors in DP Initialization**

A subtle but critical pitfall is incorrectly initializing the DP base case:

```python
# WRONG: Doesn't account for no initial tire change
dp[0] = 0

# CORRECT: Compensates for the extra changeTime added in the loop
dp[0] = -changeTime
```

Since the DP recurrence adds `changeTime` for every segment (including the first one), we need to initialize `dp[0] = -changeTime` to cancel out the unnecessary tire change time at the very beginning of the race.

**4. Incorrect Array Sizing**

Using a fixed array size without proper justification can lead to either:
- **Too small:** Missing optimal solutions for certain inputs
- **Too large:** Wasting memory and computation time

```python
# WRONG: Arbitrary size without justification
min_cost_consecutive = [float('inf')] * 50

# CORRECT: Size 18 is sufficient based on problem constraints
min_cost_consecutive = [float('inf')] * 18
```

The size 18 is chosen because with minimum ratio = 2 and maximum changeTime = 10^5, the lap time will exceed `changeTime + first_lap_time` well before reaching 18 consecutive laps.

**5. Not Handling Edge Cases in Range**

When iterating through possible segment sizes in the DP phase:

```python
# WRONG: May go out of bounds
for last_segment_laps in range(1, 18):
    dp[lap] = min(dp[lap], dp[lap - last_segment_laps] + ...)

# CORRECT: Ensures we don't exceed available laps
for last_segment_laps in range(1, min(18, lap + 1)):
    dp[lap] = min(dp[lap], dp[lap - last_segment_laps] + ...)
```

Always use `min(MAX_CONSECUTIVE_LAPS, lap + 1)` to ensure you don't try to use more consecutive laps than the total laps remaining.

