# 302. Smallest Rectangle Enclosing Black Pixels

> Source: https://algo.monster/liteproblems/302

## Problem Statement

You have a binary matrix `image` of size `m x n` where each cell contains either `'0'` (white pixel) or `'1'` (black pixel).

The key properties of this matrix are:
- All black pixels (`'1'`) form a single connected region
- Black pixels are connected horizontally and vertically (not diagonally)
- You're given coordinates `(x, y)` of one black pixel in this region

Your task is to find the area of the smallest axis-aligned rectangle that can enclose all the black pixels in the matrix.

For example, if you have black pixels scattered in various positions, you need to find the tightest bounding box that contains all of them. The area would be calculated as `(height × width)` of this bounding rectangle.

**Important constraint**: Your algorithm must run in less than `O(mn)` time complexity, meaning you cannot simply scan through every cell in the matrix.

The solution uses binary search to efficiently find the boundaries of the black pixel region:
- Finds the topmost row containing a black pixel (upper boundary `u`)
- Finds the bottommost row containing a black pixel (lower boundary `d`)
- Finds the leftmost column containing a black pixel (left boundary `l`)
- Finds the rightmost column containing a black pixel (right boundary `r`)

The final area is calculated as `(d - u + 1) * (r - l + 1)`, representing the height times the width of the bounding rectangle.

## Intuition

The key insight comes from understanding what we're really looking for and the properties of the black pixel region.

Since all black pixels form a single connected region, and we need the smallest rectangle that encloses them all, we're essentially finding the **extreme boundaries** of this region - the topmost, bottommost, leftmost, and rightmost black pixels.

The naive approach would be to perform DFS/BFS from the given black pixel `(x, y)` to explore all connected black pixels and track the min/max coordinates. However, this would take `O(mn)` time in the worst case, violating our complexity requirement.

Here's the crucial observation: **If we know there's at least one black pixel in a row or column, we can use binary search to find the boundary rows and columns.**

Think about finding the topmost row with a black pixel:
- We know row `x` has a black pixel (given)
- We know rows above some boundary have no black pixels
- We know rows at or below the boundary have at least one black pixel
- This creates a binary property we can search on!

For each boundary:
1. **Upper boundary**: Binary search rows `[0, x]` to find the smallest row index that contains a black pixel
2. **Lower boundary**: Binary search rows `[x, m-1]` to find the largest row index that contains a black pixel
3. **Left boundary**: Binary search columns `[0, y]` to find the smallest column index that contains a black pixel
4. **Right boundary**: Binary search columns `[y, n-1]` to find the largest column index that contains a black pixel

To check if a row has a black pixel, we scan through all columns in that row (`O(n)` time). Similarly, checking a column takes `O(m)` time.

This gives us a total complexity of `O(m log n + n log m)`:
- Finding horizontal boundaries: 2 binary searches × `O(log m)` iterations × `O(n)` per check
- Finding vertical boundaries: 2 binary searches × `O(log n)` iterations × `O(m)` per check

The area of the bounding rectangle is then simply `(bottom - top + 1) × (right - left + 1)`.

## Solution Approach

The implementation uses binary search to find the four boundaries of the black pixel region. We apply the standard binary search template consistently for all four searches.

**The Binary Search Template:**
```python
left, right = min_val, max_val
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

return first_true_index
```

**Finding the Upper Boundary (topmost row with black pixel):**

We search for the first row that contains a black pixel. The feasibility pattern is:
- `feasible(row) = True` if row contains at least one black pixel
- Pattern: `[F, F, ..., F, T, T, ..., T]` where T starts at the topmost row with a black pixel

```python
left, right = 0, x
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if has_black_pixel_in_row(mid):
        first_true_index = mid
        right = mid - 1  # Look for earlier row
    else:
        left = mid + 1
top_boundary = first_true_index
```

**Finding the Lower Boundary (bottommost row with black pixel):**

We search for the first row (from bottom) that does NOT contain a black pixel, then return the row before it. Alternatively, we redefine feasible:
- `feasible(row) = True` if row does NOT contain any black pixel
- We search in range `[x+1, m]` for the first empty row, then lower_boundary = first_true_index - 1

```python
left, right = x + 1, rows
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if mid >= rows or not has_black_pixel_in_row(mid):  # Row past the end, or first row with no black pixels
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
bottom_boundary = first_true_index - 1 if first_true_index != -1 else rows - 1
```

**Finding the Left Boundary (leftmost column with black pixel):**

Same pattern as upper boundary, but searching columns:
- `feasible(col) = True` if column contains at least one black pixel

```python
left, right = 0, y
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if has_black_pixel_in_col(mid):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
left_boundary = first_true_index
```

**Finding the Right Boundary (rightmost column with black pixel):**

