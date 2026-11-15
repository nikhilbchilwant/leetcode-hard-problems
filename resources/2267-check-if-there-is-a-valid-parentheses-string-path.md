# 2267. Check if There Is a Valid Parentheses String Path

> Source: https://algo.monster/liteproblems/2267

## Problem Statement

You are given an `m x n` matrix where each cell contains either `'('` or `')'`. Your task is to determine if there exists a valid path from the top-left corner `(0, 0)` to the bottom-right corner `(m-1, n-1)` that forms a valid parentheses string.

A valid parentheses string must follow these rules:
- It can be `()`
- It can be formed by concatenating two valid parentheses strings (like `AB` where both `A` and `B` are valid)
- It can be formed by wrapping a valid parentheses string with parentheses (like `(A)` where `A` is valid)

The path must satisfy these constraints:
- Start at cell `(0, 0)`
- End at cell `(m-1, n-1)`  
- Only move down or right at each step
- The sequence of parentheses collected along the path forms a valid parentheses string

For example, if you traverse a path and collect `"(())"`, this would be valid. But if you collect `"(()"` or `")("`, these would be invalid.

Return `true` if such a valid path exists, otherwise return `false`.

## Intuition

To form a valid parentheses string, we need equal numbers of `'('` and `')'`, with `'('` never being outnumbered by `')'` at any point when reading left to right. This gives us our first insight: the total path length `m + n - 1` must be even (since we need pairs of parentheses).

The key observation is tracking the "balance" - the difference between open and close parentheses seen so far. As we traverse the path:
- When we see `'('`, the balance increases by 1
- When we see `')'`, the balance decreases by 1
- The balance must never go negative (that would mean more `')'` than `'('` at some point)
- The balance must equal 0 when we reach the destination

We can use DFS to explore all possible paths from `(0, 0)` to `(m-1, n-1)`. At each cell `(i, j)`, we have at most two choices: go right or go down. We keep track of the current balance `k` as we explore.

An important optimization comes from realizing that if our current balance `k` exceeds the number of remaining cells to the destination (`m - i + n - j`), we can't possibly close all open parentheses even if all remaining cells were `')'`. This allows us to prune invalid paths early.

