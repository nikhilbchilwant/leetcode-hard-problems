# 1444. Number of Ways of Cutting a Pizza

> Source: https://algo.monster/liteproblems/1444

## Problem Statement

You have a rectangular pizza represented as a `rows x cols` matrix. Each cell in the matrix contains either:
- `'A'` representing an apple
- `'.'` representing an empty cell

Given an integer `k`, you need to cut the pizza into exactly `k` pieces using `k-1` cuts.

**Cutting Rules:**
1. Each cut must be either vertical or horizontal
2. Cuts are made at cell boundaries (between cells)
3. When making a horizontal cut, you give away the upper portion of the pizza
4. When making a vertical cut, you give away the left portion of the pizza
5. The last remaining piece goes to the last person

**Constraint:** Each piece of pizza must contain at least one apple.

The problem asks you to find the total number of ways to cut the pizza such that every piece has at least one apple. Since the answer can be very large, return the result modulo `10^9 + 7`.

**Example Visualization:**
If you have a 3x3 pizza and need to make 2 cuts (creating 3 pieces):
- First cut could be horizontal at row 1, giving away the top portion
- Second cut could be vertical at column 1 of the remaining pizza, giving away the left portion
- The final remaining piece is the last piece

Each cutting sequence that ensures all pieces have apples counts as one valid way.

## Intuition

When we need to cut the pizza into `k` pieces with `k-1` cuts, we can think of this as a recursive problem. After making the first cut, we're left with a smaller rectangle that needs to be cut into `k-1` pieces with `k-2` cuts.

The key insight is that we can work with the remaining rectangle after each cut. When we make a cut:
- A horizontal cut at row `x` means we give away the rectangle from row `i` to row `x-1`, and continue working with the rectangle from row `x` to the bottom
- A vertical cut at column `y` means we give away the rectangle from column `j` to column `y-1`, and continue working with the rectangle from column `y` to the right

This naturally leads to a recursive approach where we track:
- The top-left corner `(i, j)` of our current working rectangle
- The number of remaining cuts `k` we need to make

For each state `(i, j, k)`, we try all possible positions for the next cut, but only if the piece we're giving away contains at least one apple.

The challenge is efficiently checking if a sub-rectangle contains apples. Checking every cell would be too slow. This is where 2D prefix sums come in - they allow us to calculate the number of apples in any rectangle in `O(1)` time.

With a 2D [prefix sum](/problems/subarray_sum) array `s[i][j]` representing the total apples in the rectangle from `(0,0)` to `(i-1,j-1)`, we can find the apples in any sub-rectangle `(r1,c1)` to `(r2,c2)` using:
`apples = s[r2+1][c2+1] - s[r1][c2+1] - s[r2+1][c1] + s[r1][c1]`

Since we'll revisit the same states multiple times (same remaining rectangle with same number of cuts left), [memoization](/problems/memoization_intro) helps avoid redundant calculations, making the solution efficient.

