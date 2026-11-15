# 2392. Build a Matrix With Conditions

> Source: https://algo.monster/liteproblems/2392

## Problem Statement

You need to build a `k x k` matrix that places numbers from `1` to `k` exactly once, with all other cells containing `0`.

The placement must follow ordering constraints:
- **Row constraints** (`rowConditions`): Each constraint `[above_i, below_i]` means number `above_i` must appear in a row strictly above the row containing number `below_i`
- **Column constraints** (`colConditions`): Each constraint `[left_i, right_i]` means number `left_i` must appear in a column strictly to the left of the column containing number `right_i`

The solution uses **topological sorting** to determine valid orderings:
1. Function `f(cond)` performs [topological sort](/problems/topo_intro) on the constraints:
   - Builds a directed [graph](/problems/graph_intro) where edge `a → b` means `a` must come before `b`
   - Tracks in-degrees (number of incoming edges) for each node
   - Uses BFS starting from nodes with no dependencies (in-degree = 0)
   - Returns the sorted order if valid, or `None` if a cycle exists

2. Applies [topological sort](/problems/topo_intro) to both row and column constraints separately:
   - `row` gives the top-to-bottom ordering of numbers
   - `col` gives the left-to-right ordering of numbers

3. Constructs the final matrix:
   - Creates a mapping `m` from each number to its column position
   - Places each number at position `[row_index][column_index]`
   - Number `v` at position `i` in row ordering goes to row `i`, column `m[v]`

Returns an empty list if either ordering is invalid (contains a cycle), otherwise returns the constructed matrix.

## Intuition

The key insight is that we can decompose this 2D placement problem into two independent 1D ordering problems.

Think about it this way: if we need to place numbers `1` to `k` in a matrix with certain "above/below" and "left/right" relationships, we essentially need to:
1. Determine which row each number should go in (respecting the row constraints)
2. Determine which column each number should go in (respecting the column constraints)

These two orderings are **independent** - the row position of a number doesn't affect its column position and vice versa. This means we can solve them separately.

For each set of constraints, we're essentially asking: "Given that some numbers must come before others, what's a valid ordering?" This is the classic **topological sorting** problem. If `A` must be above `B`, then `A` comes earlier in the row ordering. If `C` must be left of `D`, then `C` comes earlier in the column ordering.

The beauty of this approach is that [topological sort](/problems/topo_intro) also naturally detects impossibilities - if there's a cycle (like `A` must be above `B`, `B` must be above `C`, and `C` must be above `A`), the topological sort will fail to include all numbers, telling us no valid arrangement exists.

Once we have both orderings:
- The row ordering tells us: "number at position 0 goes in row 0, number at position 1 goes in row 1, etc."
- The column ordering tells us: "number at position 0 goes in column 0, number at position 1 goes in column 1, etc."

By combining these two pieces of information, we know exactly where each number should be placed in the matrix. The remaining cells are simply filled with `0`.

**Learn more about [Graph](/problems/graph_intro) and [Topological Sort](/problems/topo_intro) patterns.**

## Solution Approach

The implementation uses **Kahn's algorithm** for topological sorting to find valid orderings for both rows and columns.

**Step 1: [Topological Sort](/problems/topo_intro) Function `f(cond)`**

For each set of constraints, we build a directed [graph](/problems/graph_intro) and perform [topological sort](/problems/topo_intro):

1. **Build the [graph](/problems/graph_intro)**: 
   - Create adjacency list `g` where `g[a]` contains all nodes that must come after `a`
   - Track in-degrees in array `indeg` where `indeg[b]` counts how many nodes must come before `b`
   - For each constraint `[a, b]`, add edge `a → b` and increment `indeg[b]`

2. **Initialize BFS queue**:
   - Find all nodes with `indeg = 0` (no prerequisites)
   - These nodes can be placed first in the ordering

3. **Process nodes level by level**:
   - Remove a node from queue and add to result
   - For each neighbor, decrement its in-degree
   - If neighbor's in-degree becomes 0, add it to queue
   - This ensures we only process nodes after all their prerequisites

4. **Validate the result**:
   - If `len(res) != k`, there's a cycle (impossible constraints)
   - Return `None` for invalid case, otherwise return the ordering

**Step 2: Get Row and Column Orderings**

```python
row = f(rowConditions)  # Get top-to-bottom ordering
col = f(colConditions)  # Get left-to-right ordering
```

If either returns `None`, no valid matrix exists, so return empty list.

**Step 3: Build the Matrix**

1. Create a `k × k` matrix initialized with zeros: `ans = [[0] * k for _ in range(k)]`

2. Create position mapping for columns:
   ```python
   m = [0] * (k + 1)  # m[number] = column_index
   for i, v in enumerate(col):
       m[v] = i
   ```
   This maps each number to its column position based on the column ordering.

3. Place numbers in the matrix:
   ```python
   for i, v in enumerate(row):
       ans[i][m[v]] = v
   ```
   - `i` is the row index (position in row ordering)
   - `v` is the number to place
   - `m[v]` is the column index where `v` should go
   - Place number `v` at position `[i][m[v]]`

