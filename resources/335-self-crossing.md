# 335. Self Crossing

> Source: https://algo.monster/liteproblems/335

## Problem Statement

You start at position `(0, 0)` on a 2D coordinate plane. You're given an array `distance` containing integers that represent how far you move in each step.

The movement pattern follows these directions in order:
- `distance[0]`: Move north (positive y-direction)
- `distance[1]`: Move west (negative x-direction)  
- `distance[2]`: Move south (negative y-direction)
- `distance[3]`: Move east (positive x-direction)
- `distance[4]`: Move north again, and so on...

The direction changes counter-clockwise after each move, cycling through: North → West → South → East → North...

Your task is to determine if the path you trace ever crosses itself. Return `true` if any two line segments of your path intersect (excluding shared endpoints), and `false` otherwise.

For example:
- If `distance = [2, 1, 1, 2]`, you would move 2 units north, 1 unit west, 1 unit south, and 2 units east. This creates a path that crosses itself when the fourth segment intersects with the first segment.
- If `distance = [1, 2, 3, 4]`, the path would not cross itself.

The solution checks for three specific patterns where crossings can occur:
1. The current line crosses the line from 3 steps ago
2. The current line crosses the line from 4 steps ago  
3. The current line crosses the line from 5 steps ago

These are the only possible crossing scenarios due to the counter-clockwise movement pattern, as lines that are 2 steps apart are parallel and cannot cross.

## Intuition

When we trace the path, we need to think about when line segments can possibly intersect. Since we're moving in a counter-clockwise pattern (North → West → South → East → North...), certain lines can never cross each other:

- Lines that are 1 step apart are perpendicular and share an endpoint, so they can't cross
- Lines that are 2 steps apart are parallel (both horizontal or both vertical), so they can't cross

This means the earliest possible crossing is between the current line and the line from 3 steps ago. For example, if we're drawing the 4th line (index 3), it could potentially cross the 1st line (index 0).

Let's visualize the three possible crossing patterns:

**Pattern 1: Current line crosses line from 3 steps ago**
- This happens when we form a shape that curves back on itself after 4 moves
- Condition: `d[i] >= d[i-2]` and `d[i-1] <= d[i-3]`
- The current line is long enough to reach or pass the parallel line from 2 steps ago, AND the previous line is short enough that it doesn't push us too far away

**Pattern 2: Current line crosses line from 4 steps ago**  
- This occurs when we have 5 lines and the path wraps around
- Condition: `d[i-1] == d[i-3]` and `d[i] + d[i-4] >= d[i-2]`
- The lines from 1 and 3 steps ago align perfectly, and the current line plus the line from 4 steps ago can reach across

**Pattern 3: Current line crosses line from 5 steps ago**
- This is the most complex pattern, happening with 6 or more lines
- Multiple conditions must align for the lines to be in positions where they can intersect
- We need to check relative positions and lengths of multiple segments

The key insight is that due to the counter-clockwise movement pattern, these are the ONLY three ways lines can cross. We don't need to check every pair of lines - just these specific relationships. Once we move beyond 5 steps ago, the geometric constraints of the counter-clockwise pattern make it impossible for those older lines to be crossed by the current line without having already triggered one of these three patterns.

## Solution Approach

The implementation iterates through the distance array starting from index 3 (since we need at least 4 lines to have a crossing). For each position `i`, we check the three possible crossing patterns:

**Pattern 1: 4-line crossing (current crosses 3 steps ago)**
```python
if d[i] >= d[i - 2] and d[i - 1] <= d[i - 3]:
    return True
```
This checks if:
- The current line `d[i]` is long enough to reach or exceed the parallel line from 2 steps ago `d[i-2]`
- The previous line `d[i-1]` is short enough compared to the line from 3 steps ago `d[i-3]`

When both conditions are met, the current line will cross the line from 3 steps ago.

**Pattern 2: 5-line crossing (current crosses 4 steps ago)**
```python
if i >= 4 and d[i - 1] == d[i - 3] and d[i] + d[i - 4] >= d[i - 2]:
    return True
```
This requires `i >= 4` (at least 5 lines) and checks if:
- The lines from 1 and 3 steps ago have equal length `d[i-1] == d[i-3]`, creating alignment
- The sum of the current line and the line from 4 steps ago can span across `d[i] + d[i-4] >= d[i-2]`

