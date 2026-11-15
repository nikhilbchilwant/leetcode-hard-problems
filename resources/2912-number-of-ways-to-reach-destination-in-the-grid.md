# 2912. Number of Ways to Reach Destination in the Grid

> Source: https://algo.monster/liteproblems/2912

## Problem Statement

You have a grid with dimensions `n × m` (both 1-indexed), and you need to find the number of ways to move from a starting position `source` to an ending position `dest` in exactly `k` moves.

The movement rules are:
- From any cell `[x₁, y₁]`, you can move to another cell `[x₂, y₂]` if either:
  - `x₁ == x₂` (same row, different column), OR
  - `y₁ == y₂` (same column, different row)
- You cannot stay in the same cell (both coordinates must not be the same)

Given:
- `n`: number of rows in the grid
- `m`: number of columns in the grid
- `k`: exact number of moves to make
- `source`: starting position `[x, y]` (1-indexed)
- `dest`: destination position `[x, y]` (1-indexed)

Return the number of ways to reach `dest` from `source` in exactly `k` moves, modulo `10⁹ + 7`.

For example, in a move from cell `[2, 3]`, you could go to:
- Any cell in row 2 except `[2, 3]` itself
- Any cell in column 3 except `[2, 3]` itself

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with state transitions. It tracks four states based on the relationship between current position and source:
- State 0: at the source position itself
- State 1: different row, same column as source
- State 2: same row, different column as source
- State 3: different row and different column from source

The transitions between states are calculated based on how many valid moves are possible from each state, considering the grid dimensions and movement constraints.

## Intuition

The key insight is recognizing that after any number of moves, the position relative to the source can only be in one of four states:
1. Back at the source itself
2. Different row but same column as source
3. Same row but different column as source
4. Different row AND different column from source

Why does this matter? Because the number of ways to reach any specific cell depends only on which of these four states it belongs to, not on the exact coordinates. This dramatically reduces the problem complexity.

Consider what happens when we make a move:
- From the source position, we can move to `(n-1)` cells in the same column or `(m-1)` cells in the same row
- From a position in the same column (but different row), we can return to source, move to `(n-2)` other cells in that column, or jump to any of `(m-1)` columns (landing in state 3)
- Similar logic applies for positions in the same row or in completely different rows/columns

Instead of tracking paths to every possible cell (which would be computationally expensive), we track only how many ways we can reach each of these four states after each move. This transforms an intractable problem into one with just 4 variables that we update `k` times.

The transition formulas emerge naturally when we count the valid moves from each state:
- `g[0]`: Ways to return to source = from state 1 (same column) we have `(n-1)` such positions, plus from state 2 (same row) we have `(m-1)` such positions
- `g[1]`: Ways to reach same column = from source (1 way) + from within same column `(n-2)` ways + from different row/column positions that can jump to this column `(m-1)` ways
- And so on...

After `k` iterations, we simply check which state the destination belongs to relative to the source and return the corresponding count.

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

We implement the solution using [dynamic programming](/problems/dynamic_programming_intro) with state transitions. The approach uses four variables to track the number of ways to reach each state after each move.

**State Definition:**
- `a` (or `f[0]`): number of ways to be at the source position
- `b` (or `f[1]`): number of ways to be in a different row but same column as source
- `c` (or `f[2]`): number of ways to be in the same row but different column as source
- `d` (or `f[3]`): number of ways to be in a different row AND different column from source

