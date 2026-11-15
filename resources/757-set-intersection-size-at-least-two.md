# 757. Set Intersection Size At Least Two

> Source: https://algo.monster/liteproblems/757

## Problem Statement

You are given a collection of intervals, where each interval is represented as `[start, end]` and includes all integers from `start` to `end` (both inclusive).

Your task is to find the smallest set of integers such that every interval contains at least 2 integers from this set.

For example, given intervals `[[1,3], [3,7], [8,9]]`:
- The interval `[1,3]` contains integers 1, 2, and 3
- The interval `[3,7]` contains integers 3, 4, 5, 6, and 7  
- The interval `[8,9]` contains integers 8 and 9

A valid containing set could be `[2,3,4,8,9]` because:
- Interval `[1,3]` contains 2 and 3 from the set (2 integers ✓)
- Interval `[3,7]` contains 3 and 4 from the set (2 integers ✓)
- Interval `[8,9]` contains 8 and 9 from the set (2 integers ✓)

The solution uses a [greedy](/problems/greedy_intro) approach by [sorting](/problems/sorting_summary) intervals by their end points (and by start points in descending order when end points are equal). It then tracks the last two selected points (`s` and `e`) and decides how many new points to add based on the overlap with the current interval:
- If the interval's start is less than or equal to `s`, it already contains both tracked points
- If the interval's start is greater than `e`, no overlap exists, so we add 2 new points from the interval's end
- Otherwise, there's partial overlap with one point, so we add 1 new point

## Intuition

The key insight is to think about this problem greedily - we want to place our selected integers as strategically as possible to cover multiple intervals simultaneously.

Consider what happens when we process intervals in a specific order. If we sort intervals by their ending points, we can make locally optimal choices that lead to a globally optimal solution. Why? Because once we've decided which points to use for an interval, those points might also satisfy future intervals that overlap with it.