This forms a specific geometric configuration where the path wraps around and crosses itself.

**Pattern 3: 6-line crossing (current crosses 5 steps ago)**
```python
if (i >= 5 
    and d[i - 2] >= d[i - 4]
    and d[i - 1] <= d[i - 3]
    and d[i] >= d[i - 2] - d[i - 4]
    and d[i - 1] + d[i - 5] >= d[i - 3]):
    return True
```
This requires `i >= 5` (at least 6 lines) and checks multiple conditions:
- `d[i-2] >= d[i-4]`: The line from 2 steps ago is at least as long as the one from 4 steps ago
- `d[i-1] <= d[i-3]`: The previous line is no longer than the one from 3 steps ago
- `d[i] >= d[i-2] - d[i-4]`: The current line is long enough to reach the crossing point
- `d[i-1] + d[i-5] >= d[i-3]`: The combined length creates the crossing opportunity

If none of these three patterns are detected throughout the entire array, the path doesn't cross itself, and we return `false`.

The algorithm runs in `O(n)` time where `n` is the length of the distance array, as we check each position once with constant-time comparisons. The space complexity is `O(1)` since we only use the input array and a few variables.

## Example Walkthrough

Let's trace through the path with `distance = [2, 1, 1, 2]` to see how we detect a crossing.

**Step-by-step path construction:**
- Start at `(0, 0)`
- Move North 2 units: `(0, 0)` → `(0, 2)` — Line 0
- Move West 1 unit: `(0, 2)` → `(-1, 2)` — Line 1  
- Move South 1 unit: `(-1, 2)` → `(-1, 1)` — Line 2
- Move East 2 units: `(-1, 1)` → `(1, 1)` — Line 3

**Checking for crossings:**

When `i = 3` (drawing the 4th line), we check Pattern 1:
- Is `d[3] >= d[1]`? Check: `2 >= 1` ✓
- Is `d[2] <= d[0]`? Check: `1 <= 2` ✓

Both conditions are satisfied! This means Line 3 (the horizontal line from `(-1, 1)` to `(1, 1)`) crosses Line 0 (the vertical line from `(0, 0)` to `(0, 2)`). The intersection occurs at point `(0, 1)`.

The function returns `true` immediately.

---

Now let's verify a non-crossing example with `distance = [1, 2, 3, 4]`:

**Step-by-step path construction:**
- Start at `(0, 0)`
- Move North 1 unit: `(0, 0)` → `(0, 1)` — Line 0
- Move West 2 units: `(0, 1)` → `(-2, 1)` — Line 1
- Move South 3 units: `(-2, 1)` → `(-2, -2)` — Line 2
- Move East 4 units: `(-2, -2)` → `(2, -2)` — Line 3

**Checking for crossings:**

When `i = 3`, check Pattern 1:
- Is `d[3] >= d[1]`? Check: `4 >= 2` ✓
- Is `d[2] <= d[0]`? Check: `3 <= 1` ✗

The second condition fails. Line 2 is too long (3 units) compared to Line 0 (1 unit), which pushes Line 3 too far down to cross Line 0. The lines don't intersect.

Since we only have 4 distances, we can't check Patterns 2 or 3 (which require at least 5 and 6 lines respectively). The function returns `false`.

## Python Solution

