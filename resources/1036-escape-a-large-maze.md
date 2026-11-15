# 1036. Escape a Large Maze

> Source: https://algo.monster/liteproblems/1036

## Problem Statement

You have a very large grid with dimensions 1 million by 1 million on an XY-plane. Each square in the grid has coordinates `(x, y)`.

You need to determine if you can travel from a starting position `source = [sx, sy]` to a destination position `target = [tx, ty]`. However, there are some blocked squares that you cannot pass through, given in an array `blocked` where each `blocked[i] = [xi, yi]` represents a square at coordinates `(xi, yi)` that is impassable.

Movement rules:
- From any square, you can move one step in four directions: north (up), east (right), south (down), or west (left)
- You cannot move to a blocked square
- You cannot move outside the grid boundaries (coordinates must stay between 0 and 999,999)

The task is to return `true` if there exists at least one valid path from the source to the target following these movement rules, and `false` otherwise.

## Intuition

The key insight comes from understanding what the blocked squares can actually do in such a massive grid. With a 1 million × 1 million grid and only up to 200 blocked squares, it's impossible for the blocked squares to create a wall that spans the entire grid.

Think about it this way: if the blocked squares try to form a barrier to trap either the source or target point, what's the maximum area they can enclose? Imagine the most efficient use of blocked squares to create an enclosure - this would be forming something like a diagonal line from the edge of the grid. With `n` blocked squares, the maximum area that can be trapped is approximately `n^2 / 2`.

This leads to a clever observation: if we start from the source point and can reach more than `len(blocked)^2 / 2` unique squares, we must have "escaped" any potential enclosure formed by the blocked squares. Why? Because if we were trapped, we couldn't possibly visit more squares than the maximum area that can be enclosed.

But there's another scenario to consider: what if both source and target are inside the same enclosure? They could reach each other but both would be trapped. Or what if they're in different enclosures? They couldn't reach each other even though neither is technically "stuck" in a small area.

This is why we need to check from both directions:
1. Can the source "escape" (reach enough squares) OR reach the target directly?
2. Can the target "escape" (reach enough squares) OR reach the source directly?

If both checks pass, it means either:
- Both points are free (not enclosed), so they can definitely reach each other
- Both points are in the same enclosure (since they can reach each other during the DFS)

The beauty of this approach is that we don't need to explore the entire million × million grid. We only need to explore up to `len(blocked)^2 / 2` squares from each starting point, making the solution feasible despite the enormous grid size.

**Learn more about [Depth-First Search](/problems/dfs_intro) and [Breadth-First Search](/problems/bfs_intro) patterns.**

## Solution Approach

The implementation uses DFS ([Depth-First Search](/problems/dfs_intro)) with a clever early termination condition based on the mathematical insight about the maximum area that can be trapped.

**Key Data Structures:**
- `s`: A set containing all blocked positions as tuples for O(1) lookup
- `vis`: A visited set to track explored squares during each DFS
- `dirs`: Direction array `(-1, 0, 1, 0, -1)` used with `pairwise` to generate the four movement directions

**DFS Function Logic:**

The `dfs` function takes a source position, target position, and a visited set. Here's how it works:

1. **Mark Current Position**: Add the current source position to the visited set as a tuple
 
2. **Escape Check**: If `len(vis) > m` where `m = len(blocked)^2 / 2`, return `True`. This means we've explored enough squares to guarantee we're not trapped in an enclosure.

3. **Explore Neighbors**: For each of the four directions (north, east, south, west):
   - Calculate new coordinates: `x, y = source[0] + a, source[1] + b`
   - Check if the new position is:
     - Within grid bounds: `0 <= x < n and 0 <= y < n` where `n = 10^6`
     - Not blocked: `(x, y) not in s`
     - Not visited: `(x, y) not in vis`
 
4. **Target Check**: If the new position equals the target, return `True` (path found)

