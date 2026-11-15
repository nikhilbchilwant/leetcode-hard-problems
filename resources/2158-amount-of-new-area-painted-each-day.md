# 2158. Amount of New Area Painted Each Day

> Source: https://algo.monster/liteproblems/2158

## Problem Statement

You have a long painting represented as a number line that needs to be painted over multiple days. You're given a 2D array `paint` where each element `paint[i] = [start_i, end_i]` represents the painting task for day `i`. On day `i`, you need to paint the area from position `start_i` to position `end_i` (exclusive of `end_i`).

The key constraint is that each area of the painting should be painted at most once to avoid creating an uneven painting. If an area has already been painted on a previous day, you should not paint it again.

Your task is to return an array `worklog` where `worklog[i]` represents the amount of new area that was actually painted on day `i`. This means if some portions of the range `[start_i, end_i]` were already painted on previous days, you only count the unpainted portions.

For example, if on day 0 you paint `[1, 4]` (painting 3 units), and on day 1 you need to paint `[3, 6]`, you would only paint the new area `[4, 6]` (2 units) since `[3, 4]` was already painted on day 0.

## Intuition

The core challenge is tracking which parts of the painting have already been painted across multiple days. We need to efficiently determine for each new painting task how much of it overlaps with already-painted areas.

A naive approach would be to maintain a boolean array marking each position as painted or unpainted. For each day, we'd iterate through the range and count unpainted positions, then mark them as painted. However, this becomes inefficient when dealing with large ranges, potentially requiring O(n * range_size) operations.

The key insight is that we're dealing with interval operations - we need to mark entire ranges as "painted" and query how much of a range is already painted. This naturally leads us to consider data structures optimized for range operations.

A [Segment Tree](/problems/segment_tree_intro) is perfect for this scenario because it excels at:
1. **Range Updates**: Marking an entire interval `[start, end]` as painted in O(log n) time
2. **Range Queries**: Checking how much of an interval is already painted in O(log n) time

The solution maintains a [segment tree](/problems/segment_tree_intro) where each node stores how much of its range is painted. When we process a painting task for day `i`:
- First, we query the range `[start, end]` to find out how much is already painted
- The new area painted = total range size - already painted area
- Then we update the segment tree to mark the entire range as painted

The lazy propagation technique in the [segment tree](/problems/segment_tree_intro) ensures we don't unnecessarily update all affected nodes immediately, making the solution more efficient. The `add` field in each node acts as a lazy flag indicating that the entire subtree should be marked as painted when needed.

By transforming the problem into range query and update operations, we achieve an efficient O(n log m) solution where n is the number of painting tasks and m is the range of coordinates.

**Learn more about [Segment Tree](/problems/segment_tree_intro) patterns.**

## Solution Approach

The solution implements a [Segment Tree](/problems/segment_tree_intro) with lazy propagation to efficiently handle range updates and queries. Let's walk through the implementation step by step:

### [Segment Tree](/problems/segment_tree_intro) Structure

The `Node` class represents each node in the [segment tree](/problems/segment_tree_intro):
- `l, r`: The left and right boundaries of the interval this node represents
- `mid`: The midpoint, calculated as `(l + r) >> 1` (bitwise right shift for division by 2)
- `v`: The count of painted units in this node's range
- `add`: Lazy propagation flag (1 if the entire range should be marked as painted)

### Core Operations

**1. Tree Initialization**
```python
self.root = Node(1, 10**5 + 10)
```
Creates a root node covering the range `[1, 100010]`, which accommodates all possible painting positions.

**2. Query Operation** (`query(l, r)`)
- Returns the number of already-painted units in range `[l, r]`
- If the current node's range is completely within `[l, r]`, return `node.v`
- Otherwise, push down lazy updates and recursively query left and right children
- Sum up the results from both subtrees

