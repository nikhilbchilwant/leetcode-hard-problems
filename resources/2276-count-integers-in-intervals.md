# 2276. Count Integers in Intervals

> Source: https://algo.monster/liteproblems/2276

## Problem Statement

You need to implement a data structure that maintains a collection of intervals and supports two operations:

1. **Add an interval**: Given a range `[left, right]`, add this interval to your collection. The interval includes all integers from `left` to `right` inclusive.

2. **Count unique integers**: Return the total count of distinct integers that appear in at least one interval in your collection.

The key challenge is handling overlapping intervals efficiently. When you add a new interval, it might overlap with existing intervals, and you need to count each integer only once regardless of how many intervals contain it.

For example:
- If you add intervals `[1, 5]` and `[3, 7]`, the integers covered are `{1, 2, 3, 4, 5, 6, 7}`, so the count would be 7.
- Adding another interval `[2, 4]` doesn't change the count since all integers in `[2, 4]` are already covered.

The solution uses a **[Segment Tree](/problems/segment_tree_intro) with Lazy Propagation** to efficiently handle these operations:

- The [segment tree](/problems/segment_tree_intro) dynamically creates nodes as needed (dynamic opening) to handle the large range of possible values (up to `10^9`)
- Each node in the tree represents a range and stores how many integers in that range are covered
- The `add` operation marks a range as fully covered using lazy propagation
- The `count` operation queries the root to get the total number of covered integers
- Lazy propagation ensures that updates are propagated down the tree only when necessary, maintaining `O(log n)` time complexity for both operations

The [segment tree](/problems/segment_tree_intro) divides intervals recursively: each node `[l, r]` has a left child `[l, mid]` and right child `[mid+1, r]` where `mid = (l + r) // 2`. When an interval is added, the tree marks the appropriate nodes as covered and uses the `add` flag for lazy propagation to avoid unnecessary traversals.

## Intuition

When we need to track which integers are covered by multiple intervals, the naive approach would be to maintain a set or array marking each covered integer. However, with values up to `10^9`, this becomes impractical in terms of space.

The key insight is that we're dealing with **continuous ranges** rather than individual points. When we add an interval `[left, right]`, we're essentially marking all integers in that range as "covered". This naturally leads us to think about data structures that efficiently handle range updates.

Consider what happens when we add overlapping intervals:
- Adding `[1, 5]` covers 5 integers
- Adding `[3, 7]` extends the coverage to 7 integers total (not 5 + 5 = 10)
- The overlap `[3, 5]` should only be counted once

This suggests we need a way to:
1. Mark ranges as covered
2. Handle overlaps automatically
3. Query the total coverage efficiently

A **[Segment Tree](/problems/segment_tree_intro)** is perfect for this because:
- It naturally represents ranges hierarchically
- It can mark entire ranges as covered in `O(log n)` time
- It automatically handles overlaps - once a range is marked as covered, marking it again doesn't change anything
- It can sum up the total coverage by querying the root

The challenge with the large value range (`10^9`) is solved through **dynamic opening** - we only create tree nodes when we actually need them. Instead of pre-allocating a massive tree, we start with just the root node and create children on-demand during operations.

**Lazy propagation** is crucial for efficiency. When we mark a large range as covered, instead of immediately updating all descendant nodes, we just mark the current node with a "lazy" flag. This flag gets pushed down to children only when we need to access them later. This way, marking a range takes `O(log n)` time regardless of the range size.

The brilliance of this approach is that each node stores `v` - the count of covered integers in its range. When we mark a range `[l, r]` as covered, we simply set `v = r - l + 1`. The total count is just the `v` value at the root, which represents the entire possible range `[1, 10^9]`.

**Learn more about [Segment Tree](/problems/segment_tree_intro) patterns.**

## Solution Approach

The implementation uses a **[Segment Tree](/problems/segment_tree_intro) with Dynamic Opening and Lazy Propagation**. Let's walk through each component:

### Node Structure
Each node in the [segment tree](/problems/segment_tree_intro) contains:
- `left`, `right`: Pointers to child nodes (initially `None` for dynamic opening)
- `l`, `r`: The range boundaries this node represents
- `mid`: The midpoint `(l + r) // 2` for splitting the range
- `v`: Count of covered integers in this range
- `add`: Lazy propagation flag (1 if entire range is covered, 0 otherwise)

### SegmentTree Class

**Initialization**: Creates only the root node covering the entire range `[1, 10^9 + 1]`.

