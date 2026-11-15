# 715. Range Module

> Source: https://algo.monster/liteproblems/715

## Problem Statement

You need to design a data structure called Range Module that tracks ranges of numbers. The ranges are represented as **half-open intervals** `[left, right)`, which means they include all real numbers `x` where `left <= x < right` (includes left but excludes right).

The Range Module should support the following operations:

1. **`RangeModule()`**: Initializes an empty range module.

2. **`addRange(int left, int right)`**: Adds the half-open interval `[left, right)` to the module. This means you start tracking all numbers in this range. If the new interval overlaps with existing tracked intervals, they should be merged - any numbers in `[left, right)` that aren't already tracked should now be tracked.

3. **`queryRange(int left, int right)`**: Returns `true` if **every** real number in the interval `[left, right)` is currently being tracked, and `false` otherwise. The entire range must be covered for this to return `true`.

4. **`removeRange(int left, int right)`**: Stops tracking all real numbers in the half-open interval `[left, right)`. This removes the specified range from the tracked intervals, potentially splitting existing intervals.

For example:
- If you add range `[10, 20)` and then add `[15, 25)`, the module tracks `[10, 25)`.
- If you're tracking `[10, 30)` and remove `[15, 20)`, you'll be left tracking `[10, 15)` and `[20, 30)`.
- A query for `[15, 17)` returns `true` only if all numbers from 15 (inclusive) to 17 (exclusive) are being tracked.

## Intuition

When we need to efficiently manage interval operations (add, remove, query), we should think about data structures that can handle range updates and queries efficiently. The key insight is that we're dealing with continuous ranges that can overlap, merge, or split.

Initially, we might consider maintaining a sorted list of disjoint intervals. While this works, operations like merging overlapping intervals or splitting intervals during removal can be complex and potentially `O(n)` in the worst case.

A [segment tree](/problems/segment_tree_intro) is particularly well-suited for this problem because:

1. **Range operations are natural**: Segment trees are designed specifically for range queries and updates. Each node represents an interval, making it intuitive to track whether ranges are covered.

2. **Efficient updates**: Instead of modifying every single point in a range, we can mark entire subtrees as "tracked" or "not tracked" using lazy propagation. This gives us `O(log n)` complexity for updates.

3. **Handling the large range**: The problem doesn't specify bounds, but the solution assumes a range of `[1, 10^9]`. A regular array would be too memory-intensive. A dynamic [segment tree](/problems/segment_tree_intro) only creates nodes as needed, making it memory-efficient.

The key realization is that we can use a boolean value at each node to indicate whether that entire range is being tracked. With lazy propagation, we can defer updates until necessary:
- When adding a range, we mark nodes as tracked (`add = 1`)
- When removing a range, we mark nodes as not tracked (`add = -1`)
- The lazy value propagates down only when we need to traverse deeper into the tree

This approach transforms what could be complex interval manipulation into simple tree traversal with lazy updates, achieving `O(log n)` time complexity for all operations.

**Learn more about [Segment Tree](/problems/segment_tree_intro) patterns.**

## Solution Approach

The solution implements a **dynamic [segment tree](/problems/segment_tree_intro) with lazy propagation** to efficiently handle range operations. Let's walk through the key components:

### Node Structure
Each node in the [segment tree](/problems/segment_tree_intro) contains:
- `left`, `right`: Pointers to child nodes (created dynamically)
- `add` (`lazy_tag` in the code): Lazy propagation flag (`1` for add, `-1` for remove, `0` for no pending operation)
- `v` (`is_covered` in the code): Boolean indicating if the entire range represented by this node is tracked

### Core Operations

**1. Modify Operation (`modify`)**
This handles both `addRange` and `removeRange`:
- Takes parameters: `left`, `right` (target range), `v` (1 for add, -1 for remove)
- If current node's range `[l, r]` is completely within target range `[left, right]`:
  - Apply the operation directly using lazy propagation
  - Set `node.add` and update `node.v`
- Otherwise, push down any pending updates and recursively modify children
- The range is split at `mid = (l + r) >> 1`
- After recursion, push up to update parent based on children's states

**2. Query Operation (`query`)**
Checks if a range is fully tracked:
- If current node's range is completely within query range, return `node.v`
- Otherwise, push down pending updates and recursively query children
- Returns `true` only if all queried portions are tracked

**3. Lazy Propagation Helpers**

`pushdown`: Propagates pending updates to children
- Creates child nodes if they don't exist (dynamic allocation)
- Transfers the `add` value to children
- Updates children's `v` values based on the operation
- Clears the current node's `add` flag

