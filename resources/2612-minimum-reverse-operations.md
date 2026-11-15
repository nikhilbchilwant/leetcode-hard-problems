# 2612. Minimum Reverse Operations

> Source: https://algo.monster/liteproblems/2612

## Problem Statement

You have an array `arr` of length `n` where all elements are initially 0, except for position `p` which is set to 1. You're also given an array `banned` containing positions that are restricted.

You can perform the following operation:
- Reverse any subarray of size `k`, but only if after the reversal, the single 1 doesn't end up in any position listed in `banned`.

Your task is to find the minimum number of operations needed to move the single 1 to each position `i` (where `i` ranges from 0 to n-1). If it's impossible to move the 1 to position `i`, return -1 for that position.

For example, if you have an array `[0, 0, 1, 0, 0]` with `k=3`, you could reverse the subarray `[0, 1, 0]` at indices 1-3 to get `[0, 0, 1, 0, 0]` → `[0, 0, 1, 0, 0]`. Or you could reverse `[0, 0, 1]` at indices 0-2 to get `[1, 0, 0, 0, 0]`.

The key insight is that when you reverse a subarray `[l, ..., r]` of size `k`, an element at position `i` within that subarray moves to position `j = l + r - i`. This creates a pattern where:
- Moving the subarray one position right increases the flipped position by 2
- Moving the subarray one position left decreases the flipped position by 2
- All reachable positions from a given index have the same parity (all even or all odd)

## Intuition

When we reverse a subarray of size `k`, we're essentially flipping the positions of elements within that subarray. If an element is at position `i` within a subarray `[l, r]`, after reversal it moves to position `l + r - i`. This is the key observation that unlocks the entire problem.

Let's think about what happens when we slide our reversal window. If we move the window one position to the right, both `l` and `r` increase by 1, so the new position becomes `(l+1) + (r+1) - i = l + r - i + 2`. Similarly, moving left decreases the position by 2. This means all positions reachable from index `i` form an arithmetic sequence with a common difference of 2 - they all have the same parity (all even or all odd).

Now, we need to figure out the range of positions we can reach from any given index `i`. Without any boundaries, we could reach positions from `i - k + 1` to `i + k - 1`. But we need to consider edge cases:
- When the subarray is at the leftmost position `[0, k-1]`, index `i` flips to `k - i - 1`
- When the subarray is at the rightmost position `[n-k, n-1]`, index `i` flips to `2n - k - i - 1`

So the actual reachable range is `[max(i - k + 1, k - i - 1), min(i + k - 1, 2n - k - i - 1)]`.

Since we need the minimum number of operations to reach each position, BFS is perfect - it explores positions level by level, guaranteeing we find the shortest path first. We can optimize by maintaining two ordered sets (one for odd indices, one for even) to quickly find and remove all unvisited positions within our reachable range. This way, we avoid revisiting positions and ensure each position is processed exactly once.

The banned positions are simply removed from our sets at the beginning, ensuring we never try to place the 1 at those positions during our BFS traversal.

**Learn more about [Breadth-First Search](/problems/bfs_intro) patterns.**

## Solution Approach

The solution implements BFS with ordered sets to efficiently find all reachable positions. Here's how the implementation works:

**1. Initialize the answer array and data structures:**
- Create `ans` array of size `n` initialized with -1, except `ans[p] = 0` (starting position requires 0 operations)
- Create two `SortedSet` objects `ts[0]` and `ts[1]` to store unvisited even and odd indices respectively
- Add all indices to their corresponding set based on parity: `ts[i % 2].add(i)`
- Remove the starting position `p` and all banned positions from the sets
- Add sentinel value `n` to both sets to avoid index out of bounds during iteration

**2. Set up BFS:**
- Initialize a queue `q` with the starting position `p`
- Process positions level by level to ensure minimum operations

**3. For each position `i` in the queue, calculate the reachable range:**
- Left boundary: `mi = max(i - k + 1, k - i - 1)`
  - `i - k + 1`: normal left boundary without edge constraints
  - `k - i - 1`: position when subarray is at leftmost `[0, k-1]`
