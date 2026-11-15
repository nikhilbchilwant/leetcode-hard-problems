# 407. Trapping Rain Water II

> Source: https://algo.monster/liteproblems/407

## Problem Statement

You are given a 2D elevation map represented as an `m x n` matrix called `heightMap`, where each cell contains an integer value representing the height at that position. When it rains, water gets trapped in the lower areas between higher elevations. Your task is to calculate the total volume of water that can be trapped after raining.

The water trapping works similar to a real-world scenario: water fills up the lower areas and is contained by the surrounding higher walls. Water can only be trapped if it's surrounded by heights that are greater than or equal to the water level on all sides (including diagonally adjacent cells are not considered - only up, down, left, right).

Think of it like pouring water onto a 3D terrain model - the water will pool in valleys and depressions, with the boundaries of the matrix acting as edges where water can flow off. The volume of trapped water at each cell is determined by the minimum height of all possible paths to reach the boundary of the matrix.

For example:
- If a cell with height 3 is completely surrounded by cells with heights 5 or greater, and the minimum "wall" height to reach the boundary is 5, then that cell can trap 2 units of water (5 - 3 = 2).
- Cells on the boundary of the matrix cannot trap any water as water would flow off the edge.

The function should return the total volume of water that can be trapped across the entire elevation map.

## Intuition

Think about how water behaves in real life - it always flows from high to low areas and eventually finds its way out through the lowest possible exit point. In this 2D elevation map, water can only escape through the boundary cells, so the key insight is: **the amount of water that can be trapped at any cell depends on the minimum height barrier it needs to overcome to reach the boundary**.

Imagine you're standing at an interior cell and water is rising around you. The water level at your position can only rise as high as the lowest "wall" you'd need to climb over to escape to the boundary. If that minimum escape height is 5 and you're standing at height 3, then water can fill up to level 5, trapping 2 units of water above you.

To find this minimum escape height for each cell, we can think of it like flood-filling from the outside inward. We start from all boundary cells simultaneously (since water can escape from any boundary) and gradually move inward, always processing the lowest height cells first. This is crucial because:

1. **Water enters from the lowest points first**: Just like pouring water into a container, it will overflow at the lowest rim first. By using a min-heap to always process the cell with minimum height, we simulate this natural water flow.

2. **The water level is determined by the path of least resistance**: When we visit a new cell from a processed cell, the water level at the new cell is at least as high as the maximum we've seen so far on the path from the boundary. This is because water needs to overcome all barriers on its way out.

3. **Once a cell is visited, we know its final water level**: Since we always process cells in order of increasing height from the boundaries, when we reach a cell, we've already found the optimal (minimum height) path to the boundary. Any other path would have a higher maximum height.

The algorithm essentially simulates water "flooding" inward from all boundaries simultaneously, with the water level at each point determined by the minimum height barrier needed to reach that point from any boundary. The difference between this water level and the actual ground height gives us the trapped water volume at each cell.

## Solution Approach

The implementation uses a **priority queue (min-heap)** combined with **BFS traversal** to simulate water flooding from the boundaries inward:

**1. Initialize the boundary:**
- Create a visited matrix `vis` to track processed cells
- Use a min-heap `pq` to store cells as tuples `(height, row, col)`
- Add all boundary cells (first/last row and first/last column) to the heap and mark them as visited
- These boundary cells represent where water can escape, so no water can be trapped there

**2. Process cells from lowest to highest:**
```python
while pq:
    h, i, j = heappop(pq)  # Get the cell with minimum height
```
- Extract the cell with the minimum height from the heap
- The variable `h` represents the water level at this position (the minimum height needed to reach the boundary)

**3. Check all 4 adjacent neighbors:**
```python
dirs = (-1, 0, 1, 0, -1)  # Compact way to represent 4 directions
for a, b in pairwise(dirs):  # Generates pairs: (-1,0), (0,1), (1,0), (0,-1)
    x, y = i + a, j + b  # Calculate neighbor coordinates
```
- Use the `pairwise` function to generate direction pairs for up, right, down, left
- For each unvisited neighbor within bounds

**4. Calculate trapped water:**
```python
ans += max(0, h - heightMap[x][y])
```
- The water level at the current position is `h` (inherited from the path to boundary)
- If `h > heightMap[x][y]`, water is trapped: volume = `h - heightMap[x][y]`
- If `h <= heightMap[x][y]`, no water is trapped (the `max(0, ...)` handles this)