`pushup`: Updates parent node based on children
- A node is tracked only if both children exist and are tracked
- Uses `node.v = node.left.v and node.right.v`

### RangeModule Integration
The main class wraps the [segment tree](/problems/segment_tree_intro):
- `addRange(left, right)`: Calls `modify(left, right-1, 1)` 
- `removeRange(left, right)`: Calls `modify(left, right-1, -1)`
- `queryRange(left, right)`: Calls `query(left, right-1)`

Note: The implementation converts half-open intervals `[left, right)` to closed intervals `[left, right-1]` for the [segment tree](/problems/segment_tree_intro) operations.

### Time & Space Complexity
- **Time**: `O(log n)` per operation where `n` is the range size (`10^9`)
- **Space**: `O(m log n)` where `m` is the number of operations, due to dynamic node creation

The dynamic allocation ensures we only create nodes for ranges that are actually accessed, making this approach memory-efficient despite the large potential range.

## Example Walkthrough

Let's walk through a sequence of operations to see how the dynamic segment tree handles range tracking:

**Initial State**: Empty tree with range [1, 10^9]

**Operation 1: `addRange(10, 20)`**
- We want to track interval [10, 20)
- The segment tree calls `modify(10, 19, 1)` (converting to closed interval)
- Starting from root [1, 10^9]:
  - Not fully contained, so split at mid = 5×10^8
  - Recursively go left to [1, 5×10^8]
  - Continue splitting until reaching a node that can represent [10, 19]
  - Mark this node with `v = true` and `add = 1` (lazy flag)
- Result: Range [10, 19] is now tracked

**Operation 2: `addRange(15, 25)`**
- We want to track interval [15, 25)
- The segment tree calls `modify(15, 24, 1)`
- The tree traversal finds:
  - [15, 19] overlaps with existing tracked range [10, 19]
  - [20, 24] is a new range to track
- After this operation, we effectively have [10, 24] tracked
- The tree structure now has nodes representing these merged ranges

**Operation 3: `queryRange(12, 18)`**
- We want to check if [12, 18) is fully tracked
- The segment tree calls `query(12, 17)`
- Starting from root:
  - Navigate down to find nodes covering [12, 17]
  - Push down any lazy updates along the path
  - Since [12, 17] is within the tracked range [10, 24], return `true`

**Operation 4: `removeRange(14, 16)`**
- We want to stop tracking [14, 16)
- The segment tree calls `modify(14, 15, -1)`
- This splits the tracked range:
  - [10, 13] remains tracked
  - [14, 15] is marked as not tracked (v = false)
  - [16, 24] remains tracked
- The tree handles this by:
  - Finding the node covering [14, 15]
  - Setting its `add = -1` and `v = false`
  - Using `pushup` to update parent nodes

**Operation 5: `queryRange(13, 15)`**
- We want to check if [13, 15) is fully tracked
- The segment tree calls `query(13, 14)`
- The query finds:
  - [13, 13] is tracked (part of [10, 13])
  - [14, 14] is NOT tracked (was removed)
- Returns `false` because not all of [13, 14] is tracked

The key insight is that the segment tree dynamically creates nodes only for the ranges we actually use, avoiding the need to pre-allocate nodes for the entire [1, 10^9] range. The lazy propagation ensures we don't update every affected node immediately, but only when necessary during subsequent operations.

## Python Solution