**modify(l, r, v)**: Marks the range `[l, r]` as covered.
1. If the current node's range is completely within `[l, r]`, mark the entire node:
   - Set `node.v = node.r - node.l + 1` (all integers in range are covered)
   - Set `node.add = 1` for lazy propagation
   - Return early (no need to go deeper)
2. Otherwise, push down any lazy updates and recursively modify children:
   - If `l <= node.mid`, modify the left subtree
   - If `r > node.mid`, modify the right subtree
3. After modifying children, update current node's value using `pushup()`

**query(l, r)**: Returns count of covered integers in range `[l, r]`.
1. If the current node's range is completely within `[l, r]`, return `node.v`
2. Otherwise, push down lazy updates and query relevant children:
   - Query left child if `l <= node.mid`
   - Query right child if `r > node.mid`
   - Sum the results

**pushdown(node)**: Handles lazy propagation.
1. Create child nodes if they don't exist (dynamic opening)
2. If `node.add != 0`, propagate the update to children:
   - Set children's `add` flags to 1
   - Set children's values to their full range counts
   - Clear the current node's `add` flag

**pushup(node)**: Updates parent value based on children.
- Sets `node.v = node.left.v + node.right.v`

### CountIntervals Class

Simply wraps the [segment tree](/problems/segment_tree_intro):
- `add(left, right)`: Calls `tree.modify(left, right, 1)` to mark range as covered
- `count()`: Calls `tree.query(1, 10^9)` to get total covered integers

### Time Complexity
- **add()**: `O(log n)` where `n` is the range size (up to `10^9`)
- **count()**: `O(log n)` for querying the full range

### Space Complexity
- `O(m × log n)` where `m` is the number of `add` operations, as we only create nodes when needed

The key efficiency comes from:
1. **Dynamic opening** - only creating nodes when accessed
2. **Lazy propagation** - deferring updates until necessary
3. **Range representation** - storing counts instead of individual integers

## Example Walkthrough

Let's walk through a concrete example to understand how the segment tree handles interval operations.

**Initial Setup**: We start with an empty segment tree. The root node represents the range `[1, 1000000000]` with `v = 0` (no integers covered).

**Step 1: Add interval [2, 5]**

Starting from the root `[1, 1000000000]`:
- Since `[2, 5]` doesn't cover the entire root range, we need to recurse
- Calculate mid = `(1 + 1000000000) // 2 = 500000000`
- Since `5 <= 500000000`, we only go left
- Create left child node `[1, 500000000]` dynamically
- Continue recursing down until we find a node that `[2, 5]` completely covers

Eventually, we reach a node representing range `[2, 5]` (after several recursive splits):
- Set this node's `v = 5 - 2 + 1 = 4` (four integers covered)
- Set `add = 1` (lazy flag indicating full coverage)
- Propagate the count back up: root's `v` becomes 4

**Step 2: Add interval [4, 8]**

Starting from root `[1, 1000000000]` with `v = 4`:
- `[4, 8]` doesn't cover the entire range, so recurse
- This time we need to traverse to nodes that overlap with `[4, 8]`

When we reach the previously created nodes:
- The node for `[2, 5]` partially overlaps with `[4, 8]`
- We push down the lazy update (creating children if needed)
- The overlap `[4, 5]` is already marked as covered
- We mark the new portion `[6, 8]` as covered (3 new integers)

After propagation back up:
- Root's `v = 4 + 3 = 7` (integers 2, 3, 4, 5, 6, 7, 8 are covered)

**Step 3: Add interval [3, 6]**

Starting from root with `v = 7`:
- `[3, 6]` overlaps with existing covered ranges
- As we traverse down, we find:
  - `[3, 5]` is already covered (from previous intervals)
  - `[6, 6]` is already covered (from interval `[4, 8]`)
- No new integers are added

Root's `v` remains 7.

**Step 4: Count operation**

Query the root node `[1, 1000000000]`:
- Simply return root's `v = 7`

**Key Observations**:

1. **Dynamic Node Creation**: We only created nodes for ranges we actually needed (around `[2, 8]`), not the entire billion-integer range.

2. **Lazy Propagation in Action**: When we marked `[2, 5]` as covered, we didn't immediately create all child nodes. We just set `add = 1` and `v = 4`. Children were created only when needed for the `[4, 8]` operation.

3. **Automatic Overlap Handling**: When adding `[3, 6]`, the tree automatically recognized that all these integers were already covered. No double-counting occurred.

4. **Efficient Counting**: Getting the total count is just reading the root's `v` value - O(1) after the tree is built.

The tree structure after these operations (simplified) might look like:
```
                Root [1, 10^9]: v=7
                      /
            [1, 500M]: v=7
                /
          [1, 250K]: v=7
              /
        ... (more nodes)
            /
      [2, 8]: v=7
      (with appropriate child nodes)
```

