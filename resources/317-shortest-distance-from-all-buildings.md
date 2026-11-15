# 317. Shortest Distance from All Buildings

> Source: https://algo.monster/liteproblems/317

## Problem Statement

You have an `m x n` grid where each cell contains one of three values:
- `0`: Empty land that you can walk through freely
- `1`: A building that you cannot pass through
- `2`: An obstacle that you cannot pass through

Your goal is to find the best location on an empty land (a cell with value `0`) to build a house such that the total travel distance from this house to all buildings is minimized.

Movement is restricted to four directions: up, down, left, and right (no diagonal movement).

The total travel distance is calculated as the sum of the shortest path distances from your chosen location to each building in the grid. The distance between cells is measured using the Manhattan distance (number of steps needed when moving only horizontally or vertically).

Return the minimum total travel distance if it's possible to find such a location that can reach all buildings. If no valid location exists (for example, if some buildings are unreachable from any empty land), return `-1`.

**Example scenario:** If there are 3 buildings in the grid and you choose an empty cell, you need to calculate the shortest path from that cell to each of the 3 buildings, then sum these distances. The answer would be the empty cell that gives you the smallest sum.

**Key constraints:**
- You can only build on empty land (cells with `0`)
- The chosen location must be able to reach ALL buildings in the grid
- Obstacles and buildings block movement paths

## Intuition

The key insight is to reverse our thinking. Instead of starting from each empty land and finding distances to all buildings (which would be inefficient), we start from each building and find distances to all reachable empty lands.

Think about it this way: if we have 3 buildings and 20 empty cells, it's more efficient to run BFS 3 times (once from each building) rather than 20 times (once from each empty cell).

Here's the thought process:

1. **Why BFS from buildings?** Each building needs to contribute its distance to every potential house location. By running BFS from each building, we can calculate how far that building is from every empty cell in one traversal.

2. **Accumulating distances**: As we run BFS from each building, we maintain two key pieces of information for each empty cell:
   - `dist[i][j]`: The total sum of distances from all buildings processed so far
   - `cnt[i][j]`: How many buildings can reach this cell

3. **The aggregation trick**: After running BFS from all buildings, each empty cell will have accumulated the sum of distances to all buildings that can reach it. This is exactly what we need - the total travel distance!

4. **Handling unreachable cells**: Some empty cells might be blocked from certain buildings by obstacles. That's why we track `cnt[i][j]`. Only cells where `cnt[i][j] == total_buildings` are valid locations (they can reach all buildings).

5. **Finding the minimum**: Once we have all the accumulated distances, we simply scan through all valid empty cells and pick the one with the minimum total distance.

The beauty of this approach is that it transforms a complex pathfinding problem into a simple accumulation problem. Each BFS adds its "layer" of distances, and at the end, we have a complete picture of the total travel distances for all potential locations.

## Solution Approach

Let's walk through the implementation step by step:

**1. Initialize Data Structures:**
- `total`: Counter for the total number of buildings in the grid
- `cnt`: 2D array tracking how many buildings can reach each empty cell
- `dist`: 2D array accumulating the total distance from all buildings to each empty cell
- `q`: Deque for BFS traversal

**2. Find All Buildings and Run BFS from Each:**
```python
for i in range(m):
    for j in range(n):
        if grid[i][j] == 1:  # Found a building
            total += 1
            q.append((i, j))
            # Run BFS from this building
```

**3. BFS Implementation from Each Building:**
- Start BFS from building position `(i, j)`
- Use `d` to track the current distance level (starts at 0, increments with each layer)
- Use `vis` set to avoid revisiting cells in the same BFS run

```python
d = 0
vis = set()
while q:
    d += 1  # Move to next distance level
    for _ in range(len(q)):  # Process all cells at current distance
        r, c = q.popleft()
        # Check all 4 directions
        for a, b in [[0, 1], [0, -1], [1, 0], [-1, 0]]:
            x, y = r + a, c + b
```

**4. Process Each Valid Empty Cell:**
For each adjacent cell `(x, y)`:
- Check if it's within bounds: `0 <= x < m and 0 <= y < n`
- Check if it's empty land: `grid[x][y] == 0`
- Check if not visited in this BFS: `(x, y) not in vis`

If all conditions are met:
```python
cnt[x][y] += 1      # One more building can reach this cell
dist[x][y] += d     # Add distance from current building
q.append((x, y))    # Add to queue for next level
vis.add((x, y))     # Mark as visited
```

**5. Find the Optimal Location:**
After BFS from all buildings:
```python
ans = inf
for i in range(m):
    for j in range(n):
        if grid[i][j] == 0 and cnt[i][j] == total:
            ans = min(ans, dist[i][j])
```

Only consider empty cells where `cnt[i][j] == total` (reachable from all buildings). Among these valid locations, choose the one with minimum `dist[i][j]`.

**6. Return Result:**
- If `ans` remains `inf`, no valid location exists, return `-1`
- Otherwise, return `ans` (the minimum total distance)