5. **Recursive Exploration**: Recursively call `dfs` from the new position. If it returns `True`, propagate the success.

6. **No Path Found**: If all directions are exhausted without success, return `False`

**Bidirectional Check:**

The solution performs DFS from both directions:
```python
return dfs(source, target, set()) and dfs(target, source, set())
```

This ensures that:
- The source can either escape its potential enclosure or reach the target
- The target can either escape its potential enclosure or reach the source

Both conditions must be true for a valid path to exist. This handles all cases:
- Both points are free (not enclosed) → both DFS calls succeed
- Both points are in the same enclosure → each can reach the other
- Points are in different enclosures → at least one DFS fails
- One point is enclosed, the other is free → the enclosed point's DFS fails

The algorithm's efficiency comes from the early termination - we never need to explore more than `O(len(blocked)^2)` squares, making it practical despite the grid's enormous size.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach. Imagine we have a much smaller 10×10 grid (instead of 1 million × 1 million) to visualize the concept.

**Example Setup:**
- Grid size: 10×10 (coordinates from 0 to 9)
- Source: `[1, 1]`
- Target: `[8, 8]`
- Blocked squares: `[[0,2], [1,2], [2,2], [2,1], [2,0]]` (forming an L-shaped barrier)

```
  0 1 2 3 4 5 6 7 8 9
0 . . X . . . . . . .
1 . S X . . . . . . .
2 X X X . . . . . . .
3 . . . . . . . . . .
4 . . . . . . . . . .
5 . . . . . . . . . .
6 . . . . . . . . . .
7 . . . . . . . . . .
8 . . . . . . . . T .
9 . . . . . . . . . .

S = Source, T = Target, X = Blocked
```

**Step 1: Calculate Maximum Trapped Area**
- Number of blocked squares: 5
- Maximum area that can be trapped: `5^2 / 2 = 12.5` → we'll use 12
- If either DFS can visit more than 12 squares, that point has "escaped"

**Step 2: DFS from Source [1,1]**

Starting from `[1,1]`, the DFS explores:
1. Visit `[1,1]` → visited set: `{(1,1)}`
2. Try north `[0,1]` → valid, visit it → visited: `{(1,1), (0,1)}`
3. From `[0,1]`, try west `[0,0]` → valid, visit → visited: `{(1,1), (0,1), (0,0)}`
4. Continue exploring...

The DFS would find it's blocked by the L-shaped barrier on certain sides but can escape to the east:
- Path discovered: `[1,1] → [1,0] → [0,0] → [0,1] → ... → [3,1] → [4,1] → ...`
- After visiting 13 squares (more than our threshold of 12), the function returns `True` - the source has escaped!

**Step 3: DFS from Target [8,8]**

Starting from `[8,8]`, the DFS explores freely as there are no nearby blocked squares:
1. Visit `[8,8]` → visited set: `{(8,8)}`
2. Try north `[7,8]` → valid, visit
3. Try east `[8,9]` → valid, visit
4. Continue exploring...

The target quickly reaches 13+ squares since it's in open space, returning `True`.

**Step 4: Final Result**

Both DFS calls return `True`:
- Source DFS: `True` (escaped the potential enclosure)
- Target DFS: `True` (was never enclosed)

Since both are true, the function returns `True` - there exists a path from source to target.

**Alternative Scenario - Same Enclosure:**

If the blocked squares formed a complete circle around both source and target:
```
Blocked: forms a 5×5 box around both points
Source: [3,3]
Target: [4,4]

. . . . . . .
. X X X X X .
. X . . . X .
. X . S . X .
. X . . T X .
. X X X X X .
. . . . . . .
```

Here:
- Source DFS would find the target directly before hitting the escape threshold → returns `True`
- Target DFS would find the source directly before hitting the escape threshold → returns `True`
- Result: `True` (they can reach each other even though both are trapped)

This example demonstrates how the bidirectional check ensures we correctly identify whether a path exists, regardless of whether the points are enclosed or free.

