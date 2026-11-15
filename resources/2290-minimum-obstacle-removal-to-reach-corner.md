# 2290. Minimum Obstacle Removal to Reach Corner

> Source: https://algo.monster/liteproblems/2290

## Problem Statement

You have a 2D grid of size `m x n` where each cell contains either:
- `0`: an empty cell you can freely move through
- `1`: an obstacle that blocks your path (but can be removed)

Starting from the top-left corner `(0, 0)`, you need to reach the bottom-right corner `(m-1, n-1)`. You can move in four directions: up, down, left, or right, but only to empty cells.

The task is to find the **minimum number of obstacles you need to remove** to create a valid path from start to finish.

For example, if you have a grid where some cells are blocked by obstacles (1s), you might need to remove some of these obstacles to make a path possible. The goal is to minimize how many obstacles you remove.

Key points:
- You start at position `(0, 0)`
- You want to reach position `(m-1, n-1)`
- You can only move to adjacent cells (up, down, left, right)
- You can only move through empty cells (0s) unless you remove an obstacle
- Each obstacle removal has a cost of 1
- Find the minimum total cost (minimum obstacles to remove)

This is essentially finding the path with the least "cost" where moving through an empty cell costs 0 and moving through an obstacle costs 1.

## Intuition

The key insight is recognizing that this problem is about finding the "cheapest" path through a grid, where the cost is the number of obstacles we need to remove.

Think of it this way: imagine you're walking through a maze where some paths are clear (empty cells) and others have barriers you can break through (obstacles). Walking through a clear path is free, but breaking through a barrier costs you energy. You want to reach the exit using the least amount of energy.

This naturally translates to a [shortest path](/problems/shortest_path_unweight) problem with edge weights:
- Moving to an empty cell (0) → cost = 0
- Moving to an obstacle (1) → cost = 1

Now, why use BFS with a deque instead of regular BFS or Dijkstra's?

Regular BFS assumes all edges have the same weight, which isn't true here. Dijkstra's algorithm works but is overkill since we only have two possible weights (0 and 1).

The clever trick is using a **double-ended queue (deque)** to maintain the BFS invariant that nodes are processed in order of their distance from the source. Here's the intuition:
- When we move to an empty cell (cost 0), the total cost stays the same, so we add it to the **front** of the queue
- When we move to an obstacle (cost 1), the total cost increases by 1, so we add it to the **back** of the queue

This ensures that at any point, all nodes in the queue with cost `k` are processed before any node with cost `k+1`. It's like having two priority levels: "free moves" get processed immediately, while "costly moves" wait their turn.

This 0-1 BFS technique guarantees we find the minimum cost path because we always explore all paths with fewer obstacles before considering paths with more obstacles.

**Learn more about [Breadth-First Search](/problems/bfs_intro), [Graph](/problems/graph_intro), [Shortest Path](/problems/shortest_path_unweight) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution implements a 0-1 BFS using a double-ended queue, which is optimal for graphs with edge weights of only 0 and 1.

**Data Structures Used:**
- `deque`: A double-ended queue to implement 0-1 BFS
- `vis` (set): To track visited cells and avoid revisiting them
- Each queue element: tuple `(i, j, k)` where `(i, j)` is the position and `k` is the number of obstacles removed so far

**Algorithm Steps:**

1. **Initialization**: Start with the source cell `(0, 0)` with 0 obstacles removed. Add `(0, 0, 0)` to the deque.

2. **Direction vectors**: Use `dirs = (-1, 0, 1, 0, -1)` with `pairwise` to generate the four directions. Each pair `(dirs[i], dirs[i+1])` gives us:
   - `(-1, 0)`: up
   - `(0, 1)`: right  
   - `(1, 0)`: down
   - `(0, -1)`: left

3. **BFS Loop**: Continue until we reach the destination:
   - Pop from the **front** of the deque (always the minimum cost node)
   - If we've reached `(m-1, n-1)`, return the cost `k`
   - If already visited, skip this cell
   - Mark the cell as visited

4. **Exploring Neighbors**: For each valid neighbor `(x, y)`:
   - Check boundaries: `0 <= x < m` and `0 <= y < n`
   - If `grid[x][y] == 0` (empty cell):
     - Add `(x, y, k)` to the **front** of deque (same cost)
   - If `grid[x][y] == 1` (obstacle):
     - Add `(x, y, k+1)` to the **back** of deque (cost increases by 1)

