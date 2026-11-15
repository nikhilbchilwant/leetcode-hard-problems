# 3464. Maximize the Distance Between Points on a Square

> Source: https://algo.monster/liteproblems/3464

## Problem Statement

You are given an integer `side`, which represents the edge length of a square. The square's four corners are located at `(0, 0)`, `(0, side)`, `(side, 0)`, and `(side, side)` on a Cartesian plane.

In addition, you are given a **positive** integer `k` and a 2D integer array `points`. Each entry `points[i] = [xi, yi]` is the coordinate of a point that lies on the **boundary** (perimeter) of the square.

Your task is to choose `k` points from `points` so that the **minimum** Manhattan distance between any two chosen points is as **large** as possible.

In other words, among all the ways to pick `k` points, look at the smallest pairwise Manhattan distance in each selection, and return the **maximum** value this smallest distance can reach.

The Manhattan distance between two points `(xi, yi)` and `(xj, yj)` is defined as `|xi - xj| + |yi - yj|`.

Return the **maximum** possible **minimum** Manhattan distance among the `k` selected points.

## Intuition

The key phrase in this problem is "maximize the minimum distance." Whenever we see a problem that asks us to **maximize a minimum** (or **minimize a maximum**), it is a strong hint that we can use **[binary search](/problems/binary_search_intro) on the answer**.

Why does [binary search](/problems/binary_search_intro) work here? Notice the monotonic property: if we can successfully pick `k` points whose pairwise minimum distance is at least `d`, then we can certainly also pick `k` points whose pairwise minimum distance is at least any smaller value `d' < d`. Conversely, if a distance `d` is impossible to achieve, any larger distance is also impossible. This monotonicity lets us binary search over the candidate answer `d` and, for each guess, just ask a simpler yes/no question: "Can we select `k` points such that every pair is at least `d` apart?"

The next challenge is that the points lie on the **boundary of a square**, which is awkward to reason about in 2D. However, there is a neat observation: for any two points on the perimeter, walking along the boundary from one to the other, the **Manhattan distance equals the boundary (perimeter) distance**, as long as we travel the shorter arc. This is because moving along an edge changes only one coordinate at a time, and turning a corner is exactly the point where the Manhattan path switches direction. So we can "unfold" the square's perimeter into a **1D circular axis** of total length `4 * side`.

To do this unfolding, we map every boundary point to a single value based on which edge it sits on, walking around the perimeter in order:

- On the left edge (`x = 0`), the value is `y`;
- On the top edge (`y = side`), the value is `side + x`;
- On the right edge (`x = side`), the value is `3 * side - y`;
- On the bottom edge (otherwise), the value is `4 * side - x`.

After this mapping and [sorting](/problems/sorting_summary), the problem becomes: place `k` markers on a **circle of circumference `4 * side`** so that adjacent markers are at least `d` apart. Because it is a circle (the last point must also be at least `d` away from the first when wrapping around), we try each point as a **starting point**, then **greedily** jump forward to the nearest point that is at least `d` away, repeating `k - 1` times. Greedy works because, to fit as many points as possible, always taking the earliest valid next point leaves the most room for the remaining picks.

We use a [binary search](/problems/binary_search_intro) (`bisect`) inside the greedy step to quickly find the next reachable point, and we cap the reachable end at `start + 4 * side - d` to guarantee the wrap-around gap back to the start point also respects the distance `d`. If any starting point lets us place all `k` markers, then `d` is feasible, and we push the binary search higher; otherwise, we lower it.

