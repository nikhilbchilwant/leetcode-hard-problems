# 1263. Minimum Moves to Move a Box to Their Target Location

> Source: https://algo.monster/liteproblems/1263

## Problem Statement

This problem is about a Sokoban-style puzzle game where you control a player who needs to push a box to a target location on a grid.

The game is played on an `m x n` grid where each cell contains one of the following characters:
- `'S'` - The starting position of the player
- `'B'` - The position of the box that needs to be moved
- `'T'` - The target position where the box should be pushed to
- `'.'` - An empty floor cell where the player can walk
- `'#'` - A wall that blocks movement

The rules of the game are:
1. The player can move up, down, left, or right to any adjacent floor cell (empty cell), but cannot walk through walls or the box
2. To move the box, the player must be adjacent to it and then move in the direction of the box - this pushes the box one cell in that direction
3. The box can only be pushed to an empty floor cell (not through walls)
4. There is exactly one box and one target position in the grid

The goal is to find the **minimum number of pushes** needed to move the box from its starting position to the target position. If it's impossible to push the box to the target, return `-1`.

For example, if the player is to the left of the box and moves right, the box will be pushed one cell to the right (assuming there's an empty cell there). The player ends up where the box was, and the box moves to the next cell in that direction.

## Intuition

The key insight is to recognize that this isn't just about finding a path for the player - it's about tracking two entities simultaneously: the player and the box. Each state in our search space needs to capture both positions: `(player_position, box_position)`.

Why BFS? We want the **minimum number of pushes**. BFS naturally finds the shortest path in terms of the number of operations. But there's a subtle twist here - not all moves are equal. When the player moves without pushing the box, we haven't made progress toward our goal (no push occurred). When the player pushes the box, that's when we've made one unit of progress.

This leads to a critical optimization: we can use a **double-ended queue** (deque) to implement a "0-1 BFS". When the player moves without pushing (cost = 0), we add this state to the front of the queue. When the player pushes the box (cost = 1), we add it to the back. This ensures we explore all free moves before considering additional pushes, effectively grouping states by their push count.

The state representation is clever too. Instead of using 4D coordinates `(player_i, player_j, box_i, box_j)`, we can flatten each 2D position into a single number using `f(i, j) = i * n + j`. This makes it easier to track visited states in a 2D array `vis[player_state][box_state]`.

The search process works as follows:
1. Try to move the player in all four directions
2. If the player moves into the box's position, the box gets pushed in that same direction (if valid)
3. If the player moves to an empty cell, it's just a repositioning move (no push)
4. Keep track of visited states to avoid cycles
5. Continue until the box reaches the target position

The beauty of this approach is that BFS guarantees we find the minimum number of pushes, while the deque optimization ensures we efficiently explore the state space by prioritizing free moves over pushes.

**Learn more about [Breadth-First Search](/problems/bfs_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation uses **BFS with a double-ended queue** to find the minimum number of pushes. Let's walk through the key components:

**State Representation:**
- We define a function `f(i, j) = i * n + j` to map 2D coordinates to a 1D state number
- Each state is represented as `(f(player_position), f(box_position), push_count)`
- This allows us to use a 2D visited array: `vis[player_state][box_state]`

**Initialization:**
1. First, traverse the grid to find initial positions:
   - Player position: `(si, sj)` where `grid[i][j] = 'S'`
   - Box position: `(bi, bj)` where `grid[i][j] = 'B'`
2. Initialize a deque `q` with the starting state: `(f(si, sj), f(bi, bj), 0)`
3. Mark the initial state as visited in `vis`

**BFS Process:**
For each state `(s, b, d)` dequeued from `q`:
1. Convert state numbers back to coordinates:
   - Box position: `bi = b // n`, `bj = b % n`
   - Player position: `si = s // n`, `sj = s % n`

2. **Check termination:** If `grid[bi][bj] = 'T'`, return `d` (minimum pushes found)

3. **Try all four directions** for player movement:
   - Calculate new player position: `(sx, sy) = (si + a, sj + b)`
   - Use `check()` function to verify it's a valid position (within bounds and not a wall)

4. **Handle two cases:**
 
   **Case 1: Player moves into box position** `(sx, sy) == (bi, bj)`:
   - This means the player pushes the box
   - Calculate new box position: `(bx, by) = (bi + a, bj + b)`
   - If the new box position is valid and this state hasn't been visited:
     - Mark `vis[f(sx, sy)][f(bx, by)]` as visited
     - **Append** `(f(sx, sy), f(bx, by), d + 1)` to the **back** of queue (push occurred, cost +1)
 
   **Case 2: Player moves to empty cell**:
   - This is just player repositioning (no push)
   - If state `vis[f(sx, sy)][f(bi, bj)]` hasn't been visited:
     - Mark it as visited
     - **Append** `(f(sx, sy), f(bi, bj), d)` to the **front** of queue (no push, cost +0)

**Key Optimization - 0-1 BFS:**
- States with no push (cost 0) are added to the **front** using `appendleft()`
- States with a push (cost 1) are added to the **back** using `append()`
- This ensures we explore all possible player repositioning before considering another push, effectively processing states level by level based on push count

**Termination:**
- If the queue becomes empty without finding the target, return `-1` (no solution exists)

The algorithm guarantees finding the minimum number of pushes because BFS explores states in order of increasing push count, and the 0-1 BFS optimization ensures efficient exploration of the state space.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach:

```
Grid:
# # # # #
# T . . #
# . B . #
# . S . #
# # # # #
```

**Initial Setup:**
- Player starts at `S` position: `(3, 2)`
- Box starts at `B` position: `(2, 2)`
- Target is at `T` position: `(1, 1)`
- Grid dimensions: `m=5, n=5`

Using our state encoding function `f(i,j) = i*5 + j`:
- Initial player state: `f(3,2) = 17`
- Initial box state: `f(2,2) = 12`

**BFS Process:**

**Step 1:** Initialize
- Queue: `[(17, 12, 0)]` (player at 17, box at 12, 0 pushes)
- Mark `vis[17][12] = true`

**Step 2:** Process state `(17, 12, 0)`
- Player at `(3,2)`, Box at `(2,2)`
- Try all 4 directions:
  - **Up**: Player moves to `(2,2)` - this is the box position!
    - Box gets pushed to `(1,2)` ✓ valid empty cell
    - Add `(12, 7, 1)` to **back** of queue (push occurred)
  - **Down**: Player moves to `(4,2)` - wall `#`, invalid
  - **Left**: Player moves to `(3,1)` - empty cell `.`
    - Add `(16, 12, 0)` to **front** of queue (just repositioning)
  - **Right**: Player moves to `(3,3)` - empty cell `.`
    - Add `(18, 12, 0)` to **front** of queue (just repositioning)

Queue now: `[(16, 12, 0), (18, 12, 0), (12, 7, 1)]`

**Step 3:** Process state `(16, 12, 0)`
- Player at `(3,1)`, Box at `(2,2)`
- Try all 4 directions:
  - **Up**: Player to `(2,1)` - empty cell, add `(11, 12, 0)` to front
  - Other moves either hit walls or revisit states

**Step 4:** Process state `(18, 12, 0)`
- Player at `(3,3)`, Box at `(2,2)`
- Similar exploration of moves...

**Step 5:** Process state `(11, 12, 0)`
- Player at `(2,1)`, Box at `(2,2)`
- **Right**: Player moves to `(2,2)` - box position!
    - Box pushed to `(2,3)` ✓ valid
    - Add `(12, 13, 1)` to back

**Step 6:** Eventually process state `(12, 7, 1)`
- Player at `(2,2)`, Box at `(1,2)`
- **Left**: Player moves to `(2,1)` - empty cell
    - Add `(11, 7, 1)` to front (no push)

**Step 7:** Process state `(11, 7, 1)`
- Player at `(2,1)`, Box at `(1,2)`
- **Up**: Player moves to `(1,1)` - empty cell (the target!)
    - Add `(6, 7, 1)` to front
- **Right**: Player moves to `(2,2)` - empty cell
    - Add `(12, 7, 1)` to front (already visited, skip)

**Step 8:** Process state `(6, 7, 1)`
- Player at `(1,1)`, Box at `(1,2)`
- **Right**: Player moves to `(1,2)` - box position!
    - Box pushed to `(1,3)` ✓ valid
    - Add `(7, 8, 2)` to back

Continue until...

**Step N:** Process state `(7, 6, 2)`
- Player at `(1,2)`, Box at `(1,1)`
- Check: Box position `(1,1)` is the target `T`!
- **Return 2** (minimum pushes needed)

**Key Observations:**
1. The 0-1 BFS ensures we explore all repositioning moves (cost 0) before considering additional pushes
2. States with the same push count are processed together
3. The first time the box reaches the target, we've found the minimum number of pushes
4. The player can take many paths to get into position, but only actual pushes count toward our answer

## Python Solution

```python
class Solution:
    def minPushBox(self, grid: List[List[str]]) -> int:
        """
        Find minimum number of pushes to move box 'B' to target 'T' with person 'S'.
        Uses BFS with state (person_position, box_position).
        """
      
        def encode_position(row: int, col: int) -> int:
            """Encode 2D grid position to 1D integer for efficient storage."""
            return row * cols + col
      
        def is_valid_cell(row: int, col: int) -> bool:
            """Check if a cell is within bounds and not a wall."""
            return 0 <= row < rows and 0 <= col < cols and grid[row][col] != "#"
      
        # Find initial positions of person (S) and box (B)
        person_row, person_col = 0, 0
        box_row, box_col = 0, 0
      
        for i, row in enumerate(grid):
            for j, cell in enumerate(row):
                if cell == "S":
                    person_row, person_col = i, j
                elif cell == "B":
                    box_row, box_col = i, j
      
        rows, cols = len(grid), len(grid[0])
      
        # Direction vectors: up, right, down, left
        directions = (-1, 0, 1, 0, -1)
      
        # BFS queue: (person_position, box_position, push_count)
        queue = deque([(encode_position(person_row, person_col), 
                       encode_position(box_row, box_col), 0)])
      
        # Visited states: visited[person_pos][box_pos] = True if visited
        visited = [[False] * (rows * cols) for _ in range(rows * cols)]
        visited[encode_position(person_row, person_col)][encode_position(box_row, box_col)] = True
      
        while queue:
            person_pos, box_pos, pushes = queue.popleft()
          
            # Decode box position
            current_box_row, current_box_col = box_pos // cols, box_pos % cols
          
            # Check if box reached target
            if grid[current_box_row][current_box_col] == "T":
                return pushes
          
            # Decode person position
            current_person_row, current_person_col = person_pos // cols, person_pos % cols
          
            # Try moving person in all four directions
            for i in range(4):
                next_person_row = current_person_row + directions[i]
                next_person_col = current_person_col + directions[i + 1]
              
                # Skip if next position is invalid
                if not is_valid_cell(next_person_row, next_person_col):
                    continue
              
                # Check if person would move to box position (push the box)
                if next_person_row == current_box_row and next_person_col == current_box_col:
                    # Calculate new box position after push
                    next_box_row = current_box_row + directions[i]
                    next_box_col = current_box_col + directions[i + 1]
                  
                    # Skip if box can't be pushed there or state already visited
                    if not is_valid_cell(next_box_row, next_box_col):
                        continue
                    if visited[encode_position(next_person_row, next_person_col)][encode_position(next_box_row, next_box_col)]:
                        continue
                  
                    # Mark as visited and add to queue (push action increases count)
                    visited[encode_position(next_person_row, next_person_col)][encode_position(next_box_row, next_box_col)] = True
                    queue.append((encode_position(next_person_row, next_person_col), 
                                 encode_position(next_box_row, next_box_col), 
                                 pushes + 1))
                else:
                    # Person moves without pushing box
                    if not visited[encode_position(next_person_row, next_person_col)][encode_position(current_box_row, current_box_col)]:
                        # Mark as visited and add to front of queue (no push, same distance)
                        visited[encode_position(next_person_row, next_person_col)][encode_position(current_box_row, current_box_col)] = True
                        queue.appendleft((encode_position(next_person_row, next_person_col), 
                                         encode_position(current_box_row, current_box_col), 
                                         pushes))
      
        # No solution found
        return -1
```

## Time and Space Complexity

**Time Complexity:** `O(m^2 × n^2)`

The algorithm uses BFS to explore different states where each state consists of the shopkeeper's position and the box's position. Since both the shopkeeper and the box can be at any valid cell in the grid:
- Total possible positions for the shopkeeper: `m × n`
- Total possible positions for the box: `m × n`
- Total possible states: `(m × n) × (m × n) = m^2 × n^2`

Each state is visited at most once due to the visited array `vis[f(si, sj)][f(bi, bj)]`. For each state, we explore 4 directions (constant time operations), so the overall time complexity is `O(m^2 × n^2)`.

**Space Complexity:** `O(m^2 × n^2)`

The space complexity is dominated by:
- The visited array `vis`: A 2D boolean array of size `(m × n) × (m × n)`, which stores whether each combination of shopkeeper position and box position has been visited. This requires `O(m^2 × n^2)` space.
- The BFS queue `q`: In the worst case, it could contain all possible states, which is `O(m^2 × n^2)`.
- Other variables like the grid itself and auxiliary variables use `O(m × n)` space, which is negligible compared to the visited array.

Therefore, the overall space complexity is `O(m^2 × n^2)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect State Representation - Not Tracking Player Position**

**Pitfall:** A common mistake is to only track the box position in the BFS state, thinking that only the box's location matters for the minimum pushes.

```python
# WRONG: Only tracking box position
queue = deque([(box_row, box_col, 0)])
visited = set()
visited.add((box_row, box_col))
```

**Why it fails:** The player's position is crucial because:
- To push the box in a certain direction, the player must be on the opposite side
- The same box position with different player positions represents different game states
- Without tracking player position, you can't determine which pushes are actually possible

**Solution:** Always track both player and box positions as a combined state:
```python
# CORRECT: Track both positions
queue = deque([(player_pos, box_pos, pushes)])
visited = [[False] * (rows * cols) for _ in range(rows * cols)]
visited[player_pos][box_pos] = True
```

### 2. **Using Regular BFS Instead of 0-1 BFS**

**Pitfall:** Treating all moves equally in the queue, which can lead to suboptimal solutions:

```python
# WRONG: All moves added to back of queue
if person_moves_to_box_position:
    queue.append((new_player_pos, new_box_pos, pushes + 1))
else:
    queue.append((new_player_pos, box_pos, pushes))  # Wrong!
```

**Why it fails:** 
- Player repositioning (cost 0) should be explored before additional pushes (cost 1)
- Regular BFS might find a path with more pushes first

**Solution:** Use 0-1 BFS pattern - add cost-0 moves to front, cost-1 moves to back:
```python
# CORRECT: 0-1 BFS
if person_moves_to_box_position:
    queue.append((new_player_pos, new_box_pos, pushes + 1))  # Cost 1: to back
else:
    queue.appendleft((new_player_pos, box_pos, pushes))      # Cost 0: to front
```

### 3. **Not Validating Box Push Destination**

**Pitfall:** Forgetting to check if the box can actually be pushed to the new position:

```python
# WRONG: Only checking if player can move
if is_valid_cell(next_person_row, next_person_col):
    if next_person_row == box_row and next_person_col == box_col:
        # Forgot to check if box can move to its new position!
        new_box_row = box_row + directions[i]
        new_box_col = box_col + directions[i + 1]
        queue.append(...)
```

**Solution:** Always validate both the player's move AND the box's new position when pushing:
```python
# CORRECT: Check both player and box destinations
if is_valid_cell(next_person_row, next_person_col):
    if next_person_row == box_row and next_person_col == box_col:
        new_box_row = box_row + directions[i]
        new_box_col = box_col + directions[i + 1]
        if not is_valid_cell(new_box_row, new_box_col):
            continue  # Can't push box there
        # Now safe to add to queue
```

### 4. **Inefficient State Encoding**

**Pitfall:** Using tuples or strings for state representation in visited set:

```python
# INEFFICIENT: Using tuples in set
visited = set()
visited.add((person_row, person_col, box_row, box_col))
```

**Why it's problematic:**
- Set operations with tuples can be slower for large grids
- Memory overhead from storing multiple integers per state

**Solution:** Use integer encoding and 2D array for O(1) lookups:
```python
# EFFICIENT: Integer encoding with 2D array
def encode_position(row, col):
    return row * cols + col

visited = [[False] * (rows * cols) for _ in range(rows * cols)]
visited[encode_position(person_row, person_col)][encode_position(box_row, box_col)] = True
```

