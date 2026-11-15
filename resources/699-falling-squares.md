# 699. Falling Squares

> Source: https://algo.monster/liteproblems/699

## Problem Statement

You are simulating squares falling onto a 2D plane along the X-axis. When a square is dropped, it falls straight down until it either lands on top of another square or reaches the ground (X-axis).

You're given a 2D array `positions` where each `positions[i] = [left_i, sideLength_i]` represents:
- `left_i`: The X-coordinate where the left edge of the square is positioned
- `sideLength_i`: The side length of the square

The squares are dropped one at a time in the order given. Each square:
1. Falls vertically downward (in the negative Y direction)
2. Stops when it lands on the top of another square or on the X-axis
3. Freezes in place once it lands (cannot move anymore)
4. Does NOT count as landing if it only brushes the sides of another square

After each square is dropped, you need to record the maximum height among all stacks of squares at that moment.

Your task is to return an array `ans` where `ans[i]` represents the tallest stack height after dropping the `i-th` square.

For example, if you drop a square with side length 2 at position 1, it lands on the X-axis with height 2. If you then drop another square with side length 3 at position 2 (which overlaps with the first square from position 2 to 3), it will land on top of the first square, resulting in a height of 2 + 3 = 5 at that position.

## Intuition

When a square falls, we need to find the maximum height of all existing squares within its landing range `[left, left + sideLength - 1]`. The new square will land on top of the highest point in this range, so its final height will be `max_height_in_range + sideLength`.

After placing the square, we need to update the height for the entire range it covers to this new height, since any future square landing in this range needs to know about this new obstacle.

This problem essentially requires us to:
1. **Query** the maximum height in a given range `[l, r]`
2. **Update** all positions in a range `[l, r]` to a new height value
3. Track the overall maximum height after each operation

These operations naturally lead us to think about data structures that efficiently handle range queries and range updates. A naive approach would be to maintain an array of heights for each position, but with coordinates up to `10^8`, this becomes impractical.

A [Segment Tree](/problems/segment_tree_intro) is perfect for this scenario because:
- It can query the maximum value in any range in `O(log n)` time
- It can update all values in a range to a new value in `O(log n)` time using lazy propagation
- With dynamic node creation, we only create nodes as needed, avoiding the memory issue of large coordinate ranges

The lazy propagation technique is crucial here - when we update a range to a new height, we don't immediately update all child nodes. Instead, we mark the node with an `add` flag and only push down the update when we need to access the children later. This keeps our updates efficient even for large ranges.

By maintaining the running maximum height (`mx`) as we process each square, we can easily return the tallest stack height after each drop.

**Learn more about [Segment Tree](/problems/segment_tree_intro) patterns.**

## Solution Approach

The solution uses a dynamic [Segment Tree](/problems/segment_tree_intro) with lazy propagation to efficiently handle range queries and updates.

### [Segment Tree](/problems/segment_tree_intro) Structure

Each node in the [segment tree](/problems/segment_tree_intro) contains:
- `l, r`: The left and right boundaries of the interval this node represents
- `mid`: The midpoint `(l + r) >> 1` for splitting the interval
- `v`: The maximum height value in this interval
- `add`: The lazy propagation marker for pending updates
- `left, right`: Pointers to child nodes (created dynamically)

The tree is initialized with root covering the range `[1, 10^9]` to handle all possible coordinates.

### Key Operations

**1. Query Operation (`query(l, r)`)**
- Returns the maximum height in the range `[l, r]`
- If the current node's interval is completely within `[l, r]`, return its value `v`
- Otherwise, recursively query relevant child nodes and return the maximum
- Before accessing children, apply any pending updates via `pushdown()`

**2. Modify Operation (`modify(l, r, v)`)**
- Updates all positions in range `[l, r]` to height `v`
- If the current node's interval is completely within `[l, r]`, update its value and mark with lazy flag
- Otherwise, recursively modify relevant child nodes
- After modifying children, update parent's value via `pushup()`

**3. Lazy Propagation (`pushdown()`)**
- Creates child nodes dynamically if they don't exist
- If there's a pending update (`add != 0`), propagate it to both children
- Sets children's values and lazy markers to the parent's `add` value
- Clears the parent's lazy marker

