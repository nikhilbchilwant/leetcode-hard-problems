# 305. Number of Islands II

> Source: https://algo.monster/liteproblems/305

## Problem Statement

You start with an `m x n` grid that represents a map where all cells are initially water (represented by `0`). You can perform operations to add land (represented by `1`) to specific positions on this map.

Given an array `positions` where `positions[i] = [ri, ci]` represents the coordinates where you want to add land in the i-th operation, you need to track how many islands exist after each operation.

An island is defined as a group of adjacent land cells (`1`s) that are connected horizontally or vertically. Land cells that are only diagonally adjacent do not form the same island. The entire grid is surrounded by water.

Your task is to return an array `answer` where `answer[i]` represents the total number of islands present in the grid after performing the i-th operation (adding land at position `[ri, ci]`).

For example:
- If you add land to an isolated water cell, you create a new island (island count increases by 1)
- If you add land adjacent to existing land, you extend that island (island count stays the same)
- If you add land between two or more separate islands, you connect them into one island (island count decreases)
- If you try to add land to a position that's already land, nothing changes (island count stays the same)

The solution uses a Union-Find data structure to efficiently track and merge islands as land is added. Each position in the grid is mapped to a unique index `i * n + j` for use in the union-find structure. When adding new land, the algorithm checks all four adjacent cells and merges connected components as needed, updating the island count accordingly.

## Intuition

The key insight is recognizing that this is a dynamic connectivity problem. As we add land pieces one by one, we need to efficiently track which land cells belong to the same island and update the total island count.

Initially, when we add a land cell to water, we create a new island. But the complexity arises when this new land cell is adjacent to existing land cells. If it's next to one existing island, it simply becomes part of that island. If it's adjacent to multiple separate islands, it acts as a bridge that merges them into one.

Think of it like dropping stones into water - each stone initially forms its own "island". But if you drop a stone next to an existing stone, they form one larger island. If you drop a stone between two separate groups of stones, all three merge into one island.

The challenge is efficiently tracking these merging operations. A naive approach would be to run a full island counting algorithm (like DFS or BFS) after each land addition, but this would be too slow for large grids with many operations.

This naturally leads us to Union-Find (Disjoint Set Union), a data structure specifically designed for tracking connected components that can merge over time. Each land cell can be treated as a node, and adjacent land cells should belong to the same set (island).

