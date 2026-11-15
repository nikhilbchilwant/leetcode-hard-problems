# 2258. Escape the Spreading Fire

> Source: https://algo.monster/liteproblems/2258

## Problem Statement

You are given a 2D grid representing a field where:
- `0` represents grass cells you can walk on
- `1` represents fire cells that will spread
- `2` represents walls that block both you and fire

You start at the top-left corner `(0, 0)` and need to reach the safehouse at the bottom-right corner `(m-1, n-1)`.

The game mechanics work as follows:
1. Initially, you can choose to stay at your starting position for some number of minutes
2. Each minute you stay, the fire spreads to all adjacent grass cells (not through walls)
3. Once you start moving, you can move to an adjacent grass cell each minute
4. After each of your moves, the fire spreads again to adjacent grass cells
5. You cannot move through fire or walls

The goal is to find the **maximum number of minutes** you can wait at the starting position before beginning your journey, while still being able to safely reach the safehouse.

Return values:
- Return the maximum number of minutes you can wait if it's possible to reach the safehouse
- Return `-1` if it's impossible to reach the safehouse at all
- Return `10^9` if you can wait indefinitely and still reach the safehouse

Note that reaching the safehouse counts as safe even if fire spreads to it immediately after you arrive there.

Adjacent cells are those directly north, east, south, or west (sharing a side).

## Intuition

The key insight is recognizing that this problem has a **monotonic property**: if we can safely reach the safehouse after waiting `t` minutes, then we can also reach it after waiting any time less than `t`. Why? Because less waiting time means the fire has spread less, giving us more safe paths to work with.

