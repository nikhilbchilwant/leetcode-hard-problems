# 52. N-Queens II

> Source: https://algo.monster/liteproblems/52

## Problem Statement

The n-queens puzzle asks you to place `n` queens on an `n x n` chessboard such that no two queens can attack each other. 

In chess, a queen can attack any piece that is:
- In the same row
- In the same column  
- On the same diagonal (both positive and negative diagonals)

Given an integer `n` representing the size of the chessboard, you need to find and return the total number of distinct valid arrangements where all `n` queens can be placed on the board without any queen attacking another.

For example, if `n = 4`, there are 2 distinct solutions to place 4 queens on a 4x4 board where no queens attack each other. The function should return `2` in this case.

The challenge is to explore all possible placements systematically while ensuring that each placement follows the constraint that no two queens can attack each other.

## Intuition

When we think about placing queens on a chessboard, we need to ensure no two queens can attack each other. Since queens attack along rows, columns, and diagonals, we need a systematic way to explore valid placements.

The key insight is that we must place exactly one queen per row (since n queens on n rows means one queen per row). This immediately simplifies our problem - instead of considering all possible positions on the board, we only need to decide which column to place the queen in for each row.

We can build our solution row by row. Starting from the first row, we try placing a queen in each column. For each placement, we check if it conflicts with previously placed queens. If it's valid, we move to the next row and repeat. If we successfully place queens in all n rows, we've found one valid solution.

The challenge is tracking conflicts efficiently. A queen at position `(i, j)` attacks:
- All positions in column `j`
- All positions on the diagonal where `row + col = i + j` (positive diagonal)
- All positions on the diagonal where `row - col = i - j` (negative diagonal)

We can use three boolean arrays to track:
- `cols[j]`: whether column `j` is occupied
- `dg[i+j]`: whether the positive diagonal passing through `(i,j)` is occupied
- `udg[i-j+n]`: whether the negative diagonal passing through `(i,j)` is occupied (we add `n` to avoid negative indices)

When we can't place a queen in any column of the current row (all positions conflict), we backtrack to the previous row, remove the queen we placed there, and try the next column. This [backtracking](/problems/backtracking) ensures we explore all possible valid configurations systematically.

The beauty of this approach is that we're essentially building a decision tree where each level represents a row, and each branch represents choosing a column for that row's queen. We traverse this tree depth-first, counting each path that reaches the bottom (all n queens successfully placed).

**Learn more about [Backtracking](/problems/backtracking) patterns.**

## Solution Approach

The implementation uses a depth-first search (DFS) with [backtracking](/problems/backtracking) to systematically explore all possible queen placements.

**Core Algorithm Structure:**

The solution defines a recursive function `dfs(i)` that attempts to place queens starting from row `i`. The base case occurs when `i == n`, meaning we've successfully placed queens in all n rows, so we increment our answer counter.

**Data Structures:**

Three boolean arrays track the attack zones:
- `cols[j]`: Tracks if column `j` has a queen
- `dg[a]`: Tracks if diagonal with sum `a = row + col` has a queen  
- `udg[b]`: Tracks if anti-diagonal with difference `b = row - col + n` has a queen

The arrays are sized to handle all possible diagonal indices:
- `cols` needs size `n`
- `dg` needs size `2n-1`, since `row + col` ranges from `0` to `2n-2`
- `udg` needs size `2n`, since `row - col + n` ranges from `1` to `2n-1`

The Python and TypeScript solutions allocate `n`, `2n`, and `2n`. The Java and C++ solutions use fixed sizes 10, 20, and 20, which is enough because LeetCode limits `n` to at most 9.

**Step-by-Step Process:**

1. Start with `dfs(0)` to begin placing queens from row 0.

2. For the current row `i`, iterate through each column `j` from 0 to n-1.

3. For each position `(i, j)`, calculate:
   - Diagonal index: `a = i + j`
   - Anti-diagonal index: `b = i - j + n` (adding `n` to ensure non-negative index)

4. Check if placing a queen at `(i, j)` causes a conflict:
   - If `cols[j]` or `dg[a]` or `udg[b]` is True, skip this column

5. If no conflict, place the queen:
   - Mark `cols[j] = dg[a] = udg[b] = True`
   - Recursively call `dfs(i + 1)` to place queens in the next row

6. After the recursive call returns (exploring all possibilities with this placement), backtrack:
   - Reset `cols[j] = dg[a] = udg[b] = False`
   - Continue to try the next column

7. The variable `ans` accumulates the count each time we successfully place all n queens (reach the base case).

**Why This Works:**

The algorithm ensures we explore every valid configuration exactly once. By processing row by row and using the conflict-checking arrays, we avoid generating invalid configurations. The [backtracking](/problems/backtracking) mechanism ensures we don't miss any valid solutions - when we hit a dead end, we systematically undo our choices and try alternatives.

The time complexity is O(n!), as in the worst case we might explore n choices for the first row, n-1 for the second, and so on. The space complexity is O(n) for the recursion stack and the tracking arrays.