## Python Solution

```python
class Solution:
    def isEscapePossible(
        self, blocked: List[List[int]], source: List[int], target: List[int]
    ) -> bool:
        """
        Determines if it's possible to reach target from source in a 10^6 x 10^6 grid
        with certain cells blocked.
      
        The key insight: if blocked cells form an enclosure, the maximum area they can
        enclose is approximately (len(blocked))^2 / 2 (when forming a diagonal barrier
        against a corner). If we can explore more cells than this, we've escaped any
        potential enclosure.
      
        Args:
            blocked: List of blocked cell coordinates
            source: Starting position [row, col]
            target: Target position [row, col]
          
        Returns:
            True if target is reachable from source, False otherwise
        """
      
        def can_escape_or_reach(start_pos: List[int], end_pos: List[int], visited: set) -> bool:
            """
            Performs DFS to check if we can either:
            1. Reach the end_pos from start_pos, or
            2. Explore enough cells to prove we're not enclosed
          
            Args:
                start_pos: Current starting position
                end_pos: Target position to reach
                visited: Set of already visited cells
              
            Returns:
                True if we can reach end_pos or escape enclosure
            """
            # Mark current position as visited
            visited.add(tuple(start_pos))
          
            # If we've explored more cells than the maximum possible enclosed area,
            # we've definitely escaped any enclosure
            if len(visited) > max_enclosed_area:
                return True
          
            # Explore all four directions (up, right, down, left)
            for i in range(4):
                # Calculate next position using direction vectors
                next_row = start_pos[0] + directions[i]
                next_col = start_pos[1] + directions[i + 1]
              
                # Check if next position is valid:
                # 1. Within grid boundaries
                # 2. Not blocked
                # 3. Not visited
                if (0 <= next_row < grid_size and 
                    0 <= next_col < grid_size and 
                    (next_row, next_col) not in blocked_set and 
                    (next_row, next_col) not in visited):
                  
                    # Check if we've reached the target
                    if [next_row, next_col] == end_pos:
                        return True
                  
                    # Continue DFS from the next position
                    if can_escape_or_reach([next_row, next_col], end_pos, visited):
                        return True
          
            return False
      
        # Convert blocked list to set for O(1) lookup
        blocked_set = {(row, col) for row, col in blocked}
      
        # Direction vectors for moving up, right, down, left
        # Using the pattern: (row_delta, col_delta) pairs
        directions = [-1, 0, 1, 0, -1]  # Creates pairs: (-1,0), (0,1), (1,0), (0,-1)
      
        # Grid size (10^6 x 10^6)
        grid_size = 10**6
      
        # Maximum area that can be enclosed by blocked cells
        # This is approximately (number of blocked cells)^2 / 2
        # This represents the worst-case scenario where blocked cells form a diagonal
        max_enclosed_area = len(blocked) ** 2 // 2
      
        # Check both directions:
        # 1. Can we reach target from source?
        # 2. Can we reach source from target?
        # Both must be true for a valid path to exist
        return (can_escape_or_reach(source, target, set()) and 
                can_escape_or_reach(target, source, set()))
```

## Time and Space Complexity

**Time Complexity:** `O(m)` where `m = len(blocked)^2 / 2`

The algorithm performs DFS from both source and target positions. The key insight is that the DFS is bounded by the condition `len(vis) > m`. Since `m = len(blocked)^2 / 2`, and there are at most 200 blocked cells, we have `m ≤ 200^2 / 2 = 20,000`.

For each DFS call:
- The recursion explores at most `m + 1` cells before returning `True` (when `len(vis) > m`)
- Each cell is visited at most once due to the `vis` set
- For each visited cell, we check 4 directions, which is `O(1)`
- Checking if a position is in the blocked set `s` or visited set `vis` takes `O(1)` average time for hash set operations

Since we run DFS twice (once from source, once from target), the total time complexity is `O(2m) = O(m)`.

