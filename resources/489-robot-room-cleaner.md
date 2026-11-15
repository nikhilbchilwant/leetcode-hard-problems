# 489. Robot Room Cleaner

> Source: https://algo.monster/liteproblems/489

## Problem Statement

You need to control a robot to clean all accessible cells in a room. The room is represented as an `m x n` grid where `1` represents an empty cell that needs cleaning and `0` represents a wall that blocks movement.

The challenge is that you don't know:
- The room's layout
- The robot's starting position
- The size of the room

You can only interact with the robot through these four APIs:
- `move()`: Attempts to move the robot forward one cell. Returns `true` if successful (cell is empty), `false` if blocked by a wall
- `turnLeft()`: Rotates the robot 90 degrees counter-clockwise without moving
- `turnRight()`: Rotates the robot 90 degrees clockwise without moving
- `clean()`: Cleans the current cell where the robot is standing

Key constraints:
- The robot starts facing upward
- The robot begins at an empty cell (guaranteed)
- All edges of the grid are surrounded by walls
- You must clean every reachable empty cell in the room

The solution uses a depth-first search (DFS) approach with backtracking. Starting from position `(0, 0)` as a relative coordinate system, the algorithm:

1. Marks the current cell as visited and cleans it
2. Tries to explore all 4 directions (up, right, down, left) by rotating
3. For each direction, if the cell hasn't been visited and the robot can move there, recursively explores that path
4. After exploring a direction, backtracks to the original position and orientation by turning 180 degrees, moving back, then turning 180 degrees again
5. Continues rotating to try the next direction

The `dirs` array `(-1, 0, 1, 0, -1)` represents direction vectors where consecutive pairs form (row_offset, col_offset) for up, right, down, and left movements. The visited set `vis` keeps track of cleaned cells using relative coordinates to avoid revisiting them.

## Intuition

The key insight is that we're essentially blindfolded - we don't know where we are or what the room looks like. This is similar to solving a maze in complete darkness where you can only feel if there's a wall in front of you.

Think of it like being in a dark room with a flashlight that only shows the spot you're standing on. You need to clean every spot you can reach, but you can't see the whole room at once. The natural approach is to:
1. Clean where you're standing
2. Try to move forward
3. If you can move, explore that new area completely
4. When done exploring, come back to where you were
5. Try a different direction

This naturally leads to a depth-first exploration pattern. Why DFS over BFS? Because with the robot's limited API, it's much easier to backtrack one step at a time (going back the way we came) than to jump between different branches of exploration.

The crucial realization is that we need to maintain our orientation. When the robot moves forward and explores a new area, it might turn multiple times. To properly backtrack, we need to:
- Turn 180 degrees (two right turns)
- Move back one step
- Turn 180 degrees again to restore the original orientation

This ensures that when we return from exploring one direction, we're in the exact same position and facing the same way, ready to try the next direction.

We use relative coordinates `(0, 0)` as our starting point since we don't know our absolute position. By tracking visited cells in this relative coordinate system and systematically trying all four directions from each cell, we guarantee that every reachable cell will be cleaned exactly once.

The pattern of trying all 4 directions is achieved by rotating right after each attempt, effectively scanning 360 degrees from each position. This systematic rotation combined with recursive exploration and proper backtracking ensures complete coverage of the room.

## Solution Approach

The implementation uses DFS with backtracking to systematically explore and clean the entire room. Let's break down the key components:

**Data Structures:**
- `vis`: A set storing visited cells as tuples `(i, j)` using relative coordinates
- `dirs`: An array `(-1, 0, 1, 0, -1)` encoding direction vectors where consecutive pairs represent (row_offset, col_offset)

**Direction Encoding:**
The robot faces one of four directions, encoded as:
- `d = 0`: Up (move by `(-1, 0)`)
- `d = 1`: Right (move by `(0, 1)`)
- `d = 2`: Down (move by `(1, 0)`)
- `d = 3`: Left (move by `(0, -1)`)

**Core Algorithm - DFS Function:**

```python
def dfs(i, j, d):
```
Takes current position `(i, j)` and direction `d` the robot is facing.

1. **Mark and Clean:** Add current position to `vis` and clean the cell
   ```python
   vis.add((i, j))
   robot.clean()
   ```

2. **Explore All 4 Directions:** Loop through 4 possible moves
   ```python
   for k in range(4):
       nd = (d + k) % 4
   ```
   - `k = 0`: Try current direction
   - `k = 1`: Try after one right turn
   - `k = 2`: Try after two right turns
   - `k = 3`: Try after three right turns

3. **Calculate Next Position:**
   ```python
   x, y = i + dirs[nd], j + dirs[nd + 1]
   ```
   Using the direction index `nd` to get the appropriate offset from `dirs`.

4. **Move and Explore:** If cell is unvisited and movement succeeds:
   ```python
   if (x, y) not in vis and robot.move():
       dfs(x, y, nd)
   ```
   Recursively explore from the new position with updated direction.

5. **Backtrack:** After exploring, return to original position:
   ```python
   robot.turnRight()
   robot.turnRight()
   robot.move()
   robot.turnRight()
   robot.turnRight()
   ```
   - First two right turns: Face opposite direction (180°)
   - Move back one cell
   - Two more right turns: Restore original orientation

