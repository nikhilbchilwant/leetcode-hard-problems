# 1074. Number of Submatrices That Sum to Target

> Source: https://algo.monster/liteproblems/1074

## Problem Statement

You are given a 2D matrix of integers and a target value. Your task is to count how many submatrices within this matrix have elements that sum up to the target value.

A submatrix is defined by its top-left corner `(x1, y1)` and bottom-right corner `(x2, y2)`. It includes all cells `matrix[x][y]` where `x1 ≤ x ≤ x2` and `y1 ≤ y ≤ y2`.

Two submatrices are considered different if they have at least one different coordinate. For example, if the top-left corners are different, they are counted as separate submatrices even if they might contain the same values.

The solution uses a technique that combines:
1. **Dimension reduction**: The 2D problem is reduced to a 1D problem by fixing row boundaries and computing column sums
2. **Subarray sum counting**: For each fixed pair of rows `(i, j)`, it creates an array `col` where `col[k]` represents the sum of elements in column `k` from row `i` to row `j`
3. **Hash map optimization**: The helper function `f` counts subarrays with sum equal to target using the classic [prefix sum](/problems/subarray_sum) + hash map approach

The algorithm iterates through all possible top rows `i`, then for each top row, it considers all possible bottom rows `j ≥ i`. For each such pair, it maintains running column sums and uses the 1D subarray sum problem solution to count valid submatrices.

The time complexity is `O(m²n)` where `m` is the number of rows and `n` is the number of columns, as we iterate through all row pairs and process each column.

## Intuition

Finding all submatrices with a target sum in a 2D matrix seems overwhelming at first - we'd need to check every possible rectangle, which could be `O(m²n²)` rectangles, and calculating each sum could take `O(mn)` time, leading to a very inefficient solution.

The key insight is recognizing that this 2D problem can be transformed into multiple 1D problems. We know how to efficiently solve "count subarrays with target sum" in 1D using prefix sums and a hash map in `O(n)` time. Can we use this?

Think about fixing the top and bottom boundaries of our submatrix. If we fix rows `i` (top) and `j` (bottom), then any submatrix between these rows is determined solely by which columns we choose. More importantly, if we compress all values between rows `i` and `j` into a single array by summing each column, finding submatrices becomes equivalent to finding subarrays in this compressed array.

For example, if we have a matrix and fix rows 1 to 3, we can create an array where each element is the sum of that column from row 1 to row 3. Now finding a submatrix with target sum between columns `c1` and `c2` is the same as finding a subarray with target sum in our compressed array.

This transformation works because:
- The sum of a submatrix from `(i, c1)` to `(j, c2)` equals the sum of compressed array elements from index `c1` to `c2`
- We can incrementally build these compressed arrays as we expand our bottom boundary, avoiding redundant calculations

By iterating through all possible pairs of top and bottom rows `(i, j)` where `i ≤ j`, we ensure we consider every possible submatrix. For each pair, we solve the 1D subarray problem, accumulating the total count. This reduces our time complexity from potentially `O(m²n² × mn)` to `O(m²n)`, making the problem tractable.

