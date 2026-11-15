# 1632. Rank Transform of a Matrix

> Source: https://algo.monster/liteproblems/1632

## Problem Statement

You are given an `m x n` matrix of integers. Your task is to create a new matrix where each element is replaced by its **rank**.

The rank of an element follows these rules:

1. **Ranks start from 1** - The smallest rank assigned is 1, not 0.

2. **Same row/column ordering must be preserved** - For any two elements `p` and `q` that are in the same row OR the same column:
   - If `p < q`, then `rank(p) < rank(q)`
   - If `p == q`, then `rank(p) == rank(q)`  
   - If `p > q`, then `rank(p) > rank(q)`

3. **Ranks should be as small as possible** - While maintaining the ordering constraints, use the smallest possible rank values.

For example, if you have a matrix:
```
[[1, 2],
 [3, 4]]
```

The rank matrix would be:
```
[[1, 2],
 [2, 3]]
```

Here's why:
- `1` gets rank 1 (smallest element)
- `2` must have rank > 1 (same row as 1, and 2 > 1), so rank 2
- `3` must have rank > 1 (same column as 1, and 3 > 1), so rank 2
- `4` must have rank > 2 (same row as 3 which has rank 2, and 4 > 3), and rank > 2 (same column as 2 which has rank 2, and 4 > 2), so rank 3

The key challenge is handling equal values - they must receive the same rank even if they appear in different positions, as long as they can "see" each other through shared rows or columns. This creates connected components of cells that must all have the same rank.

## Intuition

The key insight is to process elements by their values in ascending order. This ensures that when we assign a rank to an element, all smaller elements have already been assigned their ranks, making it easy to maintain the ordering constraint.

Consider what happens when we process all cells with the same value. These cells might be connected through shared rows or columns. For instance, if cells `(0,1)` and `(0,2)` have the same value, they're in the same row. If cell `(1,2)` also has the same value, then all three cells form a connected group - `(0,1)` and `(0,2)` share row 0, while `(0,2)` and `(1,2)` share column 2. All cells in such a connected group must receive the same rank.

This naturally leads us to think about Union-Find (Disjoint Set Union). We can treat each row and column as a node, and when we encounter a cell at position `(i,j)`, we union row `i` with column `j`. All cells that end up in the same connected component must have the same rank.