This monotonic property immediately suggests **[binary search](/problems/binary_search_intro)** on the answer. We can search for the maximum waiting time between 0 and `m × n` (the total number of cells, which is an upper bound since fire can't spread more than this).

For each candidate waiting time `mid` in our [binary search](/problems/binary_search_intro), we need to check if it's possible to reach the safehouse. This check involves two phases:

1. **Fire spreading phase**: First, simulate the fire spreading for `mid` minutes while we wait at the starting position. We use BFS to spread fire level by level, where each level represents one minute.

2. **Escape phase**: After waiting, we start moving. Here's where it gets tricky - we need to simulate both our movement AND the fire continuing to spread simultaneously. We move one step per minute, and after each round of our movement, the fire spreads one more level.

The simulation works like this:
- Use BFS for the player's movement, exploring all reachable cells at distance `d` before moving to distance `d+1`
- After processing each distance level (one minute of player movement), spread the fire one more level
- If we reach `(m-1, n-1)` before getting caught by fire, this waiting time works
- If the fire blocks all paths or catches us, this waiting time doesn't work

Special cases to consider:
- If we can wait `m × n` minutes and still escape, we can wait indefinitely (return `10^9`)
- If we can't escape even without waiting (when `t = 0`), return `-1`

The beauty of this approach is that [binary search](/problems/binary_search_intro) reduces our search space from `O(m × n)` possible waiting times to just `O(log(m × n))` checks, and each check is done efficiently using BFS in `O(m × n)` time.

**Learn more about [Breadth-First Search](/problems/bfs_intro) and [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution implements [binary search](/problems/binary_search_intro) combined with BFS simulation. Let's break down the implementation:

### [Binary Search](/problems/binary_search_intro) Using first_true_index Template

This problem finds the **maximum** waiting time where escape is possible. We use the first_true_index template by inverting the condition: instead of finding where escape IS possible, we find where escape is NOT possible.

**Setup:**
- Search space: `0` to `m * n` (maximum possible waiting time)
- Inverted condition: `cannot_escape(mid)` returns true when we **cannot** escape with `mid` minutes wait
- Pattern: `false, false, ..., true, true` (can escape, can escape, ..., cannot escape, cannot escape)
- Answer: `first_true_index - 1` gives the last time we CAN escape

```python
left, right = 0, rows * cols
first_true_index = rows * cols + 1  # Default: all times allow escape

while left <= right:
    mid = (left + right) // 2
    if not can_escape_with_delay(mid):  # Cannot escape = condition is true
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

# Maximum wait time = first_true_index - 1
return 10**9 if first_true_index > rows * cols else first_true_index - 1
```

### The Check Function
The `check(t)` function determines if we can reach the safehouse after waiting `t` minutes. It consists of two main phases:

**Phase 1: Initial Fire Spreading**
```python
# Initialize fire grid and queue with all initial fire positions
fire = [[False] * n for _ in range(m)]
q1 = deque()
for i, row in enumerate(grid):
    for j, x in enumerate(row):
        if x == 1:  # Found fire
            fire[i][j] = True
            q1.append((i, j))

# Spread fire for t minutes while we wait
while t and q1:
    q1 = spread(q1)
    t -= 1
```

**Phase 2: Simultaneous Movement and Fire Spreading**
```python
# Check if starting position is on fire
if fire[0][0]:
    return False

# BFS for player movement
q2 = deque([(0, 0)])
vis = [[False] * n for _ in range(m)]
vis[0][0] = True

while q2:
    # Process all cells at current distance (one minute of movement)
    for _ in range(len(q2)):
        i, j = q2.popleft()
        if fire[i][j]:  # Skip if current position caught fire
            continue

        # Try all 4 directions
        for a, b in pairwise(dirs):  # dirs = (-1, 0, 1, 0, -1)
            x, y = i + a, j + b
            if valid_move(x, y):  # Check bounds, not visited, not on fire, is grass
                if x == m - 1 and y == n - 1:  # Reached safehouse!
                    return True
                vis[x][y] = True
                q2.append((x, y))

    # After each round of movement, fire spreads one more level
    q1 = spread(q1)
```

### The Spread Function
The `spread` function handles one minute of fire spreading using BFS:
```python
def spread(q: Deque[int]) -> Deque[int]:
    nq = deque()  # New queue for next level
    while q:
        i, j = q.popleft()
        for a, b in pairwise(dirs):  # Check all 4 directions
            x, y = i + a, j + b
            if 0 <= x < m and 0 <= y < n and not fire[x][y] and grid[x][y] == 0:
                fire[x][y] = True  # Mark as on fire
                nq.append((x, y))  # Add to next level
    return nq
```

### Direction Handling
The solution uses a clever pattern for iterating through 4 directions:
- `dirs = (-1, 0, 1, 0, -1)`
- `pairwise(dirs)` generates pairs: `(-1, 0)`, `(0, 1)`, `(1, 0)`, `(0, -1)`
- These represent movements: up, right, down, left

### Final Result
After [binary search](/problems/binary_search_intro) completes:
- If `l == m * n`: We can wait for the maximum time, return `10^9`
- Otherwise: Return `l` as the maximum waiting time

The time complexity is `O(m * n * log(m * n))` where each [binary search](/problems/binary_search_intro) iteration runs a BFS simulation in `O(m * n)` time.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a 3×3 grid:
```
0 0 0
0 1 0
0 0 0
```
Where we start at (0,0) and need to reach (2,2). There's fire at (1,1).

**Step 1: Binary Search Setup**
- Initialize: `l = -1`, `r = 3 × 3 = 9`
- We'll binary search for the maximum waiting time

**Step 2: First Binary Search Iteration**
- `mid = (l + r + 1) / 2 = (-1 + 9 + 1) / 2 = 4`
- Check if we can escape after waiting 4 minutes

**Checking wait time = 4:**

*Phase 1: Fire spreads for 4 minutes while we wait*
- Minute 0: Fire at (1,1)
- Minute 1: Fire spreads to (0,1), (1,0), (1,2), (2,1)
  ```
  0 1 0
  1 1 1
  0 1 0
  ```
- Minute 2: Fire spreads to (0,0), (0,2), (2,0), (2,2)
  ```
  1 1 1
  1 1 1
  1 1 1
  ```
- Minutes 3-4: All cells already on fire

*Phase 2: Try to escape*
- Starting position (0,0) is on fire → Cannot escape!
- `check(4)` returns False
- Update: `r = mid - 1 = 3`

**Step 3: Second Binary Search Iteration**
- `mid = (-1 + 3 + 1) / 2 = 1`
- Check if we can escape after waiting 1 minute

**Checking wait time = 1:**

*Phase 1: Fire spreads for 1 minute*
- Initial: Fire at (1,1)
- After 1 minute: Fire at (1,1), (0,1), (1,0), (1,2), (2,1)
  ```
  0 1 0
  1 1 1
  0 1 0
  ```

*Phase 2: Escape attempt*
- Start at (0,0) - not on fire ✓
- Our movement minute by minute:

  **Minute 1 (our first move):**
  - From (0,0), we can move to (1,0) or (0,1)
  - But both are on fire! Cannot move anywhere
  - `check(1)` returns False
  - Update: `r = mid - 1 = 0`

**Step 4: Third Binary Search Iteration**
- `mid = (-1 + 0 + 1) / 2 = 0`
- Check if we can escape without waiting

**Checking wait time = 0:**

*Phase 1: No waiting (skip)*

*Phase 2: Immediate escape*
- Fire initially only at (1,1)
- Our path simulation:

  **Minute 1:**
  - Player moves: (0,0) → can reach (1,0) or (0,1)
  - Add both to queue
  - Fire spreads: (1,1) → (0,1), (1,0), (1,2), (2,1)

  **Minute 2:**
  - Player moves from (0,1): blocked by fire
  - Player moves from (1,0): blocked by fire
  - Both positions caught by fire after minute 1
  - No valid moves remain

- `check(0)` returns False
- Update: `r = mid - 1 = -1`

**Step 5: Binary Search Completes**
- `l = -1`, `r = -1`, so `l == r`
- Final answer: `l = -1`
- Return **-1** (impossible to reach safehouse)

**Alternative Scenario:**
If the grid had been:
```
0 0 0
0 0 1
0 0 0
```
With fire at (1,2) instead, we could find a path:
- Without waiting: (0,0) → (0,1) → (0,2) → (1,2) would be blocked
- But: (0,0) → (1,0) → (2,0) → (2,1) → (2,2) works!
- Binary search would find we can wait 0 minutes and still escape
- If we could wait longer and still escape, binary search would find that maximum

This example shows how the algorithm combines binary search to find the optimal waiting time with BFS simulation to check if escape is possible for each candidate time.

## Python Solution

```python
class Solution:
    def maximumMinutes(self, grid: List[List[int]]) -> int:
        def spread_fire(fire_queue: Deque[Tuple[int, int]]) -> Deque[Tuple[int, int]]:
            """
            Spread fire to adjacent cells for one time unit.
            Returns a new queue with newly burning cells.
            """
            new_queue = deque()
            while fire_queue:
                row, col = fire_queue.popleft()
                # Check all 4 adjacent cells (up, right, down, left)
                for i in range(4):
                    new_row = row + directions[i]
                    new_col = col + directions[i + 1]
                    # Check if the new position is valid and can catch fire
                    if (0 <= new_row < rows and
                        0 <= new_col < cols and
                        not is_on_fire[new_row][new_col] and
                        grid[new_row][new_col] == 0):
                        is_on_fire[new_row][new_col] = True
                        new_queue.append((new_row, new_col))
            return new_queue

        def can_escape_with_delay(wait_time: int) -> bool:
            """
            Check if we can reach the safehouse starting after 'wait_time' minutes.
            """
            # Reset fire grid
            for row in range(rows):
                for col in range(cols):
                    is_on_fire[row][col] = False

            # Initialize fire starting positions
            fire_queue = deque()
            for row in range(rows):
                for col in range(cols):
                    if grid[row][col] == 1:
                        is_on_fire[row][col] = True
                        fire_queue.append((row, col))

            # Let fire spread for 'wait_time' minutes before we start moving
            remaining_time = wait_time
            while remaining_time > 0 and fire_queue:
                fire_queue = spread_fire(fire_queue)
                remaining_time -= 1

            # If starting position is on fire, we can't escape
            if is_on_fire[0][0]:
                return False

            # BFS to find if we can reach the safehouse
            person_queue = deque([(0, 0)])
            visited = [[False] * cols for _ in range(rows)]
            visited[0][0] = True

            while person_queue:
                # Process all positions reachable in current minute
                current_level_size = len(person_queue)
                for _ in range(current_level_size):
                    row, col = person_queue.popleft()
                    # Skip if current position is on fire
                    if is_on_fire[row][col]:
                        continue

                    # Try moving to adjacent cells
                    for i in range(4):
                        new_row = row + directions[i]
                        new_col = col + directions[i + 1]

                        if (0 <= new_row < rows and
                            0 <= new_col < cols and
                            not visited[new_row][new_col] and
                            not is_on_fire[new_row][new_col] and
                            grid[new_row][new_col] == 0):
                            # Check if we reached the safehouse
                            if new_row == rows - 1 and new_col == cols - 1:
                                return True
                            visited[new_row][new_col] = True
                            person_queue.append((new_row, new_col))

                # Fire spreads after each minute of movement
                fire_queue = spread_fire(fire_queue)

            return False

        # Initialize grid dimensions
        rows, cols = len(grid), len(grid[0])

        # Direction vectors for adjacent cells: up, right, down, left
        directions = [-1, 0, 1, 0, -1]

        # Track which cells are on fire
        is_on_fire = [[False] * cols for _ in range(rows)]

        # Binary search using first_true_index template
        # We invert the condition: find first time where escape FAILS
        left, right = 0, rows * cols
        first_true_index = rows * cols + 1  # Default: all times allow escape

        while left <= right:
            mid = (left + right) // 2
            if not can_escape_with_delay(mid):  # Cannot escape = condition is true
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1

        # Maximum wait time = first_true_index - 1
        # If first_true_index > rows * cols, all times allow escape (return 10^9)
        # Otherwise, return first_true_index - 1 (the last time escape is possible)
        if first_true_index > rows * cols:
            return int(1e9)
        return first_true_index - 1
```

## Time and Space Complexity

**Time Complexity:** `O(m × n × log(m × n))`

The algorithm uses binary search on the range `[-1, m × n]`, which requires `O(log(m × n))` iterations. For each binary search iteration, the `check` function is called.

Within the `check` function:
- Initial fire spreading for `t` minutes: Each spread operation processes at most `O(m × n)` cells, and we do this at most `t` times where `t ≤ m × n`, giving `O(m × n)` time.
- BFS for the person's movement: The person's BFS visits each cell at most once, taking `O(m × n)` time.
- Fire spreading during person's movement: The fire also spreads while the person moves, visiting each cell at most once throughout the entire process, contributing `O(m × n)` time.
- Grid initialization and copying: `O(m × n)` for initializing the `fire` and `vis` arrays.

Therefore, each `check` function call takes `O(m × n)` time, and with `O(log(m × n))` binary search iterations, the total time complexity is `O(m × n × log(m × n))`.

**Space Complexity:** `O(m × n)`

The algorithm uses:
- `fire` array: `O(m × n)` space to track cells on fire
- `vis` array: `O(m × n)` space to track visited cells during BFS
- Two deques `q1` and `q2`: In the worst case, these can contain `O(m × n)` elements
- Recursive stack space: The algorithm is iterative, so no additional stack space is needed

The dominant space usage is `O(m × n)` from the 2D arrays and queues.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template**

This problem finds the **maximum** waiting time where escape is possible. The correct approach inverts the condition to use the first_true_index template.

**Wrong approach:**
```python
left, right = -1, rows * cols
while left < right:
    mid = (left + right + 1) >> 1
    if can_escape(mid):
        left = mid
    else:
        right = mid - 1
return left
```

**Correct approach (first_true_index template):**
```python
left, right = 0, rows * cols
first_true_index = rows * cols + 1  # Default: all times allow escape

while left <= right:
    mid = (left + right) // 2
    if not can_escape(mid):  # Inverted: find first time where escape FAILS
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

return 10**9 if first_true_index > rows * cols else first_true_index - 1
```

The key insight is to invert the condition: instead of finding where we CAN escape, we find the first time where we CANNOT escape, then return `first_true_index - 1`.

### 2. **Incorrect Handling of the Safehouse Arrival Condition**

One of the most common mistakes is incorrectly handling when the player reaches the safehouse at the same time fire spreads to it. The problem states that "reaching the safehouse counts as safe even if fire spreads to it immediately after you arrive there."

**Pitfall Code:**
```python
# WRONG: Checking if safehouse is on fire before attempting to reach it
if is_on_fire[rows - 1][cols - 1]:
    return False  # This prevents us from reaching safehouse even if we arrive at the same time

# Processing movement...
if new_row == rows - 1 and new_col == cols - 1:
    return True
```

**Correct Approach:**
```python
# Check for safehouse DURING movement processing, not before
for i in range(4):
    new_row = row + directions[i]
    new_col = col + directions[i + 1]

    if (0 <= new_row < rows and
        0 <= new_col < cols and
        not visited[new_row][new_col] and
        not is_on_fire[new_row][new_col] and  # Fire hasn't reached here YET
        grid[new_row][new_col] == 0):

        # Check safehouse immediately after confirming we can move there
        if new_row == rows - 1 and new_col == cols - 1:
            return True  # Safe arrival even if fire spreads here next
```

### 3. **Fire Spreading Order Within BFS**

Another critical pitfall is spreading fire at the wrong time relative to player movement. Fire should spread AFTER each complete round of player movements, not before or during.

**Pitfall Code:**
```python
while person_queue:
    # WRONG: Spreading fire before processing movements
    fire_queue = spread_fire(fire_queue)

    for _ in range(len(person_queue)):
        row, col = person_queue.popleft()
        # Process movement...
```

**Correct Approach:**
```python
while person_queue:
    current_level_size = len(person_queue)

    # Process ALL movements at current distance first
    for _ in range(current_level_size):
        row, col = person_queue.popleft()
        # Process movement...

    # Fire spreads AFTER all movements in this minute complete
    fire_queue = spread_fire(fire_queue)
```

### 4. **Binary Search Boundary Initialization**

A subtle but important pitfall is incorrectly initializing the binary search boundaries, particularly when no waiting is possible.

**Pitfall Code:**
```python
# WRONG: Starting from 0 misses the case where we can't wait at all
left, right = 0, rows * cols

# This would return 0 when we should return -1 for impossible cases
```

**Correct Approach:**
```python
# Start from -1 to handle the case where immediate movement is required
left, right = -1, rows * cols

# This allows the binary search to correctly identify:
# - Return -1 when impossible (left stays at -1)
# - Return 0 when we must move immediately but can escape
# - Return positive values for actual waiting times
```

### 5. **Resetting State Between Binary Search Iterations**

Failing to properly reset the fire state between different binary search attempts can lead to incorrect results.

**Pitfall Code:**
```python
# WRONG: Using a global fire state that persists between checks
is_on_fire = [[False] * cols for _ in range(rows)]  # Defined once

def can_escape_with_delay(wait_time):
    # Forgetting to reset is_on_fire
    # Previous fire positions contaminate new simulation
```

**Correct Approach:**
```python
def can_escape_with_delay(wait_time):
    # Reset fire grid at the start of each check
    for row in range(rows):
        for col in range(cols):
            is_on_fire[row][col] = False

    # Then initialize with original fire positions
    for row in range(rows):
        for col in range(cols):
            if grid[row][col] == 1:
                is_on_fire[row][col] = True
                fire_queue.append((row, col))
```

### 6. **Edge Case: Starting Position Already Has Fire**

Not checking if the starting position (0, 0) initially contains fire or a wall.

**Solution Enhancement:**
```python
# Add this check at the beginning of the main function
if grid[0][0] != 0 or grid[rows-1][cols-1] != 0:
    return -1  # Starting position or safehouse is not grass
```

