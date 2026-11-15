# 1001. Grid Illumination

> Source: https://algo.monster/liteproblems/1001

## Problem Statement

You have an `n x n` grid where each cell contains a lamp that starts in the "off" state.

You're given an array `lamps` where each element `lamps[i] = [row_i, col_i]` represents a lamp at position `(row_i, col_i)` that is turned on. If the same lamp position appears multiple times in the array, it's still just one lamp that's turned on.

When a lamp is on, it illuminates:
- Its own cell
- All cells in the same row
- All cells in the same column  
- All cells on the same diagonal (both main diagonal and anti-diagonal)

You're also given an array `queries` where each `queries[j] = [row_j, col_j]` represents a query position. For each query, you need to:
1. Check if the cell at `(row_j, col_j)` is currently illuminated (by any lamp)
2. After checking, turn off the lamp at `(row_j, col_j)` if it exists, AND turn off all lamps in the 8 adjacent cells (the cells that share a side or corner with the query cell)

Return an array `ans` where `ans[j] = 1` if the cell at the j-th query position was illuminated when checked, or `0` if it wasn't illuminated.

The key challenge is efficiently tracking which cells are illuminated as lamps get turned on and off, especially since a single lamp can illuminate an entire row, column, and both diagonals.

## Intuition

The key insight is that we don't need to track every single illuminated cell in the grid - that would be inefficient for large grids. Instead, we can track which rows, columns, and diagonals have at least one lamp turned on.

Think about it this way: if there's even one lamp in row 5, then ALL cells in row 5 are illuminated. We don't need to mark each cell individually. The same logic applies to columns and diagonals.

For diagonals, we need a way to uniquely identify each diagonal line. If you observe cells on the same main diagonal (going from top-left to bottom-right), they all have the same value for `row - col`. For example, cells `(2,1)`, `(3,2)`, and `(4,3)` all have `row - col = 1`. Similarly, cells on the same anti-diagonal (going from top-right to bottom-left) have the same value for `row + col`.

So instead of a 2D grid tracking illumination, we can use four counters:
- One for counting lamps in each row
- One for counting lamps in each column  
- One for counting lamps on each main diagonal (identified by `row - col`)
- One for counting lamps on each anti-diagonal (identified by `row + col`)

When checking if a cell `(i, j)` is illuminated, we just need to check if any of these four counters are non-zero:
- Is there any lamp in row `i`?
- Is there any lamp in column `j`?
- Is there any lamp on the main diagonal with value `i - j`?
- Is there any lamp on the anti-diagonal with value `i + j`?

For turning off lamps, we need to maintain a set of actual lamp positions. When we turn off a lamp at position `(x, y)`, we decrement the corresponding counters for that lamp's row, column, and diagonals. This way, we can efficiently track when a line no longer has any lamps illuminating it.

## Solution Approach

Let's walk through the implementation step by step:

**1. Initialize Data Structures**

In the code, `s` is named `lamp_positions`, the counters `row`, `col`, `diag1`, `diag2` are `illuminated_rows`, `illuminated_cols`, `illuminated_main_diag`, `illuminated_anti_diag`, and `ans` is `result`.

First, we create a set `s` to store unique lamp positions: `s = {(i, j) for i, j in lamps}`. This removes duplicate lamp positions since we treat repeated lamps as a single lamp.

Then we create four Counter objects to track the number of lamps on each line:
- `row[i]`: counts lamps in row `i`
- `col[j]`: counts lamps in column `j`
- `diag1[i-j]`: counts lamps on the main diagonal with value `i-j`
- `diag2[i+j]`: counts lamps on the anti-diagonal with value `i+j`

**2. Count Initial Lamps**

For each unique lamp position `(i, j)` in the set:
```python
row[i] += 1
col[j] += 1
diag1[i - j] += 1
diag2[i + j] += 1
```
This populates our counters with the initial lamp configuration.

**3. Process Queries**

For each query at position `(i, j)`:

**Check Illumination:** A cell is illuminated if any of the following counters are non-zero:
- `row[i]` (any lamp in the same row)
- `col[j]` (any lamp in the same column)
- `diag1[i - j]` (any lamp on the same main diagonal)
- `diag2[i + j]` (any lamp on the same anti-diagonal)

If any of these are true, we set `ans[k] = 1`.

**4. Turn Off Adjacent Lamps**

After checking illumination, we need to turn off the lamp at the query position and all 8 adjacent lamps. We iterate through a 3×3 grid centered at `(i, j)`:
```python
for x in range(i - 1, i + 2):
    for y in range(j - 1, j + 2):
```

For each position `(x, y)` in this 3×3 grid, if there's a lamp (i.e., `(x, y) in s`):
- Remove it from the set: `s.remove((x, y))`
- Decrement all corresponding counters:
  ```python
  row[x] -= 1
  col[y] -= 1
  diag1[x - y] -= 1
  diag2[x + y] -= 1
  ```

This ensures that if a line no longer has any lamps, its counter becomes 0, and cells on that line won't be considered illuminated in future queries.

