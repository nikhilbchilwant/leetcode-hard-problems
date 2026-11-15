# 51. N-Queens

> Source: https://algo.monster/liteproblems/51

## Problem Statement

The n-queens puzzle asks you to place `n` queens on an `n x n` chessboard such that no two queens can attack each other.

In chess, a queen can attack any piece that is:
- In the same row
- In the same column  
- On the same diagonal (both main diagonal and anti-diagonal)

Given an integer `n`, you need to find all possible valid arrangements of `n` queens on the board.

Each solution should be represented as a list of strings where:
- Each string represents one row of the chessboard
- `'Q'` indicates a queen's position
- `'.'` indicates an empty square

For example, if `n = 4`, one valid solution would be:
```
[".Q..",
 "...Q",
 "Q...",
 "..Q."]
```

This represents a 4x4 board where queens are placed at positions (0,1), (1,3), (2,0), and (3,2), and no two queens can attack each other.

The function should return all distinct valid board configurations. The order of solutions doesn't matter.

## Intuition

When we think about placing queens on a chessboard, we need to ensure no two queens can attack each other. Since we need to place `n` queens on an `n x n` board, and queens can attack any piece in their row, we know that **each row must contain exactly one queen**.

This observation immediately simplifies our problem - instead of considering all possible positions on the board, we can process the board row by row, deciding which column to place the queen in for each row.

The key insight is that when we place a queen at position `(i, j)`, it "blocks" certain positions for future queens:
- **Column `j`** is blocked for all subsequent rows
- **The diagonal** going through `(i, j)` is blocked
- **The anti-diagonal** going through `(i, j)` is blocked

To track these constraints efficiently, we can use a clever indexing trick:
- For columns: simply use index `j`
- For diagonals: cells on the same diagonal share the same value of `i + j`
- For anti-diagonals: cells on the same anti-diagonal share the same value of `n - i + j` (or `i - j + n - 1`)

This means we can use three arrays to mark which columns and diagonals are already occupied by queens.

The [backtracking](/problems/backtracking) approach naturally emerges from this:
1. Start with row 0
2. Try placing a queen in each column of the current row
3. Check if the position is safe (column and both diagonals are free)
4. If safe, mark the position as occupied and move to the next row
5. If we successfully place queens in all rows, we found a solution
6. If we can't place a queen in any column of the current row, backtrack to the previous row and try the next column

The beauty of this approach is that by processing row by row, we automatically ensure no two queens are in the same row, and our tracking arrays ensure no conflicts in columns or diagonals.

**Learn more about [Backtracking](/problems/backtracking) patterns.**

## Solution Approach

The implementation uses a depth-first search (DFS) with [backtracking](/problems/backtracking) to systematically explore all valid queen placements.

**Data Structures Used:**
- `g`: A 2D list representing the chessboard, initialized with all `'.'` (empty squares)
- `col`: An array of size `n` to track which columns have queens (`col[j] = 1` if column `j` has a queen)
- `dg`: An array of size `2n` for diagonal tracking (using index `i + j`)
- `udg`: An array of size `2n` for anti-diagonal tracking (using index `n - i + j`)
- `ans`: List to store all valid board configurations

**Core Algorithm - DFS Function:**

The `dfs(i)` function places queens starting from row `i`:

1. **Base Case**: If `i == n`, we've successfully placed all `n` queens. Convert the current board state `g` into the required string format and add it to `ans`.

2. **Recursive Case**: For the current row `i`, try placing a queen in each column `j` from 0 to `n-1`:
 
   - **Check if position is safe**: Verify that `col[j] + dg[i + j] + udg[n - i + j] == 0`
     - This clever check ensures the column, diagonal, and anti-diagonal are all free (all values are 0)
 
   - **Place the queen**: If safe, update the board and tracking arrays:
     ```python
     g[i][j] = "Q"
     col[j] = dg[i + j] = udg[n - i + j] = 1
     ```
 
   - **Recurse**: Call `dfs(i + 1)` to place queens in the next row
 
   - **Backtrack**: After recursion returns, restore the original state:
     ```python
     col[j] = dg[i + j] = udg[n - i + j] = 0
     g[i][j] = "."
     ```

**Why the Diagonal Indexing Works:**
- **`dg` diagonals** (`i + j`): Moving one step down-left (row `+1`, column `-1`) keeps `i + j` unchanged, so all cells on a line running from top-right to bottom-left share the same sum. Range is `[0, 2n-2]`.
- **`udg` diagonals** (`n - i + j`): Moving one step down-right (row `+1`, column `+1`) keeps `j - i` unchanged, so all cells on a line running from top-left to bottom-right share the same difference. Adding `n` makes every index positive; the range is `[1, 2n-1]`.

**Time Complexity**: `O(n!)` - In the worst case, we explore all possible placements. The first queen has `n` choices, the second has at most `n-2`, and so on.

**Space Complexity**: `O(n)` for the recursion stack depth, plus `O(n²)` for storing the board state.

The solution elegantly combines the constraint tracking with [backtracking](/problems/backtracking) to efficiently find all valid n-queens configurations without redundant checks.

