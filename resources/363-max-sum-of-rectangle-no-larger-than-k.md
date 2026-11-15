# 363. Max Sum of Rectangle No Larger Than K

> Source: https://algo.monster/liteproblems/363

## Problem Statement

You are given an `m x n` matrix filled with integers and a target value `k`. Your task is to find a rectangle (submatrix) within this matrix that has the maximum possible sum, with the constraint that this sum must not exceed `k`.

The problem guarantees that at least one valid rectangle exists with a sum less than or equal to `k`.

For example, if you have a matrix and `k = 2`, you need to find all possible rectangles within the matrix, calculate their sums, filter out those with sums greater than `k`, and return the maximum sum among the remaining valid rectangles.

The solution approach transforms this 2D problem into a 1D problem by:
1. Fixing the top and bottom boundaries of potential rectangles (rows `i` to `j`)
2. Compressing the rows between these boundaries into a single array `nums` where each element represents the column sum within the fixed row boundaries
3. Finding the maximum subarray sum in `nums` that doesn't exceed `k` using a sorted set to efficiently track prefix sums

The sorted set maintains prefix sums seen so far. For each new prefix sum `s`, we search for the smallest prefix sum `p` such that `s - p ≤ k` (equivalently, `p ≥ s - k`). The difference `s - p` gives us a subarray sum that doesn't exceed `k`, and we track the maximum such sum across all possible rectangles.

## Intuition

The key insight is recognizing that finding the maximum sum rectangle in a 2D matrix can be reduced to solving multiple 1D maximum subarray problems.

Think about how rectangles are formed in a matrix - every rectangle has four boundaries: top, bottom, left, and right. If we fix the top and bottom boundaries, we essentially "compress" multiple rows into a single row where each element represents the sum of elements in that column between our fixed boundaries.

For instance, if we fix rows `i` to `j`, we can create an array where `nums[col] = sum of all elements in column col from row i to row j`. Now the problem becomes: find the maximum subarray sum in this 1D array `nums` that doesn't exceed `k`.

The 1D subarray problem with a constraint is where the clever use of prefix sums and a sorted set comes in. When we have a prefix sum `s` at the current position, we want to find a previous prefix sum `p` such that `s - p` (the subarray sum) is as large as possible but still `≤ k`. This means we need `p ≥ s - k`, and we want the smallest such `p` to maximize `s - p`.

A sorted set allows us to efficiently find this smallest `p` that is greater than or equal to `s - k` using binary search (`bisect_left`). We maintain all previous prefix sums in the sorted set, including 0 (for subarrays starting from the beginning).

By trying all possible pairs of top and bottom boundaries and solving the 1D problem for each, we ensure we've considered all possible rectangles in the matrix and found the one with maximum sum not exceeding `k`.

## Solution Approach

The implementation follows a three-layer nested approach to explore all possible rectangles:

**Step 1: Enumerate Top Boundaries**
We iterate through each possible top row `i` from `0` to `m-1`. This will serve as the upper boundary of our rectangles.

**Step 2: Enumerate Bottom Boundaries**
For each fixed top row `i`, we enumerate all possible bottom rows `j` from `i` to `m-1`. This gives us all possible row ranges `[i, j]`.

**Step 3: Compress Rows into Column Sums**
For each row range `[i, j]`, we maintain an array `nums` where `nums[h]` represents the sum of elements in column `h` from row `i` to row `j`. This is done incrementally:
- When `j = i`, we initialize `nums[h] = matrix[i][h]`
- As we extend to `j+1`, we update `nums[h] += matrix[j+1][h]`

**Step 4: Find Maximum Subarray Sum ≤ k**
With the compressed 1D array `nums`, we need to find the maximum subarray sum that doesn't exceed `k`. This is accomplished using:

1. **Prefix Sum Calculation**: We maintain a running sum `s` as we traverse the array.

2. **Sorted Set for Efficient Search**: We use a `SortedSet` (ordered set) to store all previous prefix sums, initialized with `0` to handle subarrays starting from index 0.

3. **Binary Search for Valid Subarrays**: For each position with prefix sum `s`:
   - We search for the smallest prefix sum `p` where `p >= s - k` using `bisect_left(s - k)`
   - If such a `p` exists, the subarray sum `s - p` is valid (≤ k)
   - We update our answer with `max(ans, s - p)`
   - Add current prefix sum `s` to the sorted set for future iterations

**Time Complexity**: `O(m² × n × log n)` where:
- `O(m²)` for enumerating all row ranges
- `O(n)` for processing each column
- `O(log n)` for binary search operations in the sorted set