To determine what rank to assign to a connected component, we need to look at the maximum rank already assigned in any row or column that the component touches. Why? Because our new rank must be greater than all previously assigned ranks in those rows and columns (since we're processing in ascending order of values).

Here's the step-by-step thought process:
1. Group cells by their values
2. Process groups in ascending order of values
3. For each group, use Union-Find to identify connected components (cells that share rows/columns)
4. For each component, find the maximum existing rank in any affected row or column
5. Assign `max_existing_rank + 1` to all cells in the component
6. Update the row and column maximum ranks for future iterations
7. Reset the Union-Find structure for the next group

The Union-Find structure elegantly handles the transitive connections - if cell A shares a row with cell B, and cell B shares a column with cell C, then all three must have the same rank even if A and C don't directly share a row or column.

**Learn more about [Union Find](/problems/dsu_intro), [Graph](/problems/graph_intro), [Topological Sort](/problems/topo_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation uses a Union-Find data structure and processes matrix elements grouped by their values in ascending order.

**Step 1: Group cells by values**
```python
d = defaultdict(list)
for i, row in enumerate(matrix):
    for j, v in enumerate(row):
        d[v].append((i, j))
```
We create a dictionary where keys are cell values and values are lists of `(row, column)` positions. This allows us to process all cells with the same value together.

**Step 2: Initialize tracking arrays**
```python
row_max = [0] * m  # Track maximum rank in each row
col_max = [0] * n  # Track maximum rank in each column
ans = [[0] * n for _ in range(m)]  # Result matrix
```

**Step 3: Create Union-Find structure**
```python
uf = UnionFind(m + n)
```
We create a Union-Find with `m + n` nodes. Rows are represented by indices `[0, m-1]` and columns by indices `[m, m+n-1]`. This clever indexing allows us to union rows and columns in the same structure.

**Step 4: Process values in ascending order**
```python
for v in sorted(d):
```
By processing values from smallest to largest, we ensure that when assigning ranks to cells with value `v`, all cells with smaller values already have their ranks assigned.

**Step 5: Union cells with the same value**
```python
for i, j in d[v]:
    uf.union(i, j + m)
```
For each cell at position `(i, j)`, we union row `i` with column `j + m`. This creates connected components where all cells that share rows or columns (directly or transitively) are grouped together.

**Step 6: Calculate rank for each component**
```python
rank = defaultdict(int)
for i, j in d[v]:
    rank[uf.find(i)] = max(rank[uf.find(i)], row_max[i], col_max[j])
```
For each connected component (identified by its root in Union-Find), we find the maximum rank already present in any row or column that the component touches. The new rank will be this maximum plus 1.

**Step 7: Assign ranks and update maximums**
```python
for i, j in d[v]:
    ans[i][j] = row_max[i] = col_max[j] = 1 + rank[uf.find(i)]
```
All cells in the same connected component get the same rank: `1 + maximum_existing_rank`. We simultaneously update the `row_max` and `col_max` arrays for future iterations.

**Step 8: Reset Union-Find for next value group**
```python
for i, j in d[v]:
    uf.reset(i)
    uf.reset(j + m)
```
After processing all cells with value `v`, we reset the Union-Find structure so it can be reused for the next value group. This is more efficient than creating a new Union-Find for each value.

**Union-Find Implementation Details:**
- `find(x)`: Uses path compression to find the root of element `x`
- `union(a, b)`: Merges two sets using union by size to keep the tree balanced
- `reset(x)`: Resets element `x` to be its own parent (standalone set)

The time complexity is `O(mn * log(mn) * α(mn))` where `α` is the inverse Ackermann function (practically constant). The [sorting](/problems/sorting_summary) dominates the complexity.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach:

Given matrix:
```
[[7, 3, 6],
 [1, 4, 5],
 [9, 8, 2]]
```

**Step 1: Group cells by values**
```
1: [(1,0)]
2: [(2,2)]
3: [(0,1)]
4: [(1,1)]
5: [(1,2)]
6: [(0,2)]
7: [(0,0)]
8: [(2,1)]
9: [(2,0)]
```

**Step 2: Initialize tracking arrays**
```
row_max = [0, 0, 0]  # Maximum rank in each row
col_max = [0, 0, 0]  # Maximum rank in each column
ans = [[0, 0, 0],
       [0, 0, 0],
       [0, 0, 0]]
```

**Step 3: Process value 1 at position (1,0)**
- Union row 1 with column 0 (union(1, 0+3=3))
- Component contains only this cell
- Max existing rank in row 1: 0, column 0: 0
- Assign rank: 1 + max(0, 0) = 1
- Update: ans[1][0] = 1, row_max[1] = 1, col_max[0] = 1
- Reset Union-Find

**Step 4: Process value 2 at position (2,2)**
- Union row 2 with column 2 (union(2, 2+3=5))
- Max existing rank in row 2: 0, column 2: 0
- Assign rank: 1 + max(0, 0) = 1
- Update: ans[2][2] = 1, row_max[2] = 1, col_max[2] = 1

**Step 5: Process value 3 at position (0,1)**
- Union row 0 with column 1 (union(0, 1+3=4))
- Max existing rank in row 0: 0, column 1: 0
- Assign rank: 1 + max(0, 0) = 1
- Update: ans[0][1] = 1, row_max[0] = 1, col_max[1] = 1

**Step 6: Process value 4 at position (1,1)**
- Union row 1 with column 1 (union(1, 1+3=4))
- Max existing rank in row 1: 1 (from value 1), column 1: 1 (from value 3)
- Assign rank: 1 + max(1, 1) = 2
- Update: ans[1][1] = 2, row_max[1] = 2, col_max[1] = 2

**Step 7: Process value 5 at position (1,2)**
- Union row 1 with column 2 (union(1, 2+3=5))
- Max existing rank in row 1: 2 (from value 4), column 2: 1 (from value 2)
- Assign rank: 1 + max(2, 1) = 3
- Update: ans[1][2] = 3, row_max[1] = 3, col_max[2] = 3

**Step 8: Process value 6 at position (0,2)**
- Union row 0 with column 2 (union(0, 2+3=5))
- Max existing rank in row 0: 1 (from value 3), column 2: 3 (from value 5)
- Assign rank: 1 + max(1, 3) = 4
- Update: ans[0][2] = 4, row_max[0] = 4, col_max[2] = 4

**Step 9: Process value 7 at position (0,0)**
- Union row 0 with column 0 (union(0, 0+3=3))
- Max existing rank in row 0: 4 (from value 6), column 0: 1 (from value 1)
- Assign rank: 1 + max(4, 1) = 5
- Update: ans[0][0] = 5, row_max[0] = 5, col_max[0] = 5

**Step 10: Process value 8 at position (2,1)**
- Union row 2 with column 1 (union(2, 1+3=4))
- Max existing rank in row 2: 1 (from value 2), column 1: 2 (from value 4)
- Assign rank: 1 + max(1, 2) = 3
- Update: ans[2][1] = 3, row_max[2] = 3, col_max[1] = 3

**Step 11: Process value 9 at position (2,0)**
- Union row 2 with column 0 (union(2, 0+3=3))
- Max existing rank in row 2: 3 (from value 8), column 0: 5 (from value 7)
- Assign rank: 1 + max(3, 5) = 6
- Update: ans[2][0] = 6, row_max[2] = 6, col_max[0] = 6

**Final Result:**
```
[[5, 1, 4],
 [1, 2, 3],
 [6, 3, 1]]
```

The algorithm ensures that:
- Each element's rank respects the ordering in its row and column
- Equal values that can "see" each other get the same rank
- Ranks are minimized while maintaining constraints

## Python Solution

```python
from typing import List
from collections import defaultdict


class UnionFind:
    """Union-Find (Disjoint Set Union) data structure with path compression and union by size."""
  
    def __init__(self, n: int) -> None:
        """Initialize Union-Find structure with n elements."""
        self.parent = list(range(n))  # Each element is its own parent initially
        self.size = [1] * n  # Size of each component is 1 initially
  
    def find(self, x: int) -> int:
        """Find the root of element x with path compression."""
        if self.parent[x] != x:
            # Path compression: make all nodes point directly to root
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
  
    def union(self, a: int, b: int) -> None:
        """Union two elements a and b by size (attach smaller tree to larger)."""
        root_a, root_b = self.find(a), self.find(b)
      
        if root_a != root_b:
            # Union by size: attach smaller component to larger one
            if self.size[root_a] > self.size[root_b]:
                self.parent[root_b] = root_a
                self.size[root_a] += self.size[root_b]
            else:
                self.parent[root_a] = root_b
                self.size[root_b] += self.size[root_a]
  
    def reset(self, x: int) -> None:
        """Reset element x to be its own parent with size 1."""
        self.parent[x] = x
        self.size[x] = 1


class Solution:
    def matrixRankTransform(self, matrix: List[List[int]]) -> List[List[int]]:
        """
        Transform matrix so that each element's rank is the smallest possible positive integer
        respecting the constraints that larger values get larger ranks in the same row/column.
        """
        m, n = len(matrix), len(matrix[0])
      
        # Group all positions by their values
        value_to_positions = defaultdict(list)
        for i, row in enumerate(matrix):
            for j, value in enumerate(row):
                value_to_positions[value].append((i, j))
      
        # Track maximum rank achieved in each row and column
        row_max_rank = [0] * m
        col_max_rank = [0] * n
      
        # Initialize result matrix
        result = [[0] * n for _ in range(m)]
      
        # Union-Find to handle rows and columns (rows: 0 to m-1, columns: m to m+n-1)
        uf = UnionFind(m + n)
      
        # Process values in ascending order
        for value in sorted(value_to_positions.keys()):
            # Dictionary to store the maximum rank for each connected component
            component_max_rank = defaultdict(int)
          
            # Union all positions with the same value that share row or column
            for row, col in value_to_positions[value]:
                # Connect row and column (column index offset by m)
                uf.union(row, col + m)
          
            # Find maximum existing rank for each connected component
            for row, col in value_to_positions[value]:
                root = uf.find(row)
                # Component's rank should be at least 1 more than max of row/col
                component_max_rank[root] = max(
                    component_max_rank[root], 
                    row_max_rank[row], 
                    col_max_rank[col]
                )
          
            # Assign ranks to all positions with current value
            for row, col in value_to_positions[value]:
                root = uf.find(row)
                new_rank = 1 + component_max_rank[root]
                result[row][col] = new_rank
                row_max_rank[row] = new_rank
                col_max_rank[col] = new_rank
          
            # Reset Union-Find for next value group
            for row, col in value_to_positions[value]:
                uf.reset(row)
                uf.reset(col + m)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(mn * log(mn) + mn * α(m+n))`

Breaking down the time complexity:
- Creating the dictionary `d` by iterating through all matrix elements: `O(mn)`
- Sorting the unique values in the dictionary: `O(k * log(k))` where `k` is the number of unique values, worst case `k = mn`, so `O(mn * log(mn))`
- For each unique value, we process all cells with that value:
  - Total cells processed across all values: `O(mn)`
  - For each cell, we perform Union-Find operations:
    - `union(i, j+m)`: `O(α(m+n))` per operation where `α` is the inverse Ackermann function
    - `find(i)`: `O(α(m+n))` per operation
    - `reset(i)` and `reset(j+m)`: `O(1)` per operation
  - Total Union-Find operations: `O(mn * α(m+n))`

The dominant term is `O(mn * log(mn))` from sorting, giving us overall time complexity of `O(mn * log(mn) + mn * α(m+n))`. Since `α(m+n)` grows extremely slowly and is practically constant for all reasonable inputs, this can be simplified to `O(mn * log(mn))`.

**Space Complexity:** `O(mn)`

Breaking down the space complexity:
- Dictionary `d` storing all cell positions: `O(mn)`
- UnionFind data structure with parent array and size array: `O(m+n)`
- `row_max` array: `O(m)`
- `col_max` array: `O(n)`
- Answer matrix `ans`: `O(mn)`
- `rank` dictionary in the worst case when all elements in a group belong to different components: `O(m+n)`

The dominant term is `O(mn)` from the dictionary and answer matrix, giving us overall space complexity of `O(mn)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrectly Handling Connected Components of Equal Values

**The Problem:**
A common mistake is failing to recognize that equal values form transitive connections through shared rows and columns. For example, if cells A and B are in the same row with equal values, and cells B and C are in the same column with equal values, then A, B, and C must all have the same rank, even though A and C don't directly share a row or column.

**Incorrect Approach:**
```python
# WRONG: Only considering direct row/column relationships
for row, col in value_to_positions[value]:
    new_rank = 1 + max(row_max_rank[row], col_max_rank[col])
    result[row][col] = new_rank
    row_max_rank[row] = new_rank
    col_max_rank[col] = new_rank
```

This fails on cases like:
```
[[7, 3, 6],
 [1, 4, 5],
 [9, 8, 2]]
```
Where multiple equal values might need to coordinate their ranks through transitive connections.

**The Solution:**
Use Union-Find to group all cells with the same value that are connected through shared rows/columns, then assign the same rank to the entire connected component.

### Pitfall 2: Not Resetting the Union-Find Structure

**The Problem:**
Forgetting to reset the Union-Find after processing each value group causes incorrect unions between different values, leading to wrong rank assignments.

**Incorrect Code:**
```python
# WRONG: Missing reset
for value in sorted(value_to_positions.keys()):
    for row, col in value_to_positions[value]:
        uf.union(row, col + m)
    # Process and assign ranks...
    # MISSING: uf.reset() calls
```

**Why It Fails:**
Without resetting, when processing value `v2` after `v1`, the Union-Find still maintains connections from `v1`, causing cells with different values to be incorrectly grouped together.

**The Solution:**
Always reset the Union-Find nodes after processing each value group:
```python
for row, col in value_to_positions[value]:
    uf.reset(row)
    uf.reset(col + m)
```

### Pitfall 3: Incorrect Index Mapping for Rows and Columns

**The Problem:**
Using the same index space for both rows and columns in the Union-Find structure causes collisions.

**Incorrect Code:**
```python
# WRONG: Row and column indices overlap
uf = UnionFind(max(m, n))  # Not enough space
for row, col in value_to_positions[value]:
    uf.union(row, col)  # Collision when row index equals column index
```

**Why It Fails:**
If you have a 3×3 matrix, row 2 and column 2 would both map to index 2 in the Union-Find, causing incorrect unions.

**The Solution:**
Use separate index spaces: rows use indices `[0, m-1]` and columns use indices `[m, m+n-1]`:
```python
uf = UnionFind(m + n)
for row, col in value_to_positions[value]:
    uf.union(row, col + m)  # Offset column index by m
```

### Pitfall 4: Processing Values in Wrong Order

**The Problem:**
Processing values in random or descending order violates the rank ordering constraint.

**Incorrect Code:**
```python
# WRONG: Processing in arbitrary order
for value in value_to_positions.keys():  # Unordered
    # Process cells with this value
```

**Why It Fails:**
If you process larger values before smaller ones, you might assign a rank to a large value that's too small, making it impossible to assign correct ranks to smaller values later.

**The Solution:**
Always process values in ascending order to ensure smaller values get smaller ranks:
```python
for value in sorted(value_to_positions.keys()):
    # Process cells with this value
```

