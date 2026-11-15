# 1240. Tiling a Rectangle with the Fewest Squares

> Source: https://algo.monster/liteproblems/1240

## Problem Statement

You are given a rectangle with dimensions `n x m`. Your task is to completely cover this rectangle using square tiles, where each square tile must have integer side lengths. The goal is to find the minimum number of square tiles needed to tile the entire rectangle.

For example, if you have a 2x3 rectangle, you could tile it with 6 squares of size 1x1, but a better solution would be to use 2 squares of size 2x2 and 2 squares of size 1x1, for a total of 3 squares.

The squares can be of different sizes, and you need to arrange them in such a way that:
- The entire rectangle is covered
- There are no overlapping squares
- There are no gaps between squares
- The total number of squares used is minimized

The function should return the minimum number of integer-sided squares required to tile the given `n x m` rectangle.

## Intuition

When faced with tiling a rectangle with the minimum number of squares, we need to think about how to systematically explore all possible ways to place squares.

The key insight is that we can process the rectangle position by position, from left to right and top to bottom. At each empty position, we have choices - we can place squares of different sizes starting from that position. For instance, if we're at position `(i, j)` and have a 3x3 empty space available, we could place a 1x1, 2x2, or 3x3 square there.

Why [backtracking](/problems/backtracking)? Because we don't know which choice will lead to the optimal solution until we try them all. After placing a square, we move to the next empty position and repeat the process. If we reach a configuration that uses more squares than our current best solution, we can stop exploring that branch (pruning). When we finish one path, we backtrack by removing the last placed square and trying a different size.

The state compression technique comes from realizing that we only need to know which cells are filled or empty. Since each cell has only two states (filled/empty), we can use bits to represent them. For each row, we use an integer where the `j`-th bit indicates whether position `(i, j)` is filled. This makes checking and updating the state very efficient using bitwise operations.

The recursive approach naturally handles the exploration: 
- Start from position `(0, 0)`
- For each empty position, try all possible square sizes that fit
- Mark the covered cells, recurse to handle the rest
- When [backtracking](/problems/backtracking), unmark those cells to try other options
- Keep track of the minimum number of squares needed

This exhaustive search with pruning ensures we find the optimal solution despite the exponential search space, which is manageable due to the small constraints.

**Learn more about [Backtracking](/problems/backtracking) patterns.**

## Solution Approach

The implementation uses recursive [backtracking](/problems/backtracking) with state compression to efficiently track which cells have been filled.

**State Representation:**
- We use an integer array `filled` of length `n`, where `filled[i]` represents the state of row `i`
- If the `j`-th bit of `filled[i]` is 1, it means position `(i, j)` is filled
- This allows us to check if a cell is filled using: `filled[i] >> j & 1`

**Main DFS Function:**
The `dfs(i, j, t)` function explores all possible tilings starting from position `(i, j)` with `t` squares already placed.

1. **Base Cases:**
   - If `j == m`: We've reached the end of a row, move to the next row by calling `dfs(i + 1, 0, t)`
   - If `i == n`: All positions are filled, update the answer with `t`

2. **Skip Filled Positions:**
   - If position `(i, j)` is already filled (`filled[i] >> j & 1`), recurse to the next position

3. **Try Different Square Sizes:**
   - First, find the maximum square size possible from position `(i, j)`:
     - Count consecutive empty cells to the right (`c`)
     - Count consecutive empty cells downward (`r`)
     - Maximum square size is `mx = min(r, c)`
 
4. **Place and Recurse:**
   - For each possible square size `w` from 1 to `mx`:
     - Mark all cells in the `w × w` square as filled using bitwise OR operations:
       ```
       for k in range(w):
           filled[i + w - 1] |= 1 << (j + k)  # Fill bottom row of square
           filled[i + k] |= 1 << (j + w - 1)  # Fill right column of square
       ```
     - Recurse to the next position: `dfs(i, j + w, t + 1)`

5. **Backtrack:**
   - After exploring, restore the state by unmarking all cells in the square:
     ```
     for x in range(i, i + mx):
         for y in range(j, j + mx):
             filled[x] ^= 1 << y  # XOR to flip bits back
     ```

**Pruning:**
The condition `t + 1 < ans` ensures we only explore paths that could potentially improve our answer, pruning branches that already use too many squares.

**Initialization:**
- Start with `ans = n * m` (worst case: all 1×1 squares)
- Initialize `filled = [0] * n` (all cells empty)
- Call `dfs(0, 0, 0)` to start from the top-left corner with 0 squares placed

The algorithm systematically tries all valid square placements, using bit manipulation for efficient state tracking and pruning to avoid unnecessary exploration.

## Example Walkthrough

Let's walk through tiling a 2×3 rectangle to understand the solution approach.

