# 1411. Number of Ways to Paint N × 3 Grid

> Source: https://algo.monster/liteproblems/1411

## Problem Statement

You have a grid with `n` rows and `3` columns that needs to be painted. Each cell must be painted with exactly one of three colors: Red, Yellow, or Green. The constraint is that no two adjacent cells (cells that share a vertical or horizontal side) can have the same color.

Given `n` (the number of rows), you need to find the total number of ways to paint the entire `n x 3` grid while following the coloring constraint. Since the answer can be very large, return the result modulo `10^9 + 7`.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with state compression. It classifies all valid coloring patterns for a single row into two types based on symmetry:
- Type `010`: patterns where the first and third cells have the same color (like Red-Yellow-Red)
- Type `012`: patterns where all three cells have different colors (like Red-Yellow-Green)

For the first row, there are `6` ways for each type (total of `12` valid patterns).

The transitions between rows follow these rules:
- From a `010` type row, the next row can have `3` patterns of `010` type and `2` patterns of `012` type
- From a `012` type row, the next row can have `2` patterns of `010` type and `2` patterns of `012` type

The code maintains two variables `f0` and `f1` representing the count of `010` and `012` type patterns respectively, and updates them iteratively for each row using the formulas:
- `g0 = (3 * f0 + 2 * f1) % mod`
- `g1 = (2 * f0 + 2 * f1) % mod`

After processing all `n` rows, the final answer is `(f0 + f1) % mod`.

## Intuition

When faced with this problem, the first observation is that with only 3 columns, we can enumerate all possible valid colorings for a single row. With 3 colors and 3 positions, there are `3^3 = 27` total combinations, but many violate the adjacency constraint.

The key insight is recognizing that we don't need to track the exact colors, but rather the **pattern** of colors. Since we can permute colors freely, what matters is whether adjacent cells are the same or different.

For a row with 3 cells, there are only two fundamental patterns:
1. **Same-Different-Same** (like `R-Y-R`): The first and third cells share the same color
2. **All-Different** (like `R-Y-G`): All three cells have different colors

Why are these the only patterns? Consider all possibilities:
- All three same: Invalid (adjacent cells would match)
- Two adjacent same: Invalid (violates constraint)
- First and third same: Valid (Same-Different-Same pattern)
- All different: Valid (All-Different pattern)

Now, counting the actual colorings:
- For Same-Different-Same: Choose 1 color for positions 1 and 3 (3 choices), then choose a different color for position 2 (2 choices) = `3 × 2 = 6` ways
- For All-Different: Choose 3 different colors and arrange them = `3! = 6` ways

The brilliant part is realizing that when placing row `i+1` below row `i`, we only need to know row `i`'s pattern type, not its specific colors. The compatibility between consecutive rows depends solely on their pattern types.

By working through examples, we can determine the transition rules:
- From a Same-Different-Same row, we can place `3` Same-Different-Same patterns and `2` All-Different patterns
- From an All-Different row, we can place `2` Same-Different-Same patterns and `2` All-Different patterns

This transforms the problem into a simple [dynamic programming](/problems/dynamic_programming_intro) recurrence where we track counts of each pattern type and update them row by row.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

Based on the pattern classification, we implement a [dynamic programming](/problems/dynamic_programming_intro) solution that tracks the count of each pattern type row by row.

**State Definition:**
- `f0`: Number of ways to paint rows ending with a Same-Different-Same pattern (010 type)
- `f1`: Number of ways to paint rows ending with an All-Different pattern (012 type)

**Initialization:**
For the first row, both pattern types have `6` valid colorings each:
- `f0 = 6` (Same-Different-Same patterns)
- `f1 = 6` (All-Different patterns)

**Transition Formula:**
For each subsequent row, we calculate the new counts based on the compatibility rules:
- `g0 = (3 * f0 + 2 * f1) % mod` - New count for 010 type
- `g1 = (2 * f0 + 2 * f1) % mod` - New count for 012 type

