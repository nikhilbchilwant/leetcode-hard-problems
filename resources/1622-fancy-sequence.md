# 1622. Fancy Sequence

> Source: https://algo.monster/liteproblems/1622

## Problem Statement

You need to implement a class called `Fancy` that maintains a sequence of integers and supports four operations:

1. **`Fancy()`**: Creates a new empty sequence.

2. **`append(val)`**: Adds an integer `val` to the end of the sequence.

3. **`addAll(inc)`**: Adds an integer `inc` to every existing element in the sequence. This operation only affects elements that were already in the sequence before this call.

4. **`multAll(m)`**: Multiplies every existing element in the sequence by an integer `m`. This operation only affects elements that were already in the sequence before this call.

5. **`getIndex(idx)`**: Returns the value at position `idx` (using 0-based indexing) in the current sequence, modulo `10^9 + 7`. If `idx` is greater than or equal to the length of the sequence, return `-1`.

The key challenge is efficiently handling the `addAll` and `multAll` operations. A naive approach of iterating through all elements for each operation would be too slow for large sequences. The solution uses a [Segment Tree](/problems/segment_tree_intro) with lazy propagation to efficiently apply range updates (addition and multiplication) to the entire sequence.

The [Segment Tree](/problems/segment_tree_intro) maintains lazy propagation tags for both addition and multiplication operations. When applying operations:
- For addition: the new value becomes `(old_value + inc)`
- For multiplication: the new value becomes `(old_value * m)` and any pending additions also get multiplied

Each element is stored at a position in the [segment tree](/problems/segment_tree_intro) (1-indexed internally), and operations are applied to ranges. The `append` operation adds a new element at position `n+1`, while `addAll` and `multAll` apply their operations to the range `[1, n]` where `n` is the current sequence length.

All arithmetic operations are performed modulo `10^9 + 7` to prevent integer overflow.

## Intuition

The first instinct might be to simply store all values in an array and update them directly when `addAll` or `multAll` is called. However, this becomes inefficient when we have many elements and frequent update operations - each `addAll` or `multAll` would require O(n) time to update all elements.

The key insight is recognizing that `addAll` and `multAll` are **range update operations** that affect all existing elements uniformly. Instead of updating each element immediately, we can defer these operations and apply them lazily when needed.

Think of it like this: if we have a sequence `[a, b, c]` and we perform `addAll(5)` followed by `multAll(2)`, instead of immediately computing `[2*(a+5), 2*(b+5), 2*(c+5)]`, we can store the operations themselves and apply them only when we need to retrieve a specific value.

This naturally leads us to consider data structures that excel at range updates with lazy propagation. A [Segment Tree](/problems/segment_tree_intro) is perfect for this because:

1. **Range Updates**: We can efficiently mark an entire range (in our case, all existing elements) with pending operations in O(log n) time.

2. **Lazy Propagation**: We don't need to immediately apply operations to every element. Instead, we store "lazy" tags at nodes representing the pending operations. These get pushed down the tree only when necessary.

3. **Operation Composition**: When multiple operations stack up (like `add` followed by `multiply`), we need to compose them correctly. The formula becomes: `new_value = old_value * mul + add`. The multiplication affects both the original value and any previous additions.

4. **Modular Arithmetic**: Since we need results modulo `10^9 + 7`, we can apply the modulo operation at each step without affecting correctness.

The [segment tree](/problems/segment_tree_intro) approach transforms our O(n) update operations into O(log n) operations, making the solution efficient even for large sequences with many operations.

**Learn more about [Segment Tree](/problems/segment_tree_intro) and [Math](/problems/math-basics) patterns.**

## Solution Approach

The solution implements a [Segment Tree](/problems/segment_tree_intro) with lazy propagation to efficiently handle range updates. Let's walk through the key components:

### Data Structures

**Node Class**: Each node in the [segment tree](/problems/segment_tree_intro) contains:
- `l, r`: The range `[l, r]` this node represents
- `mid`: The midpoint for splitting into child nodes
- `v`: The sum of all values in this range
- `add`: Lazy tag for pending addition operations
- `mul`: Lazy tag for pending multiplication operations (initialized to 1)

**SegmentTree Class**: Manages the tree operations with a root node covering range `[1, 100001]`.

### Core Operations

**1. Lazy Propagation (`pushdown`)**

