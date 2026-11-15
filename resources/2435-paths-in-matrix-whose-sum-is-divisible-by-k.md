# 2435. Paths in Matrix Whose Sum Is Divisible by K

> Source: https://algo.monster/liteproblems/2435

## Problem Statement

You have an `m x n` grid of integers and need to find paths from the top-left corner `(0, 0)` to the bottom-right corner `(m-1, n-1)`. You can only move right or down at each step.

The task is to count how many such paths have a total sum that is divisible by a given integer `k`. As you traverse a path, you add up all the numbers in the cells you visit, and you want this sum to be divisible by `k` (i.e., `sum % k == 0`).

Since the number of valid paths could be very large, return the answer modulo `10^9 + 7`.

For example, if you have a grid and `k = 5`, you need to find all possible paths from top-left to bottom-right where the sum of all elements along the path is divisible by 5.

## Intuition

The key insight is that we don't need to track the actual sum of each path - we only care whether the sum is divisible by `k`. This means we can work with remainders (modulo `k`) instead of actual sums, which keeps our numbers bounded between `0` and `k-1`.

Think about it this way: if we have a path sum of 1000 and `k = 7`, we only need to know that `1000 % 7 = 6`. If we add another cell with value 8, the new remainder is `(6 + 8) % 7 = 0`, making it divisible by `k`. This modular arithmetic property allows us to track just the remainder at each step.

Since we can only move right or down, this naturally forms a recursive structure. At each cell `(i, j)`, we face two choices:
- Move down to `(i+1, j)` 
- Move right to `(i, j+1)`

For each choice, we update our running remainder by adding the current cell's value and taking modulo `k`.

The overlapping subproblems emerge because multiple paths can reach the same cell with the same remainder. For instance, reaching cell `(2, 2)` with remainder `3` could happen via different routes, but the number of valid paths from `(2, 2)` to the destination with starting remainder `3` is always the same. This makes memoization perfect - we can cache results based on three parameters: row position, column position, and current remainder.

The base case is when we reach the bottom-right corner: we return `1` if our accumulated remainder is `0` (meaning the path sum is divisible by `k`), otherwise `0`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses **memoization search** (top-down [dynamic programming](/problems/dynamic_programming_intro)) to efficiently count the valid paths.

We define a recursive function `dfs(i, j, s)` where:
- `i, j` represents the current position in the grid
- `s` represents the current path sum modulo `k`

The function returns the number of paths from position `(i, j)` to the destination `(m-1, n-1)` such that when combined with the current remainder `s`, the total path sum is divisible by `k`.

**Implementation Details:**

1. **Base Cases:**
   - If we go out of bounds (`i >= m` or `j >= n`), return `0`
   - If we reach the destination `(m-1, n-1)`, check if the final remainder is `0`. If yes, we found a valid path, return `1`; otherwise return `0`

2. **Recursive Step:**
   - Add the current cell value to our running remainder: `s = (s + grid[i][j]) % k`
   - Explore both possible moves:
     - Move down: `dfs(i + 1, j, s)`
     - Move right: `dfs(i, j + 1, s)`
   - Sum up the results from both directions

3. **Memoization:**
   - The `@cache` decorator automatically memoizes the function results
   - This prevents recalculating the same state `(i, j, s)` multiple times
   - Since there are `m × n × k` possible states, this significantly improves efficiency

4. **Modular Arithmetic:**
   - All intermediate results are taken modulo `10^9 + 7` to prevent overflow
   - The final answer is `dfs(0, 0, 0)`, starting from the top-left with initial remainder `0`

The recurrence relation can be expressed as:
```
dfs(i, j, s) = dfs(i + 1, j, (s + grid[i][j]) % k) + dfs(i, j + 1, (s + grid[i][j]) % k)
```

The time complexity is `O(m × n × k)` since we compute each state at most once, and the space complexity is also `O(m × n × k)` for the memoization cache.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a 2×3 grid with `k = 3`:
```
grid = [[1, 2, 1],
        [1, 1, 2]]
```

We want to count paths from (0,0) to (1,2) where the path sum is divisible by 3.

