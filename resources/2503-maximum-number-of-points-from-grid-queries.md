# 2503. Maximum Number of Points From Grid Queries

> Source: https://algo.monster/liteproblems/2503

## Problem Statement

You have an `m x n` matrix called `grid` filled with integers, and you need to process `k` queries stored in an array `queries`.

For each query value `queries[i]`, you start at the top-left corner of the matrix (position `[0,0]`) and explore the grid following these rules:

- From your current cell, you can only move to an adjacent cell (up, down, left, or right) if `queries[i]` is **strictly greater** than the value in your current cell
- When you visit a cell for the **first time** during this query, you earn 1 point
- If you revisit a cell, you don't earn additional points
- The exploration stops when you can't move anymore (when `queries[i]` is not greater than the current cell value)

Your task is to return an array `answer` where `answer[i]` represents the maximum number of points you can earn for `queries[i]`.

**Key Points:**
- Each query is processed independently - you always start fresh from `[0,0]`
- You can visit the same cell multiple times during one query, but only score points on the first visit
- Movement is only allowed to adjacent cells (4-directional: up, down, left, right)
- Movement requires the query value to be strictly greater than the current cell's value

**Example:**
If you have a 3x3 grid and a query value of 5, you start at `[0,0]`. If `grid[0][0] = 2`, since 5 > 2, you get 1 point and can move to adjacent cells. You continue exploring, earning points for each new cell you visit where the cell value is less than 5.

## Intuition

The key insight is recognizing that for each query, we're essentially asking: "How many cells can I reach from `[0,0]` if I can only step on cells with values less than my query value?"

If we process queries independently, we'd need to run a BFS for each query, which would be inefficient. However, notice that if we can reach a set of cells for query value `x`, we can definitely reach those same cells (and possibly more) for any query value `y` where `y > x`. This monotonic property suggests we can reuse our work.

Instead of starting fresh for each query, what if we process queries in ascending order? For a smaller query value, we explore and count reachable cells. When we move to a larger query value, we can continue from where we left off, expanding our explored region to include cells with values less than the new query threshold.

Think of it like water flooding a terrain: if water level is at height 3, it covers all land below height 3. When water rises to height 5, it keeps everything it already covered and floods additional land between heights 3 and 5.

To implement this efficiently, we use a min-heap to always know the next smallest unvisited cell value. We start from `[0,0]` and gradually expand our explored region. For each sorted query:
1. Pop all cells from the heap whose values are less than the current query value
2. Mark them as visited and increment our count
3. Add their unvisited neighbors to the heap
4. The current count is the answer for this query

By processing queries in sorted order but maintaining their original indices, we can fill in the answer array correctly. This approach transforms multiple BFS runs into a single, incremental exploration of the grid, dramatically improving efficiency from `O(k * m * n)` to `O(m * n * log(m * n) + k * log k)` where `k` is the number of queries.

**Learn more about [Breadth-First Search](/problems/bfs_intro), [Union Find](/problems/dsu_intro), [Two Pointers](/problems/two_pointers_intro), [Sorting](/problems/sorting_summary) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation follows the offline query processing pattern with BFS using a priority queue (min-heap):

**1. Query Preprocessing:**
First, we sort the queries while preserving their original indices. We create a list of tuples `qs = sorted((v, i) for i, v in enumerate(queries))` where each tuple contains `(query_value, original_index)`. This allows us to process queries in ascending order while still being able to place answers in the correct positions.

**2. Data Structures Setup:**
- **Min-Heap `q`:** Initialized with the starting cell `[(grid[0][0], 0, 0)]` containing `(cell_value, row, col)`
- **Visited Matrix `vis`:** A 2D boolean array to track which cells have been visited, initially all `False` except `vis[0][0] = True`
- **Counter `cnt`:** Tracks the number of cells visited so far
- **Answer Array `ans`:** Initialized with zeros, same length as queries

**3. Processing Queries in Order:**
For each sorted query `(v, k)` where `v` is the query value and `k` is the original index:

```python
while q and q[0][0] < v:
    _, i, j = heappop(q)
    cnt += 1
    # Explore 4 adjacent cells
```

This loop continues popping cells from the heap as long as the minimum cell value is less than the current query value `v`. Each popped cell contributes 1 to our count.

**4. Exploring Adjacent Cells:**
For each popped cell at position `(i, j)`, we check all 4 adjacent cells:
```python
for a, b in pairwise((-1, 0, 1, 0, -1)):
    x, y = i + a, j + b
```