When we add new land:
1. We initially increment the island count (assuming it's a new island)
2. We check all four adjacent cells
3. For each adjacent land cell that belongs to a different island (different root in Union-Find), we merge them and decrement the island count
4. If the adjacent land already belongs to the same island as our new cell, no count change is needed

The Union-Find structure with path compression and union by size ensures near-constant time operations, making this approach efficient even for large grids with many position updates.

### Watch it run

Step through the additions below. Each new cell starts its own island, then merges with any land it touches. Cells sharing a colour belong to the same component.

<iframe src="/widgets/islands-ii-union-find.html" width="100%" height="580" frameborder="0" style="border:1px solid #e5e7eb;border-radius:12px;width:100%;min-height:160px;touch-action:pan-y;" title="Number of Islands II — union-find walkthrough"></iframe>

## Solution Approach

The solution uses a Union-Find data structure to efficiently track and merge islands. Let's walk through the implementation step by step:

**1. Union-Find Data Structure Setup**

We create a `UnionFind` class that maintains:
- `p`: parent array where `p[x]` stores the parent of node `x`
- `size`: array tracking the size of each component
- `find(x)`: finds the root of the component containing `x`, with path compression for optimization
- `union(a, b)`: merges two components and returns `True` if they were previously separate, `False` if already connected

**2. Grid Initialization**

- Create a 2D `grid` of size `m x n`, initially all zeros (water)
- Initialize `cnt = 0` to track the current number of islands
- Create a Union-Find instance with `m * n` nodes (one for each grid cell)

**3. Position Mapping**

Each grid position `(i, j)` is mapped to a unique index using the formula `i * n + j`. This allows us to use a 1D Union-Find structure for a 2D grid.

**4. Processing Each Position**

For each position `(i, j)` in `positions`:

- **Check if already land**: If `grid[i][j] == 1`, this position is already land, so append current `cnt` to answer and continue

- **Add new land**:
  - Set `grid[i][j] = 1`
  - Increment `cnt` by 1 (initially assume it's a new island)

- **Check adjacent cells**: Use `dirs = (-1, 0, 1, 0, -1)` with `pairwise` to generate the four directions:
  - Calculate adjacent position: `(x, y) = (i + a, j + b)`
  - If the adjacent cell is:
    - Within bounds: `0 <= x < m` and `0 <= y < n`
    - Is land: `grid[x][y] == 1`
    - Not yet connected: `uf.union(i * n + j, x * n + y)` returns `True`
  - Then decrement `cnt` by 1 (two islands merge into one)

- **Record result**: Append current `cnt` to the answer array

**5. Union Operation Details**

The `union` operation in the Union-Find:
- Finds roots of both components using `find`
- If they share the same root, returns `False` (already connected)
- Otherwise, merges the smaller component into the larger one (union by size optimization)
- Returns `True` to indicate a successful merge

The time complexity is `O(k × α(m × n))` where `k` is the number of positions and `α` is the inverse Ackermann function (practically constant). The space complexity is `O(m × n)` for the grid and Union-Find structure.

## Example Walkthrough

<details>
<summary>Prefer text? Read the step-by-step walkthrough.</summary>

Let's walk through a small example with a 3×3 grid and positions = [[0,0], [0,1], [1,1], [2,1], [1,0]].

**Initial State:**
```
Grid:        Islands: 0
. . .
. . .
. . .
```

**Operation 1: Add land at [0,0]**
- Grid[0][0] = 0 (water), so we add land
- Increment island count: 0 → 1
- Check 4 neighbors: all are water
- No unions needed
- Result: 1 island

```
Grid:        Islands: 1
1 . .
. . .
. . .
```

**Operation 2: Add land at [0,1]**
- Grid[0][1] = 0 (water), so we add land
- Increment island count: 1 → 2
- Check neighbors:
  - Left [0,0] is land! Union(1, 0) returns True
  - Decrement count: 2 → 1
- Result: 1 island (merged with existing)

```
Grid:        Islands: 1
1 1 .
. . .
. . .
```

**Operation 3: Add land at [1,1]**
- Grid[1][1] = 0 (water), so we add land
- Increment island count: 1 → 2
- Check neighbors:
  - Up [0,1] is land! Union(4, 1) returns True
  - Decrement count: 2 → 1
- Result: 1 island (extended the existing island)

```
Grid:        Islands: 1
1 1 .
. 1 .
. . .
```

**Operation 4: Add land at [2,1]**
- Grid[2][1] = 0 (water), so we add land
- Increment island count: 1 → 2
- Check neighbors:
  - Up [1,1] is land! Union(7, 4) returns True
  - Decrement count: 2 → 1
- Result: 1 island (still one connected island)

```
Grid:        Islands: 1
1 1 .
. 1 .
. 1 .
```

**Operation 5: Add land at [1,0]**
- Grid[1][0] = 0 (water), so we add land
- Increment island count: 1 → 2
- Check neighbors:
  - Up [0,0] is land! Union(3, 0) returns True
  - Decrement count: 2 → 1
  - Right [1,1] is land! Union(3, 4) returns False (already connected through [0,0]→[0,1]→[1,1])
  - No decrement since they're already in the same component
- Result: 1 island

```
Grid:        Islands: 1
1 1 .
1 1 .
. 1 .
```

**Final Answer: [1, 1, 1, 1, 1]**

The key insight is how Union-Find efficiently tracks connectivity. When we try to union [1,0] with [1,1] in the last step, they're already connected through the path [1,0]→[0,0]→[0,1]→[1,1], so union returns False and we don't double-count the merge.

</details>

## Python Solution

```python
class UnionFind:
    """Union-Find (Disjoint Set Union) data structure with path compression and union by size."""

    def __init__(self, n: int):
        """Initialize Union-Find with n elements (0 to n-1)."""
        self.parent = list(range(n))  # Each element is initially its own parent
        self.size = [1] * n  # Size of each component initially 1

    def find(self, x: int) -> int:
        """Find the root of element x with path compression."""
        if self.parent[x] != x:
            # Path compression: make x point directly to root
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, a: int, b: int) -> bool:
        """
        Union two elements a and b.
        Returns True if they were in different components, False otherwise.
        """
        root_a = self.find(a)
        root_b = self.find(b)

        # Already in the same component
        if root_a == root_b:
            return False

        # Union by size: attach smaller tree under larger tree
        if self.size[root_a] > self.size[root_b]:
            self.parent[root_b] = root_a
            self.size[root_a] += self.size[root_b]
        else:
            self.parent[root_a] = root_b
            self.size[root_b] += self.size[root_a]

        return True


class Solution:
    def numIslands2(self, m: int, n: int, positions: List[List[int]]) -> List[int]:
        """
        Given a m x n grid initially filled with water, return the number of islands
        after each land addition from positions.

        Args:
            m: Number of rows in the grid
            n: Number of columns in the grid
            positions: List of [row, col] positions where land is added

        Returns:
            List containing number of islands after each addition
        """
        # Initialize Union-Find for m*n grid cells
        uf = UnionFind(m * n)

        # Track which cells are land (1) or water (0)
        grid = [[0] * n for _ in range(m)]

        # Result list to store island count after each addition
        result = []

        # Direction vectors for 4-directional movement (up, right, down, left)
        directions = [(-1, 0), (0, 1), (1, 0), (0, -1)]

        # Current number of islands
        island_count = 0

        for row, col in positions:
            # If this position is already land, no change in island count
            if grid[row][col] == 1:
                result.append(island_count)
                continue

            # Mark this position as land
            grid[row][col] = 1
            # Initially, this creates a new island
            island_count += 1

            # Check all 4 adjacent cells
            for dx, dy in directions:
                new_row, new_col = row + dx, col + dy

                # Check if adjacent cell is valid and is land
                if (0 <= new_row < m and
                    0 <= new_col < n and
                    grid[new_row][new_col] == 1):

                    # Convert 2D coordinates to 1D index for Union-Find
                    current_idx = row * n + col
                    adjacent_idx = new_row * n + new_col

                    # If union is successful (they were in different components),
                    # we've merged two islands into one
                    if uf.union(current_idx, adjacent_idx):
                        island_count -= 1

            result.append(island_count)

        return result
```

## Time and Space Complexity

**Time Complexity:** `O(k × α(m × n))` where `k` is the length of `positions` and `α` is the inverse Ackermann function.

- For each position in `positions` (k iterations total):
  - Checking if the cell is already land: `O(1)`
  - Adding the land and incrementing count: `O(1)`
  - Checking up to 4 adjacent cells and performing union operations if needed
  - Each union operation involves two `find` operations and the actual union
  - With path compression and union by size, `find` operations take `O(α(m × n))` amortized time
  - The `union` operation itself (after finding roots) takes `O(1)`
  - Since we check at most 4 neighbors, this is `O(4 × α(m × n))` = `O(α(m × n))`

The inverse Ackermann function `α(m × n)` grows extremely slowly and can be considered effectively constant for all practical values of `m × n` (it's less than 5 for any reasonable input size).

**Space Complexity:** `O(m × n)`

- The UnionFind data structure maintains:
  - Parent array `p`: `O(m × n)`
  - Size array `size`: `O(m × n)`
- The `grid` matrix to track land cells: `O(m × n)`
- The `ans` list to store results: `O(k)`
- Recursive call stack for `find` operation: `O(log(m × n))` in worst case before path compression, but effectively `O(α(m × n))` with path compression

Overall space complexity is dominated by `O(m × n)` for the UnionFind structure and grid.

## Common Pitfalls

### 1. **Duplicate Position Handling**
One of the most common mistakes is not properly handling duplicate positions in the input. If the same position `[r, c]` appears multiple times in the `positions` array, adding land to an already-land cell should not change the island count.

**Incorrect approach:**
```python
for row, col in positions:
    grid[row][col] = 1
    island_count += 1
    # Check neighbors and union...
```

**Correct approach:**
```python
for row, col in positions:
    if grid[row][col] == 1:  # Already land
        result.append(island_count)
        continue
    grid[row][col] = 1
    island_count += 1
    # Check neighbors and union...
```

### 2. **Incorrect Index Mapping for Union-Find**
When converting 2D grid coordinates to 1D Union-Find indices, using the wrong formula can cause incorrect unions between unrelated cells.

**Common mistakes:**
- Using `i + j * m` instead of `i * n + j`
- Mixing up row/column dimensions
- Off-by-one errors when the Union-Find is 1-indexed

**Correct formula:**
```python
# For 0-indexed Union-Find
current_idx = row * n + col  # n is number of columns
adjacent_idx = new_row * n + new_col
```

### 3. **Forgetting to Decrement Island Count After Union**
When two separate islands are connected by adding new land, the total island count should decrease. A common mistake is to only increment the count when adding new land but forget to decrement when merging islands.

**Incorrect:**
```python
island_count += 1  # New land added
if uf.union(current_idx, adjacent_idx):
    # Forgot to decrement!
    pass
```

**Correct:**
```python
island_count += 1  # New land added
if uf.union(current_idx, adjacent_idx):
    island_count -= 1  # Two islands merged into one
```

### 4. **Union-Find Return Value Misunderstanding**
The `union` method should return `True` only when two previously separate components are merged, not just when the union operation is called.

**Incorrect union implementation:**
```python
def union(self, a: int, b: int) -> bool:
    root_a = self.find(a)
    root_b = self.find(b)
    self.parent[root_b] = root_a
    return True  # Always returns True!
```

**Correct implementation:**
```python
def union(self, a: int, b: int) -> bool:
    root_a = self.find(a)
    root_b = self.find(b)
    if root_a == root_b:
        return False  # Already connected
    # Perform union...
    return True  # Successfully merged two components
```

### 5. **Boundary Checking Errors**
Failing to properly check grid boundaries when examining adjacent cells can lead to index out of bounds errors or incorrect connections.

**Common boundary check mistakes:**
- Using `<= m` instead of `< m`
- Checking only one dimension
- Wrong order of conditions causing short-circuit evaluation issues

**Correct boundary check:**
```python
if (0 <= new_row < m and
    0 <= new_col < n and
    grid[new_row][new_col] == 1):
    # Process valid land neighbor
```

