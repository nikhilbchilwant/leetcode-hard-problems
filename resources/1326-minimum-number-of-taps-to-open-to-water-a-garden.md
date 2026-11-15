# 1326. Minimum Number of Taps to Open to Water a Garden

> Source: https://algo.monster/liteproblems/1326

## Problem Statement

You have a one-dimensional garden along the x-axis that starts at position `0` and ends at position `n`. The garden has a total length of `n` units.

There are `n + 1` taps installed at positions `[0, 1, 2, ..., n]` along the garden. Each tap has a specific watering range. The `i-th` tap (located at position `i`) can water the area from position `[i - ranges[i]]` to position `[i + ranges[i]]` when turned on, where `ranges[i]` represents the radius of coverage for that tap.

Your task is to find the minimum number of taps that need to be turned on to water the entire garden from position `0` to position `n`. If it's impossible to water the whole garden, return `-1`.

For example, if `n = 5` and `ranges = [3, 4, 1, 1, 0, 0]`:
- Tap at position 0 can water `[0-3, 0+3] = [-3, 3]` (effectively `[0, 3]` since we can't water negative positions)
- Tap at position 1 can water `[1-4, 1+4] = [-3, 5]` (effectively `[0, 5]`)
- Tap at position 2 can water `[2-1, 2+1] = [1, 3]`
- And so on...

The goal is to select the minimum number of taps such that every position from `0` to `n` gets watered.

## Intuition

This problem is essentially an interval covering problem where we need to cover the entire range `[0, n]` using the minimum number of intervals. Each tap creates an interval that it can water.

The key insight is that this is similar to a "jump game" problem. We want to reach from position `0` to position `n` using the minimum number of "jumps" (taps). 

Think of it this way: if we're standing at any position, we want to know what's the farthest position we can reach from here. For any left endpoint that a tap can cover, we should always choose the tap that extends the farthest to the right. Why? Because this gives us the maximum coverage and potentially reduces the number of taps needed.

The [greedy](/problems/greedy_intro) strategy emerges from this observation:
1. For each position in the garden, we need to know the farthest right position that can be reached by any tap that covers this position
2. We traverse from left to right, and at each position, we check if we can still cover the current position with our current reach
3. When our current coverage is about to end, we need to "activate" a new tap - we choose the one that extends our reach the farthest

The preprocessing step with `last[l] = max(last[l], r)` captures this idea perfectly. For each left endpoint `l` that a tap can cover, we store the maximum right endpoint `r` that can be reached. This way, when we're at position `l`, we immediately know the best tap to use.

The traversal mimics a [greedy](/problems/greedy_intro) jumping strategy: we keep track of the current maximum reach (`mx`) and the previous maximum reach (`pre`). When we reach the end of our previous coverage (`pre == i`), we must use a new tap, incrementing our answer and updating our coverage boundary.

**Learn more about [Greedy](/problems/greedy_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses a [greedy](/problems/greedy_intro) approach with preprocessing to efficiently find the minimum number of taps.

**Step 1: Preprocessing - Building the `last` array**

```python
last = [0] * (n + 1)
for i, x in enumerate(ranges):
    l, r = max(0, i - x), i + x
    last[l] = max(last[l], r)
```

We create an array `last` where `last[i]` stores the farthest right position that can be reached from position `i`. For each tap at position `i` with range `ranges[i]`:
- Calculate the left boundary: `l = max(0, i - ranges[i])` (we use `max(0, ...)` because we can't water negative positions)
- Calculate the right boundary: `r = i + ranges[i]`
- Update `last[l]` to store the maximum right boundary that can be reached from position `l`

This preprocessing ensures that for any position, we know the best tap to use (the one that extends the farthest).

**Step 2: [Greedy](/problems/greedy_intro) Traversal**

```python
ans = mx = pre = 0
for i in range(n):
    mx = max(mx, last[i])
    if mx <= i:
        return -1
    if pre == i:
        ans += 1
        pre = mx
```

We use three variables:
- `ans`: counts the minimum number of taps needed
- `mx`: tracks the farthest position we can reach with the taps we've considered so far
- `pre`: marks the boundary of the previous tap's coverage

The algorithm works as follows:

1. Traverse positions from `0` to `n-1`
2. At each position `i`, update `mx = max(mx, last[i])` to track the farthest reachable position
3. Check if `mx <= i`: This means we can't reach beyond the current position, so the garden cannot be fully watered. Return `-1`
4. Check if `pre == i`: This means we've reached the end of the previous tap's coverage and need to turn on a new tap:
   - Increment `ans` by 1 (we're using a new tap)
   - Update `pre = mx` (the new tap covers up to position `mx`)

The algorithm ensures that we only turn on a new tap when absolutely necessary (when we reach the boundary of our current coverage), and we always choose the tap that extends our reach the farthest. This [greedy](/problems/greedy_intro) choice guarantees the minimum number of taps.

**Time Complexity**: `O(n)` - we iterate through the ranges once for preprocessing and once for the main algorithm.

**Space Complexity**: `O(n)` - for the `last` array.

## Example Walkthrough

Let's walk through a small example with `n = 5` and `ranges = [3, 0, 0, 1, 0, 0]`.

**Step 1: Calculate tap coverage intervals**
- Tap 0 (range 3): waters `[0-3, 0+3] = [0, 3]`
- Tap 1 (range 0): waters `[1-0, 1+0] = [1, 1]`
- Tap 2 (range 0): waters `[2-0, 2+0] = [2, 2]`
- Tap 3 (range 1): waters `[3-1, 3+1] = [2, 4]`
- Tap 4 (range 0): waters `[4-0, 4+0] = [4, 4]`
- Tap 5 (range 0): waters `[5-0, 5+0] = [5, 5]`

**Step 2: Build the `last` array**
Initialize: `last = [0, 0, 0, 0, 0, 0]`

Processing each tap:
- Tap 0: `l=0, r=3` → `last[0] = max(0, 3) = 3`
- Tap 1: `l=1, r=1` → `last[1] = max(0, 1) = 1`
- Tap 2: `l=2, r=2` → `last[2] = max(0, 2) = 2`
- Tap 3: `l=2, r=4` → `last[2] = max(2, 4) = 4`
- Tap 4: `l=4, r=4` → `last[4] = max(0, 4) = 4`
- Tap 5: `l=5, r=5` → `last[5] = max(0, 5) = 5`

Result: `last = [3, 1, 4, 0, 4, 5]`

**Step 3: Greedy traversal**
Initialize: `ans = 0, mx = 0, pre = 0`

- **i = 0**: 
  - `mx = max(0, last[0]) = max(0, 3) = 3`
  - `mx (3) > i (0)` ✓ (can continue)
  - `pre (0) == i (0)` → Turn on tap! `ans = 1`, `pre = 3`

- **i = 1**:
  - `mx = max(3, last[1]) = max(3, 1) = 3`
  - `mx (3) > i (1)` ✓ (can continue)
  - `pre (3) ≠ i (1)` → No new tap needed

- **i = 2**:
  - `mx = max(3, last[2]) = max(3, 4) = 4`
  - `mx (4) > i (2)` ✓ (can continue)
  - `pre (3) ≠ i (2)` → No new tap needed

- **i = 3**:
  - `mx = max(4, last[3]) = max(4, 0) = 4`
  - `mx (4) > i (3)` ✓ (can continue)
  - `pre (3) == i (3)` → Turn on tap! `ans = 2`, `pre = 4`

- **i = 4**:
  - `mx = max(4, last[4]) = max(4, 4) = 4`
  - `mx (4) == i (4)` but we need `mx > i`, so this would fail...

Wait, let me recalculate. Actually, at position 4, we have `mx = max(4, last[4]) = max(4, 4) = 4`. Since `mx (4) ≤ i (4)`, we cannot reach beyond position 4, which means we cannot water position 5. 

Let me use a better example: `n = 5` and `ranges = [3, 4, 1, 1, 0, 0]`.

**Step 1: Calculate tap coverage intervals**
- Tap 0 (range 3): waters `[0, 3]`
- Tap 1 (range 4): waters `[0, 5]` (left bound is `max(0, 1-4) = 0`)
- Tap 2 (range 1): waters `[1, 3]`
- Tap 3 (range 1): waters `[2, 4]`
- Tap 4 (range 0): waters `[4, 4]`
- Tap 5 (range 0): waters `[5, 5]`

**Step 2: Build the `last` array**
- Tap 0: `l=0, r=3` → `last[0] = 3`
- Tap 1: `l=0, r=5` → `last[0] = max(3, 5) = 5`
- Tap 2: `l=1, r=3` → `last[1] = 3`
- Tap 3: `l=2, r=4` → `last[2] = 4`
- Tap 4: `l=4, r=4` → `last[4] = 4`
- Tap 5: `l=5, r=5` → `last[5] = 5`

Result: `last = [5, 3, 4, 0, 4, 5]`

**Step 3: Greedy traversal**
Initialize: `ans = 0, mx = 0, pre = 0`

- **i = 0**: `mx = 5`, `pre == i` → Turn on tap! `ans = 1`, `pre = 5`
- **i = 1**: `mx = 5`, `pre (5) > i (1)` → No new tap needed
- **i = 2**: `mx = 5`, `pre (5) > i (2)` → No new tap needed  
- **i = 3**: `mx = 5`, `pre (5) > i (3)` → No new tap needed
- **i = 4**: `mx = 5`, `pre (5) > i (4)` → No new tap needed

**Result**: Only 1 tap needed (tap at position 1 covers the entire garden `[0, 5]`)

## Python Solution

```python
class Solution:
    def minTaps(self, n: int, ranges: List[int]) -> int:
        # Create an array to store the furthest position that can be reached from each index
        furthest_reach = [0] * (n + 1)
      
        # For each tap, calculate its coverage range and update the furthest reach
        for tap_position, tap_range in enumerate(ranges):
            # Calculate left and right boundaries of current tap's coverage
            left_boundary = max(0, tap_position - tap_range)
            right_boundary = tap_position + tap_range
          
            # Update the furthest position reachable from the left boundary
            furthest_reach[left_boundary] = max(furthest_reach[left_boundary], right_boundary)
      
        # Initialize variables for greedy algorithm
        tap_count = 0  # Number of taps needed
        current_max_reach = 0  # Maximum position reachable in current jump
        previous_max_reach = 0  # End position of the last selected tap
      
        # Iterate through each position in the garden (except the last one)
        for position in range(n):
            # Update the maximum reachable position from current position
            current_max_reach = max(current_max_reach, furthest_reach[position])
          
            # If we cannot reach beyond current position, garden cannot be watered
            if current_max_reach <= position:
                return -1
          
            # If we've reached the end of the previous tap's range, need a new tap
            if previous_max_reach == position:
                tap_count += 1
                previous_max_reach = current_max_reach
      
        return tap_count
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm consists of two main parts:
1. First loop iterates through the `ranges` array once, which has `n+1` elements, taking `O(n)` time. Each iteration performs constant time operations (calculating bounds and updating maximum).
2. Second loop iterates from `0` to `n-1`, performing constant time operations in each iteration, taking `O(n)` time.

Therefore, the overall time complexity is `O(n) + O(n) = O(n)`.

**Space Complexity:** `O(n)`

The algorithm uses an additional array `last` of size `n+1` to store the maximum reachable position from each index. The other variables (`ans`, `mx`, `pre`, `i`, `l`, `r`) use constant space. Thus, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Off-by-One Error in Loop Range

**The Problem:**
A common mistake is iterating through `range(n+1)` instead of `range(n)` in the main greedy loop. This can cause incorrect results or index out of bounds errors.

```python
# INCORRECT - iterating to n instead of n-1
for position in range(n + 1):  # Wrong!
    current_max_reach = max(current_max_reach, furthest_reach[position])
    if current_max_reach <= position:
        return -1
    if previous_max_reach == position:
        tap_count += 1
        previous_max_reach = current_max_reach
```

**Why This Happens:**
The confusion arises because:
- The garden extends from position 0 to n (inclusive)
- We have n+1 taps at positions [0, 1, 2, ..., n]
- However, we only need to check positions 0 to n-1 in our greedy traversal

**The Solution:**
The loop should only go up to `n-1` because:
1. When we reach position `n-1`, we check if we can reach position `n` or beyond
2. If `current_max_reach >= n` at position `n-1`, the entire garden is covered
3. We don't need to check position `n` itself since it's the endpoint

```python
# CORRECT - iterate only to n-1
for position in range(n):  # Correct!
    current_max_reach = max(current_max_reach, furthest_reach[position])
    # ... rest of the logic
```

### Pitfall 2: Forgetting to Handle Edge Case When n = 0

**The Problem:**
When `n = 0`, the garden has no length (just a single point at position 0). The loop `for position in range(0)` won't execute at all, potentially returning 0 taps when we might need 1 tap if the tap at position 0 has range 0.

**The Solution:**
Add a special check for this edge case:

```python
def minTaps(self, n: int, ranges: List[int]) -> int:
    # Edge case: garden with no length
    if n == 0:
        return 0 if ranges[0] >= 0 else -1
  
    # Continue with the main algorithm...
    furthest_reach = [0] * (n + 1)
    # ... rest of the code
```

### Pitfall 3: Incorrect Boundary Calculation for Negative Positions

**The Problem:**
Forgetting to use `max(0, tap_position - tap_range)` and instead using just `tap_position - tap_range` can lead to negative indices:

```python
# INCORRECT - can cause negative index access
left_boundary = tap_position - tap_range  # Wrong!
furthest_reach[left_boundary] = max(furthest_reach[left_boundary], right_boundary)
```

**Why This Matters:**
If a tap at position 1 has range 3, its theoretical left boundary would be -2. Without the `max(0, ...)` check, we'd try to access `furthest_reach[-2]`, causing unexpected behavior.

**The Solution:**
Always clamp the left boundary to 0:

```python
# CORRECT - ensures non-negative indices
left_boundary = max(0, tap_position - tap_range)  # Correct!
```

### Pitfall 4: Misunderstanding When to Increment Tap Count

**The Problem:**
A subtle but critical mistake is incrementing the tap count at the wrong time or checking the wrong condition:

```python
# INCORRECT - checking current_max_reach instead of previous_max_reach
if current_max_reach == position:  # Wrong condition!
    tap_count += 1
    previous_max_reach = current_max_reach
```

**Why This Is Wrong:**
The condition `previous_max_reach == position` means we've exhausted the range of our currently selected taps and must select a new one. Using `current_max_reach == position` would incorrectly trigger tap selection.

**The Solution:**
Check against `previous_max_reach` to know when the current tap's coverage ends:

```python
# CORRECT - check when we've reached the boundary of previous coverage
if previous_max_reach == position:  # Correct!
    tap_count += 1
    previous_max_reach = current_max_reach
```