6. **Rotate for Next Direction:**
   ```python
   robot.turnRight()
   ```
   Turn right to try the next direction in the loop.

**Initialization:**
Start DFS from relative position `(0, 0)` facing up (direction `0`):
```python
dfs(0, 0, 0)
```

The algorithm guarantees that every reachable cell is visited exactly once through the combination of:
- Systematic exploration of all 4 directions from each cell
- Tracking visited cells to avoid redundant cleaning
- Proper backtracking to maintain position and orientation consistency

## Example Walkthrough

Let's walk through a small example to illustrate how the DFS with backtracking solution works. Consider a 3x3 room of empty cells, surrounded by walls, where the robot starts at the center:

```
Grid (unknown to us):     Robot's view:
# # # # #                 Just knows it can
# . . . #                 move or hit walls
# . R . #
# . . . #                 R = Robot start
# # # # #                 . = Empty cell
                          # = Wall
```

The robot starts at what we'll call position `(0, 0)` in our relative coordinate system, facing up (direction `d = 0`).

**Step 1: Initial Position (0, 0)**
- Mark `(0, 0)` as visited: `vis = {(0, 0)}`
- Clean current cell
- Try 4 directions by rotating:
  - **Direction 0 (Up)**: Calculate next position `(-1, 0)`. Not visited. Try `move()` → returns `true`
  - Move to `(-1, 0)`

**Step 2: Position (-1, 0)**
- Mark `(-1, 0)` as visited: `vis = {(0, 0), (-1, 0)}`
- Clean current cell
- Try 4 directions:
  - **Direction 0 (Up)**: Try `(-2, 0)`. `move()` → returns `false` (wall)
  - Turn right
  - **Direction 1 (Right)**: Try `(-1, 1)`. Not visited. `move()` → returns `true`
  - Move to `(-1, 1)`

**Step 3: Position (-1, 1)** (facing Right)
- Mark and clean `(-1, 1)`
- **Right**: `(-1, 2)` is a wall, `move()` → `false`. Turn right
- **Down**: `(0, 1)` is not visited. `move()` → `true`. Move to `(0, 1)`

**Step 4: Following the outer ring**
Each new cell is entered with the robot facing the direction it moved, and the first direction it tries is that same direction. This makes the robot circle the outer ring clockwise:
- At `(0, 1)` facing Down: `(1, 1)` is open → move
- At `(1, 1)` facing Down: `(2, 1)` is a wall; turn right to face Left; `(1, 0)` is open → move
- At `(1, 0)` facing Left: `(1, -1)` is open → move
- At `(1, -1)` facing Left: `(1, -2)` is a wall; turn right to face Up; `(0, -1)` is open → move
- At `(0, -1)` facing Up: `(-1, -1)` is open → move

**Step 5: Position (-1, -1)** (facing Up)
- Mark and clean `(-1, -1)`
- Up `(-2, -1)` and Left `(-1, -2)` are walls; Down `(0, -1)` and Right `(-1, 0)` are visited
- All four directions are done, so the call returns and the caller runs `backtrack()`: turn 180°, move back to `(0, -1)`, turn 180° to face Up again

**Step 6: Unwinding the recursion**
- Each earlier call resumes its loop, turns right, and checks its remaining directions
- Every remaining neighbor is either a wall or already in `vis`, so no new moves happen
- Each call finishes and backtracks one cell, until the robot is back at `(0, 0)` facing its original direction

All 9 empty cells are cleaned, in this order: `(0, 0)`, `(-1, 0)`, `(-1, 1)`, `(0, 1)`, `(1, 1)`, `(1, 0)`, `(1, -1)`, `(0, -1)`, `(-1, -1)`.

The key insights from this walkthrough:
1. We systematically try all 4 directions from each position by rotating
2. We use relative coordinates since we don't know our absolute position
3. After exploring each branch, we carefully backtrack to maintain our position and orientation
4. The visited set prevents us from cleaning the same cell twice
5. The combination of DFS and systematic rotation ensures complete coverage

## Python Solution

```python
class Solution:
    def cleanRoom(self, robot: 'Robot') -> None:
        """
        Clean all reachable cells in the room using DFS exploration.

        :type robot: Robot
        :rtype: None
        """

        def backtrack():
            """Move the robot back to its previous cell and restore original direction."""
            robot.turnRight()
            robot.turnRight()
            robot.move()
            robot.turnRight()
            robot.turnRight()

        def depth_first_search(row: int, col: int, direction: int) -> None:
            """
            Recursively explore and clean the room using DFS.

            Args:
                row: Current row position (relative to starting point)
                col: Current column position (relative to starting point)
                direction: Current facing direction (0: up, 1: right, 2: down, 3: left)
            """
            # Mark current cell as visited and clean it
            visited_cells.add((row, col))
            robot.clean()

            # Try all 4 directions (up, right, down, left)
            for turn_count in range(4):
                # Calculate new direction after turning right 'turn_count' times
                new_direction = (direction + turn_count) % 4

                # Calculate next cell coordinates based on new direction
                next_row = row + direction_deltas[new_direction]
                next_col = col + direction_deltas[new_direction + 1]

                # If the next cell hasn't been visited and robot can move there
                if (next_row, next_col) not in visited_cells and robot.move():
                    # Recursively explore from the new cell
                    depth_first_search(next_row, next_col, new_direction)
                    # Backtrack: return to current cell facing original direction
                    backtrack()

                # Turn right to try next direction
                robot.turnRight()

        # Direction deltas for (row, col) in order: up, right, down, left
        # Using a circular array pattern where index i gives row delta, i+1 gives col delta
        direction_deltas = (-1, 0, 1, 0, -1)

        # Set to track visited cells (relative coordinates)
        visited_cells = set()

        # Start DFS from origin (0, 0) facing up (direction 0)
        depth_first_search(0, 0, 0)
```