**Why This Works:**

The 0-1 BFS maintains the invariant that all nodes with distance `d` are processed before any node with distance `d+1`. By using `appendleft()` for 0-weight edges and `append()` for 1-weight edges, we ensure that:
- Cells reachable with `k` obstacles are always explored before cells requiring `k+1` obstacles
- The first time we reach the destination, we've found the minimum cost path

**Time Complexity:** `O(m × n)` - Each cell is visited at most once
**Space Complexity:** `O(m × n)` - For the visited set and queue in the worst case

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this 3×3 grid:
```
[0, 1, 1]
[1, 1, 0]
[1, 0, 0]
```

We need to find the minimum obstacles to remove to get from (0,0) to (2,2).

**Step-by-step execution:**

**Initial State:**
- Deque: `[(0, 0, 0)]` - start at top-left with 0 obstacles removed
- Visited: empty set

**Iteration 1:** Pop `(0, 0, 0)` from front
- Current position: (0, 0), obstacles removed: 0
- Mark (0, 0) as visited
- Check neighbors:
  - Right (0, 1): grid[0][1] = 1 (obstacle) → Add `(0, 1, 1)` to **back** of deque
  - Down (1, 0): grid[1][0] = 1 (obstacle) → Add `(1, 0, 1)` to **back** of deque
- Deque: `[(0, 1, 1), (1, 0, 1)]`

**Iteration 2:** Pop `(0, 1, 1)` from front
- Current position: (0, 1), obstacles removed: 1
- Mark (0, 1) as visited
- Check neighbors:
  - Right (0, 2): grid[0][2] = 1 (obstacle) → Add `(0, 2, 2)` to **back** of deque
  - Down (1, 1): grid[1][1] = 1 (obstacle) → Add `(1, 1, 2)` to **back** of deque
- Deque: `[(1, 0, 1), (0, 2, 2), (1, 1, 2)]`

**Iteration 3:** Pop `(1, 0, 1)` from front
- Current position: (1, 0), obstacles removed: 1
- Mark (1, 0) as visited
- Check neighbors:
  - Right (1, 1): grid[1][1] = 1 (obstacle) → Add `(1, 1, 2)` to **back** (already in queue)
  - Down (2, 0): grid[2][0] = 1 (obstacle) → Add `(2, 0, 2)` to **back** of deque
- Deque: `[(0, 2, 2), (1, 1, 2), (2, 0, 2)]`

**Iteration 4:** Pop `(0, 2, 2)` from front
- Current position: (0, 2), obstacles removed: 2
- Mark (0, 2) as visited
- Check neighbors:
  - Down (1, 2): grid[1][2] = 0 (empty) → Add `(1, 2, 2)` to **front** of deque (same cost!)
- Deque: `[(1, 2, 2), (1, 1, 2), (2, 0, 2)]`

**Iteration 5:** Pop `(1, 2, 2)` from front
- Current position: (1, 2), obstacles removed: 2
- Mark (1, 2) as visited
- Check neighbors:
  - Down (2, 2): grid[2][2] = 0 (empty) → Add `(2, 2, 2)` to **front** of deque
- Deque: `[(2, 2, 2), (1, 1, 2), (2, 0, 2)]`

**Iteration 6:** Pop `(2, 2, 2)` from front
- Current position: (2, 2) - **We've reached the destination!**
- Return the cost: **2 obstacles removed**

**Key Observations:**
1. When we moved from (0, 2) to (1, 2), both cells had the same total cost (2), but since (1, 2) was empty, we added it to the **front** of the deque, allowing it to be processed immediately.
2. This ensured we explored the path through empty cells before considering paths that would require removing more obstacles.
3. The path taken was: (0,0) → (0,1) → (0,2) → (1,2) → (2,2), removing 2 obstacles at positions (0,1) and (0,2).

## Python Solution

