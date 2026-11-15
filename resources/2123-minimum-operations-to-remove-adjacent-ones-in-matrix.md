# 2123. Minimum Operations to Remove Adjacent Ones in Matrix

> Source: https://algo.monster/liteproblems/2123

## Problem Statement

You have a binary matrix (containing only 0s and 1s) where you can flip any 1 to become a 0. Your goal is to make the matrix "well-isolated" using the minimum number of flips.

A matrix is considered **well-isolated** when no two 1s are adjacent to each other horizontally or vertically (4-directionally connected). In other words, every 1 in the matrix should be completely surrounded by 0s in the four cardinal directions (up, down, left, right).

For example:
- A matrix with 1s at positions that share an edge (horizontally or vertically adjacent) is NOT well-isolated
- A matrix where all 1s are separated by at least one 0 in all four directions IS well-isolated
- 1s that are diagonally adjacent are allowed (since we only consider 4-directional connectivity)

The problem asks you to find the minimum number of 1s that need to be flipped to 0 to achieve this well-isolated state.

## Intuition

The key insight is recognizing that this problem is about breaking connections between adjacent 1s. When two 1s are adjacent, we must flip at least one of them to make the grid well-isolated.

Think of the grid like a checkerboard where we color cells black and white alternately. Any cell at position `(i, j)` where `(i + j)` is even gets one color, and cells where `(i + j)` is odd get another color. The crucial observation is that **adjacent cells always have different colors** in this checkerboard pattern.

Now, consider pairs of adjacent 1s in the grid. Since adjacent cells have different colors, every pair of connected 1s forms an edge between a black cell and a white cell. This naturally creates a bipartite [graph](/problems/graph_intro) where:
- One set contains all 1s on black cells
- Another set contains all 1s on white cells
- Edges exist between adjacent 1s

To make the grid well-isolated, we need to break all these edges. Breaking an edge means flipping at least one of the two 1s it connects. The question becomes: what's the minimum number of 1s to flip to break all edges?

This is exactly the **Minimum Vertex Cover** problem in bipartite graphs - finding the smallest set of vertices (1s to flip) such that every edge (connection between adjacent 1s) has at least one endpoint in this set.

By König's theorem, in a bipartite [graph](/problems/graph_intro), the size of the minimum vertex cover equals the size of the maximum matching. A matching pairs up vertices from the two sets such that no vertex appears in more than one pair. The maximum matching gives us the optimal way to "cover" the most edges with the fewest vertices.

Therefore, we can solve this by finding the maximum matching in the bipartite [graph](/problems/graph_intro) formed by adjacent 1s, which directly gives us the minimum number of flips needed.

