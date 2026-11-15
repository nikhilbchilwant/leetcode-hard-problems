# 749. Contain Virus

> Source: https://algo.monster/liteproblems/749

## Problem Statement

You're given an `m x n` grid representing a world where a virus is spreading. In this grid:
- `isInfected[i][j] == 0` means the cell is uninfected (healthy)
- `isInfected[i][j] == 1` means the cell is infected with the virus

The virus spreads each night to all 4-directionally adjacent cells (up, down, left, right) unless blocked by a wall. You can install walls between any two adjacent cells on their shared boundary.

Each day, you have limited resources and can only build walls around **one infected region** (a continuous block of connected infected cells). You must choose the region that threatens the most uninfected cells for the next night's spread. The problem guarantees there will never be a tie in this choice.

The process works as follows:
1. During the day, identify all separate infected regions
2. Determine which region would infect the most uninfected cells if not contained
3. Build walls completely around that most dangerous region to quarantine it
4. At night, all non-quarantined infected regions spread to their adjacent uninfected cells
5. Repeat this process until either all infected regions are quarantined or the entire world becomes infected

Your task is to return the total number of walls used throughout this entire process. If the world becomes fully infected before you can contain all regions, return the number of walls used up to that point.

For example, if you have multiple infected regions and one threatens 3 uninfected cells while another threatens 2, you would quarantine the first region by building walls around its entire perimeter, then let the second region spread overnight before reassessing the situation the next day.

## Intuition

The key insight is that we need to simulate the virus containment process day by day. Each day, we face a decision: which infected region should we quarantine? The greedy choice is clear - we should always quarantine the region that threatens the most uninfected cells, as this prevents the maximum spread for the next night.

To implement this strategy, we need to solve several subproblems:

1. **Identify all separate infected regions**: Since infected cells form connected components in the grid, we can use DFS to explore each region. Starting from any unvisited infected cell, DFS will naturally traverse all connected infected cells, giving us one complete region.

2. **Count walls needed for each region**: As we explore a region with DFS, every time we encounter a boundary between an infected cell and an uninfected cell, we need a wall. The trick is that if an infected region touches the same uninfected cell from multiple sides, we need multiple walls (one for each shared boundary).

3. **Identify threatened uninfected cells**: While counting walls, we also track which uninfected cells are adjacent to each infected region. These are the cells that would become infected overnight if we don't quarantine this region. We use a set to avoid counting the same threatened cell multiple times.

4. **Make the greedy choice**: After analyzing all regions, we quarantine the one threatening the most unique uninfected cells (the one with the largest boundary set). We mark these quarantined cells with `-1` to indicate they're contained and won't spread anymore.

5. **Simulate the spread**: For all non-quarantined regions, we let the virus spread to their adjacent uninfected cells. This changes the grid state for the next day.

6. **Repeat until done**: We continue this process daily until either all infected regions are quarantined (no more regions found) or the world becomes fully infected.

The beauty of this approach is that DFS naturally handles the exploration of irregular-shaped regions, and by tracking three pieces of information simultaneously (region cells, wall count, and threatened cells), we can make optimal decisions at each step. The solution elegantly combines graph traversal with greedy strategy to solve what initially seems like a complex simulation problem.

**Learn more about [Depth-First Search](/problems/dfs_intro) and [Breadth-First Search](/problems/bfs_intro) patterns.**

## Solution Approach

The implementation uses a simulation approach with DFS to identify and process infected regions iteratively. Let's walk through the key components:

**Main Loop Structure:**
The solution runs an infinite loop that continues until no infected regions remain. Each iteration represents one day in the simulation.

**Data Structures Used:**
- `vis`: A 2D boolean array to track visited cells during DFS traversal
- `areas`: List of lists, where each inner list contains coordinates `(i, j)` of cells belonging to one infected region
- `boundaries`: List of sets, where each set contains coordinates of unique uninfected cells threatened by the corresponding region
- `c`: List of integers, where each element counts the total walls needed to quarantine the corresponding region

