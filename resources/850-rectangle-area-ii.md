# 850. Rectangle Area II

> Source: https://algo.monster/liteproblems/850

## Problem Statement

You are given a collection of axis-aligned rectangles on a 2D plane. Each rectangle is defined by four values `[x1, y1, x2, y2]` where:
- `(x1, y1)` represents the bottom-left corner of the rectangle
- `(x2, y2)` represents the top-right corner of the rectangle

Your task is to calculate the total area covered by all these rectangles combined. When rectangles overlap, the overlapping area should only be counted once (not multiple times).

For example, if two rectangles each have an area of 4 square units and they overlap by 1 square unit, the total covered area would be 7 square units (not 8).

Since the total area might be a very large number, you need to return the result modulo `10^9 + 7`.

The challenge involves handling overlapping rectangles correctly - you cannot simply sum up all individual rectangle areas because that would count overlapping regions multiple times. Instead, you need to find the union of all rectangles and calculate the area of that union.

## Intuition

To find the total area covered by overlapping rectangles, we can think of this problem as a sweeping line problem. Imagine a vertical line sweeping from left to right across the plane.

At any given x-coordinate, this vertical line intersects with some rectangles, creating vertical segments. As we sweep from left to right, these vertical segments change - new segments appear when we encounter the left edge of a rectangle, and segments disappear when we pass the right edge of a rectangle.

The key insight is that between any two consecutive x-coordinates where rectangle edges occur, the vertical segments remain constant. So we can calculate the area as: `(width between x-coordinates) × (total height of vertical segments)`.

For example, if between `x = 2` and `x = 5` (width = 3), our vertical line intersects segments covering heights from `y = 1` to `y = 3` and `y = 4` to `y = 6` (total height = 2 + 2 = 4), then the area for this section is `3 × 4 = 12`.

The challenge is efficiently tracking which y-intervals are "active" at each x-coordinate. This is where the [segment tree](/problems/segment_tree_intro) comes in - it helps us:
1. Add a y-interval when we encounter a rectangle's left edge (increment count)
2. Remove a y-interval when we pass a rectangle's right edge (decrement count)
3. Quickly calculate the total length of active y-intervals

We process events (rectangle edges) from left to right. For each rectangle, we create two events: one for the left edge (adding the y-interval `[y1, y2]`) and one for the right edge (removing the y-interval `[y1, y2]`). By processing these events in order and maintaining the active y-intervals, we can calculate the total area without double-counting overlaps.

**Learn more about [Segment Tree](/problems/segment_tree_intro) and [Line Sweep](/problems/line_sweep_intro) patterns.**

## Solution Approach

The solution implements a sweep line algorithm with a [segment tree](/problems/segment_tree_intro) to efficiently track and merge overlapping y-intervals. Here's how the implementation works:

**1. Event Creation and Coordinate Compression:**
- For each rectangle `[x1, y1, x2, y2]`, create two events:
  - Left edge event: `(x1, y1, y2, 1)` - adds the y-interval `[y1, y2]`
  - Right edge event: `(x2, y1, y2, -1)` - removes the y-interval `[y1, y2]`
- Collect all unique y-coordinates and sort them for coordinate compression
- Map each y-coordinate to an index (0, 1, 2, ...) to work with the [segment tree](/problems/segment_tree_intro)

**2. [Segment Tree](/problems/segment_tree_intro) Structure:**
The `SegmentTree` class maintains:
- `SegmentTreeNode` objects with:
  - `left_bound, right_bound`: left and right boundaries of the segment (compressed indices)
  - `coverage_count`: count of how many intervals cover this segment
  - `covered_length`: actual length covered in this segment
- `y_coords`: the sorted array of actual y-coordinates
- The tree is built to manage intervals between consecutive y-coordinates

**3. Core Operations:**
- `modify(node_idx, query_left, query_right, delta)`: Adds `delta` to the count of interval `[query_left, query_right]`
  - When `delta = 1`: a new rectangle's y-interval is added
  - When `delta = -1`: a rectangle's y-interval is removed
- `push_up(node_idx)`: Updates the covered length of a node:
  - If `coverage_count > 0`: the entire segment is covered, so `covered_length = y_coords[right_bound + 1] - y_coords[left_bound]`
  - If `coverage_count = 0` and it's a leaf: no coverage, so `covered_length = 0`
  - If `coverage_count = 0` and it's not a leaf: `covered_length` is the sum of the children's covered lengths

