# 1183. Maximum Number of Ones

> Source: https://algo.monster/liteproblems/1183

## Problem Statement

You have a matrix `M` with dimensions `width × height`. Every cell in this matrix can only contain either 0 or 1. There's a special constraint: any square sub-matrix of size `sideLength × sideLength` within `M` can have at most `maxOnes` ones.

Your task is to find the maximum possible number of ones that can be placed in the entire matrix `M` while respecting this constraint.

For example, if you have a 3×3 matrix with `sideLength = 2` and `maxOnes = 1`, then every 2×2 square region in the matrix can contain at most 1 one. You need to strategically place ones to maximize the total count while ensuring no 2×2 square violates this rule.

## Intuition

Think about overlaying a grid of `sideLength × sideLength` squares across the entire matrix. Each square can have at most `maxOnes` ones. The key observation is that these squares repeat in a pattern - they tile the matrix.

Imagine we have a single `sideLength × sideLength` template square. When we tile this template across the matrix, position `(0, 0)` in the template appears at positions `(0, 0)`, `(0, sideLength)`, `(0, 2×sideLength)`, ... and also at `(sideLength, 0)`, `(2×sideLength, 0)`, etc. in the actual matrix.

Each position `(i, j)` in the matrix belongs to exactly one position in this template square, specifically position `(i % sideLength, j % sideLength)`. This means that if we decide to place a 1 at position `(r, c)` in the template (where `r < sideLength` and `c < sideLength`), then ALL positions `(i, j)` in the matrix where `i % sideLength = r` and `j % sideLength = c` must also be 1.

Since we can only choose `maxOnes` positions in the template to set as 1, we want to choose the positions that appear most frequently when the template is tiled across the entire matrix. A position `(r, c)` in the template appears exactly `⌈width/sideLength⌉ × ⌈height/sideLength⌉` times (accounting for partial tiles at the edges).

By counting how many times each template position appears in the full matrix and selecting the `maxOnes` positions with the highest counts, we maximize the total number of ones in the matrix while respecting the constraint that each `sideLength × sideLength` square has at most `maxOnes` ones.

**Learn more about [Greedy](/problems/greedy_intro), [Math](/problems/math-basics), [Sorting](/problems/sorting_summary) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation follows the pattern-based counting strategy identified in our intuition. Let's denote `x = sideLength` for simplicity.

We create an array `cnt` of size `x * x` to represent all possible positions in our template square. Each index in this array corresponds to a unique position `(i % x, j % x)` in the template.

For each cell `(i, j)` in the matrix:
1. Calculate its corresponding position in the template: `(i % x, j % x)`
2. Convert this 2D position to a 1D index: `k = (i % x) * x + (j % x)`
3. Increment the counter at `cnt[k]` by 1

This counting process tells us how many times each template position appears when tiled across the entire `width × height` matrix.

