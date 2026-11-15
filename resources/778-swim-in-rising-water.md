# 778. Swim in Rising Water

> Source: https://algo.monster/liteproblems/778

## Problem Statement

You have an `n x n` grid where each cell contains an integer representing the elevation at that position. When it starts raining, the water level rises uniformly over time. At any time `t`, the water level is at height `t`, which means any cell with elevation ≤ `t` becomes submerged (accessible for swimming).

You start at the top-left corner `(0, 0)` and want to reach the bottom-right corner `(n-1, n-1)`. You can swim from one cell to any of its 4 adjacent cells (up, down, left, right) only if both cells have elevations ≤ `t` at time `t`. Swimming takes no time - you can travel infinite distances instantly as long as the cells are accessible.

The goal is to find the minimum time `t` at which you can swim from the starting position to the ending position. In other words, what's the smallest water level needed so that there exists a valid path from `(0, 0)` to `(n-1, n-1)` where all cells along the path are submerged?

For example, if you have a 2x2 grid:
```
[[0, 2],
 [1, 3]]
```
At time `t = 3`, all cells are submerged (have elevation ≤ 3), allowing you to swim from `(0,0)` to `(1,1)`. The answer would be 3 since you need to wait until the water reaches level 3 to have a complete path.

## Intuition

Think about what happens as the water level rises from 0 to higher values. Initially, at time `t = 0`, only cells with elevation 0 are accessible. As time increases to `t = 1`, cells with elevation ≤ 1 become accessible, and so on.

The key insight is that we're looking for the exact moment when a path first becomes available from start to end. Before this critical time, the start and end positions are in separate "islands" of accessible cells. At the critical time, these islands merge into one connected region.

This naturally leads to a Union-Find (Disjoint Set Union) approach. We can simulate the water rising by processing cells in order of their elevation. For each elevation level:
1. The cell at that elevation becomes "active" (submerged)
2. We check its 4 neighbors - if they're already submerged (elevation ≤ current time), we unite them into the same connected component
3. After each union operation, we check if the start `(0, 0)` and end `(n-1, n-1)` are now in the same connected component

The moment they become connected is our answer - that's the minimum water level needed.

Why is this more efficient than trying different water levels with DFS? Instead of repeatedly checking connectivity at different water levels (which would involve multiple graph traversals), we build the connectivity incrementally. We process each cell exactly once in elevation order, and union operations are nearly constant time with path compression. This gives us an elegant `O(n² log n)` solution (dominated by sorting if needed) or `O(n²)` if we process elevations from 0 to n²-1 in order.

