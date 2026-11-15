# 3454. Separate Squares II

> Source: https://algo.monster/liteproblems/3454

## Problem Statement

You are given a 2D integer array `squares`. Each `squares[i] = [xi, yi, li]` represents the coordinates of the bottom-left point and the side length of a square parallel to the x-axis.

Find the **minimum** y-coordinate value of a horizontal line such that the total area covered by squares above the line **equals** the total area covered by squares below the line.

Answers within `10^-5` of the actual answer will be accepted.

**Note**: Squares **may** overlap. Overlapping areas should be counted **only once** in this version.

In other words, each square spans horizontally from `xi` to `xi + li` and vertically from `yi` to `yi + li`. Because squares can overlap, when computing the covered area you must count each region of the plane only once, regardless of how many squares cover it. Your goal is to choose a horizontal line `y = k` so that the union area lying strictly below the line is equal to the union area lying strictly above the line. Among all valid lines, you should return the smallest such `k`.

## Intuition

The key observation is that the horizontal line we want is determined by the **union area** of all squares. If the total covered area (counting overlaps only once) is `S`, then we need a line `y = k` such that the union area below it equals `S / 2`. As we move a horizontal line upward from the bottom, the area below it increases **monotonically** from `0` to `S`, so there is exactly one position where the area below equals half. This monotonic growth is what lets us pinpoint the answer.

The challenge is computing the union area while accounting for overlaps. A natural way to handle overlapping rectangular regions is the **sweep line** technique. Imagine sweeping a horizontal line from the bottom to the top. At any given height, the squares that are currently "active" cover some set of intervals on the x-axis. The total length of this covered x-interval (union of intervals, overlaps counted once) tells us the **width** of the covered region at that height. Multiplying this width by the vertical distance the sweep line travels before the active set changes gives us an area slice.

The active set only changes at the **top and bottom edges** of squares. So we treat each square's bottom edge as an event that **adds** its x-interval to the active coverage, and each top edge as an event that **removes** it. By sorting all these events by their `y` coordinate and processing them in order, we can accumulate the union area piece by piece between consecutive event heights.

To efficiently track the total covered x-length as intervals are added and removed, we use a **[segment tree](/problems/segment_tree_intro) over the discretized x-coordinates**. Each node maintains a `cnt` (how many active intervals fully cover this range) and a `length` (the covered length within this range). When `cnt > 0`, the whole range is covered; otherwise the covered length is the sum from its children. This lets us query the current covered width in `O(1)` at the root after each update.

The algorithm therefore runs in two passes. The **first pass** sweeps through all events to compute the total union area `S`, giving us the target `S / 2`. The **second pass** repeats the sweep, accumulating area slice by slice. As soon as adding the next slice would reach or exceed the target, we know the answer lies within that slice. Since the width stays constant during that interval, we can solve for the exact `y` by a simple linear interpolation: `y0 + (target - area) / width`, where `width` is the current covered x-length. Because the area grows monotonically, the first `y` reaching half is automatically the minimum valid line.