**5. Return Results**

Finally, return the `ans` array containing the illumination status for each query.

The time complexity is `O(L + Q)` where `L` is the number of lamps and `Q` is the number of queries, since we process each lamp once initially and for each query we check at most 9 positions. The space complexity is `O(L)` for storing the lamp positions and counters.

## Example Walkthrough

Let's walk through a small example with a 4×4 grid, 3 lamps, and 2 queries.

**Initial Setup:**
- Grid size: n = 4
- Lamps: `[[1,1], [2,2], [0,3]]`
- Queries: `[[1,0], [2,2]]`

**Step 1: Initialize and count lamps**

Create a set of unique lamp positions: `s = {(1,1), (2,2), (0,3)}`

Initialize counters and populate them:
- Lamp at (1,1):
  - `row[1] = 1`, `col[1] = 1`, `diag1[0] = 1`, `diag2[2] = 1`
- Lamp at (2,2):
  - `row[2] = 1`, `col[2] = 1`, `diag1[0] = 2`, `diag2[4] = 1`
- Lamp at (0,3):
  - `row[0] = 1`, `col[3] = 1`, `diag1[-3] = 1`, `diag2[3] = 1`

Final counters:
- `row = {0:1, 1:1, 2:1}`
- `col = {1:1, 2:1, 3:1}`
- `diag1 = {0:2, -3:1}` (note: diagonal 0 has 2 lamps!)
- `diag2 = {2:1, 3:1, 4:1}`

**Step 2: Process Query 1 at (1,0)**

Check if (1,0) is illuminated:
- `row[1] = 1` ✓ (row 1 has a lamp)
- `col[0] = 0` ✗ (column 0 has no lamps)
- `diag1[1-0] = diag1[1] = 0` ✗ (diagonal 1 has no lamps)
- `diag2[1+0] = diag2[1] = 0` ✗ (anti-diagonal 1 has no lamps)

Result: Cell (1,0) IS illuminated (because row 1 has a lamp), so `ans[0] = 1`

Turn off lamps in 3×3 area around (1,0):
- Check positions: (0,-1), (0,0), (0,1), (1,-1), (1,0), (1,1), (2,-1), (2,0), (2,1)
- Only (1,1) has a lamp, so remove it:
  - Remove (1,1) from set: `s = {(2,2), (0,3)}`
  - Update counters: `row[1]--`, `col[1]--`, `diag1[0]--`, `diag2[2]--`

Updated counters:
- `row = {0:1, 1:0, 2:1}`
- `col = {1:0, 2:1, 3:1}`
- `diag1 = {0:1, -3:1}`
- `diag2 = {2:0, 3:1, 4:1}`

**Step 3: Process Query 2 at (2,2)**

Check if (2,2) is illuminated:
- `row[2] = 1` ✓ (row 2 has a lamp)
- `col[2] = 1` ✓ (column 2 has a lamp)
- `diag1[2-2] = diag1[0] = 1` ✓ (diagonal 0 has a lamp)
- `diag2[2+2] = diag2[4] = 1` ✓ (anti-diagonal 4 has a lamp)

Result: Cell (2,2) IS illuminated, so `ans[1] = 1`

Turn off lamps in 3×3 area around (2,2):
- Check positions: (1,1), (1,2), (1,3), (2,1), (2,2), (2,3), (3,1), (3,2), (3,3)
- Only (2,2) has a lamp, so remove it:
  - Remove (2,2) from set: `s = {(0,3)}`
  - Update counters: `row[2]--`, `col[2]--`, `diag1[0]--`, `diag2[4]--`

Final counters:
- `row = {0:1, 1:0, 2:0}`
- `col = {1:0, 2:0, 3:1}`
- `diag1 = {0:0, -3:1}`
- `diag2 = {2:0, 3:1, 4:0}`

**Final Answer:** `[1, 1]`

Both query positions were illuminated when checked. The key insight is that we never tracked individual cell illumination - we only tracked which lines (rows, columns, diagonals) had lamps, making the solution efficient even for large grids.

## Python Solution

