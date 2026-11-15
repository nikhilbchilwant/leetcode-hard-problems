# 683. K Empty Slots

> Source: https://algo.monster/liteproblems/683

## Problem Statement

You have `n` bulbs arranged in a row, numbered from `1` to `n`. Initially, all bulbs are turned off. Each day, you turn on exactly one bulb, and after `n` days, all bulbs will be turned on.

You're given an array `bulbs` of length `n` where `bulbs[i] = x` means that on day `(i+1)`, you turn on the bulb at position `x`. Note that `i` is 0-indexed (starts from 0) while bulb positions `x` are 1-indexed (start from 1).

Given an integer `k`, you need to find the **minimum day number** where there exist two turned-on bulbs that have **exactly** `k` bulbs between them, and all those `k` bulbs in between are turned off.

For example, if `k = 1` and on some day you have bulbs at positions 3 and 5 turned on with bulb 4 turned off, this would satisfy the condition (exactly 1 bulb between them that is off).

If no such day exists where this condition is met, return `-1`.

The solution uses a Binary Indexed Tree (Fenwick Tree) to efficiently track which bulbs are turned on and check if there are exactly `k` turned-off bulbs between any two turned-on bulbs. On each day when a bulb is turned on, it checks:
- If there's a turned-on bulb exactly `k+1` positions to the left with all bulbs in between turned off
- If there's a turned-on bulb exactly `k+1` positions to the right with all bulbs in between turned off

The Binary Indexed Tree helps query the count of turned-on bulbs in any range in `O(log n)` time, making it efficient to verify if all `k` bulbs between two positions are turned off.

## Intuition

The key insight is that we need to efficiently check, for each newly turned-on bulb, whether it forms a valid pair with any previously turned-on bulb with exactly `k` turned-off bulbs between them.

When we turn on a bulb at position `x`, we only need to check two specific positions:
- Position `x - k - 1` (a bulb that would be `k+1` positions to the left)
- Position `x + k + 1` (a bulb that would be `k+1` positions to the right)

Why only these two positions? Because if there's a valid pair involving the current bulb, the other bulb must be exactly `k+1` positions away. Any other distance wouldn't give us exactly `k` bulbs in between.

The challenge is verifying that all `k` bulbs between two positions are turned off. A naive approach would check each bulb individually, taking `O(k)` time per check. Since we might do this check for each of `n` days, this could be inefficient.

This is where the Binary Indexed Tree comes in. By maintaining a prefix sum of turned-on bulbs, we can query how many bulbs are turned on in any range in `O(log n)` time. For positions `y` and `x` with `y < x`, if exactly 0 bulbs are turned on between them, then `query(x-1) - query(y)` should equal 0. This means the cumulative count of turned-on bulbs up to position `x-1` minus the count up to position `y` equals zero, confirming all bulbs between `y` and `x` are off.

The `vis` array tracks which bulbs are already on, so we can quickly check if the potential partner bulb at position `y` is turned on before checking the range between them.

By processing bulbs in the order they're turned on and checking these conditions immediately, we can find the earliest day when the condition is satisfied.