## Example Walkthrough

Let's walk through finding all solutions for n=4 (a 4x4 chessboard).

**Initial Setup:**
- We have arrays: `cols[4]`, `dg[8]`, `udg[8]` (size `2n`) all initialized to False
- Start with `dfs(0)` to place queens from row 0
- Our answer counter `ans = 0`

**Step 1: Row 0, Try Column 0**
- Position (0,0): diagonal = 0+0 = 0, anti-diagonal = 0-0+4 = 4
- No conflicts (all arrays are False)
- Set `cols[0] = dg[0] = udg[4] = True`
- Call `dfs(1)`

**Step 2: Row 1, Try Columns**
- Column 0: (1,0) → diagonal=1, anti-diagonal=5
  - `cols[0]` is True → CONFLICT, skip
- Column 1: (1,1) → diagonal=2, anti-diagonal=4
  - `udg[4]` is True → CONFLICT, skip
- Column 2: (1,2) → diagonal=3, anti-diagonal=3
  - No conflicts! Set `cols[2] = dg[3] = udg[3] = True`
  - Call `dfs(2)`

**Step 3: Row 2, Try Columns**
- Column 0: `cols[0]` is True → CONFLICT
- Column 1: (2,1) → diagonal=3, anti-diagonal=5
  - `dg[3]` is True → CONFLICT
- Column 2: `cols[2]` is True → CONFLICT
- Column 3: (2,3) → diagonal=5, anti-diagonal=3
  - `udg[3]` is True → CONFLICT
- All columns conflict! Backtrack to Row 1

**Step 4: Backtrack to Row 1, Continue**
- Undo: `cols[2] = dg[3] = udg[3] = False`
- Column 3: (1,3) → diagonal=4, anti-diagonal=2
  - No conflicts! Set `cols[3] = dg[4] = udg[2] = True`
  - Call `dfs(2)`

**Step 5: Row 2, With New Configuration**
- Column 0: `cols[0]` is True → CONFLICT
- Column 1: (2,1) → diagonal=3, anti-diagonal=5
  - No conflicts! Set `cols[1] = dg[3] = udg[5] = True`
  - Call `dfs(3)`

**Step 6: Row 3, Try to Complete**
- Column 0: `cols[0]` is True → CONFLICT
- Column 1: `cols[1]` is True → CONFLICT
- Column 2: (3,2) → diagonal=5, anti-diagonal=5
  - `udg[5]` is True → CONFLICT
- Column 3: `cols[3]` is True → CONFLICT
- Dead end! Backtrack

**Continuing the Process:**
The algorithm continues backtracking and trying new combinations. Eventually, it finds the first valid solution when queens are placed at columns [1,3,0,2] for rows [0,1,2,3]:
```
. Q . .
. . . Q
Q . . .
. . Q .
```

When `dfs(4)` is reached (all 4 queens placed), `ans` increments to 1.

The algorithm then backtracks completely and continues exploring. It finds the second valid solution with queens at columns [2,0,3,1]:
```
. . Q .
Q . . .
. . . Q
. Q . .
```

After exploring all possibilities from row 0, the algorithm returns `ans = 2`.

**Key Observations:**
- The diagonal tracking prevents conflicts efficiently: when Queen at (0,0) is placed, any position with diagonal sum 0 or anti-diagonal difference 4 is immediately marked as conflicting
- Backtracking ensures we explore all branches: when we hit a dead end in row 2, we return to row 1 and try the next option
- The row-by-row approach guarantees exactly one queen per row, simplifying our search space

## Python Solution

```python
class Solution:
    def totalNQueens(self, n: int) -> int:
        def backtrack(row: int) -> None:
            # Base case: all queens have been successfully placed
            if row == n:
                nonlocal solution_count
                solution_count += 1
                return
          
            # Try placing a queen in each column of the current row
            for col in range(n):
                # Calculate diagonal and anti-diagonal indices
                # Diagonal: row + col (constant along each diagonal)
                # Anti-diagonal: row - col + n (add n to ensure positive index)
                diagonal_idx = row + col
                anti_diagonal_idx = row - col + n
              
                # Check if current position conflicts with existing queens
                if columns_used[col] or diagonals_used[diagonal_idx] or anti_diagonals_used[anti_diagonal_idx]:
                    continue
              
                # Place the queen by marking column and diagonals as used
                columns_used[col] = True
                diagonals_used[diagonal_idx] = True
                anti_diagonals_used[anti_diagonal_idx] = True
              
                # Recursively place queens in the next row
                backtrack(row + 1)
              
                # Backtrack: remove the queen and unmark the positions
                columns_used[col] = False
                diagonals_used[diagonal_idx] = False
                anti_diagonals_used[anti_diagonal_idx] = False
      
        # Initialize tracking arrays for column and diagonal conflicts
        columns_used = [False] * n  # Track which columns have queens
        diagonals_used = [False] * (2 * n)  # Track main diagonals (row + col)
        anti_diagonals_used = [False] * (2 * n)  # Track anti-diagonals (row - col + n)
      
        # Counter for total number of valid solutions
        solution_count = 0
      
        # Start the backtracking from the first row
        backtrack(0)
      
        return solution_count
```

