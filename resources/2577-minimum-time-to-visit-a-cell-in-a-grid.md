# 2577. Minimum Time to Visit a Cell In a Grid

> Source: https://algo.monster/liteproblems/2577

## Problem Statement

You are given an `m x n` matrix `grid` consisting of non-negative integers. Each cell `grid[row][col]` represents the **minimum** time required to visit that cell - meaning you can only visit cell `(row, col)` when your current time is greater than or equal to `grid[row][col]`.

You start at the top-left cell `(0, 0)` at time `0`. From any cell, you can move to an adjacent cell in four directions (up, down, left, right), and each move takes exactly `1` second.

Your goal is to find the **minimum time** required to reach the bottom-right cell `(m-1, n-1)` of the matrix. If it's impossible to reach the bottom-right cell, return `-1`.

**Key Points:**
- You begin at position `(0, 0)` at time `0`
- Each move to an adjacent cell takes `1` second
- You can only enter a cell if your current time ≥ the cell's value
- You need to reach position `(m-1, n-1)` in minimum time
- Return `-1` if the destination is unreachable

**Example of the time constraint:** If you're at a cell at time `5` and want to move to an adjacent cell with value `8`, you cannot enter immediately. You would need to wait until time `8` to enter that cell.

## Intuition

The key insight is recognizing that this is a [shortest path](/problems/shortest_path_unweight) problem with a twist - we have time constraints on when we can enter each cell. Let's think through why standard BFS won't work and how we arrive at the solution.

In a typical [shortest path](/problems/shortest_path_unweight) problem on a grid, BFS would work perfectly because each move has the same cost (1 second). However, here we have an additional constraint: we can only enter a cell when our current time is at least `grid[x][y]`.

Consider what happens when we reach a cell at time `t` but need to enter an adjacent cell that requires time `grid[x][y] > t + 1`. We can't just move there immediately - we need to wait. But how do we wait? We can move back and forth between accessible cells to "kill time" until we reach the required timestamp.

This leads to an important observation: if we can move back and forth between two cells, we can extend our waiting time by any even number of seconds (move away and back = 2 seconds). This means:
- If we arrive at time `t` and need time `grid[x][y]`, and the difference `(grid[x][y] - t)` is odd, we arrive at `grid[x][y]`
- If the difference is even, we need one extra second, arriving at `grid[x][y] + 1`

This waiting mechanism only works if we can actually move from the starting position. If both adjacent cells from `(0,0)` require time > 1, we're stuck and should return `-1`.

Since different paths might reach the same cell at different times, and we want the minimum time to reach the destination, we need to track the earliest arrival time at each cell. This naturally leads us to Dijkstra's algorithm - we use a priority queue to always process the cell we can reach earliest, ensuring we find the optimal path.

The algorithm essentially becomes: explore cells in order of their earliest reachable time, calculating the actual arrival time considering both movement cost and waiting time needed to satisfy the cell's time constraint.

**Learn more about [Breadth-First Search](/problems/bfs_intro), [Graph](/problems/graph_intro), [Shortest Path](/problems/shortest_path_unweight) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**Initial Check for Impossibility:**
First, we check if movement is possible at all. If both `grid[0][1] > 1` and `grid[1][0] > 1`, we cannot move from the starting position `(0,0)` since we start at time 0 and need at least time 2 to enter either adjacent cell. In this case, we return `-1`.

**Data Structure Setup:**
- Create a 2D array `dist` of size `m × n` initialized with infinity (`inf`) to track the minimum time to reach each cell
- Set `dist[0][0] = 0` since we start at the top-left corner at time 0
- Initialize a min-heap priority queue `q` with the starting state `(0, 0, 0)` representing `(time, row, col)`

**Dijkstra's Algorithm with Time Adjustment:**
We process cells in order of their earliest reachable time using the priority queue:

1. **Extract minimum:** Pop the cell `(t, i, j)` with the smallest time from the heap
2. **Check destination:** If we've reached `(m-1, n-1)`, return the time `t`
3. **Explore neighbors:** For each of the 4 adjacent cells `(x, y)`:
   - Calculate the base arrival time: `nt = t + 1`
   - **Handle time constraints:** If `nt < grid[x][y]`, we need to wait:
     - Calculate the waiting time needed: `wait = grid[x][y] - nt`
     - If `wait` is even, we arrive exactly at `grid[x][y]`
     - If `wait` is odd, we need one extra second, arriving at `grid[x][y] + 1`
     - This is elegantly handled by: `nt = grid[x][y] + (grid[x][y] - nt) % 2`
   - **Update if better:** If this new time `nt < dist[x][y]`:
     - Update `dist[x][y] = nt`
     - Push `(nt, x, y)` to the priority queue

