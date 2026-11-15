# 827. Making A Large Island

> Source: https://algo.monster/liteproblems/827

## Problem Statement

You have an `n x n` binary matrix `grid` where each cell contains either `0` or `1`. A group of `1`s that are connected horizontally or vertically (4-directionally) forms an island.

The problem asks you to find the size of the largest possible island after changing **at most one** `0` to `1`.

Key points to understand:
- An island consists of `1`s that are connected through their 4 adjacent neighbors (up, down, left, right)
- You can change at most one `0` to `1` (you can also choose not to change any)
- You need to return the maximum island size achievable

For example, if you have a grid like:
```
1 0
0 1
```
By changing the `0` at position `(0,1)` to `1`, you can connect all cells into one island of size 4.

The solution approach uses [depth-first search](/problems/dfs_intro) to:
1. First identify all existing islands and label them with unique identifiers
2. Count the size of each existing island
3. For each `0` in the grid, check what would happen if we changed it to `1` - it would connect all adjacent islands plus itself
4. Track the maximum possible island size throughout this process

The code maintains:
- `island_assignment[i][j]`: stores which island component the cell `(i,j)` belongs to
- `island_sizes[island_id]`: stores the size of the island with identifier `island_id`
- For each `0`, it checks adjacent cells to find unique neighboring islands and calculates the total size if this `0` were changed to `1`

## Intuition

The key insight is that changing a `0` to `1` can potentially merge multiple existing islands into one larger island. Instead of trying every possible change and recalculating island sizes each time (which would be inefficient), we can be smarter about it.

Think of it this way: if we already know all the existing islands and their sizes, then when we consider changing a `0` to `1`, we just need to check which islands this cell would connect. The new island size would be the sum of all connected islands plus 1 (for the cell itself).

Here's the thought process:

1. **First observation**: We need to identify all existing islands. Each island is a connected component of `1`s. We can use DFS to traverse each island and mark all cells belonging to the same island with a unique identifier.

2. **Second observation**: While identifying islands, we can also count their sizes. This way, we know that island #1 has size 5, island #2 has size 3, etc.

3. **Critical insight**: When we change a `0` to `1`, it acts as a "bridge" that can connect up to 4 different islands (the ones adjacent to it in 4 directions). The resulting island size would be:
   - Size of all unique adjacent islands combined
   - Plus 1 for the cell we're changing

4. **Edge cases to consider**:
   - A `0` might be adjacent to the same island from multiple directions (e.g., a `0` surrounded by `1`s from the same island). We need to count each unique island only once.
   - The grid might already be all `1`s, in which case we can't improve it.
   - The grid might be all `0`s, in which case the best we can do is create an island of size 1.

This approach is efficient because we only need to:
- Do one pass to identify and size all islands (using DFS)
- Do another pass to check each `0` and calculate what would happen if we changed it
- Track the maximum size we can achieve

The beauty of this solution is that we precompute all the information we need (island identities and sizes) and then use it to quickly evaluate each potential change.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro) and [Union Find](/problems/dsu_intro) patterns.**

## Solution Approach

The implementation follows the intuition by using DFS to identify and measure islands, then evaluating each `0` position for potential merging. The snippets below use short names: `p` is `island_assignment`, `cnt` is `island_sizes`, `root` is `island_id`, `s` is `adjacent_islands`, and `ans` is `max_island_size` in the Python solution. The Python solution walks the four directions with `directions[k], directions[k + 1]` for `k` in `range(4)`, which produces the same pairs as `pairwise(dirs)`.

**Step 1: Initialize Data Structures**
- `p`: A 2D array where `p[i][j]` stores the island identifier that cell `(i, j)` belongs to. Initially all zeros.
- `cnt`: A Counter (dictionary) where `cnt[root]` stores the size of island with identifier `root`.
- `root`: A counter starting from 0 to assign unique identifiers to each island.
- `dirs`: Direction array `(-1, 0, 1, 0, -1)` used with `pairwise` to generate the 4 directions: up, right, down, left.

**Step 2: Identify and Measure Islands using DFS**
```python
def dfs(i: int, j: int):
    p[i][j] = root  # Mark this cell as belonging to island 'root'
    cnt[root] += 1  # Increment the size of this island
    for a, b in pairwise(dirs):  # Check all 4 directions
        x, y = i + a, j + b
        if 0 <= x < n and 0 <= y < n and grid[x][y] and p[x][y] == 0:
            dfs(x, y)  # Recursively explore unvisited land cells
```

For each unvisited land cell (`grid[i][j] = 1` and `p[i][j] = 0`):
- Increment `root` to get a new island identifier
- Call `dfs(i, j)` to mark all connected cells with this identifier and count them

**Step 3: Find Maximum Existing Island**
```python
ans = max(cnt.values() or [0])
```
This handles the case where we might not need to change any `0` (if the largest existing island is already optimal).

