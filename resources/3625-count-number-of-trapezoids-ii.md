# 3625. Count Number of Trapezoids II

> Source: https://algo.monster/liteproblems/3625

## Problem Statement

You are given a 2D integer array `points` where `points[i] = [xᵢ, yᵢ]` represents the coordinates of the `iᵗʰ` point on the Cartesian plane.

Return the number of unique **trapezoids** that can be formed by choosing any four distinct points from `points`.

A **trapezoid** is a convex quadrilateral with **at least one pair** of parallel sides. Two lines are parallel if and only if they have the same slope.

In other words, your task is to count how many groups of four distinct points can be arranged into a convex quadrilateral such that at least one pair of its opposite sides are parallel. Since two sides are parallel exactly when they share the same slope, you need to look for pairs of point-pairs that have equal slopes but lie on different lines (so that they form two distinct parallel sides rather than the same line). Care must also be taken with parallelograms, which have two pairs of parallel sides and could therefore be counted more than once.

## Intuition

The key observation is that a trapezoid needs **at least one pair of parallel sides**. Two segments are parallel when they have the same slope. So instead of thinking about four points at once, let's think about **pairs of points**, because every side of a quadrilateral is just a segment connecting two points.

If we enumerate every pair of points and compute the slope of the line through them, then any two pairs that share the **same slope** could potentially serve as the two parallel sides of a trapezoid. So a natural idea is: group all point-pairs by their slope, and for each slope, count how many ways we can pick two of these parallel segments.

But there is a subtle trap. Two segments with the same slope might actually lie on the **same line**. In that case the four points are collinear and cannot form a quadrilateral at all. To avoid this, we don't just group by slope `k`; we also distinguish by the **intercept** `b` of the line. Segments with the same slope but **different intercepts** lie on different parallel lines, which is exactly what we want. So within each slope group, we further split by intercept, and only count pairs of segments coming from **different intercepts**.

This is where the counting trick comes in. For a fixed slope `k`, suppose the intercepts have occurrence counts `t₁, t₂, t₃, ...`. The number of valid cross-intercept pairs can be computed by sweeping through the counts: keep a running sum `s` of everything seen so far, and for each new count `t`, add `s * t` to the answer. This efficiently counts all pairs that come from two different intercepts without double-counting.

Now comes the second subtlety: **parallelograms**. A parallelogram has **two** pairs of parallel sides. When we count by "pairs of parallel segments," a parallelogram gets counted **twice**, once for each pair of parallel sides. Since the problem asks for unique trapezoids, we must subtract these extra counts once.

How do we detect a parallelogram? A quadrilateral is a parallelogram exactly when its **diagonals share the same midpoint**. So we again enumerate all point-pairs, but this time we treat each pair as a potential **diagonal**. We group pairs by their **midpoint**, and within the same midpoint, two pairs form a parallelogram. To make sure we don't accidentally count degenerate cases (where the points are collinear), we additionally distinguish by slope within each midpoint group. Using the same sweep-and-accumulate trick, we count these parallelograms and **subtract** them from the answer.

Putting it together: encode the midpoint as a single number `p = (x₁ + x₂ + 2000) * 4000 + (y₁ + y₂ + 2000)`, where the `+2000` offsets keep the values non-negative so they can be safely used as hash keys. The final answer is the total parallel-pair count minus the parallelogram count, which gives us exactly the number of unique trapezoids.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

We use the **Hash Table + Enumeration** pattern. The core idea is to enumerate all pairs of points and record two kinds of information using two hash tables.

**Data structures.** We set up two nested hash tables:

- `cnt1`: maps a slope `k` to another hash table that records, for each intercept `b`, how many segments with slope `k` lie on the line with that intercept. So the structure is `k -> (b -> count)`.
- `cnt2`: maps a midpoint code `p` to another hash table that records, for each slope `k`, how many point-pairs share that midpoint with that slope. So the structure is `p -> (k -> count)`.

Both are built with `defaultdict(lambda: defaultdict(int))` so we can increment counts without checking for existence.