**4. Main Algorithm:**
```python
1. Sort all events by x-coordinate
2. Initialize segment tree with compressed y-coordinates
3. For each event (x, y1, y2, delta):
   a. If not the first event:
      - Calculate area: segment_tree.length × (current_x - previous_x)
      - Add to total area
   b. Update segment tree:
      - modify(1, y_to_index[y1], y_to_index[y2] - 1, delta)
      - This adds/removes the y-interval [y1, y2]
4. Return total area % (10^9 + 7)
```

**Example walkthrough:**
For rectangles `[[0,0,2,2], [1,0,3,1]]`:
- Events: `[(0,0,2,1), (1,0,1,1), (2,0,2,-1), (3,0,1,-1)]`
- Y-coordinates: `[0, 1, 2]` → indices `{0:0, 1:1, 2:2}`
- At `x=0`: Add interval `[0,2]`, length = 2
- At `x=1`: Area += 2×(1-0) = 2, Add interval `[0,1]`, length still 2
- At `x=2`: Area += 2×(2-1) = 2, Remove interval `[0,2]`, length = 1
- At `x=3`: Area += 1×(3-2) = 1, Remove interval `[0,1]`, length = 0
- Total area = 2 + 2 + 1 = 5

The [segment tree](/problems/segment_tree_intro) efficiently handles the merging of overlapping y-intervals, ensuring each point is counted exactly once.

## Example Walkthrough

Let's walk through a simple example with two overlapping rectangles to illustrate the solution approach.

**Input:** `rectangles = [[0,0,3,3], [2,1,4,4]]`

These rectangles look like:
- Rectangle 1: bottom-left (0,0), top-right (3,3) - area = 9
- Rectangle 2: bottom-left (2,1), top-right (4,4) - area = 2 × 3 = 6
- They overlap in the region from (2,1) to (3,3) - overlap area = 1 × 2 = 2

**Step 1: Create Events**
For each rectangle, we create a left edge event (add) and right edge event (remove):
- Rectangle 1: `(0, 0, 3, +1)` and `(3, 0, 3, -1)`
- Rectangle 2: `(2, 1, 4, +1)` and `(4, 1, 4, -1)`

Sorted events by x-coordinate: `[(0,0,3,+1), (2,1,4,+1), (3,0,3,-1), (4,1,4,-1)]`

**Step 2: Coordinate Compression**
Unique y-coordinates: `[0, 1, 3, 4]`
Mapping: `{0:0, 1:1, 3:2, 4:3}`

**Step 3: Process Events with Segment Tree**

Initialize segment tree with y-coordinates [0, 1, 3, 4].

**At x = 0:**
- Add interval [0,3] to segment tree (indices 0→2)
- Active y-coverage: from y=0 to y=3, length = 3
- No area calculation (first event)

**At x = 2:**
- Calculate area: `3 × (2-0) = 6` square units
- Add interval [1,4] to segment tree (indices 1→3)
- Active y-coverage now:
  - [0,1]: covered once (length = 1)
  - [1,3]: covered twice (length = 2) 
  - [3,4]: covered once (length = 1)
  - Total length = 4

**At x = 3:**
- Calculate area: `4 × (3-2) = 4` square units
- Remove interval [0,3] from segment tree
- Active y-coverage: only [1,4] remains, length = 3

**At x = 4:**
- Calculate area: `3 × (4-3) = 3` square units
- Remove interval [1,4] from segment tree
- Active y-coverage: none, length = 0

**Total Area:** 6 + 4 + 3 = 13 square units

This matches our expectation: Rectangle 1 (area 9) + Rectangle 2 (area 6) - Overlap (area 2) = 13.

The segment tree efficiently tracked the y-intervals at each x-position, automatically handling the overlap by maintaining counts of how many rectangles cover each segment. When multiple rectangles cover the same y-range, the segment tree ensures we only count that range once in our length calculation.

## Python Solution

