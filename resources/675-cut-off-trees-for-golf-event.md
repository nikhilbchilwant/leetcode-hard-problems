# 675. Cut Off Trees for Golf Event

> Source: https://algo.monster/liteproblems/675

## Problem Statement

You are given a forest represented as an `m x n` matrix where you need to cut down all trees for a golf event. Each cell in the matrix has the following meanings:

- `0`: The cell is blocked and cannot be walked through
- `1`: An empty cell that you can walk through
- A number greater than `1`: A tree with that specific height that you can walk through

You start at position `(0, 0)` and can move one step at a time in four directions: north, east, south, or west. You can only move to cells that are walkable (not `0`).

The key constraint is that **you must cut the trees in order from shortest to tallest**. Each tree has a unique height, and when you cut a tree, that cell becomes `1` (empty).

Your task is to find the **minimum number of steps** needed to walk from the starting position to cut all trees in the required order. If it's impossible to reach all trees, return `-1`.

For example, if there are trees with heights 2, 3, and 4, you must:
1. First walk from `(0, 0)` to the tree with height 2 and cut it
2. Then walk from that position to the tree with height 3 and cut it  
3. Finally walk to the tree with height 4 and cut it

The total steps would be the sum of steps for each journey between consecutive trees.

## Intuition

The key insight is to break down this problem into smaller, manageable pieces. We're not trying to find one optimal path that visits all trees - instead, we need to visit trees in a specific order (shortest to tallest).

Think of it like running errands in a city where you must visit stores in a specific order based on their opening times. You can't optimize the overall route freely; you must go to the earliest-opening store first, then the next, and so on.

This constraint naturally leads us to a **greedy approach**:
1. First, identify all trees and sort them by height
2. Visit them one by one in that sorted order
3. For each transition (current position → next tree), find the shortest path

Why BFS for each segment? Since we can move one step at a time in four directions (like moving on a chess board), and each move costs the same (1 step), we're dealing with an **unweighted graph problem**. BFS guarantees the shortest path in such scenarios because it explores all positions at distance `d` before exploring positions at distance `d+1`.

The solution cleverly optimizes the BFS with an A* heuristic using Manhattan distance `abs(i - x) + abs(j - y)`. This helps prioritize exploring cells that are geometrically closer to the target, making the search more efficient. The Manhattan distance provides a lower bound on the actual distance since we can only move horizontally or vertically.

The overall strategy is:
- **Decompose** the problem: Instead of finding one complex path, find multiple simple shortest paths
- **Order matters**: We must respect the height ordering constraint
- **Sum of local optimums**: The minimum total steps equals the sum of minimum steps for each segment

If any tree is unreachable (BFS returns -1), the entire task becomes impossible, so we return -1.

