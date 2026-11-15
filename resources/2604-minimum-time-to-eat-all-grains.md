# 2604. Minimum Time to Eat All Grains

> Source: https://algo.monster/liteproblems/2604

## Problem Statement

You have `n` hens and `m` grains positioned on a number line. The initial positions of the hens are given in an integer array `hens` of size `n`, and the positions of the grains are given in an integer array `grains` of size `m`.

A hen can eat a grain if they are at the same position, and eating takes no time. Each hen can eat multiple grains.

In each second, a hen can move either left or right by 1 unit. All hens can move simultaneously and independently of each other.

Your task is to find the minimum time required for the hens to eat all the grains if they move optimally.

For example, if a hen is at position 5 and there are grains at positions 3 and 7, the hen could move left to position 3 (taking 2 seconds) to eat that grain, then move right to position 7 (taking 4 seconds) to eat the other grain, for a total of 6 seconds. However, if another hen helps, the time could be reduced.

The goal is to coordinate all hens' movements to minimize the maximum time any hen needs to travel to ensure all grains are consumed.

## Intuition

The key insight is that if we can eat all grains within time `t`, then we can definitely eat all grains within any time greater than `t`. This monotonic property suggests using [binary search](/problems/binary_search_intro) on the answer.

Think about it this way: if we fix a time limit `t`, we can check whether it's possible for the hens to eat all grains within this time. Each hen has a "reachable range" - the set of positions it can visit within `t` seconds. The question becomes: can we assign grains to hens such that each grain falls within some hen's reachable range?

To make this assignment optimal, we should process hens and grains in sorted order from left to right. Why? Because if a hen on the left skips a grain to go further right, a hen on the right would need to come back left to eat that skipped grain, which is inefficient.

For each hen at position `x` with time budget `t`, it can reach any position in the range `[x - t, x + t]`. However, the hen might need to eat multiple grains, so its movement pattern matters. If a hen needs to eat grains on both sides of its starting position, it has two strategies:
1. Go left first, eat grains, then go right
2. Go right first, eat grains, then go left

The optimal choice depends on the grain positions. If the hen goes left by distance `d` first, then returns and goes right, the total distance for reaching a grain at position `y` (where `y > x`) would be `d + (y - x) + d = 2d + (y - x)`. The hen should eat all grains where this total distance is within `t`.

By [sorting](/problems/sorting_summary) both arrays and using a greedy approach where each hen eats as many grains as possible from left to right within its time budget, we can efficiently check if a given time `t` is sufficient. [Binary search](/problems/binary_search_intro) then finds the minimum such `t`.