**Enumerating point pairs.** Let `n = len(points)`. We loop over all unordered pairs `(i, j)` with `j < i`. For each pair `(x1, y1)` and `(x2, y2)`, we compute `dx = x2 - x1` and `dy = y2 - y1`:

- If `dx == 0`, the two points lie on the same vertical line. We use a sentinel slope `k = 1e9` and set the intercept `b = x1` (the line is `x = x1`).
- Otherwise, the slope is `k = dy / dx`. The intercept is computed as `b = (y1 * dx - x1 * dy) / dx`. This comes from rearranging the line equation so that segments on the same line share the same `b`.

We then update `cnt1[k][b] += 1`.

For the parallelogram detection, we encode the midpoint of the pair as a single integer:

```
p = (x1 + x2 + 2000) * 4000 + (y1 + y2 + 2000)
```

The `+2000` offsets keep both coordinate sums non-negative, and multiplying the `x`-part by `4000` guarantees that distinct midpoints map to distinct codes (since each coordinate sum fits within a `4000`-wide range). We then update `cnt2[p][k] += 1`.

**Counting parallel-side pairs.** We iterate over each slope group in `cnt1`. Within a single slope `k`, we want to count pairs of segments that have **different intercepts**. We do this with a running-sum sweep:

```python
for e in cnt1.values():
    s = 0
    for t in e.values():
        ans += s * t
        s += t
```

Here `s` accumulates the total count of segments seen so far (from earlier intercepts), and `t` is the count for the current intercept. Multiplying `s * t` gives the number of cross-intercept pairs introduced by this intercept, which is exactly the number of valid trapezoid-forming parallel pairs for that slope. This avoids pairing a segment with another segment on the **same** line.

**Subtracting parallelograms.** Each parallelogram has two pairs of parallel sides, so it was counted twice above. We subtract one count per parallelogram using `cnt2`. A parallelogram is identified by two diagonals sharing the same midpoint and same slope grouping. We apply the identical sweep, but subtract this time:

```python
for e in cnt2.values():
    s = 0
    for t in e.values():
        ans -= s * t
        s += t
```

For a fixed midpoint `p`, pairing point-pairs with the same midpoint forms parallelograms, and `s * t` counts these combinations. Distinguishing by slope `k` inside each midpoint group keeps degenerate collinear cases consistent with how they were counted in `cnt1`.

**Final answer.** After adding all parallel-pair counts and subtracting the duplicate parallelogram counts, `ans` holds the number of unique trapezoids, which we return.

**Complexity.** Let `n` be the number of points. We enumerate `O(n²)` pairs, and each insertion into the hash tables is `O(1)` on average. The final sweeps over the hash table entries also total `O(n²)`. Thus the time complexity is `O(n²)`, and the space complexity is `O(n²)` to store all the point-pair entries.

## Example Walkthrough

Let's use a small set of **5 points** that contains both a "pure" trapezoid and a parallelogram, so we can see every part of the algorithm working — including the parallelogram subtraction.

```
points = [
  [0, 0],   # P0
  [4, 0],   # P1
  [1, 2],   # P2
  [3, 2],   # P3
  [5, 2],   # P4
]
```

Here is the picture (y grows upward):

```
y=2:      P2(1,2)   P3(3,2)   P4(5,2)
y=0:  P0(0,0)             P1(4,0)
```

Our goal is to count unique convex quadrilaterals with **at least one pair of parallel sides**.

---

#### Step 1 — Enumerate all point-pairs, fill `cnt1` (slope → intercept → count)

For each unordered pair we compute `dx`, `dy`, then slope `k = dy/dx` and intercept `b = (y1·dx − x1·dy)/dx`. (Vertical pairs use sentinel `k = 1e9`, `b = x`.)

| Pair | (dx, dy) | slope `k` | intercept `b` |
|------|----------|-----------|---------------|
| P0–P1 | (4, 0) | 0 | 0 |
| P0–P2 | (1, 2) | 2 | 0 |
| P0–P3 | (3, 2) | 2/3 | 0 |
| P0–P4 | (5, 2) | 2/5 | 0 |
| P1–P2 | (−3, 2) | −2/3 | 8/3 |
| P1–P3 | (−1, 2) | −2 | 8 |
| P1–P4 | (1, 2) | 2 | −8 |
| P2–P3 | (2, 0) | 0 | 2 |
| P2–P4 | (4, 0) | 0 | 2 |
| P3–P4 | (2, 0) | 0 | 2 |

