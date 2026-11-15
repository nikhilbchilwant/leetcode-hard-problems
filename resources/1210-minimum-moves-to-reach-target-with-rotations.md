# 1210. Minimum Moves to Reach Target with Rotations

> Source: https://algo.monster/liteproblems/1210

## Problem Statement

You have an `n x n` grid where a snake occupies exactly 2 cells. The snake starts at the top-left corner, occupying cells `(0, 0)` and `(0, 1)`. The grid contains:
- Empty cells (represented by `0`) that the snake can move through
- Blocked cells (represented by `1`) that the snake cannot pass

The snake needs to reach the target position at the bottom-right corner, occupying cells `(n-1, n-2)` and `(n-1, n-1)`.

The snake can perform four types of moves:

1. **Move Right**: The snake moves one cell to the right while maintaining its orientation (horizontal or vertical). Both cells of the snake must move into empty cells.

2. **Move Down**: The snake moves one cell down while maintaining its orientation. Both cells of the snake must move into empty cells.

3. **Rotate Clockwise** (only when horizontal): If the snake is in a horizontal position at `(r, c)` and `(r, c+1)`, and both cells directly below (`(r+1, c)` and `(r+1, c+1)`) are empty, the snake can rotate clockwise. After rotation, the snake occupies `(r, c)` and `(r+1, c)`, changing from horizontal to vertical orientation.

4. **Rotate Counterclockwise** (only when vertical): If the snake is in a vertical position at `(r, c)` and `(r+1, c)`, and both cells to the right (`(r, c+1)` and `(r+1, c+1)`) are empty, the snake can rotate counterclockwise. After rotation, the snake occupies `(r, c)` and `(r, c+1)`, changing from vertical to horizontal orientation.

Each action counts as one move. You need to find the minimum number of moves required for the snake to reach the target position. If it's impossible to reach the target, return `-1`.

## Intuition

The key insight is to think of this problem as finding the shortest path in a state space. Each state represents not just where the snake is, but also its orientation (horizontal or vertical). This is crucial because the same position with different orientations represents different states that may require different moves to reach the target.

Why do we need to track orientation? Consider that a snake at position `(r, c)` and `(r, c+1)` (horizontal) has different movement options than a snake at `(r, c)` and `(r+1, c)` (vertical). The horizontal snake can rotate clockwise if there's space below, while the vertical snake can rotate counterclockwise if there's space to the right.

Since each move has equal cost (exactly 1 move), we're dealing with an unweighted graph problem. This immediately suggests BFS as the optimal approach because BFS explores states level by level - all states reachable in 1 move, then 2 moves, then 3 moves, and so on. The first time we encounter the target state, we're guaranteed to have found it via the minimum number of moves.

To represent the snake's position efficiently, we can use the indices of its two cells. Since the grid is `n x n`, we can flatten the 2D coordinates into 1D indices: a cell at `(i, j)` becomes index `i * n + j`. This makes it easier to store and compare positions.

The visited set needs to store both position and orientation to avoid revisiting the same state. We represent this as `(tail_position, status)` where `status` is 0 for horizontal and 1 for vertical. This prevents us from exploring the same configuration multiple times, which would lead to inefficiency or infinite loops.

The four possible moves (right, down, clockwise rotation, counterclockwise rotation) are systematically checked for each state. The rotation moves have additional constraints - we need to check that the cells the snake will rotate through are empty, not just the final destination cells.

**Learn more about [Breadth-First Search](/problems/bfs_intro) patterns.**

## Solution Approach

The solution implements BFS using a queue to explore all possible states of the snake. Let's walk through the implementation details:

**State Representation:**
- Each snake position is represented by two indices `(a, b)` where `a` is the tail position and `b` is the head position in the flattened 1D array
- For a cell at `(i, j)` in the 2D grid, its 1D index is `i * n + j`
- The snake's orientation is tracked separately: `status = 0` for horizontal, `status = 1` for vertical

**Data Structures:**
- **Queue `q`**: Stores current positions as tuples `(a, b)`. Initially contains `(0, 1)` representing the starting position
- **Set `vis`**: Tracks visited states as `(tail_position, status)` to avoid revisiting the same configuration. Initially contains `(0, 0)` for the starting horizontal state
- **Target**: Fixed at `(n*n - 2, n*n - 1)`, representing the last two cells of the grid

