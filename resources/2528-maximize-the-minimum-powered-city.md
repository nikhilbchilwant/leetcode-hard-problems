# 2528. Maximize the Minimum Powered City

> Source: https://algo.monster/liteproblems/2528

## Problem Statement

You have `n` cities arranged in a line (indexed from 0 to n-1), and each city has some number of power stations given by the array `stations`, where `stations[i]` represents the number of power stations in city `i`.

Each power station can provide power to cities within a fixed range `r`. Specifically, a power station located in city `i` can provide power to all cities `j` where `|i - j| <= r`. The power of a city is defined as the total number of power stations that can provide power to it.

The government wants to build `k` additional power stations. Each new power station:
- Can be built in any city
- Has the same range `r` as existing power stations
- Multiple power stations can be built in the same city

Your task is to find the **maximum possible minimum power** among all cities after optimally placing the `k` additional power stations.

In other words, you want to maximize the power of the weakest city by strategically placing the new power stations.

**Example:**
- If you have cities with initial power values and you can add `k` stations, you want to place them such that the city with the least power has as much power as possible.
- The goal is to maximize the minimum value across all cities.

**Constraints:**
- The range `r` determines how far a power station's influence extends
- You have exactly `k` new power stations to place
- You need to return the highest achievable minimum power level among all cities

## Intuition

The key insight is that we're trying to maximize the minimum power across all cities. This is a classic "maximize the minimum" optimization problem, which naturally leads us to think about [binary search](/problems/binary_search_intro).

Why [binary search](/problems/binary_search_intro)? Consider this: if we can achieve a minimum power of `x` across all cities with `k` additional stations, then we can definitely achieve any minimum power less than `x`. Conversely, if we cannot achieve a minimum power of `x`, then we cannot achieve any value greater than `x` either. This monotonic property makes binary search the perfect tool.

The next question is: given a target minimum power `x`, how do we check if it's achievable with `k` additional stations?

We can use a [greedy](/problems/greedy_intro) approach. As we traverse each city from left to right:
- If a city's current power is less than `x`, we need to add power stations
- Where should we place them? Since we're going left to right, we want to place new stations as far right as possible while still covering the current city. This way, each new station can potentially help future cities too
- The optimal position is at `min(i + r, n - 1)` - as far right as we can go while still covering city `i`

To efficiently track how many power stations affect each city, we use a difference array technique. This allows us to:
1. Initially compute how many existing stations cover each city using range updates
2. During our check, efficiently add new stations and update the power coverage for multiple cities at once

The difference array is powerful here because adding a power station at position `j` affects all cities in the range `[j - r, j + r]`. Instead of updating each city individually, we can mark the start and end of this range in the difference array, then use prefix sums to get the actual values.

The combination of [binary search](/problems/binary_search_intro) (to find the optimal minimum power) + [greedy](/problems/greedy_intro) placement (to minimize stations needed) + difference arrays (for efficient range updates) gives us an elegant solution to this complex optimization problem.

**Learn more about [Greedy](/problems/greedy_intro), [Queue](/problems/queue_intro), [Binary Search](/problems/binary_search_intro), [Prefix Sum](/problems/subarray_sum) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

The solution consists of two main parts: [binary search](/problems/binary_search_intro) for the answer and a [greedy](/problems/greedy_intro) check function with difference arrays.

**Step 1: Initial Power Calculation Using Difference Arrays**

First, we need to calculate how many power stations initially cover each city. We use a difference array `d` to efficiently handle range updates:
- For each station at position `i` with `v` stations, it affects cities in range `[max(0, i - r), min(i + r, n - 1)]`
- We add `v` at the start of the range and subtract `v` at position `right + 1`
- After processing all stations, we use [prefix sum](/problems/subarray_sum) (`accumulate`) to get the actual power for each city in array `s`

```python
d = [0] * (n + 1)
for i, v in enumerate(stations):
    left, right = max(0, i - r), min(i + r, n - 1)
    d[left] += v
    d[right + 1] -= v
s = list(accumulate(d))
```

**Step 2: [Binary Search](/problems/binary_search_intro) Setup**

We search for the maximum achievable minimum power:
- Left boundary: `0` (minimum possible power)
- Right boundary: `1 << 40` (a sufficiently large value)
- We use [binary search](/problems/binary_search_intro) to find the largest value `x` such that we can make all cities have at least `x` power using at most `k` additional stations

```python
left, right = 0, 1 << 40
while left < right:
    mid = (left + right + 1) >> 1
    if check(mid, k):
        left = mid
    else:
        right = mid - 1
```

**Step 3: The Check Function - [Greedy](/problems/greedy_intro) Placement with Difference Arrays**

The `check(x, k)` function determines if we can achieve minimum power `x` for all cities:

1. Initialize a new difference array `d` to track additional power stations we place
2. Maintain a running sum `t` to track cumulative effect of new stations
3. For each city `i`:
   - Update `t` with the difference array value at position `i`
   - Calculate deficit: `dist = x - (s[i] + t)` where `s[i]` is initial power and `t` is power from new stations
   - If `dist > 0`, we need to add stations:
     - If `dist > k`, we can't achieve target `x`, return `False`
     - Otherwise, greedily place `dist` stations at position `j = min(i + r, n - 1)` (rightmost position that still covers city `i`)
     - Update difference array for the range these new stations cover: `[max(0, j - r), min(j + r, n - 1)]`
     - Deduct from available stations: `k -= dist`
     - Update running sum: `t += dist`

```python
def check(x, k):
    d = [0] * (n + 1)
    t = 0
    for i in range(n):
        t += d[i]
        dist = x - (s[i] + t)
        if dist > 0:
            if k < dist:
                return False
            k -= dist
            j = min(i + r, n - 1)
            left, right = max(0, j - r), min(j + r, n - 1)
            d[left] += dist
            d[right + 1] -= dist
            t += dist
    return True
```

The [greedy](/problems/greedy_intro) strategy of placing stations as far right as possible ensures we maximize coverage for future cities while satisfying the current city's requirements. The difference array technique allows us to efficiently update power coverage for multiple cities when placing new stations, avoiding the need for nested loops.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Given:**
- `stations = [1, 2, 4, 5, 0]` (5 cities with initial power stations)
- `r = 1` (each station covers cities within distance 1)
- `k = 2` (we can add 2 new power stations)

**Step 1: Calculate Initial Power for Each City**

First, we compute how many stations initially cover each city using a difference array.

For each existing station:
- Station at city 0 (1 station): covers cities [0, 1]
  - `d[0] += 1`, `d[2] -= 1`
- Station at city 1 (2 stations): covers cities [0, 2]
  - `d[0] += 2`, `d[3] -= 2`
- Station at city 2 (4 stations): covers cities [1, 3]
  - `d[1] += 4`, `d[4] -= 4`
- Station at city 3 (5 stations): covers cities [2, 4]
  - `d[2] += 5`, `d[5] -= 5`
- Station at city 4 (0 stations): no effect

Difference array: `d = [3, 4, 4, -2, -4, -5]`

Using prefix sum: `s = [3, 7, 11, 9, 5, 0]`

Initial power for each city: `[3, 7, 11, 9, 5]`
- City 0: power = 3
- City 1: power = 7
- City 2: power = 11
- City 3: power = 9
- City 4: power = 5

Current minimum power = 3 (city 0)

**Step 2: Binary Search for Maximum Minimum Power**

Binary search range: `[0, 1<<40]`

Let's check if we can achieve minimum power = 5:

**Check if minimum power = 5 is achievable:**

Traverse each city from left to right:
- City 0: Current power = 3, need 5 → deficit = 2
  - Place 2 stations at position `min(0 + 1, 4) = 1`
  - These stations at city 1 cover cities [0, 2]
  - Update: `k = 0` stations remaining
  - Running power additions: city 0 gets +2, city 1 gets +2, city 2 gets +2

- City 1: Current power = 7 + 2 = 9 ≥ 5 ✓
- City 2: Current power = 11 + 2 = 13 ≥ 5 ✓
- City 3: Current power = 9 + 0 = 9 ≥ 5 ✓
- City 4: Current power = 5 + 0 = 5 ≥ 5 ✓

Result: We can achieve minimum power = 5 with exactly 2 stations.

**Check if minimum power = 6 is achievable:**

- City 0: Current power = 3, need 6 → deficit = 3
  - We only have k = 2 stations available, but need 3
  - Return False

Result: Cannot achieve minimum power = 6.

**Final Answer:** The maximum achievable minimum power is 5.

The solution efficiently:
1. Uses difference arrays to track power coverage without nested loops
2. Greedily places new stations as far right as possible to maximize coverage
3. Uses binary search to find the optimal answer in O(n log(max_power)) time

## Python Solution

```python
class Solution:
    def maxPower(self, stations: List[int], r: int, k: int) -> int:
        def can_achieve_min_power(target_min_power: int, available_stations: int) -> bool:
            """
            Check if we can achieve at least target_min_power for all cities
            using at most available_stations additional power stations.

            Uses a greedy approach: place new stations as far right as possible
            when a city needs more power.
            """
            # Track power additions using difference array
            power_additions = [0] * (num_cities + 1)
            current_added_power = 0

            for city_idx in range(num_cities):
                # Update current added power based on difference array
                current_added_power += power_additions[city_idx]

                # Calculate power deficit for current city
                current_city_power = initial_power[city_idx] + current_added_power
                power_deficit = target_min_power - current_city_power

                if power_deficit > 0:
                    # Check if we have enough stations to cover the deficit
                    if available_stations < power_deficit:
                        return False

                    # Greedily place new stations as far right as possible
                    # while still covering the current city
                    available_stations -= power_deficit

                    # Find the rightmost position to place new stations
                    rightmost_position = min(city_idx + r, num_cities - 1)

                    # Calculate the range affected by stations at rightmost_position
                    affected_start = max(0, rightmost_position - r)
                    affected_end = min(rightmost_position + r, num_cities - 1)

                    # Update difference array to add power to affected range
                    power_additions[affected_start] += power_deficit
                    power_additions[affected_end + 1] -= power_deficit

                    # Update current added power for this city
                    current_added_power += power_deficit

            return True

        num_cities = len(stations)

        # Calculate initial power for each city using difference array technique
        # Each station affects cities within range r
        power_diff = [0] * (num_cities + 1)

        for station_idx, station_power in enumerate(stations):
            # Calculate the range of cities affected by this station
            left_bound = max(0, station_idx - r)
            right_bound = min(station_idx + r, num_cities - 1)

            # Update difference array
            power_diff[left_bound] += station_power
            power_diff[right_bound + 1] -= station_power

        # Convert difference array to actual power values using prefix sum
        initial_power = list(accumulate(power_diff))

        # Binary search for the maximum achievable minimum power
        left, right = 0, 1 << 40  # Upper bound is 2^40 (large enough for the problem)

        while left < right:
            # Use upper mid to maximize the result
            mid = (left + right + 1) >> 1

            if can_achieve_min_power(mid, k):
                # Can achieve mid, try for higher minimum power
                left = mid
            else:
                # Cannot achieve mid, try lower values
                right = mid - 1

        return left
```

