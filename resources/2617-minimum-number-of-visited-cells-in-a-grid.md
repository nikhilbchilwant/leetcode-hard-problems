# 2617. Minimum Number of Visited Cells in a Grid

> Source: https://algo.monster/liteproblems/2617

## Problem Statement

You have an `m x n` integer matrix called `grid`, and you start at the top-left cell `(0, 0)`. Your goal is to reach the bottom-right cell `(m-1, n-1)` with the minimum number of cell visits.

From any cell `(i, j)`, you can only move in two ways:
- **Move right**: Jump to any cell `(i, k)` where `j < k <= grid[i][j] + j`. This means you can jump rightward to any cell within a range determined by the current cell's value.
- **Move down**: Jump to any cell `(k, j)` where `i < k <= grid[i][j] + i`. This means you can jump downward to any cell within a range determined by the current cell's value.

The value `grid[i][j]` determines how far you can jump from that cell. For example, if you're at cell `(2, 3)` and `grid[2][3] = 4`, you can:
- Jump right to any cell `(2, k)` where `3 < k <= 7` (since `4 + 3 = 7`)
- Jump down to any cell `(k, 3)` where `2 < k <= 6` (since `4 + 2 = 6`)

The problem asks you to find the minimum number of cells you need to visit (including the starting and ending cells) to reach from `(0, 0)` to `(m-1, n-1)`. If it's impossible to reach the destination, return `-1`.

## Intuition

When we think about finding the minimum number of cells to visit, we're essentially looking for the shortest path from start to finish. The key insight is that this is a graph problem where each cell can "jump" to multiple other cells based on its value.

The standard BFS approach would work, but there's a catch - from each cell `(i, j)`, we might be able to jump to many cells (potentially up to `grid[i][j]` cells in each direction). If we naively add all reachable cells to our BFS queue, we could end up with a lot of redundant work.

Here's where the clever optimization comes in: instead of thinking "from this cell, where can I go?", we flip the perspective to "to reach this cell, where could I have come from?". This reverse thinking leads us to maintain priority queues for each row and column.

For any cell `(i, j)` we're trying to reach:
- It could be reached from any cell in the same row to its left (cells `(i, j')` where `j' < j`)
- It could be reached from any cell in the same column above it (cells `(i', j)` where `i' < i`)

But not all cells to the left or above can actually reach `(i, j)`. A cell `(i, j')` can only reach `(i, j)` if `grid[i][j'] + j' >= j`. Similarly for cells above.

The priority queue optimization works because:
1. We process cells in order (row by row, column by column)
2. For each row `i`, we maintain a min-heap of cells that could potentially jump to future cells in that row
3. As we move right in a row, some cells become "expired" - they can no longer reach the current position, so we remove them
4. The cell with minimum distance that can still reach our current position is always at the top of the heap

This transforms what could be an expensive operation (checking all possible source cells) into an efficient one using heaps. The `dist[i][j]` tracks the minimum cells visited to reach `(i, j)`, and we update it based on the best option from either the row heap or column heap.

**Learn more about [Stack](/problems/stack_intro), [Breadth-First Search](/problems/bfs_intro), [Union Find](/problems/dsu_intro), [Dynamic Programming](/problems/dynamic_programming_intro), [Monotonic Stack](/problems/mono_stack_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

Let's denote the number of rows of the grid as `m` and the number of columns as `n`. We define `dist[i][j]` to be the shortest distance from the coordinate `(0, 0)` to the coordinate `(i, j)`. Initially, `dist[0][0] = 1` (starting cell counts as 1 visit) and `dist[i][j] = -1` for all other positions (unreachable by default).

The key idea is to maintain priority queues (min-heaps) for efficient lookups:
- `row[i]`: A min-heap for row `i` containing pairs `(dist[i][j], j)` representing cells in that row that could potentially jump to cells further right
- `col[j]`: A min-heap for column `j` containing pairs `(dist[i][j], i)` representing cells in that column that could potentially jump to cells further down

The algorithm processes cells in row-major order (left to right, top to bottom):

1. **For each cell `(i, j)`:**

2. **Check horizontal jumps (from left cells in the same row):**
   - Clean up expired entries: Remove cells from `row[i]` heap that can't reach position `j` (where `grid[i][j'] + j' < j`)
   - If valid cells remain in the heap, the top element gives the minimum distance to reach `(i, j)` from the left
   - Update `dist[i][j] = row[i][0][0] + 1` if this gives a better path

3. **Check vertical jumps (from upper cells in the same column):**
   - Clean up expired entries: Remove cells from `col[j]` heap that can't reach position `i` (where `grid[i'][j] + i' < i`)
   - If valid cells remain in the heap, the top element gives the minimum distance to reach `(i, j)` from above
   - Update `dist[i][j] = col[j][0][0] + 1` if this gives a better path

4. **Add current cell to heaps:**
   - If `dist[i][j] != -1` (cell is reachable), add `(dist[i][j], j)` to `row[i]` and `(dist[i][j], i)` to `col[j]`
   - These entries will be considered for future cells that might be reachable from `(i, j)`

