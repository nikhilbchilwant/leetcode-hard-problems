# 1728. Cat and Mouse II

> Source: https://algo.monster/liteproblems/1728

## Problem Statement

This is a strategic game between a cat and a mouse on a grid. The grid contains:
- `'C'` - Cat's starting position
- `'M'` - Mouse's starting position  
- `'F'` - Food location
- `'.'` - Floor (walkable)
- `'#'` - Wall (not walkable)

**Game Rules:**
1. Mouse moves first, then they alternate turns
2. Each player can jump up to their maximum jump distance (`mouseJump` or `catJump`) in one of four directions (up, down, left, right)
3. Players can choose to jump any distance from 0 (stay in place) to their maximum
4. Players cannot jump through walls or outside the grid
5. Mouse can jump over Cat (but not vice versa)

**Winning Conditions:**
- **Cat wins if:**
  - Cat catches Mouse (occupies same position)
  - Cat reaches the food first
  - Mouse doesn't reach food within 1000 turns
- **Mouse wins if:**
  - Mouse reaches the food first

The problem asks: Given the grid and jump distances, determine if Mouse can win when both players play optimally. Return `true` if Mouse can win, `false` otherwise.

The solution uses game theory and [dynamic programming](/problems/dynamic_programming_intro) to model all possible game states as `(mouse_position, cat_position, whose_turn)`. It works backwards from terminal states (where the winner is known) to determine the outcome of the initial state. The algorithm uses BFS and degree counting to propagate winning/losing states through the game tree, considering that:
- A player wins if they can move to any winning state
- A player loses if all possible moves lead to losing states

The final answer is determined by checking the outcome of the initial state `(mouse_start, cat_start, mouse_turn)`.

## Intuition

To solve this game problem, we need to think about it from a game theory perspective. Since both players play optimally, we need to determine who has a winning strategy from any given position.

The key insight is that this is a **finite, deterministic game** - there are a limited number of possible game states, and from each state, the outcome is predetermined if both players play perfectly. Each game state can be represented as a tuple: `(mouse_position, cat_position, whose_turn)`.

We can work **backwards from the end states** where we know the winner for certain:
- If mouse reaches food → mouse wins
- If cat reaches food → cat wins  
- If cat catches mouse → cat wins

From these terminal states, we can propagate the winning/losing information backwards. The reasoning is:
- If it's your turn and you can move to a state where you win, then the current state is a winning state for you
- If it's your turn and all your possible moves lead to states where you lose, then the current state is a losing state for you

This naturally leads to a **BFS approach** starting from terminal states. However, there's a subtlety: when determining if a state is losing, we need to ensure we've checked ALL possible moves. This is where the **degree counting** technique comes in:
- We track how many possible moves each player has from each state
- A state becomes a losing state only when its degree (count of unchecked moves) reaches 0, meaning all moves have been confirmed as losing

The algorithm assigns values to states:
- `0` = unknown
- `1` = mouse wins
- `2` = cat wins

By processing states level by level through BFS and carefully tracking degrees, we can determine the outcome of the initial state `(mouse_start, cat_start, mouse_turn=0)`. If this evaluates to `1`, mouse can win with optimal play.