## Time and Space Complexity

**Time Complexity:** `O(n × log M)` where `n` is the length of the stations array (number of cities) and `M = 2^40`.

The binary search runs from `left = 0` to `right = 2^40`, performing `O(log M)` iterations where `M = 2^40`. In each binary search iteration, the `check` function is called, which contains a single loop that iterates through all `n` cities. Within this loop, all operations (including difference array updates and accumulation) are `O(1)`. Therefore, each call to `check` takes `O(n)` time. The initial preprocessing (creating the difference array and computing prefix sums) also takes `O(n)` time. Thus, the overall time complexity is `O(n) + O(n × log M) = O(n × log M)`.

**Space Complexity:** `O(n)` where `n` is the number of cities.

The algorithm uses several arrays of size `n+1`: the difference array `d` (used both in preprocessing and in the `check` function), and the prefix sum array `s`. Each of these arrays requires `O(n)` space. The binary search and other variables use `O(1)` additional space. Therefore, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Greedy Placement Strategy**

**Pitfall:** A common mistake is placing new power stations at the position of the current city that needs power, rather than at the optimal position within range.

**Why it's wrong:** When city `i` needs more power, placing stations directly at city `i` might not be optimal. We want to maximize coverage for future cities while still satisfying the current city's needs.

**Example:** If city 2 needs power and `r = 2`, placing a station at city 2 covers cities [0, 1, 2, 3, 4]. But placing it at city 4 (the rightmost position that still covers city 2) covers cities [2, 3, 4, 5, 6], which helps more with upcoming cities.

**Solution:** Always place new stations at position `min(i + r, n - 1)` when city `i` needs power. This greedy rightmost placement ensures maximum coverage for cities we haven't processed yet.

### 2. **Difference Array Boundary Errors**

**Pitfall:** Forgetting to allocate `n + 1` size for the difference array, leading to index out of bounds errors when updating `d[right + 1]`.

**Why it happens:** When updating a range `[left, right]` in a difference array, we need to set `d[right + 1] -= value`. If `right = n - 1`, then `right + 1 = n`, requiring array size of at least `n + 1`.

**Solution:** Always initialize difference arrays with size `n + 1`:
```python
d = [0] * (n + 1)  # Not [0] * n
```

### 3. **Accumulating Power Updates Incorrectly**

**Pitfall:** Not maintaining the running sum `t` correctly when checking if we can achieve a target power level. Some implementations reset `t` or forget to update it when placing new stations.

**Why it's problematic:** The variable `t` tracks the cumulative effect of all newly placed power stations up to the current position. If not maintained properly, the power calculation `s[i] + t` will be incorrect.

**Solution:**
- Update `t` at the beginning of each iteration: `t += d[i]`
- When placing new stations, immediately update `t` if they affect the current city: `t += dist`

### 4. **Binary Search Boundary Selection**

**Pitfall:** Using standard binary search with `mid = (left + right) // 2` instead of `mid = (left + right + 1) // 2` when searching for maximum value.

**Why it matters:** When searching for the maximum achievable value, using the lower mid can cause infinite loops when `left = right - 1`. The condition `check(mid)` being true would set `left = mid`, keeping us stuck.

**Solution:** When maximizing, use upper mid:
```python
mid = (left + right + 1) >> 1  # Upper mid for maximization
if check(mid, k):
    left = mid
else:
    right = mid - 1
```

### 5. **Modifying Initial Power Array During Check**

**Pitfall:** Directly modifying the initial power array `s` inside the check function, which corrupts data for subsequent binary search iterations.

**Why it's wrong:** The binary search calls `check()` multiple times with different target values. If `s` is modified, subsequent checks will use corrupted initial power values.

**Solution:** Never modify the initial power array. Use a separate difference array and running sum to track additions:
```python
# DON'T DO: s[j] += dist for j in range(...)
# DO: Use difference array d and running sum t
d[left] += dist
d[right + 1] -= dist
t += dist  # Update running sum for current position
```

