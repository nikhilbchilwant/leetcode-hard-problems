# 782. Transform to Chessboard

> Source: https://algo.monster/liteproblems/782

## Problem Statement

You have an `n x n` binary grid called `board` containing only 0s and 1s. Your goal is to transform this grid into a valid chessboard pattern through a series of moves.

In each move, you can:
- Swap any two complete rows with each other, OR
- Swap any two complete columns with each other

A **chessboard board** is defined as a grid where no two adjacent cells (horizontally or vertically adjacent) have the same value. In other words, 0s and 1s must alternate in all four directions, creating a checkerboard pattern like:
```
0 1 0 1
1 0 1 0
0 1 0 1
1 0 1 0
```
or
```
1 0 1 0
0 1 0 1
1 0 1 0
0 1 0 1
```

The problem asks you to find the **minimum number of moves** needed to transform the given board into a valid chessboard. If it's impossible to achieve a chessboard pattern through any number of row and column swaps, return `-1`.

Key observations about valid chessboards:
- There can only be two distinct row patterns in the entire board, and they must be complements of each other (if one row is "0101", the other must be "1010")
- The same property applies to columns
- For even-sized boards (`n = 2k`), each row and column must have exactly `k` ones and `k` zeros
- For odd-sized boards (`n = 2k+1`), each row and column must have either `k` ones and `k+1` zeros, or `k+1` ones and `k` zeros

## Intuition

The key insight is that row and column swaps don't change the internal pattern of any row or column - they only change positions. This means if we can't form a chessboard with the existing row and column patterns, no amount of swapping will help.

Let's think about what makes a valid chessboard. In any valid chessboard, there are exactly **two types of rows**: one pattern and its complement. For example, if one row is `"0101"`, then all other rows must be either `"0101"` or `"1010"`. The same applies to columns.

Why is this true? Consider any valid chessboard - if we fix a row pattern, the row directly below it must have the opposite pattern (to ensure no vertical adjacencies of same values). The row below that must match the first row again, and so on. This creates an alternating pattern of just two row types.

