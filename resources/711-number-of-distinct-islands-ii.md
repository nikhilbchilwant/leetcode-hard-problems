# 711. Number of Distinct Islands II

> Source: https://algo.monster/liteproblems/711

## Problem Statement

You have an `m x n` binary matrix `grid` where `1` represents land and `0` represents water. An island is formed by connecting adjacent lands horizontally or vertically (4-directionally connected). The entire grid is surrounded by water.

The key challenge is identifying **distinct** islands. Two islands are considered the same if:
- They have the exact same shape, OR
- One can be transformed into the other through:
  - **Rotation**: 90°, 180°, or 270° rotations
  - **Reflection**: flipping horizontally (left/right) or vertically (up/down)

Your task is to count how many distinct island shapes exist in the grid.

For example, if you have an L-shaped island and another island that looks like a rotated L, they count as the same shape (only 1 distinct island). Similarly, if one island is a mirror image of another, they're also considered the same.

## Intuition

When we look at this problem, the first challenge is understanding what makes two islands "the same." If we only cared about exact matches, we could simply record each island's shape and compare them directly. But since islands can be rotated or reflected and still be considered identical, we need a way to handle all these transformations.

Think about it like puzzle pieces - if you have two L-shaped pieces, one might be rotated 90° from the other, but they're still the same piece. The key insight is that for any given shape, there are exactly 8 possible orientations when considering all rotations and reflections:
- 4 rotations (0°, 90°, 180°, 270°)
- Each rotation can also be reflected (flipped)

So our strategy becomes: for each island we find, generate all 8 possible transformations of its shape. But we still need a consistent way to compare shapes. This is where **normalization** comes in.

The normalization process works by:
1. Taking all 8 transformations of an island's coordinates
2. Sorting each transformation to create a consistent ordering
3. Translating each shape so its "first" point is at origin `(0, 0)` - this makes the shape position-independent
4. Choosing the lexicographically smallest transformation as the "canonical" representation

By using DFS to explore each island and collect its coordinates, then normalizing these coordinates to a canonical form, we can use a set to automatically handle duplicates. Two islands that are the same shape (considering rotations and reflections) will have the same canonical form and thus be counted only once.

The beauty of this approach is that we don't need to explicitly check if two islands match through various transformations - the normalization process handles all of that automatically by reducing every possible variant of a shape to a single, unique representation.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro) and [Union Find](/problems/dsu_intro) patterns.**

## Solution Approach

The implementation uses DFS combined with a normalization technique to identify distinct islands. Let's walk through the key components:

**1. DFS Exploration (`dfs` function)**

The DFS function explores an entire island starting from a given cell `(i, j)`:
- Records the current cell's coordinates in the `shape` list
- Marks the cell as visited by setting `grid[i][j] = 0`
- Recursively explores all 4 adjacent cells (up, down, left, right)
- Builds a complete list of coordinates representing the island's shape

```python
def dfs(i, j, shape):
    shape.append([i, j])
    grid[i][j] = 0
    for a, b in [[1, 0], [-1, 0], [0, 1], [0, -1]]:
        x, y = i + a, j + b
        if 0 <= x < m and 0 <= y < n and grid[x][y] == 1:
            dfs(x, y, shape)
```

**2. Shape Normalization (`normalize` function)**

This is the core logic that handles rotations and reflections. For each island shape, it generates all 8 possible transformations:

```python
shapes[0].append([i, j])      # Original
shapes[1].append([i, -j])     # Reflect left-right
shapes[2].append([-i, j])     # Reflect up-down
shapes[3].append([-i, -j])    # Rotate 180°
shapes[4].append([j, i])      # Reflect over main diagonal
shapes[5].append([j, -i])     # Rotate 90°
shapes[6].append([-j, i])     # Rotate 270°
shapes[7].append([-j, -i])    # Reflect over anti-diagonal
```

After generating all transformations:
- Each transformation is sorted to create consistent ordering
- Each shape is translated so its first point becomes `(0, 0)`:
  ```python
  for i in range(len(e) - 1, -1, -1):
      e[i][0] -= e[0][0]
      e[i][1] -= e[0][1]
  ```
- The lexicographically smallest transformation is chosen as the canonical form
- The result is converted to a tuple (immutable and hashable) for set storage