When we need to access child nodes, we first push down any pending operations:
```
new_value = old_value * mul + (range_size * add)
new_add = old_add * mul + add
new_mul = old_mul * mul
```

This ensures operations are applied in the correct order: previous operations get scaled by new multiplications.

**2. Range Addition (`modifyAdd`)**

To add `inc` to all elements in range `[l, r]`:
- If current node is completely within range: update the node's value and lazy `add` tag
- Otherwise: push down lazy tags, recursively update children, then update current node's value

**3. Range Multiplication (`modifyMul`)**

To multiply all elements in range `[l, r]` by `m`:
- If current node is completely within range: multiply the node's value, `add` tag, and `mul` tag by `m`
- Otherwise: push down lazy tags, recursively update children, then update current node's value

**4. Point Query (`query`)**

To get value at a specific position:
- Navigate down the tree, pushing down lazy tags along the path
- Return the value when we reach the target range

### Fancy Class Implementation

The `Fancy` class wraps the [segment tree](/problems/segment_tree_intro):

**`append(val)`**: 
- Increment counter `n` 
- Add `val` at position `n` using `modifyAdd(n, n, val)`

**`addAll(inc)`**: 
- Apply addition to range `[1, n]` using `modifyAdd(1, n, inc)`

**`multAll(m)`**: 
- Apply multiplication to range `[1, n]` using `modifyMul(1, n, m)`

**`getIndex(idx)`**: 
- Return `-1` if `idx >= n`
- Otherwise query position `idx + 1` (converting 0-indexed to 1-indexed)

### Time Complexity

- `append`: O(log n) - single point update
- `addAll`: O(log n) - range update with lazy propagation
- `multAll`: O(log n) - range update with lazy propagation
- `getIndex`: O(log n) - point query

The [segment tree](/problems/segment_tree_intro) approach ensures all operations remain efficient even with a large number of elements, as we avoid the O(n) cost of updating every element directly.

## Example Walkthrough

Let's walk through a concrete example to see how the segment tree with lazy propagation handles the operations:

**Initial State**: Empty sequence, segment tree initialized with all zeros.

**Step 1: `append(2)`**
- Increment n from 0 to 1
- Call `modifyAdd(1, 1, 2)` to add value 2 at position 1
- The tree node representing position 1 gets value 2
- Sequence: `[2]`

**Step 2: `append(3)`**
- Increment n from 1 to 2  
- Call `modifyAdd(2, 2, 3)` to add value 3 at position 2
- The tree node representing position 2 gets value 3
- Sequence: `[2, 3]`

**Step 3: `append(7)`**
- Increment n from 2 to 3
- Call `modifyAdd(3, 3, 7)` to add value 7 at position 3
- Sequence: `[2, 3, 7]`

**Step 4: `multAll(2)`**
- Call `modifyMul(1, 3, 2)` to multiply all elements in range [1, 3] by 2
- Instead of updating each element individually, we update the lazy `mul` tag at the root node covering [1, 3]
- The lazy tag says "multiply everything in this range by 2"
- Actual values aren't computed yet - they're still logically `[2, 3, 7]` with a pending ×2 operation
- Logical sequence: `[4, 6, 14]`

**Step 5: `addAll(5)`**
- Call `modifyAdd(1, 3, 5)` to add 5 to all elements in range [1, 3]
- The root node now has both multiplication (×2) and addition (+5) lazy tags
- These compose as: `new_value = old_value * 2 + 5`
- Logical sequence: `[9, 11, 19]`

**Step 6: `getIndex(0)`**
- Convert 0-indexed to 1-indexed: query position 1
- As we traverse down to position 1, lazy tags get pushed down:
  - The pending operations (×2, then +5) are applied
  - Position 1: `2 * 2 + 5 = 9`
- Return 9

**Step 7: `append(10)`**
- Increment n from 3 to 4
- Call `modifyAdd(4, 4, 10)` to add value 10 at position 4
- This new element is NOT affected by previous `multAll` or `addAll` operations
- Sequence: `[9, 11, 19, 10]`

**Step 8: `multAll(3)`**
- Call `modifyMul(1, 4, 3)` to multiply all current elements by 3
- For positions 1-3: Previous operations (×2, +5) now become (×6, +15) after multiplication by 3
- Position 4: Gets ×3 operation
- Logical sequence: `[27, 33, 57, 30]`