**3. Modify Operation** (`modify(l, r, v)`)
- Marks the range `[l, r]` as painted
- If the current node's range is completely within `[l, r]`, mark the entire range as painted:
  - Set `node.v = node.r - node.l + 1` (all units in range are painted)
  - Set `node.add = 1` (lazy flag for children)
- Otherwise, push down lazy updates and recursively modify appropriate children
- Update current node's value based on children (`pushup`)

**4. Lazy Propagation**
- `pushdown`: Creates children nodes if they don't exist and propagates the lazy flag
  - If `node.add = 1`, mark both children as fully painted
  - Clear the parent's lazy flag after propagation
- `pushup`: Updates parent node's painted count as sum of children's counts

### Main Algorithm

For each painting task `[start, end]`:
1. Adjust coordinates: `l = start + 1, r = end` (shifting to 1-indexed)
2. Query how many units in `[l, r]` are already painted: `v = tree.query(l, r)`
3. Calculate new area painted: `r - l + 1 - v` (total range size minus already painted)
4. Mark the entire range as painted: `tree.modify(l, r, 1)`
5. Add the new painted area to the result

### Time Complexity
- Each query and modify operation: O(log m) where m is the coordinate range
- Total for n painting tasks: O(n log m)

### Space Complexity
- O(m) for the [segment tree](/problems/segment_tree_intro) nodes created on demand through lazy initialization

The elegance of this solution lies in using lazy propagation to avoid unnecessary updates - we only create and update nodes when actually needed, making it both time and space efficient.

## Example Walkthrough

Let's walk through a concrete example with `paint = [[1, 4], [3, 6], [2, 5]]`.

**Initial State:**
- Segment tree covers range [1, 100010], all unpainted (v = 0)
- Result array: []

**Day 0: Paint [1, 4]**
- Convert to 1-indexed: query and modify range [2, 4]
- Query [2, 4]: Returns 0 (nothing painted yet)
- New area painted: (4 - 2 + 1) - 0 = 3 units
- Modify [2, 4]: Mark positions 2, 3, 4 as painted
- Tree state: Range [2, 4] has v = 3
- Result: [3]

**Day 1: Paint [3, 6]**
- Convert to 1-indexed: query and modify range [4, 6]
- Query [4, 6]:
  - Check overlap with already painted [2, 4]
  - Position 4 is painted, positions 5, 6 are not
  - Returns 1 (one unit already painted)
- New area painted: (6 - 4 + 1) - 1 = 2 units
- Modify [4, 6]: Mark entire range as painted (4 already was, 5 and 6 newly painted)
- Tree state: [2, 4] has v = 3, [5, 6] has v = 2 (internally merged as [2, 6] with v = 5)
- Result: [3, 2]

**Day 2: Paint [2, 5]**
- Convert to 1-indexed: query and modify range [3, 5]
- Query [3, 5]:
  - Positions 3, 4, 5 are all already painted (from days 0 and 1)
  - Returns 3 (all three units already painted)
- New area painted: (5 - 3 + 1) - 3 = 0 units
- Modify [3, 5]: No actual change since already painted
- Result: [3, 2, 0]

**Final Output:** [3, 2, 0]

The segment tree efficiently tracks painted intervals:
- Day 0: Painted 3 new units
- Day 1: Painted 2 new units (skipping the 1 overlapping unit)
- Day 2: Painted 0 new units (entire range already covered)

## Python Solution

