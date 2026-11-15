# 1368. Minimum Cost to Make at Least One Valid Path in a Grid

> Source: https://algo.monster/liteproblems/1368

## Problem Statement

You are given an `m x n` grid where each cell contains a directional sign (value 1-4) that tells you which cell to move to next:
- `1`: Move right to `grid[i][j+1]`
- `2`: Move left to `grid[i][j-1]`  
- `3`: Move down to `grid[i+1][j]`
- `4`: Move up to `grid[i-1][j]`

Starting from the top-left cell `(0, 0)`, you need to find a path that reaches the bottom-right cell `(m-1, n-1)` by following the directional signs. Some signs may point outside the grid boundaries.

You can change the direction sign of any cell with a cost of 1, but each cell's sign can only be modified once. The goal is to find the minimum cost (minimum number of sign changes) needed to create at least one valid path from start to finish.

## Intuition

The key insight is to reframe this grid navigation problem as a [graph](/problems/graph_intro) [shortest path](/problems/shortest_path_unweight) problem. Each cell in the grid is a node, and we can move to any of its 4 neighbors (up, down, left, right). The twist is that moving in the direction the cell already points to is "free" (cost 0), while moving in any other direction requires changing the sign (cost 1).

This creates a special type of weighted [graph](/problems/graph_intro) where all edge weights are either 0 or 1. When we move from cell `(i,j)` to an adjacent cell:
- If we move in the direction `grid[i][j]` already points to → cost = 0
- If we move in any other valid direction → cost = 1 (we need to change the sign)

For finding shortest paths in 0-1 weighted graphs, we can use a more efficient variant of BFS instead of Dijkstra's algorithm. The trick is to use a double-ended queue (deque) where:
- When we traverse an edge with weight 0, we add the new node to the **front** of the queue
- When we traverse an edge with weight 1, we add the new node to the **back** of the queue

This ordering ensures that we always process nodes in non-decreasing order of their distance from the source. Nodes reachable with fewer sign changes will be processed before those requiring more changes.

Why does this work? Because when we add a 0-cost neighbor to the front, it has the same total cost as the current node, so it should be processed with the same priority. When we add a 1-cost neighbor to the back, it has a higher cost and should be processed after all nodes with the current cost level.

By the time we reach the destination cell `(m-1, n-1)`, we're guaranteed to have found the minimum number of sign changes needed to create a valid path.

**Learn more about [Breadth-First Search](/problems/bfs_intro), [Graph](/problems/graph_intro), [Shortest Path](/problems/shortest_path_unweight) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation uses a 0-1 BFS approach with a double-ended queue to find the minimum cost path. Let's walk through the key components:

**1. Direction Mapping**
```python
dirs = [[0, 0], [0, 1], [0, -1], [1, 0], [-1, 0]]
```
The `dirs` array maps the grid values (1-4) to their corresponding movement directions. Index 0 is unused (placeholder), while indices 1-4 represent right, left, down, and up movements respectively.

**2. Initialization**
```python
q = deque([(0, 0, 0)])  # (row, col, cost)
vis = set()
```
We start with the source cell `(0, 0)` with cost 0 in our deque. The `vis` set tracks visited cells to avoid reprocessing.

**3. BFS with Double-ended Queue**
The main loop processes cells in order of their cost:
```python
while q:
    i, j, d = q.popleft()
    if (i, j) in vis:
        continue
    vis.add((i, j))
```
We pop from the left (front) of the queue and mark the cell as visited. This ensures we process cells with lower costs first.

**4. Target Check**
```python
if i == m - 1 and j == n - 1:
    return d
```
If we reach the destination, we immediately return the cost, as it's guaranteed to be minimal due to our processing order.

**5. Exploring Neighbors**
For each cell, we try all 4 possible directions:
```python
for k in range(1, 5):
    x, y = i + dirs[k][0], j + dirs[k][1]
    if 0 <= x < m and 0 <= y < n:
        if grid[i][j] == k:
            q.appendleft((x, y, d))
        else:
            q.append((x, y, d + 1))
```

