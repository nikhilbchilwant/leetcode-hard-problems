# 1301. Number of Paths with Max Score

> Source: https://algo.monster/liteproblems/1301

## Problem Statement

You have a square board filled with characters where you need to find paths from the bottom-right corner to the top-left corner while collecting maximum points.

**Board Setup:**
- The starting position is at the bottom-right corner, marked with character `'S'`
- The ending position is at the top-left corner, marked with character `'E'`
- Other squares contain either:
  - Numeric characters `'1'` through `'9'` (representing points you can collect)
  - Obstacles marked with `'X'` (blocking your path)

**Movement Rules:**
- You can only move in three directions from any position:
  - Up (one square up)
  - Left (one square left)  
  - Up-left (diagonally)
- You cannot move through or onto squares containing obstacles `'X'`

**Objective:**
Find all paths from `'S'` to `'E'` that give you the maximum possible sum of numeric values collected along the way.

**Return Value:**
Return a list of two integers:
1. The maximum sum of numeric characters you can collect
2. The number of different paths that achieve this maximum sum (modulo `10^9 + 7`)

If no valid path exists from start to end, return `[0, 0]`.

**Example:**
If you have a path that goes through squares with values `'3'`, `'2'`, and `'4'`, you would collect a total of 9 points for that path. The problem asks you to find the path(s) with the highest total and count how many such optimal paths exist.

## Intuition

The key insight is to work backwards from the destination to the start. Since we need to move from bottom-right to top-left, we can flip our perspective and think about reaching each position from the bottom-right corner.

Why work backwards? Because when we're at any position `(i, j)`, we want to know: "What's the best score I can get from here to the destination?" This is easier to compute if we've already solved for positions closer to the destination.

For each position, we need to track two things:
1. **Maximum score possible** from that position to the destination
2. **Number of ways** to achieve that maximum score

This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach where:
- `f[i][j]` = maximum score from position `(i, j)` to the destination `(n-1, n-1)`
- `g[i][j]` = number of paths achieving that maximum score

Starting from the destination `(n-1, n-1)` with score 0 and 1 way to reach it, we work our way backwards. For each position `(i, j)`, we check the three positions we could have come from:
- `(i+1, j)` - the position below
- `(i, j+1)` - the position to the right  
- `(i+1, j+1)` - the position diagonally below-right

