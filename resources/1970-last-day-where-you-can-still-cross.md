# 1970. Last Day Where You Can Still Cross

> Source: https://algo.monster/liteproblems/1970

## Problem Statement

You have a grid/matrix with `row` rows and `col` columns. Initially (day 0), the entire grid is land (represented by 0). Each day, one cell gets flooded with water (changes from 0 to 1).

You're given an array `cells` where `cells[i] = [ri, ci]` tells you that on day `i`, the cell at row `ri` and column `ci` will be flooded with water. Note that all coordinates are 1-based (rows and columns start from 1, not 0).

Your task is to find the **last day** when you can still walk from any cell in the top row to any cell in the bottom row by only stepping on land cells (cells with value 0). You can move in four directions: up, down, left, and right.

For example:
- If you have a 2×2 grid and `cells = [[1,1], [2,1], [1,2], [2,2]]`
- Day 0: All cells are land
- Day 1: Cell (1,1) becomes water
- Day 2: Cell (2,1) becomes water
- Day 3: Cell (1,2) becomes water
- Day 4: Cell (2,2) becomes water

You need to determine on which day you can no longer traverse from the top row to the bottom row while staying on land.

## Intuition

The key insight is recognizing the **monotonic property** of this problem. If you can walk from top to bottom on day `k`, you can definitely walk on any earlier day because fewer cells would be flooded. Conversely, if you cannot walk on day `k`, you cannot walk on any later day either because more cells will be flooded.

This monotonic behavior immediately suggests **[binary search](/problems/binary_search_intro)**. Instead of checking every single day sequentially (which would be inefficient), we can use binary search to find the exact turning point - the last day when crossing is still possible.

Think of it like this: imagine you're watching a time-lapse video of an island slowly sinking into the ocean. At first, there are many paths from north to south. As more land floods each day, paths start disappearing. Eventually, there comes a specific day when the last possible path gets cut off. We want to find that exact day.

For any given day `mid`, we need to check if a path exists. We do this by:
1. Flooding all cells that should be water by day `mid` (the first `mid` cells from the `cells` array)
2. Checking if we can still traverse from top to bottom

The traversal check is a classic **graph connectivity** problem. We can use either DFS or BFS to explore from any starting cell in the top row and see if we can reach any cell in the bottom row. The solution uses BFS, but DFS would work equally well.

The [binary search](/problems/binary_search_intro) bounds are straightforward:
- Minimum: day 1 (we know day 0 always works since everything is land)
- Maximum: total number of cells (worst case, all cells get flooded)