The critical logic here:
- If direction `k` matches `grid[i][j]` (the cell's current direction), we follow it with no additional cost → add to **front** with `appendleft((x, y, d))`
- Otherwise, we need to change the sign (cost = 1) → add to **back** with `append((x, y, d + 1))`

This ordering maintains the invariant that cells are processed in non-decreasing order of cost, ensuring optimality when we first reach the destination.

The algorithm effectively explores all reachable cells, prioritizing paths that require fewer sign changes, until it finds the minimum-cost path to the destination.

## Example Walkthrough

Let's walk through a small 3x3 grid example to illustrate the 0-1 BFS solution approach:

```
Grid:
[1, 3, 3]
[3, 2, 1]
[3, 1, 1]

Where: 1=right, 2=left, 3=down, 4=up
```

**Goal**: Find minimum sign changes to create a path from (0,0) to (2,2).

**Step-by-step execution:**

**Initial State:**
- Deque: `[(0,0,0)]` - start at top-left with cost 0
- Visited: `{}`

**Iteration 1:** Process (0,0) with cost 0
- Mark (0,0) as visited
- Cell value is 1 (right), so moving right to (0,1) costs 0
- Try all 4 directions from (0,0):
  - Right (0,1): Matches grid value 1 → cost 0 → add to **front**
  - Down (1,0): Doesn't match (need 3, have 1) → cost 1 → add to **back**
- Deque: `[(0,1,0), (1,0,1)]`

**Iteration 2:** Process (0,1) with cost 0
- Mark (0,1) as visited  
- Cell value is 3 (down), so moving down to (1,1) costs 0
- Try all 4 directions:
  - Right (0,2): Doesn't match → cost 1 → add to **back**
  - Down (1,1): Matches grid value 3 → cost 0 → add to **front**
  - Left (0,0): Already visited, skip
- Deque: `[(1,1,0), (1,0,1), (0,2,1)]`

**Iteration 3:** Process (1,1) with cost 0
- Mark (1,1) as visited
- Cell value is 2 (left), so moving left to (1,0) costs 0
- Try all 4 directions:
  - Right (1,2): Doesn't match → cost 1 → add to **back**
  - Down (2,1): Doesn't match → cost 1 → add to **back**
  - Left (1,0): Matches grid value 2 → cost 0 → add to **front**
  - Up (0,1): Already visited, skip
- Deque: `[(1,0,0), (1,0,1), (0,2,1), (1,2,1), (2,1,1)]`

**Iteration 4:** Process (1,0) with cost 0
- Mark (1,0) as visited
- Cell value is 3 (down), so moving down to (2,0) costs 0
- Try all 4 directions:
  - Right (1,1): Already visited, skip
  - Down (2,0): Matches grid value 3 → cost 0 → add to **front**
  - Up (0,0): Already visited, skip
- Deque: `[(2,0,0), (1,0,1), (0,2,1), (1,2,1), (2,1,1)]`

Note: (1,0,1) in deque is duplicate but with higher cost, will be skipped when processed.

**Iteration 5:** Process (2,0) with cost 0
- Mark (2,0) as visited
- Cell value is 3 (down), but can't go down (out of bounds)
- Try all 4 directions:
  - Right (2,1): Doesn't match → cost 1 → add to **back**
  - Up (1,0): Already visited, skip
- Deque: `[(1,0,1), (0,2,1), (1,2,1), (2,1,1), (2,1,1)]`

Continue processing... Eventually:

**Key Iteration:** Process (2,1) with cost 1
- Cell value is 1 (right), so moving right to (2,2) costs 0 relative to current
- Total cost to (2,2) = 1 + 0 = 1
- Deque gets: `[(2,2,1), ...]`

**Final:** Process (2,2) with cost 1
- **Reached destination!** Return cost = 1

**Result**: Minimum 1 sign change needed. One optimal path would be:
- (0,0) → (0,1) [free, follows sign]
- (0,1) → (1,1) [free, follows sign]  
- (1,1) → (1,2) [cost 1, change sign from 2 to 1]
- (1,2) → (2,2) [free, follows sign]

The 0-1 BFS ensures we find this minimum cost by always processing lower-cost paths first through the deque's front/back structure.

## Python Solution

```python
from collections import deque
from typing import List

class Solution:
    def minCost(self, grid: List[List[int]]) -> int:
        # Get grid dimensions
        rows, cols = len(grid), len(grid[0])
      
        # Direction vectors: placeholder at index 0, then right, left, down, up
        # Corresponds to grid values 1, 2, 3, 4
        directions = [[0, 0], [0, 1], [0, -1], [1, 0], [-1, 0]]
      
        # Initialize BFS queue with starting position (row, col, cost)
        queue = deque([(0, 0, 0)])
      
        # Track visited cells to avoid revisiting
        visited = set()
      
        # Process queue using 0-1 BFS
        while queue:
            current_row, current_col, current_cost = queue.popleft()
          
            # Skip if already visited
            if (current_row, current_col) in visited:
                continue
          
            # Mark current cell as visited
            visited.add((current_row, current_col))
          
            # Check if reached destination
            if current_row == rows - 1 and current_col == cols - 1:
                return current_cost
          
            # Try all four possible directions
            for direction_index in range(1, 5):
                next_row = current_row + directions[direction_index][0]
                next_col = current_col + directions[direction_index][1]
              
                # Check if next position is within grid bounds
                if 0 <= next_row < rows and 0 <= next_col < cols:
                    # If moving in the direction indicated by current cell (no cost)
                    if grid[current_row][current_col] == direction_index:
                        # Add to front of queue (priority for 0-cost moves)
                        queue.appendleft((next_row, next_col, current_cost))
                    else:
                        # Add to back of queue (cost increases by 1)
                        queue.append((next_row, next_col, current_cost + 1))
      
        # Should not reach here for valid inputs
        return -1
```

## Time and Space Complexity

**Time Complexity:** `O(m * n)`

The algorithm uses a 0-1 BFS approach with a deque. Each cell `(i, j)` in the grid is processed at most once due to the visited set `vis`. For each cell, we explore at most 4 directions (constant time). Since we have `m * n` cells total and each cell is visited once, the time complexity is `O(m * n)`.

**Space Complexity:** `O(m * n)`

The space complexity consists of:
- The deque `q` which in the worst case can contain all cells: `O(m * n)`
- The visited set `vis` which can store up to all cells: `O(m * n)`
- The `dirs` array which is constant space: `O(1)`

Therefore, the overall space complexity is `O(m * n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Regular BFS Instead of 0-1 BFS**
A common mistake is attempting to solve this with regular BFS using a standard queue, which doesn't guarantee finding the minimum cost path.

**Incorrect approach:**
```python
# Regular BFS - WRONG!
queue = deque([(0, 0, 0)])
while queue:
    row, col, cost = queue.popleft()
    # ... processing ...
    for direction in range(1, 5):
        # Always append to the back regardless of cost
        queue.append((new_row, new_col, new_cost))
```

**Why it fails:** Regular BFS processes nodes in the order they're added, not by cost. You might reach the destination with a suboptimal cost before finding the minimum-cost path.

**Solution:** Use 0-1 BFS with a deque, adding 0-cost moves to the front and 1-cost moves to the back.

### 2. **Visiting Cells Multiple Times**
Another pitfall is allowing cells to be processed multiple times, leading to incorrect results or infinite loops.

**Incorrect approach:**
```python
visited = set()
while queue:
    row, col, cost = queue.popleft()
    visited.add((row, col))  # Adding to visited AFTER popping
  
    # This allows the same cell to be added to queue multiple times
    for direction in range(1, 5):
        if (new_row, new_col) not in visited:  # Check happens too late
            # Add to queue
```

**Why it fails:** The same cell can be added to the queue multiple times with different costs before it's marked as visited, leading to redundant processing and potentially wrong results.

**Solution:** Check visited status immediately after popping from the queue and skip if already visited:
```python
while queue:
    row, col, cost = queue.popleft()
    if (row, col) in visited:
        continue  # Skip already processed cells
    visited.add((row, col))
```

### 3. **Incorrect Direction Mapping**
Misaligning the direction array indices with grid values is a subtle but critical error.

**Incorrect approach:**
```python
# Wrong indexing or order
directions = [[0, 1], [0, -1], [1, 0], [-1, 0]]  # Missing placeholder at index 0
# Or
directions = [[0, 0], [1, 0], [-1, 0], [0, 1], [0, -1]]  # Wrong order
```

**Why it fails:** Grid values 1-4 won't map to their intended directions, causing the algorithm to explore wrong paths.

**Solution:** Ensure the directions array has a placeholder at index 0 and correct mappings:
```python
directions = [[0, 0], [0, 1], [0, -1], [1, 0], [-1, 0]]
# Index 0: placeholder
# Index 1: right (grid value 1)
# Index 2: left (grid value 2)
# Index 3: down (grid value 3)
# Index 4: up (grid value 4)
```

### 4. **Not Handling Edge Cases**
Forgetting to handle grids where the destination is unreachable or the grid has only one cell.

**Solution:** The 0-1 BFS naturally handles these cases, but ensure your implementation:
- Returns 0 when the grid is 1x1 (start equals destination)
- The algorithm will explore all reachable cells, so unreachable destinations won't cause errors
- Consider adding a return statement at the end (though it shouldn't be reached in valid inputs)