**Learn more about [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The implementation consists of two main components: a helper function for the 1D subarray problem and the main logic that reduces the 2D problem to multiple 1D problems.

**Helper Function `f(nums)`:**
This function counts subarrays with sum equal to `target` using the [prefix sum](/problems/subarray_sum) technique:
- Initialize a hash map `d` with `d[0] = 1` (representing empty prefix)
- Maintain a running sum `s` and counter `cnt`
- For each element `x` in `nums`:
  - Add `x` to running sum: `s += x`
  - Check if `s - target` exists in the hash map. If it does, it means there are `d[s - target]` subarrays ending at current position with sum equal to `target`
  - Add this count to `cnt`: `cnt += d[s - target]`
  - Update hash map: `d[s] += 1` to record this prefix sum

**Main Algorithm:**
1. Get matrix dimensions: `m` rows and `n` columns
2. Initialize answer counter `ans = 0`
3. Iterate through all possible top rows `i` from `0` to `m-1`:
   - Create a column sum array `col` of size `n`, initialized to zeros
   - For each possible bottom row `j` from `i` to `m-1`:
     - **Update column sums**: For each column `k`, add `matrix[j][k]` to `col[k]`
       - At this point, `col[k]` contains the sum of elements in column `k` from row `i` to row `j`
     - **Solve 1D problem**: Call `f(col)` to count how many subarrays in `col` sum to `target`
       - Each valid subarray represents a submatrix with corners `(i, left)` and `(j, right)`
     - Add the count to `ans`: `ans += f(col)`
4. Return the total count `ans`

**Why this works:**
- By fixing rows `i` and `j`, we reduce the problem to finding contiguous column ranges
- The column sum array `col` represents the "compressed" view of the submatrix between rows `i` and `j`
- The incremental update of `col` (adding one row at a time) ensures we don't redundantly recalculate sums
- Each call to `f(col)` efficiently counts all valid submatrices with top row `i` and bottom row `j`

**Time Complexity:** `O(m²n)` - we have `O(m²)` row pairs, and for each pair, we process `n` columns
**Space Complexity:** `O(n)` - for the column sum array and hash map in function `f`

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this 3×3 matrix with target = 4:
```
matrix = [[1, 2, 1],
          [1, 1, 1],
          [2, 0, 1]]
target = 4
```

**Step 1: Fix top row i = 0**

When j = 0 (considering only row 0):
- col = [1, 2, 1] (just row 0)
- Find subarrays in [1, 2, 1] that sum to 4:
  - Running sums: 1, 3, 4
  - Subarray [1, 2, 1] sums to 4 ✓
  - Count = 1

When j = 1 (considering rows 0-1):
- col = [1+1, 2+1, 1+1] = [2, 3, 2] (sum of rows 0-1)
- Find subarrays in [2, 3, 2] that sum to 4:
  - No subarrays sum to 4
  - Count = 0

When j = 2 (considering rows 0-2):
- col = [2+2, 3+0, 2+1] = [4, 3, 3] (sum of rows 0-2)
- Find subarrays in [4, 3, 3] that sum to 4:
  - Subarray [4] sums to 4 ✓
  - Count = 1

**Step 2: Fix top row i = 1**

When j = 1 (considering only row 1):
- col = [1, 1, 1] (just row 1)
- Find subarrays in [1, 1, 1] that sum to 4:
  - No subarrays sum to 4
  - Count = 0

When j = 2 (considering rows 1-2):
- col = [1+2, 1+0, 1+1] = [3, 1, 2] (sum of rows 1-2)
- Find subarrays in [3, 1, 2] that sum to 4:
  - Running sums: 3, 4, 6
  - Subarray [3, 1] sums to 4 ✓
  - Count = 1

**Step 3: Fix top row i = 2**

When j = 2 (considering only row 2):
- col = [2, 0, 1] (just row 2)
- Find subarrays in [2, 0, 1] that sum to 4:
  - No subarrays sum to 4
  - Count = 0

**Total count = 1 + 0 + 1 + 0 + 1 + 0 = 3**

The three submatrices that sum to 4 are:
1. The entire first row: [[1, 2, 1]] 
2. The first column from rows 0-2: [[1], [1], [2]]
3. The first two columns from rows 1-2: [[1, 1], [2, 0]]

This demonstrates how the algorithm systematically examines all possible row boundaries and reduces each configuration to a 1D subarray sum problem.

## Python Solution

```python
class Solution:
    def numSubmatrixSumTarget(self, matrix: List[List[int]], target: int) -> int:
        def count_subarrays_with_target_sum(nums: List[int]) -> int:
            """
            Count the number of subarrays in nums that sum to target.
            Uses prefix sum with hashmap to achieve O(n) time complexity.
            """
            # Store frequency of each prefix sum
            prefix_sum_count = defaultdict(int)
            # Empty subarray has sum 0
            prefix_sum_count[0] = 1
          
            count = 0
            current_sum = 0
          
            for num in nums:
                current_sum += num
                # If (current_sum - target) exists in map, we found subarrays ending here
                # that sum to target
                count += prefix_sum_count[current_sum - target]
                # Add current prefix sum to map
                prefix_sum_count[current_sum] += 1
          
            return count
      
        rows, cols = len(matrix), len(matrix[0])
        result = 0
      
        # Fix top row of submatrix
        for top_row in range(rows):
            # Initialize column sums for current top row
            column_sums = [0] * cols
          
            # Extend bottom row of submatrix
            for bottom_row in range(top_row, rows):
                # Update column sums to include current bottom row
                for col in range(cols):
                    column_sums[col] += matrix[bottom_row][col]
              
                # Now column_sums[k] = sum of elements in column k from top_row to bottom_row
                # Find number of column ranges that sum to target
                result += count_subarrays_with_target_sum(column_sums)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(m² × n)`

The algorithm uses a technique that combines 2D prefix sum with the subarray sum problem:
- The outer loop runs from `i = 0` to `m-1`, iterating `m` times
- The middle loop runs from `j = i` to `m-1`, which in worst case iterates `O(m)` times
- Together, these two nested loops create `O(m²)` iterations
- For each pair `(i, j)`, we:
  - Update the column sums by iterating through `n` columns: `O(n)`
  - Call function `f(col)` which iterates through the array of size `n` once: `O(n)`
- Total operations per `(i, j)` pair: `O(n)`
- Overall time complexity: `O(m²) × O(n) = O(m² × n)`

**Space Complexity:** `O(n)`

The space usage consists of:
- `col` array of size `n`: `O(n)`
- Inside function `f`, a dictionary `d` that stores at most `n+1` unique prefix sums (one for each position plus the initial 0): `O(n)`
- Other variables (`cnt`, `s`, `ans`, loop variables): `O(1)`
- The maximum space used at any point is `O(n)`

Note that we don't count the input matrix in the space complexity as it's given as input. The algorithm efficiently reuses the `col` array across iterations rather than storing all possible column sums simultaneously.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Initialization of Prefix Sum HashMap**
A critical pitfall is forgetting to initialize the prefix sum hashmap with `prefix_sum_count[0] = 1`. This initialization represents the empty prefix (no elements selected yet) and is essential for correctly counting subarrays that start from the beginning.

**Wrong Implementation:**
```python
def count_subarrays_with_target_sum(nums):
    prefix_sum_count = defaultdict(int)
    # Missing: prefix_sum_count[0] = 1
    count = 0
    current_sum = 0
  
    for num in nums:
        current_sum += num
        count += prefix_sum_count[current_sum - target]
        prefix_sum_count[current_sum] += 1
    return count
```

**Why it fails:** Without `prefix_sum_count[0] = 1`, subarrays starting from index 0 won't be counted. For example, if `nums = [5]` and `target = 5`, the count would incorrectly be 0 instead of 1.

### 2. **Column Sum Array Not Reset Between Different Top Rows**
Another common mistake is reusing the same column sum array across different top rows without resetting it, leading to incorrect accumulated sums.

**Wrong Implementation:**
```python
rows, cols = len(matrix), len(matrix[0])
result = 0
column_sums = [0] * cols  # Created once outside the loop

for top_row in range(rows):
    # column_sums not reset - carries over values from previous iterations
    for bottom_row in range(top_row, rows):
        for col in range(cols):
            column_sums[col] += matrix[bottom_row][col]
        result += count_subarrays_with_target_sum(column_sums)
```

**Correct Implementation:**
```python
for top_row in range(rows):
    column_sums = [0] * cols  # Reset for each new top row
    for bottom_row in range(top_row, rows):
        # ... rest of the code
```

### 3. **Order of Operations in Prefix Sum Calculation**
Updating the hashmap before checking for complementary sums can lead to double-counting when the target is 0 or when specific patterns exist.

**Potentially Problematic:**
```python
for num in nums:
    current_sum += num
    prefix_sum_count[current_sum] += 1  # Update first
    count += prefix_sum_count[current_sum - target]  # Then check
```

**Correct Order:**
```python
for num in nums:
    current_sum += num
    count += prefix_sum_count[current_sum - target]  # Check first
    prefix_sum_count[current_sum] += 1  # Then update
```

The correct order ensures we're looking for previously seen prefix sums, not including the current position itself in the count.

### 4. **Integer Overflow in Large Matrices**
While Python handles arbitrary precision integers automatically, in other languages or when optimizing with NumPy, large matrices with large values could cause integer overflow when computing sums.

**Solution:** Be aware of the value ranges in your matrix and use appropriate data types (e.g., `long long` in C++ or explicit dtype specification in NumPy).