**Learn more about [Breadth-First Search](/problems/bfs_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation follows a systematic approach to solve this problem:

**1. Tree Collection and Sorting**
```python
trees = [(forest[i][j], i, j) for i in range(m) for j in range(n) if forest[i][j] > 1]
trees.sort()
```
We first scan the entire grid to find all trees (cells with value > 1), storing them as tuples `(height, row, col)`. Sorting this list ensures we process trees in ascending order of height.

**2. A* Enhanced BFS Implementation**
The `bfs(i, j, x, y)` function finds the shortest path from position `(i, j)` to target `(x, y)`:

- **Priority Queue with Heuristic**: Uses a min-heap where entries are `(f_score, row, col)`
  - `f_score = actual_distance + heuristic`
  - The heuristic `f(i, j, x, y) = abs(i - x) + abs(j - y)` is the Manhattan distance

- **Distance Tracking**: 
  ```python
  dist = {i * n + j: 0}
  ```
  Maps each cell to its shortest known distance from the start. We use `row * n + col` as a unique identifier for each cell to avoid using tuples as dictionary keys (more efficient).

- **Exploration Logic**:
  ```python
  for a, b in [[0, -1], [0, 1], [-1, 0], [1, 0]]:
      c, d = i + a, j + b
  ```
  Explores all four directions from the current position.

- **Path Validation**: Only considers cells that are:
  - Within grid boundaries: `0 <= c < m and 0 <= d < n`
  - Walkable: `forest[c][d] > 0`

- **Distance Update**: Updates the distance to a neighbor only if:
  - It hasn't been visited yet, OR
  - A shorter path is found: `dist[c * n + d] > step + 1`

**3. Main Algorithm Flow**
```python
i = j = 0  # Start at (0, 0)
ans = 0    # Total steps counter

for _, x, y in trees:
    t = bfs(i, j, x, y)
    if t == -1:
        return -1  # Tree unreachable
    ans += t
    i, j = x, y  # Update current position
```

The algorithm:
1. Starts at `(0, 0)`
2. For each tree in height order:
   - Finds the shortest path from current position to the tree
   - If any tree is unreachable, returns -1
   - Adds the path length to the total
   - Updates the current position to the tree's location

**Key Optimizations**:
- **A* Heuristic**: The Manhattan distance heuristic guides the search toward the target, reducing unnecessary exploration
- **Early Termination**: Returns immediately when the target is reached
- **Efficient Cell Indexing**: Uses `row * n + col` instead of tuple keys for better performance

The time complexity is `O(m²n² log(mn))` in the worst case (for each of up to `mn` trees, one search may push and pop every cell through the heap), where `m` and `n` are the grid dimensions. The space complexity is `O(mn)` for the distance dictionary and priority queue.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach:

```
Forest grid:
[[1, 2, 3],
 [0, 0, 4],
 [7, 6, 5]]

Starting position: (0, 0)
```

**Step 1: Identify and sort trees**
- Trees found: `(2, 0, 1)`, `(3, 0, 2)`, `(4, 1, 2)`, `(5, 2, 2)`, `(6, 2, 1)`, `(7, 2, 0)`
- After sorting by height: `[2, 3, 4, 5, 6, 7]`

**Step 2: Visit trees in order**

**Journey 1: (0,0) → Tree 2 at (0,1)**
- Direct path: Right 1 step
- Steps: 1
- Current position: (0,1)

**Journey 2: (0,1) → Tree 3 at (0,2)**
- Direct path: Right 1 step  
- Steps: 1
- Current position: (0,2)

**Journey 3: (0,2) → Tree 4 at (1,2)**
- Cell (1,2) is directly below (0,2) and is walkable (cells (1,0) and (1,1) are blocked, but they are not on this path)
- Direct path: Down 1 step
- Steps: 1
- Current position: (1,2)

**Journey 4: (1,2) → Tree 5 at (2,2)**
- Direct path: Down 1 step
- Steps: 1
- Current position: (2,2)

**Journey 5: (2,2) → Tree 6 at (2,1)**
- Direct path: Left 1 step
- Steps: 1
- Current position: (2,1)

**Journey 6: (2,1) → Tree 7 at (2,0)**
- Direct path: Left 1 step
- Steps: 1
- Current position: (2,0)

**Total steps: 1 + 1 + 1 + 1 + 1 + 1 = 6**

The A* optimization helps by prioritizing exploration toward the target. For instance, when finding a path from (0,1) to (0,2), the algorithm will immediately explore (0,2) since it has the lowest f_score:
- f_score = distance_traveled(1) + manhattan_distance(0) = 1

This prevents unnecessary exploration of other cells, making the search more efficient than standard BFS.

## Python Solution

```python
class Solution:
    def cutOffTree(self, forest: List[List[int]]) -> int:
        """
        Cut trees in ascending order of height and return minimum steps.
        Uses A* search to find shortest path between trees.
        """
      
        def heuristic(row1: int, col1: int, row2: int, col2: int) -> int:
            """Calculate Manhattan distance as heuristic for A* search."""
            return abs(row1 - row2) + abs(col1 - col2)
      
        def find_shortest_path(start_row: int, start_col: int, 
                               target_row: int, target_col: int) -> int:
            """
            Find shortest path from start to target using A* algorithm.
            Returns number of steps or -1 if unreachable.
            """
            # Priority queue: (f_score, row, col) where f_score = g_score + heuristic
            priority_queue = [(heuristic(start_row, start_col, target_row, target_col), 
                              start_row, start_col)]
          
            # Store minimum distance to each cell using flattened index
            distances = {start_row * cols + start_col: 0}
          
            while priority_queue:
                _, current_row, current_col = heappop(priority_queue)
                current_distance = distances[current_row * cols + current_col]
              
                # Check if we reached the target
                if (current_row, current_col) == (target_row, target_col):
                    return current_distance
              
                # Explore four directions: up, down, left, right
                for delta_row, delta_col in [[0, -1], [0, 1], [-1, 0], [1, 0]]:
                    next_row = current_row + delta_row
                    next_col = current_col + delta_col
                  
                    # Check if next position is valid and walkable (not 0)
                    if (0 <= next_row < rows and 
                        0 <= next_col < cols and 
                        forest[next_row][next_col] > 0):
                      
                        next_distance = current_distance + 1
                        next_index = next_row * cols + next_col
                      
                        # Update if we found a shorter path or visiting for first time
                        if (next_index not in distances or 
                            distances[next_index] > next_distance):
                          
                            distances[next_index] = next_distance
                            f_score = next_distance + heuristic(next_row, next_col, 
                                                               target_row, target_col)
                            heappush(priority_queue, (f_score, next_row, next_col))
          
            # Target is unreachable
            return -1
      
        # Get grid dimensions
        rows, cols = len(forest), len(forest[0])
      
        # Collect all trees (height > 1) with their positions
        trees = [(forest[row][col], row, col) 
                 for row in range(rows) 
                 for col in range(cols) 
                 if forest[row][col] > 1]
      
        # Sort trees by height to determine cutting order
        trees.sort()
      
        # Start from top-left corner
        current_row = current_col = 0
        total_steps = 0
      
        # Visit each tree in ascending height order
        for _, target_row, target_col in trees:
            steps_to_tree = find_shortest_path(current_row, current_col, 
                                              target_row, target_col)
          
            # If any tree is unreachable, the task is impossible
            if steps_to_tree == -1:
                return -1
          
            total_steps += steps_to_tree
            # Update current position to the tree we just cut
            current_row, current_col = target_row, target_col
      
        return total_steps
```

## Time and Space Complexity

**Time Complexity:** `O(m²n² log(mn))`

The algorithm consists of finding paths between consecutive trees in sorted order:
- Sorting trees takes `O(mnlog(mn))` where there are at most `m*n` trees
- For each tree (at most `m*n` trees), we perform an A* search using BFS
- Each A* search in worst case visits all `m*n` cells
- For each cell visited, we perform heap operations which take `O(log(mn))` in worst case
- The heap can contain at most `m*n` elements
- Each cell is pushed a constant number of times per search (at most once per neighbor that lowers its distance), so one search costs `O(mn log(mn))`
- Total for all searches: `O(mn * mn * log(mn))` = `O(m²n² log(mn))`, which dominates the sorting cost

**Space Complexity:** `O(mn)`

The space usage includes:
- `trees` list storing at most `m*n` tree positions: `O(mn)`
- In each BFS call:
  - Priority queue `priority_queue` holds `O(mn)` entries (each cell is pushed at most a few times): `O(mn)`
  - Dictionary `distances` can store at most `m*n` entries: `O(mn)`
- The space for BFS is reused for each search, not accumulated

Therefore, the total space complexity is `O(mn)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Choosing Between Standard BFS and A***
A standard BFS is also correct here: every move costs 1, so BFS returns the shortest path, and it runs in `O(mn)` per search without heap overhead. Its drawback is that it explores cells uniformly in all directions, so it may visit many cells far from the target. A* is an optional refinement, not a correctness requirement.

**Standard BFS (correct):**
```python
# Standard BFS without heuristic
queue = deque([(start_row, start_col, 0)])
visited = set()
```

**A* alternative:**
Use the Manhattan distance heuristic to guide the search toward the target:
```python
priority_queue = [(heuristic(start_row, start_col, target_row, target_col), 
                  start_row, start_col)]
```

### 2. **Incorrect Distance Tracking Leading to Suboptimal Paths**
In a plain BFS, a visited set that marks a cell when it is first enqueued is correct, because the first discovery of a cell is always along a shortest path. In A*, cells are popped in order of `distance + heuristic`, not by distance alone, so for a general heuristic the first path found to a cell is not guaranteed to be the shortest one. Locking in that first distance can then return a longer path.

**Risky with A*:**
```python
# Locks in the first distance found for each cell
if (next_row, next_col) in visited:
    continue
visited.add((next_row, next_col))
```

**Solution:**
Track actual distances and update when finding shorter paths, which is correct for any admissible heuristic:
```python
if (next_index not in distances or 
    distances[next_index] > next_distance):
    distances[next_index] = next_distance
    # Add to queue for further exploration
```

### 3. **Adding a Blanket Check for a Blocked Start**
The solution only checks walkability for cells it moves into; it never checks the cell it starts on. A tempting extra check is:

```python
if forest[0][0] == 0:
    return -1
```

This changes the answer for a forest with no trees, where 0 steps are needed and the solution correctly returns 0. If you want special handling for a blocked start, apply it only when there is at least one tree to reach.

### 4. **Using Tuples as Dictionary Keys (Performance Issue)**
Using coordinate tuples as dictionary keys can be slower than using flattened indices, especially for large grids.

**Problem:**
```python
# Slower approach with tuple keys
distances = {(start_row, start_col): 0}
if (next_row, next_col) not in distances:
    distances[(next_row, next_col)] = next_distance
```

**Solution:**
Use flattened index for better performance:
```python
# Faster with integer keys
distances = {start_row * cols + start_col: 0}
next_index = next_row * cols + next_col
if next_index not in distances:
    distances[next_index] = next_distance
```

### 5. **Not Handling the Case Where Starting Position Has a Tree**
If `forest[0][0] > 1`, it's a tree that needs to be cut, but since we start there, we need to ensure it's included in our tree list and handled correctly.

**Problem:**
```python
# Might miss counting steps if (0,0) has a tree
for _, target_row, target_col in trees:
    # If first tree is at (0,0), find_shortest_path returns 0
    # but we still need to process it
```

**Solution:**
The current implementation handles this correctly by:
- Including all trees in the sorted list regardless of position
- Starting path-finding from (0,0) to the first tree (even if it's at (0,0), returning 0 steps)
- Properly updating position after each tree

### 6. **Collisions in the Flattened Cell Index**
The flattened index `row * cols + col` is unique only when it multiplies by the number of columns. Using `rows` by mistake (`row * rows + col`) maps different cells to the same key on non-square grids.

**Problem:**
```python
# Wrong on non-square grids: (0, rows) and (1, 0) collide
next_index = next_row * rows + next_col
```

**Solution:**
Multiply by the column count, as the solution does. Overflow is not a concern here: with `m, n <= 50` the index stays below 2500.