**Space Complexity**: `O(n)` for the `nums` array and the sorted set which can contain at most `n` prefix sums.

The algorithm guarantees finding the maximum sum rectangle not exceeding `k` by exhaustively checking all possible rectangles through the row boundary enumeration and efficiently solving the constrained 1D subarray problem.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider the matrix:
```
matrix = [[2, 1],
          [1, 2]]
k = 3
```

**Step 1: Fix top boundary at row 0 (i = 0)**

- **Bottom boundary at row 0 (j = 0):**
  - Compress rows [0,0] into column sums: `nums = [2, 1]`
  - Find max subarray sum ≤ 3 in `nums`:
    - Start with sorted set = {0}
    - Position 0: prefix sum s = 2
      - Search for p ≥ 2-3 = -1, found p = 0
      - Subarray sum = 2-0 = 2 ≤ 3 ✓
      - Update answer = 2
      - Add 2 to set: {0, 2}
    - Position 1: prefix sum s = 2+1 = 3
      - Search for p ≥ 3-3 = 0, found p = 0
      - Subarray sum = 3-0 = 3 ≤ 3 ✓
      - Update answer = 3
      - Add 3 to set: {0, 2, 3}

- **Bottom boundary at row 1 (j = 1):**
  - Compress rows [0,1] into column sums: `nums = [2+1, 1+2] = [3, 3]`
  - Find max subarray sum ≤ 3 in `nums`:
    - Start with sorted set = {0}
    - Position 0: prefix sum s = 3
      - Search for p ≥ 3-3 = 0, found p = 0
      - Subarray sum = 3-0 = 3 ≤ 3 ✓
      - Answer remains 3
      - Add 3 to set: {0, 3}
    - Position 1: prefix sum s = 3+3 = 6
      - Search for p ≥ 6-3 = 3, found p = 3
      - Subarray sum = 6-3 = 3 ≤ 3 ✓
      - Answer remains 3

**Step 2: Fix top boundary at row 1 (i = 1)**

- **Bottom boundary at row 1 (j = 1):**
  - Compress rows [1,1] into column sums: `nums = [1, 2]`
  - Find max subarray sum ≤ 3 in `nums`:
    - Position 0: prefix sum s = 1, subarray sum = 1 ≤ 3 ✓
    - Position 1: prefix sum s = 3, subarray sum = 3 ≤ 3 ✓
    - Answer remains 3

**Final Result: 3**

The maximum rectangle sum not exceeding k=3 is achieved by:
- The first row [2, 1] with sum 3
- The second row [1, 2] with sum 3
- The first column [2, 1] with sum 3
- The second column [1, 2] with sum 3

All have sum = 3, which is the maximum possible without exceeding k.

## Python Solution

```python
from typing import List
from sortedcontainers import SortedList
import math

class Solution:
    def maxSumSubmatrix(self, matrix: List[List[int]], k: int) -> int:
        """
        Find the maximum sum of a rectangle in the matrix such that its sum is no larger than k.
      
        Args:
            matrix: 2D list of integers
            k: Target upper bound for rectangle sum
          
        Returns:
            Maximum sum of rectangle that doesn't exceed k
        """
        rows, cols = len(matrix), len(matrix[0])
        max_sum = -math.inf
      
        # Fix the top row of the rectangle
        for top_row in range(rows):
            # Initialize column sums for current top row
            column_sums = [0] * cols
          
            # Extend the rectangle by moving the bottom row down
            for bottom_row in range(top_row, rows):
                # Add current row values to column sums
                for col in range(cols):
                    column_sums[col] += matrix[bottom_row][col]
              
                # Now we have a 1D array problem: find max subarray sum <= k
                # Use prefix sum with sorted set to efficiently find the answer
                current_sum = 0
                # Store prefix sums in sorted order, initialize with 0
                prefix_sums = SortedList([0])
              
                for value in column_sums:
                    current_sum += value
                    # We need to find smallest prefix_sum such that:
                    # current_sum - prefix_sum <= k
                    # Which means: prefix_sum >= current_sum - k
                    target = current_sum - k
                    index = prefix_sums.bisect_left(target)
                  
                    # If we found a valid prefix sum
                    if index < len(prefix_sums):
                        # Update max_sum with the rectangle sum
                        max_sum = max(max_sum, current_sum - prefix_sums[index])
                  
                    # Add current prefix sum for future iterations
                    prefix_sums.add(current_sum)
      
        return max_sum
```

## Time and Space Complexity

**Time Complexity:** `O(m^2 × n × log n)`