**The `move` Helper Function:**
This function validates and adds new positions to the queue:
1. Checks if both cells `(i1, j1)` and `(i2, j2)` are within bounds
2. Converts 2D coordinates to 1D indices: `a = i1 * n + j1`, `b = i2 * n + j2`
3. Determines orientation: `status = 0` if `i1 == i2` (horizontal), else `status = 1` (vertical)
4. If the state `(a, status)` hasn't been visited and both cells are empty (`grid[i][j] == 0`), adds the position to queue and marks as visited

**BFS Exploration:**
For each position `(a, b)` dequeued:

1. **Check Target**: If current position equals target `(n*n - 2, n*n - 1)`, return the current move count `ans`

2. **Convert to 2D**: Extract coordinates from 1D indices:
   - `i1, j1 = a // n, a % n` (tail)
   - `i2, j2 = b // n, b % n` (head)

3. **Try All Moves**:
   - **Move Right**: `move(i1, j1 + 1, i2, j2 + 1)` - both cells shift right
   - **Move Down**: `move(i1 + 1, j1, i2 + 1, j2)` - both cells shift down
   - **Clockwise Rotation** (if horizontal): When `i1 == i2`, check if `grid[i1 + 1][j2] == 0` (cell below head is empty), then `move(i1, j1, i1 + 1, j1)` rotates tail down
   - **Counterclockwise Rotation** (if vertical): When `j1 == j2`, check if `grid[i2][j1 + 1] == 0` (cell right of head is empty), then `move(i1, j1, i1, j1 + 1)` rotates tail right

4. **Level Tracking**: After processing all positions at current distance, increment `ans` by 1

The algorithm continues until either the target is reached (return `ans`) or the queue is empty (return `-1` indicating no path exists).

The time complexity is `O(n²)` as we can visit at most `2n²` states (each cell with two orientations), and space complexity is also `O(n²)` for the visited set and queue.

## Example Walkthrough

Let's walk through a small example with a 3x3 grid:

```
Grid:
[0, 0, 0]
[0, 0, 0]  
[0, 0, 0]
```

The snake starts at cells (0,0) and (0,1) horizontally, and needs to reach cells (2,1) and (2,2).

**Initial State:**
- Queue: `[(0, 1)]` (using 1D indices where 0 = cell(0,0), 1 = cell(0,1))
- Visited: `{(0, 0)}` (tail at index 0, status 0 for horizontal)
- Moves: 0

**Move 1 - Process initial position (0, 1):**
- Convert to 2D: tail at (0,0), head at (0,1)
- Try all possible moves:
  - Right: Move to (0,1) and (0,2) → Add `(1, 2)` to queue
  - Down: Move to (1,0) and (1,1) → Add `(3, 4)` to queue  
  - Clockwise rotation: Rotate to (0,0) and (1,0) → Add `(0, 3)` to queue
  - Counterclockwise: Not applicable (snake is horizontal)
- Queue after: `[(1, 2), (3, 4), (0, 3)]`
- Visited adds: `{(1, 0), (3, 0), (0, 1)}` 
- Increment moves to 1

**Move 2 - Process position (1, 2):**
- Snake at (0,1) and (0,2) horizontally
- Try moves:
  - Right: Would go out of bounds
  - Down: Move to (1,1) and (1,2) → Add `(4, 5)` to queue
  - Clockwise rotation: Rotate to (0,1) and (1,1) → Add `(1, 4)` to queue

**Move 2 - Process position (3, 4):**
- Snake at (1,0) and (1,1) horizontally
- Try moves:
  - Right: Move to (1,1) and (1,2) → Add `(4, 5)` (already added)
  - Down: Move to (2,0) and (2,1) → Add `(6, 7)` to queue
  - Clockwise rotation: Rotate to (1,0) and (2,0) → Add `(3, 6)` to queue