**DFS Implementation:**
The `dfs(i, j)` function explores an infected region starting from cell `(i, j)`:
1. Mark the current cell as visited: `vis[i][j] = True`
2. Add the cell to the current region: `areas[-1].append((i, j))`
3. Explore all 4 directions `[[0, -1], [0, 1], [-1, 0], [1, 0]]`
4. For each neighbor `(x, y)`:
   - If it's an unvisited infected cell (`isInfected[x][y] == 1`), recursively call DFS
   - If it's an uninfected cell (`isInfected[x][y] == 0`):
     - Increment wall count: `c[-1] += 1` (we need one wall for this boundary)
     - Add to threatened cells set: `boundaries[-1].add((x, y))`

**Daily Process:**
1. **Initialize tracking structures** for the current day
2. **Identify all infected regions** by iterating through the grid and calling DFS on unvisited infected cells
3. **Break if no regions found** (all infections contained)
4. **Select the most dangerous region**: `idx = boundaries.index(max(boundaries, key=len))`
   - This finds the region threatening the most unique uninfected cells
5. **Add walls to answer**: `ans += c[idx]`
6. **Process each region**:
   - For the selected region (`k == idx`): Mark all cells as quarantined using `-1`
   - For other regions: Spread the virus to adjacent uninfected cells by setting them to `1`

**Key Observations:**
- The wall count `c[idx]` correctly counts all walls needed because DFS increments the counter for every infected-uninfected boundary encountered
- Using a set for `boundaries` ensures each threatened cell is counted only once per region, even if touched from multiple sides
- Marking quarantined cells as `-1` prevents them from being processed in future iterations
- The virus spread is simulated by changing uninfected neighbors of non-quarantined regions from `0` to `1`

**Time Complexity:** `O(m*n*days)` where days is the number of iterations needed
**Space Complexity:** `O(m*n)` for the various tracking structures

The algorithm continues until either all infected regions are successfully quarantined or the infection spreads completely, returning the total number of walls used throughout the process.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach:

```
Initial grid (3x4):
0 1 1 0
1 1 0 0
0 0 1 0
```

**Day 1:**

1. **Identify infected regions using DFS:**
   - Starting from (0,1), DFS explores (directions in order left, right, up, down): (0,1) → (0,2) → (1,1) → (1,0)
     - Region 1: [(0,1), (0,2), (1,1), (1,0)]
   - Starting from (2,2), DFS explores: (2,2)
     - Region 2: [(2,2)]

2. **Count walls and threatened cells for each region:**
   - Region 1 walls and boundaries:
     - From (0,1): wall to (0,0), so c[0] = 1, boundaries[0] = {(0,0)}
     - From (0,2): walls to (0,3) and (1,2), so c[0] = 3, boundaries[0] = {(0,0), (0,3), (1,2)}
     - From (1,1): walls to (1,2) and (2,1), so c[0] = 5, boundaries[0] = {(0,0), (0,3), (1,2), (2,1)}
     - From (1,0): walls to (0,0) and (2,0), so c[0] = 7, boundaries[0] = {(0,0), (0,3), (1,2), (2,1), (2,0)}
     - Total: 7 walls needed, threatens 5 unique cells ((0,0) and (1,2) are each touched from two sides)

   - Region 2 walls and boundaries:
     - From (2,2): walls to (2,1), (2,3), (1,2), so c[1] = 3
     - boundaries[1] = {(2,1), (2,3), (1,2)}
     - Total: 3 walls needed, threatens 3 unique cells

3. **Select most dangerous region:**
   - Region 1 threatens 5 cells vs Region 2 threatens 3 cells
   - Choose Region 1 to quarantine
   - Add 7 walls to total: ans = 7

4. **Update grid:**
   - Quarantine Region 1 (mark with -1):
     ```
     0 -1 -1 0
     -1 -1 0 0
     0  0  1 0
     ```
   - Let Region 2 spread overnight:
     ```
     0 -1 -1 0
     -1 -1 1 0
     0  1  1 1
     ```

**Day 2:**

1. **Identify remaining infected regions:**
   - Starting from (1,2), DFS explores: (1,2) → (2,2) → (2,1) → (2,3)
   - Region: [(1,2), (2,2), (2,1), (2,3)]

