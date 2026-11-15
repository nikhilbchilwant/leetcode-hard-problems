# 773. Sliding Puzzle

> Source: https://algo.monster/liteproblems/773

## Problem Statement

You have a 2x3 game board with tiles numbered 1 through 5 and one empty space (represented as 0). The tiles can slide into the empty space if they are directly adjacent to it (up, down, left, or right - no diagonal moves).

The goal is to rearrange the tiles to reach the target configuration:
```
[[1, 2, 3],
 [4, 5, 0]]
```

In this target state, the numbers 1-5 are arranged in order from left to right, top to bottom, with the empty space (0) in the bottom-right corner.

A **move** consists of sliding one tile that is adjacent to the empty space into that empty position (equivalently, swapping the 0 with an adjacent number).

Given an initial board configuration, you need to find the minimum number of moves required to reach the target state. If it's impossible to reach the target state from the given configuration, return `-1`.

For example, if the initial board is:
```
[[1, 2, 3],
 [4, 0, 5]]
```
You can slide the 5 into the empty space (where 0 is) to get the target configuration in just 1 move.

## Intuition

The key insight is to think of this puzzle as a graph traversal problem. Each possible board configuration is a node in our graph, and moving a tile creates an edge between two configurations. Since we want the minimum number of moves, we're looking for the shortest path from our starting configuration to the target configuration `[[1,2,3],[4,5,0]]`.

Why BFS? When all moves have equal cost (each move counts as 1), BFS naturally finds the shortest path because it explores all configurations reachable in 1 move, then all configurations reachable in 2 moves, and so on. The first time we encounter our target, we've found it using the minimum number of moves.

To implement this, we need to:
1. **Represent board states efficiently**: Converting the 2D board to a string like `"123450"` makes it easy to store in a set and check for duplicates
2. **Track visited states**: We use a set to avoid revisiting the same configuration, preventing infinite loops
3. **Generate all possible next states**: From any configuration, we find where the empty space (0) is, then try swapping it with each adjacent tile (up, down, left, right)
4. **Level-by-level exploration**: Using a queue, we process all states at distance `k` before moving to states at distance `k+1`

The solution converts between board representation and string representation (`board_to_string()` and `string_to_board()`) for efficient state management. The function `get_next_states()` generates all valid next states by finding the empty space and swapping it with adjacent tiles. If we exhaust all possible states without finding the target, the puzzle is unsolvable, so we return `-1`.

**Learn more about [Breadth-First Search](/problems/bfs_intro), [Memoization](/problems/memoization_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Backtracking](/problems/backtracking) patterns.**

## Solution Approach

The implementation uses BFS with string representation of board states. Let's walk through the key components:

**1. State Representation**
- Convert the 2D board to a string for efficient storage and comparison
- `board_to_string()`: Flattens the 2D board into a string by concatenating rows (e.g., `[[1,2,3],[4,0,5]]` becomes `"123405"`)
- `string_to_board(state)`: Reconstructs the 2D board from a string representation

**2. Generating Next States (`get_next_states()` function)**
- Find the position `(i, j)` of the empty space (0) in the current board
- Try moving the empty space in 4 directions: `[[0, -1], [0, 1], [1, 0], [-1, 0]]` (left, right, down, up)
- For each valid adjacent position `(x, y)`:
  - Swap the empty space with the adjacent tile
  - Convert to string and add to results
  - Swap back to restore original state (backtrack)

**3. BFS Implementation**
```python
start_state = board_to_string()  # Initial board state as string
target_state = "123450"  # Target state
visited = {start_state}  # Visited states set
queue = deque([start_state])  # BFS queue
moves = 0  # Move counter
```

**4. Level-by-Level Traversal**
- At the start of each level, increment `moves`; it now counts the moves needed to reach the states generated in this level
- For each state in the current level:
  - Convert string back to board using `string_to_board(current_state)`
  - Generate all possible next states using `get_next_states()`
  - If target state found, return `moves`
  - Add unvisited states to queue and visited set

**5. Early Termination**
- Check if initial state equals target (return 0)
- Return `-1` if queue empties without finding target (unsolvable)