These formulas come from:
- From `f0` (010 type): Can transition to `3` patterns of 010 type and `2` patterns of 012 type
- From `f1` (012 type): Can transition to `2` patterns of 010 type and `2` patterns of 012 type

**Implementation Steps:**
1. Initialize `f0 = f1 = 6` for the first row
2. Iterate `n-1` times (for rows 2 through n):
   - Calculate new counts `g0` and `g1` using the transition formulas
   - Apply modulo `10^9 + 7` to prevent overflow
   - Update `f0` and `f1` with the new values
3. Return `(f0 + f1) % mod` as the total number of valid ways

**Time Complexity:** `O(n)` - Single loop through n rows
**Space Complexity:** `O(1)` - Only using constant variables to track states

The elegance of this solution lies in reducing a seemingly complex coloring problem to tracking just two numbers representing pattern counts, making it both efficient and easy to implement.

## Example Walkthrough

Let's walk through the solution for a grid with `n = 2` rows (a 2×3 grid).

**Step 1: Initialize Row 1**
- We start with the first row, which can have two pattern types:
  - **010 type (Same-Different-Same)**: Examples include R-Y-R, R-G-R, Y-R-Y, Y-G-Y, G-R-G, G-Y-G → 6 ways
  - **012 type (All-Different)**: Examples include R-Y-G, R-G-Y, Y-R-G, Y-G-R, G-R-Y, G-Y-R → 6 ways
- Initialize: `f0 = 6`, `f1 = 6`

**Step 2: Calculate Row 2**
Now we need to count valid colorings for row 2 based on what row 1 could be.

If row 1 is **010 type** (like R-Y-R):
- Row 2 can be 010 type in 3 ways. For example, if row 1 is R-Y-R:
  - Y-R-Y ✓ (valid: no adjacent cells match)
  - Y-G-Y ✓ (valid: no adjacent cells match)  
  - G-R-G ✓ (valid: no adjacent cells match)
- Row 2 can be 012 type in 2 ways. For example, if row 1 is R-Y-R:
  - Y-R-G ✓ (valid: no adjacent cells match)
  - G-R-Y ✓ (valid: no adjacent cells match)

If row 1 is **012 type** (like R-Y-G):
- Row 2 can be 010 type in 2 ways. For example, if row 1 is R-Y-G:
  - Y-R-Y ✓ (valid: no adjacent cells match)
  - G-R-G ✓ (valid: no adjacent cells match)
- Row 2 can be 012 type in 2 ways. For example, if row 1 is R-Y-G:
  - Y-G-R ✓ (valid: no adjacent cells match)
  - G-R-Y ✓ (valid: no adjacent cells match)

**Step 3: Apply Transition Formulas**
- `g0 = (3 * f0 + 2 * f1) % mod = (3 * 6 + 2 * 6) % mod = 30`
- `g1 = (2 * f0 + 2 * f1) % mod = (2 * 6 + 2 * 6) % mod = 24`

**Step 4: Update States**
- `f0 = 30` (ways to have row 2 end with 010 pattern)
- `f1 = 24` (ways to have row 2 end with 012 pattern)

**Step 5: Calculate Final Answer**
Total ways = `(f0 + f1) % mod = (30 + 24) % mod = 54`

Therefore, there are **54** ways to paint a 2×3 grid following the coloring constraints.

## Python Solution