## Example Walkthrough

Let's walk through solving the 4-queens problem step by step.

We start with a 4×4 board and need to place 4 queens. Our tracking arrays are:
- `col[4]` = [0,0,0,0] (tracks columns)
- `dg[8]` = [0,0,0,0,0,0,0,0] (tracks diagonals using i+j)
- `udg[8]` = [0,0,0,0,0,0,0,0] (tracks anti-diagonals using n-i+j)

**Step 1: Row 0, Try column 0**
- Check position (0,0): col[0]=0, dg[0+0]=0, udg[4-0+0]=0 ✓ All clear!
- Place queen at (0,0): Set col[0]=1, dg[0]=1, udg[4]=1
- Board state: `Q...`
- Move to row 1

**Step 2: Row 1, Try columns**
- Column 0: col[0]=1 ✗ (blocked by queen at row 0)
- Column 1: col[1]=0, dg[1+1]=0, udg[4-1+1]=1 ✗ (anti-diagonal blocked)
- Column 2: col[2]=0, dg[1+2]=0, udg[4-1+2]=0 ✓ All clear!
- Place queen at (1,2): Set col[2]=1, dg[3]=1, udg[5]=1
- Board state: `Q...` / `..Q.`
- Move to row 2

**Step 3: Row 2, Try columns**
- Column 0: col[0]=1 ✗ (blocked)
- Column 1: col[1]=0, dg[2+1]=1 ✗ (diagonal blocked by queen at (1,2))
- Column 2: col[2]=1 ✗ (blocked)
- Column 3: col[3]=0, dg[2+3]=0, udg[4-2+3]=1 ✗ (anti-diagonal blocked by queen at (1,2))
- No valid position! **Backtrack to row 1**

**Step 4: Back to Row 1, continue from column 3**
- Remove queen from (1,2): Reset col[2]=0, dg[3]=0, udg[5]=0
- Column 3: col[3]=0, dg[1+3]=0, udg[4-1+3]=0 ✓ All clear!
- Place queen at (1,3): Set col[3]=1, dg[4]=1, udg[6]=1
- Board state: `Q...` / `...Q`

**Step 5: Row 2, Try columns**
- Column 0: col[0]=1 ✗
- Column 1: col[1]=0, dg[2+1]=0, udg[4-2+1]=0 ✓ All clear!
- Place queen at (2,1): Set col[1]=1, dg[3]=1, udg[3]=1
- Board state: `Q...` / `...Q` / `.Q..`

**Step 6: Row 3, Try columns**
- Column 0: col[0]=1 ✗
- Column 1: col[1]=1 ✗
- Column 2: col[2]=0, dg[3+2]=0, udg[4-3+2]=1 ✗ (anti-diagonal blocked)
- Column 3: col[3]=1 ✗
- No valid position! **Backtrack to row 2**

This process continues, exploring different combinations through backtracking. Eventually, we find a valid solution when:
- Queen at (0,1): col[1]=1, dg[1]=1, udg[5]=1
- Queen at (1,3): col[3]=1, dg[4]=1, udg[6]=1  
- Queen at (2,0): col[0]=1, dg[2]=1, udg[2]=1
- Queen at (3,2): col[2]=1, dg[5]=1, udg[3]=1

Final solution:
```
.Q..
...Q
Q...
..Q.
```

The algorithm continues this backtracking process until all possibilities are explored, finding both solutions for the 4-queens problem.

## Python Solution

```python
class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        def backtrack(row: int) -> None:
            """
            Recursively place queens row by row using backtracking.
          
            Args:
                row: Current row index to place a queen
            """
            # Base case: all queens are successfully placed
            if row == n:
                # Convert the board to the required string format and add to results
                solution = ["".join(row_chars) for row_chars in board]
                results.append(solution)
                return
          
            # Try placing a queen in each column of the current row
            for col in range(n):
                # Check if placing queen at (row, col) is safe
                if (columns_used[col] == 0 and 
                    diagonals_used[row + col] == 0 and 
                    anti_diagonals_used[n - row + col] == 0):
                  
                    # Place the queen
                    board[row][col] = "Q"
                  
                    # Mark the column and diagonals as occupied
                    columns_used[col] = 1
                    diagonals_used[row + col] = 1
                    anti_diagonals_used[n - row + col] = 1
                  
                    # Recursively place queens in the next row
                    backtrack(row + 1)
                  
                    # Backtrack: remove the queen and unmark the positions
                    columns_used[col] = 0
                    diagonals_used[row + col] = 0
                    anti_diagonals_used[n - row + col] = 0
                    board[row][col] = "."
      
        # Initialize the result list to store all valid solutions
        results = []
      
        # Initialize the chess board with all empty cells
        board = [["."] * n for _ in range(n)]
      
        # Track which columns are occupied by queens
        columns_used = [0] * n
      
        # Track which diagonals are occupied (row + col identifies each diagonal)
        # Maximum possible value is (n-1) + (n-1) = 2n-2, so we need 2n-1 elements
        diagonals_used = [0] * (2 * n)
      
        # Track which anti-diagonals are occupied (n - row + col identifies each anti-diagonal)
        # Range is from n - (n-1) + 0 = 1 to n - 0 + (n-1) = 2n-1, so we need 2n elements
        anti_diagonals_used = [0] * (2 * n)
      
        # Start the backtracking from row 0
        backtrack(0)
      
        return results
```