Now group into `cnt1` as `k -> (b -> count)`:

```
slope  0    : { b=0 : 1 (P0P1),  b=2 : 3 (P2P3, P2P4, P3P4) }
slope  2    : { b=0 : 1 (P0P2),  b=-8 : 1 (P1P4) }
slope  2/3  : { b=0 : 1 (P0P3) }
slope  2/5  : { b=0 : 1 (P0P4) }
slope -2/3  : { b=8/3 : 1 (P1P2) }
slope -2    : { b=8 : 1 (P1P3) }
```

---

#### Step 2 — Count parallel-side pairs (the running-sum sweep)

For each slope, sweep its intercept counts keeping running sum `s`, adding `s·t`:

- **slope 0**: intercepts have counts `[1, 3]`
  - `s=0`, t=1 → add `0`, then `s=1`
  - t=3 → add `1·3 = 3`, then `s=4`
  - contributes **3**
- **slope 2**: counts `[1, 1]`
  - t=1 → add 0, `s=1`
  - t=1 → add `1·1 = 1`
  - contributes **1**
- all other slopes have a single intercept group → contribute **0**

Running total `ans = 3 + 1 = 4`.

These 4 candidate parallel pairs are:
1. **P0P1 ∥ P2P3** (slope 0)
2. **P0P1 ∥ P2P4** (slope 0)
3. **P0P1 ∥ P3P4** (slope 0)
4. **P0P2 ∥ P1P4** (slope 2)

Each represents a quadrilateral with one pair of parallel sides.

---

#### Step 3 — Detect parallelograms via `cnt2` (midpoint → slope → count)

A parallelogram's diagonals share the same midpoint. We treat each pair as a *diagonal*, encode its midpoint as
`p = (x1+x2+2000)·4000 + (y1+y2+2000)`, and group by `(midpoint, slope)`.

Look at candidate **#4**: points `P0(0,0), P2(1,2), P1(4,0), P4(5,2)`.

```
P2(1,2)        P4(5,2)
   P0(0,0)        P1(4,0)
```

This is a **parallelogram** (P0P2 ∥ P1P4 *and* P0P1 ∥ P2P4). So it was counted **twice** in Step 2 — once as candidate #2 (P0P1 ∥ P2P4) and once as candidate #4 (P0P2 ∥ P1P4).

Its diagonals are **P0–P4** and **P1–P2**:
- P0–P4 midpoint sum = (0+5, 0+2) = (5, 2), slope = 2/5
- P1–P2 midpoint sum = (4+1, 0+2) = (5, 2), slope = −2/3

Both share the same encoded midpoint `p`, so in `cnt2`:

```
midpoint(5,2) : { slope 2/5 : 1 (P0P4),  slope -2/3 : 1 (P1P2) }
```

(No other pairs share a midpoint here, so every other `cnt2` group has a single entry.)

Sweep `cnt2` the same way:
- midpoint(5,2): counts `[1, 1]` → add `1·1 = 1`
- everything else contributes 0

So we **subtract 1**: `ans = 4 − 1 = 3`.

---

#### Step 4 — Final answer

```
ans = 4 (parallel pairs) − 1 (parallelogram double-count) = 3
```

The **3 unique trapezoids** are:
1. P0, P1, P2, P3
2. P0, P1, P2, P4 ← this is the parallelogram (counted once, correctly)
3. P0, P1, P3, P4