**Learn more about [Graph](/problems/graph_intro), [Topological Sort](/problems/topo_intro), [Memoization](/problems/memoization_intro), [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation consists of two main parts: preprocessing the [graph](/problems/graph_intro) and calculating the game outcome using BFS with degree counting.

**1. [Graph](/problems/graph_intro) Construction:**

First, we build adjacency lists for both mouse and cat movements from each position:
```python
g_mouse = [[] for _ in range(m * n)]  # Mouse's possible moves from each position
g_cat = [[] for _ in range(m * n)]    # Cat's possible moves from each position
```

Each grid position `(i, j)` is mapped to a single integer `i * n + j` for easier state representation. For each valid position, we compute all reachable positions within the jump limit:
- Iterate through four directions using `dirs = (-1, 0, 1, 0, -1)` with `pairwise`
- For each direction, try jumping distances from 0 to `mouseJump`/`catJump`
- Stop if we hit a wall or grid boundary
- Add all valid positions to the respective adjacency list

**2. Game State Calculation:**

The `calc` function implements the backward induction algorithm:

**Initialize degree matrix:**
```python
degree[i][j][0] = len(g_mouse[i])  # Mouse's turn, count of mouse's moves
degree[i][j][1] = len(g_cat[j])     # Cat's turn, count of cat's moves
```

**Set terminal states:**
```python
ans[hole][i][1] = 1  # Mouse at food, any cat position, cat's turn → mouse wins
ans[i][hole][0] = 2  # Cat at food, any mouse position, mouse's turn → cat wins
ans[i][i][0] = ans[i][i][1] = 2  # Same position → cat wins (caught mouse)
```

**BFS propagation:**
Starting from terminal states in the queue, for each state we:

1. Find all previous states that could lead to the current state using `get_prev_states`:
   - If current turn is cat's (t=1), previous states are all positions cat could have come from
   - If current turn is mouse's (t=0), previous states are all positions mouse could have come from

2. For each previous state:
   - If the previous player can move to a winning state (`pt == t - 1`), mark it as winning immediately
   - Otherwise, decrement the degree counter
   - If degree reaches 0 (all moves explored and all are losing), mark the state as losing

**3. Result:**

The final answer is obtained by checking `ans[mouse_start][cat_start][0]`:
- Returns `1` if mouse wins from the initial position
- Returns `2` if cat wins from the initial position

The algorithm guarantees correctness because:
- It explores all reachable states from terminal positions
- It correctly identifies winning states (any winning move available)
- It correctly identifies losing states (all moves lead to opponent winning)
- The degree counting ensures we don't prematurely mark a state as losing

## Example Walkthrough

Let me walk through a small example to illustrate the solution approach.

Consider this simple 2x3 grid:
```
C . F
# . M
```
With `mouseJump = 1` and `catJump = 1`.

**Step 1: Convert grid positions to indices**
- Cat starts at (0,0) → index 0
- Mouse starts at (1,2) → index 5  
- Food is at (0,2) → index 2
- Wall at (1,0) → index 3

**Step 2: Build adjacency lists**

For Mouse from position 5 (1,2):
- Can jump to: 5 (stay), 4 (left one), 2 (up one)
- `g_mouse[5] = [5, 4, 2]`

For Cat from position 0 (0,0):
- Can jump to: 0 (stay), 1 (right one)
- Cannot jump down (wall blocks)
- `g_cat[0] = [0, 1]`

**Step 3: Initialize degrees**
- `degree[5][0][0] = 3` (mouse at 5, has 3 moves)
- `degree[5][0][1] = 2` (cat at 0, has 2 moves)

**Step 4: Set terminal states and process BFS**

Terminal states added to queue:
- `ans[2][0][1] = 1` (mouse at food, cat at 0, cat's turn → mouse wins)
- `ans[5][2][0] = 2` (cat at food, mouse at 5, mouse's turn → cat wins)

Processing from queue:

From state (2,0,1) where mouse wins:
- Previous states: positions mouse could have come from to reach food
- State (5,0,0): mouse's turn, can move to winning state (2,0,1)
- Mark `ans[5][0][0] = 1` (mouse wins!)

From state (5,2,0) where cat wins:
- Previous states: positions cat could have come from to reach food
- State (5,1,1): cat at 1, can move to 2 (food) → cat wins
- Mark `ans[5][1][1] = 2`

Continue processing until queue is empty.

**Step 5: Check initial state**
- Initial state is (5,0,0) - mouse at 5, cat at 0, mouse's turn
- We found `ans[5][0][0] = 1`
- Therefore, mouse can win with optimal play!

The key insight: Mouse can reach the food in one jump from position 5 to position 2, while cat needs at least 2 moves from position 0. Since mouse moves first and plays optimally, mouse wins.

### Template Solution

```python
class Solution:
    def canMouseWin(self, grid: List[str], catJump: int, mouseJump: int) -> bool:
        # Step 1: Parse grid and find initial positions
        m, n = len(grid), len(grid[0])
        mouse_start = cat_start = food = 0
      
        for i in range(m):
            for j in range(n):
                if grid[i][j] == 'M':
                    mouse_start = i * n + j
                elif grid[i][j] == 'C':
                    cat_start = i * n + j
                elif grid[i][j] == 'F':
                    food = i * n + j
      
        # Step 2: Build adjacency lists for valid moves
        def build_graph(max_jump):
            graph = [[] for _ in range(m * n)]
            dirs = (-1, 0, 1, 0, -1)
          
            for i in range(m):
                for j in range(n):
                    if grid[i][j] == '#':
                        continue
                    pos = i * n + j
                    graph[pos].append(pos)  # Can stay in place
                  
                    for dx, dy in zip(dirs, dirs[1:]):
                        for jump in range(1, max_jump + 1):
                            ni, nj = i + dx * jump, j + dy * jump
                            if 0 <= ni < m and 0 <= nj < n and grid[ni][nj] != '#':
                                graph[pos].append(ni * n + nj)
                            else:
                                break  # Can't jump through walls
            return graph
      
        g_mouse = build_graph(mouseJump)
        g_cat = build_graph(catJump)
      
        # Step 3: Game state calculation with BFS
        # ans[i][j][k]: outcome when mouse at i, cat at j, turn k (0=mouse, 1=cat)
        # 0=unknown, 1=mouse wins, 2=cat wins
        ans = [[[0] * 2 for _ in range(m * n)] for _ in range(m * n)]
        degree = [[[0] * 2 for _ in range(m * n)] for _ in range(m * n)]
      
        # Initialize degrees
        for i in range(m * n):
            for j in range(m * n):
                degree[i][j][0] = len(g_mouse[i])
                degree[i][j][1] = len(g_cat[j])
      
        # Set terminal states
        queue = deque()
        for i in range(m * n):
            for t in range(2):
                # Mouse reaches food
                ans[food][i][t] = 1
                queue.append((food, i, t))
                # Cat reaches food
                if i != food:
                    ans[i][food][t] = 2
                    queue.append((i, food, t))
                # Cat catches mouse
                if i > 0:
                    ans[i][i][t] = 2
                    queue.append((i, i, t))
      
        # BFS to propagate outcomes
        while queue:
            mouse, cat, turn = queue.popleft()
            current_result = ans[mouse][cat][turn]
          
            # Get previous states
            if turn == 1:  # Was cat's turn, so previous was mouse's turn
                for prev_mouse in g_mouse[mouse]:
                    if ans[prev_mouse][cat][0]:
                        continue
                    if current_result == 1:  # Mouse wins
                        ans[prev_mouse][cat][0] = 1
                        queue.append((prev_mouse, cat, 0))
                    else:
                        degree[prev_mouse][cat][0] -= 1
                        if degree[prev_mouse][cat][0] == 0:
                            ans[prev_mouse][cat][0] = 2
                            queue.append((prev_mouse, cat, 0))
            else:  # Was mouse's turn, so previous was cat's turn
                for prev_cat in g_cat[cat]:
                    if ans[mouse][prev_cat][1]:
                        continue
                    if current_result == 2:  # Cat wins
                        ans[mouse][prev_cat][1] = 2
                        queue.append((mouse, prev_cat, 1))
                    else:
                        degree[mouse][prev_cat][1] -= 1
                        if degree[mouse][prev_cat][1] == 0:
                            ans[mouse][prev_cat][1] = 1
                            queue.append((mouse, prev_cat, 1))
      
        # Check if mouse wins from initial state
        return ans[mouse_start][cat_start][0] == 1
```

## Python Solution

```python
class Solution:
    def canMouseWin(self, grid: List[str], catJump: int, mouseJump: int) -> bool:
        """
        Determine if the mouse can win the game.
      
        Args:
            grid: 2D grid where 'M' is mouse, 'C' is cat, 'F' is food, '#' is wall
            catJump: Maximum jump distance for cat
            mouseJump: Maximum jump distance for mouse
          
        Returns:
            True if mouse can win, False otherwise
        """
        rows, cols = len(grid), len(grid[0])
        cat_position = mouse_position = food_position = 0
      
        # Direction vectors for 4 directions (up, right, down, left)
        directions = (-1, 0, 1, 0, -1)
      
        # Build adjacency lists for mouse and cat movements
        mouse_graph = [[] for _ in range(rows * cols)]
        cat_graph = [[] for _ in range(rows * cols)]
      
        # Parse grid and build movement graphs
        for row_idx, row in enumerate(grid):
            for col_idx, cell in enumerate(row):
                if cell == "#":  # Skip walls
                    continue
                  
                # Convert 2D position to 1D index
                current_pos = row_idx * cols + col_idx
              
                # Record special positions
                if cell == "C":
                    cat_position = current_pos
                elif cell == "M":
                    mouse_position = current_pos
                elif cell == "F":
                    food_position = current_pos
              
                # Build possible moves for both mouse and cat
                for dx, dy in pairwise(directions):
                    # Add mouse's possible moves (including stay at same position)
                    for jump_dist in range(mouseJump + 1):
                        new_row, new_col = row_idx + jump_dist * dx, col_idx + jump_dist * dy
                      
                        # Check boundaries and walls
                        if not (0 <= new_row < rows and 0 <= new_col < cols and grid[new_row][new_col] != "#"):
                            break
                        mouse_graph[current_pos].append(new_row * cols + new_col)
                  
                    # Add cat's possible moves (including stay at same position)
                    for jump_dist in range(catJump + 1):
                        new_row, new_col = row_idx + jump_dist * dx, col_idx + jump_dist * dy
                      
                        # Check boundaries and walls
                        if not (0 <= new_row < rows and 0 <= new_col < cols and grid[new_row][new_col] != "#"):
                            break
                        cat_graph[current_pos].append(new_row * cols + new_col)
      
        # Calculate game outcome using minimax algorithm
        result = self.calc(mouse_graph, cat_graph, mouse_position, cat_position, food_position)
        return result == 1  # Return True if mouse wins (result == 1)
  
    def calc(
        self,
        mouse_graph: List[List[int]],
        cat_graph: List[List[int]],
        mouse_start: int,
        cat_start: int,
        food_pos: int,
    ) -> int:
        """
        Calculate game outcome using retrograde analysis.
      
        Args:
            mouse_graph: Adjacency list for mouse movements
            cat_graph: Adjacency list for cat movements
            mouse_start: Mouse starting position
            cat_start: Cat starting position
            food_pos: Food position
          
        Returns:
            1 if mouse wins, 2 if cat wins, 0 if draw
        """
      
        def get_previous_states(state):
            """
            Get all possible previous states that could lead to current state.
          
            Args:
                state: Tuple of (mouse_pos, cat_pos, turn)
              
            Returns:
                List of previous states
            """
            mouse_pos, cat_pos, turn = state
            previous_turn = turn ^ 1  # Toggle turn (0->1 or 1->0)
            previous_states = []
          
            if previous_turn == 1:  # Previous turn was cat's
                # Find all positions cat could have been at
                for prev_cat_pos in cat_graph[cat_pos]:
                    if game_results[mouse_pos][prev_cat_pos][1] == 0:
                        previous_states.append((mouse_pos, prev_cat_pos, previous_turn))
            else:  # Previous turn was mouse's
                # Find all positions mouse could have been at
                for prev_mouse_pos in mouse_graph[mouse_pos]:
                    if game_results[prev_mouse_pos][cat_pos][0] == 0:
                        previous_states.append((prev_mouse_pos, cat_pos, 0))
          
            return previous_states
      
        num_positions = len(mouse_graph)
      
        # Initialize degree array (number of possible moves from each state)
        # degree[mouse_pos][cat_pos][turn] = number of moves
        degree = [[[0, 0] for _ in range(num_positions)] for _ in range(num_positions)]
        for mouse_pos in range(num_positions):
            for cat_pos in range(num_positions):
                degree[mouse_pos][cat_pos][0] = len(mouse_graph[mouse_pos])  # Mouse's turn
                degree[mouse_pos][cat_pos][1] = len(cat_graph[cat_pos])      # Cat's turn
      
        # Initialize game results array
        # game_results[mouse_pos][cat_pos][turn] = outcome (0: unknown, 1: mouse wins, 2: cat wins)
        game_results = [[[0, 0] for _ in range(num_positions)] for _ in range(num_positions)]
      
        # Queue for BFS
        queue = deque()
      
        # Initialize terminal states
        for pos in range(num_positions):
            # Mouse reaches food (mouse wins regardless of cat position)
            game_results[food_pos][pos][1] = 1
            queue.append((food_pos, pos, 1))
          
            # Cat reaches food (cat wins)
            game_results[pos][food_pos][0] = 2
            queue.append((pos, food_pos, 0))
          
            # Mouse and cat at same position (cat wins)
            game_results[pos][pos][0] = game_results[pos][pos][1] = 2
            queue.append((pos, pos, 0))
            queue.append((pos, pos, 1))
      
        # Perform retrograde analysis using BFS
        while queue:
            current_state = queue.popleft()
            current_result = game_results[current_state[0]][current_state[1]][current_state[2]]
          
            # Process all states that could lead to current state
            for prev_state in get_previous_states(current_state):
                prev_mouse_pos, prev_cat_pos, prev_turn = prev_state
              
                # If previous player can force the same outcome
                if prev_turn == current_result - 1:
                    game_results[prev_mouse_pos][prev_cat_pos][prev_turn] = current_result
                    queue.append(prev_state)
                else:
                    # Decrease degree (one less option leads to unfavorable outcome)
                    degree[prev_mouse_pos][prev_cat_pos][prev_turn] -= 1
                  
                    # If all moves lead to unfavorable outcome
                    if degree[prev_mouse_pos][prev_cat_pos][prev_turn] == 0:
                        game_results[prev_mouse_pos][prev_cat_pos][prev_turn] = current_result
                        queue.append(prev_state)
      
        # Return result for initial state (mouse's turn)
        return game_results[mouse_start][cat_start][0]
```

## Time and Space Complexity

**Time Complexity:** `O(n^2 * m^2 * (catJump + mouseJump))`

The time complexity breaks down as follows:
- Building the graph: `O(n * m * (catJump + mouseJump))` - For each cell in the grid, we explore up to `catJump + mouseJump` positions in 4 directions
- BFS traversal: `O((n * m)^2 * 2)` - We have states represented as `(mouse_position, cat_position, turn)`, where:
  - Mouse can be in any of `n * m` positions
  - Cat can be in any of `n * m` positions  
  - Turn can be 0 or 1
  - Total states: `(n * m) * (n * m) * 2 = O(n^2 * m^2)`
- For each state, we process its previous states which is bounded by the degree (number of edges), giving us `O(n * m)` work per state in the worst case
- Overall: `O(n * m * (catJump + mouseJump) + n^2 * m^2)` = `O(n^2 * m^2 * max(1, (catJump + mouseJump)/(n*m)))`

Since typically `catJump` and `mouseJump` are constants or small relative to grid size, the dominant term is `O(n^2 * m^2)`.

**Space Complexity:** `O(n^2 * m^2)`

The space complexity consists of:
- Graph adjacency lists `g_mouse` and `g_cat`: `O(n * m * max(catJump, mouseJump))` - Each cell can connect to at most `4 * max(catJump, mouseJump)` other cells
- `degree` array: `O((n * m)^2 * 2)` = `O(n^2 * m^2)` - Stores degree for each state
- `ans` array: `O((n * m)^2 * 2)` = `O(n^2 * m^2)` - Stores result for each state
- Queue `q`: `O(n^2 * m^2)` in worst case when all states are enqueued

The dominant space requirement is `O(n^2 * m^2)` from the state arrays.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Turn Encoding Leading to Wrong Win Condition Mapping**

One of the most common pitfalls in this solution is confusing the turn encoding with the winner encoding, which can lead to incorrect game outcome determination.

**The Problem:**
The code uses:
- Turn encoding: `0` for mouse's turn, `1` for cat's turn
- Result encoding: `1` for mouse wins, `2` for cat wins

The critical line that often causes confusion is:
```python
if prev_turn == current_result - 1:
```

This condition checks if the previous player can force a win. The logic is:
- If `current_result = 1` (mouse wins) and `prev_turn = 0` (mouse's turn), then `0 == 1 - 1` ✓
- If `current_result = 2` (cat wins) and `prev_turn = 1` (cat's turn), then `1 == 2 - 1` ✓

**Why It's Easy to Get Wrong:**
Developers often mistakenly think the turn number should directly correspond to the winner number, leading to incorrect implementations like:
```python
# WRONG: This doesn't correctly identify when a player can force a win
if prev_turn == current_result:  # Incorrect logic
```

**Solution:**
Always remember that the encoding offset exists: mouse (turn 0) wins with result 1, cat (turn 1) wins with result 2. Document this clearly and consider using named constants:

```python
# Better approach with clear constants
MOUSE_TURN = 0
CAT_TURN = 1
MOUSE_WINS = 1
CAT_WINS = 2

# Then the condition becomes clearer:
if prev_turn == current_result - 1:
    # Previous player can force this winning outcome
```

### 2. **Forgetting to Handle the "Stay in Place" Move**

**The Problem:**
The game rules allow players to jump 0 distance (stay in place), but it's easy to accidentally start the jump range from 1:

```python
# WRONG: Missing the stay-in-place option
for jump_dist in range(1, mouseJump + 1):  # Starts from 1, missing 0
    # ...
```

**Why This Matters:**
Staying in place can be a crucial strategic move. For example:
- Mouse might need to stay put to force cat into a bad position
- Cat might stay to guard the food location

**Solution:**
Always include 0 in the jump range:
```python
# CORRECT: Include 0 to allow staying in place
for jump_dist in range(mouseJump + 1):  # Starts from 0
    new_row, new_col = row_idx + jump_dist * dx, col_idx + jump_dist * dy
    # ...
```

### 3. **Duplicate States in Adjacency Lists**

**The Problem:**
When building the movement graph, the current implementation can add duplicate positions to the adjacency lists. For example, a player at position (1,1) can reach (1,1) by jumping 0 in any of the 4 directions, potentially adding the same position 4 times.

**Why It Causes Issues:**
- Inflates the degree count, making it harder for states to reach degree 0
- Can cause incorrect game outcome calculations
- Wastes computational resources

**Solution:**
Use a set to track unique positions before converting to a list:

```python
# Build adjacency lists without duplicates
for row_idx, row in enumerate(grid):
    for col_idx, cell in enumerate(row):
        if cell == "#":
            continue
      
        current_pos = row_idx * cols + col_idx
        mouse_moves = set()  # Use set to avoid duplicates
        cat_moves = set()
      
        for dx, dy in pairwise(directions):
            for jump_dist in range(mouseJump + 1):
                new_row, new_col = row_idx + jump_dist * dx, col_idx + jump_dist * dy
                if 0 <= new_row < rows and 0 <= new_col < cols and grid[new_row][new_col] != "#":
                    mouse_moves.add(new_row * cols + new_col)
                else:
                    break
          
            for jump_dist in range(catJump + 1):
                new_row, new_col = row_idx + jump_dist * dx, col_idx + jump_dist * dy
                if 0 <= new_row < rows and 0 <= new_col < cols and grid[new_row][new_col] != "#":
                    cat_moves.add(new_row * cols + new_col)
                else:
                    break
      
        mouse_graph[current_pos] = list(mouse_moves)
        cat_graph[current_pos] = list(cat_moves)
```

This ensures each reachable position appears exactly once in the adjacency list, leading to correct degree counting and game outcome calculation.