**Move 2 - Process position (0, 3):**
- Snake at (0,0) and (1,0) vertically
- Try moves:
  - Right: Move to (0,1) and (1,1) → Add `(1, 4)` (already added)
  - Down: Move to (1,0) and (2,0) → Add `(3, 6)` (already added)
  - Counterclockwise: Rotate to (0,0) and (0,1) → Already visited

After processing all positions at distance 1, increment moves to 2.

**Move 3 - Continue processing:**
The BFS continues exploring positions at distance 2. Eventually, we'll reach position `(7, 8)` which represents the snake at cells (2,1) and (2,2) - our target!

**Key Observations:**
1. The snake explores multiple paths simultaneously through BFS
2. Each state tracks both position AND orientation (horizontal vs vertical)
3. Rotations are only possible when there's enough empty space
4. The first time we reach the target is guaranteed to be the shortest path

In this example, the minimum number of moves would be 3: starting horizontally at top-left, the snake can move down, then down again, then rotate and move to reach the target position at bottom-right.

## Python Solution

```python
class Solution:
    def minimumMoves(self, grid: List[List[int]]) -> int:
        """
        Find minimum moves for a snake to reach target position.
        The snake occupies 2 cells and can move right, down, or rotate.
      
        Args:
            grid: 2D grid where 0 = empty, 1 = obstacle
          
        Returns:
            Minimum number of moves to reach target, or -1 if impossible
        """
        def try_move(head_row: int, head_col: int, tail_row: int, tail_col: int) -> None:
            """
            Try to move snake to new position if valid.
          
            Args:
                head_row, head_col: New head position
                tail_row, tail_col: New tail position
            """
            # Check if both positions are within bounds
            if (0 <= head_row < grid_size and 0 <= head_col < grid_size and 
                0 <= tail_row < grid_size and 0 <= tail_col < grid_size):
              
                # Convert 2D coordinates to 1D indices
                head_index = head_row * grid_size + head_col
                tail_index = tail_row * grid_size + tail_col
              
                # Determine orientation: 0 for horizontal, 1 for vertical
                orientation = 0 if head_row == tail_row else 1
              
                # Check if this state hasn't been visited and both cells are empty
                if ((head_index, orientation) not in visited and 
                    grid[head_row][head_col] == 0 and grid[tail_row][tail_col] == 0):
                  
                    queue.append((head_index, tail_index))
                    visited.add((head_index, orientation))
      
        grid_size = len(grid)
        # Target position: snake at bottom-right corner, horizontal orientation
        target_position = (grid_size * grid_size - 2, grid_size * grid_size - 1)
      
        # BFS initialization: start with snake at top-left, horizontal
        queue = deque([(0, 1)])  # Head at (0,0), tail at (0,1)
        visited = {(0, 0)}  # (head_index, orientation)
        moves_count = 0
      
        # BFS to find shortest path
        while queue:
            # Process all positions at current distance
            for _ in range(len(queue)):
                head_index, tail_index = queue.popleft()
              
                # Check if we reached the target
                if (head_index, tail_index) == target_position:
                    return moves_count
              
                # Convert 1D indices back to 2D coordinates
                head_row, head_col = head_index // grid_size, head_index % grid_size
                tail_row, tail_col = tail_index // grid_size, tail_index % grid_size
              
                # Try moving right (maintaining current orientation)
                try_move(head_row, head_col + 1, tail_row, tail_col + 1)
              
                # Try moving down (maintaining current orientation)
                try_move(head_row + 1, head_col, tail_row + 1, tail_col)
              
                # If horizontal, try clockwise rotation (pivot on head)
                if head_row == tail_row and head_row + 1 < grid_size and grid[head_row + 1][tail_col] == 0:
                    try_move(head_row, head_col, head_row + 1, head_col)
              
                # If vertical, try counter-clockwise rotation (pivot on head)
                if head_col == tail_col and head_col + 1 < grid_size and grid[tail_row][head_col + 1] == 0:
                    try_move(head_row, head_col, head_row, head_col + 1)
          
            moves_count += 1
      
        # Target unreachable
        return -1
```

## Time and Space Complexity

**Time Complexity: `O(n^2)`**