**Key Optimizations:**
- **Level-order BFS**: Processing all nodes at the same distance level together ensures we calculate the shortest paths
- **Single Pass Accumulation**: Instead of storing individual distances and summing later, we accumulate as we go
- **Early Validation**: By tracking `cnt`, we can quickly identify which cells are valid candidates

The time complexity is `O(m * n * k)` where `k` is the number of buildings, as we run BFS from each building. Space complexity is `O(m * n)` for the auxiliary arrays.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this 3x3 grid:
```
[1, 0, 0]
[0, 0, 0]
[0, 0, 1]
```
Where `1` = building, `0` = empty land

**Step 1: Initialize data structures**
- `total = 0` (building counter)
- `cnt = [[0,0,0], [0,0,0], [0,0,0]]` (tracks building reach)
- `dist = [[0,0,0], [0,0,0], [0,0,0]]` (accumulates distances)

**Step 2: Process first building at (0,0)**
- `total = 1`
- Run BFS from (0,0):
  - Distance level 1: Reach (0,1) and (1,0)
    - `cnt[0][1] = 1`, `dist[0][1] = 1`
    - `cnt[1][0] = 1`, `dist[1][0] = 1`
  - Distance level 2: Reach (0,2), (1,1), (2,0)
    - `cnt[0][2] = 1`, `dist[0][2] = 2`
    - `cnt[1][1] = 1`, `dist[1][1] = 2`
    - `cnt[2][0] = 1`, `dist[2][0] = 2`
  - Distance level 3: Reach (1,2), (2,1)
    - `cnt[1][2] = 1`, `dist[1][2] = 3`
    - `cnt[2][1] = 1`, `dist[2][1] = 3`

After first building:
```
cnt:  [[0,1,1], [1,1,1], [1,1,0]]
dist: [[0,1,2], [1,2,3], [2,3,0]]
```

**Step 3: Process second building at (2,2)**
- `total = 2`
- Run BFS from (2,2):
  - Distance level 1: Reach (2,1) and (1,2)
    - `cnt[2][1] = 2`, `dist[2][1] = 3 + 1 = 4`
    - `cnt[1][2] = 2`, `dist[1][2] = 3 + 1 = 4`
  - Distance level 2: Reach (2,0), (1,1), (0,2)
    - `cnt[2][0] = 2`, `dist[2][0] = 2 + 2 = 4`
    - `cnt[1][1] = 2`, `dist[1][1] = 2 + 2 = 4`
    - `cnt[0][2] = 2`, `dist[0][2] = 2 + 2 = 4`
  - Distance level 3: Reach (1,0), (0,1)
    - `cnt[1][0] = 2`, `dist[1][0] = 1 + 3 = 4`
    - `cnt[0][1] = 2`, `dist[0][1] = 1 + 3 = 4`

Final state:
```
cnt:  [[0,2,2], [2,2,2], [2,2,0]]
dist: [[0,4,4], [4,4,4], [4,4,0]]
```

**Step 4: Find optimal location**
- Check all cells where `grid[i][j] == 0` and `cnt[i][j] == total (2)`:
  - (0,1): valid, total distance = 4
  - (0,2): valid, total distance = 4
  - (1,0): valid, total distance = 4
  - (1,1): valid, total distance = 4
  - (1,2): valid, total distance = 4
  - (2,0): valid, total distance = 4
  - (2,1): valid, total distance = 4

**Result:** The minimum total distance is 4. Any of these empty cells would be optimal. The cell at (1,1) is particularly interesting as it's centrally located - it's 2 steps from each building (1→1 diagonally translates to 2 Manhattan distance steps).

This example demonstrates how:
- BFS from each building accumulates distances
- The `cnt` array ensures we only consider cells reachable from all buildings
- The final `dist` values give us the total travel distance for each valid location

## Python Solution