**Learn more about [Two Pointers](/problems/two_pointers_intro), [Binary Search](/problems/binary_search_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

First, we sort both the `hens` and `grains` arrays by position from left to right. This allows us to process them in order and apply a greedy strategy.

**Binary Search Template:**
We use the `first_true_index` template to find the minimum time:
- Initialize `left = 0` and `right = |hens[0] - grains[0]| + grains[-1] - grains[0]`
- Track the answer with `first_true_index = -1`
- Use `while left <= right` loop structure
- When `feasible(mid)` returns `True`, update `first_true_index = mid` and search left with `right = mid - 1`
- When `feasible(mid)` returns `False`, search right with `left = mid + 1`

**Feasible Function:**
The `feasible(time_limit)` function determines if all grains can be eaten within the given time:

1. **Initialize a pointer `j = 0`** to track the leftmost uneaten grain.

2. **For each hen at position `x`**, determine which grains it can eat:

   - **Case 1: Grain is to the left (`y ≤ x`)**
     - Calculate distance `d = x - y`
     - If `d > time_limit`, return `False`
     - Skip all grains at or left of the hen's position
     - Continue while the hen can reach grains to its right using: `min(d, grains[j] - x) + grains[j] - y ≤ time_limit`

   - **Case 2: Grain is to the right (`y > x`)**
     - Simply move `j` right while `grains[j] - x ≤ time_limit`

3. Return `True` if all grains are eaten (`j == m`).

The time complexity is `O((n + m) log(n + m) + (n + m) log R)` where `R` is the search range.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Input:**
- `hens = [4, 6, 109, 111, 213, 215]`
- `grains = [5, 110, 214]`

**Step 1: Sort both arrays** (already sorted in this case)
- `hens = [4, 6, 109, 111, 213, 215]`
- `grains = [5, 110, 214]`

**Step 2: Binary search for minimum time**

The search range is from 0 to approximately 210 (distance from first hen to first grain plus grain span).

Let's check if `t = 3` seconds is sufficient:

**Check function with t = 3:**
- Start with grain pointer `j = 0` (grain at position 5)

**Hen 1 at position 4:**
- First grain is at position 5 (to the right)
- Distance = 5 - 4 = 1 ≤ 3 ✓
- Hen can reach it, move `j` to 1
- Check next grain at 110: distance = 110 - 4 = 106 > 3 ✗
- Hen 1 eats grain at position 5

**Hen 2 at position 6:**
- Current grain is at position 110 (to the right)
- Distance = 110 - 6 = 104 > 3 ✗
- Hen 2 can't reach any remaining grains

**Hen 3 at position 109:**
- Current grain is at position 110 (to the right)
- Distance = 110 - 109 = 1 ≤ 3 ✓
- Hen can reach it, move `j` to 2
- Check next grain at 214: distance = 214 - 109 = 105 > 3 ✗
- Hen 3 eats grain at position 110

**Hen 4 at position 111:**
- Current grain is at position 214 (to the right)
- Distance = 214 - 111 = 103 > 3 ✗
- Hen 4 can't reach any remaining grains

**Hen 5 at position 213:**
- Current grain is at position 214 (to the right)
- Distance = 214 - 213 = 1 ≤ 3 ✓
- Hen can reach it, move `j` to 3
- All grains consumed! (`j = 3 = m`)

Result: `t = 3` is sufficient. Through binary search, we'd find that 3 is actually the minimum time needed.

**Why this works:**
- Hen 1 moves right 1 unit to eat grain at 5
- Hen 3 moves right 1 unit to eat grain at 110
- Hen 5 moves right 1 unit to eat grain at 214
- Maximum time any hen travels: 1 second

Each hen efficiently handles grains in its vicinity, and the greedy left-to-right assignment ensures no grain is left behind.

## Python Solution

```python
class Solution:
    def minimumTime(self, hens: List[int], grains: List[int]) -> int:
        def feasible(time_limit):
            """Check if all grains can be eaten within the given time limit."""
            grain_index = 0
            num_grains = len(grains)

            for hen_position in hens:
                if grain_index == num_grains:
                    return True

                current_grain_position = grains[grain_index]

                if current_grain_position <= hen_position:
                    distance_to_left = hen_position - current_grain_position
                    if distance_to_left > time_limit:
                        return False

                    while grain_index < num_grains and grains[grain_index] <= hen_position:
                        grain_index += 1

                    while grain_index < num_grains:
                        right_distance = grains[grain_index] - hen_position
                        total_distance = min(distance_to_left, right_distance) + grains[grain_index] - current_grain_position
                        if total_distance <= time_limit:
                            grain_index += 1
                        else:
                            break
                else:
                    while grain_index < num_grains and grains[grain_index] - hen_position <= time_limit:
                        grain_index += 1

            return grain_index == num_grains

        hens.sort()
        grains.sort()

        left, right = 0, abs(hens[0] - grains[0]) + grains[-1] - grains[0]
        first_true_index = -1

        while left <= right:
            mid = (left + right) // 2
            if feasible(mid):
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1

        return first_true_index
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n + m × log m + (m + n) × log U)`

The time complexity breaks down as follows:
- `O(n × log n)` for sorting the hens array where `n` is the number of hens
- `O(m × log m)` for sorting the grains array where `m` is the number of grains
- The binary search using `bisect_left` runs in `O(log r)` iterations where `r` represents the search range
- The value of `r` is bounded by `abs(hens[0] - grains[0]) + grains[-1] - grains[0] + 1`, which is `O(U)` where `U` is the maximum value among all hen and grain positions
- For each binary search iteration, the `check` function is called, which runs in `O(m + n)` time:
  - The outer loop iterates through all `n` hens
  - The inner while loops collectively traverse the `m` grains at most once per `check` call due to the monotonic movement of index `j`
- Therefore, the binary search contributes `O((m + n) × log U)` to the total complexity

**Space Complexity:** `O(log m + log n)`

The space complexity comes from:
- `O(log n)` for the sorting algorithm's recursion stack when sorting the hens array
- `O(log m)` for the sorting algorithm's recursion stack when sorting the grains array
- `O(1)` for the variables used in the `check` function and main logic
- The sorting operations use in-place algorithms (like Timsort in Python), which require logarithmic extra space for the recursion stack

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Using Wrong Binary Search Template

**Incorrect:**
```python
while left < right:
    mid = (left + right) // 2
    if feasible(mid):
        right = mid
    else:
        left = mid + 1
return left
```

**Correct (template-compliant):**
```python
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
return first_true_index
```

### 2. Incorrect Distance Calculation for Two-Direction Movement

When a hen goes left first then right (or vice versa), the distance formula matters:
- Go left first, then right: `distance_to_left + (right_grain - left_grain)`
- Go right first, then left: `2 * right_distance + distance_to_left`

The solution correctly uses `min(distance_to_left, right_distance) + grains[j] - first_grain` which captures the optimal strategy.

### 3. Not Handling Edge Cases

The code should handle empty arrays gracefully. If `grains` is empty, return 0. The problem constraints typically guarantee non-empty arrays, but the template's `first_true_index = -1` provides a fallback.

### 4. Forgetting to Sort Arrays

The greedy approach only works when both arrays are sorted. Without sorting, the left-to-right assignment strategy fails.