**Learn more about [Memoization](/problems/memoization_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The solution uses **2D [Prefix Sum](/problems/subarray_sum)** combined with **Memoized Search ([Dynamic Programming](/problems/dynamic_programming_intro))** to efficiently solve the problem.

### Building the 2D [Prefix Sum](/problems/subarray_sum) Array

First, we construct a 2D [prefix sum](/problems/subarray_sum) array `s` where `s[i][j]` represents the total number of apples in the rectangle from `(0,0)` to `(i-1,j-1)`.

The [prefix sum](/problems/subarray_sum) is calculated using the inclusion-exclusion principle:
```
s[i][j] = s[i-1][j] + s[i][j-1] - s[i-1][j-1] + (pizza[i-1][j-1] == 'A')
```

This allows us to query the number of apples in any sub-rectangle in `O(1)` time using:
```
apples in rectangle (r1,c1) to (r2,c2) = s[r2][c2] - s[r1][c2] - s[r2][c1] + s[r1][c1]
```

### Recursive Function with [Memoization](/problems/memoization_intro)

We define `dfs(i, j, k)` which returns the number of ways to cut the rectangle from `(i, j)` to `(m-1, n-1)` using `k` cuts.

**Base Case:** When `k = 0` (no more cuts needed):
- Check if the current rectangle contains at least one apple
- Return `1` if it does, `0` otherwise
- The apple count is: `s[m][n] - s[i][n] - s[m][j] + s[i][j]`

**Recursive Case:** When `k > 0`:

1. **Try horizontal cuts** at positions `x` where `i < x < m`:
   - Check if the upper piece `(i,j)` to `(x-1,n-1)` has apples: `s[x][n] - s[i][n] - s[x][j] + s[i][j] > 0`
   - If yes, recursively solve for the remaining rectangle: `dfs(x, j, k-1)`

2. **Try vertical cuts** at positions `y` where `j < y < n`:
   - Check if the left piece `(i,j)` to `(m-1,y-1)` has apples: `s[m][y] - s[i][y] - s[m][j] + s[i][j] > 0`
   - If yes, recursively solve for the remaining rectangle: `dfs(i, y, k-1)`

3. Sum all valid ways and return the result modulo `10^9 + 7`

### Implementation Details

- The `@cache` decorator automatically memoizes the recursive function, avoiding redundant calculations
- We start with `dfs(0, 0, k-1)` since we need `k-1` cuts to create `k` pieces
- The [prefix sum](/problems/subarray_sum) array has dimensions `(m+1) × (n+1)` for easier boundary handling
- Time Complexity: `O(m × n × k × (m + n))` - for each state `(i,j,k)`, we try `O(m+n)` cutting positions
- Space Complexity: `O(m × n × k)` for the [memoization](/problems/memoization_intro) cache

## Example Walkthrough

Let's walk through a small example with a 3×3 pizza that needs to be cut into 3 pieces (requiring 2 cuts):

```
Pizza:        Prefix Sum Array s:
A . A         0 0 0 0
. A .         0 1 1 2
A . .         0 1 2 3
              0 2 3 4
```

**Step 1: Build the 2D Prefix Sum**
- `s[1][1] = 1` (one apple at position [0][0])
- `s[2][2] = 2` (two apples in rectangle from [0][0] to [1][1])
- `s[3][3] = 4` (total apples in entire pizza)

**Step 2: Start Recursive Search with dfs(0, 0, 2)**

We need to make our first cut. Let's explore the options:

**Horizontal Cuts:**
- Cut at row 1: Give away row 0, keep rows 1-2
  - Check apples in row 0: `s[1][3] - s[0][3] - s[1][0] + s[0][0] = 2 - 0 - 0 + 0 = 2` ✓
  - Recurse: `dfs(1, 0, 1)` (work with rows 1-2, need 1 more cut)

- Cut at row 2: Give away rows 0-1, keep row 2
  - Check apples in rows 0-1: `s[2][3] - s[0][3] - s[2][0] + s[0][0] = 3 - 0 - 0 + 0 = 3` ✓
  - Recurse: `dfs(2, 0, 1)` (work with row 2, need 1 more cut)

**Vertical Cuts:**
- Cut at column 1: Give away column 0, keep columns 1-2
  - Check apples in column 0: `s[3][1] - s[0][1] - s[3][0] + s[0][0] = 2 - 0 - 0 + 0 = 2` ✓
  - Recurse: `dfs(0, 1, 1)` (work with columns 1-2, need 1 more cut)

**Step 3: Continue Recursion (example path: horizontal cut at row 1)**

From `dfs(1, 0, 1)` (working with rows 1-2, columns 0-2):

**Horizontal Cut:**
- Cut at row 2: Give away row 1, keep row 2
  - Check apples in row 1: `s[2][3] - s[1][3] - s[2][0] + s[1][0] = 3 - 2 - 0 + 0 = 1` ✓
  - Recurse: `dfs(2, 0, 0)` (final piece: row 2)

**Vertical Cuts:**
- Cut at column 1: Give away column 0 of rows 1-2
  - Check apples: `s[3][1] - s[1][1] - s[3][0] + s[1][0] = 2 - 1 - 0 + 0 = 1` ✓
  - Recurse: `dfs(1, 1, 0)` (final piece: rows 1-2, columns 1-2)

**Step 4: Base Case (k=0)**

For `dfs(2, 0, 0)`:
- Check if final piece (row 2, all columns) has apples
- Apples = `s[3][3] - s[2][3] - s[3][0] + s[2][0] = 4 - 3 - 0 + 0 = 1` ✓
- Return 1 (valid way)

**Step 5: Aggregate Results**

The function backtracks, summing all valid paths. Each valid cutting sequence where all pieces contain at least one apple contributes 1 to the final count.

The key efficiency comes from:
1. **O(1) apple checking** using prefix sums instead of O(m×n) scanning
2. **Memoization** preventing recalculation of identical subproblems (same rectangle with same cuts remaining)

## Python Solution

```python
class Solution:
    def ways(self, pizza: List[str], k: int) -> int:
        MOD = 10**9 + 7
        rows, cols = len(pizza), len(pizza[0])
      
        # Build 2D prefix sum array to quickly calculate apples in any rectangle
        # prefix_sum[i][j] represents total apples in rectangle from (0,0) to (i-1,j-1)
        prefix_sum = [[0] * (cols + 1) for _ in range(rows + 1)]
        for row in range(1, rows + 1):
            for col in range(1, cols + 1):
                # Add current cell value (1 if apple, 0 otherwise)
                # Use inclusion-exclusion principle for 2D prefix sum
                prefix_sum[row][col] = (prefix_sum[row - 1][col] + 
                                        prefix_sum[row][col - 1] - 
                                        prefix_sum[row - 1][col - 1] + 
                                        int(pizza[row - 1][col - 1] == 'A'))
      
        @cache
        def count_ways(top_row: int, left_col: int, cuts_remaining: int) -> int:
            # Base case: no more cuts needed
            if cuts_remaining == 0:
                # Check if remaining piece has at least one apple
                # Calculate apples in rectangle from (top_row, left_col) to (rows-1, cols-1)
                apples_in_piece = (prefix_sum[rows][cols] - 
                                   prefix_sum[top_row][cols] - 
                                   prefix_sum[rows][left_col] + 
                                   prefix_sum[top_row][left_col])
                return 1 if apples_in_piece > 0 else 0
          
            total_ways = 0
          
            # Try horizontal cuts - give away top portion
            for cut_row in range(top_row + 1, rows):
                # Check if top piece (from top_row to cut_row-1) has at least one apple
                apples_in_top = (prefix_sum[cut_row][cols] - 
                                 prefix_sum[top_row][cols] - 
                                 prefix_sum[cut_row][left_col] + 
                                 prefix_sum[top_row][left_col])
                if apples_in_top > 0:
                    # Recursively process remaining bottom piece
                    total_ways += count_ways(cut_row, left_col, cuts_remaining - 1)
          
            # Try vertical cuts - give away left portion
            for cut_col in range(left_col + 1, cols):
                # Check if left piece (from left_col to cut_col-1) has at least one apple
                apples_in_left = (prefix_sum[rows][cut_col] - 
                                 prefix_sum[top_row][cut_col] - 
                                 prefix_sum[rows][left_col] + 
                                 prefix_sum[top_row][left_col])
                if apples_in_left > 0:
                    # Recursively process remaining right piece
                    total_ways += count_ways(top_row, cut_col, cuts_remaining - 1)
          
            return total_ways % MOD
      
        # Start with full pizza (0,0) and need to make k-1 cuts to get k pieces
        return count_ways(0, 0, k - 1)
```

## Time and Space Complexity

**Time Complexity:** `O(k * m * n * (m + n))`

The analysis breaks down as follows:
- The DFS function has three parameters: `i`, `j`, and `k`, where `i` ranges from `0` to `m-1`, `j` ranges from `0` to `n-1`, and `k` ranges from `0` to `k-1`
- Due to memoization with `@cache`, each unique state `(i, j, k)` is computed only once, giving us `O(m * n * k)` total states
- For each state, we iterate through:
  - All possible horizontal cuts: `x` from `i+1` to `m-1`, which is `O(m)` operations
  - All possible vertical cuts: `y` from `j+1` to `n-1`, which is `O(n)` operations
- Each cut involves checking if there's at least one apple in the piece being given away using the prefix sum array, which takes `O(1)` time
- Therefore, the total time complexity is `O(m * n * k) * O(m + n) = O(k * m * n * (m + n))`

**Space Complexity:** `O(m * n * k + m * n)`

The space complexity consists of:
- The prefix sum array `s`: `O((m+1) * (n+1)) = O(m * n)`
- The memoization cache storing all possible states: `O(m * n * k)`
- The recursion call stack depth in the worst case: `O(k)` (when we make `k` consecutive cuts)
- Since `k ≤ min(m, n)`, the recursion stack is dominated by the cache space
- Total space complexity: `O(m * n * k + m * n) = O(m * n * k)` when `k > 1`, or `O(m * n)` when `k = 1`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Prefix Sum Calculation for Rectangle Queries**

One of the most common mistakes is incorrectly calculating the number of apples in a sub-rectangle using the prefix sum array. The formula uses inclusion-exclusion principle, and it's easy to mix up the indices or signs.

**Incorrect Implementation:**
```python
# Wrong: Using wrong indices or forgetting to add/subtract terms
apples = prefix_sum[r2][c2] - prefix_sum[r1][c2] - prefix_sum[r2][c1]  # Missing + prefix_sum[r1][c1]
# Or
apples = prefix_sum[r2+1][c2+1] - prefix_sum[r1][c2+1] - prefix_sum[r2+1][c1] + prefix_sum[r1][c1]  # Wrong indices
```

**Correct Implementation:**
```python
# For rectangle from (r1, c1) to (r2, c2) inclusive:
apples = prefix_sum[r2+1][c2+1] - prefix_sum[r1][c2+1] - prefix_sum[r2+1][c1] + prefix_sum[r1][c1]
```

### 2. **Off-by-One Errors in Cut Positions**

Another frequent mistake is confusion about what portion of the pizza is being given away and what remains after a cut.

**Incorrect Understanding:**
```python
# Wrong: Thinking cut_row is the last row we keep
for cut_row in range(top_row + 1, rows + 1):  # Wrong upper bound
    # Checking wrong piece for apples
    apples_in_bottom = ...  # Should check top piece that's given away
```

**Correct Understanding:**
- For a horizontal cut at position `cut_row`, we give away rows from `top_row` to `cut_row-1`
- The remaining piece starts from row `cut_row`
- Similar logic applies to vertical cuts with columns

### 3. **Forgetting the Modulo Operation**

The problem requires returning the answer modulo 10^9 + 7, but it's easy to forget applying modulo at the right places.

**Problematic Code:**
```python
def count_ways(top_row, left_col, cuts_remaining):
    # ... calculation ...
    total_ways += count_ways(cut_row, left_col, cuts_remaining - 1)
    # ... more additions ...
    return total_ways  # Forgot modulo - can cause integer overflow in other languages
```

**Better Approach:**
```python
def count_ways(top_row, left_col, cuts_remaining):
    # ... calculation ...
    total_ways = (total_ways + count_ways(cut_row, left_col, cuts_remaining - 1)) % MOD
    # Or apply modulo once at the end of the function
    return total_ways % MOD
```

### 4. **Misunderstanding the Number of Cuts vs Pieces**

A subtle but critical error is confusing the relationship between cuts and pieces.

**Wrong:**
```python
# Starting with k cuts instead of k-1
return count_ways(0, 0, k)  # Wrong: k cuts would create k+1 pieces
```

**Correct:**
```python
# k pieces require k-1 cuts
return count_ways(0, 0, k - 1)
```

### 5. **Not Checking if Given-Away Piece Has Apples**

Some implementations forget to verify that the piece being given away contains at least one apple before making the cut.

**Incomplete Solution:**
```python
for cut_row in range(top_row + 1, rows):
    # Missing check for apples in the piece being given away
    total_ways += count_ways(cut_row, left_col, cuts_remaining - 1)
```

**Complete Solution:**
```python
for cut_row in range(top_row + 1, rows):
    apples_in_top = (prefix_sum[cut_row][cols] - 
                     prefix_sum[top_row][cols] - 
                     prefix_sum[cut_row][left_col] + 
                     prefix_sum[top_row][left_col])
    if apples_in_top > 0:  # Essential check
        total_ways += count_ways(cut_row, left_col, cuts_remaining - 1)
```

### Prevention Tips:
1. Draw out small examples to verify your prefix sum calculations
2. Clearly comment which piece is being given away vs kept
3. Test with edge cases where k=1 (no cuts) and k=rows*cols (maximum cuts)
4. Verify your understanding of 0-indexed vs 1-indexed coordinates in the prefix sum array