```python
from typing import List


class SegmentTreeNode:
    """Node for segment tree to track interval coverage."""
  
    def __init__(self):
        self.left_bound = 0  # Left boundary of the interval
        self.right_bound = 0  # Right boundary of the interval
        self.coverage_count = 0  # Number of times this interval is covered
        self.covered_length = 0  # Total length covered in this interval


class SegmentTree:
    """Segment tree for maintaining covered length of y-axis intervals."""
  
    def __init__(self, y_coordinates):
        """
        Initialize segment tree with compressed y-coordinates.
      
        Args:
            y_coordinates: Sorted list of unique y-coordinates
        """
        n = len(y_coordinates) - 1
        self.y_coords = y_coordinates
        self.tree = [SegmentTreeNode() for _ in range(n << 2)]  # 4 * n nodes
        self.build(1, 0, n - 1)
  
    def build(self, node_idx, left, right):
        """
        Build the segment tree structure.
      
        Args:
            node_idx: Current node index in the tree
            left: Left boundary of the interval
            right: Right boundary of the interval
        """
        self.tree[node_idx].left_bound = left
        self.tree[node_idx].right_bound = right
      
        if left != right:
            mid = (left + right) >> 1
            # Build left child
            self.build(node_idx << 1, left, mid)
            # Build right child
            self.build(node_idx << 1 | 1, mid + 1, right)
  
    def modify(self, node_idx, query_left, query_right, delta):
        """
        Update coverage count for the interval [query_left, query_right].
      
        Args:
            node_idx: Current node index in the tree
            query_left: Left boundary of query interval
            query_right: Right boundary of query interval
            delta: Change in coverage count (+1 for adding, -1 for removing)
        """
        current_node = self.tree[node_idx]
      
        # If current interval is completely within query interval
        if current_node.left_bound >= query_left and current_node.right_bound <= query_right:
            current_node.coverage_count += delta
        else:
            mid = (current_node.left_bound + current_node.right_bound) >> 1
            # Update left child if needed
            if query_left <= mid:
                self.modify(node_idx << 1, query_left, query_right, delta)
            # Update right child if needed
            if query_right > mid:
                self.modify(node_idx << 1 | 1, query_left, query_right, delta)
      
        self.push_up(node_idx)
  
    def push_up(self, node_idx):
        """
        Update the covered length for the current node based on its coverage count.
      
        Args:
            node_idx: Current node index in the tree
        """
        current_node = self.tree[node_idx]
      
        if current_node.coverage_count > 0:
            # If covered, the length is the actual y-coordinate difference
            current_node.covered_length = (
                self.y_coords[current_node.right_bound + 1] - 
                self.y_coords[current_node.left_bound]
            )
        elif current_node.left_bound == current_node.right_bound:
            # Leaf node with no coverage
            current_node.covered_length = 0
        else:
            # Internal node: sum of children's covered lengths
            left_child = self.tree[node_idx << 1]
            right_child = self.tree[node_idx << 1 | 1]
            current_node.covered_length = (
                left_child.covered_length + right_child.covered_length
            )
  
    @property
    def length(self):
        """Get the total covered length of the y-axis."""
        return self.tree[1].covered_length


class Solution:
    def rectangleArea(self, rectangles: List[List[int]]) -> int:
        """
        Calculate the total area covered by all rectangles.
      
        Uses sweep line algorithm with segment tree for efficient computation.
      
        Args:
            rectangles: List of rectangles, each as [x1, y1, x2, y2]
      
        Returns:
            Total area covered by all rectangles modulo 10^9 + 7
        """
        # Prepare events for sweep line algorithm
        events = []
        y_coordinates = set()
      
        for x1, y1, x2, y2 in rectangles:
            # Add left edge (start of rectangle)
            events.append((x1, y1, y2, 1))
            # Add right edge (end of rectangle)
            events.append((x2, y1, y2, -1))
            # Collect all unique y-coordinates
            y_coordinates.update([y1, y2])
      
        # Sort events by x-coordinate
        events.sort()
      
        # Compress y-coordinates
        sorted_y_coords = sorted(y_coordinates)
        segment_tree = SegmentTree(sorted_y_coords)
      
        # Map y-coordinate to compressed index
        y_to_index = {y: idx for idx, y in enumerate(sorted_y_coords)}
      
        # Process events and calculate area
        total_area = 0
      
        for i, (x, y1, y2, delta) in enumerate(events):
            if i > 0:
                # Add area from previous x to current x
                width = x - events[i - 1][0]
                height = segment_tree.length
                total_area += width * height
          
            # Update segment tree for current event
            # Note: y2 - 1 because we're dealing with intervals
            segment_tree.modify(1, y_to_index[y1], y_to_index[y2] - 1, delta)
      
        # Return result modulo 10^9 + 7
        MOD = int(1e9 + 7)
        return total_area % MOD
```

## Time and Space Complexity