## Time and Space Complexity

**Time Complexity:** `O(n² × n!)`

The algorithm uses backtracking to explore all possible queen placements. For each valid solution found, there are several operations:
- The recursive function `dfs` explores at most `n!` valid board configurations (upper bound on the number of ways to place n queens)
- At each recursive level `i`, we iterate through `n` columns to try placing a queen
- When a complete solution is found (`i == n`), we construct the answer by iterating through the `n × n` board, which takes `O(n²)` time
- The total time complexity is `O(n × n!)` for exploring all configurations, but since we need `O(n²)` time to construct each solution, the overall complexity becomes `O(n² × n!)`

**Space Complexity:** `O(n²)` auxiliary space for this implementation, excluding the output list.

The space usage consists of:
- The recursion stack depth is at most `n` (one recursive call per row): `O(n)`
- The `col` array of size `n`: `O(n)`
- The `dg` (diagonal) array of size `2n`: `O(n)`
- The `udg` (anti-diagonal) array of size `2n`: `O(n)`
- The board `g` is `n × n`: `O(n²)`
- The answer list `ans` stores the solutions but is not counted in auxiliary space complexity

The tracking arrays and recursion stack are `O(n)`, but the maintained board dominates the auxiliary space at `O(n²)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Diagonal Indexing Formula**

**Pitfall:** One of the most common mistakes is using incorrect formulas for diagonal indexing, such as:
- Using `row - col` directly as an array index (it is negative whenever `col > row`; in Python a negative index silently reads from the end of the list)
- Using `row + col + n` for the second diagonal family (this just shifts the `row + col` family and still never detects ↘ conflicts)
- Mixing up the formulas for diagonals and anti-diagonals

**Why it happens:** The diagonal indexing is counterintuitive. Without understanding why `row + col` and `n - row + col` work, developers often guess at formulas.

**Solution:** 
- For diagonals running ↙ (top-right to bottom-left): Use `row + col`. Points on the same line have the same sum.
- For diagonals running ↘ (top-left to bottom-right): Use `n - row + col` or `col - row + n - 1`. Points on the same line have the same difference `col - row`, and the offset keeps every index non-negative.
- Draw a small grid and verify your indexing manually before coding.

### 2. **Forgetting to Backtrack Properly**

**Pitfall:** Failing to reset the state after exploring a branch:
```python
# Incorrect - forgetting to reset the board
board[row][col] = "Q"
columns_used[col] = 1
diagonals_used[row + col] = 1
anti_diagonals_used[n - row + col] = 1
backtrack(row + 1)
# Missing the backtrack step!
```

**Why it happens:** It's easy to focus on the forward recursion and forget that backtracking requires undoing changes.

**Solution:** Always ensure every state change has a corresponding reset:
```python
# Place queen
board[row][col] = "Q"
columns_used[col] = 1
diagonals_used[row + col] = 1
anti_diagonals_used[n - row + col] = 1

# Explore
backtrack(row + 1)

# Backtrack - undo all changes in reverse order
anti_diagonals_used[n - row + col] = 0
diagonals_used[row + col] = 0
columns_used[col] = 0
board[row][col] = "."
```

### 3. **Creating Board Representation Incorrectly**

**Pitfall:** Creating the board with shared references:
```python
# Incorrect - all rows reference the same list!
board = [["."] * n] * n

# When you modify board[0][0], all board[i][0] change
```

**Why it happens:** Python's list multiplication creates references, not copies, for mutable objects.

**Solution:** Use list comprehension to create independent rows:
```python
# Correct - each row is a separate list
board = [["."] * n for _ in range(n)]
```

### 4. **Converting Board to Output Format Incorrectly**

**Pitfall:** Forgetting to create a deep copy of the board state when saving solutions:
```python
# Incorrect - appending references to the same board
if row == n:
    results.append(board)  # This saves a reference, not a copy!
```

**Why it happens:** The board array is reused throughout the recursion. Saving references means all solutions will show the final board state.

**Solution:** Convert the board to strings immediately when saving:
```python
# Correct - create new string representations
if row == n:
    solution = ["".join(row_chars) for row_chars in board]
    results.append(solution)
```

### 5. **Off-by-One Errors in Array Sizing**

**Pitfall:** Incorrectly sizing the tracking arrays:
```python
# Incorrect - too small for diagonal tracking
diagonals_used = [0] * n  # Should be 2*n
anti_diagonals_used = [0] * n  # Should be 2*n
```

**Why it happens:** Not calculating the maximum possible index values. For an n×n board:
- `row + col` ranges from 0 to 2n-2
- `n - row + col` ranges from 1 to 2n-1

**Solution:** Always allocate `2*n` elements for diagonal tracking arrays to safely cover all possible indices:
```python
diagonals_used = [0] * (2 * n)
anti_diagonals_used = [0] * (2 * n)
```