**Learn more about [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The implementation uses the **Hungarian Algorithm** (specifically, the augmenting path method) to find the maximum bipartite matching.

**Step 1: Build the Bipartite [Graph](/problems/graph_intro)**

We iterate through the grid and only consider cells where `(i + j) % 2 == 1` (one partition of our checkerboard). For each 1 in this partition, we:
- Convert the 2D position `(i, j)` to a 1D index: `x = i * n + j`
- Check all four adjacent cells (up, down, left, right)
- If an adjacent cell contains a 1, we add an edge in our [graph](/problems/graph_intro) `g`

```python
for i, row in enumerate(grid):
    for j, v in enumerate(row):
        if (i + j) % 2 and v:  # Only process 1s in odd-sum positions
            x = i * n + j
            # Check all 4 directions and add edges to adjacent 1s
```

**Step 2: Initialize Matching Array**

We create a `match` array of size `m * n` initialized to `-1`. This array tracks which vertex from the even-sum partition is matched to each vertex from the odd-sum partition.

**Step 3: Find Maximum Matching**

For each vertex in our [graph](/problems/graph_intro) (1s at odd-sum positions), we try to find an augmenting path using DFS:

```python
def find(i: int) -> int:
    for j in g[i]:  # Try all adjacent 1s
        if j not in vis:  # Not visited in current DFS
            vis.add(j)
            if match[j] == -1 or find(match[j]):  # j is unmatched or can be rematched
                match[j] = i
                return 1
    return 0
```

The `find` function implements the augmenting path search:
- It tries to match vertex `i` with any of its neighbors `j`
- If `j` is unmatched (`match[j] == -1`), we directly match them
- If `j` is already matched to some other vertex, we recursively try to find an alternative match for that vertex
- The `vis` set prevents cycles during a single augmenting path search

**Step 4: Count the Matches**

For each vertex in the odd-sum partition with edges, we call `find` and accumulate successful matches:

```python
ans = 0
for i in g.keys():
    vis = set()  # Reset visited set for each new augmenting path search
    ans += find(i)
```

The final answer `ans` represents the maximum matching size, which by König's theorem equals the minimum vertex cover - the minimum number of 1s we need to flip to make the grid well-isolated.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this 3×3 binary matrix:
```
1 1 0
1 1 1
0 1 0
```

**Step 1: Identify the checkerboard pattern**

We color cells based on `(i + j) % 2`:
- Even sum (0): positions (0,0), (0,2), (1,1), (2,0), (2,2)
- Odd sum (1): positions (0,1), (1,0), (1,2), (2,1)

```
E O E    (E = even, O = odd)
O E O
E O E
```

Our matrix with the pattern overlay:
```
1(E) 1(O) 0(E)
1(O) 1(E) 1(O)
0(E) 1(O) 0(E)
```

**Step 2: Build the bipartite graph**

We process 1s at odd-sum positions and find their adjacent 1s:
- Position (0,1) has value 1, checks neighbors:
  - Up: out of bounds
  - Down: (1,1) has value 1 ✓
  - Left: (0,0) has value 1 ✓
  - Right: (0,2) has value 0
  - Adds edges: (0,1) → {(1,1), (0,0)}

- Position (1,0) has value 1, checks neighbors:
  - Up: (0,0) has value 1 ✓
  - Down: (2,0) has value 0
  - Left: out of bounds
  - Right: (1,1) has value 1 ✓
  - Adds edges: (1,0) → {(0,0), (1,1)}

- Position (1,2) has value 1, checks neighbors:
  - Up: (0,2) has value 0
  - Down: (2,2) has value 0
  - Left: (1,1) has value 1 ✓
  - Right: out of bounds
  - Adds edges: (1,2) → {(1,1)}

- Position (2,1) has value 1, checks neighbors:
  - Up: (1,1) has value 1 ✓
  - Down: out of bounds
  - Left: (2,0) has value 0
  - Right: (2,2) has value 0
  - Adds edges: (2,1) → {(1,1)}

Converting to 1D indices (i * 3 + j):
- (0,1) = 1, (1,0) = 3, (1,2) = 5, (2,1) = 7 [odd-sum vertices]
- (0,0) = 0, (1,1) = 4 [even-sum vertices they connect to]

Graph g:
- g[1] = {4, 0}
- g[3] = {0, 4}
- g[5] = {4}
- g[7] = {4}

**Step 3: Find maximum matching using augmenting paths**

Initialize match array: `match = [-1, -1, -1, -1, -1, -1, -1, -1, -1]`

Process vertex 1:
- Try to match with 4: unmatched, so `match[4] = 1` ✓
- Return 1

Process vertex 3:
- Try to match with 0: unmatched, so `match[0] = 3` ✓
- Return 1

Process vertex 5:
- Try to match with 4: already matched to 1
- Try to rematch 1: can it find another match?
  - Vertex 1 tries 0: already matched to 3
  - Try to rematch 3: can it find another match?
    - Vertex 3 tries 4: visited in this path
    - Return 0
  - Return 0
- Return 0

Process vertex 7:
- Try to match with 4: already matched to 1
- Try to rematch 1: can it find another match?
  - Vertex 1 tries 0: already matched to 3
  - Try to rematch 3: can it find another match?
    - Vertex 3 tries 4: visited in this path
    - Return 0
  - Return 0
- Return 0

**Step 4: Result**

Maximum matching size = 2 (vertices 1↔4 and 3↔0 are matched)

This means we need to flip at least 2 ones to make the grid well-isolated. Looking at our original grid, the matching tells us an optimal solution: we can flip the 1s at positions (0,0) and (1,1), which breaks all adjacencies:

Original:
```
1 1 0
1 1 1
0 1 0
```

After flipping (0,0) and (1,1):
```
0 1 0
1 0 1
0 1 0
```

Now no two 1s are adjacent - the grid is well-isolated with 2 flips, which matches our algorithm's answer.

## Python Solution

```python
class Solution:
    def minimumOperations(self, grid: List[List[int]]) -> int:
        def find_augmenting_path(source_node: int) -> int:
            """
            Find an augmenting path from source_node using DFS.
            Returns 1 if a matching is found, 0 otherwise.
            """
            # Try to find an unmatched neighbor or augment existing matching
            for neighbor in adjacency_list[source_node]:
                if neighbor not in visited:
                    visited.add(neighbor)
                    # If neighbor is unmatched or we can find an augmenting path from its match
                    if matching[neighbor] == -1 or find_augmenting_path(matching[neighbor]):
                        matching[neighbor] = source_node
                        return 1
            return 0

        # Build adjacency list for bipartite graph
        adjacency_list = defaultdict(list)
        rows, cols = len(grid), len(grid[0])

        # Process each cell in the grid
        for row_idx, row in enumerate(grid):
            for col_idx, value in enumerate(row):
                # Only process cells where (row + col) is odd and cell value is non-zero
                # This creates one partition of the bipartite graph
                if (row_idx + col_idx) % 2 and value:
                    # Convert 2D coordinates to 1D index
                    current_cell = row_idx * cols + col_idx

                    # Check bottom neighbor
                    if row_idx < rows - 1 and grid[row_idx + 1][col_idx]:
                        adjacency_list[current_cell].append(current_cell + cols)

                    # Check top neighbor
                    if row_idx and grid[row_idx - 1][col_idx]:
                        adjacency_list[current_cell].append(current_cell - cols)

                    # Check right neighbor
                    if col_idx < cols - 1 and grid[row_idx][col_idx + 1]:
                        adjacency_list[current_cell].append(current_cell + 1)

                    # Check left neighbor
                    if col_idx and grid[row_idx][col_idx - 1]:
                        adjacency_list[current_cell].append(current_cell - 1)

        # Initialize matching array (-1 means unmatched)
        matching = [-1] * (rows * cols)
        max_matching = 0

        # Find maximum matching using augmenting paths
        for source_node in adjacency_list.keys():
            visited = set()
            max_matching += find_augmenting_path(source_node)

        return max_matching
```

## Time and Space Complexity

**Time Complexity:** `O(m * n * E)` where `m` and `n` are the dimensions of the grid, and `E` is the number of edges in the bipartite graph (which is `O(m * n)` in the worst case).

- Building the graph `g` takes `O(m * n)` time as we iterate through each cell once and check at most 4 neighbors for each cell.
- The matching algorithm (Hungarian/Ford-Fulkerson for bipartite matching) runs with:
  - Outer loop iterates through all keys in `g`, which is at most `O(m * n / 2)` (only cells where `(i + j) % 2` is true and `v` is non-zero)
  - For each key, the `find()` function performs a DFS that can visit at most `O(m * n)` nodes in the worst case
  - Each edge is explored at most once per DFS call due to the `vis` set
- Overall: `O(m * n) + O(m * n) * O(m * n) = O((m * n)^2)`

**Space Complexity:** `O(m * n)`

- Graph `g` stores adjacency lists with total edges `O(m * n)` in worst case
- `match` array uses `O(m * n)` space
- `vis` set uses at most `O(m * n)` space during each DFS call
- The recursion stack for `find()` can go up to `O(m * n)` depth in the worst case
- Overall space complexity: `O(m * n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Bipartite Graph Construction**
A common mistake is building edges from both partitions of the checkerboard, which would create duplicate edges and lead to incorrect matching results.

**Wrong approach:**
```python
# Processing ALL cells with value 1
for i in range(rows):
    for j in range(cols):
        if grid[i][j] == 1:  # Wrong: processes both partitions
            # Add edges to all adjacent 1s
```

**Correct approach:**
```python
# Only process cells where (i + j) % 2 == 1
for i in range(rows):
    for j in range(cols):
        if (i + j) % 2 == 1 and grid[i][j] == 1:  # Correct: single partition
            # Add edges to adjacent 1s
```

### 2. **Reusing the Visited Set Across Multiple Augmenting Path Searches**
Each augmenting path search needs a fresh visited set. Reusing the same set would prevent finding valid augmenting paths in subsequent iterations.

**Wrong approach:**
```python
visited = set()  # Single visited set for all searches
for source_node in adjacency_list.keys():
    max_matching += find_augmenting_path(source_node)  # Wrong: visited never cleared
```

**Correct approach:**
```python
for source_node in adjacency_list.keys():
    visited = set()  # Fresh visited set for each search
    max_matching += find_augmenting_path(source_node)
```

### 3. **Misunderstanding the Problem Requirements**
Some might think we need to flip ALL 1s that have adjacent 1s, but we only need to flip the MINIMUM number to break all adjacencies.

**Example:**
```
1 1 1  →  We only need to flip the middle 1: 1 0 1
```
Not all three 1s need to be flipped!

### 4. **Incorrect Index Conversion Between 2D and 1D**
When converting between 2D grid coordinates and 1D indices, using the wrong dimension for multiplication causes incorrect neighbor identification.

**Wrong approach:**
```python
current_cell = row_idx * rows + col_idx  # Wrong: should use cols, not rows
```

**Correct approach:**
```python
current_cell = row_idx * cols + col_idx  # Correct: multiply by number of columns
```

### 5. **Not Handling Edge Cases in Boundary Checks**
Forgetting to check grid boundaries before accessing neighbors can cause index out of bounds errors.

**Wrong approach:**
```python
# Checking neighbor without boundary validation
if grid[row_idx + 1][col_idx]:  # Can cause IndexError
    adjacency_list[current_cell].append(current_cell + cols)
```

**Correct approach:**
```python
# Check boundary first
if row_idx < rows - 1 and grid[row_idx + 1][col_idx]:
    adjacency_list[current_cell].append(current_cell + cols)
```