Search for first column with no black pixels, starting from y+1:
```python
left, right = y + 1, cols
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if mid >= cols or not has_black_pixel_in_col(mid):  # Column past the end counts as empty
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
right_boundary = first_true_index - 1 if first_true_index != -1 else cols - 1
```

**Calculating the Area:**
```python
return (bottom_boundary - top_boundary + 1) * (right_boundary - left_boundary + 1)
```
- Height: `bottom_boundary - top_boundary + 1` (inclusive of both boundaries)
- Width: `right_boundary - left_boundary + 1` (inclusive of both boundaries)

## Example Walkthrough

Let's walk through a concrete example using the binary search template to find the bounding rectangle.

Consider this 5×7 matrix with a connected black pixel region:
```
    0 1 2 3 4 5 6
0   0 0 0 0 0 0 0
1   0 0 1 1 1 0 0
2   0 1 1 1 0 0 0
3   0 0 1 0 0 0 0
4   0 0 0 0 0 0 0
```

Given: Starting black pixel at position `(x=2, y=3)` (row 2, column 3). Matrix has 5 rows (m=5) and 7 columns (n=7).

**Step 1: Find Upper Boundary (topmost row with black pixel)**
- Feasible condition: row contains a black pixel
- Search range: `[0, 2]`, `first_true_index = -1`
- Iteration 1: `left=0, right=2`, `mid = 1`
  - Row 1 has black pixel → feasible!
  - `first_true_index = 1`, `right = 0`
- Iteration 2: `left=0, right=0`, `mid = 0`
  - Row 0 has no black pixel → not feasible
  - `left = 1`
- Loop ends (`left > right`): `first_true_index = 1`
- Result: `top_boundary = 1`

**Step 2: Find Lower Boundary (bottommost row with black pixel)**
- Feasible condition: row does NOT contain a black pixel (inverted)
- Search range: `[3, 5]` (from x+1 to rows), `first_true_index = -1`
- Iteration 1: `left=3, right=5`, `mid = 4`
  - Row 4 has no black pixel → feasible!
  - `first_true_index = 4`, `right = 3`
- Iteration 2: `left=3, right=3`, `mid = 3`
  - Row 3 has black pixel → not feasible
  - `left = 4`
- Loop ends: `first_true_index = 4`
- Result: `bottom_boundary = 4 - 1 = 3`

**Step 3: Find Left Boundary (leftmost column with black pixel)**
- Feasible condition: column contains a black pixel
- Search range: `[0, 3]`, `first_true_index = -1`
- Iteration 1: `left=0, right=3`, `mid = 1`
  - Column 1 has black pixel → feasible!
  - `first_true_index = 1`, `right = 0`
- Iteration 2: `left=0, right=0`, `mid = 0`
  - Column 0 has no black pixel → not feasible
  - `left = 1`
- Loop ends: `first_true_index = 1`
- Result: `left_boundary = 1`

**Step 4: Find Right Boundary (rightmost column with black pixel)**
- Feasible condition: column does NOT contain a black pixel (inverted)
- Search range: `[4, 7]` (from y+1 to cols), `first_true_index = -1`
- Iteration 1: `left=4, right=7`, `mid = 5`
  - Column 5 has no black pixel → feasible!
  - `first_true_index = 5`, `right = 4`
- Iteration 2: `left=4, right=4`, `mid = 4`
  - Column 4 has black pixel → not feasible
  - `left = 5`
- Loop ends: `first_true_index = 5`
- Result: `right_boundary = 5 - 1 = 4`

**Step 5: Calculate Area**
- Height = `bottom_boundary - top_boundary + 1 = 3 - 1 + 1 = 3`
- Width = `right_boundary - left_boundary + 1 = 4 - 1 + 1 = 4`
- Area = `3 × 4 = 12`

The bounding rectangle encompasses rows [1,3] and columns [1,4]:
```
    0 1 2 3 4 5 6
0   0 0 0 0 0 0 0
1   0 [0 1 1 1] 0 0
2   0 [1 1 1 0] 0 0
3   0 [0 1 0 0] 0 0
4   0 0 0 0 0 0 0
```

**Template Compliance:**
- ✅ Uses `first_true_index = -1` to track answer
- ✅ Uses `while left <= right`
- ✅ Uses `right = mid - 1` when feasible
- ✅ Clearly defined feasible conditions for each boundary

## Python Solution