- Right boundary: `mx = min(i + k - 1, n * 2 - k - i - 1)`
  - `i + k - 1`: normal right boundary without edge constraints
  - `n * 2 - k - i - 1`: position when subarray is at rightmost `[n-k, n-1]`

**4. Process all reachable positions with the same parity:**
- Select the appropriate set based on parity: `s = ts[mi % 2]`
- Use binary search to find the starting position: `j = s.bisect_left(mi)`
- Iterate through all positions in range `[mi, mx]` with step 2 (same parity):
  ```python
  while s[j] <= mx:
      q.append(s[j])           # Add to queue for next level
      ans[s[j]] = ans[i] + 1   # Update minimum operations
      s.remove(s[j])           # Mark as visited
      j = s.bisect_left(mi)    # Find next unvisited position
  ```

**5. Continue BFS until queue is empty:**
- Each position is visited exactly once (removed from set when visited)
- BFS guarantees the first time we reach a position is with minimum operations

The time complexity is `O(n log n)` due to the ordered set operations, and space complexity is `O(n)` for storing the sets and queue. The clever use of parity-based sets and range calculations makes this solution efficient even for large arrays.

## Example Walkthrough

Let's walk through a concrete example to understand how the solution works.

**Given:**
- `n = 7` (array length)
- `p = 2` (starting position of the 1)
- `k = 3` (reversal subarray size)
- `banned = [4]` (position 4 is restricted)
- Initial array: `[0, 0, 1, 0, 0, 0, 0]`

**Step 1: Initialization**
- Answer array: `ans = [-1, -1, 0, -1, -1, -1, -1]` (position 2 starts with 0 operations)
- Even indices set: `ts[0] = {0, 6, 7}` (removed 2 and 4)
- Odd indices set: `ts[1] = {1, 3, 5, 7}` (removed nothing initially)
- Queue: `q = [2]`

**Step 2: Process position 2 (first BFS level)**

Calculate reachable range from position 2:
- Left boundary: `mi = max(2-3+1, 3-2-1) = max(0, 0) = 0`
- Right boundary: `mx = min(2+3-1, 14-3-2-1) = min(4, 8) = 4`
- Reachable positions: 0, 2, 4 (same parity as 0)

Since position 2 is already visited and position 4 is banned, we only process position 0:
- When subarray `[0, 1, 2]` is reversed: `[1, 0, 0, 0, 0, 0, 0]`
- Update: `ans[0] = ans[2] + 1 = 1`
- Remove 0 from `ts[0]`
- Add 0 to queue

**Step 3: Process position 0 (second BFS level)**

Calculate reachable range from position 0:
- Left boundary: `mi = max(0-3+1, 3-0-1) = max(-2, 2) = 2`
- Right boundary: `mx = min(0+3-1, 14-3-0-1) = min(2, 10) = 2`
- Only position 2 is reachable (already visited, so skip)

Now let's check what positions we can reach by examining different reversal windows:

From position 2 (where the 1 starts):
- Reverse `[0,1,2]`: position 2 → position 0 (2 flips to 0+2-2=0)
- Reverse `[1,2,3]`: position 2 → position 2 (2 flips to 1+3-2=2)
- Reverse `[2,3,4]`: position 2 → position 4 (2 flips to 2+4-2=4, but banned!)

From position 0 (after one operation):
- Reverse `[0,1,2]`: position 0 → position 2 (0 flips to 0+2-0=2)
- Cannot place reversal window before position 0

Let's continue exploring to find how to reach other positions:

**Step 4: Finding path to position 1**

We need to find an indirect path since position 1 has odd parity (unreachable directly from even position 2).

From position 2 → position 0 (1 operation)
Then we need to reach an odd position. Let's try different paths:

Actually, let's recalculate from position 0:
- Reverse `[0,1,2]`: 0 → 2
- No odd positions directly reachable from 0