**Direction Handling:**
The code uses a clever pattern with `dirs = (-1, 0, 1, 0, -1)` and `pairwise(dirs)` to generate the four directions:
- `(-1, 0)` for up
- `(0, 1)` for right  
- `(1, 0)` for down
- `(0, -1)` for left

**Why This Works:**
The priority queue ensures we always process the cell we can reach earliest. When we update a cell's distance, we're guaranteed to have found the optimal way to reach it considering both movement time and waiting constraints. The modulo operation cleverly handles the back-and-forth movement pattern needed to adjust our arrival time to match the cell's requirement.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this 3x3 grid:
```
grid = [[0, 2, 4],
        [3, 2, 1],
        [1, 0, 4]]
```

**Initial Setup:**
- Start at (0,0) at time 0
- Check if movement is possible: grid[0][1] = 2 and grid[1][0] = 3
- Since 2 > 1 and 3 > 1, we cannot move immediately from start. Return -1.

Let's modify the example to make it solvable:
```
grid = [[0, 1, 4],
        [3, 2, 1],
        [1, 0, 4]]
```

**Step-by-step execution:**

1. **Initialize:**
   - dist = [[0, inf, inf], [inf, inf, inf], [inf, inf, inf]]
   - Priority queue: [(0, 0, 0)]

2. **Process (0, 0, 0)** - time=0, position=(0,0):
   - Check neighbors:
     - Right (0,1): nt = 0 + 1 = 1, grid[0][1] = 1, so arrive at time 1
       - Update dist[0][1] = 1, add (1, 0, 1) to queue
     - Down (1,0): nt = 0 + 1 = 1, grid[1][0] = 3
       - Need to wait: 3 - 1 = 2 (even difference)
       - Arrive at time 3
       - Update dist[1][0] = 3, add (3, 1, 0) to queue

3. **Process (1, 0, 1)** - time=1, position=(0,1):
   - Check neighbors:
     - Left (0,0): Already visited with better time
     - Right (0,2): nt = 1 + 1 = 2, grid[0][2] = 4
       - Need to wait: 4 - 2 = 2 (even difference)
       - Arrive at time 4
       - Update dist[0][2] = 4, add (4, 0, 2) to queue
     - Down (1,1): nt = 1 + 1 = 2, grid[1][1] = 2
       - Can enter exactly at time 2
       - Update dist[1][1] = 2, add (2, 1, 1) to queue

4. **Process (2, 1, 1)** - time=2, position=(1,1):
   - Check neighbors:
     - Up (0,1): Already visited with better time
     - Right (1,2): nt = 2 + 1 = 3, grid[1][2] = 1
       - No wait needed (3 ≥ 1)
       - Update dist[1][2] = 3, add (3, 1, 2) to queue
     - Down (2,1): nt = 2 + 1 = 3, grid[2][1] = 0
       - No wait needed (3 ≥ 0)
       - Update dist[2][1] = 3, add (3, 2, 1) to queue

5. **Continue processing** until we reach (2,2):
   - From (1,2) at time 3: Can reach (2,2) at time 4 (since grid[2][2] = 4)
   - From (2,1) at time 3: Can reach (2,2) at time 5 (need to wait: 4-4=0, but we arrive at 4+1=5 due to odd/even rule)

6. **Result:** The minimum time to reach (2,2) is 4.

**Key observations from this example:**
- We can't always move immediately - sometimes we need to wait
- The waiting mechanism (back-and-forth movement) adds either 0 or 1 extra second depending on parity
- Dijkstra's algorithm ensures we find the optimal path by always processing the earliest reachable cell first
- The initial impossibility check prevents wasted computation when no path exists

## Python Solution

