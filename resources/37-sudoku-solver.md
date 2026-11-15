# 37. Sudoku Solver

> Source: https://algo.monster/liteproblems/37

## Problem Statement

You need to write a program that solves a Sudoku puzzle by filling in all the empty cells on a 9x9 grid.

The puzzle is represented as a 9x9 board where:
- Each cell contains either a digit from `'1'` to `'9'` or a `'.'` character
- The `'.'` character represents an empty cell that needs to be filled

To solve the Sudoku puzzle correctly, your solution must satisfy these three rules:

1. **Row Rule**: Each digit from `1` to `9` must appear exactly once in every row
2. **Column Rule**: Each digit from `1` to `9` must appear exactly once in every column  
3. **Sub-box Rule**: Each digit from `1` to `9` must appear exactly once in each of the nine 3x3 sub-boxes that make up the grid

The 9x9 grid is divided into nine 3x3 sub-boxes like this:
- Top-left 3x3 box: rows 0-2, columns 0-2
- Top-middle 3x3 box: rows 0-2, columns 3-5
- Top-right 3x3 box: rows 0-2, columns 6-8
- And so on for the middle and bottom rows

Your task is to fill all empty cells (marked with `'.'`) with appropriate digits from `1` to `9` such that the completed board satisfies all Sudoku rules. The solution modifies the input board in-place rather than returning a new board.

## Intuition

When we look at solving a Sudoku puzzle manually, we typically start by finding empty cells and trying different numbers to see which ones work. If we place a number and later discover it leads to a conflict, we erase it and try a different number. This natural problem-solving approach directly translates to the [backtracking](/problems/backtracking) algorithm.

The key insight is that we need to keep track of three things simultaneously:
1. Which numbers have already been used in each row
2. Which numbers have already been used in each column  
3. Which numbers have already been used in each 3x3 sub-box

By maintaining this information, we can quickly check if placing a digit `v` at position `(i, j)` is valid by verifying:
- Has `v` already appeared in row `i`?
- Has `v` already appeared in column `j`?
- Has `v` already appeared in the 3x3 box containing `(i, j)`?

The 3x3 box index can be calculated using integer division: the box at position `(i // 3, j // 3)` contains the cell at `(i, j)`.

Rather than checking the entire board every time we want to validate a placement, we use boolean arrays to track used numbers. For example, `row[i][v]` tells us if digit `v+1` has been used in row `i`. This gives us O(1) validation checks.

The [backtracking](/problems/backtracking) process follows this pattern:
1. Find an empty cell
2. Try digits 1-9 in that cell
3. For each valid digit (not violating any rules), mark it as used in the corresponding row, column, and box
4. Recursively solve the rest of the puzzle
5. If we hit a dead end (no valid digits for a cell), backtrack by unmarking the digit and trying the next one
6. If we successfully fill all empty cells, we've found the solution

The elegance of this approach is that it mirrors human intuition while being systematic enough to guarantee finding a solution if one exists.

**Learn more about [Backtracking](/problems/backtracking) patterns.**

## Solution Approach

The implementation uses [backtracking](/problems/backtracking) with efficient state tracking through boolean arrays. Let's walk through the key components:

**Data Structures:**
- `row`: A 9x9 boolean array where `row[i][v]` indicates if digit `v+1` has been used in row `i`
- `col`: A 9x9 boolean array where `col[j][v]` indicates if digit `v+1` has been used in column `j`  
- `block`: A 3x3x9 boolean array where `block[i//3][j//3][v]` indicates if digit `v+1` has been used in the corresponding 3x3 sub-box
- `t`: A list storing the coordinates `(i, j)` of all empty cells that need to be filled
- `ok`: A flag to indicate when we've found a valid solution

**Initialization Phase:**
1. Create the three boolean tracking arrays initialized to `False`
2. Traverse the entire board once:
   - For empty cells (marked with `'.'`), add their coordinates to list `t`
   - For filled cells, mark the digit as used in the corresponding row, column, and block arrays
   - Convert digit character to index: `v = int(board[i][j]) - 1`

