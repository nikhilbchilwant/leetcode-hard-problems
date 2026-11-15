# 1293. Shortest Path in a Grid with Obstacles Elimination

> Source: https://algo.monster/liteproblems/1293

## Problem Statement

You have an `m x n` grid where each cell contains either a `0` (representing an empty cell) or a `1` (representing an obstacle). Your task is to find the shortest path from the top-left corner `(0, 0)` to the bottom-right corner `(m-1, n-1)`.

Movement rules:
- You can move in four directions: up, down, left, or right
- Each move counts as one step
- You can only move to empty cells (`0`)
- You cannot normally move through obstacles (`1`)

Special ability:
- You have the power to eliminate up to `k` obstacles
- This means you can walk through at most `k` cells that contain `1`, effectively treating them as `0`

The goal is to return the minimum number of steps needed to reach the destination. If it's impossible to reach the destination even after eliminating up to `k` obstacles, return `-1`.

For example, if you have a 3x3 grid with some obstacles and `k=2`, you need to find the shortest path that goes through at most 2 obstacles to get from the top-left to the bottom-right corner.

The solution uses BFS ([Breadth-First Search](/problems/bfs_intro)) to explore all possible paths. The key insight is to track not just the position `(i, j)` but also the remaining obstacle elimination count at each position. The state `(i, j, k)` represents being at position `(i, j)` with `k` eliminations remaining. The algorithm explores all reachable states level by level, ensuring the first time we reach the destination is via the shortest path.

The optimization `if k >= m + n - 3: return m + n - 2` handles cases where we have enough eliminations to simply go straight right and then down (or down then right), which would be the shortest possible path of `m + n - 2` steps.

## Intuition

The key insight is that this isn't just a simple shortest path problem - we have a special power to eliminate obstacles. This changes how we think about the problem.

In a standard grid shortest path problem, we'd use BFS and track visited cells with a 2D array. But here, visiting the same cell `(i, j)` with different amounts of remaining eliminations represents fundamentally different states. For instance, reaching cell `(2, 3)` with 5 eliminations left is different from reaching it with only 1 elimination left - the first state gives us more flexibility for the rest of the path.

Think of it like having a limited number of "wall-breaking hammers". Each time we encounter an obstacle, we can choose to use a hammer to break through it. The challenge is that we might need to save our hammers for obstacles that block the only path to the destination.

This leads us to expand our state representation from just `(row, column)` to `(row, column, remaining_eliminations)`. Each unique combination represents a different state in our search space. We use BFS because:

1. **Level-by-level exploration**: BFS explores all cells at distance `d` before moving to distance `d+1`, guaranteeing the first time we reach the destination is via the shortest path.

2. **State tracking**: We track visited states as `(i, j, k)` tuples to avoid revisiting the same position with the same elimination count, preventing infinite loops while allowing revisits with different elimination counts.

3. **Branching logic**: At each cell, we can move to an empty neighbor (keeping our elimination count) or move through an obstacle (decreasing our elimination count by 1).

The optimization `if k >= m + n - 3` is clever: if we have enough eliminations to break through all obstacles on the Manhattan distance path (the straight right-then-down or down-then-right path), we can directly return `m + n - 2` steps, which is the theoretical minimum for any grid path from top-left to bottom-right.

**Learn more about [Breadth-First Search](/problems/bfs_intro) patterns.**

## Solution Approach

The implementation uses BFS with a modified state representation to handle the obstacle elimination constraint. Let's walk through the key components:

**1. Initial Optimization Check**
```python
if k >= m + n - 3:
    return m + n - 2
```
This handles the case where we have enough eliminations to simply take the Manhattan distance path. Since the minimum steps from `(0, 0)` to `(m-1, n-1)` is `m + n - 2`, and the maximum obstacles we'd encounter on this path is `m + n - 3` (excluding start and end), we can return immediately.

**2. BFS Setup**
```python
q = deque([(0, 0, k)])
vis = {(0, 0, k)}
ans = 0
```
- Initialize a queue with the starting state: position `(0, 0)` with `k` eliminations available
- Use a set `vis` to track visited states as 3-tuples `(row, col, remaining_eliminations)`
- `ans` tracks the current distance/steps

**3. Level-by-Level BFS Traversal**
```python
while q:
    ans += 1
    for _ in range(len(q)):
        i, j, k = q.popleft()
```
Process all nodes at the current level before moving to the next level. This ensures we find the shortest path.

**4. Exploring Neighbors**
```python
for a, b in [[0, -1], [0, 1], [1, 0], [-1, 0]]:
    x, y = i + a, j + b
```
Check all four directions (up, down, right, left) from the current position.