```python
from typing import List, Optional


class Node:
    """Represents a node in the segment tree."""

    def __init__(self, left_bound: int, right_bound: int):
        """
        Initialize a segment tree node.

        Args:
            left_bound: Left boundary of the interval this node represents
            right_bound: Right boundary of the interval this node represents
        """
        self.left_child: Optional[Node] = None
        self.right_child: Optional[Node] = None
        self.left_bound = left_bound
        self.right_bound = right_bound
        self.mid = (left_bound + right_bound) >> 1  # Midpoint for splitting
        self.painted_count = 0  # Number of painted cells in this interval
        self.lazy_tag = 0  # Lazy propagation tag (1 if entire interval should be painted)


class SegmentTree:
    """Dynamic segment tree for tracking painted intervals."""

    def __init__(self):
        """Initialize segment tree with range [1, 100010]."""
        self.root = Node(1, 100010)

    def modify(self, left: int, right: int, value: int, node: Optional[Node] = None) -> None:
        """
        Mark interval [left, right] as painted.

        Args:
            left: Left boundary of interval to paint
            right: Right boundary of interval to paint
            value: Value to set (1 for painted)
            node: Current node in recursion (defaults to root)
        """
        if left > right:
            return

        if node is None:
            node = self.root

        # If current node's interval is completely covered by [left, right]
        if node.left_bound >= left and node.right_bound <= right:
            # Mark entire interval as painted
            node.painted_count = node.right_bound - node.left_bound + 1
            node.lazy_tag = value
            return

        # Push down lazy updates before recursing
        self.pushdown(node)

        # Recursively update left and right children
        if left <= node.mid:
            self.modify(left, right, value, node.left_child)
        if right > node.mid:
            self.modify(left, right, value, node.right_child)

        # Update current node's value based on children
        self.pushup(node)

    def query(self, left: int, right: int, node: Optional[Node] = None) -> int:
        """
        Query number of painted cells in interval [left, right].

        Args:
            left: Left boundary of query interval
            right: Right boundary of query interval
            node: Current node in recursion (defaults to root)

        Returns:
            Number of painted cells in the queried interval
        """
        if left > right:
            return 0

        if node is None:
            node = self.root

        # If current node's interval is completely covered by query range
        if node.left_bound >= left and node.right_bound <= right:
            return node.painted_count

        # Push down lazy updates before querying children
        self.pushdown(node)

        # Query left and right children
        painted_cells = 0
        if left <= node.mid:
            painted_cells += self.query(left, right, node.left_child)
        if right > node.mid:
            painted_cells += self.query(left, right, node.right_child)

        return painted_cells

    def pushup(self, node: Node) -> None:
        """
        Update parent node's value based on children's values.

        Args:
            node: Node to update based on its children
        """
        node.painted_count = node.left_child.painted_count + node.right_child.painted_count

    def pushdown(self, node: Node) -> None:
        """
        Push lazy updates from parent to children (lazy propagation).

        Args:
            node: Node whose lazy updates need to be pushed to children
        """
        # Create children nodes if they don't exist (dynamic segment tree)
        if node.left_child is None:
            node.left_child = Node(node.left_bound, node.mid)
        if node.right_child is None:
            node.right_child = Node(node.mid + 1, node.right_bound)

        # If there's a lazy update to push down
        if node.lazy_tag:
            left_child = node.left_child
            right_child = node.right_child

            # Mark children's entire intervals as painted
            left_child.painted_count = left_child.right_bound - left_child.left_bound + 1
            right_child.painted_count = right_child.right_bound - right_child.left_bound + 1

            # Propagate lazy tag to children
            left_child.lazy_tag = node.lazy_tag
            right_child.lazy_tag = node.lazy_tag

            # Clear parent's lazy tag
            node.lazy_tag = 0


class Solution:
    def amountPainted(self, paint: List[List[int]]) -> List[int]:
        """
        Calculate amount of new area painted for each paint operation.

        Args:
            paint: List of [start, end) intervals to paint

        Returns:
            List where each element is the amount of new area painted
        """
        tree = SegmentTree()
        result = []

        for start, end in paint:
            # Convert to 1-indexed closed interval [left, right]
            left, right = start + 1, end

            # Query how many cells are already painted
            already_painted = tree.query(left, right)

            # Calculate newly painted cells
            newly_painted = (right - left + 1) - already_painted
            result.append(newly_painted)

            # Mark the interval as painted
            tree.modify(left, right, 1)

        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n * log(M))` where `n` is the number of paint operations and `M` is the range size (10^5 in this case).

