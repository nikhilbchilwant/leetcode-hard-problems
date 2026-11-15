# 732. My Calendar III

> Source: https://algo.monster/liteproblems/732

## Problem Statement

You need to implement a calendar system that tracks the maximum number of overlapping events at any point in time.

A **k-booking** occurs when k events overlap at some point in time. For example, if 3 events all share a common time period, that creates a 3-booking.

The task is to implement a `MyCalendarThree` class with the following methods:

1. `MyCalendarThree()`: Initializes an empty calendar object.

2. `book(startTime, endTime)`: Adds a new event to the calendar with time interval `[startTime, endTime)` (note: the interval is half-open, meaning it includes `startTime` but excludes `endTime`). After adding this event, the method returns the maximum k-booking among all events in the calendar.

For each call to `book()`, you need to:
- Add the new event to your calendar
- Determine the maximum number of events that overlap at any single point in time across the entire calendar
- Return this maximum overlap count

The solution uses a **[Segment Tree](/problems/segment_tree_intro)** data structure with lazy propagation to efficiently handle the large time range (up to `10^9`) and track overlapping intervals. The segment tree:
- Dynamically creates nodes as needed to handle the large range
- Uses the `modify()` operation to increment the count for the interval `[start+1, end]` by 1
- Uses the `query()` operation to find the maximum overlap count across all time points
- Employs lazy propagation (`add` field) to efficiently update interval ranges without updating every single point

## Intuition

When we think about finding the maximum number of overlapping events, we need to track how many events are active at each point in time. The naive approach would be to maintain a counter for every single time point, but with a time range up to `10^9`, this would require too much memory.

The key observation is that we're dealing with **interval updates** - when we book an event from time `start` to `end`, we're essentially saying "increase the booking count by 1 for all time points in this range." After multiple bookings, we want to know the maximum booking count across all time points.

This naturally leads us to think about data structures that can efficiently handle:
1. Range updates (add 1 to all values in a range)
2. Range queries (find the maximum value in a range)

A **[Segment Tree](/problems/segment_tree_intro)** is perfect for this because it can perform both operations in `O(log n)` time. However, updating every single point in a range would still be inefficient. This is where **lazy propagation** comes in - instead of immediately updating all child nodes when we modify a range, we mark the node with a "lazy" value (`add`) that will be pushed down to children only when needed.

The call is `modify(start + 1, end, 1)` because the [segment tree](/problems/segment_tree_intro) covers indices starting at 1, so time `t` is stored at index `t + 1`. The half-open event `[start, end)` covers times `start` through `end - 1`, which are indices `start + 1` through `end`.

Since the time range can be up to `10^9`, creating all nodes upfront would waste memory. Instead, we use **dynamic node creation** - nodes are only created when they're actually needed during modifications or queries. This way, we only create nodes for the parts of the time range that are actually used, making the solution both time and space efficient.

The maximum k-booking is simply the maximum value across the entire range after all bookings, which we obtain by querying the full range `[1, 10^9 + 1]`.