2. **Count walls and threatened cells:**
   - From (1,2): neighbors (1,1) and (0,2) are -1 (no wall counted), (1,3) is 0, so c[0] = 1
   - From (2,2): no uninfected neighbors
   - From (2,1): (2,0) is 0, so c[0] = 2; (1,1) is -1
   - From (2,3): (1,3) is 0, so c[0] = 3
   - Total: 3 walls needed, threatens {(1,3), (2,0)}

3. **Quarantine the only region:**
   - Add 3 walls to total: ans = 7 + 3 = 10

4. **Final grid (all infected regions contained):**
   ```
   0 -1 -1 0
   -1 -1 -1 0
   0 -1 -1 -1
   ```

**Day 3:** No cell has value 1, so no regions are found and the loop ends.

**Result:** Total walls used = 10

This example demonstrates how the algorithm:
- Uses DFS to identify connected infected regions
- Counts walls needed by checking boundaries with uninfected cells
- Makes greedy choices to quarantine the most threatening region
- Simulates virus spread for non-quarantined regions
- Continues until all infections are contained

## Python Solution

```python
class Solution:
    def containVirus(self, isInfected: List[List[int]]) -> int:
        def dfs(row: int, col: int) -> None:
            """
            Depth-first search to explore a virus region.
            Collects infected cells, counts walls needed, and identifies threatened cells.
            """
            visited[row][col] = True
            current_region.append((row, col))
          
            # Check all 4 adjacent cells
            for delta_row, delta_col in [[0, -1], [0, 1], [-1, 0], [1, 0]]:
                new_row, new_col = row + delta_row, col + delta_col
              
                if 0 <= new_row < rows and 0 <= new_col < cols:
                    if isInfected[new_row][new_col] == 1 and not visited[new_row][new_col]:
                        # Continue exploring infected cells
                        dfs(new_row, new_col)
                    elif isInfected[new_row][new_col] == 0:
                        # Count walls needed and track threatened cells
                        walls_needed[-1] += 1
                        threatened_cells[-1].add((new_row, new_col))
      
        rows, cols = len(isInfected), len(isInfected[0])
        total_walls = 0
      
        while True:
            # Initialize tracking structures for this iteration
            visited = [[False] * cols for _ in range(rows)]
            infected_regions = []  # List of infected cell coordinates for each region
            walls_needed = []      # Number of walls needed for each region
            threatened_cells = []  # Set of uninfected cells threatened by each region
          
            # Find all distinct infected regions
            for i in range(rows):
                for j in range(cols):
                    if isInfected[i][j] == 1 and not visited[i][j]:
                        # Start new region exploration
                        current_region = []
                        infected_regions.append(current_region)
                        threatened_cells.append(set())
                        walls_needed.append(0)
                        dfs(i, j)
          
            # If no infected regions remain, we're done
            if not infected_regions:
                break
          
            # Find the region that threatens the most cells
            most_dangerous_idx = threatened_cells.index(max(threatened_cells, key=len))
          
            # Add walls needed for the most dangerous region
            total_walls += walls_needed[most_dangerous_idx]
          
            # Process each region
            for region_idx, region in enumerate(infected_regions):
                if region_idx == most_dangerous_idx:
                    # Quarantine the most dangerous region (mark as contained with -1)
                    for i, j in region:
                        isInfected[i][j] = -1
                else:
                    # Spread virus from other regions to adjacent uninfected cells
                    for i, j in region:
                        for delta_row, delta_col in [[0, -1], [0, 1], [-1, 0], [1, 0]]:
                            new_row, new_col = i + delta_row, j + delta_col
                            if (0 <= new_row < rows and 0 <= new_col < cols 
                                and isInfected[new_row][new_col] == 0):
                                isInfected[new_row][new_col] = 1
      
        return total_walls
```

## Time and Space Complexity

**Time Complexity:** `O(m * n * (m + n))`

The algorithm runs in a loop that continues until no more infected regions exist. In the worst case, we might need to quarantine regions `O(m * n)` times (when each cell becomes infected one at a time).