The `pairwise` function with `(-1, 0, 1, 0, -1)` generates the pairs: `(-1, 0)`, `(0, 1)`, `(1, 0)`, `(0, -1)`, representing movements up, right, down, and left respectively.

**5. Adding New Cells to Heap:**
For each valid adjacent cell that hasn't been visited:
- Check boundaries: `0 <= x < m and 0 <= y < n`
- Check if unvisited: `not vis[x][y]`
- If valid, add to heap: `heappush(q, (grid[x][y], x, y))`
- Mark as visited: `vis[x][y] = True`

**6. Recording Answers:**
After processing all cells with values less than the current query `v`, we record `ans[k] = cnt`. The count `cnt` accumulates across queries since larger queries include all cells accessible by smaller queries.

**Time Complexity:** `O(m * n * log(m * n) + k * log k)` where:
- `m * n * log(m * n)` for heap operations on all grid cells
- `k * log k` for [sorting](/problems/sorting_summary) queries

**Space Complexity:** `O(m * n)` for the visited matrix and heap storage.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- Grid (3x3):
```
[1, 3, 5]
[2, 6, 4]
[7, 8, 9]
```
- Queries: `[4, 6, 2]`

**Step 1: Query Preprocessing**
Sort queries with original indices:
- `qs = [(2, 2), (4, 0), (6, 1)]`
- This means: process value 2 (originally at index 2), then 4 (index 0), then 6 (index 1)

**Step 2: Initial Setup**
- Min-heap: `[(1, 0, 0)]` (starting cell value=1 at position [0,0])
- Visited: `vis[0][0] = True`, all others `False`
- Counter: `cnt = 0`
- Answer array: `ans = [0, 0, 0]`

**Step 3: Process Query value=2 (original index 2)**
- While heap top (1) < 2:
  - Pop `(1, 0, 0)`, increment `cnt = 1`
  - Check adjacents:
    - Right [0,1]: value=3, not visited → push `(3, 0, 1)` to heap, mark visited
    - Down [1,0]: value=2, not visited → push `(2, 1, 0)` to heap, mark visited
  - Heap now: `[(2, 1, 0), (3, 0, 1)]`
- Heap top (2) is NOT < 2, so stop
- `ans[2] = 1` (we visited 1 cell with value < 2)

**Step 4: Process Query value=4 (original index 0)**
- While heap top (2) < 4:
  - Pop `(2, 1, 0)`, increment `cnt = 2`
  - Check adjacents:
    - Right [1,1]: value=6, not visited → push `(6, 1, 1)` to heap, mark visited
    - Down [2,0]: value=7, not visited → push `(7, 2, 0)` to heap, mark visited
  - Heap now: `[(3, 0, 1), (6, 1, 1), (7, 2, 0)]`
- Continue while heap top (3) < 4:
  - Pop `(3, 0, 1)`, increment `cnt = 3`
  - Check adjacents:
    - Right [0,2]: value=5, not visited → push `(5, 0, 2)` to heap, mark visited
    - Down [1,1]: already visited, skip
  - Heap now: `[(5, 0, 2), (6, 1, 1), (7, 2, 0)]`