```python
class Solution:
    def gridIllumination(
        self, n: int, lamps: List[List[int]], queries: List[List[int]]
    ) -> List[int]:
        # Convert lamp positions to a set for O(1) lookup and to remove duplicates
        lamp_positions = {(row, col) for row, col in lamps}
      
        # Initialize counters to track illuminated rows, columns, and diagonals
        # Each counter tracks how many lamps illuminate that particular line
        illuminated_rows = Counter()
        illuminated_cols = Counter()
        illuminated_main_diag = Counter()  # Main diagonal (top-left to bottom-right)
        illuminated_anti_diag = Counter()  # Anti-diagonal (top-right to bottom-left)
      
        # Count illumination for each lamp
        for row, col in lamp_positions:
            illuminated_rows[row] += 1
            illuminated_cols[col] += 1
            illuminated_main_diag[row - col] += 1  # Main diagonal has constant row - col
            illuminated_anti_diag[row + col] += 1  # Anti-diagonal has constant row + col
      
        # Initialize result array
        result = [0] * len(queries)
      
        # Process each query
        for query_idx, (query_row, query_col) in enumerate(queries):
            # Check if the queried cell is illuminated
            # A cell is illuminated if its row, column, or either diagonal has any lamp
            if (illuminated_rows[query_row] or 
                illuminated_cols[query_col] or 
                illuminated_main_diag[query_row - query_col] or 
                illuminated_anti_diag[query_row + query_col]):
                result[query_idx] = 1
          
            # Turn off all lamps in the 3x3 grid centered at the query position
            for neighbor_row in range(query_row - 1, query_row + 2):
                for neighbor_col in range(query_col - 1, query_col + 2):
                    if (neighbor_row, neighbor_col) in lamp_positions:
                        # Remove the lamp from the set
                        lamp_positions.remove((neighbor_row, neighbor_col))
                      
                        # Update illumination counters
                        illuminated_rows[neighbor_row] -= 1
                        illuminated_cols[neighbor_col] -= 1
                        illuminated_main_diag[neighbor_row - neighbor_col] -= 1
                        illuminated_anti_diag[neighbor_row + neighbor_col] -= 1
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(m + q)`

The time complexity breaks down as follows:
- Creating the set `s` from lamps takes `O(m)` time, where `m` is the number of lamps
- Initializing the counters (row, col, diag1, diag2) by iterating through the set takes `O(m)` time
- Processing queries takes `O(q)` time, where `q` is the number of queries:
  - For each query, checking if the position is illuminated takes `O(1)` time
  - Turning off adjacent lamps involves checking a 3×3 grid (at most 9 cells), which is `O(1)` constant time
  - Set removal and counter updates are `O(1)` operations
- Total: `O(m) + O(m) + O(q) = O(m + q)`

**Space Complexity:** `O(m)`

The space complexity consists of:
- Set `s` stores at most `m` unique lamp positions: `O(m)`
- Four Counter objects (row, col, diag1, diag2) each store at most `m` entries: `O(m)`
- Answer array stores `q` results: `O(q)`
- Total: `O(m) + O(q)`, which simplifies to `O(m + q)`

However, if we consider that the output array is required for the result and not counted as extra space, the auxiliary space complexity is `O(m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Handling Grid Boundaries When Turning Off Adjacent Lamps**

The code iterates through a 3×3 block around the query without checking whether each cell lies inside the grid (0 to n-1):

```python
for neighbor_row in range(query_row - 1, query_row + 2):
    for neighbor_col in range(query_col - 1, query_col + 2):
        if (neighbor_row, neighbor_col) in lamp_positions:
            # Process lamp...
```

**Is this a bug?** No. When the query is at the edge of the grid (e.g., position (0, 0) or (n-1, n-1)), the loop looks up positions such as (-1, -1) or (n, n). No lamp can exist there, so the set lookup simply returns `False` and nothing changes. The pitfall arises only in a version that stores lamps in a 2D array instead of a set, where an index of -1 or n would read the wrong cell or go out of bounds.

**Optional:** Clamping the ranges skips the impossible positions:

```python
for neighbor_row in range(max(0, query_row - 1), min(n, query_row + 2)):
    for neighbor_col in range(max(0, query_col - 1), min(n, query_col + 2)):
        if (neighbor_row, neighbor_col) in lamp_positions:
            # Process lamp...
```

### 2. **Processing Duplicate Lamps Multiple Times**

Another common mistake is forgetting to handle duplicate lamp positions in the input array. If the same lamp position appears multiple times in the `lamps` array, counting it multiple times would incorrectly inflate the illumination counters.

**Problem Example:** If `lamps = [[1, 1], [1, 1], [2, 2]]`, processing the lamp at (1, 1) twice would set `illuminated_rows[1] = 2` instead of 1, causing incorrect illumination detection even after the lamp is turned off.

**Solution:** The code correctly handles this by converting the lamp list to a set first:
```python
lamp_positions = {(row, col) for row, col in lamps}
```

This ensures each unique position is processed only once when initializing the counters.

### 3. **Using Wrong Diagonal Formulas**

A subtle but critical error is using incorrect formulas for diagonal identification. The correct formulas are:
- Main diagonal: `row - col` (constant for all cells on the same main diagonal)
- Anti-diagonal: `row + col` (constant for all cells on the same anti-diagonal)

**Common Mistake:** Using the same formula for both diagonal counters (for example `row + col` twice), or computing a different key when checking a query than when adding or removing a lamp. Which counter is called "main" and which "anti" does not matter, as long as every lamp update and every query uses the same key for the same counter. Note also that `row - col` can be negative, which is fine for a hash map but needs an offset of `n - 1` if the counters are arrays.

**Verification:** You can verify the formulas by checking a few examples:
- Main diagonal through (2,1), (3,2), (4,3): all have `row - col = 1`
- Anti-diagonal through (0,2), (1,1), (2,0): all have `row + col = 2`