**DFS [Backtracking](/problems/backtracking) Function:**
The `dfs(k)` function processes the `k`-th empty cell in our list:

1. **Base Case**: If `k == len(t)`, we've successfully filled all empty cells, so set `ok = True` and return

2. **Recursive Case**: For the current empty cell at position `(i, j) = t[k]`:
   - Try each digit from 0 to 8 (representing digits 1-9)
   - Check validity: `row[i][v] == col[j][v] == block[i//3][j//3][v] == False`
   - If valid:
     - Mark the digit as used: `row[i][v] = col[j][v] = block[i//3][j//3][v] = True`
     - Place the digit on the board: `board[i][j] = str(v + 1)`
     - Recursively solve for the next empty cell: `dfs(k + 1)`
     - If not successful (ok is still False), backtrack by unmarking: `row[i][v] = col[j][v] = block[i//3][j//3][v] = False`
   - If `ok` becomes `True`, immediately return to preserve the solution

**Key Optimizations:**
1. Pre-collecting all empty cells avoids repeatedly searching the board
2. Using boolean arrays gives O(1) validity checks instead of scanning rows/columns/blocks
3. The early termination with `ok` flag prevents unnecessary [backtracking](/problems/backtracking) once a solution is found
4. Index calculation `i // 3` and `j // 3` efficiently maps any cell to its 3x3 block

The solution modifies the board in-place, so when `dfs(0)` completes, the board contains the solved Sudoku puzzle.

## Example Walkthrough

Let's walk through the backtracking approach on a simplified 4x4 mini-Sudoku (digits 1-4, 2x2 sub-boxes). The algorithm is identical to the 9x9 version, just with fewer digits to try.

```
Initial board:
[., 4, 3, .]
[3, ., 2, 4]
[4, ., ., 3]
[., 3, 4, .]
```

**Step 1: Initialization**
- Scan the board and mark the existing digits in the row, column, and block arrays. For example, row 0 has `{3, 4}`, column 0 has `{3, 4}`, and the top-left block has `{3, 4}`.
- Collect the empty cells in row-major order: `[(0,0), (0,3), (1,1), (2,1), (2,2), (3,0), (3,3)]`

**Step 2: Cell (0,0)**
- Try 1: not in row 0, column 0, or the top-left block ✓
- Place 1 at (0,0), mark it used, and recurse to the next empty cell

**Step 3: Cell (0,3) — dead end**
- Try 1: already in row 0 (we just placed it) ❌
- Try 2: already in the top-right block (from `board[1][2]`) ❌
- Try 3: already in row 0 ❌
- Try 4: already in row 0 ❌
- No digit fits, so this call returns without setting the `solved` flag

**Step 4: Backtrack to cell (0,0)**
- Undo the 1: `row[0][0] = col[0][0] = block[0][0][0] = False`
- Try 2: not in row 0, column 0, or the top-left block ✓
- Place 2 at (0,0) and recurse again

**Step 5: Continue forward**
- (0,3): 1 is the only digit missing from row 0 and it fits the column and block ✓
- (1,1): 1 is the only digit missing from row 1 ✓
- (2,1): 1 is already in column 1, so place 2 ✓
- (2,2): 1 ✓
- (3,0): 1 ✓
- (3,3): 1 conflicts with row 3, so place 2 ✓

**Step 6: All empty cells filled**
- `dfs(7)` sees `k == len(t)`, sets `solved = True`, and every caller returns immediately without undoing its placement.

```
Final solution:
[2, 4, 3, 1]
[3, 1, 2, 4]
[4, 2, 1, 3]
[1, 3, 4, 2]
```

This example demonstrates:
1. How we track used digits in rows, columns, and blocks
2. The systematic trial of digits in increasing order (1-4 here, 1-9 in the real problem)
3. The backtracking mechanism: placing 1 at (0,0) leaves no valid digit for (0,3), so the algorithm undoes it and tries 2
4. How marking/unmarking keeps the tracking arrays consistent with the board
5. How the `solved` flag stops the search as soon as the last empty cell is filled

