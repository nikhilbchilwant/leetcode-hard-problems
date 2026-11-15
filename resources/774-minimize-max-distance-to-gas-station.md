# 774. Minimize Max Distance to Gas Station

> Source: https://algo.monster/liteproblems/774

## Problem Statement

You have gas stations positioned along a straight line (x-axis), represented by an integer array `stations` where each element indicates the position of a gas station.

Your task is to add exactly `k` new gas stations. These new stations can be placed at any position on the x-axis, including non-integer positions (like 3.5 or 7.25).

After adding all `k` new stations, you need to minimize the "penalty", which is defined as the maximum distance between any two adjacent gas stations (including both original and newly added stations).

For example, if you have stations at positions `[1, 5, 10]` and the distances between adjacent stations are 4 (between 1 and 5) and 5 (between 5 and 10), the penalty would be 5 (the maximum distance). By adding new stations strategically, you can reduce this maximum distance.

The goal is to find the optimal placement of `k` new stations such that the penalty (maximum distance between adjacent stations) is as small as possible. Return this minimum possible penalty value.

The answer will be accepted if it's within `10^-6` of the actual answer, allowing for floating-point precision tolerance.

## Intuition

The key insight is recognizing that this is an optimization problem where we want to minimize the maximum value - a classic scenario for [binary search](/problems/binary_search_intro) on the answer.

Think about it this way: if someone tells you "the maximum distance between adjacent stations must be at most `x`", you can calculate exactly how many new stations you'd need to achieve this. For each existing gap between stations, if the gap is larger than `x`, you'll need to add stations to break it down into smaller segments of size at most `x`.

For a gap of size `d`, the number of new stations needed is `floor(d / x)`. Why? Because we're essentially dividing the gap into segments of size `x`, and the number of divisions tells us how many stations to insert.

This creates a monotonic relationship: as `x` decreases (we want smaller maximum distance), the number of required stations increases. Conversely, as `x` increases (we allow larger maximum distance), fewer stations are needed.

Since we have exactly `k` stations to add, we're looking for the smallest value of `x` such that we need at most `k` stations to ensure no gap exceeds `x`. This is perfect for [binary search](/problems/binary_search_intro):
- If a certain maximum distance `x` requires more than `k` stations, then `x` is too small
- If it requires `k` or fewer stations, then `x` is feasible and we can try an even smaller value

The search space is bounded between 0 (theoretical minimum) and the largest existing gap (or some large value like `10^8`). We keep narrowing down this range until we converge on the optimal maximum distance, using a precision threshold of `10^-6` to determine when to stop.