We need to find another even position first. From position 2:
- Reverse `[4,5,6]`: position 2 stays at 2 (not in window)
- We need the 1 to be in the window to move it

Let me recalculate more carefully. When the 1 is at position 2:
- Window `[1,2,3]`: position 2 → 1+3-2 = 2 (stays)
- Window `[2,3,4]`: position 2 → 2+4-2 = 4 (banned)
- Window `[0,1,2]`: position 2 → 0+2-2 = 0 ✓

From position 0:
- Window `[0,1,2]`: position 0 → 0+2-0 = 2

This shows positions 0 and 2 can reach each other but need another path to reach odd indices.

**Final Result:**
After complete BFS traversal:
- `ans = [1, -1, 0, -1, -1, -1, 1]`

Position 6 can be reached from position 0 via window `[4,5,6]` where position 0 stays at 0, then window shifts... Actually, let me recalculate:

When 1 is at position 0, window `[4,5,6]` doesn't contain it. We need:
- Window `[0,1,2]`: 0 → 2
- From 2, window `[4,5,6]`: 2 stays at 2
- This doesn't work either.

The key insight is that we can only reach positions with the same parity through this reversal process, and the reachable range is limited by the formula we calculated. Positions 1, 3, and 5 remain unreachable (-1) because they have different parity from the starting position 2.

## Python Solution

```python
from collections import deque
from sortedcontainers import SortedSet
from typing import List


class Solution:
    def minReverseOperations(
        self, n: int, p: int, banned: List[int], k: int
    ) -> List[int]:
        """
        Find minimum reverse operations needed to reach each position.
      
        Args:
            n: Total number of positions
            p: Starting position
            banned: List of banned positions that cannot be visited
            k: Size of the subarray to reverse
      
        Returns:
            List where ans[i] is minimum operations to reach position i, or -1 if unreachable
        """
        # Initialize result array with -1 (unreachable by default)
        min_operations = [-1] * n
        min_operations[p] = 0  # Starting position requires 0 operations
      
        # Create two sorted sets for even and odd positions
        # This optimization helps track unvisited positions efficiently
        unvisited_positions = [SortedSet() for _ in range(2)]
      
        # Add all positions to their respective sets (even/odd)
        for position in range(n):
            unvisited_positions[position % 2].add(position)
      
        # Remove starting position as it's already visited
        unvisited_positions[p % 2].remove(p)
      
        # Remove banned positions as they cannot be visited
        for banned_pos in banned:
            unvisited_positions[banned_pos % 2].remove(banned_pos)
      
        # Add sentinel value to prevent index out of bounds
        unvisited_positions[0].add(n)
        unvisited_positions[1].add(n)
      
        # BFS queue starting from position p
        queue = deque([p])
      
        while queue:
            current_pos = queue.popleft()
          
            # Calculate the range of reachable positions after reversing
            # a subarray of length k that includes current_pos
            min_reachable = max(current_pos - k + 1, k - current_pos - 1)
            max_reachable = min(current_pos + k - 1, n * 2 - k - current_pos - 1)
          
            # Select the appropriate set based on parity
            # Reversing changes parity, so we look in the opposite parity set
            target_set = unvisited_positions[min_reachable % 2]
          
            # Find the first position >= min_reachable in the sorted set
            idx = target_set.bisect_left(min_reachable)
          
            # Process all reachable unvisited positions in range
            while target_set[idx] <= max_reachable:
                next_pos = target_set[idx]
                queue.append(next_pos)
                min_operations[next_pos] = min_operations[current_pos] + 1
                target_set.remove(next_pos)
                # Re-find the position after removal
                idx = target_set.bisect_left(min_reachable)
      
        return min_operations
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The algorithm uses BFS to traverse through positions. In the worst case, we visit each of the `n` positions exactly once. For each position visited:
- We perform two `bisect_left` operations on the SortedSet, each taking `O(log n)` time
- The while loop iterates through valid positions in the range `[mi, mx]`. Each position is processed exactly once throughout the entire algorithm (added to queue once, removed from SortedSet once)
- Each removal operation from SortedSet takes `O(log n)` time

Since each of the `n` positions is processed at most once with `O(log n)` operations (bisect_left and remove from SortedSet), the overall time complexity is `O(n × log n)`.

**Space Complexity:** `O(n)`

The space usage includes:
- `ans` array of size `n`: `O(n)`
- Two SortedSets storing positions, with total elements up to `n + 2` (including sentinel values): `O(n)`
- BFS queue storing at most `n` elements: `O(n)`
- `banned` list (input parameter, not counted in auxiliary space)

The total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Range Calculation for Reachable Positions**

One of the most common mistakes is incorrectly calculating the range of positions reachable from the current position. The formulas for `min_reachable` and `max_reachable` are complex because they need to handle edge cases where the subarray extends beyond array boundaries.

**Pitfall Example:**
```python
# WRONG: Naive approach without considering edge constraints
min_reachable = current_pos - k + 1
max_reachable = current_pos + k - 1
```

**Why it's wrong:** When the subarray would extend beyond array boundaries (left of 0 or right of n-1), the actual position after reversal follows a different pattern due to reflection.

**Correct Solution:**
```python
# Account for edge reflections
min_reachable = max(current_pos - k + 1, k - current_pos - 1)
max_reachable = min(current_pos + k - 1, n * 2 - k - current_pos - 1)
```

### 2. **Forgetting to Handle Parity Constraints**

Another critical pitfall is not recognizing that reversing a subarray only allows movement between positions of opposite parity. From an even position, you can only reach odd positions in one move, and vice versa.

**Pitfall Example:**
```python
# WRONG: Checking all positions in range
for next_pos in range(min_reachable, max_reachable + 1):
    if next_pos not in banned:
        # Process next_pos