**4. Value Update (`pushup()`)**
- After modifying children, updates parent's value to be the maximum of its children's values
- Ensures the tree maintains correct maximum values after modifications

### Main Algorithm

For each square `[l, w]` in positions:
1. Calculate the right boundary: `r = l + w - 1`
2. Query the maximum height in range `[l, r]`: `h_base = tree.query(l, r)`
3. Calculate new height for this square: `h = h_base + w`
4. Update the running maximum: `mx = max(mx, h)`
5. Record the current maximum in the answer array
6. Update the [segment tree](/problems/segment_tree_intro): set all positions in `[l, r]` to height `h`

The time complexity is `O(n log C)` where `n` is the number of squares and `C` is the coordinate range (`10^9`). The space complexity is `O(n log C)` for the dynamically created nodes.

## Example Walkthrough

Let's walk through a small example with `positions = [[1, 2], [2, 3], [6, 1]]`.

**Initial Setup:**
- Create a segment tree with root covering range `[1, 10^9]`
- Initialize `mx = 0` (running maximum height)
- Initialize empty result array `ans = []`

**Drop Square 1: [1, 2]** (left=1, width=2)
- Right boundary: `r = 1 + 2 - 1 = 2`, so square covers `[1, 2]`
- Query maximum height in range `[1, 2]`: 
  - Tree is empty, returns `0`
- New height: `h = 0 + 2 = 2`
- Update maximum: `mx = max(0, 2) = 2`
- Add to result: `ans = [2]`
- Update tree: Set range `[1, 2]` to height `2`
  - Tree now has: positions 1-2 with height 2

**Drop Square 2: [2, 3]** (left=2, width=3)
- Right boundary: `r = 2 + 3 - 1 = 4`, so square covers `[2, 4]`
- Query maximum height in range `[2, 4]`:
  - Position 2 has height 2 (from square 1)
  - Positions 3-4 have height 0
  - Maximum in range = `2`
- New height: `h = 2 + 3 = 5` (lands on top of square 1)
- Update maximum: `mx = max(2, 5) = 5`
- Add to result: `ans = [2, 5]`
- Update tree: Set range `[2, 4]` to height `5`
  - Tree now has: position 1 with height 2, positions 2-4 with height 5

**Drop Square 3: [6, 1]** (left=6, width=1)
- Right boundary: `r = 6 + 1 - 1 = 6`, so square covers `[6, 6]`
- Query maximum height in range `[6, 6]`:
  - Position 6 has height 0 (no squares here)
  - Maximum in range = `0`
- New height: `h = 0 + 1 = 1` (lands on ground)
- Update maximum: `mx = max(5, 1) = 5` (remains 5)
- Add to result: `ans = [2, 5, 5]`
- Update tree: Set range `[6, 6]` to height `1`
  - Tree now has: position 1 with height 2, positions 2-4 with height 5, position 6 with height 1

**Final Result:** `[2, 5, 5]`

The segment tree efficiently handles these operations:
- When querying range `[2, 4]` in step 2, it finds the overlap with existing square at position 2
- Lazy propagation ensures we don't update every single position individually
- Dynamic node creation means we only create nodes for positions we actually use (1-6) rather than the entire range `[1, 10^9]`

## Python Solution