**Space Complexity:** `O(m)` where `m = len(blocked)^2 / 2`

The space is used for:
- The blocked set `s`: `O(len(blocked))` which is at most `O(200)`
- The visited set `vis`: `O(m)` as it stores at most `m + 1` positions
- The recursion call stack: `O(m)` in the worst case when DFS goes `m` levels deep

The dominant term is `O(m)`, so the overall space complexity is `O(m)` where `m ≤ 20,000`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Maximum Enclosed Area Calculation**

**Pitfall:** Using the wrong formula for the maximum area that blocked cells can enclose. A common mistake is assuming blocked cells can form a square perimeter, leading to an incorrect bound like `4 * len(blocked)` or `len(blocked)^2`.

**Why it happens:** The maximum enclosed area occurs when blocked cells form a diagonal barrier against a corner of the grid, creating a triangular region. This gives approximately `len(blocked)^2 / 2` cells.

**Solution:** Use the correct formula:
```python
max_enclosed_area = len(blocked) ** 2 // 2
```

### 2. **Not Checking Both Directions**

**Pitfall:** Only checking if source can reach target, without verifying the reverse direction:
```python
# Incorrect: Only one direction
return can_escape_or_reach(source, target, set())
```

**Why it fails:** Consider this scenario:
- Source is trapped in a small enclosure
- Target is free in the open grid
- The forward DFS from source will quickly return `True` after exploring the small enclosed area (exceeding the threshold)
- But source and target are actually unreachable from each other

**Solution:** Always check both directions:
```python
return (can_escape_or_reach(source, target, set()) and 
        can_escape_or_reach(target, source, set()))
```

### 3. **Using List Instead of Set for Blocked Positions**

**Pitfall:** Keeping blocked positions in a list and checking membership with `in`:
```python
# Inefficient
if [next_row, next_col] in blocked:  # O(n) lookup
```

**Why it's problematic:** Each lookup takes O(n) time where n is the number of blocked cells. With potentially thousands of lookups during DFS, this creates a significant performance bottleneck.

**Solution:** Convert to a set of tuples for O(1) lookups:
```python
blocked_set = {(row, col) for row, col in blocked}
# Then check:
if (next_row, next_col) not in blocked_set:  # O(1) lookup
```

### 4. **Forgetting to Handle Edge Cases**

**Pitfall:** Not considering special cases like:
- Source and target are the same position
- Source or target is in the blocked list
- Empty blocked list

**Solution:** Add validation at the beginning:
```python
def isEscapePossible(self, blocked, source, target):
    # Handle edge cases
    if source == target:
        return True
    if not blocked:
        return True
  
    blocked_set = {(row, col) for row, col in blocked}
    if tuple(source) in blocked_set or tuple(target) in blocked_set:
        return False
  
    # Continue with main algorithm...
```

### 5. **Stack Overflow with Deep Recursion**

**Pitfall:** Using recursive DFS on a potentially large search space can cause stack overflow, especially in Python which has a default recursion limit.

**Why it happens:** In worst case, DFS might need to explore up to `len(blocked)^2 / 2` cells, which could be 20,000 cells for 200 blocked squares.

**Solution:** Either increase recursion limit or use iterative DFS:
```python
def can_escape_or_reach_iterative(start_pos, end_pos):
    stack = [start_pos]
    visited = set()
  
    while stack:
        curr = stack.pop()
        if curr == end_pos:
            return True
          
        visited.add(tuple(curr))
        if len(visited) > max_enclosed_area:
            return True
          
        for i in range(4):
            next_row = curr[0] + directions[i]
            next_col = curr[1] + directions[i + 1]
          
            if (0 <= next_row < grid_size and 
                0 <= next_col < grid_size and 
                (next_row, next_col) not in blocked_set and 
                (next_row, next_col) not in visited):
                stack.append([next_row, next_col])
  
    return False
```