**Initial Setup:**
- Rectangle dimensions: n=2, m=3
- `filled = [0, 0]` (binary: `[000, 000]` - all cells empty)
- Start with `ans = 6` (worst case: six 1×1 squares)
- Call `dfs(0, 0, 0)` - start at position (0,0) with 0 squares placed

**Step 1: First Square Placement**
At position (0,0):
- Check maximum square size: 
  - Consecutive empty cells right: 3
  - Consecutive empty cells down: 2
  - Maximum square size: min(2,3) = 2

Try placing a 2×2 square:
- Mark cells: `filled = [6, 6]` (binary: `[110, 110]`)
- Visual state:
  ```
  [X X .]
  [X X .]
  ```
- Recurse to `dfs(0, 2, 1)` - move to position (0,2) with 1 square placed

**Step 2: Second Square at (0,2)**
At position (0,2):
- Maximum square size possible: 1×1 (only one column left)
- Place 1×1 square at (0,2):
  - Update: `filled[0] |= 1 << 2` → `filled = [7, 6]` (binary: `[111, 110]`)
  - Visual state:
    ```
    [X X X]
    [X X .]
    ```
- Recurse to `dfs(0, 3, 2)` - reached end of row

**Step 3: Move to Next Row**
Since j=3 (end of row), move to `dfs(1, 0, 2)`

**Step 4: Process Row 1**
At position (1,0):
- Cell already filled (bit check: `filled[1] >> 0 & 1 = 0`), skip to (1,1)

At position (1,1):
- Cell already filled, skip to (1,2)

At position (1,2):
- Empty cell found, place 1×1 square:
  - Update: `filled[1] |= 1 << 2` → `filled = [7, 7]` (binary: `[111, 111]`)
  - Visual state:
    ```
    [X X X]
    [X X X]
    ```
- Recurse to `dfs(1, 3, 3)`

**Step 5: Complete First Solution**
- Reached i=2 (all rows processed)
- Update `ans = min(6, 3) = 3`
- Backtrack to try other configurations

**Step 6: Backtrack and Try Alternative**
Back at position (0,0), now try a 1×1 square instead:
- Mark only (0,0): `filled = [1, 0]` (binary: `[001, 000]`)
- Visual state:
  ```
  [X . .]
  [. . .]
  ```
- Continue exploring...

This process would continue, trying different square sizes at each position. For example:
- 1×1 at (0,0), then 2×2 at (0,1), then 1×1 at (1,0) → 3 squares total
- Six 1×1 squares → 6 squares total

The algorithm explores all possibilities while pruning paths that exceed the current best answer. The bit manipulation (`filled[i] >> j & 1`) efficiently checks if cells are occupied, and the backtracking ensures we try all valid configurations to find the minimum of 3 squares.

## Python Solution

```python
class Solution:
    def tilingRectangle(self, n: int, m: int) -> int:
        def backtrack(row: int, col: int, num_squares: int) -> None:
            """
            Recursively try to fill the rectangle with squares using backtracking.
          
            Args:
                row: Current row position
                col: Current column position  
                num_squares: Number of squares used so far
            """
            nonlocal min_squares
          
            # Move to next row if we've reached the end of current row
            if col == m:
                row += 1
                col = 0
          
            # If we've filled all rows, update the minimum answer
            if row == n:
                min_squares = num_squares
                return
          
            # If current cell is already filled, move to next cell
            if filled_cells[row] >> col & 1:
                backtrack(row, col + 1, num_squares)
            # Only continue if we can potentially get a better solution
            elif num_squares + 1 < min_squares:
                # Find maximum possible square size from current position
                max_rows = 0
                max_cols = 0
              
                # Count empty rows from current position
                for k in range(row, n):
                    if filled_cells[k] >> col & 1:
                        break
                    max_rows += 1
              
                # Count empty columns from current position
                for k in range(col, m):
                    if filled_cells[row] >> k & 1:
                        break
                    max_cols += 1
              
                # Maximum square size is limited by available rows and columns
                max_square_size = min(max_rows, max_cols)
              
                # Try all possible square sizes from 1 to max_square_size
                for square_size in range(1, max_square_size + 1):
                    # Mark cells as filled for current square
                    for k in range(square_size):
                        # Fill bottom row of the square
                        filled_cells[row + square_size - 1] |= 1 << (col + k)
                        # Fill rightmost column of the square
                        filled_cells[row + k] |= 1 << (col + square_size - 1)
                  
                    # Recursively try to fill remaining cells
                    backtrack(row, col + square_size, num_squares + 1)
              
                # Backtrack: unmark all cells that were filled
                for x in range(row, row + max_square_size):
                    for y in range(col, col + max_square_size):
                        filled_cells[x] ^= 1 << y
      
        # Initialize minimum squares to worst case (all 1x1 squares)
        min_squares = n * m
      
        # Track filled cells using bit manipulation (each row is a bitmask)
        filled_cells = [0] * n
      
        # Start backtracking from top-left corner
        backtrack(0, 0, 0)
      
        return min_squares
```