For each iteration:
- We traverse the entire grid to find infected regions: `O(m * n)`
- The DFS traversal visits each cell at most once: `O(m * n)`
- For each infected cell during DFS, we check 4 neighbors: `O(1)`
- Finding the maximum boundary set: `O(number of regions)` which is at most `O(m * n)`
- Marking quarantined areas and spreading infection: `O(m * n)`

Since we might have up to `O(min(m * n, m + n))` iterations (practically bounded by the grid's dimensions for spreading), the overall time complexity is `O(m * n * (m + n))`.

**Space Complexity:** `O(m * n)`

The space usage includes:
- `vis` array: `O(m * n)` for each iteration
- `areas` list: stores all infected cells, at most `O(m * n)`
- `boundaries` list of sets: stores unique boundary cells, at most `O(m * n)`
- `c` list: stores wall counts for each region, at most `O(m * n)` regions
- DFS recursion stack: at most `O(m * n)` in the worst case when all cells form a single connected component

The dominant space usage is `O(m * n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrectly Counting Walls for Shared Boundaries

**The Issue:**
A critical mistake is misunderstanding how walls should be counted when an infected region borders the same uninfected cell from multiple sides. Consider this scenario:

```
0 1 0
1 1 1
0 1 0
```

The plus-shaped infected region threatens the four corner cells, and each corner touches the region on two sides. Counting only unique threatened cells gives 4 walls, but the correct count is 8 walls (one for each shared boundary).

**Why This Happens:**
Developers often confuse two different metrics:
- The number of unique uninfected cells threatened (used to determine which region to quarantine)
- The total number of walls needed (every infected-uninfected boundary needs a wall)

**The Solution:**
The code correctly handles this by:
1. Using a **set** for `threatened_cells` to count unique uninfected cells (for region selection)
2. Using a **counter** `walls_needed` that increments for every boundary encountered (for wall counting)

```python
elif isInfected[new_row][new_col] == 0:
    walls_needed[-1] += 1  # Count EVERY boundary
    threatened_cells[-1].add((new_row, new_col))  # Count UNIQUE cells
```

### Pitfall 2: Virus Spreading During the Wrong Phase

**The Issue:**
Another common mistake is spreading the virus immediately when exploring regions with DFS, rather than waiting until after quarantine decisions are made. This would cause the simulation to behave incorrectly, as the virus should only spread at night after walls are built during the day.

**Why This Happens:**
It's tempting to modify the grid during DFS exploration for efficiency, but this violates the problem's day/night cycle where:
1. Day: Analyze regions and build walls
2. Night: Non-quarantined regions spread

**The Solution:**
The code maintains proper timing by:
1. Using DFS only for analysis (collecting region data without modifying the grid)
2. Modifying the grid only after selecting the quarantine target:
   - Mark quarantined cells as `-1`
   - Spread virus from non-quarantined regions to adjacent cells

```python
# First analyze all regions without modification
for i in range(rows):
    for j in range(cols):
        if isInfected[i][j] == 1 and not visited[i][j]:
            # Only collect data, don't modify
            dfs(i, j)

# Then modify the grid based on decisions
for region_idx, region in enumerate(infected_regions):
    if region_idx == most_dangerous_idx:
        # Quarantine
        for i, j in region:
            isInfected[i][j] = -1
    else:
        # Spread virus
        for i, j in region:
            # Spread to adjacent uninfected cells
```

### Pitfall 3: Not Handling Already Quarantined Regions

**The Issue:**
Failing to properly mark quarantined regions can cause them to be processed again in subsequent iterations, leading to incorrect wall counts or infinite loops.

**Why This Happens:**
Simply changing quarantined cells to `0` (uninfected) would make them vulnerable to reinfection from neighboring regions. Leaving them as `1` would cause them to be processed again.

**The Solution:**
The code uses `-1` as a special marker for quarantined cells:
- They won't be considered as infected (`== 1`) in future DFS explorations
- They won't be treated as uninfected (`== 0`) targets for virus spread
- This effectively removes them from all future processing

```python
if isInfected[i][j] == 1 and not visited[i][j]:  # Skips -1 values
    # Process only active infected regions
```