**Time Complexity:** `O(n log n + n log m)` where `n` is the number of rectangles and `m` is the number of unique y-coordinates.

- Creating events and extracting unique y-coordinates: `O(n)` 
- Sorting events: `O(n log n)` since we have `2n` events
- Sorting unique y-coordinates: `O(m log m)` where `m ≤ 2n`
- Building the segment tree: `O(m)` 
- Processing each event with a modify operation: `O(n log m)` since each modify takes `O(log m)` and we have `2n` events
- Overall: `O(n log n + m log m + n log m) = O(n log n)` since `m ≤ 2n`

**Space Complexity:** `O(n + m)`

- Event list `events`: `O(n)` storing `2n` events
- Set `y_coordinates` and sorted list `sorted_y_coords`: `O(m)` where `m ≤ 2n`
- Segment tree nodes: `O(4m) = O(m)` since we allocate `4m` nodes for the tree
- Mapping dictionary `y_to_index`: `O(m)` for storing y-coordinate to index mapping
- Overall: `O(n + m) = O(n)` since `m ≤ 2n`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Interval Indexing in Segment Tree**

**Pitfall:** One of the most common mistakes is incorrectly handling the interval indices when calling `modify()`. The segment tree works with compressed indices representing intervals between consecutive y-coordinates, not the y-coordinates themselves.

**Incorrect approach:**
```python
# Wrong: Using y2 directly as the right boundary
segment_tree.modify(1, y_to_index[y1], y_to_index[y2], delta)
```

**Why it's wrong:** If we have y-coordinates `[0, 1, 2]`, the segment tree manages interval index 0 for `[0,1)` and index 1 for `[1,2)`. A rectangle with y-range `[0,1]` should cover only index 0, but `modify(1, 0, 1, delta)` covers indices 0 and 1, so the covered length becomes 2 instead of 1 and the area is overcounted.

**Correct approach:**
```python
# Correct: Using y2 - 1 to get the last interval index
segment_tree.modify(1, y_to_index[y1], y_to_index[y2] - 1, delta)
```

### 2. **Integer Overflow and Precision Loss for Large Areas**

**Pitfall:** Coordinates go up to `10^9`, so a single strip's `width * height` can be close to `10^18`. Python integers have arbitrary precision, so `total_area += width * height` is safe there. In Java and C++ the product must be computed in a 64-bit type (`long` / `long long`), which holds values up to about `9.2 × 10^18`; the total union area is at most `10^18`, so it also fits. In TypeScript, numbers are 64-bit floats that are exact only up to `2^53 ≈ 9 × 10^15`, so `height * width` can lose precision before the modulo is applied.

**Incorrect approach (TypeScript):**
```typescript
// height * width can exceed 2^53 and be rounded
totalArea = (totalArea + height * width) % MOD;
```

For `rectangles = [[0, 0, 999999999, 999999999]]` this returns `63`, but the correct answer is `64`.

**Correct approach (TypeScript):**
```typescript
// BigInt keeps the product exact
totalArea = Number((BigInt(totalArea) + BigInt(height) * BigInt(width)) % BigInt(MOD));
```

### 3. **Forgetting to Handle Empty Event List**

**Pitfall:** The constraints guarantee at least one rectangle, but if this code is reused where the list can be empty, `SegmentTree` is built with no intervals and `build(1, 0, -2)` indexes into an empty list.

**Solution:** Add validation:
```python
if not rectangles:
    return 0
```

### 4. **Misunderstanding Coordinate Compression**

**Pitfall:** Confusing the actual y-coordinates with their compressed indices. The segment tree operates on indices (0, 1, 2, ...), but when calculating actual lengths in `push_up()`, we must use the original y-coordinates.

**Wrong:**
```python
# Using indices instead of actual coordinates
current_node.covered_length = current_node.right_bound + 1 - current_node.left_bound
```

**Correct:**
```python
# Using actual y-coordinates for length calculation
current_node.covered_length = (
    self.y_coords[current_node.right_bound + 1] - 
    self.y_coords[current_node.left_bound]
)
```

### 5. **Incorrect Tree Size Allocation**

**Pitfall:** Allocating insufficient space for the segment tree. The tree needs approximately 4n nodes for n intervals.

**Wrong:**
```python
self.tree = [SegmentTreeNode() for _ in range(n)]  # Too small
```

**Correct:**
```python
self.tree = [SegmentTreeNode() for _ in range(n << 2)]  # 4 * n nodes
```