By combining [binary search](/problems/binary_search_intro) with path-finding (BFS/DFS), we efficiently find the answer in `O(row × col × log(row × col))` time instead of checking every single day sequentially.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro), [Union Find](/problems/dsu_intro) and [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution implements **[binary search](/problems/binary_search_intro) combined with BFS** to find the last day we can cross from top to bottom.

**Binary Search Template Application:**

We need to find the **last day** where crossing is possible. To use the standard "find first true" template, we reframe the problem: find the **first day** where crossing becomes **impossible**, then return `first_true_index - 1`.

The feasible function is defined as: `feasible(day) = !canCross(day)` (returns true when crossing is NOT possible).

**Algorithm Steps:**

1. **Initialize search bounds:**
   - `left = 1`, `right = row * col`
   - `first_true_index = -1` (tracks the first day where crossing becomes impossible)

2. **Binary search loop (`while left <= right`):**
   - Calculate `mid = (left + right) // 2`
   - If `!canCross(mid)` is true (crossing impossible on day `mid`):
     - Save `first_true_index = mid`
     - Search for an earlier impossible day: `right = mid - 1`
   - If `!canCross(mid)` is false (crossing still possible):
     - Need a later day: `left = mid + 1`

3. **Return the last crossable day:** `first_true_index - 1`

**The canCross Helper Function (BFS):**

For each candidate day `k`, check if crossing is possible:

1. **Build the grid state:** Create a 2D grid, mark the first `k` cells as water (1).

2. **BFS from top row:** Initialize queue with all land cells from row 0.

3. **BFS exploration:** For each cell, check if we've reached the bottom row. Explore all 4 directions, adding unvisited land cells to the queue.

4. **Return result:** `True` if we reach bottom row, `False` otherwise.

**Time Complexity:** `O(row × col × log(row × col))`
- Binary search iterations: `O(log(row × col))`
- Each canCross check: `O(row × col)` for BFS traversal

**Space Complexity:** `O(row × col)` for the grid and BFS queue

## Example Walkthrough

Let's walk through a small example with a 3×3 grid and `cells = [[1,2], [2,2], [3,2], [1,1], [3,3]]`.

**Initial Setup:**
- Grid dimensions: 3 rows × 3 columns
- `left = 1`, `right = 5`, `first_true_index = -1`
- We're finding the first day where crossing becomes impossible, then subtracting 1.
- `feasible(day) = !canCross(day)` (true when crossing is impossible)

**Binary Search Iteration 1:**
- `mid = (1 + 5) // 2 = 3`
- Check `!canCross(3)`: After flooding cells [1,2], [2,2], [3,2]
- Grid state:
  ```
  0 1 0
  0 1 0
  0 1 0
  ```
- BFS from (0,0): (0,0) → (1,0) → (2,0) - reached bottom row!
- `canCross(3) = true`, so `!canCross(3) = false`
- Not feasible (can still cross): `left = 3 + 1 = 4`

**Binary Search Iteration 2:**
- `mid = (4 + 5) // 2 = 4`
- Check `!canCross(4)`: After flooding cells [1,2], [2,2], [3,2], [1,1]
- Grid state:
  ```
  1 1 0
  0 1 0
  0 1 0
  ```
- BFS from (0,2): (0,2) → blocked by water, no path to bottom
- `canCross(4) = false`, so `!canCross(4) = true`
- Feasible (crossing impossible): `first_true_index = 4`, `right = 4 - 1 = 3`

**Binary Search Iteration 3:**
- `left = 4 > right = 3`, loop ends

**Final Result:**
- `first_true_index = 4` (first day crossing is impossible)
- Return `first_true_index - 1 = 3` (last day crossing is possible)

The last day we can cross from top to bottom is day 3.

## Python Solution

```python
class Solution:
    def latestDayToCross(self, row: int, col: int, cells: List[List[int]]) -> int:
        def can_cross(day: int) -> bool:
            """Check if it's possible to cross from top to bottom on the given day."""
            grid = [[0] * col for _ in range(row)]

            for i in range(day):
                r, c = cells[i]
                grid[r - 1][c - 1] = 1

            queue = []
            for j in range(col):
                if grid[0][j] == 0:
                    queue.append((0, j))
                    grid[0][j] = 1

            idx = 0
            while idx < len(queue):
                x, y = queue[idx]
                idx += 1

                if x == row - 1:
                    return True

                for dx, dy in [(-1, 0), (1, 0), (0, -1), (0, 1)]:
                    next_x, next_y = x + dx, y + dy
                    if 0 <= next_x < row and 0 <= next_y < col and grid[next_x][next_y] == 0:
                        queue.append((next_x, next_y))
                        grid[next_x][next_y] = 1

            return False

        def feasible(day: int) -> bool:
            """Returns true when crossing is NOT possible (inverted condition)."""
            return not can_cross(day)

        # Binary search for first day where crossing becomes impossible
        left, right = 1, len(cells)
        first_true_index = -1

        while left <= right:
            mid = (left + right) // 2
            if feasible(mid):
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1

        # Return the last day where crossing was still possible
        return first_true_index - 1
```

## Time and Space Complexity

**Time Complexity:** `O(m × n × log(m × n))`

The algorithm uses binary search on the range `[1, m × n]`, which requires `O(log(m × n))` iterations. For each binary search iteration, the `check` function is called, which performs a BFS traversal on an `m × n` grid. The BFS visits each cell at most once, taking `O(m × n)` time. Therefore, the overall time complexity is `O(m × n × log(m × n))`.

**Space Complexity:** `O(m × n)`

The space complexity is dominated by:
- The grid `g` created in the `check` function: `O(m × n)`
- The queue `q` used for BFS, which in the worst case can contain all cells: `O(m × n)`
- The `cells` array is given as input and not counted in auxiliary space

Therefore, the total space complexity is `O(m × n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template Variant**
A common mistake is using the `while left < right` with `left = mid` variant. The standard template uses `while left <= right` with `right = mid - 1`.

**Incorrect Implementation:**
```python
while left < right:
    mid = (left + right + 1) // 2
    if can_cross(mid):
        left = mid  # Wrong pattern
    else:
        right = mid - 1
return left
```

**Correct Implementation (using template with inverted condition):**
```python
left, right = 1, len(cells)
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if not can_cross(mid):  # feasible = cannot cross
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

return first_true_index - 1
```

### 2. **Coordinate System Confusion (1-based vs 0-based)**
The problem provides 1-based coordinates in the `cells` array, but arrays are 0-based. Forgetting to convert leads to IndexError or wrong cells being flooded.

**Incorrect:**
```python
grid[r][c] = 1  # Direct use of 1-based coordinates
```

**Correct:**
```python
grid[r - 1][c - 1] = 1  # Convert to 0-based
```

### 3. **Not Marking Cells as Visited Immediately**
Marking cells as visited only after dequeuing (instead of when enqueuing) can cause the same cell to be added multiple times, leading to TLE.

**Incorrect:**
```python
while queue:
    x, y = queue[idx]
    grid[x][y] = 1  # Mark visited after dequeuing - TOO LATE!
    for dx, dy in directions:
        if valid and grid[nx][ny] == 0:
            queue.append((nx, ny))
```

**Correct:**
```python
for dx, dy in directions:
    if valid and grid[nx][ny] == 0:
        queue.append((nx, ny))
        grid[nx][ny] = 1  # Mark visited immediately when enqueuing
```

### 4. **Forgetting to Invert the Condition for Template**
When using the "find first true" template for a "find last true" problem, you must invert the feasible condition.

**Why:**
The template finds the first index where `feasible(x)` returns true. For "find last day where crossing is possible", we define `feasible(day) = !canCross(day)` (first day where crossing is impossible), then return `first_true_index - 1`.

### 5. **Binary Search Bounds Initialization**
Starting with `left = 0` instead of `left = 1` wastes an iteration since day 0 means no cells are flooded (always crossable).

**Optimal:**
```python
left, right = 1, len(cells)
```