```python
from typing import List


class Node:
    """Node for dynamic segment tree."""
  
    def __init__(self, left: int, right: int):
        self.left_child = None  # Left child node
        self.right_child = None  # Right child node
        self.left_bound = left  # Left boundary of the interval
        self.right_bound = right  # Right boundary of the interval
        self.mid = (left + right) >> 1  # Midpoint of the interval
        self.max_value = 0  # Maximum value in this interval
        self.lazy_tag = 0  # Lazy propagation tag for range updates


class SegmentTree:
    """Dynamic segment tree for range maximum query and range update."""
  
    def __init__(self):
        # Initialize root node covering range [1, 10^9]
        self.root = Node(1, int(1e9))
  
    def modify(self, left: int, right: int, value: int, node: Node = None) -> None:
        """
        Update all elements in range [left, right] to value.
      
        Args:
            left: Left boundary of update range
            right: Right boundary of update range
            value: Value to set for the range
            node: Current node in recursion (defaults to root)
        """
        if left > right:
            return
      
        if node is None:
            node = self.root
      
        # If current node's interval is completely within update range
        if node.left_bound >= left and node.right_bound <= right:
            node.max_value = value
            node.lazy_tag = value
            return
      
        # Push down lazy tag before going to children
        self._push_down(node)
      
        # Recursively update left child if needed
        if left <= node.mid:
            self.modify(left, right, value, node.left_child)
      
        # Recursively update right child if needed
        if right > node.mid:
            self.modify(left, right, value, node.right_child)
      
        # Update current node's value based on children
        self._push_up(node)
  
    def query(self, left: int, right: int, node: Node = None) -> int:
        """
        Query maximum value in range [left, right].
      
        Args:
            left: Left boundary of query range
            right: Right boundary of query range
            node: Current node in recursion (defaults to root)
      
        Returns:
            Maximum value in the specified range
        """
        if left > right:
            return 0
      
        if node is None:
            node = self.root
      
        # If current node's interval is completely within query range
        if node.left_bound >= left and node.right_bound <= right:
            return node.max_value
      
        # Push down lazy tag before querying children
        self._push_down(node)
      
        max_val = 0
      
        # Query left child if needed
        if left <= node.mid:
            max_val = max(max_val, self.query(left, right, node.left_child))
      
        # Query right child if needed
        if right > node.mid:
            max_val = max(max_val, self.query(left, right, node.right_child))
      
        return max_val
  
    def _push_up(self, node: Node) -> None:
        """
        Update parent node's value based on children's values.
      
        Args:
            node: Node to update
        """
        node.max_value = max(node.left_child.max_value, node.right_child.max_value)
  
    def _push_down(self, node: Node) -> None:
        """
        Push lazy tag down to children and create children if they don't exist.
      
        Args:
            node: Node whose lazy tag needs to be pushed down
        """
        # Create left child if it doesn't exist
        if node.left_child is None:
            node.left_child = Node(node.left_bound, node.mid)
      
        # Create right child if it doesn't exist
        if node.right_child is None:
            node.right_child = Node(node.mid + 1, node.right_bound)
      
        # If there's a lazy tag, propagate it to children
        if node.lazy_tag:
            node.left_child.max_value = node.lazy_tag
            node.right_child.max_value = node.lazy_tag
            node.left_child.lazy_tag = node.lazy_tag
            node.right_child.lazy_tag = node.lazy_tag
            node.lazy_tag = 0  # Clear the lazy tag after propagation


class Solution:
    def fallingSquares(self, positions: List[List[int]]) -> List[int]:
        """
        Simulate falling squares and return maximum height after each square falls.
      
        Args:
            positions: List of [left, side_length] pairs representing squares
      
        Returns:
            List of maximum heights after each square falls
        """
        result = []
        max_height = 0
        tree = SegmentTree()
      
        for left_pos, side_length in positions:
            # Calculate right boundary of the square
            right_pos = left_pos + side_length - 1
          
            # Find current maximum height in the range where square will land
            current_height = tree.query(left_pos, right_pos)
          
            # New height after this square lands
            new_height = current_height + side_length
          
            # Update global maximum height
            max_height = max(max_height, new_height)
            result.append(max_height)
          
            # Update the height in the range covered by this square
            tree.modify(left_pos, right_pos, new_height)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n log C)` where `n` is the number of falling squares and `C` is the coordinate range (10^9 in this case).

For each of the `n` positions:
- The `query` operation traverses the segment tree from root to leaf, creating nodes dynamically as needed. In the worst case, this takes `O(log C)` time since the tree height is bounded by `log(10^9) ≈ 30`.
- The `modify` operation similarly traverses the tree with `O(log C)` time complexity.

Since we perform both operations for each position, the total time complexity is `O(n × 2 log C) = O(n log C)`.