After counting all positions:
1. Sort the `cnt` array in descending order to prioritize positions that appear most frequently
2. Select the top `maxOnes` positions (these are the positions we'll set to 1 in our template)
3. Return the sum of counts for these selected positions

The key insight is that by selecting the `maxOnes` most frequent positions, we maximize the total number of ones. Since each selected template position forces all its corresponding positions in the matrix to be 1, choosing the positions with highest frequency gives us the maximum possible ones.

Time complexity: `O(width * height + x²log(x²))` where the first term is for counting and the second is for [sorting](/problems/sorting_summary).
Space complexity: `O(x²)` for the counting array.

## Example Walkthrough

Let's work through a concrete example with a 5×4 matrix where `sideLength = 2` and `maxOnes = 1`.

**Step 1: Understanding the Template**
We have a 2×2 template square with positions:
```
Template positions:
(0,0) (0,1)
(1,0) (1,1)
```

**Step 2: Mapping Matrix Positions to Template**
Each position in our 5×4 matrix maps to a template position based on `(i % 2, j % 2)`:

```
Matrix (5×4):         Template Position:
[0,0] [0,1] [0,2] [0,3]    (0,0) (0,1) (0,0) (0,1)
[1,0] [1,1] [1,2] [1,3]    (1,0) (1,1) (1,0) (1,1)
[2,0] [2,1] [2,2] [2,3]    (0,0) (0,1) (0,0) (0,1)
[3,0] [3,1] [3,2] [3,3]    (1,0) (1,1) (1,0) (1,1)
[4,0] [4,1] [4,2] [4,3]    (0,0) (0,1) (0,0) (0,1)
```

**Step 3: Count Frequencies**
Now we count how many times each template position appears:
- Position (0,0): appears at matrix positions [0,0], [0,2], [2,0], [2,2], [4,0], [4,2] → **6 times**
- Position (0,1): appears at matrix positions [0,1], [0,3], [2,1], [2,3], [4,1], [4,3] → **6 times**
- Position (1,0): appears at matrix positions [1,0], [1,2], [3,0], [3,2] → **4 times**
- Position (1,1): appears at matrix positions [1,1], [1,3], [3,1], [3,3] → **4 times**

Converting to 1D array indices (using `index = row * 2 + col`):
- `cnt[0]` = 6 (for template position (0,0))
- `cnt[1]` = 6 (for template position (0,1))
- `cnt[2]` = 4 (for template position (1,0))
- `cnt[3]` = 4 (for template position (1,1))

**Step 4: Select Top maxOnes Positions**
Sort counts in descending order: [6, 6, 4, 4]

Since `maxOnes = 1`, we select only 1 position. We can choose either position with count 6 (template positions (0,0) or (0,1)).

**Step 5: Calculate Result**
Maximum ones possible = 6

**Verification:**
If we choose template position (0,0), our matrix would look like:
```
[1] [0] [1] [0]
[0] [0] [0] [0]
[1] [0] [1] [0]
[0] [0] [0] [0]
[1] [0] [1] [0]
```

Every 2×2 square contains exactly 1 one:
- Square at [0,0]: contains [0,0]=1, others=0 ✓
- Square at [0,1]: contains [0,2]=1, others=0 ✓
- Square at [1,0]: contains [2,0]=1, others=0 ✓
- And so on...

The constraint is satisfied, and we've placed the maximum possible 6 ones.

## Python Solution

```python
class Solution:
    def maximumNumberOfOnes(
        self, width: int, height: int, sideLength: int, maxOnes: int
    ) -> int:
        """
        Calculate the maximum number of 1s that can be placed in a width x height matrix
        such that each sideLength x sideLength submatrix has at most maxOnes 1s.

        Strategy: The matrix can be tiled by sideLength x sideLength submatrices.
        Each position in the submatrix pattern appears multiple times across the full matrix.
        We count how many times each position appears, then greedily select the most frequent positions.

        Args:
            width: Width of the matrix
            height: Height of the matrix
            sideLength: Side length of each square submatrix
            maxOnes: Maximum number of 1s allowed in each submatrix

        Returns:
            Maximum total number of 1s that can be placed in the matrix
        """
        # Store frequency count for each position in the sideLength x sideLength pattern
        position_frequencies = [0] * (sideLength * sideLength)

        # Count how many times each position in the pattern appears in the full matrix
        for row in range(width):
            for col in range(height):
                # Map the current position to its corresponding position in the pattern
                # Using row-major indexing: position = (row_in_pattern * sideLength + col_in_pattern)
                pattern_position = (row % sideLength) * sideLength + (col % sideLength)
                position_frequencies[pattern_position] += 1

        # Sort positions by frequency in descending order to maximize the result
        position_frequencies.sort(reverse=True)

        # Select the top maxOnes positions with highest frequencies
        # and sum their frequencies to get the maximum number of 1s
        return sum(position_frequencies[:maxOnes])
```

## Time and Space Complexity

**Time Complexity:** `O(width × height)`

The algorithm iterates through every position in the matrix exactly once using two nested loops - the outer loop runs `width` times and the inner loop runs `height` times. For each position `(i, j)`, it performs constant time operations: calculating the modulo, computing index `k`, and incrementing a counter. The sorting operation takes `O(sideLength² × log(sideLength²))` time, but since `sideLength ≤ min(width, height)`, this is bounded by `O(width × height)`. Therefore, the overall time complexity is `O(width × height)`, which matches the reference answer's `O(m × n)` where `m` and `n` represent the dimensions of the matrix.

**Space Complexity:** `O(sideLength²)`

The algorithm uses an array `cnt` of size `sideLength × sideLength` to store the frequency counts for each unique position within a `sideLength × sideLength` subgrid. This is the dominant space usage in the algorithm, as all other variables use constant space. Therefore, the space complexity is `O(sideLength²)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Confusion Between Matrix Dimensions and Iteration Order**

A critical pitfall in this solution is mixing up the iteration variables with matrix dimensions. The problem states the matrix has dimensions `width × height`, which typically means `width` columns and `height` rows. However, the code iterates using:

```python
for row in range(width):
    for col in range(height):
```

This treats `width` as the number of rows and `height` as the number of columns, which is counterintuitive and could lead to incorrect results if the matrix is not square.

**Solution:**
Use clearer variable naming and iterate consistently with the matrix dimensions:

```python
# Iterate through all cells in the height × width matrix
for row in range(height):
    for col in range(width):
        pattern_position = (row % sideLength) * sideLength + (col % sideLength)
        position_frequencies[pattern_position] += 1
```

### 2. **Incorrect Pattern Position Calculation**

Another potential pitfall is incorrectly mapping 2D coordinates to 1D array indices. The formula `(row % sideLength) * sideLength + (col % sideLength)` assumes row-major ordering, but developers might accidentally use column-major ordering or mix up the formula.

**Solution:**
Add validation or use a helper function to make the mapping explicit:

```python
def get_pattern_index(row, col, sideLength):
    """Convert 2D pattern position to 1D array index."""
    pattern_row = row % sideLength
    pattern_col = col % sideLength
    return pattern_row * sideLength + pattern_col
```

### 3. **Edge Case: maxOnes Greater Than sideLength²**

If `maxOnes > sideLength * sideLength`, the code would work correctly but it's worth noting that this means we can fill the entire matrix with ones since the constraint becomes meaningless.

**Solution:**
Add a optimization check at the beginning:

```python
if maxOnes >= sideLength * sideLength:
    return width * height  # Can place 1s everywhere
```

### 4. **Integer Overflow in Large Inputs**

For very large matrices, the frequency counts could potentially overflow in languages with fixed-size integers (though Python handles this automatically).

**Solution:**
In Python this isn't an issue, but in other languages, ensure you're using appropriate data types for large values or add bounds checking.