The algorithm guarantees finding the minimum moves because BFS explores states in order of distance from the start. Time complexity is `O(n!)` where `n` is the number of tiles (6 in this case), as there are at most `6!` possible board configurations. Space complexity is also `O(n!)` for storing visited states.

## Example Walkthrough

Let's walk through a simple example where the initial board is:
```
[[1, 2, 3],
 [4, 0, 5]]
```
And we want to reach the target:
```
[[1, 2, 3],
 [4, 5, 0]]
```

**Step 1: Initialize BFS**
- Convert initial board to string: `"123405"` 
- Target string: `"123450"`
- Initialize queue with `["123405"]` and visited set with `{"123405"}`
- Set move counter `moves = 0`

**Step 2: Process Level 0 (Initial State)**
- At the start of the level, increment `moves` to 1
- Queue size = 1, so we process 1 state at this level
- Pop `"123405"` from queue
- Convert back to board: `[[1,2,3],[4,0,5]]`
- Find empty space (0) at position (1, 1)

**Step 3: Generate Next States**
From position (1, 1), the empty space is tried in the order left, right, down, up:
- **Left** (1, 1-1) = (1, 0): Swap with 4
  - New board: `[[1,2,3],[0,4,5]]` → String: `"123045"`
- **Right** (1, 1+1) = (1, 2): Swap with 5
  - New board: `[[1,2,3],[4,5,0]]` → String: `"123450"`
- **Down** (1+1, 1) = (2, 1): Invalid (out of bounds)
- **Up** (1-1, 1) = (0, 1): Swap with 2
  - New board: `[[1,0,3],[4,2,5]]` → String: `"103425"`

**Step 4: Check for Target**
- `"123045"` is checked first; it is not the target, so it is added to `visited` and the queue
- `"123450"` is checked next and equals the target
- Return `moves = 1`

The minimum number of moves is **1** (sliding the 5 left into the empty space).

**Alternative Example - Multiple Moves:**
If the initial board was:
```
[[4, 1, 2],
 [5, 0, 3]]
```
String: `"412503"`

The BFS would explore:
- **Level 0**: Initial state `"412503"`
- **Level 1**: States reachable in 1 move from initial
  - Move 0 up: `"402513"` 
  - Move 0 left: `"412053"`
  - Move 0 right: `"412530"`
- **Level 2**: States reachable in 2 moves (from unvisited Level 1 states)
- Continue until target `"123450"` is found or all states exhausted

Each level represents states reachable with exactly that many moves, ensuring we find the minimum path when we first encounter the target.

## Python Solution

```python
class Solution:
    def slidingPuzzle(self, board: List[List[int]]) -> int:
        """
        Solve the sliding puzzle problem using BFS.
        Goal: Transform the board to [[1,2,3],[4,5,0]]
        Returns minimum number of moves, or -1 if impossible.
        """
      
        def board_to_string() -> str:
            """Convert 2D board to string representation for hashing."""
            result = []
            for row in range(2):
                for col in range(3):
                    result.append(str(board[row][col]))
            return ''.join(result)
      
        def string_to_board(state: str) -> None:
            """Convert string representation back to 2D board."""
            for row in range(2):
                for col in range(3):
                    board[row][col] = int(state[row * 3 + col])
      
        def get_next_states() -> List[str]:
            """Generate all possible next states from current board configuration."""
            next_states = []
          
            # Find the position of the empty tile (0)
            zero_row, zero_col = next((i, j) for i in range(2) for j in range(3) if board[i][j] == 0)
          
            # Try moving the empty tile in all four directions
            directions = [[0, -1], [0, 1], [1, 0], [-1, 0]]  # left, right, down, up
          
            for delta_row, delta_col in directions:
                new_row = zero_row + delta_row
                new_col = zero_col + delta_col
              
                # Check if the new position is within bounds
                if 0 <= new_row < 2 and 0 <= new_col < 3:
                    # Swap empty tile with adjacent tile
                    board[zero_row][zero_col], board[new_row][new_col] = board[new_row][new_col], board[zero_row][zero_col]
                    next_states.append(board_to_string())
                    # Swap back to restore original state
                    board[zero_row][zero_col], board[new_row][new_col] = board[new_row][new_col], board[zero_row][zero_col]
          
            return next_states
      
        # Initialize BFS
        start_state = board_to_string()
        target_state = "123450"
      
        # Check if already solved
        if start_state == target_state:
            return 0
      
        # BFS setup
        visited = {start_state}
        queue = deque([start_state])
        moves = 0
      
        # BFS traversal
        while queue:
            moves += 1
            # Process all states at current level
            for _ in range(len(queue)):
                current_state = queue.popleft()
                string_to_board(current_state)
              
                # Explore all possible next states
                for next_state in get_next_states():
                    if next_state == target_state:
                        return moves
                  
                    if next_state not in visited:
                        visited.add(next_state)
                        queue.append(next_state)
      
        # No solution found
        return -1
```