**Learn more about [Segment Tree](/problems/segment_tree_intro), [Binary Search](/problems/binary_search_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The solution uses a **[Segment Tree](/problems/segment_tree_intro) with Lazy Propagation** to efficiently handle range updates and queries. Let's walk through the implementation:

### [Segment Tree](/problems/segment_tree_intro) Structure

Each `Node` in the [segment tree](/problems/segment_tree_intro) represents an interval `[l, r]` and maintains:
- `l, r`: The left and right boundaries of the interval
- `mid`: The midpoint, calculated as `(l + r) >> 1` (bitwise right shift for division by 2)
- `v`: The maximum number of bookings in this interval
- `add`: The lazy propagation value (pending updates to be pushed to children)
- `left, right`: Pointers to child nodes (created dynamically)

### Core Operations

**1. Modify Operation (`modify(l, r, v)`)**

This operation adds value `v` to all elements in range `[l, r]`:
- If the current node's interval is completely within `[l, r]`, we update the node's value and lazy marker, then return (lazy propagation)
- Otherwise, we push down any pending updates to children
- Recursively modify the left child if the target range overlaps with `[node.l, node.mid]`
- Recursively modify the right child if the target range overlaps with `[node.mid + 1, node.r]`
- After modifying children, we push up to update the current node's value

**2. Query Operation (`query(l, r)`)**

This finds the maximum value in range `[l, r]`:
- If the current node's interval is completely within `[l, r]`, return the node's value
- Otherwise, push down pending updates and recursively query children
- Return the maximum value from relevant children

**3. Push Down (`pushdown(node)`)**

This propagates lazy updates to children:
- Creates child nodes if they don't exist (dynamic node creation)
- If there's a pending update (`node.add != 0`):
  - Add the pending value to both children's values and lazy markers
  - Clear the current node's lazy marker

**4. Push Up (`pushup(node)`)**

Updates the current node's value based on its children:
- Sets `node.v = max(node.left.v, node.right.v)`

### MyCalendarThree Implementation

The `MyCalendarThree` class:
1. Initializes a [segment tree](/problems/segment_tree_intro) covering range `[1, 10^9 + 1]`
2. For each `book(start, end)` call:
   - Performs `modify(start + 1, end, 1)` to increment the booking count in the interval
   - Queries the entire range `[1, 10^9 + 1]` to find the maximum booking count
   - Returns this maximum value

The time complexity for each booking is `O(log n)` where `n` is the range size, and the space complexity is `O(m log n)` where `m` is the number of bookings, since nodes are created dynamically only when needed.

## Example Walkthrough

Let's walk through a small example to illustrate how the solution works. Suppose we have the following sequence of bookings:
1. `book(10, 20)` - Event from time 10 to 20
2. `book(50, 60)` - Event from time 50 to 60
3. `book(10, 40)` - Event from time 10 to 40
4. `book(5, 15)` - Event from time 5 to 15

**Initial State:**
- Segment tree root covers range `[1, 1000000001]`
- All values are 0 (no bookings yet)

**After `book(10, 20)`:**
- Call `modify(11, 20, 1)` to add 1 to range [11, 20]
- The segment tree lazily marks this range with +1
- Query entire range returns maximum value: **1**
- Timeline visualization: `[10, 20)` has 1 booking

**After `book(50, 60)`:**
- Call `modify(51, 60, 1)` to add 1 to range [51, 60]
- This creates a separate branch in the tree (no overlap with previous)
- Query returns maximum value: **1** (still no overlaps)
- Timeline: `[10, 20)` has 1 booking, `[50, 60)` has 1 booking

**After `book(10, 40)`:**
- Call `modify(11, 40, 1)` to add 1 to range [11, 40]
- Range [11, 20] now has value 2 (overlaps with first booking)
- Range [21, 40] has value 1
- Query returns maximum value: **2**
- Timeline: `[10, 20)` has 2 bookings, `[20, 40)` has 1 booking, `[50, 60)` has 1 booking

**After `book(5, 15)`:**
- Call `modify(6, 15, 1)` to add 1 to range [6, 15]
- Range [11, 15] now has value 3 (overlaps with first and third bookings)
- Range [6, 10] has value 1
- Range [16, 20] has value 2
- Query returns maximum value: **3**
- Timeline: `[5, 10)` has 1 booking, `[10, 15)` has 3 bookings, `[15, 20)` has 2 bookings, `[20, 40)` has 1 booking, `[50, 60)` has 1 booking

The segment tree efficiently tracks these overlaps using lazy propagation:
- When we modify a range, we don't immediately update all affected nodes
- Instead, we mark parent nodes with a lazy "add" value
- These lazy values are pushed down only when we need to access child nodes
- The tree dynamically creates nodes only for the ranges we actually use (not all 10^9 positions)

This way, each `book()` operation runs in O(log n) time, making it efficient even for the large time range.

## Python Solution

```python
class Node:
    """
    Node class for dynamic segment tree.
    Each node represents a range [l, r] and stores the maximum value in that range.
    """
    def __init__(self, left: int, right: int):
        self.left_child = None  # Left child node
        self.right_child = None  # Right child node
        self.left_bound = left  # Left boundary of the range
        self.right_bound = right  # Right boundary of the range
        self.mid = (left + right) >> 1  # Midpoint of the range
        self.max_value = 0  # Maximum value in this range
        self.lazy_add = 0  # Lazy propagation value for range updates


class SegmentTree:
    """
    Dynamic Segment Tree implementation with lazy propagation.
    Supports range updates and range maximum queries.
    """
    def __init__(self):
        # Initialize root node covering range [1, 10^9 + 1]
        self.root = Node(1, int(1e9 + 1))

    def modify(self, left: int, right: int, value: int, node: Node = None) -> None:
        """
        Add a value to all elements in range [left, right].

        Args:
            left: Left boundary of update range
            right: Right boundary of update range
            value: Value to add to the range
            node: Current node in recursion (default: root)
        """
        if left > right:
            return

        if node is None:
            node = self.root

        # If current node's range is completely within update range
        if node.left_bound >= left and node.right_bound <= right:
            node.max_value += value
            node.lazy_add += value
            return

        # Push down lazy updates before recursing
        self.pushdown(node)

        # Recursively update left subtree if needed
        if left <= node.mid:
            self.modify(left, right, value, node.left_child)

        # Recursively update right subtree if needed
        if right > node.mid:
            self.modify(left, right, value, node.right_child)

        # Update current node's value based on children
        self.pushup(node)

    def query(self, left: int, right: int, node: Node = None) -> int:
        """
        Query the maximum value in range [left, right].

        Args:
            left: Left boundary of query range
            right: Right boundary of query range
            node: Current node in recursion (default: root)

        Returns:
            Maximum value in the specified range
        """
        if left > right:
            return 0

        if node is None:
            node = self.root

        # If current node's range is completely within query range
        if node.left_bound >= left and node.right_bound <= right:
            return node.max_value

        # Push down lazy updates before querying
        self.pushdown(node)

        max_val = 0

        # Query left subtree if needed
        if left <= node.mid:
            max_val = max(max_val, self.query(left, right, node.left_child))

        # Query right subtree if needed
        if right > node.mid:
            max_val = max(max_val, self.query(left, right, node.right_child))

        return max_val

    def pushup(self, node: Node) -> None:
        """
        Update parent node's value based on children's values.

        Args:
            node: Node to update
        """
        node.max_value = max(node.left_child.max_value, node.right_child.max_value)

    def pushdown(self, node: Node) -> None:
        """
        Push lazy updates from parent to children and create children if needed.

        Args:
            node: Node whose lazy updates need to be pushed down
        """
        # Create left child if it doesn't exist
        if node.left_child is None:
            node.left_child = Node(node.left_bound, node.mid)

        # Create right child if it doesn't exist
        if node.right_child is None:
            node.right_child = Node(node.mid + 1, node.right_bound)

        # Push down lazy updates if any
        if node.lazy_add:
            node.left_child.max_value += node.lazy_add
            node.right_child.max_value += node.lazy_add
            node.left_child.lazy_add += node.lazy_add
            node.right_child.lazy_add += node.lazy_add
            node.lazy_add = 0


class MyCalendarThree:
    """
    Calendar booking system that tracks the maximum number of overlapping events.
    Uses a segment tree to efficiently handle range updates and queries.
    """
    def __init__(self):
        self.tree = SegmentTree()

    def book(self, start: int, end: int) -> int:
        """
        Book a new event in the calendar and return the maximum number of overlapping events.

        Args:
            start: Start time of the event
            end: End time of the event (exclusive)

        Returns:
            Maximum number of overlapping events after this booking
        """
        # Increment count for range [start+1, end] (using 1-indexed coordinates)
        self.tree.modify(start + 1, end, 1)

        # Query the maximum overlap across the entire range
        return self.tree.query(1, int(1e9 + 1))


# Your MyCalendarThree object will be instantiated and called as such:
# obj = MyCalendarThree()
# param_1 = obj.book(start, end)
```

## Time and Space Complexity

**Time Complexity:** `O(n × log M)` where `n` is the number of bookings and `M` is the range of values (10^9 in this case).

For each `book` operation:
- The `modify` operation traverses the segment tree from root to leaves, creating nodes as needed. The depth of the tree is `O(log M)` where `M = 10^9` is the range of the segment tree.
- The `query` operation also traverses the tree with depth `O(log M)`.
- Therefore, each booking takes `O(log M)` time.
- For `n` bookings, the total time complexity is `O(n × log M)`.

Since `M` is a constant (10^9), this can be simplified to `O(n × log 10^9)` = `O(n × 30)` = `O(n)` in practical terms, though technically it's `O(n × log M)`.

**Space Complexity:** `O(n × log M)` where `n` is the number of bookings.

The segment tree uses dynamic node creation (lazy propagation):
- Nodes are only created when needed during `pushdown` operations.
- Each `book` operation potentially creates at most `O(log M)` new nodes along the path from root to the affected leaves.
- After `n` bookings, the maximum number of nodes created is `O(n × log M)`.
- Since `log M` is about 30 for `M = 10^9`, this is often described as linear in `n` with a constant factor of about 30, but the precise bound is `O(n × log M)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Range Indexing**
One of the most common mistakes is using the wrong range for the modify operation. The problem states that the interval is `[startTime, endTime)` (half-open), but the segment tree implementation uses 1-indexed coordinates.

**Pitfall Code:**
```python
def book(self, start: int, end: int) -> int:
    # Wrong: covers times start - 1 through end - 1 (end - start + 1 points),
    # so [10, 20) and [20, 30) are reported as overlapping (returns 2)
    self.tree.modify(start, end, 1)
    # Or wrong: maps time start to index start, but the tree starts at index 1,
    # so book(0, 1) calls modify(0, 0) and recurses forever
    self.tree.modify(start, end - 1, 1)
```

**Correct Solution:**
```python
def book(self, start: int, end: int) -> int:
    # Correct: Convert to 1-indexed and handle half-open interval
    self.tree.modify(start + 1, end, 1)
```

### 2. **Memory Issues with Static Segment Tree**
Creating a complete segment tree for range `[1, 10^9]` upfront would require enormous memory and cause Memory Limit Exceeded.

**Pitfall Code:**
```python
class SegmentTree:
    def __init__(self):
        # Wrong: Pre-allocating array for entire range
        self.tree = [0] * (4 * int(1e9))  # This will cause memory error!
```

**Correct Solution:**
Dynamic node creation - nodes are only created when needed during modify/query operations:
```python
def pushdown(self, node: Node) -> None:
    # Create children only when needed
    if node.left_child is None:
        node.left_child = Node(node.left_bound, node.mid)
    if node.right_child is None:
        node.right_child = Node(node.mid + 1, node.right_bound)
```

### 3. **Forgetting Lazy Propagation**
Without lazy propagation, range updates become O(n) instead of O(log n), causing Time Limit Exceeded.

**Pitfall Code:**
```python
def modify(self, left: int, right: int, value: int, node: Node) -> None:
    if node.left_bound == node.right_bound:  # Leaf node
        node.max_value += value
        return
    # Wrong: Not using lazy propagation, updating all leaves
    if left <= node.mid:
        self.modify(left, right, value, node.left_child)
    if right > node.mid:
        self.modify(left, right, value, node.right_child)
```

**Correct Solution:**
Use lazy propagation to defer updates:
```python
def modify(self, left: int, right: int, value: int, node: Node) -> None:
    # If entire range is covered, update lazily
    if node.left_bound >= left and node.right_bound <= right:
        node.max_value += value
        node.lazy_add += value  # Store for lazy propagation
        return
```

### 4. **Incorrect Pushup Logic**
After modifying children, forgetting to update the parent node's value leads to incorrect query results.

**Pitfall Code:**
```python
def modify(self, left: int, right: int, value: int, node: Node) -> None:
    # ... recursive calls ...
    # Wrong: Forgetting to update parent after modifying children
    return
```

**Correct Solution:**
Always pushup after modifying children:
```python
def modify(self, left: int, right: int, value: int, node: Node) -> None:
    # ... handle lazy propagation and recursive calls ...

    # Critical: Update parent based on children
    self.pushup(node)
```

### 5. **Not Pushing Down Before Queries**
Forgetting to push down lazy values before querying children results in stale/incorrect values.

**Pitfall Code:**
```python
def query(self, left: int, right: int, node: Node) -> int:
    if node.left_bound >= left and node.right_bound <= right:
        return node.max_value

    # Wrong: Not pushing down lazy updates
    max_val = 0
    if left <= node.mid:
        max_val = max(max_val, self.query(left, right, node.left_child))
```

**Correct Solution:**
Always push down before accessing children:
```python
def query(self, left: int, right: int, node: Node) -> int:
    if node.left_bound >= left and node.right_bound <= right:
        return node.max_value

    # Essential: Push down lazy updates first
    self.pushdown(node)

    # Now query children with updated values
    max_val = 0
    if left <= node.mid:
        max_val = max(max_val, self.query(left, right, node.left_child))
```

