# 3459. Length of Longest V-Shaped Diagonal Segment

> Source: https://algo.monster/liteproblems/3459

## Problem Statement

You are given a 2D integer matrix `grid` of size `n x m`, where each element can only be `0`, `1`, or `2`.

The problem asks you to find the length of the longest **V-shaped diagonal segment** in the grid. A V-shaped diagonal segment follows these specific rules:

1. **Must start with the value `1`**: The first element of the segment must be `1`.

2. **Follows a specific sequence**: After the initial `1`, the values must follow the repeating pattern: `2, 0, 2, 0, 2, 0, ...`

3. **Movement along diagonals**: The segment moves along diagonal directions. There are four possible diagonal directions:
   - Top-left to bottom-right (↘)
   - Bottom-right to top-left (↖)
   - Top-right to bottom-left (↙)
   - Bottom-left to top-right (↗)

4. **Can make one turn**: The segment starts moving in one diagonal direction and can optionally make **at most one** 90-degree clockwise turn to continue in another diagonal direction. The sequence pattern must be maintained throughout the entire path, including after the turn.

For example, a valid V-shaped segment could:
- Start at a cell with value `1`
- Move diagonally right-down with values `2, 0, 2, 0...`
- Make a 90-degree clockwise turn
- Continue diagonally left-down with the sequence continuing from where it left off

The length of a V-shaped diagonal segment is the total number of cells it contains, including the starting `1`.

Your task is to find the length of the longest such V-shaped diagonal segment in the entire grid. If no valid V-shaped segment exists (for instance, if there are no `1`s in the grid), return `0`.

## Intuition

To find the longest V-shaped diagonal segment, we need to consider that every valid segment must start with a `1`. This gives us our starting points - we should check every cell containing `1` in the grid as a potential beginning of a V-shaped segment.

From each `1`, we can move in four possible diagonal directions. Since we're looking for the longest segment, we need to explore all four directions from each starting point.

The key insight is that a V-shaped segment has two parts:
1. The initial diagonal path in one direction
2. An optional continuation after making a 90-degree clockwise turn