**Step 4: Evaluate Each Water Cell**
For each water cell (`grid[i][j] = 0`):
1. Create a set `s` to store unique adjacent island identifiers
2. Check all 4 adjacent cells:
   ```python
   for a, b in pairwise(dirs):
       x, y = i + a, j + b
       if 0 <= x < n and 0 <= y < n:
           s.add(p[x][y])  # Add the island identifier (0 if water)
   ```
3. Calculate the total size if we change this cell to land:
   ```python
   sum(cnt[root] for root in s) + 1
   ```
   - We sum the sizes of all unique adjacent islands
   - Add 1 for the current cell itself
   - Note: no island has identifier 0, so water cells (identifier 0) contribute nothing; the Python solution uses `island_sizes.get(island_id, 0)` for this

**Step 5: Return Maximum**
Track and return the maximum island size found across all possibilities.

**Time Complexity**: `O(n²)` where `n` is the grid dimension
- First pass: DFS visits each cell once
- Second pass: Check each cell and its 4 neighbors

**Space Complexity**: `O(n²)` for the `p` array and DFS recursion stack in worst case

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach:

Consider this 3×3 grid:
```
1 0 1
1 1 0
0 0 1
```

**Step 1: Identify and Label Islands**

Starting from position (0,0), we find our first island:
- DFS from (0,0): visits (0,0) → (1,0) → (1,1)
- These cells get labeled with island ID = 1
- Island #1 has size 3

Next unvisited land cell is at (0,2):
- DFS from (0,2): only visits (0,2) 
- This cell gets labeled with island ID = 2
- Island #2 has size 1

Last unvisited land cell is at (2,2):
- DFS from (2,2): only visits (2,2)
- This cell gets labeled with island ID = 3
- Island #3 has size 1

After labeling, our `p` array looks like:
```
1 0 2
1 1 0
0 0 3
```

And our `cnt` dictionary: {1: 3, 2: 1, 3: 1}