**Step 9: `getIndex(2)`**
- Query position 3 (converting from 0-indexed)
- Original value at position 3 was 7
- Apply composed operations: `7 * 6 + 15 = 57`
- Return 57

This example demonstrates the key benefits:
1. **Lazy Updates**: Operations are stored as tags rather than immediately applied
2. **Operation Composition**: Multiple operations stack correctly (multiply affects both values and previous additions)  
3. **Selective Application**: New elements added after operations aren't affected by previous operations
4. **Efficient Queries**: Values are computed on-demand by applying accumulated operations

## Python Solution

```python
MOD = int(1e9 + 7)


class SegmentTreeNode:
    """
    Node for the segment tree that represents a range [left, right].
    Supports lazy propagation for addition and multiplication operations.
    """
    def __init__(self, left: int, right: int):
        self.left_child = None
        self.right_child = None
        self.left_bound = left  # Left boundary of the range
        self.right_bound = right  # Right boundary of the range
        self.mid_point = (left + right) >> 1  # Midpoint for splitting
        self.sum_value = 0  # Sum of all elements in this range
        self.lazy_add = 0  # Lazy propagation value for addition
        self.lazy_multiply = 1  # Lazy propagation value for multiplication


class SegmentTree:
    """
    Segment tree with lazy propagation supporting range addition and multiplication.
    """
    def __init__(self):
        # Initialize root node covering range [1, 100001]
        self.root = SegmentTreeNode(1, int(1e5 + 1))

    def modifyAdd(self, left: int, right: int, increment: int, node: SegmentTreeNode = None) -> None:
        """
        Add 'increment' to all elements in range [left, right].
      
        Args:
            left: Left boundary of the range (inclusive)
            right: Right boundary of the range (inclusive)
            increment: Value to add to each element
            node: Current node in the recursion (defaults to root)
        """
        if left > right:
            return
          
        if node is None:
            node = self.root
          
        # If current node's range is completely within [left, right]
        if node.left_bound >= left and node.right_bound <= right:
            # Update the sum for this range
            range_size = node.right_bound - node.left_bound + 1
            node.sum_value = (node.sum_value + range_size * increment) % MOD
            # Mark for lazy propagation
            node.lazy_add += increment
            return
          
        # Push down lazy values before processing children
        self._push_down(node)
      
        # Recursively update left and right children if they overlap with [left, right]
        if left <= node.mid_point:
            self.modifyAdd(left, right, increment, node.left_child)
        if right > node.mid_point:
            self.modifyAdd(left, right, increment, node.right_child)
          
        # Update current node's value based on children
        self._push_up(node)

    def modifyMul(self, left: int, right: int, multiplier: int, node: SegmentTreeNode = None) -> None:
        """
        Multiply all elements in range [left, right] by 'multiplier'.
      
        Args:
            left: Left boundary of the range (inclusive)
            right: Right boundary of the range (inclusive)
            multiplier: Value to multiply each element by
            node: Current node in the recursion (defaults to root)
        """
        if left > right:
            return
          
        if node is None:
            node = self.root
          
        # If current node's range is completely within [left, right]
        if node.left_bound >= left and node.right_bound <= right:
            # Apply multiplication to sum and lazy values
            node.sum_value = (node.sum_value * multiplier) % MOD
            node.lazy_add = (node.lazy_add * multiplier) % MOD
            node.lazy_multiply = (node.lazy_multiply * multiplier) % MOD
            return
          
        # Push down lazy values before processing children
        self._push_down(node)
      
        # Recursively update left and right children if they overlap with [left, right]
        if left <= node.mid_point:
            self.modifyMul(left, right, multiplier, node.left_child)
        if right > node.mid_point:
            self.modifyMul(left, right, multiplier, node.right_child)
          
        # Update current node's value based on children
        self._push_up(node)

    def query(self, left: int, right: int, node: SegmentTreeNode = None) -> int:
        """
        Query the sum of elements in range [left, right].
      
        Args:
            left: Left boundary of the range (inclusive)
            right: Right boundary of the range (inclusive)
            node: Current node in the recursion (defaults to root)
          
        Returns:
            Sum of elements in the specified range
        """
        if left > right:
            return 0
          
        if node is None:
            node = self.root
          
        # If current node's range is completely within [left, right]
        if node.left_bound >= left and node.right_bound <= right:
            return node.sum_value
          
        # Push down lazy values before querying children
        self._push_down(node)
      
        result = 0
        # Query left and right children if they overlap with [left, right]
        if left <= node.mid_point:
            result = (result + self.query(left, right, node.left_child)) % MOD
        if right > node.mid_point:
            result = (result + self.query(left, right, node.right_child)) % MOD
          
        return result

    def _push_up(self, node: SegmentTreeNode) -> None:
        """
        Update parent node's value based on its children's values.
      
        Args:
            node: Node to update
        """
        node.sum_value = (node.left_child.sum_value + node.right_child.sum_value) % MOD

    def _push_down(self, node: SegmentTreeNode) -> None:
        """
        Push lazy propagation values down to children nodes.
        Creates children if they don't exist.
      
        Args:
            node: Node whose lazy values need to be pushed down
        """
        # Create children nodes if they don't exist
        if node.left_child is None:
            node.left_child = SegmentTreeNode(node.left_bound, node.mid_point)
        if node.right_child is None:
            node.right_child = SegmentTreeNode(node.mid_point + 1, node.right_bound)
          
        left_child = node.left_child
        right_child = node.right_child
      
        # Apply lazy operations if any exist
        if node.lazy_add != 0 or node.lazy_multiply != 1:
            # Apply multiplication first, then addition (order matters!)
            # Formula: new_value = old_value * multiply + range_size * add
          
            # Update left child
            left_range_size = left_child.right_bound - left_child.left_bound + 1
            left_child.sum_value = (left_child.sum_value * node.lazy_multiply + 
                                   left_range_size * node.lazy_add) % MOD
            left_child.lazy_add = (left_child.lazy_add * node.lazy_multiply + node.lazy_add) % MOD
            left_child.lazy_multiply = (left_child.lazy_multiply * node.lazy_multiply) % MOD
          
            # Update right child
            right_range_size = right_child.right_bound - right_child.left_bound + 1
            right_child.sum_value = (right_child.sum_value * node.lazy_multiply + 
                                    right_range_size * node.lazy_add) % MOD
            right_child.lazy_add = (right_child.lazy_add * node.lazy_multiply + node.lazy_add) % MOD
            right_child.lazy_multiply = (right_child.lazy_multiply * node.lazy_multiply) % MOD
          
            # Clear parent's lazy values
            node.lazy_add = 0
            node.lazy_multiply = 1


class Fancy:
    """
    A data structure that supports:
    - Appending integers to the end
    - Adding a value to all existing elements
    - Multiplying all existing elements by a value
    - Getting the value at a specific index
    """
    def __init__(self):
        self.size = 0  # Current number of elements
        self.tree = SegmentTree()

    def append(self, val: int) -> None:
        """
        Append 'val' to the end of the sequence.
      
        Args:
            val: Value to append
        """
        self.size += 1
        # Add the value at position self.size (1-indexed)
        self.tree.modifyAdd(self.size, self.size, val)

    def addAll(self, inc: int) -> None:
        """
        Add 'inc' to all existing elements in the sequence.
      
        Args:
            inc: Value to add to all elements
        """
        self.tree.modifyAdd(1, self.size, inc)

    def multAll(self, m: int) -> None:
        """
        Multiply all existing elements in the sequence by 'm'.
      
        Args:
            m: Multiplier for all elements
        """
        self.tree.modifyMul(1, self.size, m)

    def getIndex(self, idx: int) -> int:
        """
        Get the value at index 'idx' (0-indexed).
      
        Args:
            idx: Index to query (0-indexed)
          
        Returns:
            Value at the given index, or -1 if index is out of bounds
        """
        if idx >= self.size:
            return -1
        # Convert to 1-indexed for the segment tree
        return self.tree.query(idx + 1, idx + 1)


# Your Fancy object will be instantiated and called as such:
# obj = Fancy()
# obj.append(val)
# obj.addAll(inc)
# obj.multAll(m)
# param_4 = obj.getIndex(idx)
```