This naturally leads to a recursive exploration where at each step, we have two choices:
- Continue in the same diagonal direction
- Make a turn (if we haven't already) and continue in the new direction

We need to track our current position `(i, j)`, the direction we're moving `k`, and whether we still have a turn available `cnt`. The sequence pattern (`1, 2, 0, 2, 0, ...`) determines what value we should expect at the next position - if the current cell is `1`, the next should be `2`; if it's `2`, the next should be `0`; if it's `0`, the next should be `2`.

The four diagonal directions can be represented using direction vectors. For a 90-degree clockwise turn, we simply move to the next direction in our circular list of directions: `(k + 1) % 4`.

Since we might revisit the same state multiple times (same position, same direction, same turn availability), [memoization](/problems/memoization_intro) helps us avoid redundant calculations. Each state `(i, j, k, cnt)` will always yield the same maximum length, so we can cache these results.

The overall approach becomes: try starting from every `1` in the grid, explore all four initial directions, and for each path, recursively find the maximum length by considering both continuing straight and making a turn at each valid position.

**Learn more about [Memoization](/problems/memoization_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses memoized depth-first search (DFS) to explore all possible V-shaped diagonal segments starting from each `1` in the grid.

**Core Function Design:**

We implement a recursive function `dfs(i, j, k, cnt)` where:
- `(i, j)` represents the current position in the grid
- `k` indicates the diagonal direction (0-3 for the four diagonal directions)
- `cnt` tracks whether we can still make a turn (1 means we can turn, 0 means we've already turned)

**Direction Representation:**

The four diagonal directions are encoded using a direction array `dirs = (1, 1, -1, -1, 1)`. This clever representation allows us to get direction vectors:
- Direction 0: `(dirs[0], dirs[1])` = `(1, 1)` - moving down-right
- Direction 1: `(dirs[1], dirs[2])` = `(1, -1)` - moving down-left  
- Direction 2: `(dirs[2], dirs[3])` = `(-1, -1)` - moving up-left
- Direction 3: `(dirs[3], dirs[4])` = `(-1, 1)` - moving up-right

**DFS Logic:**

1. **Calculate next position:** From current position `(i, j)` and direction `k`, compute next position as `(x, y) = (i + dirs[k], j + dirs[k + 1])`

2. **Determine expected value:** Based on the sequence pattern `1, 2, 0, 2, 0, ...`:
   - If current cell is `1`, next should be `2`
   - If current cell is `2`, next should be `0`
   - If current cell is `0`, next should be `2`
 
   This is elegantly computed as: `target = 2 if grid[i][j] == 1 else (2 - grid[i][j])`

3. **Boundary and validity check:** If the next position is out of bounds or doesn't contain the expected value, return `0`

4. **Recursive exploration:** We have two options:
   - Continue in the same direction: `dfs(x, y, k, cnt)`
   - Make a 90-degree clockwise turn (if `cnt > 0`): `dfs(x, y, (k + 1) % 4, 0)`
 
   We take the maximum of these options and add 1 for the current cell

**Main Algorithm:**

1. Iterate through the entire grid to find all cells containing `1`
2. For each `1` found at position `(i, j)`:
   - Try all four diagonal directions as starting directions
   - Call `dfs(i, j, k, 1) + 1` for each direction `k` (adding 1 for the starting cell)
   - Track the maximum length found

**[Memoization](/problems/memoization_intro):**

The `@cache` decorator automatically memoizes the `dfs` function results. This prevents redundant calculations when the same state `(i, j, k, cnt)` is encountered multiple times during the search.

**Time Complexity:** `O(m × n × 4 × 2)` = `O(m × n)` where each cell can be visited with at most 4 directions and 2 turn states.

**Space Complexity:** `O(m × n × 4 × 2)` for the [memoization](/problems/memoization_intro) cache plus recursion stack space.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this 4x4 grid:
```
1  2  0  1
0  2  0  2
2  0  2  0
1  2  0  2
```

**Step 1: Find all starting positions**
We scan the grid and find `1`s at positions (0,0), (0,3), and (3,0).

**Step 2: Explore from position (0,0)**

Starting from (0,0) with value `1`, we try all four diagonal directions:

- **Direction 0 (down-right):** 
  - From (0,0), move to (1,1)
  - Expected value: 2 (since current is 1)
  - Grid[1][1] = 2 ✓ Valid!
  - Continue from (1,1), move to (2,2)
  - Expected value: 0 (since current is 2)
  - Grid[2][2] = 2 ✗ Invalid, this path ends
  - Length so far: 1 (start) + 1 (continued) = 2

- **Direction 1 (down-left):**
  - From (0,0), move to (1,-1)
  - Out of bounds! This path ends immediately
  - Length: 1

Let's focus on a more promising path from position (0,3):

**Step 3: Explore from position (0,3) with direction 1 (down-left)**

- Start at (0,3) with value `1`
- Move down-left to (1,2): Expected 2, found 0 ✗ Invalid

**Step 4: Explore from position (0,3) with direction 0 (down-right)**

- Start at (0,3) with value `1`
- Move down-right to (1,4): Out of bounds!

**Step 5: Explore from position (3,0) with direction 0 (down-right)**

- Start at (3,0) with value `1`
- Move down-right to (4,1): Out of bounds!

**Step 6: Explore from position (3,0) with direction 3 (up-right)**

This is where it gets interesting:

- Start at (3,0) with value `1`, can_turn = 1
- Move up-right to (2,1): Expected 2, found 0 ✗ 
- But wait, we can try other directions...

Let's try a path that includes a turn. From position (0,0):

**Direction 0 with a turn:**
- Start at (0,0) with value `1`, direction 0 (down-right), can_turn = 1
- Move to (1,1): Expected 2, found 2 ✓
- From (1,1), we have two choices:

  **Choice A: Continue straight (direction 0)**
  - Move to (2,2): Expected 0, found 2 ✗ Path ends

  **Choice B: Make a 90° clockwise turn to direction 1 (down-left)**
  - Move to (2,0): Expected 0, found 2 ✗ Path ends

The DFS function returns the maximum of these choices plus 1 for the current cell.

**Memoization in action:**
If we later explore from another starting point and reach position (1,1) with direction 0 and can_turn = 1, we don't need to recalculate - we just return the cached result.

**Final Result:**
After exploring all possible V-shaped segments from all starting `1`s, we find the maximum length. In this example, the longest valid V-shaped segment has length 2 (starting from (0,0) going down-right one step).

The key insight is that at each step, we:
1. Check if the next cell matches the expected sequence value
2. If valid, recursively explore both continuing straight and making a turn (if available)
3. Cache results to avoid recalculation
4. Return the maximum path length found

## Python Solution

```python
class Solution:
    def lenOfVDiagonal(self, grid: List[List[int]]) -> int:
        from functools import cache
      
        @cache
        def dfs(row: int, col: int, direction: int, can_change_direction: int) -> int:
            """
            Depth-first search to find the longest diagonal path.
          
            Args:
                row: Current row position
                col: Current column position
                direction: Current direction index (0-3 for 4 diagonal directions)
                can_change_direction: Flag indicating if direction change is allowed (1 or 0)
          
            Returns:
                Length of the longest valid path from the next position
            """
            # Calculate next position based on current direction
            next_row = row + directions[direction]
            next_col = col + directions[direction + 1]
          
            # Determine the target value for the next cell
            # If current cell is 1, target is 2; otherwise adjust based on current value
            if grid[row][col] == 1:
                target_value = 2
            else:
                target_value = 2 - grid[row][col]
          
            # Check if next position is valid and has the target value
            if not (0 <= next_row < rows and 0 <= next_col < cols) or grid[next_row][next_col] != target_value:
                return 0
          
            # Continue in the same direction
            result = dfs(next_row, next_col, direction, can_change_direction)
          
            # If direction change is allowed, try changing direction
            if can_change_direction > 0:
                new_direction = (direction + 1) % 4
                result = max(result, dfs(next_row, next_col, new_direction, 0))
          
            return 1 + result
      
        # Grid dimensions
        rows = len(grid)
        cols = len(grid[0])
      
        # Direction vectors for diagonal movements: SE, SW, NW, NE
        # (row_delta, col_delta) pairs in cyclic order
        directions = (1, 1, -1, -1, 1)
      
        max_length = 0
      
        # Iterate through all cells in the grid
        for i in range(rows):
            for j in range(cols):
                # Start path only from cells with value 1
                if grid[i][j] == 1:
                    # Try all 4 diagonal directions
                    for k in range(4):
                        # Calculate max path length starting from this cell
                        # Add 1 to include the starting cell itself
                        path_length = dfs(i, j, k, 1) + 1
                        max_length = max(max_length, path_length)
      
        return max_length
```

## Time and Space Complexity

The time complexity is `O(m × n)`, where `m` and `n` are the number of rows and columns in the matrix, respectively.

Despite the nested loops and recursive DFS calls, the `@cache` decorator ensures that each unique state `(i, j, k, cnt)` is computed only once. Since there are at most `m × n` positions, 4 directions, and 2 possible values for `cnt` (0 or 1), the total number of unique states is bounded by `O(m × n × 4 × 2) = O(m × n)`. Each state computation takes `O(1)` time excluding recursive calls. The outer loops iterate through all `m × n` cells and try 4 directions, giving `O(m × n × 4) = O(m × n)` iterations. Therefore, the overall time complexity is `O(m × n)`.

The space complexity is `O(m × n)`, where `m` and `n` are the number of rows and columns in the matrix, respectively.

The space is dominated by the memoization cache from the `@cache` decorator, which stores up to `O(m × n × 4 × 2) = O(m × n)` unique states. Additionally, the recursion stack depth in the worst case could be `O(min(m, n))` when traversing diagonally across the grid. Since `O(min(m, n)) ≤ O(m × n)`, the overall space complexity is `O(m × n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Direction Change Logic**

**Pitfall:** A common mistake is implementing the 90-degree clockwise turn incorrectly. The problem specifically requires a clockwise turn, but developers might:
- Allow turning in any direction (not just clockwise)
- Allow multiple turns instead of at most one
- Reset the direction change capability after making a turn

**Example of incorrect implementation:**
```python
# WRONG: Allows turning to any direction
for new_direction in range(4):
    if new_direction != direction:
        result = max(result, dfs(next_row, next_col, new_direction, 0))

# WRONG: Allows multiple turns
result = max(result, dfs(next_row, next_col, (direction + 1) % 4, can_change_direction))
```

**Solution:** Ensure exactly one clockwise turn by using `(direction + 1) % 4` and setting the turn flag to 0 after using it.

### 2. **Sequence Pattern Continuation After Turn**

**Pitfall:** Failing to maintain the sequence pattern after making a turn. The sequence `1, 2, 0, 2, 0, ...` must continue seamlessly even after changing direction. Some might incorrectly restart the pattern after a turn.

**Example scenario:**
- Path goes: `1 → 2 → 0` (moving southeast)
- Makes a turn at the cell with value `0`
- Next cell must be `2` (continuing the pattern), not starting over with `1`

**Solution:** The target value calculation should depend only on the current cell's value, not on whether a turn was made:
```python
target_value = 2 if grid[row][col] == 1 else (2 - grid[row][col])
```

### 3. **Starting Cell Counting**

**Pitfall:** Forgetting to include the starting cell (with value `1`) in the total length count. The DFS function returns the length from the next position, not including the current starting position.

**Wrong approach:**
```python
max_length = max(max_length, dfs(i, j, k, 1))  # Missing the starting cell
```

**Solution:** Always add 1 for the starting cell:
```python
path_length = dfs(i, j, k, 1) + 1
```

### 4. **Memoization State Insufficiency**

**Pitfall:** Not including all necessary state variables in the memoization. If you forget to include the `can_change_direction` parameter in the cache key, the function might return incorrect cached results for states that should be different.

**Wrong approach:**
```python
@cache
def dfs(row, col, direction):  # Missing can_change_direction parameter
    # This would incorrectly treat paths with and without turn capability as the same
```

**Solution:** Include all state variables that affect the computation:
```python
@cache
def dfs(row, col, direction, can_change_direction):
    # Properly distinguishes between states with different turn capabilities
```

### 5. **Edge Case: Single Cell Path**

**Pitfall:** Not handling the case where a cell with value `1` has no valid adjacent cells to continue the path. The function should still return 1 (the length of just the starting cell).

**Solution:** The current implementation handles this correctly by returning `path_length = 0 + 1 = 1` when `dfs` returns 0 for all directions.