**Step 2: Find Current Maximum**
The largest existing island has size 3 (island #1).

**Step 3: Evaluate Each Water Cell**

Check position (0,1):
- Adjacent cells: (0,0)=island#1, (0,2)=island#2, (1,1)=island#1
- Unique adjacent islands: {1, 2}
- Potential size = cnt[1] + cnt[2] + 1 = 3 + 1 + 1 = 5

Check position (1,2):
- Adjacent cells: (0,2)=island#2, (1,1)=island#1, (2,2)=island#3
- Unique adjacent islands: {1, 2, 3}
- Potential size = cnt[1] + cnt[2] + cnt[3] + 1 = 3 + 1 + 1 + 1 = 6

Check position (2,0):
- Adjacent cells: (1,0)=island#1, (2,1)=water
- Unique adjacent islands: {1}
- Potential size = cnt[1] + 1 = 3 + 1 = 4

Check position (2,1):
- Adjacent cells: (1,1)=island#1, (2,0)=water, (2,2)=island#3
- Unique adjacent islands: {1, 3}
- Potential size = cnt[1] + cnt[3] + 1 = 3 + 1 + 1 = 5

**Step 4: Return Maximum**
The maximum achievable island size is 6, obtained by changing position (1,2) from 0 to 1, which would connect all three existing islands.

## Python Solution

```python
import sys

# The DFS can be up to n * n = 250000 calls deep, above Python's default limit of 1000
sys.setrecursionlimit(300000)

class Solution:
    def largestIsland(self, grid: List[List[int]]) -> int:
        def dfs(row: int, col: int) -> None:
            """
            Depth-first search to mark all cells in current island with island_id
            and count the size of the island
            """
            island_assignment[row][col] = island_id
            island_sizes[island_id] += 1
          
            # Check all 4 adjacent cells
            for direction_idx in range(4):
                next_row = row + directions[direction_idx]
                next_col = col + directions[direction_idx + 1]
              
                # Check if next cell is valid, is land, and not yet visited
                if (0 <= next_row < n and 0 <= next_col < n and 
                    grid[next_row][next_col] == 1 and 
                    island_assignment[next_row][next_col] == 0):
                    dfs(next_row, next_col)
      
        n = len(grid)
        island_sizes = {}  # Maps island_id to its size
        island_assignment = [[0] * n for _ in range(n)]  # Tracks which island each cell belongs to
        directions = [-1, 0, 1, 0, -1]  # Used for getting 4 directions: up, right, down, left
        island_id = 0
      
        # First pass: identify all islands and calculate their sizes
        for i in range(n):
            for j in range(n):
                if grid[i][j] == 1 and island_assignment[i][j] == 0:
                    island_id += 1
                    island_sizes[island_id] = 0
                    dfs(i, j)
      
        # Initialize answer with the size of the largest existing island
        max_island_size = max(island_sizes.values()) if island_sizes else 0
      
        # Second pass: try flipping each water cell to land and calculate resulting island size
        for i in range(n):
            for j in range(n):
                if grid[i][j] == 0:
                    # Collect unique adjacent islands
                    adjacent_islands = set()
                    for direction_idx in range(4):
                        adjacent_row = i + directions[direction_idx]
                        adjacent_col = j + directions[direction_idx + 1]
                      
                        if 0 <= adjacent_row < n and 0 <= adjacent_col < n:
                            adjacent_islands.add(island_assignment[adjacent_row][adjacent_col])
                  
                    # Calculate total size if we flip this water cell to land
                    # Add 1 for the flipped cell itself, plus sizes of all connected islands
                    connected_size = 1 + sum(island_sizes.get(island_id, 0) 
                                            for island_id in adjacent_islands)
                    max_island_size = max(max_island_size, connected_size)
      
        return max_island_size
```

## Time and Space Complexity

**Time Complexity:** `O(n²)`

The algorithm consists of two main phases:

1. **Island Identification Phase:** The code iterates through all `n²` cells in the grid. For each unvisited land cell (where `grid[i][j] == 1` and `island_assignment[i][j] == 0`), it performs a DFS traversal. Each cell is visited at most once during all DFS operations combined, as once a cell is marked with a root value in `island_assignment`, it won't be visited again. Therefore, this phase takes `O(n²)` time.

2. **Maximum Island Calculation Phase:** The code again iterates through all `n²` cells. For each water cell (where `grid[i][j] == 0`), it checks up to 4 adjacent cells and builds a set of unique root values. The operations of checking neighbors, adding to a set, and summing the counts are all `O(1)` operations (the set size is at most 4). Therefore, this phase also takes `O(n²)` time.

The overall time complexity is `O(n²) + O(n²) = O(n²)`.

**Space Complexity:** `O(n²)`

The space usage includes:
- `island_assignment`: A 2D array of size `n × n` storing root values for each cell, requiring `O(n²)` space
- `island_sizes`: In the worst case where the entire grid is land forming one island, it stores one entry, but considering multiple islands, it can store up to `O(n²)` entries (though typically much less)
- DFS recursion stack: In the worst case, the recursion depth can be `O(n²)` if the entire grid forms a single snake-like island
- Local variables and the temporary set `adjacent_islands` use `O(1)` space

The dominant factor is the 2D array `island_assignment`, giving us a space complexity of `O(n²)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Counting the Same Island Multiple Times
**Problem**: When checking adjacent cells of a water cell (0), if multiple adjacent cells belong to the same island, you might accidentally count that island's size multiple times.

**Example**: Consider this grid:
```
1 1
1 0
```
If we try to flip the cell at position (1,1), both (0,1) and (1,0) belong to the same island of size 3. Without using a set to track unique islands, we'd count this island twice, incorrectly calculating the size as 3 + 3 + 1 = 7 instead of 3 + 1 = 4.

**Solution**: Use a set to store unique island identifiers when checking adjacent cells:
```python
adjacent_islands = set()
for direction_idx in range(4):
    # ... check bounds ...
    adjacent_islands.add(island_assignment[adjacent_row][adjacent_col])
```

### Pitfall 2: Forgetting the Case Where No Flip is Needed
**Problem**: The grid might already contain an island that's larger than any island you could create by flipping a 0 to 1. If you only check water cells, you might miss this case.

**Example**: Grid with all 1s:
```
1 1
1 1
```
There are no 0s to flip, so if you only calculate sizes when flipping 0s, you'd return 0 or get an error.

**Solution**: Initialize the answer with the largest existing island size before checking water cells:
```python
max_island_size = max(island_sizes.values()) if island_sizes else 0
```

### Pitfall 3: Stack Overflow with Large Islands
**Problem**: Using recursive DFS on very large grids with large islands can cause stack overflow due to Python's recursion limit.

**Example**: A 500x500 grid filled with 1s (allowed by the constraints) makes the recursion about 250,000 calls deep. Without a change, the recursive `dfs` raises `RecursionError` on this input under Python's default limit of 1000. The solution above calls `sys.setrecursionlimit(300000)` for this reason; the iterative version below avoids deep recursion entirely.

**Solution**: Use iterative DFS with an explicit stack instead:
```python
def dfs(start_row: int, start_col: int) -> None:
    stack = [(start_row, start_col)]
    while stack:
        row, col = stack.pop()
        if (row < 0 or row >= n or col < 0 or col >= n or 
            grid[row][col] == 0 or island_assignment[row][col] != 0):
            continue
        island_assignment[row][col] = island_id
        island_sizes[island_id] += 1
        for direction_idx in range(4):
            next_row = row + directions[direction_idx]
            next_col = col + directions[direction_idx + 1]
            stack.append((next_row, next_col))
```

### Pitfall 4: Incorrect Handling of Water Cells in Adjacent Islands Set
**Problem**: When collecting adjacent islands, water cells (with island_id = 0) might be added to the set, but they shouldn't contribute to the size calculation.

**Example**: If a water cell is surrounded by other water cells and one island, the set might contain {0, island_id}, and if not handled properly, might cause issues.

**Solution**: Either filter out 0s when calculating the sum, or ensure island_sizes doesn't contain an entry for 0:
```python
connected_size = 1 + sum(island_sizes.get(island_id, 0) 
                        for island_id in adjacent_islands if island_id != 0)
```
Or rely on the fact that `island_sizes.get(0, 0)` returns 0, which doesn't affect the sum.