```python
class Solution:
    def isSelfCrossing(self, distance: List[int]) -> bool:
        """
        Determines if a path crosses itself when moving in directions: 
        north, west, south, east, repeatedly.
      
        Args:
            distance: List of distances for each move
          
        Returns:
            True if the path crosses itself, False otherwise
        """
      
        # Need at least 4 moves to have a crossing
        for i in range(3, len(distance)):
            # Case 1: 4th line crosses 1st line
            # Current line crosses the line 3 steps back
            # This happens when we move far enough in current direction
            # and the perpendicular distance is small enough
            if (i >= 3 and 
                distance[i] >= distance[i - 2] and 
                distance[i - 1] <= distance[i - 3]):
                return True
          
            # Case 2: 5th line crosses 2nd line
            # When the path forms a specific pattern where lines align
            # and the combined distances create a crossing
            if (i >= 4 and 
                distance[i - 1] == distance[i - 3] and 
                distance[i] + distance[i - 4] >= distance[i - 2]):
                return True
          
            # Case 3: 6th line crosses 3rd line
            # Most complex case involving 6 consecutive moves
            # Multiple conditions must be met for this type of crossing
            if (i >= 5 and
                distance[i - 2] >= distance[i - 4] and
                distance[i - 1] <= distance[i - 3] and
                distance[i] >= distance[i - 2] - distance[i - 4] and
                distance[i - 1] + distance[i - 5] >= distance[i - 3]):
                return True
      
        # No crossing detected
        return False
```

## Time and Space Complexity

**Time Complexity:** `O(n)` where `n` is the length of the distance array. The algorithm iterates through the distance array once with a single for loop starting from index 3 to the end. Within each iteration, it performs constant-time comparisons and arithmetic operations. Even though there are multiple conditional checks (3 different if statements), each check takes `O(1)` time as they only involve accessing array elements at specific indices and performing comparisons. Therefore, the overall time complexity is linear with respect to the input size.

**Space Complexity:** `O(1)`. The algorithm uses only a constant amount of extra space. The variable `d` is just a reference to the input array `distance`, not a copy, so it doesn't consume additional space proportional to the input. The only other variable used is the loop counter `i`, which requires constant space. No additional data structures like arrays, lists, or recursive call stacks are created during execution.

## Common Pitfalls

### 1. **Overlooking the 5-line and 6-line crossing patterns**

Many developers initially focus only on the 4-line crossing pattern (Case 1) because it's the most intuitive - when the current line directly crosses a line from 3 steps ago. However, this misses critical crossing scenarios.

**Why this happens:** The 4-line pattern seems complete because we're checking if perpendicular lines cross. It's easy to assume that checking lines 3 steps apart covers all cases since lines 2 steps apart are parallel.

**The problem:** Without checking 5-line and 6-line patterns, the solution fails on cases like:
- `distance = [1, 1, 2, 1, 1]` - Creates a 5-line crossing
- `distance = [1, 1, 2, 2, 1, 1]` - Creates a 6-line crossing (the sixth line ends exactly on the first line)

**Solution:** Always implement all three crossing patterns. Each represents a geometrically distinct way the path can intersect itself.

### 2. **Incorrect boundary conditions in the 6-line pattern**

The 6-line crossing has the most complex conditions, and getting any comparison operator wrong causes failures.

**Common mistakes:**
- Using `distance[i] > distance[i - 2] - distance[i - 4]` instead of `>=`
- Using `distance[i - 1] + distance[i - 5] > distance[i - 3]` instead of `>=`

**Why this matters:** The equality cases represent valid crossings where lines exactly meet at endpoints. For example, `[1, 1, 2, 2, 1, 1]` satisfies both conditions with equality, and the path touches its starting segment. Missing these edge cases causes the algorithm to return false negatives.

**Solution:** Carefully verify each inequality. Draw out the geometric configuration to understand why each condition needs `>=` rather than `>` for boundary cases.

### 3. **Not checking array bounds properly**

Each pattern requires a minimum number of lines to exist:
- Case 1 needs at least 4 lines (i >= 3)
- Case 2 needs at least 5 lines (i >= 4)  
- Case 3 needs at least 6 lines (i >= 5)

**Common mistake:** Starting the loop at index 0 or checking Case 2/3 without verifying enough elements exist, causing index out of bounds errors.

**Solution:** Start the main loop at index 3 and add explicit guards `i >= 4` and `i >= 5` before checking the respective patterns.

### 4. **Misunderstanding the movement pattern**

**Common misconception:** Assuming the directions are clockwise (North → East → South → West) instead of counter-clockwise (North → West → South → East).

**Why this matters:** The crossing detection logic is built on the specific counter-clockwise pattern. Using the wrong mental model leads to incorrect condition formulation.

**Solution:** Visualize or trace through small examples to internalize the counter-clockwise pattern before implementing the crossing checks.