## Time and Space Complexity

**Time Complexity:** `O((m*n)! * m*n)`

The sliding puzzle problem involves finding the shortest path from the initial board state to the target state "123450". This is essentially a BFS traversal through the state space graph.

- The total number of possible board states is `(m*n)!` where `m=2` and `n=3`, giving us `6! = 720` possible states
- For each state, we need `O(m*n)` time to:
  - Convert between board and string representations (`board_to_string()` and `string_to_board()` both take `O(6)` time)
  - Find the position of 0: `O(m*n)`
  - Generate neighbors: up to 4 neighbors, each requiring `O(m*n)` operations for swapping and string conversion
- In the worst case, BFS visits all possible states
- Therefore, the total time complexity is `O(6! * 6) = O(720 * 6) = O(4320)`, which simplifies to `O((m*n)! * m*n)`

**Space Complexity:** `O((m*n)!)`

- The `visited` set can store up to `(m*n)! = 6! = 720` unique states in the worst case
- The BFS queue can also contain up to `O((m*n)!)` states in the worst case (though typically much less)
- The temporary list in `board_to_string()` and board manipulations use `O(m*n) = O(6)` space
- The dominant factor is the visited set and queue, giving us `O((m*n)!)` space complexity

For this specific 2x3 puzzle, both complexities have constants: `O(720 * 6)` for time and `O(720)` for space, but in general form for an m×n puzzle, they are `O((m*n)! * m*n)` and `O((m*n)!)` respectively.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Inefficient State Representation and Comparison**
Many implementations create new board objects or use nested lists directly as dictionary keys, which causes errors since lists are unhashable. Some try using tuples of tuples, which works but creates unnecessary overhead.

**Problem Example:**
```python
# This will cause TypeError: unhashable type: 'list'
visited = {board}  # Can't use list as set/dict key

# This works but is inefficient
visited = {tuple(tuple(row) for row in board)}
```

**Solution:** Convert the board to a string representation for efficient hashing and comparison. This reduces memory usage and speeds up lookups.

### 2. **Not Restoring Board State After Generating Moves**
A critical bug occurs when modifying the board to generate next states without restoring it, causing incorrect state generation in subsequent iterations.

**Problem Example:**
```python
def get_next_states():
    next_states = []
    # Find zero position...
    for dr, dc in directions:
        if valid_position:
            board[zero_row][zero_col], board[new_row][new_col] = board[new_row][new_col], board[zero_row][zero_col]
            next_states.append(board_to_string())
            # MISSING: Swap back to restore original state!
    return next_states
```

**Solution:** Always swap back the tiles after recording the new state to maintain the board's integrity for generating other moves.

### 3. **Processing States Level by Level Incorrectly**
Failing to process all states at the current distance before incrementing the move counter leads to incorrect minimum move calculations.

**Problem Example:**
```python
while queue:
    current = queue.popleft()
    moves += 1  # Wrong! Increments for each state, not each level
    # Process current state...
```

**Solution:** Use a for loop with `range(len(queue))` to process exactly the states at the current level before incrementing moves:
```python
while queue:
    moves += 1
    for _ in range(len(queue)):  # Process entire level
        current = queue.popleft()
        # Generate and check next states...
```

### 4. **Forgetting Edge Cases**
Not checking if the initial state is already the target state, leading to unnecessary computation or incorrect results.

**Solution:** Always check if `start_state == target_state` before starting BFS and return 0 immediately if true.