Note: For this dynamic segment tree, the depth depends on the coordinate range `C` rather than the number of squares `n`, since the tree spans coordinates from 1 to 10^9. A segment tree built over compressed coordinates (at most `2n` distinct endpoints) would run in `O(n log n)` time.

**Space Complexity:** `O(n log C)` in the worst case.

The segment tree uses dynamic node creation (lazy initialization). Nodes are only created when accessed during `query` or `modify` operations. For each position, we might create up to `O(log C)` nodes along the path from root to the relevant leaves. With `n` positions, the worst-case space complexity is `O(n log C)`.

In practice, the space usage is often much less than the worst case because:
- Nodes are shared between operations when they access overlapping ranges
- The tree only materializes the portions that are actually accessed
- Adjacent or overlapping squares may reuse existing nodes

A segment tree built over compressed coordinates, where the endpoints are mapped to a range of size `O(n)`, would use `O(n)` space instead.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Range Calculation for Square Boundaries**

A common mistake is miscalculating the right boundary of a square. Given a square at position `left` with side length `sideLength`, developers often incorrectly calculate:
- **Wrong**: `right = left + sideLength` 
- **Correct**: `right = left + sideLength - 1`

This is because if a square starts at position `left` and has width `sideLength`, it occupies positions `[left, left + sideLength - 1]` inclusive.

**Example**: A square at position 2 with side length 3 occupies positions [2, 3, 4], not [2, 3, 4, 5].

### 2. **Not Handling Dynamic Node Creation Properly**

When implementing a dynamic segment tree, forgetting to create child nodes before accessing them leads to `NullPointerException` or `AttributeError`. The `pushdown()` method must always create child nodes if they don't exist before propagating lazy values.

**Problem Code:**
```python
def _push_down(self, node: Node) -> None:
    # Wrong: Directly accessing children without checking existence
    if node.lazy_tag:
        node.left_child.max_value = node.lazy_tag  # Error if left_child is None
        node.right_child.max_value = node.lazy_tag
```

**Solution:** Always create children first in `pushdown()`.

### 3. **Forgetting to Clear Lazy Tag After Propagation**

After pushing down a lazy tag to children, failing to clear the parent's lazy tag causes incorrect results in subsequent operations.

**Problem Code:**
```python
def _push_down(self, node: Node) -> None:
    if node.lazy_tag:
        node.left_child.lazy_tag = node.lazy_tag
        node.right_child.lazy_tag = node.lazy_tag
        # Forgot: node.lazy_tag = 0
```

This causes the same update to be applied multiple times.

### 4. **Integer Overflow with Large Coordinates**

The problem allows coordinates up to 10^9. Using incorrect data types or operations might cause overflow issues. While Python handles large integers automatically, in other languages like Java or C++, you need to be careful with intermediate calculations.

### 5. **Incorrect Boundary Conditions in Recursive Calls**

When deciding whether to recurse into left or right children, using wrong boundary comparisons causes missed updates or queries.

**Problem Code:**
```python
# Wrong comparison operators
if left < node.mid:  # Should be <=
    self.modify(left, right, value, node.left_child)
if right >= node.mid:  # Should be >
    self.modify(left, right, value, node.right_child)
```

The correct conditions are:
- Recurse left if: `left <= node.mid`
- Recurse right if: `right > node.mid`

### 6. **Not Maintaining Running Maximum Correctly**

Some implementations forget to maintain a running maximum and instead just return the height of each individual square.

**Problem Code:**
```python
for left_pos, side_length in positions:
    # ... calculate new_height
    result.append(new_height)  # Wrong: should append max(max_height, new_height)
```

The problem asks for the maximum height among ALL stacks after each drop, not just the height of the current stack.

### 7. **Edge Case: Squares Landing Side-by-Side**

Ensure that squares that only touch sides (no overlap) don't affect each other's landing height. A square at `[1, 2]` (covering positions 1-2) and another at `[3, 2]` (covering positions 3-4) should land independently on the ground if dropped in that order.

**Testing Tip:** Always test with cases like:
- `[[1, 2], [3, 2]]` - Should return `[2, 2]` (both land on ground)
- `[[1, 2], [2, 3]]` - Should return `[2, 5]` (second lands on first due to overlap at position 2)