```python
from collections import deque
from typing import List
from math import inf

class Solution:
    def shortestDistance(self, grid: List[List[int]]) -> int:
        # Get grid dimensions
        rows, cols = len(grid), len(grid[0])

        # Track total number of buildings
        total_buildings = 0

        # Count matrix: tracks how many buildings can reach each empty cell
        building_reach_count = [[0] * cols for _ in range(rows)]

        # Distance matrix: accumulates total distance from all buildings to each empty cell
        total_distance = [[0] * cols for _ in range(rows)]

        # Process each cell in the grid
        for i in range(rows):
            for j in range(cols):
                # If current cell is a building
                if grid[i][j] == 1:
                    total_buildings += 1

                    # BFS from this building to all reachable empty cells
                    queue = deque([(i, j)])
                    visited = set()
                    current_distance = 0

                    while queue:
                        current_distance += 1
                        # Process all cells at current distance level
                        level_size = len(queue)

                        for _ in range(level_size):
                            row, col = queue.popleft()

                            # Check all 4 directions
                            directions = [[0, 1], [0, -1], [1, 0], [-1, 0]]
                            for delta_row, delta_col in directions:
                                new_row, new_col = row + delta_row, col + delta_col

                                # Check if new position is valid empty cell and not visited
                                if (0 <= new_row < rows and
                                    0 <= new_col < cols and
                                    grid[new_row][new_col] == 0 and
                                    (new_row, new_col) not in visited):

                                    # Update count and distance for this empty cell
                                    building_reach_count[new_row][new_col] += 1
                                    total_distance[new_row][new_col] += current_distance

                                    # Add to queue and mark as visited
                                    queue.append((new_row, new_col))
                                    visited.add((new_row, new_col))

        # Find the minimum distance among all empty cells reachable by all buildings
        min_distance = inf
        for i in range(rows):
            for j in range(cols):
                # Check if cell is empty and reachable by all buildings
                if grid[i][j] == 0 and building_reach_count[i][j] == total_buildings:
                    min_distance = min(min_distance, total_distance[i][j])

        # Return result: -1 if no valid meeting point exists, otherwise minimum distance
        return -1 if min_distance == inf else min_distance
```

## Time and Space Complexity

**Time Complexity:** `O(m² * n²)`

The algorithm performs a BFS from each building (grid value = 1) to find distances to all reachable empty lands. In the worst case:
- There can be up to `O(m * n)` buildings in the grid
- For each building, we perform a BFS that potentially visits all `O(m * n)` cells
- Each cell is processed once per BFS with constant time operations
- Therefore, the overall time complexity is `O(m * n) * O(m * n) = O(m² * n²)`

**Space Complexity:** `O(m * n)`

The space usage consists of:
- `cnt` matrix: `O(m * n)` to track how many buildings can reach each empty cell
- `dist` matrix: `O(m * n)` to store cumulative distances from all buildings
- `vis` set: `O(m * n)` in the worst case when BFS visits all cells
- `q` deque: `O(m * n)` in the worst case when all cells are in the queue
- The `vis` set and `q` deque are reused for each building's BFS, not accumulated

The total space complexity is `O(m * n)` as all components scale linearly with the grid size.

## Common Pitfalls

### 1. **Letting the BFS Walk Through Buildings or Obstacles**
The BFS from one building must expand only into empty land. A common mistake is checking only bounds and the visited set, so the search passes through other buildings (`1`) or obstacles (`2`).

**Pitfall Example:**
```python
# WRONG: no check of the cell type
if (0 <= new_row < rows and 0 <= new_col < cols and
        (new_row, new_col) not in visited):
    queue.append((new_row, new_col))
```

Paths would then run through buildings and obstacles, producing distances that are too short and marking blocked cells as reachable.

**Solution:**
Require `grid[new_row][new_col] == 0` before visiting a neighbor, as the solution does. Because of that check, the starting building itself is never re-added, so it does not matter whether its own position is placed in `visited`.

### 2. **Incorrect Distance Tracking in BFS Levels**
Another critical mistake is updating the distance counter at the wrong time, leading to off-by-one errors in distance calculations.

**Pitfall Example:**
```python
# WRONG: Distance incremented for each cell, not each level
while queue:
    row, col = queue.popleft()
    current_distance += 1  # Problem: increments for every cell
    for delta_row, delta_col in directions:
        # Distance becomes incorrect for cells at the same level
```

**Solution:**
Increment distance once per BFS level, not per cell:
```python
while queue:
    current_distance += 1  # Increment once for the entire level
    level_size = len(queue)
    for _ in range(level_size):  # Process all cells at current distance
        row, col = queue.popleft()
        # All cells processed in this loop are at the same distance
```

### 3. **Reusing Visited Set Across Different Building BFS Runs**
Using a global visited set for all buildings instead of resetting it for each building's BFS can cause cells to be incorrectly marked as unreachable.

**Pitfall Example:**
```python
# WRONG: Global visited set
visited = set()  # Defined outside the building loop
for i in range(rows):
    for j in range(cols):
        if grid[i][j] == 1:
            # Uses the same visited set, preventing cells from being
            # reached by subsequent buildings
            queue = deque([(i, j)])
            # BFS continues with contaminated visited set
```

**Solution:**
Create a fresh visited set for each building's BFS:
```python
for i in range(rows):
    for j in range(cols):
        if grid[i][j] == 1:
            queue = deque([(i, j)])
            visited = set()  # Fresh visited set for each building
            # Each building gets its own independent BFS traversal
```

### 4. **Not Handling Edge Case of No Buildings or No Empty Land**
The code might fail or return incorrect results when the grid contains no buildings (nothing to connect to) or no empty land (nowhere to build).

**Solution:**
Add validation checks:
```python
# After counting buildings and finding minimum distance
if total_buildings == 0:
    return -1  # No buildings to connect to

# The existing check for min_distance == inf handles the no empty land case
return -1 if min_distance == inf else min_distance
```