The solution cleverly uses the array `elevation_to_position` to map each elevation value to its position in the grid, allowing us to process cells in the exact order they become submerged.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro), [Union Find](/problems/dsu_intro), [Binary Search](/problems/binary_search_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation uses Union-Find (Disjoint Set Union) to efficiently track when the start and end positions become connected as the water level rises.

**Data Structures:**
- `parent`: Parent array for Union-Find, where `parent[i]` stores the parent of node `i`. Initially, each cell is its own parent.
- `elevation_to_position`: Maps each elevation value to its flattened grid position. Since elevations range from 0 to n²-1, `elevation_to_position[elevation] = row * n + col`.

**Algorithm Steps:**

1. **Initialize Union-Find structure:**
   - Create parent array `parent` where each cell (flattened to 1D index) is initially its own parent
   - A cell at position `(i, j)` is mapped to index `i * n + j`

2. **Build elevation-to-position mapping:**
   ```python
   for i, row in enumerate(grid):
       for j, h in enumerate(row):
           elevation_to_position[h] = i * n + j
   ```
   This allows us to process cells in order of increasing elevation.

3. **Process cells by increasing elevation (time):**
   ```python
   for time in range(n * n):
       current_cell_idx = elevation_to_position[time]
       current_row = current_cell_idx // n
       current_col = current_cell_idx % n
   ```
   At time `time`, the cell with elevation `t` becomes active. We extract its 2D coordinates from the flattened index.

4. **Union with adjacent submerged cells:**
   - For each of the 4 adjacent cells `(x, y)`:
     - Check if it's within bounds and already submerged (`grid[x][y] <= time`)
     - If yes, union it with the current cell using `parent[find(x * n + y)] = find(current_cell_idx)`

5. **Check connectivity:**
   - After each neighbor is examined (inside the neighbor loop), check if `find(0) == find(n * n - 1)`
   - If the start (index 0) and end (index n²-1) are in the same component, return the current `time`

**Find function with path compression:**
```python
def find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])  # Path compression
    return parent[x]
```

The algorithm guarantees we find the minimum time because we process cells in strictly increasing order of elevation. The first moment the start and end become connected is the minimum water level needed for a valid path to exist.

**Time Complexity:** O(n² log n) in the worst case, because `find` uses path compression without union by rank; in practice it runs close to O(n²)
**Space Complexity:** O(n²) for the parent array and elevation mapping

## Example Walkthrough

Let's walk through a small 3x3 grid example to illustrate the solution approach:

```
Grid:
[[0, 6, 7],
 [2, 8, 1],
 [3, 4, 5]]
```

**Step 1: Initialize Union-Find**
- Create parent array `parent` where each cell is its own parent
- `p = [0, 1, 2, 3, 4, 5, 6, 7, 8]` (each index points to itself)
- Flattened indices: (0,0)→0, (0,1)→1, (0,2)→2, (1,0)→3, (1,1)→4, (1,2)→5, (2,0)→6, (2,1)→7, (2,2)→8

**Step 2: Build elevation-to-position mapping**
```
elevation_to_position[0] = 0 (position 0,0)
elevation_to_position[1] = 5 (position 1,2)
elevation_to_position[2] = 3 (position 1,0)
elevation_to_position[3] = 6 (position 2,0)
elevation_to_position[4] = 7 (position 2,1)
elevation_to_position[5] = 8 (position 2,2)
elevation_to_position[6] = 1 (position 0,1)
elevation_to_position[7] = 2 (position 0,2)
elevation_to_position[8] = 4 (position 1,1)
```

**Step 3: Process cells by increasing elevation**

**Time t=0:** Cell (0,0) with elevation 0 becomes active
- Check neighbors: No neighbors are submerged yet
- Components: {0}, {1}, {2}, {3}, {4}, {5}, {6}, {7}, {8}
- Start (0) and end (8) not connected

**Time t=1:** Cell (1,2) with elevation 1 becomes active
- Check neighbors: No neighbors are submerged yet
- Components: {0}, {1}, {2}, {3}, {4}, {5}, {6}, {7}, {8}
- Start (0) and end (8) not connected

**Time t=2:** Cell (1,0) with elevation 2 becomes active
- Check neighbor (0,0): elevation 0 ≤ 2, so union them
- Union operation: `parent[find(0)] = find(3)` → Components merge
- Components: {0,3}, {1}, {2}, {4}, {5}, {6}, {7}, {8}
- Start (0) and end (8) not connected

**Time t=3:** Cell (2,0) with elevation 3 becomes active
- Check neighbor (1,0): elevation 2 ≤ 3, so union them
- Union operation: `parent[find(3)] = find(6)` → Components merge
- Components: {0,3,6}, {1}, {2}, {4}, {5}, {7}, {8}
- Start (0) and end (8) not connected

**Time t=4:** Cell (2,1) with elevation 4 becomes active
- Check neighbor (2,0): elevation 3 ≤ 4, so union them
- Union operation: `parent[find(6)] = find(7)` → Components merge
- Components: {0,3,6,7}, {1}, {2}, {4}, {5}, {8}
- Start (0) and end (8) not connected

**Time t=5:** Cell (2,2) with elevation 5 becomes active
- Check neighbor (2,1) first (the left direction): elevation 4 ≤ 5, so union them with `parent[find(7)] = find(8)`
- Components: {0,3,6,7,8}, {1}, {2}, {4}, {5}
- The connectivity check right after this neighbor finds that start (0) and end (8) are connected
- Return 5 (the remaining neighbors, including (1,2), are never examined)

The minimum water level needed is 5. At this level, we can swim from (0,0) → (1,0) → (2,0) → (2,1) → (2,2), forming a valid path where all cells have elevation ≤ 5.

## Python Solution

```python
class Solution:
    def swimInWater(self, grid: List[List[int]]) -> int:
        """
        Find minimum time to swim from top-left to bottom-right.
        Uses Union-Find to connect cells as water level rises.

        Args:
            grid: 2D grid where grid[i][j] represents elevation at (i, j)

        Returns:
            Minimum time required to reach destination
        """

        def find(node: int) -> int:
            """Find root of the set containing node with path compression"""
            if parent[node] != node:
                parent[node] = find(parent[node])  # Path compression
            return parent[node]

        n = len(grid)

        # Initialize Union-Find structure
        # Each cell (i, j) is mapped to index i * n + j
        parent = list(range(n * n))

        # Create mapping from elevation to cell position
        # elevation_to_position[h] stores the flattened index of cell with elevation h
        elevation_to_position = [0] * (n * n)
        for row_idx, row in enumerate(grid):
            for col_idx, elevation in enumerate(row):
                elevation_to_position[elevation] = row_idx * n + col_idx

        # Process cells in order of increasing elevation (time)
        for time in range(n * n):
            # Get coordinates of cell with current elevation
            current_cell_idx = elevation_to_position[time]
            current_row = current_cell_idx // n
            current_col = current_cell_idx % n

            # Check all four adjacent cells
            directions = [(0, -1), (0, 1), (1, 0), (-1, 0)]  # left, right, down, up
            for delta_row, delta_col in directions:
                neighbor_row = current_row + delta_row
                neighbor_col = current_col + delta_col

                # If neighbor is valid and has elevation <= current time, union them
                if (0 <= neighbor_row < n and
                    0 <= neighbor_col < n and
                    grid[neighbor_row][neighbor_col] <= time):

                    neighbor_idx = neighbor_row * n + neighbor_col
                    # Union the two cells
                    parent[find(neighbor_idx)] = find(current_cell_idx)

                # Check if start and end are connected
                start_idx = 0  # Top-left corner (0, 0)
                end_idx = n * n - 1  # Bottom-right corner (n-1, n-1)
                if find(start_idx) == find(end_idx):
                    return time

        return -1  # Should never reach here for valid input
```

## Time and Space Complexity

**Time Complexity:** `O(n² log n)` in the worst case, where `n` is the dimension of the grid.

- Creating the position mapping array `elevation_to_position` takes `O(n²)` time as we iterate through all grid cells
- The main loop runs for at most `n²` iterations (from `t = 0` to `t = n² - 1`)
- In each iteration, we check 4 neighbors, performing at most 4 union operations
- Each union operation involves two `find` operations, and the connectivity check after each neighbor involves two more
- The `find` function uses path compression but not union by rank, so a `find` takes `O(log(n²)) = O(log n)` amortized time in the worst case
- Total: `O(n²) + O(n² × 4 × log n)` = `O(n² log n)`

Adding union by rank (or by size) would bring the amortized cost of `find` down to `O(α(n²))`, where `α` is the inverse Ackermann function, giving `O(n² × α(n²))`, which is effectively `O(n²)`.

**Space Complexity:** `O(n²)`

- The parent array `parent` stores `n²` elements: `O(n²)`
- The height-to-position mapping array `elevation_to_position` stores `n²` elements: `O(n²)`
- The recursion stack for `find` can be as deep as the tallest tree in the Union-Find forest, which is at most `O(n²)`
- Total: `O(n²) + O(n²) + O(n²)` = `O(n²)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Union Operation Without Find**
A critical mistake is performing union operations without properly using the `find` function to get the root representatives:

**Incorrect:**
```python
# Wrong: directly assigning parents without finding roots
parent[neighbor_idx] = current_cell_idx
```

**Correct:**
```python
# Correct: union the roots of both sets
parent[find(neighbor_idx)] = find(current_cell_idx)
```

Without using `find`, you're not actually merging the entire connected components but just reassigning individual nodes, which breaks the Union-Find structure.

### 2. **Where to Place the Connectivity Check**
The solution checks `find(0) == find(n * n - 1)` inside the neighbor loop, so the check runs after each neighbor is examined. This is correct: connectivity never goes away once it appears, and any time the check succeeds at level `time`, the start and end really are connected using only cells with elevation `≤ time`. The check after the last neighbor also covers the state after all unions for that cell, and it handles `n = 1` because the loop body runs even when no neighbor is in bounds.

Moving the check outside the neighbor loop gives the same answers and calls `find` fewer times:
```python
for time in range(n * n):
    current_cell_idx = elevation_to_position[time]
    current_row = current_cell_idx // n
    current_col = current_cell_idx % n

    # Process all neighbors first
    for delta_row, delta_col in directions:
        neighbor_row = current_row + delta_row
        neighbor_col = current_col + delta_col

        if (0 <= neighbor_row < n and
            0 <= neighbor_col < n and
            grid[neighbor_row][neighbor_col] <= time):
            neighbor_idx = neighbor_row * n + neighbor_col
            parent[find(neighbor_idx)] = find(current_cell_idx)

    # Then check connectivity once after all unions
    if find(0) == find(n * n - 1):
        return time
```

The mistake to avoid is placing the check inside the `if` block, so that it only runs after a union. For `n = 1` no neighbor is ever in bounds, the check never runs, and the function returns `-1` instead of `0`.

### 3. **Confusion Between Elevation Values and Time**
The problem states elevations are unique and range from 0 to n²-1, which conveniently matches the time progression. However, if the elevation values were arbitrary (not necessarily 0 to n²-1), the current approach would fail.

**More Robust Solution for Arbitrary Elevations:**
```python
# Sort all cells by elevation
cells = []
for i in range(n):
    for j in range(n):
        cells.append((grid[i][j], i, j))
cells.sort()

# Process in order of increasing elevation
for elevation, row, col in cells:
    # Union with already-processed neighbors
    for dr, dc in directions:
        nr, nc = row + dr, col + dc
        if (0 <= nr < n and 0 <= nc < n and
            grid[nr][nc] <= elevation):
            # Union operation
            parent[find(nr * n + nc)] = find(row * n + col)

    # Check connectivity
    if find(0) == find(n * n - 1):
        return elevation
```

### 4. **Missing Path Compression Optimization**
While the find function includes path compression, ensure it's consistently applied. Without it, the find operation could degrade to O(n) in worst case, making the overall complexity O(n³).

**Always use path compression:**
```python
def find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])  # Compress path
    return parent[x]
```