Similarly, we can immediately reject paths where:
- The starting cell `(0, 0)` contains `')'` (would start with negative balance)
- The ending cell `(m-1, n-1)` contains `'('` (can't end with balance 0)

The memoization with `@cache` is crucial because many different paths might reach the same cell with the same balance, and we don't want to recompute the result for each one.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses **Depth-First Search (DFS) with memoization** to explore all valid paths while avoiding redundant computations.

**Initial Validation:**
First, we check three conditions that would make a valid path impossible:
- If `(m + n - 1) % 2 == 1`: The path length is odd, so we can't have matching parentheses
- If `grid[0][0] == ")"`: Starting with a close parenthesis gives negative balance
- If `grid[m-1][n-1] == "("`: Ending with an open parenthesis prevents reaching balance 0

**DFS Function Design:**
The core function `dfs(i, j, k)` determines if there's a valid path from position `(i, j)` with current balance `k`:

1. **Update Balance**: First, we update the balance based on the current cell:
   ```python
   d = 1 if grid[i][j] == "(" else -1
   k += d
   ```

2. **Pruning Conditions**: We check if the current state is invalid:
   - If `k < 0`: More close parentheses than open ones (invalid string)
   - If `k > m - i + n - j`: Balance exceeds remaining cells, impossible to close all parentheses

3. **Base Case**: When we reach `(m-1, n-1)`, return `True` only if `k == 0` (balanced parentheses)

4. **Recursive Exploration**: We try moving right and down:
   ```python
   for a, b in pairwise((0, 1, 0)):
       x, y = i + a, j + b
   ```
   This iterates through `(0, 1)` for moving right and `(1, 0)` for moving down.

5. **Boundary Check and Recursion**: For each valid next position within bounds, we recursively check if it leads to a valid path.

**Memoization with @cache:**
The `@cache` decorator stores results for each unique `(i, j, k)` combination. This is crucial because multiple paths might reach the same cell with the same balance, and we avoid recomputing the result.

The solution starts by calling `dfs(0, 0, 0)` with initial position `(0, 0)` and balance `0` (before processing the first cell). The algorithm efficiently explores all possible paths while pruning invalid branches early and reusing computed results through memoization.

## Example Walkthrough

Let's walk through a small example with this grid:
```
[['(', '(', ')'],
 [')', '(', ')']]
```

**Initial Checks:**
- Grid dimensions: m=2, n=3
- Path length: m + n - 1 = 4 (even ✓)
- grid[0][0] = '(' (valid start ✓)
- grid[1][2] = ')' (valid end ✓)

**DFS Exploration starting from dfs(0, 0, 0):**

1. **At (0,0):** Cell contains '(', balance becomes 0+1=1
   - Balance 1 is valid (not negative)
   - Balance 1 ≤ remaining cells (4), so continue
   - Try two paths: right to (0,1) or down to (1,0)

2. **Path 1: (0,0) → (0,1)** with balance 1
   - At (0,1): Cell contains '(', balance becomes 1+1=2
   - Balance 2 ≤ remaining cells (3), valid
   - Try: right to (0,2) or down to (1,1)

3. **Path 1a: (0,0) → (0,1) → (0,2)** with balance 2
   - At (0,2): Cell contains ')', balance becomes 2-1=1
   - Balance 1 ≤ remaining cells (2), valid
   - Only option: down to (1,2)
   - At (1,2): Cell contains ')', balance becomes 1-1=0
   - Reached destination with balance 0! ✓
   - Path "(())" is valid!

4. **Path 1b: (0,0) → (0,1) → (1,1)** with balance 2
   - At (1,1): Cell contains '(', balance becomes 2+1=3
   - Balance 3 > remaining cells (2) 
   - Cannot close all parentheses, prune this path ✗

5. **Path 2: (0,0) → (1,0)** with balance 1
   - At (1,0): Cell contains ')', balance becomes 1-1=0
   - Balance 0 ≤ remaining cells (3), valid
   - Only option: right to (1,1)
   - At (1,1): Cell contains '(', balance becomes 0+1=1
   - Only option: right to (1,2)
   - At (1,2): Cell contains ')', balance becomes 1-1=0
   - Reached destination with balance 0! ✓
   - Path "()()" is valid!

**Result:** The function returns `True` because we found valid paths. The memoization would cache results like `dfs(1,1,2) = False` and `dfs(1,2,1) = True` to avoid recomputation if these states were reached again through different paths.

## Python Solution

```python
class Solution:
    def hasValidPath(self, grid: List[List[str]]) -> bool:
        from functools import cache
        from typing import List
      
        @cache
        def dfs(row: int, col: int, open_count: int) -> bool:
            """
            Depth-first search to find valid parentheses path.
          
            Args:
                row: Current row position
                col: Current column position
                open_count: Number of unmatched opening parentheses
          
            Returns:
                True if a valid path exists from current position to end
            """
            # Update open parentheses count based on current cell
            delta = 1 if grid[row][col] == "(" else -1
            open_count += delta
          
            # Pruning: invalid if negative count or impossible to balance
            if open_count < 0:
                return False
          
            # Maximum possible closing parentheses from current position to end
            remaining_steps = (rows - row - 1) + (cols - col - 1)
            if open_count > remaining_steps:
                return False
          
            # Check if we reached the destination
            if row == rows - 1 and col == cols - 1:
                return open_count == 0
          
            # Try moving right and down
            directions = [(0, 1), (1, 0)]  # right, down
            for dx, dy in directions:
                next_row, next_col = row + dx, col + dy
                # Check bounds and recursively explore
                if 0 <= next_row < rows and 0 <= next_col < cols:
                    if dfs(next_row, next_col, open_count):
                        return True
          
            return False
      
        rows, cols = len(grid), len(grid[0])
      
        # Early termination checks
        # Path length must be even for balanced parentheses
        if (rows + cols - 1) % 2 == 1:
            return False
      
        # Start must be '(' and end must be ')'
        if grid[0][0] == ")" or grid[rows - 1][cols - 1] == "(":
            return False
      
        # Start DFS from top-left corner with 0 open parentheses
        return dfs(0, 0, 0)
```

## Time and Space Complexity

**Time Complexity:** `O(m × n × (m + n))`

The time complexity is determined by the memoized DFS function. The `dfs` function has three parameters: `i` (row position from 0 to m-1), `j` (column position from 0 to n-1), and `k` (balance of parentheses). The key insight is that `k` represents the current balance of opening and closing parentheses, which can range from 0 to at most `m + n - 1` (the maximum path length from top-left to bottom-right). Due to the `@cache` decorator, each unique combination of `(i, j, k)` is computed only once. There are `m` possible values for `i`, `n` possible values for `j`, and at most `m + n` possible values for `k`. Therefore, the total number of unique states is `O(m × n × (m + n))`.

**Space Complexity:** `O(m × n × (m + n))`

The space complexity comes from two sources:
1. The memoization cache stores results for all possible states `(i, j, k)`, which requires `O(m × n × (m + n))` space
2. The recursion stack depth in the worst case can go up to `O(m + n)` (the length of the longest path)

Since the cache dominates the space usage, the overall space complexity is `O(m × n × (m + n))`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Balance Calculation Timing**
A common mistake is updating the balance after checking pruning conditions or at the wrong point in the recursion. The balance must be updated immediately upon entering a cell, before any validity checks.

**Incorrect approach:**
```python
def dfs(row, col, open_count):
    # Checking conditions before updating balance - WRONG!
    if open_count < 0:
        return False
  
    # Update balance after checks
    delta = 1 if grid[row][col] == "(" else -1
    open_count += delta
```

**Correct approach:**
```python
def dfs(row, col, open_count):
    # Update balance first
    delta = 1 if grid[row][col] == "(" else -1
    open_count += delta
  
    # Then check validity
    if open_count < 0:
        return False
```

### 2. **Off-by-One Error in Remaining Steps Calculation**
Many implementations incorrectly calculate the remaining steps to reach the destination, forgetting that we're already at position `(row, col)`.

**Incorrect calculation:**
```python
# This counts total remaining cells including current position
remaining_steps = (rows - row) + (cols - col)
```

**Correct calculation:**
```python
# Excludes current position, counts only moves needed
remaining_steps = (rows - row - 1) + (cols - col - 1)
```

### 3. **Missing Early Termination Checks**
Failing to check if a valid path is even possible before starting DFS leads to unnecessary computation.

**Incomplete validation:**
```python
# Only checking path length
if (rows + cols - 1) % 2 == 1:
    return False
# Missing checks for start and end cells
```

**Complete validation:**
```python
# Check all three conditions
if (rows + cols - 1) % 2 == 1:
    return False
if grid[0][0] == ")" or grid[rows - 1][cols - 1] == "(":
    return False
```

### 4. **Incorrect Memoization Key**
Using the wrong combination of parameters as the memoization key can lead to incorrect results or missed optimization opportunities.

**Wrong memoization:**
```python
@cache
def dfs(row, col):  # Missing balance parameter!
    balance = calculate_balance_somehow()  # Can't track properly
```

**Correct memoization:**
```python
@cache
def dfs(row, col, open_count):  # All state variables included
    # Process with complete state information
```

### 5. **Balance vs. Open Count Confusion**
Some implementations confuse "balance" (which can be negative) with "open count" (unmatched opening parentheses). While both approaches work, mixing them leads to errors.

**Consistent open count approach:**
```python
# open_count represents unmatched '(' parentheses
delta = 1 if grid[row][col] == "(" else -1
open_count += delta
if open_count < 0:  # More ')' than '(' encountered
    return False
```

### Solution to Avoid These Pitfalls:
1. Always update the balance/count immediately upon entering a cell
2. Carefully calculate remaining steps excluding the current position
3. Implement all necessary pre-validation checks before starting DFS
4. Include all state variables (row, col, balance) in the memoization decorator
5. Be consistent with your chosen representation (balance or open count) throughout the implementation
6. Test edge cases: single cell grid, all same parenthesis type, alternating patterns