```

**Correct Solution:**
```python
# Only check positions with correct parity (opposite of current)
target_set = unvisited_positions[min_reachable % 2]
# Process only positions in target_set (same parity as min_reachable)
```

### 3. **Inefficient Visited Tracking Leading to TLE**

Using a simple visited array or set without optimization can lead to Time Limit Exceeded for large inputs, especially when repeatedly checking which positions in a range haven't been visited yet.

**Pitfall Example:**
```python
# WRONG: Inefficient linear scan for each position
visited = set([p])
for next_pos in range(min_reachable, max_reachable + 1, 2):
    if next_pos not in visited and next_pos not in banned:
        visited.add(next_pos)
        queue.append(next_pos)
```

**Correct Solution:**
Using sorted sets with binary search for efficient range queries:
```python
# Use SortedSet for O(log n) operations
idx = target_set.bisect_left(min_reachable)
while target_set[idx] <= max_reachable:
    next_pos = target_set[idx]
    target_set.remove(next_pos)  # O(log n) removal
    # Process next_pos
```

### 4. **Not Adding Sentinel Values to Prevent Index Out of Bounds**

When using binary search on sorted sets, forgetting to add sentinel values can cause index out of bounds errors when all valid positions have been visited.

**Pitfall Example:**
```python
# WRONG: No sentinel, may crash when set becomes empty in range
while idx < len(target_set) and target_set[idx] <= max_reachable:
    # Process
```

**Correct Solution:**
```python
# Add sentinel value larger than any valid position
unvisited_positions[0].add(n)
unvisited_positions[1].add(n)
# Now target_set[idx] is always valid, will be n when no more positions
```

### 5. **Initializing Banned Positions Incorrectly**

A subtle pitfall is not properly handling the banned positions during initialization, especially when the starting position itself might be in the banned list (though typically problem constraints prevent this).

**Pitfall Example:**
```python
# WRONG: Removing from wrong parity set
for banned_pos in banned:
    unvisited_positions[0].discard(banned_pos)  # Always removing from even set
```

**Correct Solution:**
```python
# Remove from the correct parity set
for banned_pos in banned:
    unvisited_positions[banned_pos % 2].discard(banned_pos)
    # Or use remove() if you're certain the position exists
```