**5. Update neighbor's water level:**
```python
heappush(pq, (max(h, heightMap[x][y]), x, y))
```
- The water level at the neighbor is the maximum of:
  - Current water level `h` (water can't flow upward)
  - The neighbor's ground height `heightMap[x][y]` (water can't go below ground)
- Add this neighbor to the heap with its determined water level

**6. Mark as visited:**
```python
vis[x][y] = True
```
- Ensure each cell is processed only once
- Since we process cells in order of increasing water level, the first time we reach a cell gives us the optimal (minimum) water level

The algorithm continues until all reachable cells are processed. The total trapped water volume is accumulated in `ans` and returned as the final result.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach:

```
heightMap = [
  [3, 3, 3],
  [3, 1, 3],
  [3, 3, 3]
]
```

**Step 1: Initialize boundary cells**
- Add all boundary cells to min-heap with their heights
- Mark them as visited (v = visited, . = unvisited)
```
Heap: [(3,0,0), (3,0,1), (3,0,2), (3,1,0), (3,1,2), (3,2,0), (3,2,1), (3,2,2)]
Visited:
[v, v, v]
[v, ., v]
[v, v, v]
```

**Step 2: Process cells from heap (all boundary cells have height 3)**
- Pop (3,0,0), check neighbors - only (1,1) is unvisited but it's not adjacent
- Pop (3,0,1), check neighbors:
  - Neighbor (1,1) is unvisited
  - Water level at current position = 3
  - Ground height at (1,1) = 1
  - Trapped water = 3 - 1 = 2 units
  - Add (1,1) to heap with water level max(3, 1) = 3
  - Mark (1,1) as visited

```
Total water trapped: 2
Heap: [(3,0,2), (3,1,0), (3,1,2), (3,2,0), (3,2,1), (3,2,2), (3,1,1)]
Visited:
[v, v, v]
[v, v, v]
[v, v, v]
```

**Step 3: Continue processing remaining cells**
- All remaining cells in heap are boundary cells or already visited
- No more unvisited neighbors to process
- Algorithm terminates

**Final Result:** 2 units of water trapped

The center cell (1,1) with height 1 is surrounded by walls of height 3. Water fills up to level 3 (the minimum height needed to escape to the boundary), trapping 3 - 1 = 2 units of water above the center cell.

This demonstrates how the algorithm:
1. Starts from all boundaries simultaneously
2. Processes cells in order of increasing height
3. Determines water level based on the minimum barrier to reach the boundary
4. Calculates trapped water as the difference between water level and ground height

## Python Solution

```python
class Solution:
    def trapRainWater(self, heightMap: List[List[int]]) -> int:
        # Get dimensions of the height map
        rows, cols = len(heightMap), len(heightMap[0])
      
        # Track visited cells to avoid processing them multiple times
        visited = [[False] * cols for _ in range(rows)]
      
        # Min-heap to process cells by height (water flows from lowest boundary)
        # Each element is (height, row, col)
        min_heap = []
      
        # Add all boundary cells to the heap as starting points
        # Water cannot be trapped at boundaries - it flows out
        for row in range(rows):
            for col in range(cols):
                if row == 0 or row == rows - 1 or col == 0 or col == cols - 1:
                    heappush(min_heap, (heightMap[row][col], row, col))
                    visited[row][col] = True
      
        # Total water trapped
        total_water = 0
      
        # Direction vectors for exploring 4 adjacent cells (up, right, down, left)
        directions = (-1, 0, 1, 0, -1)
      
        # Process cells from lowest boundary height to highest
        while min_heap:
            # Get the cell with minimum height from boundary
            current_height, current_row, current_col = heappop(min_heap)
          
            # Check all 4 adjacent cells
            for i in range(4):
                next_row = current_row + directions[i]
                next_col = current_col + directions[i + 1]
              
                # Check if the adjacent cell is valid and unvisited
                if (0 <= next_row < rows and 
                    0 <= next_col < cols and 
                    not visited[next_row][next_col]):
                  
                    # Water trapped = difference between current boundary height and cell height
                    # If cell is higher than boundary, no water is trapped (max with 0)
                    total_water += max(0, current_height - heightMap[next_row][next_col])
                  
                    # Mark as visited
                    visited[next_row][next_col] = True
                  
                    # Add to heap with height = max(boundary height, cell height)
                    # This maintains the water level for inner cells
                    heappush(min_heap, (max(current_height, heightMap[next_row][next_col]), 
                                       next_row, next_col))
      
        return total_water
```