```python
class Solution:
    def minArea(self, image: List[List[str]], x: int, y: int) -> int:
        rows, cols = len(image), len(image[0])

        def has_black_in_row(row: int) -> bool:
            """Check if a row contains any black pixel."""
            for c in range(cols):
                if image[row][c] == '1':
                    return True
            return False

        def has_black_in_col(col: int) -> bool:
            """Check if a column contains any black pixel."""
            for r in range(rows):
                if image[r][col] == '1':
                    return True
            return False

        # Find topmost row with black pixel (first row where has_black is True)
        left, right = 0, x
        first_true_index = -1
        while left <= right:
            mid = (left + right) // 2
            if has_black_in_row(mid):
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1
        top_boundary = first_true_index

        # Find bottommost row with black pixel
        # Search for first row WITHOUT black pixel, then subtract 1
        left, right = x + 1, rows
        first_true_index = -1
        while left <= right:
            mid = (left + right) // 2
            if mid >= rows or not has_black_in_row(mid):
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1
        bottom_boundary = first_true_index - 1 if first_true_index != -1 else rows - 1

        # Find leftmost column with black pixel (first col where has_black is True)
        left, right = 0, y
        first_true_index = -1
        while left <= right:
            mid = (left + right) // 2
            if has_black_in_col(mid):
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1
        left_boundary = first_true_index

        # Find rightmost column with black pixel
        # Search for first column WITHOUT black pixel, then subtract 1
        left, right = y + 1, cols
        first_true_index = -1
        while left <= right:
            mid = (left + right) // 2
            if mid >= cols or not has_black_in_col(mid):
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1
        right_boundary = first_true_index - 1 if first_true_index != -1 else cols - 1

        # Calculate the area of the bounding rectangle
        height = bottom_boundary - top_boundary + 1
        width = right_boundary - left_boundary + 1
        return height * width
```

## Time and Space Complexity

**Time Complexity:** `O(m * log n + n * log m)` where `m` is the number of rows and `n` is the number of columns in the image.

The algorithm performs 4 binary searches:
- Two binary searches on rows (finding upper and lower boundaries): Each binary search takes `O(log m)` iterations, and in each iteration, we scan through all `n` columns to check if there's a '1' in that row, taking `O(n)` time. Total: `2 * O(log m * n) = O(n * log m)`
- Two binary searches on columns (finding left and right boundaries): Each binary search takes `O(log n)` iterations, and in each iteration, we scan through all `m` rows to check if there's a '1' in that column, taking `O(m)` time. Total: `2 * O(log n * m) = O(m * log n)`

Combined time complexity: `O(m * log n + n * log m)`

**Space Complexity:** `O(1)`

The algorithm only uses a constant amount of extra space for variables (`left`, `right`, `mid`, `u`, `d`, `l`, `r`, `c`) regardless of the input size. No additional data structures are created that scale with the input dimensions.

## Common Pitfalls

### 1. Using the Wrong Binary Search Template Variant

**The Pitfall:**
Using `while left < right` with `right = mid` is a different binary search variant. It works when its own bounds and final check are used, but mixing its pieces with the template below (for example `while left <= right` with `right = mid`) causes infinite loops or skipped answers. Keeping one variant for all four searches avoids that.

**The Other Variant:**
```python
# Valid on its own, but don't mix it with the template's update rules
while left < right:
    mid = (left + right) // 2
    if condition:
        right = mid
    else:
        left = mid + 1
return left
```

**The Solution:**
Always use the standard template with `first_true_index`:
```python
# CORRECT - use this template
left, right = 0, max_val
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
return first_true_index
```

### 2. Off-by-One Error in Area Calculation

**The Pitfall:**
Calculating the area as `(bottom - top) * (right - left)` instead of `(bottom - top + 1) * (right - left + 1)`. This mistake comes from forgetting that both boundary positions are inclusive.

**Example:**
If top = 2, bottom = 4, left = 1, right = 3:
- Wrong: `(4 - 2) * (3 - 1) = 2 * 2 = 4`
- Correct: `(4 - 2 + 1) * (3 - 1 + 1) = 3 * 3 = 9`

### 3. Forgetting to Handle the first_true_index == -1 Case

**The Pitfall:**
When using the binary search template, if no feasible value exists in the search range, `first_true_index` remains -1. Forgetting to handle this case can cause incorrect results or array out-of-bounds errors.

**The Solution:**
Always check if `first_true_index` is -1 before using it:
```python
if first_true_index == -1:
    # Handle the "not found" case appropriately
    return default_value
```

For this problem, we know a black pixel exists (given in the input), so the minimum boundaries will always find a valid result. For maximum boundaries, we search for the first empty row/column and subtract 1.

### 4. Not Using the Known Black Pixel Position

**The Pitfall:**
Setting initial search ranges as `[0, rows-1]` and `[0, cols-1]` for all boundaries, which wastes the given information that position `(x, y)` contains a black pixel.

**The Solution:**
Use the known position to optimize search ranges:
- Top boundary: search in `[0, x]` (answer must be ≤ x)
- Bottom boundary: search in `[x, rows-1]` (answer must be ≥ x)
- Left boundary: search in `[0, y]` (answer must be ≤ y)
- Right boundary: search in `[y, cols-1]` (answer must be ≥ y)

This reduces the search space and improves efficiency.