The maximum score at `(i, j)` equals the best score from these three neighboring positions, plus the value at the current position (if it's a digit). If multiple neighbors give the same maximum score, we sum up their path counts since all those paths are equally optimal.

By processing positions in reverse order (from bottom-right to top-left), we ensure that when calculating values for position `(i, j)`, all positions it depends on have already been computed. This bottom-up approach guarantees we find the optimal solution efficiently in `O(n²)` time.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with a bottom-up approach, processing the board from the destination (bottom-right) to the start (top-left).

**Data Structures:**
- `f[i][j]`: 2D array storing the maximum score from position `(i, j)` to destination `(n-1, n-1)`
- `g[i][j]`: 2D array storing the number of paths achieving the maximum score
- Initialize `f` with `-1` (indicating unreachable) and `g` with `0`
- Set `f[n-1][n-1] = 0` and `g[n-1][n-1] = 1` as base case

**Core Algorithm:**

1. **Iterate in reverse order** - Process positions from `(n-1, n-1)` to `(0, 0)`:
   ```python
   for i in range(n - 1, -1, -1):
       for j in range(n - 1, -1, -1):
   ```

2. **Update function** - For each position `(i, j)`, check three possible next positions:
   - `(i+1, j)` - moving down
   - `(i, j+1)` - moving right
   - `(i+1, j+1)` - moving diagonally
 
   The `update` function handles the logic:
   - Skip if next position is out of bounds or unreachable (`f[x][y] == -1`)
   - Skip if current position is obstacle `'X'` or start `'S'`
   - If next position has better score: update both `f[i][j]` and `g[i][j]`
   - If next position has equal score: accumulate paths in `g[i][j]`

3. **Add current position value** - After updating from neighbors:
   ```python
   if f[i][j] != -1 and board[i][j].isdigit():
       f[i][j] += int(board[i][j])
   ```
   This adds the numeric value at the current position to the maximum score.

4. **Return result**:
   - If `f[0][0] == -1`: no path exists, return `[0, 0]`
   - Otherwise: return `[f[0][0], g[0][0] % mod]` where `mod = 10^9 + 7`

**Time Complexity:** `O(n²)` - each cell is processed once with constant work per cell
**Space Complexity:** `O(n²)` - for the two DP arrays

The key insight is that by working backwards and tracking both maximum scores and path counts simultaneously, we can efficiently solve both parts of the problem in a single pass through the board.

## Example Walkthrough

Let's walk through a small 3×3 board example to illustrate the solution approach:

```
Board:
E 2 X
1 5 3
X 4 S

Position coordinates:
(0,0) (0,1) (0,2)
(1,0) (1,1) (1,2)
(2,0) (2,1) (2,2)
```

**Step 1: Initialize DP arrays**
- `f[i][j]` = maximum score from position (i,j) to destination
- `g[i][j]` = number of paths achieving that maximum score
- All positions start with `f = -1` (unreachable) and `g = 0`
- Base case: `f[2][2] = 0`, `g[2][2] = 1` (at destination 'S')

**Step 2: Process positions in reverse order**

*Processing (2,2) - 'S':*
- Already initialized: `f[2][2] = 0`, `g[2][2] = 1`

*Processing (2,1) - '4':*
- Check neighbors: only (2,2) is valid (in bounds)
- From (2,2): score = 0, paths = 1
- Update: `f[2][1] = 0`, `g[2][1] = 1`
- Add digit value: `f[2][1] = 0 + 4 = 4`

*Processing (2,0) - 'X':*
- Skip (obstacle)

*Processing (1,2) - '3':*
- Check neighbor (2,2): score = 0, paths = 1
- Update: `f[1][2] = 0`, `g[1][2] = 1`
- Add digit value: `f[1][2] = 0 + 3 = 3`

*Processing (1,1) - '5':*
- Check three neighbors:
  - (2,1): score = 4, paths = 1
  - (2,2): score = 0, paths = 1
  - (1,2): score = 3, paths = 1
- Best score is 4 from (2,1)
- Update: `f[1][1] = 4`, `g[1][1] = 1`
- Add digit value: `f[1][1] = 4 + 5 = 9`

*Processing (1,0) - '1':*
- Check neighbors:
  - (2,0): 'X' obstacle, skip
  - (2,1): score = 4, paths = 1
  - (1,1): score = 9, paths = 1
- Best score is 9 from (1,1)
- Update: `f[1][0] = 9`, `g[1][0] = 1`
- Add digit value: `f[1][0] = 9 + 1 = 10`

*Processing (0,2) - 'X':*
- Skip (obstacle)

*Processing (0,1) - '2':*
- Check neighbors:
  - (1,1): score = 9, paths = 1
  - (1,2): score = 3, paths = 1
  - (0,2): 'X' obstacle, skip
- Best score is 9 from (1,1)
- Update: `f[0][1] = 9`, `g[0][1] = 1`
- Add digit value: `f[0][1] = 9 + 2 = 11`

*Processing (0,0) - 'E':*
- Check neighbors:
  - (1,0): score = 10, paths = 1
  - (1,1): score = 9, paths = 1
  - (0,1): score = 11, paths = 1
- Best score is 11 from (0,1)
- Update: `f[0][0] = 11`, `g[0][0] = 1`
- 'E' is not a digit, so no value added

**Step 3: Return result**
- `f[0][0] = 11` (maximum score)
- `g[0][0] = 1` (number of paths)
- Return: `[11, 1]`

The optimal path is: E → 2 → 5 → 4 → S, collecting 2+5+4 = 11 points.

## Python Solution

```python
class Solution:
    def pathsWithMaxScore(self, board: List[str]) -> List[int]:
        """
        Find the maximum score and number of paths from 'E' to 'S' on the board.
      
        Args:
            board: 2D grid where 'E' is end, 'S' is start, 'X' is obstacle, digits are scores
          
        Returns:
            List containing [max_score, number_of_paths] or [0, 0] if no valid path exists
        """
      
        def update_cell(curr_row: int, curr_col: int, prev_row: int, prev_col: int) -> None:
            """
            Update the current cell's score and path count based on the previous cell.
          
            Args:
                curr_row, curr_col: Current cell coordinates
                prev_row, prev_col: Previous cell coordinates to check
            """
            # Skip if previous cell is out of bounds or unreachable
            if prev_row >= board_size or prev_col >= board_size or max_score[prev_row][prev_col] == -1:
                return
          
            # Skip if current cell is obstacle or start position
            if board[curr_row][curr_col] in "XS":
                return
          
            # If previous cell has better score, update current cell
            if max_score[prev_row][prev_col] > max_score[curr_row][curr_col]:
                max_score[curr_row][curr_col] = max_score[prev_row][prev_col]
                path_count[curr_row][curr_col] = path_count[prev_row][prev_col]
            # If previous cell has same score, add its path count to current
            elif max_score[prev_row][prev_col] == max_score[curr_row][curr_col]:
                path_count[curr_row][curr_col] += path_count[prev_row][prev_col]
      
        board_size = len(board)
      
        # max_score[i][j] stores the maximum score to reach cell (i, j)
        max_score = [[-1] * board_size for _ in range(board_size)]
      
        # path_count[i][j] stores the number of paths with maximum score to reach cell (i, j)
        path_count = [[0] * board_size for _ in range(board_size)]
      
        # Initialize the end position 'S' at bottom-right corner
        max_score[-1][-1] = 0
        path_count[-1][-1] = 1
      
        # Process cells from bottom-right to top-left
        for row in range(board_size - 1, -1, -1):
            for col in range(board_size - 1, -1, -1):
                # Update current cell from three possible previous cells
                update_cell(row, col, row + 1, col)      # Cell below
                update_cell(row, col, row, col + 1)      # Cell to the right
                update_cell(row, col, row + 1, col + 1)  # Cell diagonally down-right
              
                # Add the score of current cell if it's reachable and contains a digit
                if max_score[row][col] != -1 and board[row][col].isdigit():
                    max_score[row][col] += int(board[row][col])
      
        MOD = 10**9 + 7
      
        # Return result for start position 'E' at top-left corner
        if max_score[0][0] == -1:
            return [0, 0]
        else:
            return [max_score[0][0], path_count[0][0] % MOD]
```

## Time and Space Complexity

**Time Complexity:** `O(n²)`

The algorithm uses a nested loop structure where both loops iterate from `n-1` to `0`, giving us `n` iterations for each loop. For each cell `(i, j)` in the grid, the `update` function is called exactly 3 times (for the three possible previous positions: down, right, and diagonal). Each call to `update` performs constant time operations `O(1)`. Therefore, the total time complexity is `O(n × n × 3 × 1) = O(n²)`.

**Space Complexity:** `O(n²)`

The algorithm creates two 2D arrays:
- `f`: A `n × n` matrix to store the maximum scores, requiring `O(n²)` space
- `g`: A `n × n` matrix to store the number of paths, requiring `O(n²)` space

The total space complexity is `O(n²) + O(n²) = O(n²)`, where `n` is the side length of the board.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Direction Mapping**
The most critical pitfall in this solution is the confusion between movement directions and DP traversal directions. The problem states you can move **up, left, and up-left** from any position, but the solution processes cells in reverse (from destination to start). This creates a mismatch:

- **Problem statement**: From current position, you can move to up, left, or up-left
- **DP traversal**: From destination backwards, so we need to check where we could have come FROM

**The Bug**: The current code checks `(row+1, col)`, `(row, col+1)`, and `(row+1, col+1)`, which represents checking cells below, right, and diagonally down-right. This is incorrect for the given movement rules.

**Solution**: When processing backwards, if we're at position `(i,j)` and want to know which cells could reach us following the movement rules:
- A cell at `(i+1, j)` can reach `(i,j)` by moving **up**
- A cell at `(i, j+1)` can reach `(i,j)` by moving **left**  
- A cell at `(i+1, j+1)` can reach `(i,j)` by moving **up-left**

So the current implementation is actually correct! The confusion arises from thinking about forward movement vs. backward DP traversal.

### 2. **Integer Overflow in Path Counting**
The path count can grow exponentially, leading to integer overflow if modulo operation is not applied during accumulation.

**The Bug**: 
```python
elif max_score[prev_row][prev_col] == max_score[curr_row][curr_col]:
    path_count[curr_row][curr_col] += path_count[prev_row][prev_col]
```

**Solution**: Apply modulo during accumulation to prevent overflow:
```python
elif max_score[prev_row][prev_col] == max_score[curr_row][curr_col]:
    path_count[curr_row][curr_col] = (path_count[curr_row][curr_col] + path_count[prev_row][prev_col]) % MOD
```

### 3. **Handling Start and End Characters**
The code skips updating when encountering 'S' (start), but 'E' (end) should be treated differently.

**The Bug**: The condition `if board[curr_row][curr_col] in "XS":` prevents proper processing of the end position 'E'.

**Solution**: Only skip obstacles, handle 'E' and 'S' separately:
```python
if board[curr_row][curr_col] == 'X':
    return
  
# Don't add score for 'E' or 'S', only for digits
if max_score[curr_row][curr_col] != -1 and board[curr_row][curr_col].isdigit():
    max_score[curr_row][curr_col] += int(board[curr_row][curr_col])
```

### 4. **Base Case Initialization**
The base case should be set at the start position 'S' (bottom-right), not assuming it's always at `[n-1][n-1]`.

**Solution**: Find and initialize the actual start position:
```python
# Find start position 'S'
for i in range(board_size):
    for j in range(board_size):
        if board[i][j] == 'S':
            max_score[i][j] = 0
            path_count[i][j] = 1
            break
```