For a full 9x9 Sudoku, the same process applies with digits 1-9 and 3x3 sub-boxes, just with more cells to fill and more backtracking branches to explore.

## Python Solution

```python
class Solution:
    def solveSudoku(self, board: List[List[str]]) -> None:
        """
        Solves a Sudoku puzzle using backtracking.
        Modifies the board in-place.
        """
        def backtrack(index):
            """
            Recursively fill empty cells using DFS backtracking.
          
            Args:
                index: Current index in the empty_cells list
            """
            nonlocal solved
          
            # Base case: all empty cells have been filled successfully
            if index == len(empty_cells):
                solved = True
                return
          
            # Get the current empty cell's position
            row_idx, col_idx = empty_cells[index]
          
            # Try each digit from 1 to 9
            for digit in range(9):
                # Check if placing digit+1 is valid in current position
                if (not row_used[row_idx][digit] and 
                    not col_used[col_idx][digit] and 
                    not block_used[row_idx // 3][col_idx // 3][digit]):
                  
                    # Mark the digit as used in row, column, and 3x3 block
                    row_used[row_idx][digit] = True
                    col_used[col_idx][digit] = True
                    block_used[row_idx // 3][col_idx // 3][digit] = True
                  
                    # Place the digit on the board
                    board[row_idx][col_idx] = str(digit + 1)
                  
                    # Recursively solve for the next empty cell
                    backtrack(index + 1)
                  
                    # If solution found, stop backtracking
                    if solved:
                        return
                  
                    # Backtrack: undo the current placement
                    row_used[row_idx][digit] = False
                    col_used[col_idx][digit] = False
                    block_used[row_idx // 3][col_idx // 3][digit] = False
      
        # Initialize tracking arrays for used digits
        # row_used[i][d] = True if digit d+1 is used in row i
        row_used = [[False] * 9 for _ in range(9)]
        # col_used[j][d] = True if digit d+1 is used in column j
        col_used = [[False] * 9 for _ in range(9)]
        # block_used[bi][bj][d] = True if digit d+1 is used in block (bi, bj)
        block_used = [[[False] * 9 for _ in range(3)] for _ in range(3)]
      
        # List to store positions of empty cells
        empty_cells = []
      
        # Flag to indicate if solution is found
        solved = False
      
        # Initialize the board state and collect empty cells
        for row_idx in range(9):
            for col_idx in range(9):
                if board[row_idx][col_idx] == '.':
                    # Record empty cell position
                    empty_cells.append((row_idx, col_idx))
                else:
                    # Mark existing digits as used
                    digit = int(board[row_idx][col_idx]) - 1
                    row_used[row_idx][digit] = True
                    col_used[col_idx][digit] = True
                    block_used[row_idx // 3][col_idx // 3][digit] = True
      
        # Start solving from the first empty cell
        backtrack(0)
```

## Time and Space Complexity

**Time Complexity:** `O(9^m)` where `m` is the number of empty cells, which in the worst case can be up to 81, giving us `O(9^81)`.

The algorithm uses backtracking to fill empty cells in the Sudoku board. For each empty cell stored in list `t`, we try all 9 possible values (1-9). In the worst case scenario where the board is completely empty, we have 81 empty cells. For each cell, we potentially explore 9 branches, and this branching continues for all empty cells. Although pruning occurs when a number violates Sudoku rules (checking `row[i][v]`, `col[j][v]`, and `block[i//3][j//3][v]`), the theoretical worst case still requires exploring up to `9^81` possibilities.

**Space Complexity:** `O(9^2)` or `O(81)`.

The space complexity consists of:
- Three 2D/3D boolean arrays for tracking used numbers:
  - `row`: `9 × 9 = 81` space
  - `col`: `9 × 9 = 81` space  
  - `block`: `3 × 3 × 9 = 81` space
- List `t` storing coordinates of empty cells: at most `81` pairs
- Recursion stack depth: at most `m` (number of empty cells), which is at most `81`

