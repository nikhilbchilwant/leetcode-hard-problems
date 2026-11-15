# 803. Bricks Falling When Hit

> Source: https://algo.monster/liteproblems/803

## Problem Statement

You have a grid of bricks represented as an `m x n` binary matrix where `1` represents a brick and `0` represents empty space. A brick remains stable (doesn't fall) if it meets one of these conditions:
- It's in the top row (directly connected to the ceiling), OR  
- At least one of its 4 adjacent neighbors (up, down, left, right) is stable

You're given a list of hits, where each `hits[i] = (row_i, col_i)` represents erasing the brick at that position. When you erase a brick:
- If there's no brick at that position, nothing happens
- If there's a brick, it disappears
- Any bricks that become unstable as a result will immediately fall and disappear (they don't land on other bricks below)

Your task is to return an array where `result[i]` represents how many bricks fall after applying the `i-th` hit.

**Key Points:**
- Bricks fall if they lose their connection to the top row (either directly or through other stable bricks)
- When bricks fall, they completely disappear from the grid
- A hit on an empty cell causes no bricks to fall
- The hits are applied sequentially, and each hit affects the grid state left by previous hits

**Example scenario:** If hitting a brick causes a cluster of bricks to lose their only connection to the top row, all bricks in that cluster will fall and be counted in the result for that hit.

## Intuition

The key insight is to think about this problem in reverse. Instead of removing bricks and watching them fall, we can start with all the hits already applied (all bricks already removed), then add the bricks back one by one in reverse order.

Why does this help? When we remove a brick, it's hard to track which bricks will fall because we need to check if they still have a path to the top. But when we add a brick back, we can easily count how many previously disconnected bricks now become connected to the top through this newly added brick.

Think of it like this: imagine a group of floating bricks that aren't connected to the ceiling. When we place a brick that bridges them to the ceiling, all those floating bricks suddenly become stable. The number of bricks that "would have fallen" when we removed this brick is exactly the number of floating bricks that became stable when we added it back.

This naturally leads us to use Union-Find (Disjoint Set Union) data structure. We can treat all bricks connected to the top row as one giant component. Here's the process:

1. Start with the final state (all hits applied)
2. Create a virtual "super root" representing the ceiling
3. Connect all top-row bricks to this super root
4. Build the initial connected components
5. Process hits in reverse order:
   - When adding a brick back, check how many bricks were in the "ceiling component" before
   - Add the brick and connect it to its neighbors
   - Check how many bricks are in the "ceiling component" after
   - The difference (minus 1 for the brick we just added) tells us how many bricks would have fallen when this brick was originally removed

The beauty of this approach is that Union-Find efficiently tracks component sizes, so we can quickly determine how many previously disconnected bricks become connected to the ceiling when we add each brick back.

**Learn more about [Union Find](/problems/dsu_intro) patterns.**

## Solution Approach

The solution implements the reverse-time Union-Find approach. Let's walk through the implementation step by step:

**1. Union-Find Setup:**
- Create a parent array `parent` of size `m * n + 1` where each cell `(i, j)` maps to index `i * n + j`
- The extra position `m * n` serves as the "super root" representing the ceiling
- Initialize the `component_size` array to track component sizes (the snippets below use the shorter names `g` for `grid_after_hits`, `size` for `component_size`, and `ans` for `result`)

**2. Apply All Hits First:**
```python
g = deepcopy(grid)
for i, j in hits:
    g[i][j] = 0
```
Create a copy of the grid and remove all bricks that will be hit, giving us the final state.

**3. Build Initial Connected Components:**
```python
# Connect top row bricks to the super root
for j in range(n):
    if g[0][j] == 1:
        union(j, m * n)

# Connect remaining bricks to their neighbors
for i in range(1, m):
    for j in range(n):
        if g[i][j] == 0:
            continue
        if g[i - 1][j] == 1:  # Connect to brick above
            union(i * n + j, (i - 1) * n + j)
        if j > 0 and g[i][j - 1] == 1:  # Connect to brick on left
            union(i * n + j, i * n + j - 1)
```
We only check up and left directions to avoid duplicate unions (the down and right neighbors will check up and left respectively).

**4. Process Hits in Reverse:**
```python
for i, j in hits[::-1]:
    if grid[i][j] == 0:  # Original position had no brick
        ans.append(0)
        continue
  
    g[i][j] = 1  # Add brick back
    prev = size[find(m * n)]  # Size of ceiling component before
  
    if i == 0:
        union(j, m * n)  # Connect to ceiling if in top row
  
    # Connect to all 4 neighbors
    for a, b in [(-1, 0), (1, 0), (0, 1), (0, -1)]:
        x, y = i + a, j + b
        if 0 <= x < m and 0 <= y < n and g[x][y] == 1:
            union(i * n + j, x * n + y)
  
    curr = size[find(m * n)]  # Size of ceiling component after
    ans.append(max(0, curr - prev - 1))
```

For each hit in reverse:
- If the original grid had no brick at this position, 0 bricks fall
- Otherwise, add the brick back and measure the change in the ceiling component size
- The number of bricks that fell = `(new size - old size - 1)`, where we subtract 1 for the brick we just added
- Use `max(0, ...)` to handle edge cases where the brick wasn't connected to ceiling

**5. Return Results in Original Order:**
```python
return ans[::-1]
```
Since we processed hits in reverse, reverse the answer array to match the original hit order.

**Time Complexity:** `O((m * n + k) * log(m * n))` amortized, where `k` is the number of hits: building the initial components touches every cell once, and each hit performs a constant number of union-find operations. The code uses path compression without union by size, which bounds each operation by `O(log(m * n))` amortized; in practice the operations are close to constant time.

**Space Complexity:** `O(m * n)` for the Union-Find structure and grid copy

## Example Walkthrough

Let's work through a small example to illustrate the solution approach.

**Initial Setup:**
```
Grid (2x3):        Hits: [(1,1), (0,1)]
[1, 1, 1]          
[0, 1, 0]          
```

**Step 1: Apply all hits to get final state**
```
After all hits:
[1, 0, 1]
[0, 0, 0]
```

**Step 2: Build Union-Find with ceiling connection**
- Create super root at index 6 (since we have 2×3 = 6 cells)
- Connect top row bricks to ceiling:
  - Cell (0,0) → index 0 → connects to 6
  - Cell (0,2) → index 2 → connects to 6
- No other bricks to connect to neighbors

Current state: Two bricks connected to ceiling through super root 6.
Component size at root 6: 3 (brick at index 0 + brick at index 2 + super root)

**Step 3: Process hits in reverse order**

**Processing Hit #2 (originally at index 1): Position (0,1)**
- Original grid had brick here (grid[0][1] = 1)
- Previous ceiling size: 3
- Add brick back at (0,1):
  ```
  [1, 1, 1]
  [0, 0, 0]
  ```
- Since it's in top row, connect to ceiling (index 1 → 6)
- Check 4 neighbors:
  - Left (0,0): has brick, union(1, 0) - already connected via ceiling
  - Right (0,2): has brick, union(1, 2) - already connected via ceiling
- New ceiling size: 4
- Bricks that fell: 4 - 3 - 1 = 0

**Processing Hit #1 (originally at index 0): Position (1,1)**
- Original grid had brick here (grid[1][1] = 1)
- Previous ceiling size: 4
- Add brick back at (1,1):
  ```
  [1, 1, 1]
  [0, 1, 0]
  ```
- Not in top row, don't connect directly to ceiling
- Check 4 neighbors:
  - Up (0,1): has brick, union(4, 1) - connects this brick to ceiling!
- New ceiling size: 5
- Bricks that fell: 5 - 4 - 1 = 0

**Step 4: Reverse the results**
Results in reverse: [0, 0]
Final answer: [0, 0]

**Why this makes sense:**
- Hit at (1,1): The brick at (1,1) was connected to ceiling through (0,1), so removing it doesn't cause any falls
- Hit at (0,1): This is a ceiling brick surrounded by other ceiling bricks, removing it doesn't disconnect anything

Let's consider a different example where bricks actually fall:

**Example 2:**
```
Grid (3x2):        Hits: [(1,0), (0,0)]
[1, 1]          
[1, 0]
[1, 0]          
```

After applying all hits:
```
[0, 1]
[0, 0]
[1, 0]
```

The brick at (2,0) is now disconnected from ceiling!

Processing in reverse:
- Add back (0,0) (the last hit): it is in the top row and joins the ceiling component together with (0,1). Cell (1,0) is still empty, so (2,0) stays disconnected. Ceiling size goes from 2 to 3 (counting the super root), so 3 - 2 - 1 = 0 bricks fell.
- Add back (1,0) (the first hit): it connects to (0,0) above and to (2,0) below, so both (1,0) and (2,0) join the ceiling component. Ceiling size goes from 3 to 5, so 5 - 3 - 1 = 1 brick fell.

Final answer: [1, 0] (reversed from [0, 1]). Hitting (1,0) first drops the brick at (2,0); hitting (0,0) afterwards drops nothing else.

This shows how the reverse approach elegantly counts falling bricks by measuring how many disconnected bricks become connected when we add each brick back.

## Python Solution

```python
class Solution:
    def hitBricks(self, grid: List[List[int]], hits: List[List[int]]) -> List[int]:
        """
        Calculate how many bricks fall after each hit using Union-Find.
        Strategy: Process hits in reverse order to simulate adding bricks back.
        """
      
        def find(node: int) -> int:
            """Find root of node with path compression."""
            if parent[node] != node:
                parent[node] = find(parent[node])
            return parent[node]
      
        def union(node_a: int, node_b: int) -> None:
            """Union two nodes by connecting their roots."""
            root_a, root_b = find(node_a), find(node_b)
            if root_a != root_b:
                # Attach root_a under root_b and add its size
                component_size[root_b] += component_size[root_a]
                parent[root_a] = root_b
      
        rows, cols = len(grid), len(grid[0])
      
        # Initialize Union-Find structure
        # Index rows * cols represents the virtual "ceiling" node
        ceiling_node = rows * cols
        parent = list(range(rows * cols + 1))
        component_size = [1] * (rows * cols + 1)
      
        # Create grid state after all hits are applied
        grid_after_hits = [row[:] for row in grid]  # Deep copy
        for row_idx, col_idx in hits:
            grid_after_hits[row_idx][col_idx] = 0
      
        # Connect all bricks in top row to ceiling
        for col_idx in range(cols):
            if grid_after_hits[0][col_idx] == 1:
                union(col_idx, ceiling_node)
      
        # Connect all remaining bricks to their neighbors
        for row_idx in range(1, rows):
            for col_idx in range(cols):
                if grid_after_hits[row_idx][col_idx] == 0:
                    continue
                  
                current_node = row_idx * cols + col_idx
              
                # Connect to brick above if exists
                if grid_after_hits[row_idx - 1][col_idx] == 1:
                    above_node = (row_idx - 1) * cols + col_idx
                    union(current_node, above_node)
              
                # Connect to brick on left if exists
                if col_idx > 0 and grid_after_hits[row_idx][col_idx - 1] == 1:
                    left_node = row_idx * cols + col_idx - 1
                    union(current_node, left_node)
      
        result = []
      
        # Process hits in reverse order (adding bricks back)
        for row_idx, col_idx in reversed(hits):
            # If original grid had no brick here, no bricks can fall
            if grid[row_idx][col_idx] == 0:
                result.append(0)
                continue
          
            # Add brick back
            grid_after_hits[row_idx][col_idx] = 1
            current_node = row_idx * cols + col_idx
          
            # Count stable bricks before adding this brick
            stable_bricks_before = component_size[find(ceiling_node)]
          
            # If brick is in top row, connect to ceiling
            if row_idx == 0:
                union(col_idx, ceiling_node)
          
            # Connect to all adjacent bricks
            directions = [(-1, 0), (1, 0), (0, 1), (0, -1)]  # up, down, right, left
            for delta_row, delta_col in directions:
                adjacent_row = row_idx + delta_row
                adjacent_col = col_idx + delta_col
              
                # Check if adjacent position is valid and has a brick
                if (0 <= adjacent_row < rows and 
                    0 <= adjacent_col < cols and 
                    grid_after_hits[adjacent_row][adjacent_col] == 1):
                    adjacent_node = adjacent_row * cols + adjacent_col
                    union(current_node, adjacent_node)
          
            # Count stable bricks after adding this brick
            stable_bricks_after = component_size[find(ceiling_node)]
          
            # Calculate fallen bricks (exclude the added brick itself)
            fallen_bricks = max(0, stable_bricks_after - stable_bricks_before - 1)
            result.append(fallen_bricks)
      
        # Reverse result to match original hit order
        return result[::-1]
```

## Time and Space Complexity

**Time Complexity:** `O((m * n + k) * log(m * n))` amortized, where `k` is the number of hits and `m` and `n` are the dimensions of the grid.

- Initial setup and copying the grid takes `O(m * n)` time
- Removing all hit bricks: `O(k)` time
- Building the initial union-find structure after removing hits: one or two `union` calls per cell, so `O(m * n)` union-find operations
- Processing hits in reverse order: for each of the `k` hits, one `find` before, at most 5 `union` calls (ceiling plus 4 neighbors), and one `find` after, so `O(1)` union-find operations per hit
- Total: `O(m * n + k)` union-find operations

The `union` function always attaches `root_a` under `root_b` and does not use union by size or rank, so with path compression alone each operation costs `O(log(m * n))` amortized. Adding union by size would bring this down to `O(α(m * n))`, where `α` is the inverse Ackermann function. In practice the operations run in near-constant time.

**Space Complexity:** `O(m * n)`

- The `parent` array uses `O(m * n + 1)` space
- The `component_size` array uses `O(m * n + 1)` space  
- The copied grid `grid_after_hits` uses `O(m * n)` space
- The recursive `find` can use stack space proportional to the tree height, which is at most `O(m * n)`
- The result array uses `O(k)` space
- Total space complexity is `O(m * n)` since `k ≤ m * n` (hits are unique)

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Checking if Original Grid Had a Brick**

**Pitfall:** A common mistake is forgetting to check whether the original grid actually had a brick at the hit position. If you process a hit on an already empty cell (where `grid[i][j] == 0` originally), you might incorrectly calculate fallen bricks when reversing the operation.

**Why it happens:** When processing hits in reverse, we add bricks back. But if there was never a brick at that position originally, we shouldn't add one back.

**Incorrect approach:**
```python
for row_idx, col_idx in reversed(hits):
    # WRONG: Not checking if original grid had a brick
    grid_after_hits[row_idx][col_idx] = 1
    # ... rest of the logic
```

**Correct approach:**
```python
for row_idx, col_idx in reversed(hits):
    if grid[row_idx][col_idx] == 0:  # Original grid had no brick
        result.append(0)
        continue
  
    grid_after_hits[row_idx][col_idx] = 1  # Only add if originally present
    # ... rest of the logic
```

### 2. **Incorrect Calculation of Fallen Bricks**

**Pitfall:** Forgetting to subtract 1 from the difference in ceiling component sizes, which leads to counting the newly added brick itself as a "fallen" brick.

**Why it happens:** When we add a brick back and it connects to the ceiling, the ceiling component size increases. This increase includes both the newly added brick AND any previously disconnected bricks that now reconnect through it.

**Incorrect approach:**
```python
fallen_bricks = stable_bricks_after - stable_bricks_before
# WRONG: This counts the added brick as fallen
```

**Correct approach:**
```python
fallen_bricks = max(0, stable_bricks_after - stable_bricks_before - 1)
# Subtract 1 to exclude the brick we just added
# Use max(0, ...) to handle cases where brick doesn't connect to ceiling
```

### 3. **Union-Find Path Compression Bug**

**Pitfall:** Implementing path compression incorrectly by not updating the parent pointer during the recursive find operation.

**Incorrect approach:**
```python
def find(node: int) -> int:
    if parent[node] != node:
        return find(parent[node])  # WRONG: No path compression
    return parent[node]
```

**Correct approach:**
```python
def find(node: int) -> int:
    if parent[node] != node:
        parent[node] = find(parent[node])  # Compress path
    return parent[node]
```

### 4. **Modifying Original Grid Instead of Using a Copy**

**Pitfall:** Directly modifying the input `grid` when applying all hits initially, which corrupts the data needed to check if a position originally had a brick.

**Incorrect approach:**
```python
# WRONG: Modifying original grid
for i, j in hits:
    grid[i][j] = 0

# Later when processing in reverse:
if grid[i][j] == 0:  # This check is now meaningless!
    result.append(0)
```

**Correct approach:**
```python
# Create a copy to preserve original grid
grid_after_hits = [row[:] for row in grid]
for i, j in hits:
    grid_after_hits[i][j] = 0

# Original grid remains intact for checking
if grid[i][j] == 0:  # Can correctly check original state
    result.append(0)
```

### 5. **Double-Counting Connections When Building Initial Components**

**Pitfall:** Connecting bricks in all four directions when building the initial Union-Find structure, which unnecessarily doubles the work and could lead to subtle bugs if not handled consistently.

**Why it happens:** If you check all 4 neighbors for each brick, you'll process each edge twice (once from each endpoint).

**Less efficient approach:**
```python
# Checking all 4 directions for every brick
for i in range(m):
    for j in range(n):
        for di, dj in [(0,1), (0,-1), (1,0), (-1,0)]:
            # This processes each edge twice
```

**More efficient approach:**
```python
# Only check up and left (or down and right)
for i in range(1, m):
    for j in range(n):
        if grid_after_hits[i][j] == 0:
            continue
        # Check only up
        if grid_after_hits[i-1][j] == 1:
            union(i*n+j, (i-1)*n+j)
        # Check only left
        if j > 0 and grid_after_hits[i][j-1] == 1:
            union(i*n+j, i*n+j-1)
```

These pitfalls are particularly tricky because they might not cause obvious errors in simple test cases but will fail on edge cases or larger inputs.