**5. Boundary and Goal Check**
```python
if 0 <= x < m and 0 <= y < n:
    if x == m - 1 and y == n - 1:
        return ans
```
First verify the new position is within bounds. If we've reached the destination, immediately return the current step count.

**6. Movement Logic**
Two cases for valid moves:

**Empty Cell Movement:**
```python
if grid[x][y] == 0 and (x, y, k) not in vis:
    q.append((x, y, k))
    vis.add((x, y, k))
```
Move to an empty cell without using eliminations. The state is `(x, y, k)` - same elimination count.

**Obstacle Elimination:**
```python
if grid[x][y] == 1 and k > 0 and (x, y, k - 1) not in vis:
    q.append((x, y, k - 1))
    vis.add((x, y, k - 1))
```
Move through an obstacle if we have eliminations left. The new state is `(x, y, k-1)` - one less elimination available.

**7. No Path Found**
```python
return -1
```
If the queue becomes empty without reaching the destination, no valid path exists within the elimination constraint.

The algorithm's time complexity is `O(m * n * k)` since we might visit each cell with different elimination counts, and space complexity is also `O(m * n * k)` for the visited set.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a 3x3 grid with k=1 (we can eliminate at most 1 obstacle):
```
grid = [[0, 1, 0],
        [1, 1, 0],
        [0, 0, 0]]
```

**Initial Setup:**
- Start at (0,0) with k=1 elimination available
- Queue: [(0, 0, 1)]
- Visited: {(0, 0, 1)}
- Steps: 0

**Step 1 (Distance = 1):**
From (0,0), we explore neighbors:
- Right (0,1): Has obstacle (1). We can eliminate it using our power.
  - Add state (0, 1, 0) to queue (0 eliminations left after using one)
- Down (1,0): Has obstacle (1). We can eliminate it.
  - Add state (1, 0, 0) to queue

Queue: [(0, 1, 0), (1, 0, 0)]
Visited: {(0, 0, 1), (0, 1, 0), (1, 0, 0)}

**Step 2 (Distance = 2):**
Process (0, 1, 0):
- Right (0,2): Empty cell (0)
  - Add state (0, 2, 0) to queue
- Down (1,1): Has obstacle (1), but k=0 (no eliminations left), so skip

Process (1, 0, 0):
- Right (1,1): Has obstacle (1), but k=0, so skip
- Down (2,0): Empty cell (0)
  - Add state (2, 0, 0) to queue

Queue: [(0, 2, 0), (2, 0, 0)]
Visited: {(0, 0, 1), (0, 1, 0), (1, 0, 0), (0, 2, 0), (2, 0, 0)}

**Step 3 (Distance = 3):**
Process (0, 2, 0):
- Down (1,2): Empty cell (0)
  - Add state (1, 2, 0) to queue

Process (2, 0, 0):
- Right (2,1): Empty cell (0)
  - Add state (2, 1, 0) to queue

Queue: [(1, 2, 0), (2, 1, 0)]

**Step 4 (Distance = 4):**
Process (1, 2, 0):
- Down (2,2): This is our destination (2, 2)!
  - Return current distance: 4

The shortest path is 4 steps: (0,0) → (0,1) [eliminate] → (0,2) → (1,2) → (2,2)

**Key Observations:**
1. We tracked states as (row, col, remaining_eliminations), not just (row, col)
2. Using our one elimination wisely at (0,1) opened up a path
3. BFS guaranteed we found the shortest path first
4. Alternative paths like going through (1,0) would require going around more obstacles, resulting in longer paths

## Python Solution

```python
class Solution:
    def shortestPath(self, grid: List[List[int]], k: int) -> int:
        # Get grid dimensions
        rows, cols = len(grid), len(grid[0])
      
        # Early optimization: if we have enough eliminations to go straight through
        # The maximum obstacles we'd need to eliminate is (rows - 1) + (cols - 1) - 1
        if k >= rows + cols - 3:
            return rows + cols - 2
      
        # BFS queue: stores (row, col, remaining_eliminations)
        queue = deque([(0, 0, k)])
      
        # Visited set: tracks (row, col, remaining_eliminations) states
        visited = {(0, 0, k)}
      
        # Track current distance/steps
        steps = 0
      
        # BFS to find shortest path
        while queue:
            steps += 1
          
            # Process all nodes at current distance level
            for _ in range(len(queue)):
                current_row, current_col, remaining_k = queue.popleft()
              
                # Check all 4 directions: up, down, right, left
                directions = [[0, -1], [0, 1], [1, 0], [-1, 0]]
                for delta_row, delta_col in directions:
                    next_row = current_row + delta_row
                    next_col = current_col + delta_col
                  
                    # Check if next position is within grid bounds
                    if 0 <= next_row < rows and 0 <= next_col < cols:
                        # Check if we reached the destination
                        if next_row == rows - 1 and next_col == cols - 1:
                            return steps
                      
                        # If next cell is empty (0), move without using elimination
                        if grid[next_row][next_col] == 0:
                            state = (next_row, next_col, remaining_k)
                            if state not in visited:
                                queue.append(state)
                                visited.add(state)
                      
                        # If next cell is obstacle (1), use elimination if available
                        elif grid[next_row][next_col] == 1 and remaining_k > 0:
                            state = (next_row, next_col, remaining_k - 1)
                            if state not in visited:
                                queue.append(state)
                                visited.add(state)
      
        # No path found
        return -1
```