The algorithm uses BFS to explore all possible states of the snake. The snake is represented by two cells, and each cell can be in one of `n^2` positions. However, the snake has only two orientations (horizontal or vertical), and the valid positions are constrained. The total number of unique states is bounded by `O(n^2)` because:
- We track states using `(head_position, orientation)` where head_position can be at most `n^2` locations
- Each state has 2 possible orientations (horizontal=0, vertical=1)
- The visited set `vis` ensures each state is processed only once

For each state, we perform at most 4 moves (right, down, and potentially 2 rotations), each taking `O(1)` time. Therefore, the overall time complexity is `O(n^2)`.

**Space Complexity: `O(n^2)`**

The space is dominated by:
- The BFS queue `q` which can contain at most `O(n^2)` states in the worst case
- The visited set `vis` which stores at most `O(n^2)` unique states (each state is a tuple of position and orientation)
- The input grid takes `O(n^2)` space

Therefore, the total space complexity is `O(n^2)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Snake Position Representation**
A critical pitfall is confusing which cell represents the "head" and which represents the "tail" of the snake. In the provided code, there's actually a naming inconsistency - the variables are named `head_index` and `tail_index`, but based on the problem description and initial state `(0, 1)`, the snake starts at cells `(0, 0)` and `(0, 1)`. The code treats the first value as position at `(0, 0)` and second as `(0, 1)`, but names suggest otherwise.

**Fix:** Use consistent and clear naming:
```python
# Better naming convention
queue = deque([(0, 1)])  # (first_cell, second_cell) or (tail, head)
# where first_cell is at (0,0) and second_cell is at (0,1)
```

### 2. **Rotation Boundary Check Missing**
When performing rotations, the code checks if the pivot cell's adjacent cell is empty, but doesn't verify if the rotation would place the snake out of bounds. While the current implementation includes boundary checks in `try_move`, a common mistake is to forget checking the intermediate cell needed for rotation.

**Issue Example:** For clockwise rotation from horizontal position, both `(r+1, c)` and `(r+1, c+1)` must be empty AND within bounds for the rotation to be valid.

**Fix:** Add explicit validation before rotation:
```python
# For clockwise rotation (horizontal to vertical)
if head_row == tail_row:  # Currently horizontal
    # Need to check BOTH cells below are valid and empty
    if (head_row + 1 < grid_size and 
        grid[head_row + 1][head_col] == 0 and 
        grid[head_row + 1][tail_col] == 0):
        try_move(head_row, head_col, head_row + 1, head_col)
```

### 3. **Visited State Tracking Error**
The visited set stores `(position, orientation)` but uses only one position index instead of both. This could theoretically cause issues if the snake can reach the same cell pair in different orders.

**Current approach:** `visited.add((head_index, orientation))`

**Potential issue:** Two different snake configurations with the same head position and orientation but different tail positions would be considered the same state.

**Fix:** Store complete state information:
```python
# More precise state tracking
visited.add((min(head_index, tail_index), max(head_index, tail_index), orientation))
```

### 4. **Target Position Assumption**
The code assumes the target is always at `(n*n - 2, n*n - 1)` in horizontal orientation. However, the problem states the snake should occupy `(n-1, n-2)` and `(n-1, n-1)`, which translates to indices `(n*n - n - 1, n*n - 1)` not `(n*n - 2, n*n - 1)`.

**Fix:** Correct target calculation:
```python
# Correct target position
# (n-1, n-2) -> index = (n-1) * n + (n-2) = n*n - 2
# (n-1, n-1) -> index = (n-1) * n + (n-1) = n*n - 1
target_position = (grid_size * grid_size - 2, grid_size * grid_size - 1)  # This is actually correct
```

### 5. **Orientation Detection Logic**
The orientation is determined by comparing rows (`head_row == tail_row`), but this assumes a specific ordering of the snake cells. If the cell indices are swapped, the orientation detection could fail.

**Fix:** Use absolute position comparison:
```python
# More robust orientation detection
def get_orientation(pos1_row, pos1_col, pos2_row, pos2_col):
    if pos1_row == pos2_row:
        return 0  # Horizontal
    elif pos1_col == pos2_col:
        return 1  # Vertical
    else:
        raise ValueError("Invalid snake position")
```