**Learn more about [Segment Tree](/problems/segment_tree_intro) and [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

We solve this problem using the **Sweep Line** algorithm combined with a **[Segment Tree](/problems/segment_tree_intro)** for interval coverage. The implementation proceeds through the following steps.

**1. Preprocess Event Points**

For each square `[x1, y1, l]`, we compute its top-right corner as `x2 = x1 + l` and `y2 = y1 + l`. We then generate two events:

- A **bottom edge** event `(y1, x1, x2, 1)`, where `1` means we add the interval `[x1, x2]` to the active coverage.
- A **top edge** event `(y2, x1, x2, -1)`, where `-1` means we remove the interval.

We also collect every `x1` and `x2` into a set `xs` so we can discretize the x-coordinates later.

```python
xs = set()
segs = []
for x1, y1, l in squares:
    x2, y2 = x1 + l, y1 + l
    xs.update([x1, x2])
    segs.append((y1, x1, x2, 1))
    segs.append((y2, x1, x2, -1))
```

**2. Sort Event Points**

We sort `segs` by `y` coordinate in ascending order, so the sweep line moves from bottom to top.

**3. Build the [Segment Tree](/problems/segment_tree_intro)**

We sort the distinct x-coordinates into `st` and build a [segment tree](/problems/segment_tree_intro) over them. Each `Node` keeps two pieces of information:

- `cnt`: how many active intervals fully cover this node's range.
- `length`: the total covered x-length within this node's range.

A dictionary `d` maps each x-coordinate to its index for quick lookup when updating the tree.

The crucial part is `pushup`, which recomputes a node's covered length:

- If `cnt > 0`, the entire range is covered, so `length = nums[r + 1] - nums[l]`.
- Else if it's a leaf, `length = 0`.
- Otherwise, `length = left child length + right child length`.

```python
def pushup(self, u):
    if self.tr[u].cnt:
        self.tr[u].length = self.nums[self.tr[u].r + 1] - self.nums[self.tr[u].l]
    elif self.tr[u].l == self.tr[u].r:
        self.tr[u].length = 0
    else:
        self.tr[u].length = self.tr[u << 1].length + self.tr[u << 1 | 1].length
```

Note that we index intervals rather than points: the [segment tree](/problems/segment_tree_intro) covers `n - 1` unit intervals between `n` discretized coordinates. When updating the interval `[x1, x2]`, we modify the range `[d[x1], d[x2] - 1]`.

**4. First Scan — Compute the Total Union Area**

We sweep through all events in order. Between the previous sweep height `y0` and the current event height `y`, the covered width is `tree.length` (the union x-length at the root). The area of this slice is `(y - y0) * tree.length`, which we add to the running `area`. Then we apply the current event by calling `modify` to add or remove its interval, and update `y0`.

```python
area = 0
y0 = 0
for y, x1, x2, k in segs:
    area += (y - y0) * tree.length
    tree.modify(1, d[x1], d[x2] - 1, k)
    y0 = y
```

After this pass, `area` holds the total union area `S`, and the target becomes `target = area / 2`.

**5. Second Scan — Locate the Answer**

We repeat the sweep, but this time we look for the moment the accumulated area first reaches the target. For each slice, we compute its area `t = (y - y0) * tree.length`. If `area + t >= target`, the dividing line lies inside this slice. Since the width `tree.length` is constant throughout the slice, we interpolate linearly:

```python
return y0 + (target - area) / tree.length
```

Otherwise, we add `t` to `area`, apply the event, and continue.

```python
area = 0
y0 = 0
for y, x1, x2, k in segs:
    t = (y - y0) * tree.length
    if area + t >= target:
        return y0 + (target - area) / tree.length
    area += t
    tree.modify(1, d[x1], d[x2] - 1, k)
    y0 = y
```

Because the area below the sweep line grows monotonically, the first height reaching half the total area is exactly the minimum valid `y`, satisfying the problem's requirement.

**Complexity Analysis**

Let `n` be the number of squares. Sorting the events takes `O(n log n)`, and each of the `O(n)` [segment tree](/problems/segment_tree_intro) updates costs `O(log n)`. Therefore the overall time complexity is `O(n log n)`, and the space complexity is `O(n)` for storing the events and the segment tree.

## Example Walkthrough

Consider two overlapping squares:

```
squares = [[0, 0, 2], [1, 1, 2]]
```

- **Square A**: `[0, 0, 2]` → spans x from `0` to `2`, y from `0` to `2`.
- **Square B**: `[1, 1, 2]` → spans x from `1` to `3`, y from `1` to `3`.

They overlap in the region `x ∈ [1, 2]`, `y ∈ [1, 2]`.

---

**Step 1 — Preprocess Event Points**

For each square we generate a bottom edge (`+1`) and a top edge (`-1`):

| Event source   | `(y, x1, x2, k)`    |
|----------------|---------------------|
| A bottom       | `(0, 0, 2, +1)`     |
| A top          | `(2, 0, 2, -1)`     |
| B bottom       | `(1, 1, 3, +1)`     |
| B top          | `(3, 1, 3, -1)`     |

Collected x-coordinates: `xs = {0, 1, 2, 3}`.

---

**Step 2 — Sort Events by y**

```
segs = [(0, 0, 2, +1), (1, 1, 3, +1), (2, 0, 2, -1), (3, 1, 3, -1)]
```

---

**Step 3 — Build the Segment Tree**

Discretized coordinates: `st = [0, 1, 2, 3]`, with mapping `d = {0:0, 1:1, 2:2, 3:3}`.
The tree covers the **unit intervals** `[0,1]`, `[1,2]`, `[2,3]` (3 intervals between 4 points).

---

**Step 4 — First Scan: Compute Total Union Area**

Start with `area = 0`, `y0 = 0`.

| Event             | slice = `(y - y0) * width` | running `area` | action                                       | new `width` |
|-------------------|----------------------------|----------------|----------------------------------------------|-------------|
| `(0, 0, 2, +1)`   | `(0-0) * 0 = 0`            | `0`            | add `[0,2]` → cover intervals `[0,1],[1,2]`  | `2`         |
| `(1, 1, 3, +1)`   | `(1-0) * 2 = 2`           | `2`            | add `[1,3]` → also cover `[2,3]`             | `3`         |
| `(2, 0, 2, -1)`   | `(2-1) * 3 = 3`           | `5`            | remove `[0,2]` → `[0,1]` off, `[1,2]` still on via B | `2`  |
| `(3, 1, 3, -1)`   | `(3-2) * 2 = 2`           | `7`            | remove `[1,3]` → all off                      | `0`         |

Total union area `S = 7`. Target = `S / 2 = 3.5`.

> Sanity check: A has area 4, B has area 4, overlap is 1, so union = `4 + 4 - 1 = 7`. ✓

---

**Step 5 — Second Scan: Locate the Answer**

Reset `area = 0`, `y0 = 0`. We look for the slice where accumulated area first reaches `3.5`.

| Event             | `t = (y - y0) * width` | `area + t` vs target `3.5` | decision                          |
|-------------------|------------------------|----------------------------|-----------------------------------|
| `(0, 0, 2, +1)`   | `(0-0)*0 = 0`         | `0 < 3.5`                  | add slice (area=0), apply, y0=0   |
| `(1, 1, 3, +1)`   | `(1-0)*2 = 2`         | `0 + 2 = 2 < 3.5`          | add slice (area=2), apply, y0=1   |
| `(2, 0, 2, -1)`   | `(2-1)*3 = 3`         | `2 + 3 = 5 ≥ 3.5`          | **answer lies in this slice**     |

At this point the line is inside the interval `y ∈ [1, 2]` where the constant width is `3`. We interpolate:

```
y = y0 + (target - area) / width
  = 1 + (3.5 - 2) / 3
  = 1 + 1.5 / 3
  = 1.5
```

---

**Result**

The minimum y-coordinate is **`1.5`**.

> Verification: Below `y = 1.5` — square A contributes the region `y ∈ [0, 1.5]` over `x ∈ [0, 2]` (area `3`), and square B contributes `y ∈ [1, 1.5]` over `x ∈ [2, 3]` (the part of B not already counted, area `0.5`), totaling `3.5`. The area above also equals `3.5`. ✓

## Python Solution

```python
from typing import List


class Node:
    """A node in the segment tree, representing an interval over compressed x-coordinates."""

    __slots__ = ("left", "right", "count", "length")

    def __init__(self):
        # Boundaries of the interval (indices into the compressed coordinate array)
        self.left = self.right = 0
        # How many active segments fully cover this interval
        self.count = 0
        # Total covered length of the x-axis within this interval
        self.length = 0


class SegmentTree:
    """Segment tree used to track the total covered length on the x-axis during a sweep."""

    def __init__(self, nums: List[int]):
        # nums holds the sorted, unique x-coordinates (compressed boundaries)
        n = len(nums) - 1
        self.nums = nums
        # Allocate 4n nodes (a safe upper bound for a segment tree)
        self.tree = [Node() for _ in range(n << 2)]
        self.build(1, 0, n - 1)

    def build(self, u: int, left: int, right: int) -> None:
        """Recursively build the tree over the index range [left, right]."""
        self.tree[u].left, self.tree[u].right = left, right
        if left != right:
            mid = (left + right) >> 1
            self.build(u << 1, left, mid)
            self.build(u << 1 | 1, mid + 1, right)

    def modify(self, u: int, left: int, right: int, k: int) -> None:
        """Add k to the coverage count of every elementary interval inside [left, right]."""
        if self.tree[u].left >= left and self.tree[u].right <= right:
            # Current node is fully inside the target range
            self.tree[u].count += k
        else:
            mid = (self.tree[u].left + self.tree[u].right) >> 1
            if left <= mid:
                self.modify(u << 1, left, right, k)
            if right > mid:
                self.modify(u << 1 | 1, left, right, k)
        self.push_up(u)

    def push_up(self, u: int) -> None:
        """Recalculate the covered length of node u from its state and children."""
        if self.tree[u].count:
            # This interval is fully covered, so its length spans the whole interval
            self.tree[u].length = (
                self.nums[self.tree[u].right + 1] - self.nums[self.tree[u].left]
            )
        elif self.tree[u].left == self.tree[u].right:
            # Leaf node with no coverage contributes nothing
            self.tree[u].length = 0
        else:
            # Otherwise, the covered length is the sum from both children
            self.tree[u].length = (
                self.tree[u << 1].length + self.tree[u << 1 | 1].length
            )

    @property
    def length(self) -> int:
        """Total covered length on the x-axis (stored at the root)."""
        return self.tree[1].length


class Solution:
    def separateSquares(self, squares: List[List[int]]) -> float:
        xs = set()
        segments = []
        # Build horizontal edge events for the sweep along the y-axis
        for x1, y1, side in squares:
            x2, y2 = x1 + side, y1 + side
            xs.update([x1, x2])
            # Bottom edge opens coverage (+1); top edge closes it (-1)
            segments.append((y1, x1, x2, 1))
            segments.append((y2, x1, x2, -1))
        # Sort events by y-coordinate to sweep upward
        segments.sort()

        # Compress x-coordinates and map each coordinate to its index
        sorted_xs = sorted(xs)
        tree = SegmentTree(sorted_xs)
        index = {x: i for i, x in enumerate(sorted_xs)}

        # First pass: compute the total area covered by all squares
        area = 0
        prev_y = 0
        for y, x1, x2, k in segments:
            # Accumulate area = covered width * height since the last event
            area += (y - prev_y) * tree.length
            tree.modify(1, index[x1], index[x2] - 1, k)
            prev_y = y

        # We want to find the y at which the lower portion equals half the area
        target = area / 2

        # Second pass: replay the sweep until the accumulated area reaches the target
        area = 0
        prev_y = 0
        for y, x1, x2, k in segments:
            strip = (y - prev_y) * tree.length
            if area + strip >= target:
                # The dividing line lies within this strip;
                # interpolate the exact y-position using the current width
                return prev_y + (target - area) / tree.length
            area += strip
            tree.modify(1, index[x1], index[x2] - 1, k)
            prev_y = y
        return 0
```

## Time and Space Complexity

**Time Complexity: `O(n × log n)`**

Let `n` be the number of squares. The analysis proceeds as follows:

- **Coordinate collection and setup**: For each square, we add 2 x-coordinates to the set `xs` and create 2 segments. This yields `O(n)` coordinates and `O(n)` segments.

- **Sorting**: 
  - Sorting `segs` (containing `2n` segments) takes `O(n × log n)`.
  - Sorting `xs` into `st` (containing up to `2n` coordinates) takes `O(n × log n)`.

- **Segment Tree construction**: The `build` method recursively constructs a tree over up to `2n` discretized x-intervals, creating `O(n)` nodes, so it takes `O(n)`.

- **Sweep line with segment tree updates**: There are two sweep passes, each iterating over the `2n` segments. For each segment, a `modify` operation is performed on the segment tree, costing `O(log n)` per call (the recursion depth is bounded by the tree height). Each sweep thus costs `O(n × log n)`.

Combining all parts, the dominant terms are the sorting and the sweep-line updates, giving an overall time complexity of `O(n × log n)`.

**Space Complexity: `O(n)`**

The space usage breaks down as follows:

- The set `xs` and sorted list `st` store up to `2n` x-coordinates, requiring `O(n)` space.
- The list `segs` stores `2n` segments, requiring `O(n)` space.
- The dictionary `d` maps coordinates to indices, using `O(n)` space.
- The segment tree allocates `n << 2` (i.e., `4n`) `Node` objects, requiring `O(n)` space. The recursion stack during `build` and `modify` has depth `O(log n)`.

Therefore, the total space complexity is `O(n)`, where `n` is the number of squares.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Forgetting the "Index Intervals, Not Points" Convention in the Segment Tree

The single most common mistake in this sweep-line + segment-tree pattern is mixing up **coordinate points** with the **elementary intervals between them**.

When you have `n` distinct x-coordinates stored in `sorted_xs`, they define only `n - 1` elementary intervals. The segment tree must be built over indices `[0, n - 2]`, **not** `[0, n - 1]`:

```python
n = len(nums) - 1          # number of intervals, NOT number of points
self.build(1, 0, n - 1)    # indices 0 .. n-2
```

A natural but **wrong** instinct is to write `self.build(1, 0, len(nums) - 1)`, which creates a leaf for the last coordinate that has no interval to its right. This corrupts every `push_up` that reads `nums[right + 1]`, causing an index-out-of-range error or silently wrong lengths.

Tightly coupled to this is the update range. When you add a square spanning `[x1, x2]`, you must modify:

```python
tree.modify(1, index[x1], index[x2] - 1, k)   # note the  - 1
```

**Why `- 1`?** The interval `[x1, x2]` covers the elementary intervals *starting* at `index[x1]` up to *but not including* the one at `index[x2]`. The elementary interval with index `i` represents the physical span `[nums[i], nums[i+1]]`. So the last elementary interval inside `[x1, x2]` has index `index[x2] - 1`.

**Verification trick:** Pick two adjacent coordinates `a < b` with one square covering exactly `[a, b]`. After the `+1` event, `tree.length` should equal `b - a`. If it equals `0` or throws, your indexing is off by one.

---

### Pitfall 2: Using a Standard Lazy-Propagation Segment Tree (Pushdown)

Many people reach for the textbook lazy-propagation segment tree for range updates. **That is the wrong tool here.** The classic "area of union of rectangles" segment tree relies on a special property:

- Every `modify` is **balanced** — every `+1` (bottom edge) is eventually matched by a `-1` (top edge) on the *exact same* interval `[x1, x2]`.

Because of this balance, you **never push the `count` down** to children. Instead, `count` stays on the node where the range fully matches, and `push_up` recomputes `length` purely from the local `count` and the children's `length`:

```python
def push_up(self, u):
    if self.tree[u].count:           # this whole interval is covered
        self.tree[u].length = nums[right + 1] - nums[left]
    elif leaf:
        self.tree[u].length = 0
    else:
        self.tree[u].length = left_child.length + right_child.length
```

If you wrongly introduce `push_down`, a partial `-1` could "leak" a removal into children that were never individually marked, breaking the count and producing negative or nonsensical lengths. **Do not pushdown; only pushup.**

---

### Pitfall 3: Misreading "Union Area" vs. "Summed Area" (the Overlap Trap)

This problem's note explicitly states overlapping regions count **once**. A frequent error is computing area as the simple sum of `li²` over all squares, or treating each square independently in the sweep. Both ignore overlap.

The correct slice area is:

```python
area += (y - prev_y) * tree.length
```

where `tree.length` is the **union** width at the current sweep height — guaranteed by the segment tree's coverage logic. If you instead multiplied height by a *sum of active interval widths*, overlapping squares at the same height would be double-counted, inflating the total and shifting the answer.

**Contrast:** There is a sibling problem where overlaps *are* counted multiple times. For that version you'd track total summed width, not union width. Always confirm which variant you're solving.

---

### Pitfall 4: Assuming `prev_y = 0` Is a Safe Starting Height

The code initializes `prev_y = 0`. This is only correct because the problem guarantees coordinates are non-negative integers (`yi >= 0`). The very first event has `tree.length == 0` (no intervals active yet), so the slice `(first_y - 0) * 0 = 0` contributes nothing — and the off-by-zero start is harmless.

However, if the constraints ever allowed **negative y-coordinates**, starting at `0` would be a latent bug: you'd compute area over `[0, first_y]` with the wrong sign or skip the region below zero entirely.

**Robust fix:** Anchor `prev_y` to the first event's y-value, which always has zero active coverage:

```python
prev_y = segments[0][0]    # bottom-most edge; coverage is empty before this
```

This makes the logic correct regardless of the coordinate sign.

---

### Pitfall 5: Division by Zero During Interpolation

The final interpolation step divides by `tree.length`:

```python
return prev_y + (target - area) / tree.length
```

If the answer happens to land exactly at a y-level where the sweep enters a strip with `tree.length == 0` (a vertical gap with no covered region), this divides by zero. With a valid problem (`target > 0` and squares present) the target is reached while `tree.length > 0`, but defensive code should guard it:

```python
if tree.length > 0 and area + strip >= target:
    return prev_y + (target - area) / tree.length
```

Skipping zero-width strips ensures the division is always well-defined and the returned `k` is the **minimum** valid line — since accumulated area only grows on strips with positive width, the first qualifying strip yields the smallest `k`.