```python
class SegmentTreeNode:
    """Node for the dynamic segment tree."""
    __slots__ = ['left', 'right', 'lazy_tag', 'is_covered']
  
    def __init__(self):
        self.left = None  # Left child node
        self.right = None  # Right child node
        self.lazy_tag = 0  # Lazy propagation tag: 1 for add, -1 for remove, 0 for none
        self.is_covered = False  # Whether this range is fully covered


class DynamicSegmentTree:
    """Dynamic segment tree for range operations."""
    __slots__ = ['root']
  
    def __init__(self):
        self.root = SegmentTreeNode()
  
    def modify(self, query_left: int, query_right: int, operation: int, 
               tree_left: int = 1, tree_right: int = 10**9, node: SegmentTreeNode = None) -> None:
        """
        Modify a range in the segment tree.
      
        Args:
            query_left: Left boundary of the query range
            query_right: Right boundary of the query range
            operation: 1 for add operation, -1 for remove operation
            tree_left: Left boundary of current tree node's range
            tree_right: Right boundary of current tree node's range
            node: Current tree node
        """
        if node is None:
            node = self.root
      
        # If current range is completely within query range
        if tree_left >= query_left and tree_right <= query_right:
            if operation == 1:
                # Mark range as covered
                node.lazy_tag = 1
                node.is_covered = True
            else:
                # Mark range as not covered
                node.lazy_tag = -1
                node.is_covered = False
            return
      
        # Push down lazy tag before going to children
        self._push_down(node)
      
        # Recursively update children
        mid = (tree_left + tree_right) >> 1
        if query_left <= mid:
            self.modify(query_left, query_right, operation, tree_left, mid, node.left)
        if query_right > mid:
            self.modify(query_left, query_right, operation, mid + 1, tree_right, node.right)
      
        # Update current node based on children
        self._push_up(node)
  
    def query(self, query_left: int, query_right: int, 
              tree_left: int = 1, tree_right: int = 10**9, node: SegmentTreeNode = None) -> bool:
        """
        Query whether a range is fully covered.
      
        Args:
            query_left: Left boundary of the query range
            query_right: Right boundary of the query range
            tree_left: Left boundary of current tree node's range
            tree_right: Right boundary of current tree node's range
            node: Current tree node
          
        Returns:
            True if the entire range is covered, False otherwise
        """
        if node is None:
            node = self.root
      
        # If current range is completely within query range
        if tree_left >= query_left and tree_right <= query_right:
            return node.is_covered
      
        # Push down lazy tag before querying children
        self._push_down(node)
      
        # Query children and combine results
        mid = (tree_left + tree_right) >> 1
        result = True
      
        if query_left <= mid:
            result = result and self.query(query_left, query_right, tree_left, mid, node.left)
        if query_right > mid:
            result = result and self.query(query_left, query_right, mid + 1, tree_right, node.right)
      
        return result
  
    def _push_up(self, node: SegmentTreeNode) -> None:
        """
        Update parent node based on children nodes.
      
        Args:
            node: Current node to update
        """
        # A node is covered only if both children exist and are covered
        node.is_covered = bool(node.left and node.left.is_covered and 
                              node.right and node.right.is_covered)
  
    def _push_down(self, node: SegmentTreeNode) -> None:
        """
        Push down lazy tag to children nodes.
      
        Args:
            node: Current node whose lazy tag needs to be pushed down
        """
        # Create children if they don't exist
        if node.left is None:
            node.left = SegmentTreeNode()
        if node.right is None:
            node.right = SegmentTreeNode()
      
        # Push down lazy tag if it exists
        if node.lazy_tag:
            # Apply lazy tag to children
            node.left.lazy_tag = node.right.lazy_tag = node.lazy_tag
            node.left.is_covered = (node.lazy_tag == 1)
            node.right.is_covered = (node.lazy_tag == 1)
            # Clear current node's lazy tag
            node.lazy_tag = 0


class RangeModule:
    """
    Module for managing ranges with add, query, and remove operations.
    """
  
    def __init__(self):
        self.tree = DynamicSegmentTree()
  
    def addRange(self, left: int, right: int) -> None:
        """
        Add a range [left, right) to the module.
      
        Args:
            left: Start of the range (inclusive)
            right: End of the range (exclusive)
        """
        # Convert to inclusive range [left, right-1]
        self.tree.modify(left, right - 1, 1)
  
    def queryRange(self, left: int, right: int) -> bool:
        """
        Check if the range [left, right) is fully covered.
      
        Args:
            left: Start of the range (inclusive)
            right: End of the range (exclusive)
          
        Returns:
            True if the entire range is covered, False otherwise
        """
        # Convert to inclusive range [left, right-1]
        return self.tree.query(left, right - 1)
  
    def removeRange(self, left: int, right: int) -> None:
        """
        Remove a range [left, right) from the module.
      
        Args:
            left: Start of the range (inclusive)
            right: End of the range (exclusive)
        """
        # Convert to inclusive range [left, right-1]
        self.tree.modify(left, right - 1, -1)


# Your RangeModule object will be instantiated and called as such:
# obj = RangeModule()
# obj.addRange(left,right)
# param_2 = obj.queryRange(left,right)
# obj.removeRange(left,right)
```

## Time and Space Complexity

**Time Complexity:** Each operation (`addRange`, `queryRange`, `removeRange`) has a time complexity of `O(log n)`, where `n` is the data range (in this case `10^9`).

The segment tree operations work by recursively dividing the range in half at each level. Since the tree height is bounded by `log n`, and each operation traverses at most two paths from root to leaf (due to the range potentially spanning across the middle point), the time complexity remains `O(log n)` per operation.

**Space Complexity:** The space complexity is `O(m × log n)`, where `m` is the number of operations and `n` is the data range.

This implementation uses a dynamic segment tree with lazy node creation. Nodes are only created when needed during operations. In the worst case, each operation could create up to `O(log n)` new nodes along the path from root to the affected leaves. With `m` operations, the total number of nodes created can be at most `O(m × log n)`.