## Time and Space Complexity

**Time Complexity:**
- `append(val)`: `O(log n)` - The operation calls `modifyAdd` on a single element, which traverses the segment tree from root to leaf. The tree height is `O(log n)` where `n` is the range size (10^5).
- `addAll(inc)`: `O(log n)` - The operation calls `modifyAdd` on the range `[1, n]`. Due to lazy propagation, if the entire range matches a node's range, the update is done in `O(1)` at that node. Otherwise, it traverses down the tree, taking `O(log n)` time.
- `multAll(m)`: `O(log n)` - Similar to `addAll`, it calls `modifyMul` on the range `[1, n]` with lazy propagation, resulting in `O(log n)` time complexity.
- `getIndex(idx)`: `O(log n)` - The operation calls `query` for a single element, which traverses from root to the target leaf node, taking `O(log n)` time.

**Space Complexity:**
- The segment tree uses dynamic node creation (nodes are created only when needed during `pushdown`).
- In the worst case, after multiple operations, the tree could have up to `O(n)` nodes where `n` is the maximum range (10^5).
- Each node stores a constant amount of data (`l`, `r`, `mid`, `v`, `add`, `mul`).
- The `Fancy` class itself only stores the tree and a counter `n`.
- Overall space complexity: `O(n)` where `n` is up to 10^5.