**Time Complexity**: `O(k + n + m)` where `n` and `m` are the number of row and column conditions respectively. The [topological sort](/problems/topo_intro) runs in linear time relative to nodes and edges.

**Space Complexity**: `O(k²)` for the output matrix, plus `O(k)` for the [graph](/problems/graph_intro) structures and orderings.

## Example Walkthrough

Let's walk through a small example with `k = 3`, `rowConditions = [[1,2],[3,2]]`, and `colConditions = [[2,1],[3,1]]`.

**Step 1: Process Row Conditions**

Row conditions `[[1,2],[3,2]]` mean:
- Number 1 must be in a row above number 2
- Number 3 must be in a row above number 2

Building the graph:
- Edges: `1→2`, `3→2`
- In-degrees: `[0, 0, 2, 0]` (index 0 unused, numbers 1 and 3 have no incoming edges, number 2 has 2 incoming edges)

Topological sort:
1. Start with nodes having in-degree 0: queue = `[1, 3]`
2. Process node 1: Add to result, decrement in-degree of 2 (now 1)
3. Process node 3: Add to result, decrement in-degree of 2 (now 0)
4. Node 2 now has in-degree 0, add to queue
5. Process node 2: Add to result

Row ordering: `[1, 3, 2]` (top to bottom)

**Step 2: Process Column Conditions**

Column conditions `[[2,1],[3,1]]` mean:
- Number 2 must be in a column left of number 1
- Number 3 must be in a column left of number 1

Building the graph:
- Edges: `2→1`, `3→1`
- In-degrees: `[0, 2, 0, 0]` (number 1 has 2 incoming edges)

Topological sort:
1. Start with nodes having in-degree 0: queue = `[2, 3]`
2. Process node 2: Add to result, decrement in-degree of 1 (now 1)
3. Process node 3: Add to result, decrement in-degree of 1 (now 0)
4. Node 1 now has in-degree 0, add to queue
5. Process node 1: Add to result

Column ordering: `[2, 3, 1]` (left to right)

**Step 3: Build the Matrix**

Create column position mapping from `col = [2, 3, 1]`:
- `m[2] = 0` (number 2 goes in column 0)
- `m[3] = 1` (number 3 goes in column 1)
- `m[1] = 2` (number 1 goes in column 2)

Place numbers using `row = [1, 3, 2]`:
- Row 0: Place number 1 at column `m[1] = 2` → `ans[0][2] = 1`
- Row 1: Place number 3 at column `m[3] = 1` → `ans[1][1] = 3`
- Row 2: Place number 2 at column `m[2] = 0` → `ans[2][0] = 2`

Final matrix:
```
[[0, 0, 1],
 [0, 3, 0],
 [2, 0, 0]]
```

Verification:
- Row constraints satisfied: 1 is in row 0, 3 is in row 1, both above 2 in row 2 ✓
- Column constraints satisfied: 2 is in column 0, 3 is in column 1, both left of 1 in column 2 ✓

## Python Solution

```python
class Solution:
    def buildMatrix(
        self, k: int, rowConditions: List[List[int]], colConditions: List[List[int]]
    ) -> List[List[int]]:
        def topological_sort(conditions):
            """
            Perform topological sort on the given conditions.
            Returns sorted order if valid, None if cycle detected.
            """
            # Build adjacency list and track in-degrees
            adjacency_list = defaultdict(list)
            in_degree = [0] * (k + 1)  # Index 0 unused, 1 to k for actual values
          
            # Process each condition: value_a should come before value_b
            for value_a, value_b in conditions:
                adjacency_list[value_a].append(value_b)
                in_degree[value_b] += 1
          
            # Initialize queue with all nodes having zero in-degree
            # Skip index 0, enumerate from 1 to k
            queue = deque([i for i, degree in enumerate(in_degree[1:], 1) if degree == 0])
            sorted_order = []
          
            # Process nodes in topological order
            while queue:
                # Process all nodes at current level
                for _ in range(len(queue)):
                    current_node = queue.popleft()
                    sorted_order.append(current_node)
                  
                    # Reduce in-degree for all neighbors
                    for neighbor in adjacency_list[current_node]:
                        in_degree[neighbor] -= 1
                        if in_degree[neighbor] == 0:
                            queue.append(neighbor)
          
            # Check if all k nodes were processed (no cycle)
            return None if len(sorted_order) != k else sorted_order

        # Get row and column orderings via topological sort
        row_order = topological_sort(rowConditions)
        col_order = topological_sort(colConditions)
      
        # Return empty matrix if either ordering is invalid (contains cycle)
        if row_order is None or col_order is None:
            return []
      
        # Initialize k x k matrix with zeros
        result_matrix = [[0] * k for _ in range(k)]
      
        # Create mapping from value to column index
        value_to_col_index = [0] * (k + 1)  # Index 0 unused
        for col_idx, value in enumerate(col_order):
            value_to_col_index[value] = col_idx
      
        # Place each value in the matrix according to row and column orderings
        for row_idx, value in enumerate(row_order):
            col_idx = value_to_col_index[value]
            result_matrix[row_idx][col_idx] = value
      
        return result_matrix
```