**Learn more about [Segment Tree](/problems/segment_tree_intro), [Queue](/problems/queue_intro), [Sliding Window](/problems/sliding_window_maximum), [Monotonic Queue](/problems/mono_stack_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution uses a Binary Indexed Tree (Fenwick Tree) to efficiently track and query the state of bulbs. Let's walk through the implementation step by step:

**1. Binary Indexed Tree Setup**

The `BinaryIndexedTree` class maintains a prefix sum structure:
- `__init__(n)`: Creates an array `c` of size `n+1` initialized with zeros
- `update(x, delta)`: Adds `delta` to position `x` and updates all affected nodes using the formula `x += x & -x` to traverse up the tree
- `query(x)`: Returns the sum from position 1 to `x` by traversing down using `x -= x & -x`

**2. Main Algorithm**

Initialize:
- Create a Binary Indexed Tree of size `n`
- Create a boolean array `vis` of size `n+1` to track which bulbs are turned on
- Process bulbs day by day using `enumerate(bulbs, 1)` to get both day number and bulb position

**3. For Each Day**

When turning on bulb at position `x` on day `i`:

a. **Update the state:**
   - Call `tree.update(x, 1)` to mark this bulb as turned on in the BIT
   - Set `vis[x] = True` to mark this bulb as visible

b. **Check left neighbor:**
   - Calculate potential left partner: `y = x - k - 1`
   - If `y > 0` (valid position) and `vis[y]` (bulb at y is on):
     - Check if all bulbs between `y` and `x` are off using: `tree.query(x - 1) - tree.query(y) == 0`
     - If true, return current day `i`

c. **Check right neighbor:**
   - Calculate potential right partner: `y = x + k + 1`
   - If `y <= n` (valid position) and `vis[y]` (bulb at y is on):
     - Check if all bulbs between `x` and `y` are off using: `tree.query(y - 1) - tree.query(x) == 0`
     - If true, return current day `i`

**4. Range Query Logic**

The expression `tree.query(b) - tree.query(a)` gives the count of turned-on bulbs in range `(a, b]`. When this equals 0, all bulbs in that range are off.

**5. Return Value**

If no valid configuration is found after processing all days, return `-1`.

**Time Complexity:** `O(n log n)` - We process `n` bulbs, and each update/query operation takes `O(log n)` time.

**Space Complexity:** `O(n)` - For the BIT array and visibility array.

## Example Walkthrough

Let's walk through a concrete example with `bulbs = [2, 1, 3, 5, 4]` and `k = 1`.

We need to find the minimum day where two turned-on bulbs have exactly 1 bulb between them that is turned off.

**Initial Setup:**
- Binary Indexed Tree: tracks turned-on bulbs (initially all 0)
- Visibility array `vis`: [False, False, False, False, False, False] (index 0 unused)
- Bulb positions: [1, 2, 3, 4, 5]

**Day 1:** Turn on bulb at position 2
- Update BIT at position 2, set `vis[2] = True`
- Check left neighbor: position `2 - 1 - 1 = 0` (invalid, skip)
- Check right neighbor: position `2 + 1 + 1 = 4`
  - `vis[4] = False` (bulb 4 not on yet, skip)
- Current state: [OFF, **ON**, OFF, OFF, OFF]

**Day 2:** Turn on bulb at position 1
- Update BIT at position 1, set `vis[1] = True`
- Check left neighbor: position `1 - 1 - 1 = -1` (invalid, skip)
- Check right neighbor: position `1 + 1 + 1 = 3`
  - `vis[3] = False` (bulb 3 not on yet, skip)
- Current state: [**ON**, **ON**, OFF, OFF, OFF]

**Day 3:** Turn on bulb at position 3
- Update BIT at position 3, set `vis[3] = True`
- Check left neighbor: position `3 - 1 - 1 = 1`
  - `vis[1] = True` (bulb 1 is on!)
  - Check range (1, 3): `tree.query(2) - tree.query(1) = 2 - 1 = 1`
  - This counts the range (1, 2], which is just position 2
  - Position 2 has a turned-on bulb, so count = 1 ≠ 0 (not valid)
- Check right neighbor: position `3 + 1 + 1 = 5`
  - `vis[5] = False` (bulb 5 not on yet, skip)
- Current state: [**ON**, **ON**, **ON**, OFF, OFF]

**Day 4:** Turn on bulb at position 5
- Update BIT at position 5, set `vis[5] = True`
- Check left neighbor: position `5 - 1 - 1 = 3`
  - `vis[3] = True` (bulb 3 is on!)
  - Check range (3, 5): `tree.query(4) - tree.query(3) = 3 - 3 = 0`
  - All bulbs between 3 and 5 (just position 4) are off! ✓
  - **Found valid configuration: bulbs 3 and 5 are on with bulb 4 off between them**
- Return day 4

The answer is **4** - this is the first day where we have two turned-on bulbs with exactly k=1 turned-off bulb between them.

## Python Solution

```python
class BinaryIndexedTree:
    """
    Binary Indexed Tree (Fenwick Tree) for efficient range sum queries
    and point updates in O(log n) time.
    """

    def __init__(self, n: int) -> None:
        """
        Initialize a Binary Indexed Tree with n elements.

        Args:
            n: The number of elements (1-indexed)
        """
        self.size = n
        self.tree = [0] * (n + 1)  # 1-indexed array for BIT

    def update(self, index: int, delta: int) -> None:
        """
        Add delta to the element at the given index.

        Args:
            index: The position to update (1-indexed)
            delta: The value to add to the element
        """
        while index <= self.size:
            self.tree[index] += delta
            # Move to next index by adding the least significant bit
            index += index & (-index)

    def query(self, index: int) -> int:
        """
        Calculate the prefix sum from index 1 to the given index.

        Args:
            index: The end position for the prefix sum (1-indexed)

        Returns:
            The sum of elements from position 1 to index
        """
        prefix_sum = 0
        while index > 0:
            prefix_sum += self.tree[index]
            # Move to parent index by removing the least significant bit
            index -= index & (-index)
        return prefix_sum


class Solution:
    def kEmptySlots(self, bulbs: List[int], k: int) -> int:
        """
        Find the earliest day when there are exactly k empty slots between
        two turned-on bulbs.

        Args:
            bulbs: Array where bulbs[i] is the position of bulb turned on at day i+1
            k: The required number of empty slots between two turned-on bulbs

        Returns:
            The earliest day (1-indexed) when the condition is met, or -1 if never
        """
        n = len(bulbs)
        fenwick_tree = BinaryIndexedTree(n)
        is_turned_on = [False] * (n + 1)  # Track which bulbs are turned on

        # Process each day (1-indexed)
        for day, bulb_position in enumerate(bulbs, start=1):
            # Turn on the bulb at the current position
            fenwick_tree.update(bulb_position, 1)
            is_turned_on[bulb_position] = True

            # Check left neighbor: position that is k+1 slots to the left
            left_neighbor = bulb_position - k - 1
            if left_neighbor > 0 and is_turned_on[left_neighbor]:
                # Count turned-on bulbs between left_neighbor and current position
                bulbs_between = fenwick_tree.query(bulb_position - 1) - fenwick_tree.query(left_neighbor)
                if bulbs_between == 0:  # Exactly k empty slots between them
                    return day

            # Check right neighbor: position that is k+1 slots to the right
            right_neighbor = bulb_position + k + 1
            if right_neighbor <= n and is_turned_on[right_neighbor]:
                # Count turned-on bulbs between current position and right_neighbor
                bulbs_between = fenwick_tree.query(right_neighbor - 1) - fenwick_tree.query(bulb_position)
                if bulbs_between == 0:  # Exactly k empty slots between them
                    return day

        # No valid day found
        return -1
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The algorithm iterates through all `n` bulbs exactly once in the main loop. For each bulb at position `i`:
- The `update` operation on the Binary Indexed Tree takes `O(log n)` time, as it traverses at most `log n` nodes by incrementing `x` by its least significant bit (`x += x & -x`)
- The `query` operation is called at most 4 times (twice for each potential valid position check), and each query takes `O(log n)` time as it traverses the tree by removing the least significant bit (`x -= x & -x`)
- The visibility checks and arithmetic operations take `O(1)` time

Therefore, the total time complexity is `O(n)` iterations × `O(log n)` operations per iteration = `O(n × log n)`.

**Space Complexity:** `O(n)`

The algorithm uses:
- Binary Indexed Tree array `c` of size `n + 1`: `O(n)` space
- Boolean array `vis` of size `n + 1` to track which bulbs are turned on: `O(n)` space
- A few constant variables (`i`, `x`, `y`, `s`): `O(1)` space

The total space complexity is `O(n) + O(n) + O(1) = O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Confusing Index Systems (0-indexed vs 1-indexed)**

One of the most common mistakes in this problem is mixing up the different indexing conventions:
- The `bulbs` array is 0-indexed (positions 0 to n-1)
- The bulb positions within the array are 1-indexed (bulbs numbered 1 to n)
- The Binary Indexed Tree uses 1-indexed positions
- Days are counted starting from 1

**Pitfall Example:**
```python
# WRONG: Using 0-indexed for BIT or vis array
vis = [False] * n  # Should be n+1 to accommodate 1-indexed positions
for i, x in enumerate(bulbs):  # Missing the day offset
    tree.update(x, 1)
    # Checking vis[x] would cause index out of bounds
```

**Solution:**
```python
# CORRECT: Properly handle indexing
vis = [False] * (n + 1)  # Extra space for 1-indexed positions
for day, bulb_position in enumerate(bulbs, start=1):  # Start days from 1
    tree.update(bulb_position, 1)
    vis[bulb_position] = True
```

### 2. **Incorrect Range Query for Counting Bulbs Between Two Positions**

When checking if all k bulbs between two positions are off, it's crucial to query the correct range boundaries.

**Pitfall Example:**
```python
# WRONG: Including the endpoints in the range check
left_neighbor = x - k - 1
if vis[left_neighbor]:
    # This includes both endpoints, counting k+2 positions total
    bulbs_between = tree.query(x) - tree.query(left_neighbor - 1)
```

**Solution:**
```python
# CORRECT: Exclude both endpoints from the range
left_neighbor = x - k - 1
if vis[left_neighbor]:
    # Query from (left_neighbor, x) exclusive on both ends
    bulbs_between = tree.query(x - 1) - tree.query(left_neighbor)
```

### 3. **Forgetting to Check Boundary Conditions**

Not validating whether neighbor positions are within valid bounds before accessing arrays.

**Pitfall Example:**
```python
# WRONG: Not checking if positions are valid
left_neighbor = x - k - 1
if vis[left_neighbor]:  # Could cause negative index access
    # Process...

right_neighbor = x + k + 1
if vis[right_neighbor]:  # Could cause index out of bounds
    # Process...
```

**Solution:**
```python
# CORRECT: Always validate position bounds first
left_neighbor = x - k - 1
if left_neighbor > 0 and vis[left_neighbor]:
    # Safe to process...

right_neighbor = x + k + 1
if right_neighbor <= n and vis[right_neighbor]:
    # Safe to process...
```

### 4. **Misunderstanding the Problem Requirements**

A subtle but critical pitfall is misinterpreting what "exactly k bulbs between them" means.

**Pitfall Example:**
```python
# WRONG: Looking for k+1 distance instead of k bulbs between
left_neighbor = x - k  # Should be x - k - 1
right_neighbor = x + k  # Should be x + k + 1
```

**Solution:**
If we need exactly k bulbs between two turned-on bulbs, the distance between them should be k+1. For example, if k=2 and we have bulbs at positions 3 and 6, there are exactly 2 bulbs between them (positions 4 and 5).

### 5. **Inefficient Implementation Without Binary Indexed Tree**

Attempting to solve this with naive array scanning for each check.

**Pitfall Example:**
```python
# Correct but slower: O(nk) approach - checking each bulb individually
def check_range(start, end, vis):
    for i in range(start + 1, end):
        if vis[i]:
            return False
    return True
```

**Solution:**
Use the Binary Indexed Tree as shown in the main solution to achieve O(log n) range queries, resulting in O(n log n) total time complexity instead of O(nk).