**Note:** The lazy propagation technique ensures that updates are deferred and only applied when necessary, keeping the time complexity logarithmic for range updates rather than linear.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Order of Operations in Lazy Propagation**

One of the most critical pitfalls in this solution is applying multiplication and addition operations in the wrong order during lazy propagation. When pushing down lazy values, the order matters significantly.

**The Problem:**
Consider a sequence where we have pending operations:
- First: multiply by 2
- Then: add 3

If we apply these in the wrong order (add first, then multiply), we get:
- Wrong: `(value + 3) * 2 = value * 2 + 6`
- Correct: `value * 2 + 3`

**The Solution:**
Always apply multiplication first, then addition. When combining operations:
```python
# Correct order in _push_down method:
new_value = old_value * lazy_multiply + range_size * lazy_add
new_lazy_add = old_lazy_add * lazy_multiply + lazy_add
new_lazy_mul = old_lazy_mul * lazy_multiply
```

### 2. **Forgetting to Handle Modulo Operations Consistently**

**The Problem:**
Integer overflow can occur during multiplication operations, and forgetting to apply modulo at any step can lead to incorrect results or runtime errors.

**The Solution:**
Apply modulo operation after every arithmetic operation:
```python
# Always use modulo after operations
node.sum_value = (node.sum_value * multiplier) % MOD
node.lazy_add = (node.lazy_add * multiplier) % MOD
node.lazy_multiply = (node.lazy_multiply * multiplier) % MOD
```

### 3. **Not Initializing Lazy Multiplication to 1**

**The Problem:**
If lazy multiplication is initialized to 0 instead of 1, all values will become 0 when the lazy tag is applied, destroying all data.

**The Solution:**
Always initialize `lazy_multiply = 1` in the node constructor:
```python
def __init__(self, left: int, right: int):
    # ... other initializations
    self.lazy_add = 0  # Addition identity is 0
    self.lazy_multiply = 1  # Multiplication identity is 1, NOT 0!
```

### 4. **Incorrect Range Size Calculation in Lazy Propagation**

**The Problem:**
When applying an addition operation lazily, the contribution to the sum depends on the range size. Forgetting to multiply the lazy addition value by the range size will give incorrect sum values.

**The Solution:**
Always multiply the lazy addition by the range size when updating the sum:
```python
range_size = node.right_bound - node.left_bound + 1
node.sum_value = (node.sum_value + range_size * increment) % MOD
```

### 5. **Index Conversion Errors (0-indexed vs 1-indexed)**

**The Problem:**
The Fancy class uses 0-indexed positions for the user interface, but the segment tree internally uses 1-indexed positions. Mixing these up causes off-by-one errors.

**The Solution:**
Consistently convert indices:
```python
def getIndex(self, idx: int) -> int:
    if idx >= self.size:  # Check using 0-indexed logic
        return -1
    # Convert to 1-indexed for segment tree query
    return self.tree.query(idx + 1, idx + 1)
```

### 6. **Not Creating Child Nodes Before Accessing Them**

**The Problem:**
In a dynamic segment tree, child nodes might not exist yet. Attempting to access them without checking can cause AttributeError.

**The Solution:**
Always create child nodes in the `_push_down` method before accessing them:
```python
def _push_down(self, node: SegmentTreeNode) -> None:
    if node.left_child is None:
        node.left_child = SegmentTreeNode(node.left_bound, node.mid_point)
    if node.right_child is None:
        node.right_child = SegmentTreeNode(node.mid_point + 1, node.right_bound)
```