```python
class Solution:
    def minimumObstacles(self, grid: List[List[int]]) -> int:
        # Get grid dimensions
        rows, cols = len(grid), len(grid[0])
      
        # Initialize deque with starting position (row, col, obstacles_removed)
        queue = deque([(0, 0, 0)])
      
        # Set to track visited cells
        visited = set()
      
        # Direction vectors for moving up, right, down, left
        directions = [(-1, 0), (0, 1), (1, 0), (0, -1)]
      
        # BFS with 0-1 optimization
        while queue:
            current_row, current_col, obstacles_removed = queue.popleft()
          
            # Check if we reached the destination
            if current_row == rows - 1 and current_col == cols - 1:
                return obstacles_removed
          
            # Skip if already visited
            if (current_row, current_col) in visited:
                continue
          
            # Mark current cell as visited
            visited.add((current_row, current_col))
          
            # Explore all four directions
            for delta_row, delta_col in directions:
                next_row = current_row + delta_row
                next_col = current_col + delta_col
              
                # Check if next position is within grid bounds
                if 0 <= next_row < rows and 0 <= next_col < cols:
                    if grid[next_row][next_col] == 0:
                        # Empty cell: add to front (0 cost)
                        queue.appendleft((next_row, next_col, obstacles_removed))
                    else:
                        # Obstacle: add to back (cost + 1)
                        queue.append((next_row, next_col, obstacles_removed + 1))
```

## Time and Space Complexity

**Time Complexity:** `O(m * n)` where `m` is the number of rows and `n` is the number of columns in the grid.

The algorithm uses a modified BFS with a deque (0-1 BFS). Each cell in the grid is visited at most once due to the visited set `vis`. For each cell, we explore at most 4 neighboring cells (constant time). Therefore, the total number of operations is proportional to the number of cells in the grid, which is `m * n`.

**Space Complexity:** `O(m * n)`

The space complexity is dominated by:
- The visited set `vis` which can store up to `m * n` cell coordinates in the worst case: `O(m * n)`
- The deque `q` which in the worst case could contain all cells before processing: `O(m * n)`
- The `dirs` tuple which takes constant space: `O(1)`

Overall space complexity is `O(m * n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Standard BFS Instead of 0-1 BFS**

**Pitfall:** A common mistake is using a standard BFS with a regular queue, treating all moves equally. This leads to incorrect results because standard BFS doesn't account for different edge weights (0 for empty cells, 1 for obstacles).

**Why it fails:** Standard BFS explores all cells at distance `d` before moving to distance `d+1`, but here "distance" means number of steps, not the cost. A path with more steps but fewer obstacles might be better than a shorter path with more obstacles.

**Solution:** Use a deque with 0-1 BFS pattern:
- Add 0-cost moves to the front (`appendleft`)
- Add 1-cost moves to the back (`append`)
- Always pop from the front (`popleft`)

### 2. **Revisiting Cells with Different Costs**

**Pitfall:** Not properly tracking visited cells can lead to:
- Infinite loops if you allow revisiting
- Suboptimal solutions if you mark cells as visited too early

**Example scenario:** You might reach a cell with cost 3, mark it visited, then later find a path to the same cell with cost 2, but skip it because it's already visited.

**Solution:** Check if a cell is visited AFTER popping it from the queue, not before adding it. This ensures that if we reach the same cell multiple times with different costs, we process the one with minimum cost first:

```python
while queue:
    row, col, cost = queue.popleft()
  
    # Check visited AFTER popping, not before adding
    if (row, col) in visited:
        continue
    visited.add((row, col))
```

### 3. **Incorrect Handling of Starting/Ending Cells**

**Pitfall:** Assuming the starting cell `(0,0)` or ending cell `(m-1,n-1)` are always empty (value 0). If either contains an obstacle, you need to account for it.

**Solution:** The algorithm naturally handles this - if the starting cell has an obstacle, the first move will effectively "remove" it. If the ending cell has an obstacle, it will be counted when we reach it.

### 4. **Using Dijkstra's Algorithm with a Priority Queue**

**Pitfall:** While Dijkstra's algorithm would work correctly, it's overkill for this problem and less efficient.

**Why it's suboptimal:** 
- Dijkstra's uses a priority queue with O(log n) operations
- 0-1 BFS uses a deque with O(1) operations
- Since we only have weights 0 and 1, 0-1 BFS is more efficient

**Solution:** Stick with 0-1 BFS for optimal performance when dealing with 0-1 weighted graphs.

