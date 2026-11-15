# 2056. Number of Valid Move Combinations On Chessboard

> Source: https://algo.monster/liteproblems/2056

## Problem Statement

You have an `8 x 8` chessboard with `n` chess pieces on it. Each piece is either a rook, queen, or bishop. You're given:
- `pieces[i]`: a string array describing the type of each piece
- `positions[i] = [ri, ci]`: the starting position of the i-th piece (using 1-based coordinates)

You need to count how many valid move combinations exist where all pieces move simultaneously.

**Movement Rules:**
- **Rook**: moves horizontally or vertically (4 directions)
- **Queen**: moves horizontally, vertically, or diagonally (8 directions)  
- **Bishop**: moves diagonally only (4 directions)

**How Movement Works:**
1. You choose a destination square for each piece (can be its current position if it doesn't move)
2. Starting at time 0, all pieces move simultaneously toward their destinations
3. Each second, a piece moves exactly one square in its chosen direction
4. A piece stops when it reaches its destination

**Valid vs Invalid Combinations:**
- A combination is **invalid** if at any point in time, two or more pieces occupy the same square
- A combination is **valid** if no two pieces ever occupy the same square during their movement

**Special Rules:**
- No two pieces start at the same position
- A piece can choose to stay at its current position (not move at all)
- Two adjacent pieces can swap positions by moving past each other in one second

The goal is to count all possible valid move combinations where each piece either stays put or moves to a reachable destination following its movement rules, and no collisions occur during the simultaneous movement.

## Intuition

The key insight is that with at most 4 pieces and each piece having at most 8 directional choices (plus the option to stay still), we can afford to explore all possible combinations exhaustively.

Think of this problem as making a decision tree. For each piece, we decide:
1. Should it stay where it is?
2. If it moves, which direction should it go?
3. How far should it travel in that direction?

Since pieces move simultaneously, we need to track their positions over time. The critical observation is that collisions can happen in two ways:
- **Meeting at the same square**: Two pieces arrive at the same position at the same time
- **One piece passing through another's stopping point**: A piece continues moving through a square where another piece has already stopped

To handle this, we can use a time-based tracking system. For each piece, we record:
- `dist[i][x][y]`: The time when piece `i` passes through position `(x, y)`
- `end[i]`: Where and when piece `i` stops moving

The [backtracking](/problems/backtracking) approach naturally fits because:
1. We process pieces one by one (depth-first)
2. For each piece, we try all valid moves
3. Before committing to a move, we check if it creates any conflicts with previously placed pieces
4. If valid, we recurse to the next piece
5. If we successfully place all pieces without conflicts, we've found one valid combination
6. We backtrack to explore other possibilities

The beauty of this approach is that by processing pieces sequentially and checking conflicts immediately, we can prune invalid branches early. When checking if piece `i` can make a certain move, we only need to verify it doesn't conflict with pieces `0` through `i-1` that we've already placed.

This transforms a complex simultaneous movement problem into a simpler sequential decision problem where we validate each decision against previous ones.

**Learn more about [Backtracking](/problems/backtracking) patterns.**

## Solution Approach

The implementation uses DFS with [backtracking](/problems/backtracking) to explore all possible move combinations. Let's walk through the key components:

**1. Direction Setup**

First, we define the possible directions for each piece type:
- Rook: 4 directions (horizontal and vertical)
- Bishop: 4 directions (diagonals only)  
- Queen: 8 directions (all of the above)

The `get_dirs()` function returns the appropriate direction vectors based on the piece type.

**2. Core Data Structures**

- `dist[i][x][y]`: A 3D array tracking when piece `i` passes through position `(x, y)`. Value is -1 if never visited, otherwise stores the time.
- `end[i]`: Stores `(x, y, t)` representing where piece `i` stops and when it stops.
- `ans`: Counter for valid combinations.

**3. Validation Functions**

**`checkStop(i, x, y, t)`**: Checks if piece `i` can stop at position `(x, y)` at time `t`.
- Returns `True` if all previously placed pieces (indices `0` to `i-1`) either don't pass through `(x, y)` or pass through it at a different time than `t`.
- This ensures no collision at the stopping point.

**`checkPass(i, x, y, t)`**: Checks if piece `i` can pass through position `(x, y)` at time `t`.
- Returns `False` if any previous piece `j` also passes through at the same time (`dist[j][x][y] == t`)
- Returns `False` if any previous piece `j` has stopped at `(x, y)` and its stop time is ≤ `t`
- This prevents both simultaneous collisions and passing through occupied squares.

**4. DFS [Backtracking](/problems/backtracking) Process**

The `dfs(i)` function processes piece `i`:

1. **Base Case**: If `i >= n`, all pieces are successfully placed, increment `ans`.

2. **Option 1 - Stay Put**: 
   - Set `dist[i][x][y] = 0` (piece at starting position at time 0)
   - Set `end[i] = (x, y, 0)` (stops immediately)
   - If `checkStop()` validates no conflicts, recurse to next piece

3. **Option 2 - Move in Each Valid Direction**:
   - For each possible direction `(dx, dy)`:
     - Reset `dist[i]` array
     - Simulate movement step by step:
       - Move to `(nx, ny)` at time `nt`
       - Check if position is on board and `checkPass()` allows passage
       - Mark `dist[i][nx][ny] = nt`
       - Try stopping here: set `end[i] = (nx, ny, nt)`
       - If `checkStop()` validates, recurse to next piece
       - Continue moving in same direction for next iteration

**5. Main Execution**

- Initialize all data structures with board size 9 (using 1-based indexing)
- Start DFS from piece 0
- Return the total count of valid combinations

The algorithm cleverly handles the simultaneous movement problem by:
- Processing pieces sequentially in the search
- Recording time-stamped positions for validation
- Using early pruning when conflicts are detected
- Exploring all possibilities through [backtracking](/problems/backtracking)

The time complexity is manageable due to the small constraint (at most 4 pieces), making this exhaustive search practical.

## Example Walkthrough

Let's walk through a simple example with 2 pieces on a chessboard:
- Piece 0: Rook at position (1, 1)
- Piece 1: Bishop at position (2, 3)

**Step 1: Process Piece 0 (Rook)**

The DFS starts with piece 0. We have two main options:

*Option A: Rook stays at (1, 1)*
- Set `dist[0][1][1] = 0` (rook is at its start position at time 0)
- Set `end[0] = (1, 1, 0)` (stops immediately)
- Check if this is valid with `checkStop(0, 1, 1, 0)` - passes since no previous pieces exist
- Recurse to piece 1

*Option B: Rook moves right to (1, 2)*
- Initialize movement: `dist[0][1][1] = 0` (starting position)
- At time 1: move to (1, 2), set `dist[0][1][2] = 1`
- Check with `checkPass(0, 1, 2, 1)` - passes (no previous pieces)
- Try stopping here: `end[0] = (1, 2, 1)`
- Check with `checkStop(0, 1, 2, 1)` - passes
- Recurse to piece 1

**Step 2: Process Piece 1 (Bishop) - Following Option B above**

Now piece 1 needs to be placed, with piece 0 having moved from (1, 1) to (1, 2).

*Option A: Bishop stays at (2, 3)*
- Set `dist[1][2][3] = 0`
- Set `end[1] = (2, 3, 0)`
- Check with `checkStop(1, 2, 3, 0)`:
  - Does piece 0 pass through (2, 3)? No (`dist[0][2][3] = -1`)
  - Valid! This is one valid combination

*Option B: Bishop moves diagonally to (1, 2)*
- Initialize: `dist[1][2][3] = 0`
- At time 1: move to (1, 2), try to set `dist[1][1][2] = 1`
- Check with `checkPass(1, 1, 2, 1)`:
  - Piece 0 passes through (1, 2) at time 1 (`dist[0][1][2] = 1`)
  - **Collision detected!** Both pieces at (1, 2) at time 1
  - This branch is pruned - invalid combination

*Option C: Bishop moves diagonally to (3, 4)*
- Initialize: `dist[1][2][3] = 0`
- At time 1: move to (3, 4), set `dist[1][3][4] = 1`
- Check with `checkPass(1, 3, 4, 1)` - passes (piece 0 never goes there)
- Try stopping: `end[1] = (3, 4, 1)`
- Check with `checkStop(1, 3, 4, 1)` - passes
- Valid! This is another valid combination

**Step 3: Backtrack and Continue**

The algorithm backtracks to explore all other possibilities:
- Rook moves to (1, 3), (1, 4), etc. (continuing right)
- Rook moves up to (2, 1), (3, 1), etc.
- Rook moves down (if valid)
- Rook moves left (if valid)

For each rook position, we then try all bishop moves, validating collisions at each step.

**Key Observations from Example:**

1. **Time-based collision detection**: When both pieces tried to be at (1, 2) at time 1, the collision was immediately detected through the `dist` array comparison.

2. **Sequential processing with validation**: Even though pieces move simultaneously in the problem, we process them sequentially in our search, validating each new piece against all previously placed pieces.

3. **Early pruning**: Invalid moves are detected immediately (like the bishop collision), preventing exploration of invalid subtrees.

4. **Backtracking restoration**: After exploring bishop placements for one rook position, we backtrack (resetting bishop's `dist` and `end` data) and try the next rook position.

The final answer would be the sum of all valid combinations found through this exhaustive search with pruning.

## Python Solution

```python
from typing import List, Tuple

# Direction vectors for different chess pieces
ROOK_DIRECTIONS = [(1, 0), (-1, 0), (0, 1), (0, -1)]
BISHOP_DIRECTIONS = [(1, 1), (1, -1), (-1, 1), (-1, -1)]
QUEEN_DIRECTIONS = ROOK_DIRECTIONS + BISHOP_DIRECTIONS


def get_directions(piece: str) -> List[Tuple[int, int]]:
    """Get movement directions based on piece type."""
    match piece[0]:
        case "r":  # Rook
            return ROOK_DIRECTIONS
        case "b":  # Bishop
            return BISHOP_DIRECTIONS
        case _:    # Queen (default)
            return QUEEN_DIRECTIONS


class Solution:
    def countCombinations(self, pieces: List[str], positions: List[List[int]]) -> int:
        """
        Count all valid movement combinations where pieces don't collide.
      
        Args:
            pieces: List of piece types ('rook', 'bishop', or 'queen')
            positions: Starting positions of each piece [row, col]
      
        Returns:
            Number of valid movement combinations
        """
      
        def can_stop_at_position(piece_index: int, row: int, col: int, time: int) -> bool:
            """
            Check if a piece can stop at given position without collision.
            A piece can stop if no previous piece passes through this position at or after this time.
            """
            for prev_piece in range(piece_index):
                # Check if previous piece passes through this position at or after current time
                if time_to_reach[prev_piece][row][col] >= time:
                    return False
            return True
      
        def can_pass_through_position(piece_index: int, row: int, col: int, time: int) -> bool:
            """
            Check if a piece can pass through given position at given time.
            Cannot pass if:
            1. Another piece is at this position at the same time
            2. Another piece has stopped at this position before or at this time
            """
            for prev_piece in range(piece_index):
                # Check if previous piece is at this position at the same time
                if time_to_reach[prev_piece][row][col] == time:
                    return False
                # Check if previous piece has stopped at this position
                stop_row, stop_col, stop_time = final_positions[prev_piece]
                if stop_row == row and stop_col == col and stop_time <= time:
                    return False
            return True
      
        def explore_movements(piece_index: int) -> None:
            """
            Recursively explore all possible movements for pieces.
            Uses DFS to try all valid movement combinations.
            """
            # Base case: all pieces have been assigned movements
            if piece_index >= num_pieces:
                nonlocal total_combinations
                total_combinations += 1
                return
          
            start_row, start_col = positions[piece_index]
          
            # Option 1: Piece doesn't move (stays at starting position)
            time_to_reach[piece_index][:] = [[-1] * board_size for _ in range(board_size)]
            time_to_reach[piece_index][start_row][start_col] = 0
            final_positions[piece_index] = (start_row, start_col, 0)
          
            if can_stop_at_position(piece_index, start_row, start_col, 0):
                explore_movements(piece_index + 1)
          
            # Option 2: Piece moves in one of its valid directions
            valid_directions = get_directions(pieces[piece_index])
          
            for delta_row, delta_col in valid_directions:
                # Reset time matrix for this piece
                time_to_reach[piece_index][:] = [[-1] * board_size for _ in range(board_size)]
                time_to_reach[piece_index][start_row][start_col] = 0
              
                # Try moving in this direction
                current_row = start_row + delta_row
                current_col = start_col + delta_col
                current_time = 1
              
                # Continue moving in the same direction until hitting board edge or collision
                while (1 <= current_row < board_size and 
                       1 <= current_col < board_size and 
                       can_pass_through_position(piece_index, current_row, current_col, current_time)):
                  
                    # Mark this position as reachable at current time
                    time_to_reach[piece_index][current_row][current_col] = current_time
                    final_positions[piece_index] = (current_row, current_col, current_time)
                  
                    # Try stopping at this position
                    if can_stop_at_position(piece_index, current_row, current_col, current_time):
                        explore_movements(piece_index + 1)
                  
                    # Move to next position in the same direction
                    current_row += delta_row
                    current_col += delta_col
                    current_time += 1
      
        # Initialize variables
        num_pieces = len(pieces)
        board_size = 9  # Chess board is 8x8, but using 1-indexed (1-8)
      
        # time_to_reach[i][r][c] = time when piece i reaches position (r,c), -1 if never
        time_to_reach = [[[-1] * board_size for _ in range(board_size)] for _ in range(num_pieces)]
      
        # final_positions[i] = (row, col, time) where piece i stops
        final_positions = [(0, 0, 0) for _ in range(num_pieces)]
      
        # Counter for valid combinations
        total_combinations = 0
      
        # Start DFS exploration
        explore_movements(0)
      
        return total_combinations
```

## Time and Space Complexity

**Time Complexity:** `O((n × M)^n)`

The algorithm uses DFS to explore all possible movement combinations for `n` pieces. For each piece at position `i`:
- It first considers the case where the piece doesn't move (stays at initial position)
- Then it explores all possible directions (up to 8 for queen, 4 for rook/bishop)
- For each direction, it tries every possible stopping position within the board (up to `M-1` positions where `M=8` is the maximum distance a piece can travel on an 8×8 board)

At each recursion level `i`, the algorithm explores up to `O(n × M)` possibilities:
- The factor `n` comes from checking conflicts with all previously placed pieces (in `check_stop` and `check_pass` functions)
- The factor `M` comes from the maximum number of positions a piece can reach in one direction

Since there are `n` pieces total and we make decisions for each piece recursively, the total time complexity is `O((n × M)^n)`.

**Space Complexity:** `O(n × M²)`

The space is dominated by:
- `dist`: A 3D array of size `n × M × M` (where `M=9` for 1-indexed 8×8 board), storing distance information for each piece's movement path - `O(n × M²)`
- `end`: An array of size `n` storing end positions and times for each piece - `O(n)`
- Recursion stack depth: At most `n` levels deep - `O(n)`

The dominant term is `O(n × M²)`, which can be simplified to `O(n × M)` when considering `M` as the movement range rather than board dimension.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Collision Detection Between Moving Pieces**

**The Pitfall:** A common mistake is only checking if two pieces occupy the same position at their final destinations, missing collisions that occur during movement. For example, if piece A moves from (1,1) to (1,3) and piece B moves from (1,3) to (1,1), they would collide at position (1,2) at time 1, even though their final positions don't overlap.

**Solution:** Track the exact time each piece passes through each position. Use a 3D array `time_to_reach[piece][row][col]` to store when each piece reaches each square. When validating movements, check if any two pieces have the same time value for the same position.

### 2. **Mishandling Pieces That Don't Move**

**The Pitfall:** Forgetting to properly handle the case where a piece chooses not to move at all. This can lead to:
- Not marking the starting position as occupied at time 0
- Incorrectly allowing other pieces to pass through a stationary piece's position
- Missing valid combinations where some pieces stay put

**Solution:** Explicitly handle the stationary case by:
- Setting `time_to_reach[piece][start_row][start_col] = 0`
- Setting `final_positions[piece] = (start_row, start_col, 0)`
- Ensuring the validation functions check for conflicts with stationary pieces

### 3. **Off-by-One Errors with 1-Based Indexing**

**The Pitfall:** The problem uses 1-based coordinates (1-8 for chess positions), but Python uses 0-based indexing. Mixing these can cause:
- Array index out of bounds errors
- Incorrect boundary checks (e.g., checking `row < 8` instead of `row < 9` when using 1-based)
- Pieces moving off the board or not reaching edge squares

**Solution:** 
- Use a board size of 9 to accommodate 1-based indexing (indices 1-8)
- Consistently check boundaries as `1 <= position < 9`
- Don't convert between 0-based and 1-based; stick with the input format

### 4. **Incorrect Validation Order in Backtracking**

**The Pitfall:** Checking collisions with pieces that haven't been placed yet, or not properly resetting state when backtracking. This can cause:
- False positives where valid moves are rejected
- Contaminated state from previous exploration branches

**Solution:**
- Only validate against pieces with indices less than the current piece (already placed pieces)
- Reset the `time_to_reach` array for the current piece before exploring each new direction
- Ensure the validation functions only check `range(piece_index)` not `range(num_pieces)`

### 5. **Confusing Pass-Through vs Stop Validation**

**The Pitfall:** Using the same validation logic for both passing through a position and stopping at a position. The rules are different:
- **Passing through:** Can't be at the same position at the same time as another piece
- **Stopping:** Can't stop where another piece will pass through at any time >= stop time

**Solution:** Implement two separate validation functions:
- `can_pass_through_position()`: Checks for simultaneous occupation and stopped pieces
- `can_stop_at_position()`: Checks if any piece passes through at or after the stop time

This separation ensures the correct logic is applied for each scenario and makes the code more maintainable.