**3. Main Algorithm**

The main loop:
- Iterates through every cell in the grid
- When a land cell (`1`) is found, initiates DFS to explore the entire island
- Normalizes the island's shape to get its canonical representation
- Adds the canonical form to a set (automatically handles duplicates)
- Returns the size of the set, which gives the count of distinct islands

```python
s = set()
for i in range(m):
    for j in range(n):
        if grid[i][j]:
            shape = []
            dfs(i, j, shape)
            s.add(normalize(shape))
return len(s)
```

The time complexity is `O(m × n × log(m × n))`, because normalizing an island of size `k` sorts 8 lists of `k` points, and the island sizes add up to at most `m × n`. The space complexity is `O(m × n)` for storing island shapes and the set of canonical forms.

## Example Walkthrough

Let's walk through a small example to illustrate how the solution identifies distinct islands:

```
Grid:
1 1 0 0 0
1 0 0 0 0
0 0 0 1 1
0 0 0 0 1
```

**Step 1: Find First Island**
Starting at (0,0), we encounter a '1'. DFS explores the island:
- Visit (0,0), add to shape: [(0,0)]
- Visit (1,0) (down is checked first), add to shape: [(0,0), (1,0)]
- Back at (0,0), visit (0,1), add to shape: [(0,0), (1,0), (0,1)]
- Mark all visited cells as 0

Island 1 coordinates: [(0,0), (1,0), (0,1)] - This is an L-shape.

**Step 2: Normalize First Island**
Apply all 8 transformations to [(0,0), (1,0), (0,1)], then sort each one and translate it so its first point is (0,0):
1. Original `(r, c)`: [(0,0), (0,1), (1,0)]
2. Reflect left-right `(r, -c)`: [(0,0), (0,1), (1,1)]
3. Reflect up-down `(-r, c)`: [(0,0), (1,0), (1,1)]
4. Rotate 180° `(-r, -c)`: [(0,0), (1,-1), (1,0)]
5. Reflect over main diagonal `(c, r)`: [(0,0), (0,1), (1,0)]
6. Rotate 90° `(c, -r)`: [(0,0), (0,1), (1,1)]
7. Rotate 270° `(-c, r)`: [(0,0), (1,0), (1,1)]
8. Reflect over anti-diagonal `(-c, -r)`: [(0,0), (1,-1), (1,0)]

Choose lexicographically smallest: `((0,0), (0,1), (1,0))`
Add to set: {((0,0), (0,1), (1,0))}

**Step 3: Find Second Island**
Starting at (2,3), we find another island:
- Visit (2,3), add to shape: [(2,3)]
- Visit (2,4), add to shape: [(2,3), (2,4)]
- Visit (3,4), add to shape: [(2,3), (2,4), (3,4)]

Island 2 coordinates: [(2,3), (2,4), (3,4)] - This is also an L-shape, but rotated!

**Step 4: Normalize Second Island**
Original coordinates: [(2,3), (2,4), (3,4)]

First, let's look at what happens with just the relative positions (ignoring the actual positions):
- Relative to first point: [(0,0), (0,1), (1,1)]

Apply all 8 transformations, then sort and translate each one:
1. Original: [(0,0), (0,1), (1,1)]
2. Reflect left-right: [(0,0), (0,1), (1,0)]
3. Reflect up-down: [(0,0), (1,-1), (1,0)]
4. Rotate 180°: [(0,0), (1,0), (1,1)]
5. Reflect over main diagonal: [(0,0), (1,0), (1,1)]
6. Rotate 90°: [(0,0), (1,-1), (1,0)]
7. Rotate 270°: [(0,0), (0,1), (1,0)]
8. Reflect over anti-diagonal: [(0,0), (0,1), (1,1)]

The smallest of these is [(0,0), (0,1), (1,0)], produced by the left-right reflection and by the 270° rotation.

This canonical form `((0,0), (0,1), (1,0))` already exists in our set!

**Step 5: Final Result**
Set remains: {((0,0), (0,1), (1,0))}
Count of distinct islands: **1**

Even though we found two islands in different orientations, the normalization process correctly identified them as the same shape, giving us the correct count of 1 distinct island.

## Python Solution