The total auxiliary space used is `O(81 + 81 + 81 + 81 + 81) = O(405) = O(9^2)`, which simplifies to `O(9^2)` or `O(81)` since these are constants.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Backtrack Properly**
One of the most common mistakes is not correctly undoing the state changes when backtracking. If you mark a digit as used but forget to unmark it when the path doesn't lead to a solution, the algorithm will incorrectly think that digit is permanently unavailable for that position.

**Incorrect Implementation:**
```python
# Missing backtrack cleanup
if valid:
    row_used[row_idx][digit] = True
    col_used[col_idx][digit] = True
    block_used[row_idx // 3][col_idx // 3][digit] = True
    board[row_idx][col_idx] = str(digit + 1)
    backtrack(index + 1)
    # Forgot to reset the states here!
```

**Correct Implementation:**
```python
if valid:
    # Mark as used
    row_used[row_idx][digit] = True
    col_used[col_idx][digit] = True
    block_used[row_idx // 3][col_idx // 3][digit] = True
    board[row_idx][col_idx] = str(digit + 1)
  
    backtrack(index + 1)
  
    # Only backtrack if solution not found
    if not solved:
        row_used[row_idx][digit] = False
        col_used[col_idx][digit] = False
        block_used[row_idx // 3][col_idx // 3][digit] = False
        # Note: We don't need to reset board[row_idx][col_idx] since it will be overwritten
```

### 2. **Incorrect Block Index Calculation**
Many people struggle with correctly mapping a cell (i, j) to its 3x3 block. Using the wrong formula will cause the algorithm to check the wrong block constraints.

**Common Mistakes:**
```python
# Wrong: Using modulo instead of integer division
block_used[row_idx % 3][col_idx % 3][digit]

# Wrong: Not dividing both indices
block_used[row_idx // 3][col_idx][digit]

# Wrong for this 3x3x9 array: a flat block index only works
# if block_used is declared as a 9x9 list instead
block_used[(row_idx // 3) * 3 + (col_idx // 3)][digit]
```

**Correct Formula:**
```python
# Correct: Both indices divided by 3
block_used[row_idx // 3][col_idx // 3][digit]
```

### 3. **Off-by-One Errors with Digit Indexing**
Since Sudoku uses digits 1-9 but array indices start at 0, it's easy to mix up the conversions between digit values and array indices.

**Common Mistake:**
```python
# Wrong: Using digit directly as index
digit = int(board[i][j])  # This is 1-9
row_used[i][digit] = True  # Index out of bounds when digit = 9

# Wrong: Forgetting to add 1 when placing on board
board[row_idx][col_idx] = str(digit)  # Places 0-8 instead of 1-9
```

**Correct Approach:**
```python
# Reading from board: subtract 1 to get index
digit_index = int(board[i][j]) - 1  # Convert 1-9 to 0-8

# Writing to board: add 1 to get actual digit
board[row_idx][col_idx] = str(digit_index + 1)  # Convert 0-8 to 1-9
```

### 4. **Not Stopping After Finding the First Solution**
Without proper termination, the backtracking might continue searching even after finding a valid solution, potentially overwriting it or wasting computation time.

**Problematic Pattern:**
```python
def backtrack(index):
    if index == len(empty_cells):
        # Found solution but continue searching
        return True  # Return value gets ignored
  
    for digit in range(9):
        if valid:
            # ... make changes ...
            backtrack(index + 1)  # Doesn't check return value
            # ... undo changes ...
```

**Proper Solution:**
```python
def backtrack(index):
    nonlocal solved
  
    if index == len(empty_cells):
        solved = True  # Set flag
        return
  
    for digit in range(9):
        if valid:
            # ... make changes ...
            backtrack(index + 1)
          
            if solved:  # Check flag immediately
                return  # Stop searching
          
            # ... undo changes ...
```

### 5. **Modifying the Board During Initialization**
Some implementations try to be clever by filling obvious cells during initialization, but this can lead to inconsistencies if not done carefully with the tracking arrays.

**Better Approach:**
Keep initialization simple - just record the initial state and let the backtracking algorithm handle all the filling logic uniformly.