## Time and Space Complexity

**Time Complexity:** `O(k^2 + R + C)` where `k` is the matrix dimension, `R` is the length of rowConditions, and `C` is the length of colConditions.

The analysis breaks down as follows:
- The topological sort function `f()` is called twice (once for rows, once for columns)
- For each call to `f()`:
  - Building the graph takes `O(len(conditions))` time to iterate through all conditions
  - Initializing indegree array takes `O(k)` time
  - The BFS/topological sort processes each node once and each edge once, taking `O(k + len(conditions))` time
- After getting both orderings:
  - Creating the mapping array `m` takes `O(k)` time
  - Filling the answer matrix takes `O(k)` time to place `k` elements
  - Initializing the answer matrix takes `O(k^2)` time

Total: `O(k + R) + O(k + C) + O(k) + O(k) + O(k^2)` = `O(k^2 + R + C)`

**Space Complexity:** `O(k^2 + R + C)`

The space usage includes:
- Graph representation `g` stores all edges: `O(R)` for row graph and `O(C)` for column graph
- Indegree array: `O(k)` for each call to `f()`
- Queue for BFS: `O(k)` in worst case
- Result arrays `row` and `col`: `O(k)` each
- Mapping array `m`: `O(k)`
- Answer matrix: `O(k^2)`

Total: `O(k^2) + O(R) + O(C) + O(k)` = `O(k^2 + R + C)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Cycle Detection in Topological Sort**

**Pitfall**: A common mistake is assuming that if the queue becomes empty, we've successfully completed the topological sort. However, the queue can become empty even when there's a cycle in the graph - this happens when all remaining unprocessed nodes are part of a cycle (all have non-zero in-degrees).

**Wrong Implementation**:
```python
def topological_sort(conditions):
    # ... build graph and in-degrees ...
    queue = deque([i for i in range(1, k+1) if in_degree[i] == 0])
    sorted_order = []
  
    while queue:
        current = queue.popleft()
        sorted_order.append(current)
        for neighbor in adjacency_list[current]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)
  
    # WRONG: Just returning the result without checking completeness
    return sorted_order  # This might return partial ordering!
```

**Solution**: Always verify that the topological sort processed exactly `k` nodes. If fewer nodes were processed, it means there's a cycle:
```python
# Correct validation
return None if len(sorted_order) != k else sorted_order
```

### 2. **Missing Values in the Ordering**

**Pitfall**: Not all values from 1 to k might appear in the constraints. If a value doesn't appear in any constraint, it won't have any edges in the graph but should still be included in the ordering.

**Wrong Approach**:
```python
# Only considering values that appear in constraints
visited = set()
for a, b in conditions:
    visited.add(a)
    visited.add(b)
# This might miss values that don't appear in any constraint!
```

**Solution**: Initialize the queue with ALL values from 1 to k that have zero in-degree, not just those appearing in constraints:
```python
# Correct initialization - checks all values from 1 to k
queue = deque([i for i in range(1, k+1) if in_degree[i] == 0])
```

### 3. **Index Out of Bounds When Building the Matrix**

**Pitfall**: Using 1-indexed values directly as array indices when placing elements in the matrix, or incorrectly handling the mapping between values and positions.

**Wrong Implementation**:
```python
# WRONG: Using values directly as indices
for i, v in enumerate(row_order):
    result_matrix[v][col_order[i]] = v  # Values are 1-indexed, arrays are 0-indexed!
```

**Solution**: Maintain proper mappings between values (1 to k) and array indices (0 to k-1):
```python
# Create proper value-to-column-index mapping
value_to_col_index = [0] * (k + 1)  # Index 0 unused, indices 1-k for values 1-k
for col_idx, value in enumerate(col_order):
    value_to_col_index[value] = col_idx  # col_idx is 0-indexed

# Use enumeration index for row, mapped index for column
for row_idx, value in enumerate(row_order):
    col_idx = value_to_col_index[value]
    result_matrix[row_idx][col_idx] = value
```

### 4. **Handling Self-Loops and Duplicate Constraints**

**Pitfall**: The constraints might contain self-loops (e.g., `[2, 2]`) or duplicate edges (e.g., multiple `[1, 3]` constraints), which can cause incorrect in-degree counting or infinite loops.

**Solution**: Add validation to skip self-loops and handle duplicates:
```python
def topological_sort(conditions):
    adjacency_list = defaultdict(set)  # Use set to avoid duplicate edges
    in_degree = [0] * (k + 1)
  
    for value_a, value_b in conditions:
        if value_a == value_b:  # Skip self-loops
            continue
        if value_b not in adjacency_list[value_a]:  # Avoid counting duplicates
            adjacency_list[value_a].add(value_b)
            in_degree[value_b] += 1
    # ... rest of the implementation
```