The double-counting of the parallelogram (candidate #2 vs. #4) has been collapsed into a single valid quadrilateral, giving the correct count of **3**.

## Python Solution

```python
class Solution:
    def countTrapezoids(self, points: List[List[int]]) -> int:
        n = len(points)

        # slope_to_intercept_count: slope -> (intercept -> number of segments)
        # Groups segments by slope; within each slope, groups by intercept.
        # Two segments sharing the same slope but different intercepts form a
        # pair of parallel sides (a candidate trapezoid).
        slope_to_intercept_count: Dict[float, Dict[float, int]] = defaultdict(lambda: defaultdict(int))

        # midpoint_to_slope_count: midpoint key -> (slope -> number of segments)
        # Groups segments by their midpoint; within each midpoint, groups by slope.
        # Two segments sharing the same midpoint and the same slope are collinear
        # through that midpoint and must be excluded to avoid over-counting
        # (they cannot form a valid trapezoid with each other).
        midpoint_to_slope_count: Dict[int, Dict[float, int]] = defaultdict(lambda: defaultdict(int))

        for i in range(n):
            x1, y1 = points[i]
            for j in range(i):
                x2, y2 = points[j]
                dx, dy = x2 - x1, y2 - y1

                if dx == 0:
                    # Vertical segment: use a sentinel slope and the x-value as intercept.
                    slope = 1e9
                    intercept = x1
                else:
                    # Line slope and a normalized intercept derived from point-slope form.
                    slope = dy / dx
                    intercept = (y1 * dx - x1 * dy) / dx

                slope_to_intercept_count[slope][intercept] += 1

                # Encode the (doubled) midpoint into a single integer key.
                # Offset by 2000 to keep coordinates non-negative, then pack with base 4000.
                midpoint_key = (x1 + x2 + 2000) * 4000 + (y1 + y2 + 2000)
                midpoint_to_slope_count[midpoint_key][slope] += 1

        answer = 0

        # Count all unordered pairs of segments that are parallel (same slope).
        # For each slope group, sum over intercept buckets multiplying running
        # prefix count by the current bucket size to count cross pairs.
        for intercept_count in slope_to_intercept_count.values():
            prefix = 0
            for count in intercept_count.values():
                answer += prefix * count
                prefix += count

        # Subtract pairs that share both midpoint and slope, since these are
        # collinear/degenerate and were incorrectly counted above.
        for slope_count in midpoint_to_slope_count.values():
            prefix = 0
            for count in slope_count.values():
                answer -= prefix * count
                prefix += count

        return answer
```

## Time and Space Complexity

- **Time Complexity:** `O(n^2)`

  The core of the algorithm is the double loop iterating over all pairs of points:
  ```python
  for i in range(n):
      for j in range(i):
          ...
  ```
  This nested loop runs `C(n, 2) = n * (n - 1) / 2` times, which is `O(n^2)`. Each iteration performs constant-time work: computing the slope `k`, intercept `b`, midpoint key `p`, and updating the dictionaries `cnt1` and `cnt2` in `O(1)`.

  After the loops, the code aggregates results by traversing the dictionaries:
  ```python
  for e in cnt1.values():
      for t in e.values():
          ...
  for e in cnt2.values():
      for t in e.values():
          ...
  ```
  Since each pair of points contributes exactly one entry across all the inner dictionaries of `cnt1` (and likewise for `cnt2`), the total number of elements traversed is bounded by the number of pairs, i.e., `O(n^2)`. Therefore, the overall time complexity is `O(n^2)`.

- **Space Complexity:** `O(n^2)`

  The dictionaries `cnt1` and `cnt2` store information for each of the `O(n^2)` point pairs. In the worst case, each pair maps to a distinct key, so the total number of entries stored across `cnt1` and `cnt2` is proportional to the number of pairs, giving `O(n^2)` space.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Using floating-point slope and intercept as hash keys

The most dangerous trap in this solution is representing both the **slope** and **intercept** as `float` values (via `dy / dx` and `(y1 * dx - x1 * dy) / dx`).

**Why it's a problem:**

- Floating-point division introduces rounding errors. Two segments that are mathematically parallel (or collinear) may produce slightly different float values, such as `0.3333333333333333` vs `0.33333333333333337`. They would then fall into **different hash buckets**, causing the algorithm to miss valid parallel pairs or fail to subtract a parallelogram.
- Conversely, two genuinely different slopes might collide to the same float after rounding, inflating the count.
- The sentinel value `1e9` for vertical lines is fragile: if any real slope happens to equal `1e9` (unlikely but conceptually unsafe), the logic breaks. It also mixes a "magic number" with legitimate computed slopes.

**Solution:** Use an **exact rational representation** by storing the slope as a *reduced integer pair* `(dy/g, dx/g)` where `g = gcd(dx, dy)`, with a normalized sign. Do the same for the intercept by keeping it as an integer-pair fraction rather than dividing.

```python
class Solution:
    def countTrapezoids(self, points: List[List[int]]) -> int:
        from math import gcd
        n = len(points)
        slope_to_intercept_count = defaultdict(lambda: defaultdict(int))
        midpoint_to_slope_count = defaultdict(lambda: defaultdict(int))

        for i in range(n):
            x1, y1 = points[i]
            for j in range(i):
                x2, y2 = points[j]
                dx, dy = x2 - x1, y2 - y1

                # Normalize slope to an exact, canonical (dy, dx) pair.
                g = gcd(dx, dy)
                sdx, sdy = dx // g, dy // g
                if sdx < 0 or (sdx == 0 and sdy < 0):
                    sdx, sdy = -sdx, -sdy
                slope = (sdy, sdx)  # exact, hashable

                # Exact intercept: cross-product form is integer and line-invariant.
                # For points on the same line: y1*dx - x1*dy is constant up to scale.
                if sdx == 0:                 # vertical line x = x1
                    intercept = (x1, 0)
                else:
                    # b * dx = y1*dx - x1*dy ; keep as reduced fraction
                    num = y1 * sdx - x1 * sdy
                    intercept = (num, sdx)

                slope_to_intercept_count[slope][intercept] += 1

                midpoint_key = (x1 + x2, y1 + y2)   # exact integer tuple
                midpoint_to_slope_count[midpoint_key][slope] += 1

        answer = 0
        for bucket in slope_to_intercept_count.values():
            prefix = 0
            for c in bucket.values():
                answer += prefix * c
                prefix += c
        for bucket in midpoint_to_slope_count.values():
            prefix = 0
            for c in bucket.values():
                answer -= prefix * c
                prefix += c
        return answer
```

### Pitfall 2: A brittle midpoint-encoding formula

The original packing `(x1 + x2 + 2000) * 4000 + (y1 + y2 + 2000)` silently **assumes** every coordinate sum lies in `[-2000, 2000)` and that `4000` is a large enough base. If the actual constraints allow coordinates whose doubled values exceed this range, **different midpoints collide to the same key**, corrupting the parallelogram subtraction.

**Why it's a problem:**

- The "magic numbers" `2000` and `4000` are tied to a specific (unstated) constraint. Any input outside that envelope produces wrong answers with **no error raised**, making the bug very hard to detect.
- It couples correctness to constraints that may change between problem versions.

**Solution:** Use a plain integer **tuple** `(x1 + x2, y1 + y2)` as the key (shown above). Tuples are hashable, exact, and never collide regardless of coordinate magnitude—eliminating the magic numbers entirely.

### Pitfall 3: Misidentifying what the second sweep subtracts

It's easy to assume the `midpoint_to_slope_count` sweep subtracts *parallelograms* (counted twice). In reality, grouping by **same midpoint and same slope** captures **collinear point-pairs** (degenerate segments lying on one line through a shared midpoint), which must be removed so they aren't mistaken for two parallel sides.

**Why it matters:** If you reason about this incorrectly, you might "fix" the wrong sweep—for example, trying to subtract once per parallelogram via the slope/intercept table—and introduce a subtle off-by-one in the count.

**Solution:** Keep the two responsibilities separate and clearly documented:
- The **first sweep** counts every same-slope, different-intercept pair (all candidate parallel-side pairs).
- The **second sweep** removes degenerate same-midpoint, same-slope collinear configurations that would otherwise be miscounted. Verify the logic against small hand-built cases (e.g., 4 collinear points, a single parallelogram, a single trapezoid) before trusting it.