The algorithm efficiently maintains the invariant that the top of each heap contains the cell with minimum distance that can still reach future positions. By processing cells in order and using heaps to track the best sources, we avoid redundant checks while ensuring we find the optimal path.

Finally, `dist[m-1][n-1]` gives us the minimum number of cells to visit to reach the bottom-right corner, or `-1` if it's unreachable.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a 3×3 grid:
```
grid = [[2, 1, 2],
        [1, 1, 1],
        [1, 1, 1]]
```

We want to find the minimum cells to visit from (0,0) to (2,2).

**Initial Setup:**
- `dist` matrix initialized with -1 everywhere except `dist[0][0] = 1`
- `row[0], row[1], row[2]` are empty min-heaps for each row
- `col[0], col[1], col[2]` are empty min-heaps for each column

**Processing (0,0):**
- Current cell value: `grid[0][0] = 2`
- No cells to the left or above, so `dist[0][0]` remains 1
- Add to heaps: `row[0].push((1, 0))` and `col[0].push((1, 0))`
- From (0,0), we can jump right to (0,1) or (0,2), or down to (1,0) or (2,0)

**Processing (0,1):**
- Check row[0] heap: Contains (1, 0). Since `grid[0][0] + 0 = 2 + 0 = 2 >= 1`, cell (0,0) can reach (0,1)
- Update: `dist[0][1] = 1 + 1 = 2`
- Add to heaps: `row[0].push((2, 1))` and `col[1].push((2, 0))`

**Processing (0,2):**
- Check row[0] heap: Top is (1, 0). Since `grid[0][0] + 0 = 2 >= 2`, (0,0) can reach (0,2)
- Update: `dist[0][2] = 1 + 1 = 2`
- Add to heaps: `row[0].push((2, 2))` and `col[2].push((2, 0))`

**Processing (1,0):**
- Check col[0] heap: Contains (1, 0). Since `grid[0][0] + 0 = 2 >= 1`, cell (0,0) can reach (1,0)
- Update: `dist[1][0] = 1 + 1 = 2`
- Add to heaps: `row[1].push((2, 0))` and `col[0].push((2, 1))`

**Processing (1,1):**
- Check row[1] heap: Contains (2, 0). Since `grid[1][0] + 0 = 1 + 0 = 1 >= 1`, cell (1,0) can reach (1,1)
- Check col[1] heap: Contains (2, 0). Since `grid[0][1] + 0 = 1 + 0 = 1 >= 1`, cell (0,1) can reach (1,1)
- Both give distance 3, so `dist[1][1] = 3`
- Add to heaps accordingly

**Processing (1,2):**
- Check row[1] heap: Best valid source gives distance 3
- Check col[2] heap: Contains (2, 0). Since `grid[0][2] + 0 = 2 >= 1`, gives distance 3
- Update: `dist[1][2] = 3`

**Processing (2,0):**
- Check col[0] heap: Multiple entries, but (1, 0) with `grid[0][0] + 0 = 2 >= 2` can reach (2,0)
- Update: `dist[2][0] = 2`

**Processing (2,1):**
- Check row[2] heap: Contains (2, 0). Since `grid[2][0] + 0 = 1 >= 1`, gives distance 3
- Check col[1] heap: Best valid source also gives distance 3
- Update: `dist[2][1] = 3`

**Processing (2,2):**
- Check row[2] heap: Best valid source from (2,0) or (2,1)
- Check col[2] heap: Best valid source from (0,2) or (1,2)
- Minimum distance found: `dist[2][2] = 3`

**Result:** The minimum number of cells to visit is 3. One optimal path is (0,0) → (0,2) → (2,2).

The key insight is how the heaps efficiently track which cells can reach the current position, avoiding the need to check all previous cells in the row/column. The heap cleanup step removes cells that are too far away to make the jump, ensuring we only consider valid sources.

## Python Solution

```python
from typing import List
from heapq import heappush, heappop

class Solution:
    def minimumVisitedCells(self, grid: List[List[int]]) -> int:
        # Get grid dimensions
        rows, cols = len(grid), len(grid[0])
      
        # Initialize distance matrix with -1 (unreachable)
        distances = [[-1] * cols for _ in range(rows)]
        # Starting cell has distance 1
        distances[0][0] = 1
      
        # Priority queues for each row and column
        # Each heap stores tuples of (distance, position)
        row_heaps = [[] for _ in range(rows)]
        col_heaps = [[] for _ in range(cols)]
      
        # Process each cell in row-major order
        for row in range(rows):
            for col in range(cols):
                # Clean up row heap: remove cells that can't reach current position
                # A cell at position (row, prev_col) can reach up to prev_col + grid[row][prev_col]
                while row_heaps[row] and grid[row][row_heaps[row][0][1]] + row_heaps[row][0][1] < col:
                    heappop(row_heaps[row])
              
                # Update distance from leftward cells in same row
                if row_heaps[row]:
                    min_distance_from_row = row_heaps[row][0][0] + 1
                    if distances[row][col] == -1 or distances[row][col] > min_distance_from_row:
                        distances[row][col] = min_distance_from_row
              
                # Clean up column heap: remove cells that can't reach current position
                # A cell at position (prev_row, col) can reach up to prev_row + grid[prev_row][col]
                while col_heaps[col] and grid[col_heaps[col][0][1]][col] + col_heaps[col][0][1] < row:
                    heappop(col_heaps[col])
              
                # Update distance from upward cells in same column
                if col_heaps[col]:
                    min_distance_from_col = col_heaps[col][0][0] + 1
                    if distances[row][col] == -1 or distances[row][col] > min_distance_from_col:
                        distances[row][col] = min_distance_from_col
              
                # If current cell is reachable, add it to corresponding heaps
                # so it can be used to reach other cells
                if distances[row][col] != -1:
                    heappush(row_heaps[row], (distances[row][col], col))
                    heappush(col_heaps[col], (distances[row][col], row))
      
        # Return the distance to bottom-right cell
        return distances[-1][-1]
```

