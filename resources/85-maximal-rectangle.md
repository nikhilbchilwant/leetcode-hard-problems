# 85. Maximal Rectangle

> Source: https://algo.monster/liteproblems/85

## Problem Statement

You are given a 2D binary matrix of size `rows x cols` where each cell contains either '0' or '1'. Your task is to find the largest rectangle that contains only '1's and return its area.

A rectangle in the matrix is defined by its corners and must be axis-aligned (sides parallel to the matrix edges). The rectangle can only include cells that contain '1'.

For example, if you have a matrix like:
```
1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0
```

The largest rectangle containing only 1's has an area of 6 (formed by the '1's in the second and third rows, from columns 2 to 4).

The solution works by transforming this problem into multiple "Largest Rectangle in Histogram" problems. For each row, it treats that row as the base and calculates the height of consecutive '1's above (including the current row). This creates a histogram for each row, where the height represents how many consecutive '1's extend upward from that position.

The algorithm:
1. For each row, build a histogram where each bar's height represents the number of consecutive '1's from that row going upward
2. If a cell contains '0', the height resets to 0
3. If a cell contains '1', the height increases by 1 from the previous row's height at that position
4. For each row's histogram, calculate the largest rectangle area using a monotonic [stack](/problems/stack_intro) approach
5. Track the maximum area found across all rows

The `largestRectangleArea` helper function uses a monotonic [stack](/problems/stack_intro) to find the largest rectangle in a histogram by determining, for each bar, the furthest left and right positions where the height is at least as tall as the current bar.

## Intuition

The key insight is recognizing that this problem can be reduced to a simpler, well-known problem: finding the largest rectangle in a histogram.

Imagine looking at the matrix from the side. If we fix a particular row as our "base," we can count how many consecutive '1's extend upward from each position in that row. This gives us a histogram where each bar's height represents the vertical stretch of '1's at that column.

For instance, consider this small matrix:
```
1 0 1
1 1 1
```

If we use the second row as our base:
- Column 0: has 2 consecutive '1's going up (including current row) → height = 2
- Column 1: has 1 consecutive '1' (just the current row, since above is '0') → height = 1  
- Column 2: has 2 consecutive '1's going up → height = 2

This gives us a histogram with heights `[2, 1, 2]`.

The brilliant part is that any rectangle in the original matrix that ends at a particular row can be represented as a rectangle in that row's histogram. Why? Because:
- The width of the rectangle in the histogram corresponds to the width in the matrix
- The height of the rectangle in the histogram represents how many rows up the rectangle extends

By processing each row and treating it as a potential bottom edge of our rectangle, we ensure we consider all possible rectangles in the matrix. The heights array is built incrementally: when we encounter a '1', we add to the previous height (extending the bar upward), and when we encounter a '0', we reset to 0 (breaking the consecutive chain).

Once we have the histogram for each row, we can apply the classic "largest rectangle in histogram" algorithm using a monotonic [stack](/problems/stack_intro). This algorithm efficiently finds, for each bar, how far left and right we can extend while maintaining at least that bar's height, allowing us to calculate the maximum possible rectangle area for each bar as a potential minimum height.

The maximum area across all these histograms (one per row) gives us our answer.