**Starting the recursion:** `dfs(0, 0, 0)`
- Current position: (0, 0), value = 1
- Current remainder: (0 + 1) % 3 = 1
- We can move right to (0, 1) or down to (1, 0)

**Branch 1: Move right from (0,0) → (0,1):** `dfs(0, 1, 1)`
- Current position: (0, 1), value = 2  
- Current remainder: (1 + 2) % 3 = 0
- We can move right to (0, 2) or down to (1, 1)

  **Branch 1.1: (0,1) → (0,2):** `dfs(0, 2, 0)`
  - Current position: (0, 2), value = 1
  - Current remainder: (0 + 1) % 3 = 1
  - Can only move down to (1, 2)

    **Branch 1.1.1: (0,2) → (1,2):** `dfs(1, 2, 1)`
    - At destination (1, 2), value = 2
    - Final remainder: (1 + 2) % 3 = 0 ✓
    - Return 1 (valid path found!)

  **Branch 1.2: (0,1) → (1,1):** `dfs(1, 1, 0)`
  - Current position: (1, 1), value = 1
  - Current remainder: (0 + 1) % 3 = 1
  - Can only move right to (1, 2)

    **Branch 1.2.1: (1,1) → (1,2):** `dfs(1, 2, 1)`
    - At destination (1, 2), value = 2
    - Final remainder: (1 + 2) % 3 = 0 ✓
    - Return 1 (valid path found!)

**Branch 2: Move down from (0,0) → (1,0):** `dfs(1, 0, 1)`
- Current position: (1, 0), value = 1
- Current remainder: (1 + 1) % 3 = 2
- Can only move right to (1, 1)

  **Branch 2.1: (1,0) → (1,1):** `dfs(1, 1, 2)`
  - Current position: (1, 1), value = 1
  - Current remainder: (2 + 1) % 3 = 0
  - Can only move right to (1, 2)

    **Branch 2.1.1: (1,1) → (1,2):** `dfs(1, 2, 0)`
    - At destination (1, 2), value = 2
    - Final remainder: (0 + 2) % 3 = 2 ✗
    - Return 0 (path sum not divisible by 3)

**Final Result:**
- Path 1: (0,0) → (0,1) → (0,2) → (1,2): sum = 1+2+1+2 = 6, divisible by 3 ✓
- Path 2: (0,0) → (0,1) → (1,1) → (1,2): sum = 1+2+1+2 = 6, divisible by 3 ✓
- Path 3: (0,0) → (1,0) → (1,1) → (1,2): sum = 1+1+1+2 = 5, not divisible by 3 ✗

Total valid paths = 2

Note how memoization helps: `dfs(1, 2, 1)` is called twice (from branches 1.1.1 and 1.2.1), but we only compute it once and reuse the cached result.

## Python Solution

```python
class Solution:
    def numberOfPaths(self, grid: List[List[int]], k: int) -> int:
        """
        Find the number of paths from top-left to bottom-right where the sum of values
        along the path is divisible by k.
      
        Args:
            grid: 2D grid of integers
            k: The divisor to check path sums against
          
        Returns:
            Number of valid paths modulo 10^9 + 7
        """
        from functools import cache
      
        @cache
        def dfs(row: int, col: int, current_sum: int) -> int:
            """
            Recursively explore paths using DFS with memoization.
          
            Args:
                row: Current row position
                col: Current column position
                current_sum: Running sum modulo k of the path so far
              
            Returns:
                Number of valid paths from current position to destination
            """
            # Check if current position is out of bounds
            if row < 0 or row >= rows or col < 0 or col >= cols:
                return 0
          
            # Update the running sum with current cell value (modulo k)
            current_sum = (current_sum + grid[row][col]) % k
          
            # Check if we've reached the destination (bottom-right corner)
            if row == rows - 1 and col == cols - 1:
                # Return 1 if path sum is divisible by k, otherwise 0
                return int(current_sum == 0)
          
            # Explore two possible moves: down and right
            move_down = dfs(row + 1, col, current_sum)
            move_right = dfs(row, col + 1, current_sum)
          
            # Return total number of valid paths from current position
            total_paths = (move_down + move_right) % MOD
            return total_paths
      
        # Initialize grid dimensions
        rows = len(grid)
        cols = len(grid[0])
      
        # Define modulo constant for large numbers
        MOD = 10**9 + 7
      
        # Start DFS from top-left corner with initial sum of 0
        result = dfs(0, 0, 0)
      
        # Clear the cache to free memory
        dfs.cache_clear()
      
        return result
```