## Time and Space Complexity

**Time Complexity: `O(N - M)`** where `N` is the total number of cells in the room and `M` is the number of obstacles.

The algorithm uses DFS to explore all accessible cells in the room. Each accessible cell is visited exactly once due to the visited set tracking. For each cell visit, the robot:
- Cleans the cell: `O(1)`
- Attempts to move in 4 directions: `O(4)` = `O(1)`
- For each direction, performs constant operations (turning, checking, potentially moving and backtracking)

Since we visit each accessible cell once and perform constant work at each cell, the total time complexity is `O(N - M)`, which represents the number of cleanable cells.

**Space Complexity: `O(N - M)`**

The space complexity consists of:
- **Visited Set (`vis`)**: Stores coordinates of all visited cells. In the worst case, all accessible cells are visited, requiring `O(N - M)` space.
- **Recursion Stack**: The DFS recursion can go as deep as the number of accessible cells in the worst case (imagine a long winding path). This also requires `O(N - M)` space in the call stack.
- **Other Variables**: The `dirs` array and other local variables use `O(1)` space.

Therefore, the total space complexity is `O(N - M)` for storing visited cells and maintaining the recursion stack.

## Common Pitfalls

### 1. **Incorrect Backtracking Implementation**
One of the most common mistakes is implementing the backtracking incorrectly, which causes the robot to lose its position or orientation.

**Pitfall Example:**
```python
# Wrong: Only turning once or forgetting to restore orientation
def backtrack():
    robot.turnRight()
    robot.move()
    robot.turnRight()  # Missing another turnRight()
```

**Why It Fails:** After exploring a path, the robot must return to exactly the same position AND orientation it had before moving. The robot needs to:
1. Turn 180° to face the opposite direction
2. Move back one cell
3. Turn 180° again to restore the original facing direction

**Correct Implementation:**
```python
def backtrack():
    robot.turnRight()  # Turn 90°
    robot.turnRight()  # Turn another 90° (now facing opposite)
    robot.move()       # Move back to previous cell
    robot.turnRight()  # Turn 90°
    robot.turnRight()  # Turn another 90° (restored original direction)
```

### 2. **Forgetting to Turn Right After Each Direction Attempt**
Another critical mistake is forgetting to turn right after checking each direction in the loop.

**Pitfall Example:**
```python
for k in range(4):
    nd = (d + k) % 4
    x, y = i + dirs[nd], j + dirs[nd + 1]
    if (x, y) not in vis and robot.move():
        dfs(x, y, nd)
        backtrack()
    # Missing robot.turnRight() here!
```

**Why It Fails:** The algorithm assumes that for each iteration `k`, the robot has already turned right `k` times from its initial direction. Without the turn at the end of each iteration, the robot's actual orientation won't match the calculated direction `nd`.

**Correct Implementation:**
```python
for k in range(4):
    nd = (d + k) % 4
    x, y = i + dirs[nd], j + dirs[nd + 1]
    if (x, y) not in vis and robot.move():
        dfs(x, y, nd)
        backtrack()
    robot.turnRight()  # Essential: Turn right to try next direction
```

### 3. **Incorrect Direction Vector Calculation**
Using the wrong indices when accessing the direction deltas array.

**Pitfall Example:**
```python
# Wrong: Using nd twice instead of nd and nd+1
x, y = i + dirs[nd], j + dirs[nd]  # This would use the same value for both
```

**Why It Fails:** The `dirs` array stores alternating row and column offsets. For direction `nd`, the row offset is at index `nd` and the column offset is at index `nd + 1`.

**Correct Implementation:**
```python
x, y = i + dirs[nd], j + dirs[nd + 1]
```

### 4. **Not Maintaining Relative Coordinate System**
Starting with absolute coordinates or trying to determine the actual room layout.

**Pitfall Example:**
```python
# Wrong: Trying to find actual starting position
def dfs(i, j, d):
    # Attempting to map to actual room coordinates
    actual_i = i + start_row  # start_row is unknown!
    actual_j = j + start_col  # start_col is unknown!
```

**Why It Fails:** Since we don't know the robot's actual starting position in the room, we must use a relative coordinate system where the starting position is arbitrarily set as (0, 0).

**Correct Approach:**
Always work with relative coordinates where the starting position is (0, 0), regardless of where the robot actually starts in the room.