The `__slots__` optimization in the Node class helps reduce memory overhead per node, but doesn't change the asymptotic space complexity. Each node stores a constant amount of data (left pointer, right pointer, `lazy_tag`, and `is_covered`), so the space per node is `O(1)`, making the total space complexity `O(m × log n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Half-Open Intervals**

**Pitfall**: The most common mistake is forgetting to convert half-open intervals `[left, right)` to closed intervals `[left, right-1]` when working with the segment tree. This leads to off-by-one errors.

**Example of the bug**:
```python
def addRange(self, left: int, right: int) -> None:
    # WRONG: Passing right directly instead of right-1
    self.tree.modify(left, right, 1)  # Bug!
```

**Why it happens**: The problem uses half-open intervals (excludes the right boundary), but segment trees typically work with closed intervals. If you pass `right` directly, you're including one extra element.

**Solution**: Always subtract 1 from the right boundary:
```python
def addRange(self, left: int, right: int) -> None:
    # CORRECT: Convert [left, right) to [left, right-1]
    self.tree.modify(left, right - 1, 1)
```

### 2. **Forgetting to Push Down Lazy Tags Before Querying Children**

**Pitfall**: Not calling `push_down` before recursively querying or modifying children leads to stale data being used.

**Example of the bug**:
```python
def query(self, query_left, query_right, tree_left, tree_right, node):
    if tree_left >= query_left and tree_right <= query_right:
        return node.is_covered
  
    # WRONG: Forgot to push down before querying children
    mid = (tree_left + tree_right) >> 1
    result = True
    if query_left <= mid:
        result = result and self.query(query_left, query_right, tree_left, mid, node.left)
    # ... rest of code
```

**Why it happens**: Lazy propagation defers updates. If you don't push down pending updates before accessing children, the children will have outdated information.

**Solution**: Always push down before accessing children:
```python
def query(self, query_left, query_right, tree_left, tree_right, node):
    if tree_left >= query_left and tree_right <= query_right:
        return node.is_covered
  
    # CORRECT: Push down lazy tags first
    self._push_down(node)
  
    mid = (tree_left + tree_right) >> 1
    # ... rest of code
```

### 3. **Incorrect Push-Up Logic**

**Pitfall**: Setting `node.is_covered = node.left.is_covered or node.right.is_covered` instead of using `and`.

**Example of the bug**:
```python
def _push_up(self, node):
    # WRONG: Using OR instead of AND
    node.is_covered = (node.left and node.left.is_covered) or \
                     (node.right and node.right.is_covered)
```

**Why it happens**: A range is only fully covered if ALL of it is covered. Using `or` would incorrectly mark a parent as covered when only part of its range is covered.

**Solution**: Use `and` to ensure both children are covered:
```python
def _push_up(self, node):
    # CORRECT: Both children must exist and be covered
    node.is_covered = bool(node.left and node.left.is_covered and 
                          node.right and node.right.is_covered)
```

### 4. **Not Handling Node Creation in Push-Down**

**Pitfall**: Accessing `node.left` or `node.right` without checking if they exist first, causing `AttributeError`.

**Example of the bug**:
```python
def _push_down(self, node):
    # WRONG: Directly accessing children without checking existence
    if node.lazy_tag:
        node.left.lazy_tag = node.lazy_tag  # Crash if node.left is None!
        node.right.lazy_tag = node.lazy_tag
```

**Solution**: Create children nodes if they don't exist:
```python
def _push_down(self, node):
    # CORRECT: Create children first if needed
    if node.left is None:
        node.left = SegmentTreeNode()
    if node.right is None:
        node.right = SegmentTreeNode()
  
    if node.lazy_tag:
        node.left.lazy_tag = node.lazy_tag
        node.right.lazy_tag = node.lazy_tag
        # ... rest of code
```

### 5. **Forgetting to Clear Lazy Tag After Pushing Down**

**Pitfall**: Not resetting `node.lazy_tag = 0` after propagating it to children, causing the same update to be applied multiple times.

**Example of the bug**:
```python
def _push_down(self, node):
    if node.lazy_tag:
        node.left.lazy_tag = node.lazy_tag
        node.right.lazy_tag = node.lazy_tag
        # WRONG: Forgot to clear the lazy tag!
        # node.lazy_tag = 0  # Missing line
```

**Solution**: Always clear the lazy tag after propagation:
```python
def _push_down(self, node):
    if node.lazy_tag:
        # ... propagate to children ...
        node.lazy_tag = 0  # CORRECT: Clear after propagation
```