```python
from typing import List
from heapq import heappush, heappop
from math import inf

class Solution:
    def minimumTime(self, grid: List[List[int]]) -> int:
        # Edge case: if both initial adjacent cells require time > 1,
        # we can't move anywhere from start (stuck at position [0,0])
        if grid[0][1] > 1 and grid[1][0] > 1:
            return -1
      
        rows, cols = len(grid), len(grid[0])
      
        # Initialize distance matrix to track minimum time to reach each cell
        min_time = [[inf] * cols for _ in range(rows)]
        min_time[0][0] = 0
      
        # Priority queue: (time, row, col)
        # Using min-heap to always process the cell with minimum time first
        priority_queue = [(0, 0, 0)]
      
        # Direction vectors for moving up, right, down, left
        directions = [(-1, 0), (0, 1), (1, 0), (0, -1)]
      
        while priority_queue:
            current_time, current_row, current_col = heappop(priority_queue)
          
            # If we reached the destination, return the time
            if current_row == rows - 1 and current_col == cols - 1:
                return current_time
          
            # Explore all four adjacent cells
            for delta_row, delta_col in directions:
                next_row = current_row + delta_row
                next_col = current_col + delta_col
              
                # Check if the next position is within grid bounds
                if 0 <= next_row < rows and 0 <= next_col < cols:
                    # Base time to reach the next cell
                    next_time = current_time + 1
                  
                    # If we arrive too early, we need to wait
                    # We can move back and forth to kill time
                    if next_time < grid[next_row][next_col]:
                        wait_time = grid[next_row][next_col] - next_time
                        # If wait time is odd, we arrive at exact time
                        # If wait time is even, we need one more move
                        next_time = grid[next_row][next_col] + wait_time % 2
                  
                    # Only process if we found a better path to this cell
                    if next_time < min_time[next_row][next_col]:
                        min_time[next_row][next_col] = next_time
                        heappush(priority_queue, (next_time, next_row, next_col))
      
        # Should not reach here if a valid path exists
        return -1
```

## Time and Space Complexity

**Time Complexity:** `O(m × n × log(m × n))`

The algorithm uses Dijkstra's algorithm with a min-heap to find the shortest path. In the worst case:
- Each cell `(i, j)` in the `m × n` grid can be visited and added to the heap
- The heap can contain at most `m × n` elements
- Each heap operation (push and pop) takes `O(log(m × n))` time
- We perform at most `O(m × n)` pop operations (one for each cell)
- For each popped cell, we explore up to 4 neighbors, potentially pushing them to the heap
- Total heap operations: `O(m × n)` pops and `O(m × n)` pushes
- Therefore, the overall time complexity is `O(m × n × log(m × n))`

**Space Complexity:** `O(m × n)`

The space is used for:
- The `dist` matrix: `O(m × n)` to store the minimum time to reach each cell
- The priority queue `q`: In the worst case, it can contain `O(m × n)` elements
- Other variables use constant space
- Therefore, the total space complexity is `O(m × n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of the Waiting Time Calculation**

One of the most common mistakes is incorrectly calculating the arrival time when we need to wait before entering a cell. Many developers initially write something like:

```python
# INCORRECT approach
if next_time < grid[next_row][next_col]:
    next_time = grid[next_row][next_col]  # Just wait until we can enter
```

**Why this fails:** This doesn't account for the parity constraint. When we arrive too early at a cell, we can't just stand still and wait. We must keep moving (back and forth between cells) until we can enter. This back-and-forth movement means we can only arrive at times with specific parity.

**The correct solution:** We need to consider whether the waiting time is odd or even:
```python
if next_time < grid[next_row][next_col]:
    wait_time = grid[next_row][next_col] - next_time
    next_time = grid[next_row][next_col] + wait_time % 2
```

This ensures that if we need to wait an even number of seconds, we add 1 extra second (since moving back and forth takes 2 seconds per cycle).

### 2. **Missing the Initial Impossibility Check**

Another critical pitfall is forgetting to check if movement is possible at all from the starting position:

```python
# MISSING this crucial check
if grid[0][1] > 1 and grid[1][0] > 1:
    return -1
```

**Why this matters:** If both adjacent cells from (0,0) require time > 1 to enter, and we start at time 0, we're stuck. We can't move anywhere to "kill time" because there's nowhere to go. Without this check, the algorithm might run indefinitely or return an incorrect result.

### 3. **Using BFS Instead of Dijkstra's Algorithm**

Some might attempt to solve this with standard BFS:

```python
# INCORRECT: Using regular BFS
from collections import deque
queue = deque([(0, 0, 0)])  # time, row, col
visited = set()
```

**Why BFS fails:** BFS assumes all edges have equal weight, but in this problem, the actual time to move between cells varies based on the waiting requirement. We need Dijkstra's algorithm (using a priority queue) to always process the cell we can reach earliest, not just the cell that's fewest moves away.

### 4. **Not Updating Distance Only When Better**

A subtle bug can occur if we don't properly check whether we've found a better path:

```python
# INCORRECT: Missing the optimization check
heappush(priority_queue, (next_time, next_row, next_col))
min_time[next_row][next_col] = next_time
```

**The correct approach:**
```python
if next_time < min_time[next_row][next_col]:
    min_time[next_row][next_col] = next_time
    heappush(priority_queue, (next_time, next_row, next_col))
```

Without this check, we might process the same cell multiple times with suboptimal paths, leading to incorrect results or performance issues.