## Time and Space Complexity

**Time Complexity:** `O((n*m)^(n*m))`

The algorithm uses backtracking with pruning to explore all possible ways to tile an `n×m` rectangle with square tiles. In the worst case:
- Each cell in the `n×m` grid can be the starting point of a square tile
- For each position `(i,j)`, we can place squares of sizes from `1×1` up to `min(n-i, m-j)`
- The recursion tree has a maximum depth of `n*m` (if we place all `1×1` tiles)
- At each level, we potentially have multiple branching factors based on the square sizes we can place
- The state space is exponential as we explore different combinations of square placements

The pruning condition `t + 1 < ans` helps reduce the search space, but the worst-case remains exponential.

**Space Complexity:** `O(n + n*m)`

The space complexity consists of:
- `O(n)` for the `filled` array which uses `n` integers to represent the filled state of the grid using bitmasks
- `O(n*m)` for the recursion call stack in the worst case when the maximum recursion depth equals the number of cells
- Additional `O(1)` space for variables like `ans`, `r`, `c`, `mx`, etc.

The dominant factor is the recursion depth, giving us `O(n + n*m)` which simplifies to `O(n*m)` space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Bit Manipulation During Square Placement**

The most critical pitfall in this solution is the incorrect way of marking cells as filled when placing a square. The current code only marks the bottom row and rightmost column of the square, leaving the interior cells unmarked.

**Problem in Original Code:**
```python
# This only marks the edges, not the entire square!
for k in range(square_size):
    filled_cells[row + square_size - 1] |= 1 << (col + k)  # Bottom row only
    filled_cells[row + k] |= 1 << (col + square_size - 1)  # Right column only
```

For a 3×3 square, this would only mark:
```
. . X
. . X  
X X X
```
Instead of the entire square.

**Correct Solution:**
```python
# Mark ALL cells in the square as filled
for x in range(row, row + square_size):
    for y in range(col, col + square_size):
        filled_cells[x] |= 1 << y
```

### 2. **Inconsistent Backtracking Logic**

The backtracking section uses `max_square_size` instead of the actual `square_size` that was placed, leading to incorrect restoration of state.

**Problem:**
```python
# Uses max_square_size for ALL squares, regardless of which size was actually placed
for x in range(row, row + max_square_size):
    for y in range(col, col + max_square_size):
        filled_cells[x] ^= 1 << y
```

**Correct Solution:**
```python
# Backtrack should be inside the loop, specific to each square_size
for square_size in range(1, max_square_size + 1):
    # Mark cells
    for x in range(row, row + square_size):
        for y in range(col, col + square_size):
            filled_cells[x] |= 1 << y
  
    # Recurse
    backtrack(row, col + square_size, num_squares + 1)
  
    # Backtrack for THIS specific square_size
    for x in range(row, row + square_size):
        for y in range(col, col + square_size):
            filled_cells[x] ^= 1 << y
```

### 3. **Integer Overflow with Bit Operations**

For larger values of `m`, using bit shifting can cause issues since Python integers have practical limits for bit operations in certain contexts.

**Problem:**
When `m` is large (e.g., > 30), `1 << col` might behave unexpectedly in some Python implementations or be inefficient.

**Solution:**
Add validation or use alternative state representation for very large rectangles:
```python
def tilingRectangle(self, n: int, m: int) -> int:
    # Swap dimensions if needed to minimize bit operations
    if m > n:
        n, m = m, n
  
    if m > 30:  # Consider alternative approach for very wide rectangles
        # Use different state representation (e.g., 2D boolean array)
        pass
```

### Complete Corrected Code:
```python
class Solution:
    def tilingRectangle(self, n: int, m: int) -> int:
        def backtrack(row: int, col: int, num_squares: int) -> None:
            nonlocal min_squares
          
            if col == m:
                backtrack(row + 1, 0, num_squares)
                return
          
            if row == n:
                min_squares = min(min_squares, num_squares)
                return
          
            if filled_cells[row] >> col & 1:
                backtrack(row, col + 1, num_squares)
                return
              
            if num_squares + 1 >= min_squares:
                return
          
            # Find maximum square size
            max_size = min(n - row, m - col)
            for r in range(row, min(n, row + max_size)):
                for c in range(col, min(m, col + max_size)):
                    if filled_cells[r] >> c & 1:
                        max_size = min(max_size, max(r - row, c - col))
          
            # Try each square size
            for size in range(1, max_size + 1):
                # Mark all cells in the square
                for r in range(row, row + size):
                    for c in range(col, col + size):
                        filled_cells[r] |= 1 << c
              
                backtrack(row, col + size, num_squares + 1)
              
                # Unmark all cells in the square
                for r in range(row, row + size):
                    for c in range(col, col + size):
                        filled_cells[r] ^= 1 << c
      
        min_squares = n * m
        filled_cells = [0] * n
        backtrack(0, 0, 0)
        return min_squares
```