```python
class Solution:
    def numOfWays(self, n: int) -> int:
        """
        Calculate the number of ways to paint a 3×n grid with 3 colors.
        Each 1×3 column must use exactly 3 colors (ABA or ABC patterns).
        Adjacent cells cannot have the same color.
      
        Args:
            n: Number of columns in the grid
          
        Returns:
            Number of valid painting ways modulo 10^9 + 7
        """
        MOD = 10**9 + 7
      
        # Initialize counters for the first column
        # aba_pattern_count: Number of ways to paint a column with ABA pattern (same first and last color)
        # abc_pattern_count: Number of ways to paint a column with ABC pattern (all different colors)
        # Both start with 6 ways each (3 colors × 2 permutations each)
        aba_pattern_count = 6  # Patterns like: RGR, GRG, BRB, RBR, GBG, BGB
        abc_pattern_count = 6  # Patterns like: RGB, RBG, GRB, GBR, BRG, BGR
      
        # Build up the solution column by column
        for column_index in range(n - 1):
            # Calculate valid patterns for the next column based on current column
            # From ABA pattern: can transition to 3 ABA patterns + 2 ABC patterns
            # From ABC pattern: can transition to 2 ABA patterns + 2 ABC patterns
            next_aba_count = (3 * aba_pattern_count + 2 * abc_pattern_count) % MOD
            next_abc_count = (2 * aba_pattern_count + 2 * abc_pattern_count) % MOD
          
            # Update pattern counts for the next iteration
            aba_pattern_count = next_aba_count
            abc_pattern_count = next_abc_count
      
        # Return the total number of valid ways
        return (aba_pattern_count + abc_pattern_count) % MOD
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the number of rows in the grid. This is because the algorithm uses a single for loop that iterates `n - 1` times, and within each iteration, it performs a constant number of arithmetic operations (multiplications, additions, and modulo operations). Each iteration updates the values of `g0` and `g1` based on the previous values `f0` and `f1`, then assigns these new values back to `f0` and `f1`.

The space complexity is `O(1)`. The algorithm only uses a fixed amount of extra space regardless of the input size `n`. It maintains four variables (`f0`, `f1`, `g0`, `g1`) and one constant (`mod`) throughout the execution. No additional data structures that grow with the input size are created, making the space usage constant.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding Grid Orientation**
A critical pitfall is confusing the grid dimensions. The problem states an `n × 3` grid (n rows, 3 columns), but the code comments incorrectly refer to "columns" when iterating through rows. This conceptual mismatch can lead to:
- Incorrect pattern counting
- Wrong transition logic application
- Difficulty debugging when visualizing the problem

**Solution:** Maintain consistent terminology throughout. Since we're dealing with `n` rows and `3` columns, the iteration should be clearly documented as row-by-row processing:

```python
# Corrected version with proper terminology
for row_index in range(n - 1):  # Processing rows 2 through n
    # Calculate valid patterns for the next row based on current row
    next_aba_count = (3 * aba_pattern_count + 2 * abc_pattern_count) % MOD
    next_abc_count = (2 * aba_pattern_count + 2 * abc_pattern_count) % MOD
```

### 2. **Integer Overflow Without Proper Modulo Application**
Forgetting to apply modulo operations during intermediate calculations can cause integer overflow in languages with fixed integer sizes, or incorrect results even in Python when the final modulo is applied to an already overflowed value.

**Solution:** Apply modulo at each multiplication step, not just at the end:

```python
# Safe calculation preventing overflow
next_aba_count = ((3 * aba_pattern_count) % MOD + (2 * abc_pattern_count) % MOD) % MOD
next_abc_count = ((2 * aba_pattern_count) % MOD + (2 * abc_pattern_count) % MOD) % MOD
```

### 3. **Off-by-One Error in Loop Range**
Using `range(n)` instead of `range(n-1)` would process one extra row, giving incorrect results. This happens when forgetting that the first row is already initialized.

**Solution:** Remember that initialization handles row 1, so the loop should run exactly `n-1` times:

```python
# First row is already counted in initialization
for _ in range(n - 1):  # Rows 2 through n
    # transition logic
```

### 4. **Incorrect Pattern Transition Coefficients**
Mixing up the transition coefficients (3, 2, 2, 2) or applying them to wrong pattern types is easy when not carefully tracking the compatibility rules.

**Solution:** Document the transition logic clearly with examples:

```python
# Transition rules (verified by enumeration):
# ABA → ABA: 3 ways (change middle color to any of 3 options except sides)
# ABA → ABC: 2 ways (change one side to create all-different pattern)
# ABC → ABA: 2 ways (make first and last colors match)
# ABC → ABC: 2 ways (permute colors maintaining all-different constraint)
```