This demonstrates how the segment tree efficiently handles large ranges by only creating necessary nodes and using lazy propagation to defer updates until needed.

## Python Solution

```python
class Node:
    """Node class for dynamic segment tree."""
    __slots__ = ("left", "right", "l", "r", "mid", "v", "add")

    def __init__(self, left_bound: int, right_bound: int):
        """
        Initialize a segment tree node.
      
        Args:
            left_bound: Left boundary of the interval
            right_bound: Right boundary of the interval
        """
        self.left = None  # Left child node
        self.right = None  # Right child node
        self.l = left_bound  # Left boundary of current segment
        self.r = right_bound  # Right boundary of current segment
        self.mid = (left_bound + right_bound) // 2  # Midpoint for splitting
        self.v = 0  # Value stored in this node (count of covered integers)
        self.add = 0  # Lazy propagation flag (1 if entire range should be marked)


class SegmentTree:
    """Dynamic segment tree for range updates and queries."""
  
    def __init__(self):
        """Initialize segment tree with range [1, 10^9]."""
        self.root = Node(1, int(1e9) + 1)

    def modify(self, left: int, right: int, value: int, node: Node = None) -> None:
        """
        Update range [left, right] with given value.
      
        Args:
            left: Left boundary of update range
            right: Right boundary of update range
            value: Value to set (1 for marking as covered)
            node: Current node in recursion (defaults to root)
        """
        if node is None:
            node = self.root
          
        # Invalid range
        if left > right:
            return
          
        # Current segment is completely within update range
        if node.l >= left and node.r <= right:
            # Mark entire segment as covered
            node.v = node.r - node.l + 1
            node.add = value
            return
          
        # Push down lazy updates before going deeper
        self.pushdown(node)
      
        # Recursively update left child if needed
        if left <= node.mid:
            self.modify(left, right, value, node.left)
          
        # Recursively update right child if needed
        if right > node.mid:
            self.modify(left, right, value, node.right)
          
        # Update current node value based on children
        self.pushup(node)

    def query(self, left: int, right: int, node: Node = None) -> int:
        """
        Query the count of covered integers in range [left, right].
      
        Args:
            left: Left boundary of query range
            right: Right boundary of query range
            node: Current node in recursion (defaults to root)
          
        Returns:
            Count of covered integers in the range
        """
        if node is None:
            node = self.root
          
        # Invalid range
        if left > right:
            return 0
          
        # Current segment is completely within query range
        if node.l >= left and node.r <= right:
            return node.v
          
        # Push down lazy updates before querying children
        self.pushdown(node)
      
        result = 0
      
        # Query left child if needed
        if left <= node.mid:
            result += self.query(left, right, node.left)
          
        # Query right child if needed
        if right > node.mid:
            result += self.query(left, right, node.right)
          
        return result

    def pushup(self, node: Node) -> None:
        """
        Update parent node value based on children values.
      
        Args:
            node: Node to update
        """
        node.v = node.left.v + node.right.v

    def pushdown(self, node: Node) -> None:
        """
        Push down lazy updates to children and create children if needed.
      
        Args:
            node: Node whose lazy updates need to be pushed down
        """
        # Create children nodes if they don't exist (dynamic segment tree)
        if node.left is None:
            node.left = Node(node.l, node.mid)
        if node.right is None:
            node.right = Node(node.mid + 1, node.r)
          
        # Push down lazy update if exists
        if node.add != 0:
            left_child, right_child = node.left, node.right
          
            # Apply lazy update to children
            left_child.add = node.add
            right_child.add = node.add
          
            # Update children values (mark entire range as covered)
            left_child.v = left_child.r - left_child.l + 1
            right_child.v = right_child.r - right_child.l + 1
          
            # Clear lazy update from current node
            node.add = 0


class CountIntervals:
    """Class to count total number of integers covered by added intervals."""
  
    def __init__(self):
        """Initialize with an empty segment tree."""
        self.tree = SegmentTree()

    def add(self, left: int, right: int) -> None:
        """
        Add interval [left, right] to the collection.
      
        Args:
            left: Left boundary of interval
            right: Right boundary of interval
        """
        self.tree.modify(left, right, 1)

    def count(self) -> int:
        """
        Return the total count of integers covered by at least one interval.
      
        Returns:
            Count of covered integers
        """
        return self.tree.query(1, int(1e9))


# Your CountIntervals object will be instantiated and called as such:
# obj = CountIntervals()
# obj.add(left, right)
# param_2 = obj.count()
```