## Time and Space Complexity

**Time Complexity:** `O(m * n * log(m * n))`

The algorithm uses a min-heap (priority queue) to process cells from the boundary inward. Breaking down the operations:
- Initially, all boundary cells are added to the heap: `O((2m + 2n - 4))` insertions, each taking `O(log k)` where `k` is the current heap size
- In the worst case, every cell in the grid is visited exactly once and added to the heap: `O(m * n)` cells total
- For each cell popped from the heap, we check up to 4 neighbors (constant time)
- Each heap operation (push/pop) takes `O(log(m * n))` in the worst case when the heap contains all boundary cells
- Total operations: `m * n` pops and up to `m * n` pushes, each costing `O(log(m * n))`

Therefore, the overall time complexity is `O(m * n * log(m * n))`.

**Space Complexity:** `O(m * n)`

The space usage consists of:
- `vis` array: `O(m * n)` to track visited cells
- Priority queue `pq`: In the worst case, it can contain up to `O(m + n)` elements at any time (typically boundary cells and their immediate neighbors), but theoretically could hold up to `O(m * n)` elements if we consider pathological cases
- Recursive call stack: Not applicable as this is an iterative solution
- Other variables: `O(1)` for constants and loop variables

The dominant factor is the visited array and potentially the heap, giving us `O(m * n)` space complexity.

## Common Pitfalls

### 1. **Incorrect Water Level Propagation**
**Pitfall:** A common mistake is using the cell's own height instead of maintaining the maximum water level when adding neighbors to the heap.

**Incorrect approach:**
```python
# WRONG: This doesn't maintain the water barrier height
heappush(min_heap, (heightMap[next_row][next_col], next_row, next_col))
```

**Why it's wrong:** When water flows inward from boundaries, it maintains a minimum height based on the barriers it had to cross. If we only use the cell's ground height, we lose track of the water level established by outer barriers.

**Correct approach:**
```python
# RIGHT: Maintain the maximum of current water level and ground height
heappush(min_heap, (max(current_height, heightMap[next_row][next_col]), next_row, next_col))
```

### 2. **Processing Cells Multiple Times**
**Pitfall:** Forgetting to mark cells as visited or checking visited status incorrectly can lead to cells being processed multiple times, resulting in counting trapped water multiple times.

**Incorrect approach:**
```python
# WRONG: Marking visited after popping from heap
while min_heap:
    current_height, current_row, current_col = heappop(min_heap)
    visited[current_row][current_col] = True  # Too late!
    # ... rest of the code
```

**Why it's wrong:** By the time we pop a cell from the heap, it might have been added multiple times from different neighbors, leading to duplicate processing.

**Correct approach:**
```python
# RIGHT: Mark visited immediately when adding to heap
if not visited[next_row][next_col]:
    visited[next_row][next_col] = True  # Mark immediately
    heappush(min_heap, (max(current_height, heightMap[next_row][next_col]), next_row, next_col))
```

### 3. **Using Maximum Heap Instead of Minimum Heap**
**Pitfall:** Using a max-heap or processing cells from highest to lowest height.

**Why it's wrong:** Water naturally flows from low to high barriers. By processing from the lowest boundary points first, we correctly simulate how water would fill up the terrain. Processing from highest points would give incorrect water levels.

**Visual Example:**
Consider a simple 3x3 grid:
```
5 5 5
5 1 5  
5 5 5
```
- With min-heap (correct): Process boundaries first (all 5s), then inner cell. Water level = 5, trapped = 5-1 = 4
- With max-heap (wrong): Would incorrectly calculate water levels

### 4. **Edge Case: Small Grids**
**Pitfall:** Not handling grids smaller than 3x3 properly.

**Why it matters:** Grids with dimensions less than 3x3 cannot trap any water since all cells are on the boundary.

**Solution:**
```python
def trapRainWater(self, heightMap: List[List[int]]) -> int:
    rows, cols = len(heightMap), len(heightMap[0])
  
    # Early return for grids too small to trap water
    if rows < 3 or cols < 3:
        return 0
  
    # ... rest of the implementation
```

