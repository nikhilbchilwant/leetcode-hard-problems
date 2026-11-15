# 2286. Booking Concert Tickets in Groups

> Source: https://algo.monster/liteproblems/2286

## Problem Statement

You need to design a ticketing system for a concert hall with `n` rows (numbered 0 to n-1) and `m` seats per row (numbered 0 to m-1).

The system must handle two types of booking requests:

**1. Group booking (gather):** A group of `k` people wants to sit together in the same row.
- They will only book if all `k` members can sit in consecutive seats in a single row
- The row number must be ≤ `maxRow`
- If multiple rows are available, choose the row with the smallest number
- Within a row, choose the leftmost available consecutive seats

**2. Scattered booking (scatter):** A group of `k` people can sit separately.
- They will only book if all `k` members can get seats (not necessarily together)
- All seats must be in rows numbered ≤ `maxRow`
- Allocate seats starting from the smallest row numbers
- Within each row, allocate from the smallest seat numbers

You need to implement a `BookMyShow` class with three methods:

- `BookMyShow(n, m)`: Initialize with `n` rows and `m` seats per row
- `gather(k, maxRow)`: Try to book `k` consecutive seats in a single row (row ≤ `maxRow`). Returns `[row, seat]` indicating the row number and starting seat number if successful, or empty array `[]` if not possible
- `scatter(k, maxRow)`: Try to book `k` seats total across rows 0 to `maxRow` (seats don't need to be together). Returns `true` if successful and allocates the seats, `false` otherwise

The key challenge is efficiently tracking available seats and finding optimal allocations according to the "smallest row first, smallest seat first" rule while handling potentially large numbers of rows and frequent booking operations.

## Intuition

The core challenge is efficiently tracking available seats across all rows and quickly finding rows that meet specific criteria. Let's think about what operations we need:

For `gather(k, maxRow)`, we need to find the smallest row number that has at least `k` consecutive empty seats. This means we need to quickly query "what's the maximum number of available seats in rows 0 to maxRow?" and "which is the first row with at least k available seats?"

For `scatter(k, maxRow)`, we need to check if the total available seats in rows 0 to `maxRow` is at least `k`, then greedily fill seats starting from the smallest row numbers.

If we use a simple array to track available seats per row, finding the first row with enough seats would take `O(n)` time for each query, which could be inefficient for large `n`.

The key insight is that we need two types of range queries:
1. **Range maximum query**: Find the row with the most available seats in a range (for `gather`)
2. **Range sum query**: Calculate total available seats in a range (for `scatter`)

Both operations also require **point updates** when we book seats in a specific row.

This pattern - range queries with point updates - is a classic use case for a **[Segment Tree](/problems/segment_tree_intro)**. A segment tree can handle both types of queries in `O(log n)` time.

In our [segment tree](/problems/segment_tree_intro), each node stores:
- `s`: sum of available seats in its range (for checking total availability)
- `mx`: maximum available seats in any single row within its range (for finding rows that can accommodate a group)

For `gather`, we use the [segment tree](/problems/segment_tree_intro) to find the leftmost row with `mx >= k`, then update that row's available seats.

For `scatter`, we first check if the sum of available seats is sufficient, then greedily allocate seats row by row, starting from the first row that has any available seats.

The [segment tree](/problems/segment_tree_intro) structure allows us to efficiently maintain and query this information as bookings are made, reducing query complexity from `O(n)` to `O(log n)`.

**Learn more about [Segment Tree](/problems/segment_tree_intro) and [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution uses a **[Segment Tree](/problems/segment_tree_intro)** data structure to efficiently manage seat availability across all rows. Let's walk through the implementation details:

### [Segment Tree](/problems/segment_tree_intro) Node Structure

Each node in the [segment tree](/problems/segment_tree_intro) contains:
- `l, r`: The left and right endpoints of the interval (row range) this node represents
- `s`: The total sum of available seats in this interval
- `mx`: The maximum available seats in any single row within this interval

### Building the [Segment Tree](/problems/segment_tree_intro)

The `build(u, l, r)` method constructs the tree recursively:
- For leaf nodes (when `l == r`), both `s` and `mx` are initialized to `m` (all seats available)
- For internal nodes, we recursively build left and right children
- The node spans are divided at `mid = (l + r) >> 1`
- After building children, we call `pushup(u)` to update the parent's values

### Core Operations

**1. Point Update - `modify(u, x, v)`**
- Updates row `x` to have `v` available seats
- Recursively traverses down to the leaf node representing row `x`
- Updates the leaf's `s` and `mx` values to `v`
- Calls `pushup` on the path back up to update ancestor nodes

**2. Range Sum Query - `query_sum(u, l, r)`**
- Calculates total available seats in rows `[l, r]`
- If current node's interval is completely within `[l, r]`, return its `s` value
- Otherwise, recursively query relevant children and sum the results

**3. Find First Row with k Seats - `query_idx(u, l, r, k)`**
- Finds the leftmost row in range `[l, r]` with at least `k` available seats
- If current node's `mx < k`, no row in this subtree has enough seats, return 0
- For leaf nodes, return the row number if it has enough seats
- Otherwise, check left child first (for leftmost row), then right child if needed

**4. Update Parent - `pushup(u)`**
- Updates node `u` based on its children
- `s = left_child.s + right_child.s` (sum of available seats)
- `mx = max(left_child.mx, right_child.mx)` (maximum available in any row)

### BookMyShow Implementation

**gather(k, maxRow):**
1. Convert `maxRow` to 1-indexed: `maxRow += 1`
2. Use `query_idx(1, 1, maxRow, k)` to find first row `i` with ≥ `k` seats
3. If no such row exists (`i == 0`), return empty array
4. Get current available seats in row `i`: `s = query_sum(1, i, i)`
5. Book `k` seats: `modify(1, i, s - k)`
6. Return `[i - 1, m - s]` (convert back to 0-indexed row, and seat starts at `m - s`)

**scatter(k, maxRow):**
1. Convert `maxRow` to 1-indexed: `maxRow += 1`
2. Check total availability: if `query_sum(1, 1, maxRow) < k`, return `false`
3. Find first row with any seats: `i = query_idx(1, 1, maxRow, 1)`
4. Starting from row `i`, greedily allocate seats:
   - Get available seats in current row: `s = query_sum(1, j, j)`
   - If `s >= k`, book `k` seats and return `true`
   - Otherwise, book all `s` seats, update `k -= s`, and continue to next row
5. Return `true` after allocating all seats

### Time Complexity

- **Initialization**: `O(n)` to build the [segment tree](/problems/segment_tree_intro)
- **gather**: `O(log n)` for one query_idx and one modify operation
- **scatter**: `O(n × log n)` worst case when seats are scattered across many rows, but typically much better

The [segment tree](/problems/segment_tree_intro) approach transforms what would be `O(n)` linear searches into `O(log n)` operations, making the solution efficient for large concert halls with many rows.

## Example Walkthrough

Let's walk through a small example with a concert hall having **4 rows** and **5 seats per row**.

**Initial Setup:**
```
BookMyShow(4, 5)  // 4 rows (0-3), 5 seats per row (0-4)

Initial state (all seats empty):
Row 0: [_, _, _, _, _]  (5 available)
Row 1: [_, _, _, _, _]  (5 available)
Row 2: [_, _, _, _, _]  (5 available)
Row 3: [_, _, _, _, _]  (5 available)

Segment Tree (1-indexed internally):
           [1,4]
         s=20, mx=5
        /           \
    [1,2]           [3,4]
   s=10,mx=5       s=10,mx=5
    /    \          /    \
  [1]    [2]      [3]    [4]
 s=5     s=5      s=5     s=5
 mx=5    mx=5     mx=5    mx=5
```

**Operation 1: gather(3, 1)**
- Want 3 consecutive seats in rows 0-1
- Query segment tree: Find first row in [1,2] with mx ≥ 3
- Tree traversal finds row 1 (0-indexed: row 0) has 5 seats ≥ 3
- Book seats 0,1,2 in row 0
- Update: modify row 1 to have 2 seats left

```
After gather(3, 1) returns [0, 0]:
Row 0: [X, X, X, _, _]  (2 available)
Row 1: [_, _, _, _, _]  (5 available)
Row 2: [_, _, _, _, _]  (5 available)
Row 3: [_, _, _, _, _]  (5 available)

Tree update at row 1: s=2, mx=2
Parent nodes updated via pushup
```

**Operation 2: scatter(7, 2)**
- Want 7 total seats in rows 0-2
- Check sum: query_sum([1,3]) = 2+5+5 = 12 ≥ 7 ✓
- Find first row with seats: row 1 has 2 seats
- Allocation process:
  - Row 0: Take 2 seats (positions 3,4), k=5 remaining
  - Row 1: Take 5 seats (positions 0-4), k=0 remaining
- Return true

```
After scatter(7, 2) returns true:
Row 0: [X, X, X, X, X]  (0 available)
Row 1: [X, X, X, X, X]  (0 available)
Row 2: [_, _, _, _, _]  (5 available)
Row 3: [_, _, _, _, _]  (5 available)

Tree updates:
- Row 1: s=0, mx=0
- Row 2: s=0, mx=0
- Parent nodes updated
```

**Operation 3: gather(4, 3)**
- Want 4 consecutive seats in rows 0-3
- Query segment tree: Find first row in [1,4] with mx ≥ 4
- Traversal checks:
  - Left subtree [1,2]: mx=0 (no row has 4 seats)
  - Right subtree [3,4]: mx=5 (check here)
  - Row 3 has 5 seats ≥ 4
- Book seats 0,1,2,3 in row 2
- Return [2, 0]

```
After gather(4, 3) returns [2, 0]:
Row 0: [X, X, X, X, X]  (0 available)
Row 1: [X, X, X, X, X]  (0 available)
Row 2: [X, X, X, X, _]  (1 available)
Row 3: [_, _, _, _, _]  (5 available)

Tree update at row 3: s=1, mx=1
```

**Operation 4: gather(2, 1)**
- Want 2 consecutive seats in rows 0-1
- Query segment tree: Find first row in [1,2] with mx ≥ 2
- Both rows have mx=0 (no seats available)
- Return []

```
gather(2, 1) returns [] (no available seats in rows 0-1)
```

This example demonstrates how the segment tree efficiently tracks available seats, finds suitable rows for group bookings, and manages scattered allocations while maintaining the "smallest row first, smallest seat first" allocation rule.

## Python Solution

```python
from typing import List, Optional


class SegmentTreeNode:
    """Node for segment tree storing range information and statistics."""
    __slots__ = "left", "right", "sum", "max_value"

    def __init__(self):
        self.left = 0          # Left boundary of range
        self.right = 0         # Right boundary of range
        self.sum = 0           # Sum of available seats in range
        self.max_value = 0     # Maximum available seats in any row within range


class SegmentTree:
    """Segment tree for efficient range queries and updates on row seat availability."""

    def __init__(self, num_rows: int, seats_per_row: int):
        """
        Initialize segment tree for venue with given dimensions.

        Args:
            num_rows: Number of rows in the venue
            seats_per_row: Number of seats in each row
        """
        self.seats_per_row = seats_per_row
        # Allocate 4n nodes for complete binary tree representation
        self.nodes = [SegmentTreeNode() for _ in range(num_rows << 2)]
        self._build(1, 1, num_rows)

    def _build(self, node_idx: int, left: int, right: int) -> None:
        """
        Recursively build the segment tree.

        Args:
            node_idx: Current node index in tree array
            left: Left boundary of current range
            right: Right boundary of current range
        """
        self.nodes[node_idx].left = left
        self.nodes[node_idx].right = right

        if left == right:
            # Leaf node: initialize with full row capacity
            self.nodes[node_idx].sum = self.seats_per_row
            self.nodes[node_idx].max_value = self.seats_per_row
            return

        mid = (left + right) >> 1
        left_child = node_idx << 1
        right_child = (node_idx << 1) | 1

        # Build left and right subtrees
        self._build(left_child, left, mid)
        self._build(right_child, mid + 1, right)

        # Update current node based on children
        self._push_up(node_idx)

    def modify(self, node_idx: int, target_row: int, new_value: int) -> None:
        """
        Update the available seats for a specific row.

        Args:
            node_idx: Current node index in tree traversal
            target_row: Row number to update (1-indexed)
            new_value: New number of available seats
        """
        current_node = self.nodes[node_idx]

        if current_node.left == target_row and current_node.right == target_row:
            # Found target leaf node
            current_node.sum = new_value
            current_node.max_value = new_value
            return

        mid = (current_node.left + current_node.right) >> 1
        left_child = node_idx << 1
        right_child = (node_idx << 1) | 1

        # Recursively update appropriate subtree
        if target_row <= mid:
            self.modify(left_child, target_row, new_value)
        else:
            self.modify(right_child, target_row, new_value)

        # Update current node based on modified children
        self._push_up(node_idx)

    def query_sum(self, node_idx: int, query_left: int, query_right: int) -> int:
        """
        Query the total available seats in a range of rows.

        Args:
            node_idx: Current node index in tree traversal
            query_left: Left boundary of query range (1-indexed)
            query_right: Right boundary of query range (1-indexed)

        Returns:
            Total available seats in the specified range
        """
        current_node = self.nodes[node_idx]

        # Current range completely within query range
        if current_node.left >= query_left and current_node.right <= query_right:
            return current_node.sum

        mid = (current_node.left + current_node.right) >> 1
        left_child = node_idx << 1
        right_child = (node_idx << 1) | 1
        total = 0

        # Query left subtree if overlap exists
        if query_left <= mid:
            total += self.query_sum(left_child, query_left, query_right)

        # Query right subtree if overlap exists
        if query_right > mid:
            total += self.query_sum(right_child, query_left, query_right)

        return total

    def query_idx(self, node_idx: int, query_left: int, query_right: int, min_seats: int) -> int:
        """
        Find the first row with at least min_seats available seats.

        Args:
            node_idx: Current node index in tree traversal
            query_left: Left boundary of query range (1-indexed)
            query_right: Right boundary of query range (1-indexed)
            min_seats: Minimum number of required seats

        Returns:
            Row number (1-indexed) of first suitable row, or 0 if none found
        """
        current_node = self.nodes[node_idx]

        # No row in this range has enough seats
        if current_node.max_value < min_seats:
            return 0

        # Leaf node with enough seats
        if current_node.left == current_node.right:
            return current_node.left

        mid = (current_node.left + current_node.right) >> 1
        left_child = node_idx << 1
        right_child = (node_idx << 1) | 1

        # Check left subtree first (prefer earlier rows)
        if self.nodes[left_child].max_value >= min_seats:
            return self.query_idx(left_child, query_left, query_right, min_seats)

        # Check right subtree if needed
        if query_right > mid:
            return self.query_idx(right_child, query_left, query_right, min_seats)

        return 0

    def _push_up(self, node_idx: int) -> None:
        """
        Update parent node based on children's values.

        Args:
            node_idx: Index of parent node to update
        """
        left_child = node_idx << 1
        right_child = (node_idx << 1) | 1

        # Sum is total of both children
        self.nodes[node_idx].sum = (
            self.nodes[left_child].sum + self.nodes[right_child].sum
        )

        # Max is maximum of both children
        self.nodes[node_idx].max_value = max(
            self.nodes[left_child].max_value,
            self.nodes[right_child].max_value
        )


class BookMyShow:
    """System for booking seats in a show venue with multiple rows."""

    def __init__(self, n: int, m: int):
        """
        Initialize the booking system.

        Args:
            n: Number of rows in the venue
            m: Number of seats per row
        """
        self.num_rows = n
        self.segment_tree = SegmentTree(n, m)

    def gather(self, k: int, maxRow: int) -> List[int]:
        """
        Attempt to book k consecutive seats in a single row.

        Args:
            k: Number of consecutive seats to book
            maxRow: Maximum row index to consider (0-indexed)

        Returns:
            [row_index, starting_seat] if successful, empty list otherwise
        """
        # Convert to 1-indexed for segment tree
        max_row_inclusive = maxRow + 1

        # Find first row with at least k available seats
        row_number = self.segment_tree.query_idx(1, 1, max_row_inclusive, k)

        if row_number == 0:
            return []

        # Get current available seats in found row
        available_seats = self.segment_tree.query_sum(1, row_number, row_number)

        # Book the seats by reducing available count
        self.segment_tree.modify(1, row_number, available_seats - k)

        # Return 0-indexed row and starting seat position
        starting_seat = self.segment_tree.seats_per_row - available_seats
        return [row_number - 1, starting_seat]

    def scatter(self, k: int, maxRow: int) -> bool:
        """
        Attempt to book k seats, possibly across multiple rows.

        Args:
            k: Number of seats to book
            maxRow: Maximum row index to consider (0-indexed)

        Returns:
            True if booking successful, False otherwise
        """
        # Convert to 1-indexed for segment tree
        max_row_inclusive = maxRow + 1

        # Check if enough total seats available
        total_available = self.segment_tree.query_sum(1, 1, max_row_inclusive)
        if total_available < k:
            return False

        # Find first row with at least one available seat
        first_available_row = self.segment_tree.query_idx(1, 1, max_row_inclusive, 1)

        # Book seats row by row, starting from first available
        remaining_to_book = k
        for row_number in range(first_available_row, self.num_rows + 1):
            available_in_row = self.segment_tree.query_sum(1, row_number, row_number)

            if available_in_row >= remaining_to_book:
                # Can finish booking in this row
                self.segment_tree.modify(1, row_number, available_in_row - remaining_to_book)
                return True

            # Book all available seats in this row
            remaining_to_book -= available_in_row
            self.segment_tree.modify(1, row_number, 0)

        return True
```

## Time and Space Complexity

**Time Complexity:**

The overall time complexity is `O(n + q × log n)`, where `n` is the number of rows and `q` is the number of operations.

Breaking down the operations:
- **Initialization (`__init__`)**: The segment tree construction calls `build()` which visits each node once in the tree. With `4n` nodes total, this takes `O(n)` time.

- **`gather()` operation**:
  - `query_idx()`: Traverses from root to leaf in the segment tree, taking `O(log n)` time
  - `query_sum()`: Also traverses the tree path, taking `O(log n)` time
  - `modify()`: Updates values along the path from root to leaf, taking `O(log n)` time
  - Total per gather: `O(log n)`

- **`scatter()` operation**:
  - `query_sum()` for checking total: `O(log n)` time
  - `query_idx()` to find first available row: `O(log n)` time
  - In the worst case, the loop iterates through all `n` rows, and for each row:
    - `query_sum()`: `O(log n)`
    - `modify()`: `O(log n)`
  - Worst case per scatter: `O(n × log n)`

However, amortized over all operations, each seat is only allocated once. Across `q` operations, the total work for all scatter operations is bounded by `O(q × log n)` amortized.

**Space Complexity:**

The space complexity is `O(n)`.

- The segment tree uses an array of size `4n` to store nodes (using the standard `n << 2` allocation for 1-indexed segment trees)
- Each `Node` object stores 4 integer values (`l`, `r`, `s`, `mx`)
- Additional space usage is `O(1)` for other variables

Therefore, the total space used is `O(4n)` = `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Index Confusion Between 0-indexed and 1-indexed Systems**

The most common pitfall in this implementation is the mixing of 0-indexed (problem requirement) and 1-indexed (segment tree internal) coordinate systems. This creates multiple opportunities for off-by-one errors.

**Where it happens:**
- The problem uses 0-indexed rows (0 to n-1) and seats (0 to m-1)
- The segment tree internally uses 1-indexed rows (1 to n)
- Conversion happens at the interface between BookMyShow methods and SegmentTree

**Specific error-prone areas:**
```python
# In gather method:
max_row_inclusive = maxRow + 1  # Convert 0-indexed to 1-indexed
return [row_number - 1, starting_seat]  # Convert back to 0-indexed

# In scatter method:
max_row_inclusive = maxRow + 1  # Same conversion needed
```

**Solution:**
Create explicit conversion helper methods to centralize this logic:

```python
def _to_internal_row(self, external_row: int) -> int:
    """Convert 0-indexed external row to 1-indexed internal row."""
    return external_row + 1

def _to_external_row(self, internal_row: int) -> int:
    """Convert 1-indexed internal row to 0-indexed external row."""
    return internal_row - 1
```

### 2. **Incorrect Seat Position Calculation**

Another subtle pitfall is calculating the starting seat position when booking consecutive seats in the `gather` method.

**The problem:**
```python
starting_seat = self.segment_tree.seats_per_row - available_seats
```

This assumes seats are allocated from left to right, but if there's a bug in tracking which specific seats are taken (the current implementation only tracks counts), this could return incorrect positions.

**Example scenario:**
- Row has 10 seats total, 5 available
- But the available seats might not be seats 5-9; they could be scattered
- The current implementation assumes they're always the rightmost seats

**Solution:**
If precise seat tracking is needed, maintain a bitmap or list of available seats per row:

```python
class BookMyShow:
    def __init__(self, n: int, m: int):
        self.num_rows = n
        self.seats_per_row = m
        self.segment_tree = SegmentTree(n, m)
        # Track actual seat availability per row
        self.row_seats = [[True] * m for _ in range(n)]

    def _find_consecutive_seats(self, row: int, k: int) -> int:
        """Find starting position of k consecutive available seats."""
        count = 0
        for i in range(self.seats_per_row):
            if self.row_seats[row][i]:
                count += 1
                if count == k:
                    return i - k + 1
            else:
                count = 0
        return -1
```

### 3. **Range Boundary Errors in query_idx**

The `query_idx` method doesn't properly respect the query boundaries when recursing:

**Current problematic code:**
```python
def query_idx(self, node_idx: int, query_left: int, query_right: int, min_seats: int) -> int:
    # ...
    if self.nodes[left_child].max_value >= min_seats:
        return self.query_idx(left_child, query_left, query_right, min_seats)

    if query_right > mid:
        return self.query_idx(right_child, query_left, query_right, min_seats)
```

**The issue:** The method checks the left child's max_value but doesn't verify if the left child's range actually overlaps with [query_left, query_right].

**Solution:**
```python
def query_idx(self, node_idx: int, query_left: int, query_right: int, min_seats: int) -> int:
    current_node = self.nodes[node_idx]

    # No row in this range has enough seats
    if current_node.max_value < min_seats:
        return 0

    # Current range completely outside query range
    if current_node.right < query_left or current_node.left > query_right:
        return 0

    # Leaf node with enough seats
    if current_node.left == current_node.right:
        return current_node.left

    mid = (current_node.left + current_node.right) >> 1
    left_child = node_idx << 1
    right_child = (node_idx << 1) | 1

    # Check left subtree first if it overlaps with query range
    if query_left <= mid:
        result = self.query_idx(left_child, query_left, query_right, min_seats)
        if result != 0:
            return result

    # Check right subtree if it overlaps with query range
    if query_right > mid:
        return self.query_idx(right_child, query_left, query_right, min_seats)

    return 0
```

### 4. **Performance Degradation in scatter Method**

The scatter method iterates through rows sequentially after finding the first available row, which could be inefficient:

**Current approach:**
```python
for row_number in range(first_available_row, self.num_rows + 1):
    # Process each row sequentially
```

**The issue:** This creates O(n) iterations in worst case, each with O(log n) segment tree operations.

**Solution:** Use the segment tree more efficiently by finding each next available row:

```python
def scatter(self, k: int, maxRow: int) -> bool:
    max_row_inclusive = maxRow + 1

    if self.segment_tree.query_sum(1, 1, max_row_inclusive) < k:
        return False

    remaining = k
    current_row = 1

    while remaining > 0 and current_row <= max_row_inclusive:
        # Find next row with available seats
        next_row = self.segment_tree.query_idx(1, current_row, max_row_inclusive, 1)
        if next_row == 0:
            break

        available = self.segment_tree.query_sum(1, next_row, next_row)
        seats_to_book = min(available, remaining)

        self.segment_tree.modify(1, next_row, available - seats_to_book)
        remaining -= seats_to_book
        current_row = next_row + 1

    return True
```