The critical observation is that for any interval, the best strategy is to place our selected points as far to the right as possible (near the interval's end). This maximizes the chance that these points will also satisfy upcoming intervals. If we need to pick 2 points from interval `[a, b]`, choosing `b-1` and `b` gives us the best chance of reusing these points for other intervals.

Now, as we process each new interval, we need to check how many of our previously selected points it already contains:
- If it contains 2 or more points, we don't need to add anything
- If it contains exactly 1 point, we need to add 1 more point (choosing the rightmost position in the interval)
- If it contains 0 points, we need to add 2 new points (choosing the two rightmost positions)

By maintaining just the last two selected points (`s` and `e` where `s < e`), we can efficiently determine the overlap. Since intervals are sorted by endpoints, and we always choose points from the right side of intervals, these two most recent points are the only ones that could possibly overlap with the current interval.

The secondary [sorting](/problems/sorting_summary) by start point in descending order (when endpoints are equal) ensures we process narrower intervals first. A wider interval with the same endpoint then already contains both tracked points. In the other order, the update `s, e = e, b` can set `s` and `e` to the same point `b`, counting one integer twice.

**Learn more about [Greedy](/problems/greedy_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows a [greedy](/problems/greedy_intro) algorithm with careful interval processing:

**1. Sort the intervals:**
```python
intervals.sort(key=lambda x: (x[1], -x[0]))
```
We sort primarily by endpoint in ascending order, and secondarily by start point in descending order (using `-x[0]`). This ensures we process intervals ending earlier first, and among those with the same endpoint, we process narrower intervals (larger start) first.

**2. Initialize tracking variables:**
```python
s = e = -1
ans = 0
```
- `s` and `e` track the last two selected points (with `s < e`)
- Initialize them to `-1` since all intervals have non-negative values
- `ans` counts the total number of points in our containing set

**3. Process each interval greedily:**
```python
for a, b in intervals:
```
For each interval `[a, b]`, we check how many of our previously selected points it contains.

**Case 1: Interval already contains both points**
```python
if a <= s:
    continue
```
If the interval's start `a` is less than or equal to `s` (the smaller of our two tracked points), then the interval contains both `s` and `e`. No new points needed.

**Case 2: Interval contains no tracked points**
```python
if a > e:
    ans += 2
    s, e = b - 1, b
```
If the interval's start `a` is greater than `e` (the larger tracked point), the interval contains neither point. We add 2 new points: `b-1` and `b` (the rightmost positions in the interval).

**Case 3: Interval contains exactly one tracked point**
```python
else:
    ans += 1
    s, e = e, b
```
If `s < a <= e`, the interval contains only `e`. We add 1 new point at position `b`, and update our tracked points to `e` (the old one that's still in the interval) and `b` (the new one).

**4. Return the result:**
```python
return ans
```
The total count represents the minimum size of the containing set.

This [greedy](/problems/greedy_intro) approach works because by always choosing points from the right side of intervals and processing intervals in sorted order, we maximize the reuse of selected points across multiple intervals.

## Example Walkthrough

Let's walk through the algorithm with intervals `[[1,3], [1,4], [2,5], [3,5]]`.

**Step 1: Sort the intervals**
- Original: `[[1,3], [1,4], [2,5], [3,5]]`
- After sorting by (endpoint ascending, start descending): `[[1,3], [1,4], [3,5], [2,5]]` (`[3,5]` comes before `[2,5]` because its start is larger)

**Step 2: Initialize variables**
- `s = -1, e = -1` (our tracked points)
- `ans = 0` (count of points in our set)

**Step 3: Process each interval**

**Interval [1,3]:**
- Check: `a = 1 > e = -1`? Yes, no overlap with tracked points
- Add 2 points from the end: choose points 2 and 3
- Update: `ans = 2`, `s = 2`, `e = 3`
- Our set so far: `{2, 3}`

**Interval [1,4]:**
- Check: `a = 1 <= s = 2`? Yes, interval contains both tracked points (2 and 3)
- No new points needed
- Keep: `ans = 2`, `s = 2`, `e = 3`
- Our set remains: `{2, 3}`

**Interval [3,5]:**
- Check: `a = 3 <= s = 2`? No
- Check: `a = 3 > e = 3`? No
- So `s < a <= e`, meaning interval contains only point 3
- Add 1 point from the end: choose point 5
- Update: `ans = 3`, `s = 3` (the old point still in interval), `e = 5` (the new point)
- Our set now: `{2, 3, 5}`

**Interval [2,5]:**
- Check: `a = 2 <= s = 3`? Yes, interval contains both tracked points (3 and 5)
- No new points needed
- Keep: `ans = 3`, `s = 3`, `e = 5`
- Our set remains: `{2, 3, 5}`

**Final verification:**
- `[1,3]` contains {2, 3} ✓
- `[1,4]` contains {2, 3} ✓
- `[2,5]` contains {2, 3, 5} - at least 2 ✓
- `[3,5]` contains {3, 5} ✓

The algorithm returns `3`, and our minimum containing set has 3 elements.

## Python Solution

```python
class Solution:
    def intersectionSizeTwo(self, intervals: List[List[int]]) -> int:
        # Sort intervals by end point (ascending), then by start point (descending)
        # This ensures we process intervals with earlier endpoints first
        # For same endpoints, we prefer intervals with larger start points
        intervals.sort(key=lambda interval: (interval[1], -interval[0]))
      
        # Initialize two points that will be in our result set
        # second_last: the second-to-last point we've chosen
        # last: the last (most recent) point we've chosen
        second_last = -1
        last = -1
        result_size = 0
      
        for start, end in intervals:
            # Case 1: Current interval already contains both tracked points
            # No new points needed
            if start <= second_last:
                continue
          
            # Case 2: Current interval doesn't contain any tracked points
            # We need to add 2 new points (choose the two largest in the interval)
            if start > last:
                result_size += 2
                second_last = end - 1
                last = end
          
            # Case 3: Current interval contains only the last point
            # We need to add 1 new point (choose the largest in the interval)
            else:
                result_size += 1
                second_last = last
                last = end
      
        return result_size
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)` where `n` is the number of intervals.
- The sorting operation takes `O(n log n)` time
- After sorting, we iterate through all intervals once, which takes `O(n)` time
- Each operation inside the loop (comparisons and assignments) takes `O(1)` time
- Therefore, the overall time complexity is dominated by the sorting step: `O(n log n)`

**Space Complexity:** `O(1)` or `O(n)` depending on the sorting algorithm implementation.
- The algorithm uses only a constant amount of extra space for variables `s`, `e`, and `ans`
- However, the sorting operation may require additional space:
  - If Python's `sort()` uses Timsort (which it does), it requires `O(n)` space in the worst case
  - If we consider only the extra space used by our algorithm excluding the sorting, it's `O(1)`
- Therefore, the space complexity is `O(n)` when considering the sorting implementation, or `O(1)` for the algorithm's explicit space usage

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Incorrect Sorting Logic
**Pitfall:** Using only endpoint sorting without considering the start point can lead to suboptimal solutions.

```python
# WRONG: Only sorting by endpoint
intervals.sort(key=lambda x: x[1])
```

**Why it fails:** When two intervals have the same endpoint, the narrower interval (larger start) must come first. For `[[0,1], [1,5], [3,5], [5,7]]`, sorting only by endpoint keeps `[1,5]` before `[3,5]`. After `[0,1]` picks `{0, 1}`, `[1,5]` contains only `1` and adds `5`, so the tracked points become `1, 5`. Then `[3,5]` contains only `5` and the update `s, e = e, b` sets both tracked points to `5`, counting the same integer twice. The result is 4, but the correct answer is 5. With `[3,5]` processed first, it picks `{4, 5}`, `[1,5]` then contains both, and `[5,7]` adds `7`, for a total of 5.

**Solution:** Sort by endpoint ascending, then by start point descending:
```python
intervals.sort(key=lambda x: (x[1], -x[0]))
```

### 2. Choosing Points from the Wrong Side
**Pitfall:** Selecting points from the left side or middle of intervals instead of the rightmost positions.

```python
# WRONG: Choosing from the left side
if start > last:
    result_size += 2
    second_last = start      # Wrong!
    last = start + 1         # Wrong!
```

**Why it fails:** Points from the right side of an interval have a better chance of being contained in subsequent intervals (since we're processing in order of endpoints). Choosing `[start, start+1]` instead of `[end-1, end]` reduces the opportunity for point reuse.

**Solution:** Always choose the rightmost points:
```python
if start > last:
    result_size += 2
    second_last = end - 1
    last = end
```

### 3. Incorrect Overlap Detection
**Pitfall:** Using wrong comparison operators when checking interval overlap with tracked points.

```python
# WRONG: Using < instead of <=
if start < second_last:  # Should be <=
    continue
```

**Why it fails:** If `start == second_last`, the interval still contains both `second_last` and `last` (since `second_last < last`). Using strict inequality would incorrectly treat this as needing additional points.

**Solution:** Use the correct comparison operators:
```python
if start <= second_last:    # Interval contains both points
    continue
if start > last:            # Interval contains no points
    # Add 2 points
else:                       # Interval contains exactly one point (last)
    # Add 1 point
```

### 4. Not Maintaining Point Order Invariant
**Pitfall:** Forgetting to maintain `second_last < last` after updates.

```python
# WRONG: Could violate the invariant
if start > last:
    result_size += 2
    second_last = end       # Wrong order!
    last = end - 1          # Wrong order!
```

**Why it fails:** The algorithm relies on `second_last` being smaller than `last` for correct overlap detection. Reversing this order breaks the logic.

**Solution:** Always ensure `second_last < last`:
```python
if start > last:
    result_size += 2
    second_last = end - 1   # Smaller value
    last = end              # Larger value
```

### 5. Edge Case with Single-Point Intervals
**Pitfall:** Assuming an interval can be a single point such as `[5,5]`.

**Why it does not apply here:** The problem guarantees `start < end`, so every interval holds at least two integers and `end - 1` is always inside the interval. A single-point interval could never contain two distinct integers, so no valid set would exist. The solution therefore does not need special handling for this case.