## Time and Space Complexity

**Time Complexity:** `O(m × n × log(m × n))`

The algorithm iterates through each cell of the `m × n` grid exactly once. For each cell `(i, j)`:
- We perform while loops to clean up outdated entries from the heaps, but each element is removed at most once across all iterations
- We perform at most 2 heap push operations (one for `row[i]` and one for `col[j]`)
- We perform at most 2 heap pop operations during the cleanup phase

Since each cell can be pushed to heaps at most twice (once in its row heap and once in its column heap), the total number of heap operations is `O(m × n)`. Each heap operation (push/pop) takes `O(log k)` time where `k` is the heap size. In the worst case, a single row or column heap could contain up to `O(m × n)` elements (though more typically it would be `O(n)` for row heaps and `O(m)` for column heaps).

Therefore, the overall time complexity is `O(m × n × log(m × n))`.

**Space Complexity:** `O(m × n)`

The space usage includes:
- `dist` array: `O(m × n)` to store distances for each cell
- `row` array: `m` heaps that collectively can store up to `O(m × n)` elements in total
- `col` array: `n` heaps that collectively can store up to `O(m × n)` elements in total

While the heaps could theoretically store duplicate entries for cells, the total space across all heaps is bounded by `O(m × n)` since each cell contributes at most a constant number of entries.

Therefore, the overall space complexity is `O(m × n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Heap Cleanup Condition**

One of the most common mistakes is getting the cleanup condition wrong when removing expired entries from the heaps. The condition needs to check whether a cell can still reach the current position.

**Incorrect Implementation:**
```python
# Wrong: Using <= instead of <
while row_heaps[row] and grid[row][row_heaps[row][0][1]] + row_heaps[row][0][1] <= col:
    heappop(row_heaps[row])
```

**Why it's wrong:** If a cell at position `j'` has `grid[row][j'] + j' = col`, it means the cell can exactly reach position `col` (since we can jump to any position in range `(j', grid[row][j'] + j']`). Using `<=` would incorrectly remove this valid cell.

**Correct Implementation:**
```python
# Correct: A cell at j' can reach col if j' < col <= grid[row][j'] + j'
while row_heaps[row] and grid[row][row_heaps[row][0][1]] + row_heaps[row][0][1] < col:
    heappop(row_heaps[row])
```

### 2. **Forgetting to Initialize Starting Cell Distance**

Another pitfall is forgetting to set `distances[0][0] = 1` or incorrectly setting it to 0.

**Incorrect:**
```python
distances = [[-1] * cols for _ in range(rows)]
# Forgot to set distances[0][0] = 1
```

**Why it's wrong:** The problem counts the number of cells visited, including the starting cell. Without initializing the starting cell's distance, the algorithm won't propagate any distances, and all cells will remain unreachable.

### 3. **Heap Entry Order Confusion**

Mixing up what to store in the heap entries can lead to incorrect results.

**Incorrect:**
```python
# Wrong: Storing (position, distance) instead of (distance, position)
heappush(row_heaps[row], (col, distances[row][col]))
heappush(col_heaps[col], (row, distances[row][col]))
```

**Why it's wrong:** Min-heaps in Python compare tuples element by element. We want to prioritize by minimum distance first, so distance must be the first element. Putting position first would prioritize by position instead of distance.

**Correct Implementation:**
```python
# Correct: (distance, position) ensures we get minimum distance first
heappush(row_heaps[row], (distances[row][col], col))
heappush(col_heaps[col], (distances[row][col], row))
```

### 4. **Not Checking if Cell is Reachable Before Adding to Heaps**

Adding unreachable cells (with distance -1) to the heaps will cause incorrect propagation.

**Incorrect:**
```python
# Wrong: Adding cell to heaps without checking if it's reachable
heappush(row_heaps[row], (distances[row][col], col))
heappush(col_heaps[col], (distances[row][col], row))
```

**Correct Implementation:**
```python
# Only add reachable cells to the heaps
if distances[row][col] != -1:
    heappush(row_heaps[row], (distances[row][col], col))
    heappush(col_heaps[col], (distances[row][col], row))
```