## Time and Space Complexity

**Time Complexity:**
- For the `add(left, right)` operation: `O(log n)`, where `n` is the range of values (in this case `10^9`). The segment tree uses lazy propagation and only visits nodes along the path from root to the relevant leaf nodes, creating nodes dynamically as needed. The height of the tree is `log n`, so at most `O(log n)` nodes are visited.

- For the `count()` operation: `O(log n)`. Similar to the `add` operation, it traverses the tree from root to relevant nodes, with the tree height being `log n`.

**Space Complexity:**
- The space complexity is `O(m × log n)`, where `m` is the number of `add` operations performed and `n` is the data range (`10^9`). 
- This implementation uses a dynamic segment tree that creates nodes lazily only when needed. Each `add` operation may create at most `O(log n)` new nodes along the path from root to leaves.
- In the worst case, after `m` operations, the total number of nodes created is `O(m × log n)`.
- Each node stores a constant amount of data (`left`, `right`, `l`, `r`, `mid`, `v`, `add`), so the total space is `O(m × log n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow in Range Calculations**

One of the most common pitfalls when implementing this segment tree is potential integer overflow when calculating the midpoint or range sizes, especially given the large range `[1, 10^9]`.

**Pitfall Example:**
```python
# Problematic code that might cause issues in some languages
mid = (l + r) // 2  # Could overflow if l + r > MAX_INT
```

**Solution:**
In Python, this isn't an issue due to arbitrary precision integers, but in other languages or when porting this code, use:
```python
mid = l + (r - l) // 2  # Safer calculation avoiding overflow
```

### 2. **Off-by-One Errors in Range Boundaries**

A critical pitfall is inconsistent handling of inclusive vs. exclusive ranges, particularly when splitting ranges and calculating counts.

**Pitfall Example:**
```python
# Wrong: Using exclusive right boundary but calculating as inclusive
node.v = node.r - node.l  # Missing +1 for inclusive range
```

**Solution:**
Always be consistent with inclusive ranges:
```python
node.v = node.r - node.l + 1  # Correct for inclusive [l, r]
```

### 3. **Forgetting to Push Down Before Operations**

Failing to push down lazy updates before querying or modifying children leads to incorrect results.

**Pitfall Example:**
```python
def modify(self, left, right, value, node):
    # Missing pushdown before recursive calls
    if left <= node.mid:
        self.modify(left, right, value, node.left)  # Wrong without pushdown
```

**Solution:**
Always push down before accessing children:
```python
def modify(self, left, right, value, node):
    if node.l >= left and node.r <= right:
        # Full coverage - no need to push down
        return
  
    self.pushdown(node)  # Essential before accessing children
  
    if left <= node.mid:
        self.modify(left, right, value, node.left)
```

### 4. **Memory Issues with Static Segment Tree**

Creating a full segment tree for range `[1, 10^9]` statically would require enormous memory.

**Pitfall Example:**
```python
# Don't do this - would create ~2 billion nodes!
def __init__(self):
    self.tree = [Node(i, i) for i in range(1, int(1e9) + 1)]
```

**Solution:**
Use dynamic segment tree that creates nodes only when needed:
```python
def pushdown(self, node):
    # Create children only when accessed
    if node.left is None:
        node.left = Node(node.l, node.mid)
    if node.right is None:
        node.right = Node(node.mid + 1, node.r)
```

### 5. **Incorrect Lazy Propagation Clear**

Not clearing the lazy flag after pushing down causes values to be applied multiple times.

**Pitfall Example:**
```python
def pushdown(self, node):
    if node.add != 0:
        node.left.add = node.add
        node.right.add = node.add
        # Forgot to clear: node.add = 0
```

**Solution:**
Always clear the lazy flag after propagation:
```python
def pushdown(self, node):
    if node.add != 0:
        # Apply to children
        node.left.add = node.add
        node.right.add = node.add
        # Update children values
        node.left.v = node.left.r - node.left.l + 1
        node.right.v = node.right.r - node.right.l + 1
        # Clear the flag
        node.add = 0  # Critical!
```

### 6. **Pushup Without Checking Child Existence**

Attempting to push up values when children might not exist in a dynamic tree.

**Pitfall Example:**
```python
def pushup(self, node):
    # Dangerous if children don't exist
    node.v = node.left.v + node.right.v
```

**Solution:**
The current implementation handles this by ensuring pushdown creates children before pushup is called. Alternatively, add existence checks:
```python
def pushup(self, node):
    if node.left and node.right:
        node.v = node.left.v + node.right.v
```