```python
class Solution:
    def numDistinctIslands2(self, grid: List[List[int]]) -> int:
        def dfs(row: int, col: int, island_cells: List[List[int]]) -> None:
            """
            Depth-first search to explore an island and collect all its cells.
            Marks visited cells as 0 to avoid revisiting.
            """
            # Add current cell to the island shape
            island_cells.append([row, col])
            # Mark as visited
            grid[row][col] = 0

            # Explore all 4 directions
            directions = [[1, 0], [-1, 0], [0, 1], [0, -1]]
            for delta_row, delta_col in directions:
                next_row = row + delta_row
                next_col = col + delta_col
                # Check bounds and if cell is part of island
                if (0 <= next_row < rows and
                    0 <= next_col < cols and
                    grid[next_row][next_col] == 1):
                    dfs(next_row, next_col, island_cells)

        def normalize(island_cells: List[List[int]]) -> Tuple[Tuple[int, ...]]:
            """
            Generate all 8 possible transformations (rotations and reflections)
            of an island and return the lexicographically smallest one as canonical form.
            """
            # Generate all 8 transformations (4 rotations × 2 reflections)
            transformations = [[] for _ in range(8)]

            for row, col in island_cells:
                # Original and reflections
                transformations[0].append([row, col])      # Original
                transformations[1].append([row, -col])     # Reflect left-right
                transformations[2].append([-row, col])     # Reflect up-down
                transformations[3].append([-row, -col])    # Rotate 180°
                # 90° rotation and its reflections
                transformations[4].append([col, row])      # Reflect over main diagonal
                transformations[5].append([col, -row])     # Rotate 90°
                transformations[6].append([-col, row])     # Rotate 270°
                transformations[7].append([-col, -row])    # Reflect over anti-diagonal

            # Normalize each transformation by translating to origin
            for transformation in transformations:
                # Sort cells to ensure consistent ordering
                transformation.sort()
                # Translate so that first cell is at origin (0, 0)
                base_row = transformation[0][0]
                base_col = transformation[0][1]
                for idx in range(len(transformation) - 1, -1, -1):
                    transformation[idx][0] -= base_row
                    transformation[idx][1] -= base_col

            # Sort all transformations and pick the lexicographically smallest
            transformations.sort()

            # Convert to tuple for hashability
            return tuple(tuple(cell) for cell in transformations[0])

        # Initialize grid dimensions
        rows, cols = len(grid), len(grid[0])

        # Set to store unique island shapes
        unique_islands = set()

        # Explore the entire grid
        for row in range(rows):
            for col in range(cols):
                if grid[row][col] == 1:
                    # Found an unvisited island
                    island_cells = []
                    dfs(row, col, island_cells)
                    # Normalize and add to set of unique islands
                    canonical_form = normalize(island_cells)
                    unique_islands.add(canonical_form)

        return len(unique_islands)
```

## Time and Space Complexity

**Time Complexity:** `O(m * n * log(m * n))` where `m` and `n` are the dimensions of the grid.

- The outer loops iterate through all cells in the grid: `O(m * n)`
- For each cell that starts an island, we perform DFS which visits all cells in that island: `O(k)` where `k` is the island size
- The `normalize` function is called for each island:
  - Creating 8 transformations: `O(8 * k) = O(k)`
  - Sorting each of the 8 transformations: `O(8 * k * log(k)) = O(k * log(k))`
  - Normalizing coordinates by subtracting the first element: `O(8 * k) = O(k)`
  - Sorting the 8 transformations to find the canonical form: `O(8 * log(8) * k) = O(k)` (comparing lists of size `k`)
  - Converting to tuple: `O(k)`
  - Overall normalize: `O(k * log(k))`
- Every land cell belongs to exactly one island, so the island sizes add up to at most `m * n`. Summing `O(k * log(k))` over all islands gives at most `O(m * n * log(m * n))`
- Adding the `O(m * n)` grid scan and DFS work, the total is `O(m * n * log(m * n))`

**Space Complexity:** `O(m * n)`