## Time and Space Complexity

The time complexity is `O(m × n × k)`, where `m` is the number of rows, `n` is the number of columns, and `k` is the given modulus value. This is because the DFS function uses memoization with the `@cache` decorator, and there are at most `m × n × k` unique states. Each state is defined by three parameters: the current position `(i, j)` which has `m × n` possible values, and the remainder `s` which can take values from `0` to `k-1`, giving `k` possible values. Each state is computed only once due to memoization, and each computation takes `O(1)` time (excluding recursive calls).

The space complexity is `O(m × n × k)`. This consists of two main components:
1. The memoization cache stores up to `m × n × k` unique states
2. The recursion stack depth in the worst case is `O(m + n)` (moving from top-left to bottom-right)

Since `m × n × k` dominates `m + n` in most practical cases, the overall space complexity is `O(m × n × k)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Modulo Operation on Path Sum**

One of the most common mistakes is forgetting to apply modulo `k` consistently when tracking the path sum. Some developers might track the actual sum and only check divisibility at the end, which can lead to integer overflow for large grids or incorrect remainder calculations.

**Incorrect approach:**
```python
# Wrong: Tracking actual sum instead of remainder
def dfs(row, col, total_sum):
    total_sum += grid[row][col]  # Can overflow!
    if row == rows - 1 and col == cols - 1:
        return int(total_sum % k == 0)
```

**Correct approach:**
```python
# Right: Always work with remainder modulo k
def dfs(row, col, current_sum):
    current_sum = (current_sum + grid[row][col]) % k
    if row == rows - 1 and col == cols - 1:
        return int(current_sum == 0)
```

### 2. **Boundary Check Logic Error**

Another frequent mistake is checking boundaries after accessing the grid element, which causes IndexError. The boundary check must happen before any grid access.

**Incorrect approach:**
```python
def dfs(row, col, current_sum):
    # Wrong: Accessing grid before boundary check
    current_sum = (current_sum + grid[row][col]) % k
    if row >= rows or col >= cols:  # Too late!
        return 0
```

**Correct approach:**
```python
def dfs(row, col, current_sum):
    # Right: Check boundaries first
    if row >= rows or col >= cols:
        return 0
    current_sum = (current_sum + grid[row][col]) % k
```

### 3. **Memory Leak from Cache Not Being Cleared**

When using `@cache` or `@lru_cache`, the memoization dictionary persists between test cases in competitive programming platforms. This can cause incorrect results or memory issues if the same Solution instance is reused.

**Solution:**
Always clear the cache after getting the result:
```python
result = dfs(0, 0, 0)
dfs.cache_clear()  # Important for multiple test cases
return result
```

### 4. **Forgetting to Apply MOD to Intermediate Results**

The problem requires the final answer modulo `10^9 + 7`, but it's easy to forget applying this to intermediate calculations when combining path counts.

**Incorrect approach:**
```python
# Wrong: Not applying MOD to intermediate sum
move_down = dfs(row + 1, col, current_sum)
move_right = dfs(row, col + 1, current_sum)
return move_down + move_right  # Can exceed MOD!
```

**Correct approach:**
```python
# Right: Apply MOD to prevent overflow
move_down = dfs(row + 1, col, current_sum)
move_right = dfs(row, col + 1, current_sum)
return (move_down + move_right) % MOD
```

### 5. **State Design Inefficiency**

Some implementations might pass unnecessary information in the recursive state, such as the entire path or a copy of the grid, which drastically increases memory usage and slows down the solution.

**Key insight:** You only need to track:
- Current position (row, col)
- Current path sum modulo k

Any additional state information is redundant and harmful to performance.