**Learn more about [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution implements [binary search](/problems/binary_search_intro) on the answer space using the boundary-finding template pattern, adapted for floating-point precision.

### Defining the Feasible Function

For this problem, `feasible(x)` returns `true` if we can achieve a maximum distance of `x` using at most `k` new stations.

This creates a boolean pattern across possible distances:
```
distance: 0.1   0.5   1.0   2.0   3.0   4.0   ...
feasible: false false false true  true  true  ...
```

We want to find the **first true value** - the minimum distance that is feasible.

### Binary Search Template (Floating-Point Adaptation)

Since this involves floating-point numbers, we adapt the template using precision-based termination:

1. Initialize `left = 0` and `right = 1e8`
2. While `right - left > 1e-6`:
   - Calculate `mid = (left + right) / 2`
   - If `feasible(mid)` is true:
     - Search left: `right = mid` (look for smaller feasible distance); `right` always stays feasible
   - Else:
     - Search right: `left = mid`; `left` always stays infeasible
3. Return `left`. When the loop ends, `left` and `right` are within `1e-6` of each other and of the boundary, so `left` is accurate to within `1e-6`

### Feasible Check Implementation

For each gap between adjacent stations of size `d`, we need `floor(d / x)` new stations to ensure no segment exceeds `x`:
- If the gap is 10 and max distance is 3, we need `floor(10/3) = 3` stations
- This creates 4 segments of sizes ≤ 3

Sum across all gaps and check if total ≤ `k`.

### Why This Works

The feasibility function is monotonic: larger `x` values require fewer stations. Once we find a feasible `x`, all larger values are also feasible. Binary search efficiently finds the boundary between infeasible and feasible regions.

**Time Complexity:** `O(n × log(W/ε))` where `n` is stations count, `W` is the search range, `ε` is precision.
**Space Complexity:** `O(1)`

## Example Walkthrough

Let's walk through a concrete example with `stations = [1, 7, 15]` and `k = 2` new stations to add.

**Initial Setup:**
- Original stations: `[1, 7, 15]`
- Gaps between stations: `7-1=6` and `15-7=8`
- Current penalty (max gap): `8`
- We need to add exactly `2` new stations to minimize this penalty

**Binary Search Process:**

**Round 1:**
- `left = 0`, `right = 100` (using 100 as upper bound for simplicity)
- `mid = 50`
- Check if we can achieve max distance of `50` with `2` stations:
  - Gap `[1, 7]`: size `6`, needs `int(6/50) = 0` stations
  - Gap `[7, 15]`: size `8`, needs `int(8/50) = 0` stations
  - Total needed: `0` stations ≤ `2` ✓
- Since feasible, try smaller: `right = 50`

**Round 2:**
- `left = 0`, `right = 50`
- `mid = 25`
- Check if we can achieve max distance of `25`:
  - Gap `[1, 7]`: needs `int(6/25) = 0` stations
  - Gap `[7, 15]`: needs `int(8/25) = 0` stations
  - Total: `0` stations ≤ `2` ✓
- Try smaller: `right = 25`

**Round 3:**
- `left = 0`, `right = 25`
- `mid = 12.5`
- Check if we can achieve max distance of `12.5`:
  - Gap `[1, 7]`: needs `int(6/12.5) = 0` stations
  - Gap `[7, 15]`: needs `int(8/12.5) = 0` stations
  - Total: `0` stations ≤ `2` ✓
- Try smaller: `right = 12.5`

**Round 4:**
- `left = 0`, `right = 12.5`
- `mid = 6.25`
- Check if we can achieve max distance of `6.25`:
  - Gap `[1, 7]`: needs `int(6/6.25) = 0` stations
  - Gap `[7, 15]`: needs `int(8/6.25) = 1` station
  - Total: `1` station ≤ `2` ✓
- Try smaller: `right = 6.25`

**Round 5:**
- `left = 0`, `right = 6.25`
- `mid = 3.125`
- Check if we can achieve max distance of `3.125`:
  - Gap `[1, 7]`: needs `int(6/3.125) = 1` station
  - Gap `[7, 15]`: needs `int(8/3.125) = 2` stations
  - Total: `3` stations > `2` ✗
- Too ambitious, increase: `left = 3.125`

**Round 6:**
- `left = 3.125`, `right = 6.25`
- `mid = 4.6875`
- Check if we can achieve max distance of `4.6875`:
  - Gap `[1, 7]`: needs `int(6/4.6875) = 1` station
  - Gap `[7, 15]`: needs `int(8/4.6875) = 1` station
  - Total: `2` stations = `2` ✓
- Try smaller: `right = 4.6875`

**Continuing...**
The binary search continues, narrowing the range between `left` and `right` until they converge within `1e-6`.

**Final Result:**
The algorithm converges to approximately `4.0`. With this maximum distance:
- We place 1 station in gap `[1, 7]` at position `4.5` → gaps become `[1, 4.5]` (size 3.5) and `[4.5, 7]` (size 2.5)
- We place 1 station in gap `[7, 15]` at position `11` → gaps become `[7, 11]` (size 4) and `[11, 15]` (size 4)
- Final stations: `[1, 4.5, 7, 11, 15]`
- All gaps: `3.5, 2.5, 4, 4`
- Maximum gap (penalty): `4.0`

This demonstrates how binary search efficiently finds the optimal placement strategy by testing different maximum distance thresholds and checking feasibility.

## Python Solution

```python
class Solution:
    def minmaxGasDist(self, stations: List[int], k: int) -> float:
        """
        Find the minimum possible maximum distance between adjacent gas stations
        after adding k new stations optimally.

        Args:
            stations: List of positions of existing gas stations (sorted)
            k: Number of new gas stations to add

        Returns:
            Minimum possible maximum distance between adjacent stations
        """

        def can_achieve_max_distance(max_dist: float) -> bool:
            """
            Check if we can achieve a maximum distance of max_dist between
            adjacent stations by adding at most k new stations.

            For each gap between existing stations, calculate how many new
            stations we need to ensure no gap exceeds max_dist.
            """
            total_stations_needed = 0

            # Check each consecutive pair of existing stations
            for i in range(len(stations) - 1):
                gap = stations[i + 1] - stations[i]
                # Calculate stations needed to make this gap <= max_dist
                # Using int() for floor division of the gap by max_dist
                stations_needed = int(gap / max_dist)
                total_stations_needed += stations_needed

            # Return true if we can achieve this with k or fewer stations
            return total_stations_needed <= k

        # Binary search on the answer (maximum distance)
        # Left bound: 0 (minimum possible distance)
        # Right bound: 1e8 (large enough for any reasonable input)
        left, right = 0.0, 1e8

        # Continue binary search until precision is sufficient (1e-6)
        while right - left > 1e-6:
            mid = (left + right) / 2

            # If we can achieve max distance of mid with k stations
            if can_achieve_max_distance(mid):
                # Try to find a smaller maximum distance
                right = mid
            else:
                # Need a larger maximum distance
                left = mid

        return left
```

## Time and Space Complexity

**Time Complexity:** `O(n * log(M/ε))` where `n` is the number of stations, `M` is the initial search range (`1e8`), and `ε` is the precision (`1e-6`).

- The binary search halves the range `right - left` each iteration and stops when it is at most `1e-6`, so it runs `ceil(log2(1e8 / 1e-6)) = ceil(log2(1e14)) = 47` iterations.
- In each iteration, `can_achieve_max_distance` is called once.
- `can_achieve_max_distance` loops over the `n - 1` consecutive pairs of stations and does a constant amount of work for each pair: `int(gap / max_dist)`.
- Therefore, each call takes `O(n)` time.
- Total time complexity: `O(n * log(M/ε))`

**Space Complexity:** `O(1)`

- The binary search uses only the variables `left`, `right`, and `mid`.
- `can_achieve_max_distance` uses only a running total and loop variables; it does not build any list.
- Overall space complexity: `O(1)` auxiliary space (excluding input).

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Pattern for Floating-Point**

For floating-point binary search, you cannot use the exact integer template (`while left <= right` with `right = mid - 1`). You must adapt for continuous values.

**Incorrect (integer pattern):**
```python
while left <= right:  # Steps of 1 skip over fractional answers
    mid = (left + right) / 2
    if feasible(mid):
        right = mid - 1  # Wrong for floating-point
    else:
        left = mid + 1
```

**Correct (floating-point adaptation):**
```python
while right - left > 1e-6:  # Precision-based termination
    mid = (left + right) / 2
    if feasible(mid):
        right = mid  # No -1 for floats
    else:
        left = mid  # No +1 for floats
return left
```

### 2. **Incorrect Station Counting Formula**

**Pitfall:** Using `ceil((b - a) / x)` to count new stations needed in a gap. This counts *segments*, not *dividing points*.

**Example:**
- Gap: 10 units, max distance: 3 units
- Incorrect: `ceil(10/3) = 4` stations (creates 5 segments - too many!)
- Correct: `int(10/3) = 3` stations (creates 4 segments of sizes ≤ 3)

**Solution:** Use `int(gap / max_dist)` which correctly calculates the number of cuts needed.

### 3. **Floating-Point Precision Errors**

**Pitfall:** Using exact equality or insufficient precision.

```python
# Wrong: May loop forever
while left < right:
    ...
```

**Solution:** Always use a precision threshold:
```python
while right - left > 1e-6:
    ...
```

### 4. **Binary Search Boundary Issues**

**Pitfall:** Setting right boundary too small when k=0.

**Solution:** Use a sufficiently large upper bound like `1e8`, or handle k=0 as a special case returning the maximum gap directly.