- The `island_cells` list stores coordinates for one island at a time: `O(k)` where `k` is the island size
- The `transformations` list in normalize stores 8 transformations: `O(8 * k) = O(k)`
- The set `unique_islands` stores unique normalized island shapes. Each stored shape has the size of its island, and the island sizes add up to at most `m * n`: `O(m * n)`
- The recursion stack depth for DFS can be at most `O(m * n)` in the worst case of a snake-like island
- Overall space complexity: `O(m * n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Modifying the Input Grid Without Restoration**

The current solution modifies the input grid by setting visited cells to 0. This destructive approach can be problematic if:
- The grid needs to be preserved for other operations
- Multiple test cases run on the same grid instance
- The problem explicitly states not to modify the input

**Solution:**
Use a separate visited set to track explored cells instead of modifying the grid:

```python
def dfs(row: int, col: int, island_cells: List[List[int]], visited: Set[Tuple[int, int]]) -> None:
    island_cells.append([row, col])
    visited.add((row, col))

    for delta_row, delta_col in [[1, 0], [-1, 0], [0, 1], [0, -1]]:
        next_row = row + delta_row
        next_col = col + delta_col
        if (0 <= next_row < rows and
            0 <= next_col < cols and
            grid[next_row][next_col] == 1 and
            (next_row, next_col) not in visited):
            dfs(next_row, next_col, island_cells, visited)

# In main loop:
visited = set()
for row in range(rows):
    for col in range(cols):
        if grid[row][col] == 1 and (row, col) not in visited:
            island_cells = []
            dfs(row, col, island_cells, visited)
            unique_islands.add(normalize(island_cells))
```

### 2. **Incorrect Transformation Logic**

A subtle but critical pitfall is incorrectly implementing the 8 transformations. The transformations must correctly represent all possible rotations and reflections:

**Common mistakes:**
- Missing some transformations (e.g., only doing 4 instead of 8)
- Incorrect rotation formulas (mixing up clockwise vs counter-clockwise)
- Not handling both horizontal and vertical reflections

**Solution:**
Verify transformations with a simple test case. For a point (x, y):
- 90° CCW rotation: (-y, x)
- 90° CW rotation: (y, -x)
- 180° rotation: (-x, -y)
- Horizontal reflection: (x, -y)
- Vertical reflection: (-x, y)

### 3. **Inefficient Normalization Process**

The current implementation sorts transformations multiple times and creates many intermediate lists, which can be memory-intensive for large islands.

**Solution:**
Optimize by finding the canonical form more efficiently:

```python
def normalize(island_cells: List[List[int]]) -> Tuple[Tuple[int, ...]]:
    def get_canonical(cells):
        # Sort and translate to origin in one pass
        cells = sorted(cells)
        if not cells:
            return tuple()
        base_row, base_col = cells[0]
        return tuple((r - base_row, c - base_col) for r, c in cells)

    # Generate and compare transformations without storing all of them
    min_shape = None
    for transform_func in [
        lambda r, c: (r, c),      # Original
        lambda r, c: (r, -c),     # Reflect left-right
        lambda r, c: (-r, c),     # Reflect up-down
        lambda r, c: (-r, -c),    # Rotate 180°
        lambda r, c: (c, r),      # Reflect over main diagonal
        lambda r, c: (c, -r),     # Rotate 90°
        lambda r, c: (-c, r),     # Rotate 270°
        lambda r, c: (-c, -r),    # Reflect over anti-diagonal
    ]:
        transformed = [transform_func(r, c) for r, c in island_cells]
        canonical = get_canonical(transformed)
        if min_shape is None or canonical < min_shape:
            min_shape = canonical

    return min_shape
```

### 4. **Stack Overflow with Large Islands**

For very large islands, the recursive DFS can cause stack overflow.

**Solution:**
Use iterative DFS with an explicit stack:

```python
def dfs_iterative(start_row: int, start_col: int) -> List[List[int]]:
    island_cells = []
    stack = [(start_row, start_col)]
    visited.add((start_row, start_col))

    while stack:
        row, col = stack.pop()
        island_cells.append([row, col])

        for delta_row, delta_col in [[1, 0], [-1, 0], [0, 1], [0, -1]]:
            next_row = row + delta_row
            next_col = col + delta_col
            if (0 <= next_row < rows and
                0 <= next_col < cols and
                grid[next_row][next_col] == 1 and
                (next_row, next_col) not in visited):
                visited.add((next_row, next_col))
                stack.append((next_row, next_col))

    return island_cells
```