**Initialization:**
We start with `a = 1` (we begin at the source), and `b = c = d = 0` (we haven't moved yet).

**State Transitions:**
For each of the `k` moves, we calculate the new state values based on the current state:

```
aa = (n - 1) × b + (m - 1) × c
bb = a + (n - 2) × b + (m - 1) × d
cc = a + (m - 2) × c + (n - 1) × d
dd = b + c + (n - 2) × d + (m - 2) × d
```

Breaking down each transition:
- `aa`: To reach source, we can come from `(n-1)` positions in the same column (state `b`) or `(m-1)` positions in the same row (state `c`)
- `bb`: To reach a different row in the same column, we can come from source (1 way), from `(n-2)` other positions in that column, or from `(m-1)` different columns in different rows
- `cc`: To reach a different column in the same row, we can come from source (1 way), from `(m-2)` other positions in that row, or from `(n-1)` different rows in different columns
- `dd`: To reach a different row and column, we can come from state `b` (1 way), state `c` (1 way), or from `(n-2) + (m-2)` other positions in state `d`

**Modulo Operation:**
All calculations are performed modulo `10^9 + 7` to prevent overflow.

**Final Answer:**
After `k` iterations, we determine which state the destination belongs to:
- If `source[0] == dest[0]` (same row):
  - If `source[1] == dest[1]`: return `a` (destination is source)
  - Else: return `c` (different column, same row)
- Else (different row):
  - If `source[1] == dest[1]`: return `b` (different row, same column)
  - Else: return `d` (different row and column)

The time complexity is `O(k)` and space complexity is `O(1)` since we only maintain four variables throughout the computation.

## Example Walkthrough

Let's work through a small example with a 3×3 grid, moving from source `[1,1]` to destination `[2,3]` in exactly `k=2` moves.

**Initial Setup:**
- Grid: 3×3 (n=3, m=3)
- Source: `[1,1]`
- Destination: `[2,3]` (different row AND different column from source)
- States: `a=1, b=0, c=0, d=0` (we start at source)

**Move 1:**
From source `[1,1]`, we can move to:
- Same column (column 1): `[2,1]` and `[3,1]` → 2 positions (state b)
- Same row (row 1): `[1,2]` and `[1,3]` → 2 positions (state c)

Calculating new states:
- `aa = (3-1)×0 + (3-1)×0 = 0` (can't return to source in 1 move)
- `bb = 1 + (3-2)×0 + (3-1)×0 = 1` (1 way from source)
- `cc = 1 + (3-2)×0 + (3-1)×0 = 1` (1 way from source)
- `dd = 0 + 0 + (3-2)×0 + (3-2)×0 = 0` (can't reach different row AND column in 1 move)

After move 1: `a=0, b=1, c=1, d=0`

This means:
- 0 ways to be at `[1,1]` (source)
- 1 way to be in column 1 but different row (either `[2,1]` or `[3,1]`)
- 1 way to be in row 1 but different column (either `[1,2]` or `[1,3]`)
- 0 ways to be in different row AND column

**Move 2:**
Now we calculate transitions from the current states:

- `aa = (3-1)×1 + (3-1)×1 = 2 + 2 = 4`
  - From state b (different row, same column): 2 positions can reach source
  - From state c (same row, different column): 2 positions can reach source

- `bb = 0 + (3-2)×1 + (3-1)×0 = 0 + 1 + 0 = 1`
  - From source: 0 ways (not at source)
  - From within same column: 1 way (can move within column)
  - From state d: 0 ways (no positions in state d)

- `cc = 0 + (3-2)×1 + (3-1)×0 = 0 + 1 + 0 = 1`
  - From source: 0 ways
  - From within same row: 1 way
  - From state d: 0 ways

- `dd = 1 + 1 + (3-2)×0 + (3-2)×0 = 2`
  - From state b: 1 way (can jump to any different column)
  - From state c: 1 way (can jump to any different row)
  - From state d: 0 ways (no positions there yet)

After move 2: `a=4, b=1, c=1, d=2`

**Final Answer:**
Destination `[2,3]` is in a different row AND different column from source `[1,1]`, so it belongs to state d.

Therefore, there are **2 ways** to reach `[2,3]` from `[1,1]` in exactly 2 moves.

We can verify this by listing the actual paths:
1. `[1,1] → [1,3] → [2,3]` (move along row 1, then along column 3)
2. `[1,1] → [2,1] → [2,3]` (move along column 1, then along row 2)

The dynamic programming approach correctly counts these paths without explicitly enumerating them, making it efficient even for large values of k.

## Python Solution

```python
class Solution:
    def numberOfWays(
        self, n: int, m: int, k: int, source: List[int], dest: List[int]
    ) -> int:
        MOD = 10**9 + 7

        # Dynamic programming states after i steps:
        # same_pos: number of ways to stay at the same position
        # same_row_diff_col: number of ways to be in same row but different column
        # diff_row_same_col: number of ways to be in different row but same column
        # diff_row_diff_col: number of ways to be in different row and different column

        # Initialize for 0 steps (starting position)
        same_pos = 1
        same_row_diff_col = 0
        diff_row_same_col = 0
        diff_row_diff_col = 0

        # Calculate transitions for each step
        for _ in range(k):
            # Calculate next state values based on current state

            # To stay at same position: can come from same row (n-1 choices) or same column (m-1 choices)
            next_same_pos = ((n - 1) * same_row_diff_col + (m - 1) * diff_row_same_col) % MOD

            # To reach same row, different column:
            # - from same position (1 way to any of m-1 columns)
            # - from same row different column (n-2 other columns to current)
            # - from different row different column (m-1 columns in same row)
            next_same_row_diff_col = (same_pos + (n - 2) * same_row_diff_col + (m - 1) * diff_row_diff_col) % MOD

            # To reach different row, same column:
            # - from same position (1 way to any of n-1 rows)
            # - from different row same column (m-2 other rows to current)
            # - from different row different column (n-1 rows in same column)
            next_diff_row_same_col = (same_pos + (m - 2) * diff_row_same_col + (n - 1) * diff_row_diff_col) % MOD

            # To reach different row, different column:
            # - from same row different column (1 way to any of n-1 rows)
            # - from different row same column (1 way to any of m-1 columns)
            # - from different row different column ((n-2) + (m-2) ways to stay in different state)
            next_diff_row_diff_col = (same_row_diff_col + diff_row_same_col + (n - 2) * diff_row_diff_col + (m - 2) * diff_row_diff_col) % MOD

            # Update states for next iteration
            same_pos = next_same_pos
            same_row_diff_col = next_same_row_diff_col
            diff_row_same_col = next_diff_row_same_col
            diff_row_diff_col = next_diff_row_diff_col

        # Return the appropriate count based on source and destination relationship
        if source[0] == dest[0]:  # Same row
            if source[1] == dest[1]:  # Same position
                return same_pos
            else:  # Different column
                return diff_row_same_col
        else:  # Different row
            if source[1] == dest[1]:  # Same column
                return same_row_diff_col
            else:  # Different column
                return diff_row_diff_col
```

## Time and Space Complexity

The time complexity is `O(k)`, where `k` is the number of moves. This is because the algorithm runs a single loop that iterates exactly `k` times. In each iteration, it performs a constant number of arithmetic operations (additions, multiplications, and modulo operations) to update the four variables `a`, `b`, `c`, and `d`. Since each iteration takes `O(1)` time and there are `k` iterations, the overall time complexity is `O(k)`.

The space complexity is `O(1)`. The algorithm uses only a fixed number of variables (`a`, `b`, `c`, `d`, `aa`, `bb`, `cc`, `dd`, and `mod`) regardless of the input size. These variables store intermediate results and the final answer, but their number doesn't grow with `n`, `m`, or `k`. The input lists `source` and `dest` are given as parameters and don't count toward the extra space used by the algorithm itself. Therefore, the space complexity is constant, or `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect State-to-Variable Mapping**
One of the most common mistakes is mixing up which variable corresponds to which state, especially when returning the final answer. The variable names and their meanings can be confusing:

**Pitfall Example:**
```python
# Incorrect mapping when returning the result
if source[0] == dest[0]:  # Same row
    if source[1] == dest[1]:  # Same position
        return same_pos
    else:  # Different column but same row
        return diff_row_same_col  # WRONG! Should be same_row_diff_col
```

**Solution:**
Use clear, consistent naming and add comments to track what each variable represents:
```python
# State definitions (relative to source):
# same_pos: at source position
# same_row_diff_col: same row as source, different column
# diff_row_same_col: different row from source, same column
# diff_row_diff_col: different row and different column from source
```

### 2. **Incorrect Transition Coefficients**
Another critical error is using wrong coefficients in state transitions. For example, when calculating transitions to `diff_row_same_col`, you might accidentally use `(n-2)` instead of `(m-2)`:

**Pitfall Example:**
```python
# Wrong coefficient - using n-2 where m-2 should be used
next_diff_row_same_col = (same_pos + (n - 2) * diff_row_same_col + ...) % MOD
# Should be (m-2) because we're counting different rows while staying in same column
```

**Solution:**
Remember the logic for each coefficient:
- When moving within the same row: use `m` (column count)
- When moving within the same column: use `n` (row count)
- Subtract appropriately based on restrictions (e.g., `-1` for excluding current position, `-2` for excluding both current and target)

### 3. **Forgetting the Modulo Operation**
With large values of `k`, intermediate calculations can overflow. Forgetting to apply modulo at each step leads to incorrect results:

**Pitfall Example:**
```python
# Missing modulo - can cause overflow
next_same_pos = (n - 1) * same_row_diff_col + (m - 1) * diff_row_same_col
```

**Solution:**
Apply modulo after every arithmetic operation:
```python
next_same_pos = ((n - 1) * same_row_diff_col + (m - 1) * diff_row_same_col) % MOD
```

### 4. **Edge Case: Single Row or Column Grid**
When `n = 1` or `m = 1`, some transitions become impossible, and the coefficients can become negative:

**Pitfall Example:**
```python
# When n = 1, (n-2) becomes -1, causing incorrect calculations
next_same_row_diff_col = (same_pos + (n - 2) * same_row_diff_col + ...) % MOD
```

**Solution:**
Handle edge cases explicitly or ensure coefficients don't go negative:
```python
# Use max(0, coefficient) to prevent negative values
next_same_row_diff_col = (same_pos + max(0, n - 2) * same_row_diff_col + (m - 1) * diff_row_diff_col) % MOD
```

### 5. **Zero Moves Edge Case**
When `k = 0`, the answer should be 1 if source equals destination, 0 otherwise. The initialization handles this correctly, but be careful not to enter the loop:

**Solution:**
The current implementation handles this correctly by initializing `same_pos = 1` and running the loop exactly `k` times. If `k = 0`, the loop doesn't execute, and the final return logic correctly returns 1 for source == dest, 0 otherwise.