**Learn more about [Math](/problems/math-basics), [Binary Search](/problems/binary_search_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

We use **[Binary Search](/problems/binary_search_intro) on the answer + Coordinate Mapping + Greedy** to solve this problem.

### Step 1: Map 2D Boundary Points to a 1D Axis

To avoid dealing with awkward 2D geometry, we "unfold" the square's perimeter into a single 1D axis of length `4 * side`. Walking clockwise (or counter-clockwise) around the boundary, we convert each point `(x, y)` into one number according to which edge it lies on:

- If `x == 0` (left edge), the mapped value is `y`;
- If `y == side` (top edge), the mapped value is `side + x`;
- If `x == side` (right edge), the mapped value is `3 * side - y`;
- Otherwise (bottom edge), the mapped value is `4 * side - x`.

After mapping every point, we store the results in `nums` and **sort** it. Now the points are arranged on a circular axis with total circumference `4 * side`, where the Manhattan distance between two points equals their distance along this axis (taking the shorter arc).

### Step 2: [Binary Search](/problems/binary_search_intro) on the Answer

Since we want to **maximize the minimum distance**, we [binary search](/problems/binary_search_intro) over the candidate distance `lo` in the range `[1, side]`:

```python
l, r = 1, side
while l < r:
    mid = (l + r + 1) >> 1
    if check(mid):
        l = mid
    else:
        r = mid - 1
return l
```

Notice that `mid = (l + r + 1) >> 1` rounds **up**. This is the correct form when the update is `l = mid`, and it prevents an infinite loop when `l` and `r` are adjacent. If `check(mid)` returns `True`, the distance is achievable, so we try larger values by moving `l` up; otherwise, we shrink to `r = mid - 1`.

### Step 3: The `check` Function (Greedy on a Circle)

For a candidate distance `lo`, the `check` function decides whether we can pick `k` points so that every adjacent pair is at least `lo` apart.

Because the points form a **circle**, the first and last selected points must also satisfy the distance constraint when wrapping around. To handle this, we try **every point as the starting point** `start`:

- We set `end = start + side * 4 - lo`. This is the furthest position the last selected point may reach, guaranteeing the wrap-around gap from the last point back to `start` is at least `lo`.
- Starting from `cur = start`, we perform `k - 1` **greedy jumps**. In each jump, we use `bisect_left(nums, cur + lo)` to find the **nearest** point that is at least `lo` away from `cur`:

```python
j = bisect_left(nums, cur + lo)
if j == len(nums) or nums[j] > end:
    ok = False
    break
cur = nums[j]
```

  - If no such point exists (`j == len(nums)`), or the found point exceeds `end` (`nums[j] > end`), this starting point fails.
  - Otherwise, we advance `cur` to that point and continue.

- If we successfully complete all `k - 1` jumps from some starting point, then `lo` is feasible and we return `True`.

The greedy choice—always jumping to the **earliest valid next point**—is optimal because leaving as much room as possible for the remaining picks maximizes our chance of fitting all `k` points.

### Complexity Analysis

Let `n` be the number of points.

- **Time Complexity**: `O(n * k * log n * log(side))`. The [binary search](/problems/binary_search_intro) on the answer contributes `log(side)`. Each `check` call tries `n` starting points, each performing `k - 1` jumps, and every jump uses a `bisect` costing `log n`.
- **Space Complexity**: `O(n)` for the `nums` array.

## Example Walkthrough

Let's trace through a small concrete example to see how **Binary Search + Coordinate Mapping + Greedy** works together.

**Input:**
- `side = 2`
- `k = 4`
- `points = [[0, 2], [2, 0], [2, 2], [0, 0]]`

These are exactly the four corners of the square. Since we must pick `k = 4` points and there are only 4 available, we have no choice but to take all of them. Let's verify what the algorithm computes.

---

#### Step 1: Map 2D Boundary Points to a 1D Axis

The perimeter circumference is `4 * side = 8`. We unfold each point using the edge rules:

| Point `(x, y)` | Which edge? | Rule applied | Mapped value |
|----------------|-------------|--------------|--------------|
| `(0, 0)` | left (`x == 0`) | `y` | `0` |
| `(0, 2)` | left (`x == 0`) | `y` | `2` |
| `(2, 2)` | top (`y == side`) wins after left check fails... actually `x == side` | right (`x == side`) | `3*2 - 2 = 4` |
| `(2, 0)` | right (`x == side`) | `3*2 - 0 = 6` | `6` |

Sorted: `nums = [0, 2, 4, 6]`.

Notice how the four corners land evenly spaced on a circle of circumference `8`: positions `0, 2, 4, 6`. The gap between consecutive points (including the wrap from `6` back to `0`, which is `8 - 6 = 2`) is exactly `2`.

---

#### Step 2: Binary Search on the Answer

We search for the largest feasible minimum distance in range `[1, side] = [1, 2]`.

- `l = 1`, `r = 2`
- `mid = (1 + 2 + 1) >> 1 = 2`
- Call `check(2)`.

---

#### Step 3: The `check(2)` Function (Greedy on a Circle)

We try each point as the starting point. Let's start with `start = nums[0] = 0`.

- `end = start + side*4 - lo = 0 + 8 - 2 = 6`. The last selected point may reach at most position `6`.
- We need `k - 1 = 3` greedy jumps. Begin with `cur = 0`.

**Jump 1:** Find nearest point `>= cur + lo = 0 + 2 = 2`.
- `bisect_left(nums, 2)` → index `1`, `nums[1] = 2`.
- `2 <= end (6)` ✓. Set `cur = 2`.

**Jump 2:** Find nearest point `>= 2 + 2 = 4`.
- `bisect_left(nums, 4)` → index `2`, `nums[2] = 4`.
- `4 <= 6` ✓. Set `cur = 4`.

**Jump 3:** Find nearest point `>= 4 + 2 = 6`.
- `bisect_left(nums, 6)` → index `3`, `nums[3] = 6`.
- `6 <= 6` ✓. Set `cur = 6`.

All `3` jumps succeeded → we placed all `4` points `{0, 2, 4, 6}`, each adjacent pair at least `2` apart, and the wrap-around gap (`6` back to `0`) is `8 - 6 = 2 >= 2`. So `check(2)` returns **True**.

---

#### Back to Binary Search

Since `check(2)` is `True`:
- `l = mid = 2`.
- Now `l == r == 2`, loop ends.

**Return `l = 2`.**

---

#### Why This Is Correct

The four chosen points map to `0, 2, 4, 6` on the circle. Their pairwise Manhattan distances correspond to arc distances; the **minimum** adjacent gap is `2`, and there's no way to do better because we are forced to take all four corners. The algorithm correctly reports `2`.

This example highlights the three core ideas: the **mapping** flattens awkward 2D boundary geometry into clean 1D positions, the **greedy jump** packs points by always taking the earliest valid next position, and the **binary search** zeroes in on the largest achievable minimum distance.

## Python Solution

```python
from bisect import bisect_left
from typing import List


class Solution:
    def maxDistance(self, side: int, points: List[List[int]], k: int) -> int:
        # Map each perimeter point to a 1D coordinate, measuring the clockwise
        # distance from the top-left corner (0, side) is implied by the edges:
        #   - left edge   (x == 0):       coordinate = y
        #   - top edge    (y == side):    coordinate = side + x
        #   - right edge  (x == side):    coordinate = side * 3 - y
        #   - bottom edge (otherwise):    coordinate = side * 4 - x
        perimeter_coords: List[int] = []
        for x, y in points:
            if x == 0:
                perimeter_coords.append(y)
            elif y == side:
                perimeter_coords.append(side + x)
            elif x == side:
                perimeter_coords.append(side * 3 - y)
            else:
                perimeter_coords.append(side * 4 - x)

        # Sort coordinates so we can binary search for the next valid point.
        perimeter_coords.sort()
        total_length = side * 4

        def check(min_gap: int) -> bool:
            # Determine whether we can choose k points so that every pair of
            # consecutive chosen points is at least `min_gap` apart along the
            # perimeter.
            for start in perimeter_coords:
                # The farthest coordinate still allowed for the final point so
                # that the wrap-around gap back to `start` is also >= min_gap.
                end = start + total_length - min_gap
                current = start
                feasible = True

                # Greedily pick the remaining k - 1 points.
                for _ in range(k - 1):
                    # Find the first point at least `min_gap` away from current.
                    next_index = bisect_left(perimeter_coords, current + min_gap)
                    # No valid point or it exceeds the allowed end boundary.
                    if next_index == len(perimeter_coords) or perimeter_coords[next_index] > end:
                        feasible = False
                        break
                    current = perimeter_coords[next_index]

                if feasible:
                    return True
            return False

        # Binary search on the maximum achievable minimum gap.
        left, right = 1, side
        while left < right:
            mid = (left + right + 1) >> 1
            if check(mid):
                left = mid          # mid is achievable, try larger.
            else:
                right = mid - 1     # mid too large, shrink.
        return left
```

## Time and Space Complexity

- **Time Complexity**: `O(n × log(side) × n × log n)`, where `n` is the length of the `points` array.
  - First, mapping each point to its perimeter coordinate takes `O(n)`, and sorting `nums` takes `O(n log n)`.
  - The outer binary search runs over the range `[1, side]`, requiring `O(log(side))` iterations.
  - Each iteration calls `check(mid)`. Inside `check`, the outer loop iterates over all `n` starting points, and for each start, the inner loop runs up to `k - 1` times. Each inner step performs a `bisect_left`, costing `O(log n)`.
  - In the worst case, `k` can be on the order of `n`, so a single `check` call costs `O(n × k × log n) = O(n² × log n)`.
  - Combining the binary search with the cost of `check`, the total time complexity is `O(n × log(side) × n × log n)`.

- **Space Complexity**: `O(n)`. The auxiliary array `nums` stores one perimeter coordinate per point, requiring `O(n)` space. The sorting may use additional `O(n)` space, and no other data structures grow with the input size.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrectly Handling the Wrap-Around with `bisect_left`

The most subtle and dangerous bug in this solution lies in the **circular nature** of the perimeter. When we map points onto a 1D axis of length `4 * side`, the points form a **circle**, not a straight line. The greedy `bisect_left(perimeter_coords, current + min_gap)` only searches **forward** in the sorted array—it never "wraps around" past the largest coordinate back to the smallest.

**Why this matters:** Consider a starting point near the end of the sorted array. After a few jumps, `current + min_gap` may exceed the maximum coordinate, so `bisect_left` returns `len(perimeter_coords)` and the search fails—**even if** a valid point exists at the *beginning* of the array (which, on the circle, is genuinely `min_gap` away when wrapping). However, because the code fixes `end = start + total_length - min_gap` and only moves forward, it never explores the wrapped region.

**Is this actually a bug here?** No—and understanding *why* is crucial. By trying **every** point as `start` and constraining the final point to `end = start + total_length - min_gap`, the algorithm effectively "linearizes" the circle from each possible starting position. Since one of the `k` chosen points must be the "first" in clockwise order, iterating all starts guarantees we capture the correct circular arrangement. **The pitfall is "fixing" this by adding manual wrap-around logic (e.g., appending `coord + total_length` duplicates), which double-counts and breaks correctness.**

**Solution:** Trust the "try every start + bounded `end`" pattern. Do **not** add wrapped duplicates to `perimeter_coords`. The boundary check `perimeter_coords[next_index] > end` is what enforces the wrap-around gap correctly.

```python
end = start + total_length - min_gap   # enforces wrap gap implicitly
# ...
if next_index == len(perimeter_coords) or perimeter_coords[next_index] > end:
    feasible = False   # correct: no need to wrap manually
    break
```

### Pitfall 2: Wrong Binary Search Midpoint Rounding

Because the update is `left = mid` (not `left = mid + 1`), you **must** round the midpoint **up**:

```python
mid = (left + right + 1) >> 1   # correct
```

If you mistakenly write `mid = (left + right) >> 1` (rounding down) while keeping `left = mid`, you get an **infinite loop** when `left` and `right` become adjacent (`right = left + 1`): `mid` evaluates to `left`, `check` passes, `left = mid` leaves `left` unchanged, and the loop never terminates.

**Solution:** Match the rounding direction to the assignment:
- `left = mid` → round **up**: `(left + right + 1) >> 1`
- `right = mid` → round **down**: `(left + right) >> 1`

### Pitfall 3: Misjudging the Manhattan-to-Perimeter Distance Equivalence

The mapping assumes the **arc distance** along the perimeter equals the **Manhattan distance** between two boundary points. This is **only true on the same edge or adjacent edges within a certain range**. For two points on opposite edges, the straight Manhattan distance can be **smaller** than the perimeter arc.

**Example:** Points `(0, 0)` and `(side, 0)` (bottom corners). Manhattan distance is `side`. But mapped coordinates are `0` and `4*side - side = 3*side`, giving an arc distance of `3*side` (or `side` the short way). The shorter arc here coincidentally matches, but for interior-edge opposite points it may not.

**Why the solution still works:** This problem's structure (boundary points + maximizing the *minimum* gap) means that the binding constraint is almost always between **nearby** points along the perimeter, where arc distance equals Manhattan distance. The greedy spacing naturally avoids clustering, so the dangerous "opposite edge" cases rarely become the minimum.

**Solution:** Be aware that this equivalence is a **problem-specific simplification**. If you adapt this code to a variant where opposite-edge distances dominate, you must compute true Manhattan distances rather than relying solely on the 1D arc mapping.

### Pitfall 4: Forgetting That Only `k - 1` Jumps Are Needed

A common off-by-one error is looping `k` times instead of `k - 1`. The `start` point counts as the **first** selected point, so only `k - 1` additional points must be placed.

```python
for _ in range(k - 1):   # correct: start is already point #1
    ...
```

Using `range(k)` would demand one extra point and incorrectly reject feasible candidates.