## Time and Space Complexity

**Time Complexity:** `O(n!)`

The algorithm explores all possible placements of n queens on an n×n board. At the first row, we have `n` choices for placing a queen. Due to the constraints (no two queens can attack each other), the number of valid positions decreases as we place more queens. In the worst case, we have approximately `n × (n-2) × (n-4) × ...` possibilities, which gives us a time complexity of `O(n!)`. The backtracking with pruning (checking columns and diagonals) helps eliminate invalid branches early, but the worst-case complexity remains factorial.

**Space Complexity:** `O(n)`

The space complexity consists of:
- The recursion call stack depth, which goes up to `n` levels (one for each row): `O(n)`
- The `cols` array of size 10: `O(1)` since it's a fixed size regardless of input
- The `dg` (diagonal) array of size 20: `O(1)` since it's a fixed size
- The `udg` (anti-diagonal) array of size 20: `O(1)` since it's a fixed size
- Other variables (`ans`, `i`, `j`, `a`, `b`): `O(1)`

Note that while the arrays are fixed-size in this implementation (10, 20, 20), they should ideally be sized based on `n`. For a proper implementation, `cols` should be size `n`, `dg` at least `2n-1`, and `udg` at least `2n` (its index `row - col + n` reaches `2n-1`). Even with proper sizing, the space complexity would be `O(n) + O(n) + O(n) = O(n)`.

The dominant factor is the recursion stack depth of `O(n)`, giving us an overall space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Incorrect Diagonal Index Calculation

**The Pitfall:**
A frequent mistake is incorrectly calculating the diagonal indices, particularly for the anti-diagonal. Developers often use `row - col` directly without adjusting for negative indices, which causes array index out of bounds errors.

```python
# WRONG - can produce negative indices
anti_diagonal_idx = row - col  # When row=0, col=3, this gives -3!
```

**The Solution:**
Always add an offset to ensure non-negative indices for anti-diagonals:

```python
# CORRECT - ensures all indices are non-negative
anti_diagonal_idx = row - col + n - 1  # or row - col + n
```

### 2. Forgetting to Backtrack (Reset State)

**The Pitfall:**
After exploring a branch in the recursion tree, forgetting to reset the state arrays leads to incorrect results. The algorithm will think positions are occupied when they're not, missing valid solutions.

```python
# WRONG - missing backtrack step
columns_used[col] = True
diagonals_used[diagonal_idx] = True
anti_diagonals_used[anti_diagonal_idx] = True
backtrack(row + 1)
# Missing reset here!
```

**The Solution:**
Always reset the state after the recursive call returns:

```python
# CORRECT - proper backtracking
columns_used[col] = True
diagonals_used[diagonal_idx] = True
anti_diagonals_used[anti_diagonal_idx] = True

backtrack(row + 1)

# Reset state for next iteration
columns_used[col] = False
diagonals_used[diagonal_idx] = False
anti_diagonals_used[anti_diagonal_idx] = False
```

### 3. Incorrect Array Size for Diagonal Tracking

**The Pitfall:**
Using insufficient array sizes for diagonal tracking can cause index out of bounds errors. For an n×n board, `row + col` ranges up to `2n-2`, and the offset index `row - col + n` ranges up to `2n-1`.

```python
# WRONG - arrays too small
diagonals_used = [False] * n  # Not enough space!
anti_diagonals_used = [False] * n  # Not enough space!
```

**The Solution:**
Allocate `2n` slots for each diagonal array so both index formulas fit:

```python
# CORRECT - proper array sizing
diagonals_used = [False] * (2 * n)  # Safe size with slight overhead
anti_diagonals_used = [False] * (2 * n)  # Safe size with slight overhead
```

### 4. Using Global Variables Without Proper Scoping

**The Pitfall:**
Using a regular variable for the solution count inside the nested function won't work due to Python's scoping rules:

```python
# WRONG - UnboundLocalError
def totalNQueens(self, n: int) -> int:
    def backtrack(row: int):
        if row == n:
            solution_count += 1  # Error: local variable referenced before assignment
```

**The Solution:**
Use the `nonlocal` keyword to modify the outer scope variable:

```python
# CORRECT - using nonlocal
def totalNQueens(self, n: int) -> int:
    def backtrack(row: int):
        if row == n:
            nonlocal solution_count
            solution_count += 1
```

Alternatively, use a mutable container like a list:

```python
# ALTERNATIVE - using a list
def totalNQueens(self, n: int) -> int:
    solution_count = [0]  # Use list to avoid nonlocal
    def backtrack(row: int):
        if row == n:
            solution_count[0] += 1
```