**Learn more about [Stack](/problems/stack_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

The solution uses a **monotonic [stack](/problems/stack_intro)** approach combined with dynamic histogram construction. Let's walk through the implementation step by step:

### Step 1: Build Heights Array Dynamically

For each row in the matrix, we maintain a `heights` array where `heights[j]` represents the number of consecutive '1's extending upward from the current row at column `j`.

```python
heights = [0] * len(matrix[0])  # Initialize heights for all columns
for row in matrix:
    for j, v in enumerate(row):
        if v == "1":
            heights[j] += 1  # Extend the bar upward
        else:
            heights[j] = 0   # Reset to 0 when we hit a '0'
```

### Step 2: Find Largest Rectangle in Histogram

For each row's histogram, we use the monotonic [stack](/problems/stack_intro) algorithm to find the largest rectangle. This involves finding, for each bar, the nearest smaller bars on both left and right sides.

**Finding Left Boundaries:**
```python
left = [-1] * n  # leftmost position where height >= current
stk = []
for i, h in enumerate(heights):
    while stk and heights[stk[-1]] >= h:
        stk.pop()  # Remove bars taller than current
    if stk:
        left[i] = stk[-1]  # Previous smaller bar's index
    stk.append(i)
```

The [stack](/problems/stack_intro) maintains indices of bars in increasing order of height. When we encounter a bar shorter than the stack's top, we pop until we find a smaller bar or the stack is empty.

**Finding Right Boundaries:**
```python
right = [n] * n  # rightmost position where height >= current
stk = []
for i in range(n - 1, -1, -1):  # Traverse right to left
    h = heights[i]
    while stk and heights[stk[-1]] >= h:
        stk.pop()
    if stk:
        right[i] = stk[-1]
    stk.append(i)
```

### Step 3: Calculate Maximum Area

For each bar at position `i` with height `h`, the maximum rectangle using this bar as the minimum height is:
- Width: `right[i] - left[i] - 1` (exclusive boundaries)
- Height: `h`
- Area: `h * (right[i] - left[i] - 1)`

```python
max_area = max(h * (right[i] - left[i] - 1) for i, h in enumerate(heights))
```

### Why This Works

The monotonic [stack](/problems/stack_intro) efficiently finds boundaries in `O(n)` time because:
- Each element is pushed and popped at most once
- The stack maintains a sorted order, allowing us to quickly find the nearest smaller element
- `left[i]` gives us the rightmost position to the left where height < `heights[i]`
- `right[i]` gives us the leftmost position to the right where height < `heights[i]`

### Overall Algorithm

1. For each of the `m` rows, we:
   - Update the heights array in `O(n)` time
   - Calculate the largest rectangle in the histogram in `O(n)` time
2. Return the maximum area found across all rows

**Time Complexity:** `O(m * n)` where `m` is the number of rows and `n` is the number of columns.  
**Space Complexity:** `O(n)` for the heights array and auxiliary arrays used in the histogram calculation.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach:

**Input Matrix:**
```
1 0 1 1
1 1 1 1
0 1 1 0
```

**Step 1: Process Row 0**
- Initialize heights: `[0, 0, 0, 0]`
- After processing row 0: `heights = [1, 0, 1, 1]`
- This represents the histogram for row 0:
  ```
  1   1 1
  _______
  0 1 2 3 (column indices)
  ```
- Apply largest rectangle in histogram:
  - For each bar, find left and right boundaries:
    - Bar 0 (h=1): left=-1, right=1, area = 1 × (1-(-1)-1) = 1
    - Bar 1 (h=0): skip (height is 0)
    - Bar 2 (h=1): left=1, right=4, area = 1 × (4-1-1) = 2
    - Bar 3 (h=1): left=1, right=4, area = 1 × (4-1-1) = 2
  - Maximum area for row 0: **2**

**Step 2: Process Row 1**
- Update heights based on row 1: `[2, 1, 2, 2]`
  - Column 0: '1' → heights[0] = 1 + 1 = 2
  - Column 1: '1' → heights[1] = 0 + 1 = 1
  - Column 2: '1' → heights[2] = 1 + 1 = 2
  - Column 3: '1' → heights[3] = 1 + 1 = 2
- This represents accumulated heights:
  ```
  2   2 2
    1    
  _______
  0 1 2 3
  ```
- Apply largest rectangle in histogram:
  - Bar 0 (h=2): left=-1, right=1, area = 2 × (1-(-1)-1) = 2
  - Bar 1 (h=1): left=-1, right=4, area = 1 × (4-(-1)-1) = 4
  - Bar 2 (h=2): left=1, right=4, area = 2 × (4-1-1) = 4
  - Bar 3 (h=2): left=1, right=4, area = 2 × (4-1-1) = 4
- Maximum area for row 1: **4** (bar 1 gives the 1×4 rectangle covering all of row 1; bars 2 and 3 give the 2×2 rectangle covering rows 0-1, columns 2-3)

**Step 3: Process Row 2**
- Update heights based on row 2: `[0, 2, 3, 0]`
  - Column 0: '0' → heights[0] = 0 (reset)
  - Column 1: '1' → heights[1] = 1 + 1 = 2
  - Column 2: '1' → heights[2] = 2 + 1 = 3
  - Column 3: '0' → heights[3] = 0 (reset)
- This represents:
  ```
      3
    2 2
  _______
  0 1 2 3
  ```
- Apply largest rectangle in histogram:
  - Bar 0 (h=0): area 0
  - Bar 1 (h=2): left=0, right=3, area = 2 × (3-0-1) = 4
  - Bar 2 (h=3): left=1 (bar 1 is shorter), right=3, area = 3 × (3-1-1) = 3
  - Bar 3 (h=0): area 0
- Maximum area for row 2: **4** (rows 1-2, columns 1-2)

**Final Answer:** Maximum area across all rows = **4**

Several rectangles reach this area. One of them is the 2×2 block found while processing row 1:
```
Row 0: [_, _, 1, 1]
Row 1: [_, _, 1, 1]
```
Rows 0-1, columns 2-3 contain four '1's. The full row 1 (1×4) and rows 1-2, columns 1-2 (2×2) also have area 4. No rectangle of area 5 or more fits, so the answer is 4.

## Python Solution

```python
class Solution:
    def maximalRectangle(self, matrix: List[List[str]]) -> int:
        """
        Find the maximal rectangle containing only '1's in a binary matrix.
      
        Strategy: Convert each row to a histogram problem where heights represent
        consecutive '1's from top to current row, then find the largest rectangle
        in each histogram.
      
        Args:
            matrix: 2D binary matrix with '0' and '1' characters
          
        Returns:
            Area of the largest rectangle containing only '1's
        """
        if not matrix or not matrix[0]:
            return 0
      
        # Initialize heights array for histogram representation
        num_cols = len(matrix[0])
        heights = [0] * num_cols
        max_area = 0
      
        # Process each row
        for row in matrix:
            # Update heights: increment if '1', reset to 0 if '0'
            for col_idx, value in enumerate(row):
                if value == "1":
                    heights[col_idx] += 1
                else:
                    heights[col_idx] = 0
          
            # Calculate max rectangle area for current histogram
            max_area = max(max_area, self.largestRectangleArea(heights))
      
        return max_area

    def largestRectangleArea(self, heights: List[int]) -> int:
        """
        Find the largest rectangle area in a histogram.
      
        Uses monotonic stack to find the nearest smaller element on both left and right
        for each bar, then calculates the maximum rectangle area.
      
        Args:
            heights: List of bar heights in the histogram
          
        Returns:
            Area of the largest rectangle in the histogram
        """
        n = len(heights)
        if n == 0:
            return 0
      
        # Arrays to store indices of nearest smaller elements
        left_boundaries = [-1] * n  # Index of nearest smaller element on the left
        right_boundaries = [n] * n   # Index of nearest smaller element on the right
      
        # Find left boundaries using monotonic increasing stack
        stack = []
        for i, height in enumerate(heights):
            # Pop elements >= current height
            while stack and heights[stack[-1]] >= height:
                stack.pop()
          
            # If stack not empty, top element is the nearest smaller on left
            if stack:
                left_boundaries[i] = stack[-1]
          
            stack.append(i)
      
        # Find right boundaries using monotonic increasing stack (traverse right to left)
        stack = []
        for i in range(n - 1, -1, -1):
            height = heights[i]
          
            # Pop elements >= current height
            while stack and heights[stack[-1]] >= height:
                stack.pop()
          
            # If stack not empty, top element is the nearest smaller on right
            if stack:
                right_boundaries[i] = stack[-1]
          
            stack.append(i)
      
        # Calculate maximum rectangle area
        # For each bar, the rectangle extends from (left_boundary + 1) to (right_boundary - 1)
        max_area = max(
            height * (right_boundaries[i] - left_boundaries[i] - 1) 
            for i, height in enumerate(heights)
        )
      
        return max_area
```

## Time and Space Complexity

**Time Complexity:** `O(m × n)`, where `m` is the number of rows and `n` is the number of columns in the matrix.

- The outer loop iterates through all `m` rows of the matrix
- For each row, we update the heights array in `O(n)` time
- For each row, we call `largestRectangleArea` which takes `O(n)` time:
  - Building the `left` array: `O(n)` - each element is pushed and popped from the stack at most once
  - Building the `right` array: `O(n)` - each element is pushed and popped from the stack at most once
  - Finding the maximum area: `O(n)` - single pass through the heights
- Total: `m × O(n) = O(m × n)`

**Space Complexity:** `O(n)`, where `n` is the number of columns in the matrix.

- `heights` array: `O(n)` space
- In `largestRectangleArea`:
  - `left` array: `O(n)` space
  - `right` array: `O(n)` space
  - `stk` (stack): `O(n)` space in worst case
- All these are reused for each row, so the overall space complexity remains `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Histogram Height Calculation**
A common mistake is not properly resetting the height when encountering a '0'. Some implementations might mistakenly keep the previous height or incorrectly calculate cumulative heights.

**Incorrect Implementation:**
```python
# Wrong: This doesn't reset properly
for row in matrix:
    for j, v in enumerate(row):
        heights[j] = heights[j] + 1 if v == "1" else heights[j]  # Doesn't reset!
```

**Correct Implementation:**
```python
for row in matrix:
    for j, v in enumerate(row):
        heights[j] = heights[j] + 1 if v == "1" else 0  # Properly resets to 0
```

### 2. **Off-by-One Error in Rectangle Width Calculation**
When calculating the width of the rectangle using left and right boundaries, it's easy to make an off-by-one error.

**Incorrect:**
```python
# Wrong: This includes the boundary positions
width = right[i] - left[i]
```

**Correct:**
```python
# The actual width excludes both boundaries
width = right[i] - left[i] - 1
```

### 3. **Stack Comparison Consistency**
When maintaining the monotonic stack, the comparison operator must match the boundary convention used by the algorithm.

**Potentially different convention:**
```python
# Can be valid in another stack formulation, but not with every boundary formula
while stack and heights[stack[-1]] > height:
    stack.pop()
```

**This solution's convention:**
```python
# Use >= so equal-height bars collapse to the intended boundary
while stack and heights[stack[-1]] >= height:
    stack.pop()
```

### 4. **Not Handling Edge Cases**
Failing to handle empty matrices or single-element cases can cause runtime errors.

**Problematic Code:**
```python
def maximalRectangle(self, matrix):
    # Directly accessing matrix[0] without checking
    heights = [0] * len(matrix[0])  # Crashes if matrix is empty
```

**Robust Solution:**
```python
def maximalRectangle(self, matrix):
    if not matrix or not matrix[0]:
        return 0
    heights = [0] * len(matrix[0])
```

### 5. **Mixing Data Types**
The input matrix contains strings ('0' and '1'), but calculations need integers. Forgetting to handle this type difference is a common error.

**Incorrect:**
```python
# Wrong: Comparing with integer instead of string
if matrix[i][j] == 1:  # Should be "1"
    heights[j] += 1
```

**Correct:**
```python
if matrix[i][j] == "1":  # Correctly comparing with string
    heights[j] += 1
```

### 6. **Inefficient Histogram Calculation**
Creating a new heights array for each row instead of reusing and updating it leads to unnecessary memory allocation.

**Inefficient:**
```python
for i in range(len(matrix)):
    heights = []  # Creating new array each time
    for j in range(len(matrix[0])):
        # Calculate height from scratch for each row
        h = 0
        for k in range(i, -1, -1):
            if matrix[k][j] == "1":
                h += 1
            else:
                break
        heights.append(h)
```

**Efficient:**
```python
heights = [0] * len(matrix[0])  # Reuse the same array
for row in matrix:
    for j, v in enumerate(row):
        heights[j] = heights[j] + 1 if v == "1" else 0  # Update in-place
```

### 7. **Incorrect Boundary Initialization**
Setting wrong initial values for left and right boundaries can lead to incorrect area calculations.

**Incorrect:**
```python
left = [0] * n      # Wrong: should be -1
right = [n - 1] * n  # Wrong: should be n
```

**Correct:**
```python
left = [-1] * n   # Indicates no smaller element on left
right = [n] * n   # Indicates no smaller element on right
```

These pitfalls highlight the importance of careful implementation details in this algorithm, particularly around boundary conditions, data type handling, and the monotonic stack logic.