## Time and Space Complexity

**Time Complexity:** `O(m * n * k)`

The algorithm uses BFS to explore the grid. In the worst case, we need to visit each cell `(x, y)` with different remaining obstacle elimination counts from `0` to `k`. Since each state is represented as `(row, col, remaining_k)`, there are at most `m * n * (k + 1)` unique states to explore. Each state is processed once due to the visited set check, and for each state, we examine 4 directions which is `O(1)`. Therefore, the overall time complexity is `O(m * n * k)`.

**Space Complexity:** `O(m * n * k)`

The space complexity is dominated by:
- The `vis` set which stores visited states as tuples `(row, col, remaining_k)`. In the worst case, this can contain up to `m * n * (k + 1)` states, giving `O(m * n * k)` space.
- The queue `q` which in the worst case can also hold `O(m * n * k)` states during BFS traversal.
- Other variables use `O(1)` space.

The early termination optimization `if k >= m + n - 3: return m + n - 2` helps when `k` is large enough to eliminate all obstacles in any Manhattan distance path, but doesn't change the worst-case complexity analysis.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect State Representation in Visited Set**

**Pitfall:** A common mistake is tracking visited cells using only 2D coordinates `(row, col)` instead of the 3D state `(row, col, remaining_eliminations)`.

```python
# WRONG: This will miss valid paths
visited = set()
visited.add((row, col))  # Missing elimination count!
```

**Why it's wrong:** The same cell can be visited multiple times with different elimination counts remaining. For example, reaching cell `(2, 3)` with 5 eliminations left is different from reaching it with 2 eliminations left. The path with more eliminations remaining might lead to a successful route while the other might not.

**Solution:**
```python
# CORRECT: Track the complete state
visited = {(0, 0, k)}  # Include elimination count
state = (next_row, next_col, remaining_k)
if state not in visited:
    visited.add(state)
```

### 2. **Off-by-One Error in Optimization Check**

**Pitfall:** Incorrectly calculating the maximum obstacles on the shortest Manhattan path.

```python
# WRONG: Off by one
if k >= rows + cols - 2:  # This allows too many cases to skip BFS
    return rows + cols - 2
```

**Why it's wrong:** The shortest path has `rows + cols - 2` steps total. The maximum obstacles on this path is `rows + cols - 3` (excluding the guaranteed empty start and end cells). Using `rows + cols - 2` would incorrectly assume we might need to eliminate the destination cell itself.

**Solution:**
```python
# CORRECT: Maximum obstacles is total steps minus 1
if k >= rows + cols - 3:
    return rows + cols - 2
```

### 3. **Returning Steps Before Incrementing**

**Pitfall:** Forgetting that BFS starts at distance 0, not 1.

```python
# WRONG: Returns one less than actual distance
steps = 0
while queue:
    for _ in range(len(queue)):
        # ... process current level
        if next_row == rows - 1 and next_col == cols - 1:
            return steps  # Should be steps + 1!
    steps += 1
```

**Solution:** Either increment `steps` at the beginning of each level iteration (as shown in the correct code), or return `steps + 1` when the destination is found.

### 4. **Not Handling the Starting Cell Correctly**

**Pitfall:** Assuming the starting cell `(0, 0)` is always empty.

```python
# WRONG: Doesn't check if start is an obstacle
if grid[0][0] == 1:
    # Need to handle this case!
```

**Why it matters:** If the starting cell contains an obstacle, you need to use one elimination immediately, starting with `k-1` eliminations instead of `k`.

**Solution:**
```python
# CORRECT: Handle starting cell obstacle
initial_k = k if grid[0][0] == 0 else k - 1
if initial_k < 0:
    return -1  # Can't even start
queue = deque([(0, 0, initial_k)])
visited = {(0, 0, initial_k)}
```

### 5. **Edge Case: Single Cell Grid**

**Pitfall:** Not handling the case where `m = 1` and `n = 1`.

**Solution:** Add a check at the beginning:
```python
if rows == 1 and cols == 1:
    return 0  # Already at destination
```