- The SegmentTree is initialized with a range `[1, 10^5 + 10]`, creating a root node in `O(1)` time.
- For each paint operation in the list:
  - `query(l, r)`: In the worst case, this traverses from root to leaves, creating nodes lazily along the path. The tree height is `O(log M)`, so query takes `O(log M)` time.
  - `modify(l, r, v)`: Similarly, this operation traverses the tree from root to potentially multiple leaves, taking `O(log M)` time.
- Since we perform these operations for each of the `n` paint intervals, the total time complexity is `O(n * log M)`.

**Space Complexity:** `O(min(n * log(M), M))`

- The segment tree uses lazy propagation and dynamic node creation (nodes are only created when needed via `pushdown`).
- In the worst case, if we need to access many different ranges, we could create up to `O(M)` nodes to cover the entire range.
- However, since nodes are created lazily during the `n` operations, and each operation creates at most `O(log M)` nodes along its path, the space used is also bounded by `O(n * log M)`.
- Therefore, the space complexity is `O(min(n * log(M), M))`, which represents the minimum of the theoretical maximum nodes and the nodes actually created through operations.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Interval Conversion Between 0-indexed and 1-indexed Systems**

The most common pitfall in this solution is mishandling the conversion between the problem's 0-indexed half-open intervals `[start, end)` and the segment tree's 1-indexed closed intervals `[left, right]`.

**The Pitfall:**
```python
# Incorrect conversions that often occur:
left, right = start, end - 1  # Wrong! Doesn't shift to 1-indexed
left, right = start + 1, end + 1  # Wrong! Over-shifts the right boundary
left, right = start, end  # Wrong! Uses 0-indexed directly
```

**Why This Happens:**
- The problem uses `[start, end)` where `end` is exclusive (painting positions start to end-1)
- The segment tree implementation uses 1-indexed closed intervals `[left, right]` where both boundaries are inclusive
- The conversion requires both shifting indices by 1 AND adjusting for the exclusive/inclusive difference

**The Correct Solution:**
```python
# Correct conversion:
left, right = start + 1, end
```
This works because:
- `start` in 0-indexed becomes `start + 1` in 1-indexed
- `end` (exclusive) in 0-indexed becomes `end` (inclusive) in 1-indexed
- Example: `[0, 3)` in problem space (painting positions 0, 1, 2) becomes `[1, 3]` in segment tree space

### 2. **Segment Tree Range Initialization Too Small**

**The Pitfall:**
```python
# Wrong: Range too small for the problem constraints
self.root = Node(1, 10**4)  # May not cover all possible positions
```

**The Solution:**
```python
# Correct: Ensure range covers all possible painting positions
self.root = Node(1, 100010)  # or Node(1, 10**5 + 10)
```
Always check the problem constraints and add a small buffer to avoid index out of bounds issues.

### 3. **Forgetting to Handle Empty Intervals**

**The Pitfall:**
Not checking for invalid intervals where `left > right` after conversion can cause incorrect results or infinite recursion.

**The Solution:**
```python
def modify(self, left: int, right: int, value: int, node: Optional[Node] = None) -> None:
    if left > right:  # Critical check
        return
    # ... rest of the code
```

### 4. **Misunderstanding Lazy Propagation Logic**

**The Pitfall:**
```python
# Wrong: Forgetting to clear the parent's lazy tag after propagation
def pushdown(self, node: Node) -> None:
    if node.lazy_tag:
        # ... propagate to children
        # Forgot: node.lazy_tag = 0  # This line is crucial!
```

**Why This Matters:**
Without clearing the parent's lazy tag, the same update gets applied multiple times on subsequent operations, leading to incorrect painted counts.

**The Solution:**
Always clear the parent's lazy tag after propagating it to children to prevent duplicate updates.