The analysis breaks down as follows:
- The outer loop iterates through all possible top rows: `O(m)` iterations
- The middle loop iterates through all possible bottom rows starting from the current top row: `O(m)` iterations in total
- For each rectangle defined by top and bottom rows:
  - We compute the column sums by iterating through all columns: `O(n)`
  - We then iterate through the column sums array: `O(n)` iterations
  - For each element in the column sums, we perform:
    - `bisect_left` operation on the SortedSet: `O(log n)` as the set can contain at most `n+1` elements
    - `add` operation to the SortedSet: `O(log n)`

Combining these operations: `O(m) × O(m) × [O(n) + O(n) × O(log n)]` = `O(m^2 × n × log n)`

**Space Complexity:** `O(n)`

The space usage consists of:
- `nums` array to store column sums: `O(n)`
- `SortedSet ts` which stores at most `n+1` prefix sums (one initial 0 and one for each column): `O(n)`
- Other variables like `ans`, `s`, `p`: `O(1)`

The dominant space usage is `O(n)`, giving us an overall space complexity of `O(n)`.

## Common Pitfalls

### 1. **Incorrect Initialization of Prefix Sum Set**

**Pitfall**: Forgetting to initialize the sorted set with `0`, which causes the algorithm to miss rectangles that start from the leftmost column.

**Wrong Implementation**:
```python
prefix_sums = SortedList()  # Missing the initial 0
```

**Why It's Wrong**: Without the initial `0` in the prefix sum set, we cannot properly handle subarrays that start from index 0. For example, if the first few columns sum to a valid value ≤ k, we need `0` in our set to compute `current_sum - 0`.

**Correct Implementation**:
```python
prefix_sums = SortedList([0])  # Initialize with 0
```

### 2. **Using Wrong Binary Search Method**

**Pitfall**: Using `bisect_right` instead of `bisect_left` when searching for the smallest valid prefix sum.

**Wrong Implementation**:
```python
index = prefix_sums.bisect_right(target)  # Wrong method
```

**Why It's Wrong**: We need the smallest prefix sum `p` such that `p >= current_sum - k`. Using `bisect_right` would give us the insertion point after all equal values, potentially missing valid solutions when there are duplicate prefix sums.

**Correct Implementation**:
```python
index = prefix_sums.bisect_left(target)  # Find leftmost valid position
```

### 3. **Reinitializing Column Sums Incorrectly**

**Pitfall**: Creating a new column_sums array for each bottom_row instead of accumulating values.

**Wrong Implementation**:
```python
for bottom_row in range(top_row, rows):
    column_sums = [matrix[bottom_row][col] for col in range(cols)]  # Wrong: overwrites previous sums
```

**Why It's Wrong**: This resets the column sums for each new bottom row, losing the accumulation from previous rows in the range [top_row, bottom_row-1].

**Correct Implementation**:
```python
column_sums = [0] * cols  # Initialize once per top_row
for bottom_row in range(top_row, rows):
    for col in range(cols):
        column_sums[col] += matrix[bottom_row][col]  # Accumulate
```

### 4. **Adding Prefix Sum Before Checking**

**Pitfall**: Adding the current prefix sum to the sorted set before using it to find valid subarrays.

**Wrong Implementation**:
```python
for value in column_sums:
    current_sum += value
    prefix_sums.add(current_sum)  # Added too early
    target = current_sum - k
    index = prefix_sums.bisect_left(target)
    # ... rest of logic
```

**Why It's Wrong**: This lets the current prefix sum pair with itself, which represents an empty subarray with sum 0. An empty rectangle is not allowed, so the result can be too large. For `matrix = [[-5]]` and `k = 0`, the set becomes `{-5, 0}`, the search for `p >= -5` finds `-5`, and the code returns 0 instead of -5.

**Correct Implementation**:
```python
for value in column_sums:
    current_sum += value
    target = current_sum - k
    index = prefix_sums.bisect_left(target)
    if index < len(prefix_sums):
        max_sum = max(max_sum, current_sum - prefix_sums[index])
    prefix_sums.add(current_sum)  # Add after checking
```

### 5. **Not Handling Edge Cases with Initial Maximum**

**Pitfall**: Initializing max_sum with 0 or a small fixed value instead of negative infinity.

**Wrong Implementation**:
```python
max_sum = 0  # Wrong if all valid rectangles have negative sums
```

**Why It's Wrong**: If all elements in the matrix are negative and k is negative, the maximum valid rectangle sum could be negative. Starting with 0 would incorrectly return 0.

**Correct Implementation**:
```python
max_sum = -math.inf  # Or float('-inf')
```