Since swapping rows/columns only rearranges them without changing their content, we first need to verify that our board contains exactly two row patterns (and they're complements) and exactly two column patterns (also complements). If not, it's impossible to form a chessboard.

Once we know a chessboard is possible, the problem reduces to counting how many rows and columns are already in their correct positions. We can use bit manipulation to represent each row/column as a number, making comparisons efficient.

For the minimum swaps calculation:
- If `n` is even, we have two valid target patterns: starting with 0 (`"0101..."`) or starting with 1 (`"1010..."`). We calculate swaps needed for both and take the minimum.
- If `n` is odd, there's only one valid pattern based on the count of 0s and 1s. The pattern with more positions must occupy the majority slots.

The beauty of this approach is recognizing that we can solve rows and columns independently - the minimum total swaps equals the sum of minimum row swaps and minimum column swaps. This works because row swaps don't affect column patterns and vice versa.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

The solution uses **bit manipulation and state compression** to efficiently represent and compare row/column patterns.

**Step 1: Extract and Validate Patterns**

First, we extract the pattern of the first row and first column as our reference patterns:
- `rowMask`: represents the first row as a bitmask where bit `i` is set if `board[0][i] == 1`
- `colMask`: represents the first column as a bitmask where bit `i` is set if `board[i][0] == 1`

We then calculate their complements:
- `revRowMask = mask ^ rowMask` where `mask = (1 << n) - 1` gives us all bits set
- `revColMask = mask ^ colMask`

**Step 2: Verify Board Validity**

For each row and column in the board:
- Convert it to a bitmask representation (`curRowMask` and `curColMask`)
- Check if each row matches either `rowMask` or `revRowMask`
- Check if each column matches either `colMask` or `revColMask`
- If any row/column doesn't match one of the two patterns, return `-1`

While checking, we also count:
- `sameRow`: number of rows matching `rowMask`
- `sameCol`: number of columns matching `colMask`

**Step 3: Calculate Minimum Swaps**

The helper function `f(mask, cnt)` calculates the minimum swaps needed for a dimension (row or column):

For **odd n**:
- Check if the count of 1s and the count of matching patterns are valid (difference of 1 from half)
- Determine the target pattern based on which bit value appears more frequently
- Use the constants `0xAAAAAAAA` (bits set at odd positions 1, 3, 5, ...) and `0x55555555` (bits set at even positions 0, 2, 4, ...) to count how many 1s already sit where the target needs them
- If the pattern has `n//2` ones, the target (read from position 0) is `"01010..."`, whose 1s are at odd positions, so the swaps are `n // 2 - (pattern_mask & 0xAAAAAAAA).bit_count()`
- Otherwise, the target is `"10101..."`, whose 1s are at even positions, so the swaps are `(n + 1) // 2 - (pattern_mask & 0x55555555).bit_count()`

For **even n**:
- Verify exactly half the bits are 1s and exactly half the rows/columns match the pattern
- Calculate swaps needed for both possible targets:
  - `swaps_pattern_0`: swaps to achieve `"0101..."` (read from position 0; 1s at odd positions, counted with `0xAAAAAAAA`)
  - `swaps_pattern_1`: swaps to achieve `"1010..."` (1s at even positions, counted with `0x55555555`)
- Return the minimum of the two

**Step 4: Combine Results**

The total minimum moves is the sum of:
- `t1`: minimum swaps needed for rows
- `t2`: minimum swaps needed for columns

If either dimension is impossible to arrange (`t1 == -1` or `t2 == -1`), return `-1`.

The key optimization is using bit manipulation for pattern matching and counting, which reduces the complexity from checking each element to simple bitwise operations. The magic numbers `0xAAAAAAAA` and `0x55555555` act as masks for alternating bit patterns, allowing us to count positions that need swapping with a single `bit_count()` operation.

## Example Walkthrough

Let's walk through a small example with a 4×4 board:

```
Input board:
0 1 1 0
1 0 0 1
1 0 0 1
0 1 1 0
```

**Step 1: Extract Reference Patterns**

First row: `[0, 1, 1, 0]` → `rowMask = 0110` (binary) = 6
First column: `[0, 1, 1, 0]` → `colMask = 0110` (binary) = 6

Calculate complements (with `mask = 1111` = 15):
- `revRowMask = 15 ^ 6 = 1001` (binary) = 9 → pattern `[1, 0, 0, 1]`
- `revColMask = 15 ^ 6 = 1001` (binary) = 9 → pattern `[1, 0, 0, 1]`

**Step 2: Validate Board Patterns**

Check each row:
- Row 0: `[0, 1, 1, 0]` = 6 ✓ matches `rowMask`
- Row 1: `[1, 0, 0, 1]` = 9 ✓ matches `revRowMask`
- Row 2: `[1, 0, 0, 1]` = 9 ✓ matches `revRowMask`
- Row 3: `[0, 1, 1, 0]` = 6 ✓ matches `rowMask`

Count: `sameRow = 2` (rows matching `rowMask`)

Check each column:
- Col 0: `[0, 1, 1, 0]` = 6 ✓ matches `colMask`
- Col 1: `[1, 0, 0, 1]` = 9 ✓ matches `revColMask`
- Col 2: `[1, 0, 0, 1]` = 9 ✓ matches `revColMask`
- Col 3: `[0, 1, 1, 0]` = 6 ✓ matches `colMask`

Count: `sameCol = 2` (columns matching `colMask`)

**Step 3: Calculate Minimum Swaps**

For rows (`n = 4` is even):
- `rowMask = 0110` has 2 ones ✓ (exactly n/2)
- `sameRow = 2` ✓ (exactly n/2)
- Target with 1s at odd positions (`0xAAAAAAAA`): `6 & 0xAAAAAAAA = 2` (bit 1), 1 one already in place → `4 // 2 - 1 = 1` swap
- Target with 1s at even positions (`0x55555555`): `6 & 0x55555555 = 4` (bit 2), 1 one already in place → `4 // 2 - 1 = 1` swap
- Minimum row swaps = min(1, 1) = 1

For columns (same calculation):
- `colMask = 0110` has 2 ones ✓
- `sameCol = 2` ✓
- Both target patterns give 1 swap needed
- Minimum column swaps = 1

**Step 4: Final Result**

Total minimum moves = 1 (rows) + 1 (columns) = **2**

After 2 swaps, we can achieve:
```
Final board (one possible arrangement):
0 1 0 1
1 0 1 0
0 1 0 1
1 0 1 0
```

This example shows how:
1. We validate that only two complementary patterns exist
2. We use bit manipulation to efficiently compare patterns
3. We calculate swaps independently for rows and columns
4. The magic numbers help us quickly count mismatched positions

## Python Solution

```python
class Solution:
    def movesToChessboard(self, board: List[List[int]]) -> int:
        def calculate_min_swaps(pattern_mask: int, same_count: int) -> int:
            """
            Calculate minimum swaps needed to form alternating pattern.
          
            Args:
                pattern_mask: Bit representation of the row/column pattern
                same_count: Count of rows/columns matching the first row/column
          
            Returns:
                Minimum swaps needed, or -1 if impossible
            """
            ones_count = pattern_mask.bit_count()
          
            if n & 1:  # Odd board size
                # For odd n, difference between 0s and 1s must be exactly 1
                if abs(n - 2 * ones_count) != 1 or abs(n - 2 * same_count) != 1:
                    return -1
              
                # Check which pattern to use based on ones count
                if ones_count == n // 2:
                    # Pattern starts with 0 (0xAAAA... represents 1010...)
                    return n // 2 - (pattern_mask & 0xAAAAAAAA).bit_count()
                else:
                    # Pattern starts with 1 (0x5555... represents 0101...)
                    return (n + 1) // 2 - (pattern_mask & 0x55555555).bit_count()
            else:  # Even board size
                # For even n, must have equal 0s and 1s
                if ones_count != n // 2 or same_count != n // 2:
                    return -1
              
                # Try both patterns and choose minimum
                swaps_pattern_0 = n // 2 - (pattern_mask & 0xAAAAAAAA).bit_count()
                swaps_pattern_1 = n // 2 - (pattern_mask & 0x55555555).bit_count()
                return min(swaps_pattern_0, swaps_pattern_1)
      
        n = len(board)
        all_bits_mask = (1 << n) - 1  # Mask with all n bits set to 1
      
        # Create bit masks for first row and first column
        first_row_mask = 0
        first_col_mask = 0
        for i in range(n):
            first_row_mask |= board[0][i] << i
            first_col_mask |= board[i][0] << i
      
        # Create inverted masks (complement patterns)
        inverted_row_mask = all_bits_mask ^ first_row_mask
        inverted_col_mask = all_bits_mask ^ first_col_mask
      
        # Count rows/columns matching the first row/column pattern
        rows_matching_first = 0
        cols_matching_first = 0
      
        # Validate board and count matching patterns
        for i in range(n):
            current_row_mask = 0
            current_col_mask = 0
          
            # Build bit masks for current row and column
            for j in range(n):
                current_row_mask |= board[i][j] << j
                current_col_mask |= board[j][i] << j
          
            # Each row/column must match either the first pattern or its inverse
            if (current_row_mask not in (first_row_mask, inverted_row_mask) or 
                current_col_mask not in (first_col_mask, inverted_col_mask)):
                return -1
          
            # Count matches with first pattern
            rows_matching_first += (current_row_mask == first_row_mask)
            cols_matching_first += (current_col_mask == first_col_mask)
      
        # Calculate minimum swaps for rows and columns
        row_swaps = calculate_min_swaps(first_row_mask, rows_matching_first)
        col_swaps = calculate_min_swaps(first_col_mask, cols_matching_first)
      
        # Return total swaps or -1 if impossible
        if row_swaps == -1 or col_swaps == -1:
            return -1
        return row_swaps + col_swaps
```

## Time and Space Complexity

**Time Complexity:** `O(n²)`

- The first loop builds `first_row_mask` and `first_col_mask` from `n` cells each: `O(n)`
- The validation loop builds `current_row_mask` and `current_col_mask` for each of the `n` indices, reading `n` cells for each: `O(n²)`
- `calculate_min_swaps` does a constant number of AND and `bit_count()` operations on integers of at most `n ≤ 30` bits: `O(1)`

The nested validation loop dominates, giving `O(n²)` time.

**Space Complexity:** `O(1)`

The algorithm stores only a fixed number of integers (`all_bits_mask`, `first_row_mask`, `first_col_mask`, `inverted_row_mask`, `inverted_col_mask`, `rows_matching_first`, `cols_matching_first`, `current_row_mask`, `current_col_mask`, and a few locals in `calculate_min_swaps`). No data structure grows with `n`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Understanding of Swap Counting**

**Pitfall:** A common misconception is thinking that we need to count actual swap operations between misplaced rows/columns. For example, if row 0 should be at position 2 and row 2 should be at position 0, counting this as one swap.

**Why it's wrong:** The actual insight is that we only need to count how many rows/columns are in the wrong position relative to a target alternating pattern. Since we can swap any two rows/columns, the minimum number of swaps equals half the number of misplaced rows/columns (each swap fixes two positions).

**Solution:** Count the number of rows that should start with 0 but currently start with 1. Each swap moves one of these rows and one row of the opposite kind into place, so the number of swaps equals this count, which is the total number of misplaced rows divided by 2.

### 2. **Misunderstanding the Magic Numbers for Bit Patterns**

**Pitfall:** Using `0xAAAAAAAA` and `0x55555555` without considering the actual board size. These constants represent 32-bit patterns, but the board might be smaller.

**Why it's wrong:** For a board of size n < 32, we only care about the least significant n bits. The higher bits in these magic numbers don't affect the result due to the masking operation, but it's important to understand that:
- `0xAAAAAAAA & pattern_mask` effectively checks positions 1, 3, 5, 7... (odd positions)
- `0x55555555 & pattern_mask` effectively checks positions 0, 2, 4, 6... (even positions)

**Solution:** The code correctly handles this by only setting bits up to position n-1 in the pattern masks. The AND operation naturally limits the comparison to valid bit positions.

### 3. **Confusion About Which Pattern to Target**

**Pitfall:** Not understanding when to target pattern "0101..." vs "1010..." for odd-sized boards.

**Why it's wrong:** For odd n, one pattern will have more 1s than 0s. The board must match this constraint - if there are more 1s in the board, we must target the pattern with more 1s.

**Solution:** 
```python
if ones_count == n // 2:
    # We have fewer 1s, so pattern should be "01010..." (starts with 0)
    # Check positions that should be 1 (odd positions: 1, 3, 5...)
    return n // 2 - (pattern_mask & 0xAAAAAAAA).bit_count()
else:
    # We have more 1s, so pattern should be "10101..." (starts with 1)
    # Check positions that should be 1 (even positions: 0, 2, 4...)
    return (n + 1) // 2 - (pattern_mask & 0x55555555).bit_count()
```

### 4. **Off-by-One Errors in Swap Calculation**

**Pitfall:** Incorrectly calculating the number of swaps from the number of mismatched positions.

**Why it's wrong:** When we count positions that need to be 1 but are currently 0, we're identifying which rows need to be swapped. But remember: each swap fixes two positions simultaneously.

**Solution:** The formula `target_ones - actual_ones_in_target_positions` gives us the exact number of swaps needed. For example:
- If we need 4 positions to be 1 (n//2 for even n)
- And only 2 of those positions currently have 1
- We need 4 - 2 = 2 swaps to fix this

### 5. **Not Validating Board Feasibility Early**

**Pitfall:** Attempting to calculate swaps without first verifying that the board can form a valid chessboard pattern.

**Why it's wrong:** A board might have the correct number of 0s and 1s but still be impossible to transform into a chessboard if the row/column patterns don't meet the requirements.

**Solution:** The code correctly validates that:
1. There are exactly two distinct row patterns (and they're complements)
2. There are exactly two distinct column patterns (and they're complements)
3. The count of each pattern type is correct (n/2 for even n, or differs by 1 for odd n)

Only after these validations does it proceed to calculate swaps.