- Heap top (5) is NOT < 4, so stop
- `ans[0] = 3` (we've visited 3 cells total with value < 4)

**Step 5: Process Query value=6 (original index 1)**
- While heap top (5) < 6:
  - Pop `(5, 0, 2)`, increment `cnt = 4`
  - Check adjacents:
    - Down [1,2]: value=4, not visited → push `(4, 1, 2)` to heap, mark visited
  - Heap now: `[(4, 1, 2), (6, 1, 1), (7, 2, 0)]`
- Continue while heap top (4) < 6:
  - Pop `(4, 1, 2)`, increment `cnt = 5`
  - Check adjacents:
    - Left [1,1]: already visited, skip
    - Down [2,2]: value=9, not visited → push `(9, 2, 2)` to heap, mark visited
  - Heap now: `[(6, 1, 1), (7, 2, 0), (9, 2, 2)]`
- Heap top (6) is NOT < 6, so stop
- `ans[1] = 5` (we've visited 5 cells total with value < 6)

**Final Answer:** `[3, 5, 1]`
- Query 4: can reach 3 cells (values 1, 2, 3)
- Query 6: can reach 5 cells (values 1, 2, 3, 5, 4)
- Query 2: can reach 1 cell (value 1)

## Python Solution

```python
from typing import List
from heapq import heappush, heappop

class Solution:
    def maxPoints(self, grid: List[List[int]], queries: List[int]) -> List[int]:
        rows, cols = len(grid), len(grid[0])
      
        # Sort queries with their original indices to process in ascending order
        sorted_queries = sorted((value, index) for index, value in enumerate(queries))
      
        # Initialize result array to store answers in original query order
        result = [0] * len(sorted_queries)
      
        # Min heap to store cells to explore: (cell_value, row, col)
        # Start from top-left corner (0, 0)
        min_heap = [(grid[0][0], 0, 0)]
      
        # Counter for reachable cells
        reachable_count = 0
      
        # Track visited cells to avoid revisiting
        visited = [[False] * cols for _ in range(rows)]
        visited[0][0] = True
      
        # Process each query in ascending order
        for query_value, original_index in sorted_queries:
            # Expand reachable area: process all cells with value < query_value
            while min_heap and min_heap[0][0] < query_value:
                cell_value, row, col = heappop(min_heap)
                reachable_count += 1
              
                # Check all 4 adjacent cells (up, right, down, left)
                directions = [(-1, 0), (0, 1), (1, 0), (0, -1)]
                for row_delta, col_delta in directions:
                    new_row, new_col = row + row_delta, col + col_delta
                  
                    # Add valid unvisited neighbors to the heap
                    if (0 <= new_row < rows and 
                        0 <= new_col < cols and 
                        not visited[new_row][new_col]):
                        heappush(min_heap, (grid[new_row][new_col], new_row, new_col))
                        visited[new_row][new_col] = True
          
            # Store the count for this query at its original position
            result[original_index] = reachable_count
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(k × log k + m × n × log(m × n))`

The time complexity consists of two main parts:
- Sorting the queries array takes `O(k × log k)` where `k` is the number of queries
- Processing the grid using a min-heap where each cell is visited at most once. Since we have `m × n` cells total, and each heap operation (push/pop) takes `O(log(heap_size))` time, where the heap can contain at most `m × n` elements, the grid traversal takes `O(m × n × log(m × n))`

**Space Complexity:** `O(m × n)`

The space complexity is dominated by:
- The `vis` (visited) array which stores a boolean value for each cell: `O(m × n)`
- The heap `q` which can contain at most `m × n` elements (one for each cell): `O(m × n)`
- The sorted queries list and answer array: `O(k)`

Since `m × n` is typically larger than `k`, the overall space complexity is `O(m × n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Marking Cells as Visited at the Wrong Time**

**The Pitfall:** A common mistake is marking cells as visited only when they are popped from the heap (when processing them), rather than when they are first added to the heap.

**Why It's Wrong:**
```python
# INCORRECT approach:
while min_heap and min_heap[0][0] < query_value:
    cell_value, row, col = heappop(min_heap)
    if visited[row][col]:  # Skip if already visited
        continue
    visited[row][col] = True  # Mark as visited when popped
    reachable_count += 1
  
    for row_delta, col_delta in directions:
        new_row, new_col = row + row_delta, col + col_delta
        if (0 <= new_row < rows and 0 <= new_col < cols):
            heappush(min_heap, (grid[new_row][new_col], new_row, new_col))
            # NOT marking as visited here!
```

This leads to:
- **Duplicate entries in the heap:** The same cell can be added multiple times from different neighbors
- **Incorrect counting:** When duplicates are popped, they might be counted multiple times
- **Performance degradation:** The heap grows unnecessarily large with duplicates

**The Solution:** Always mark cells as visited immediately when adding them to the heap:
```python
# CORRECT approach:
if (0 <= new_row < rows and 
    0 <= new_col < cols and 
    not visited[new_row][new_col]):
    heappush(min_heap, (grid[new_row][new_col], new_row, new_col))
    visited[new_row][new_col] = True  # Mark immediately when added
```

### 2. **Resetting State Between Queries**

**The Pitfall:** Another common mistake is resetting the visited matrix or reachable count between queries, thinking each query should be processed independently from scratch.

**Why It's Wrong:**
```python
# INCORRECT approach:
for query_value, original_index in sorted_queries:
    visited = [[False] * cols for _ in range(rows)]  # Reset visited
    reachable_count = 0  # Reset count
    min_heap = [(grid[0][0], 0, 0)]  # Reset heap
    # ... process query
```

This breaks the optimization because:
- Since queries are sorted in ascending order, cells reachable by a smaller query are also reachable by all larger queries
- Resetting would require re-exploring the same cells multiple times, defeating the purpose of sorting queries

**The Solution:** Maintain cumulative state across queries. The visited cells and count accumulate as we process larger queries, which is why we sort them first. Each larger query inherits all cells accessible by smaller queries plus any additional cells it can reach.

