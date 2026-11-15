# 149. Max Points on a Line

> Source: https://algo.monster/liteproblems/149

## Problem Statement

You are given an array of points on a 2D coordinate plane, where each point is represented as `[x, y]` coordinates. Your task is to find the maximum number of points that can lie on the same straight line.

The problem asks you to determine which subset of the given points forms the largest collinear group (points that all fall on a single straight line). 

For example:
- If you have points `[[1,1], [2,2], [3,3]]`, all three points lie on the same line (y = x), so the answer would be 3
- If you have points `[[1,1], [3,2], [5,3], [4,1], [2,3], [1,4]]`, you need to check all possible lines and find which one contains the most points

The solution approach uses a triple nested loop to:
1. Pick two points to define a potential line (points at indices `i` and `j`)
2. Check all remaining points (at index `k`) to see if they lie on the same line
3. Use the cross-multiplication technique to check collinearity: three points `(x1,y1)`, `(x2,y2)`, and `(x3,y3)` are collinear if `(y2-y1) * (x3-x1) = (y3-y1) * (x2-x1)`

This avoids division and potential floating-point precision issues when checking if points have the same slope. The algorithm keeps track of the maximum count of collinear points found across all possible lines.

## Intuition

To find the maximum number of points on the same line, we need to think about what defines a line. A line is uniquely determined by any two distinct points on it. This gives us our starting point: we can enumerate all possible lines by considering every pair of points.

Once we have two points defining a line, we need to check which other points lie on this same line. The mathematical condition for three points to be collinear is that they must have the same slope between any two pairs. 

The slope between points `(x1, y1)` and `(x2, y2)` is `(y2 - y1) / (x2 - x1)`. For a third point `(x3, y3)` to be on the same line, we need:
`(y2 - y1) / (x2 - x1) = (y3 - y1) / (x3 - x1)`

However, division can cause issues with floating-point precision and vertical lines (where the denominator would be zero). To avoid these problems, we can cross-multiply the equation:
`(y2 - y1) * (x3 - x1) = (y3 - y1) * (x2 - x1)`

This gives us an elegant way to check collinearity using only integer arithmetic.

The brute force approach emerges naturally: for every possible pair of points that defines a line, count how many total points lie on that line. Keep track of the maximum count seen. Since we're already iterating through pairs with indices `i` and `j`, we only need to check points with index `k > j` to avoid redundant counting. The initial count starts at 2 (for the two points defining the line), and we increment it for each additional collinear point found.

## Solution Approach

The implementation uses a triple nested loop structure to systematically check all possible lines formed by pairs of points and count how many points lie on each line.

**Algorithm Steps:**

1. **Initialize the result**: Start with `ans = 1` since at minimum, any single point forms a "line" with itself.

2. **First loop (index i)**: Iterate through each point as the first point of a potential line.
   - Extract coordinates `(x1, y1)` from `points[i]`

3. **Second loop (index j)**: For each first point, iterate through all subsequent points to form a line.
   - Start from `j = i + 1` to avoid duplicate line checks
   - Extract coordinates `(x2, y2)` from `points[j]`
   - Initialize `cnt = 2` since we already have two points on this line

4. **Third loop (index k)**: Check all remaining points to see if they're collinear with the first two.
   - Start from `k = j + 1` to check only unprocessed points
   - Extract coordinates `(x3, y3)` from `points[k]`
   - Calculate `a = (y2 - y1) * (x3 - x1)`
   - Calculate `b = (y3 - y1) * (x2 - x1)`
   - If `a == b`, the three points are collinear, so increment `cnt`

5. **Update maximum**: After checking all points for the current line defined by points `i` and `j`, update `ans = max(ans, cnt)`

6. **Return result**: After examining all possible lines, return `ans`

**Key Implementation Details:**

- The collinearity check `(y2 - y1) * (x3 - x1) == (y3 - y1) * (x2 - x1)` avoids division and floating-point errors
- The nested loop indices `i < j < k` ensure each line is checked only once
- The counter `cnt` accumulates using `cnt += a == b`, which adds 1 when the condition is true and 0 when false
- Time complexity is `O(n³)` where n is the number of points
- Space complexity is `O(1)` as we only use a few variables

## Example Walkthrough

Let's walk through the solution with a concrete example: `points = [[1,1], [2,2], [3,3], [1,3]]`

**Initial Setup:**
- We have 4 points. Let's visualize them:
  - Point 0: (1,1)
  - Point 1: (2,2) 
  - Point 2: (3,3)
  - Point 3: (1,3)
- Initialize `ans = 1` (minimum possible answer)

**Iteration 1: i=0 (first point = [1,1])**
- j=1 (second point = [2,2]): Define line through (1,1) and (2,2)
  - `cnt = 2` (two points on the line so far)
  - k=2: Check if (3,3) is collinear
    - `a = (2-1) * (3-1) = 1 * 2 = 2`
    - `b = (3-1) * (2-1) = 2 * 1 = 2`
    - Since `a == b`, point (3,3) is on the line! `cnt = 3`
  - k=3: Check if (1,3) is collinear
    - `a = (2-1) * (1-1) = 1 * 0 = 0`
    - `b = (3-1) * (2-1) = 2 * 1 = 2`
    - Since `a ≠ b`, point (1,3) is NOT on the line. `cnt` remains 3
  - Update `ans = max(1, 3) = 3`

- j=2 (second point = [3,3]): Define line through (1,1) and (3,3)
  - `cnt = 2`
  - k=3: Check if (1,3) is collinear
    - `a = (3-1) * (1-1) = 2 * 0 = 0`
    - `b = (3-1) * (3-1) = 2 * 2 = 4`
    - Since `a ≠ b`, point (1,3) is NOT on the line. `cnt` remains 2
  - Update `ans = max(3, 2) = 3`

- j=3 (second point = [1,3]): Define line through (1,1) and (1,3)
  - `cnt = 2`
  - No more points to check (k would start at 4, which is out of bounds)
  - Update `ans = max(3, 2) = 3`

**Iteration 2: i=1 (first point = [2,2])**
- j=2 (second point = [3,3]): Define line through (2,2) and (3,3)
  - `cnt = 2`
  - k=3: Check if (1,3) is collinear
    - `a = (3-2) * (1-2) = 1 * (-1) = -1`
    - `b = (3-2) * (3-2) = 1 * 1 = 1`
    - Since `a ≠ b`, point (1,3) is NOT on the line. `cnt` remains 2
  - Update `ans = max(3, 2) = 3`

- j=3 (second point = [1,3]): Define line through (2,2) and (1,3)
  - `cnt = 2`
  - No more points to check
  - Update `ans = max(3, 2) = 3`

**Iteration 3: i=2 (first point = [3,3])**
- j=3 (second point = [1,3]): Define line through (3,3) and (1,3)
  - `cnt = 2`
  - No more points to check
  - Update `ans = max(3, 2) = 3`

**Result:** The maximum number of points on a line is **3** (points [1,1], [2,2], and [3,3] all lie on the line y = x).

## Python Solution

```python
class Solution:
    def maxPoints(self, points: List[List[int]]) -> int:
        # Get the total number of points
        num_points = len(points)
      
        # Initialize the maximum count to 1 (at least one point exists)
        max_points_on_line = 1
      
        # Iterate through each point as the first reference point
        for i in range(num_points):
            x1, y1 = points[i]
          
            # Iterate through each subsequent point as the second reference point
            for j in range(i + 1, num_points):
                x2, y2 = points[j]
              
                # Start with 2 points (the two reference points are always on the same line)
                points_on_current_line = 2
              
                # Check all remaining points to see if they're collinear with the two reference points
                for k in range(j + 1, num_points):
                    x3, y3 = points[k]
                  
                    # Check collinearity using cross product
                    # Points are collinear if (y2-y1)/(x2-x1) = (y3-y1)/(x3-x1)
                    # To avoid division, we cross-multiply: (y2-y1)*(x3-x1) = (y3-y1)*(x2-x1)
                    cross_product_left = (y2 - y1) * (x3 - x1)
                    cross_product_right = (y3 - y1) * (x2 - x1)
                  
                    # If the cross products are equal, the point is on the same line
                    if cross_product_left == cross_product_right:
                        points_on_current_line += 1
              
                # Update the maximum count if current line has more points
                max_points_on_line = max(max_points_on_line, points_on_current_line)
      
        return max_points_on_line
```

## Time and Space Complexity

**Time Complexity:** `O(n³)`

The code uses three nested loops:
- The outer loop runs `n` times (iterating through each point as the first point)
- The middle loop runs from `i+1` to `n`, which is at most `n-1` iterations
- The inner loop runs from `j+1` to `n`, which is at most `n-2` iterations

For each triple of points `(i, j, k)`, the code performs constant time operations:
- Extracting coordinates: `O(1)`
- Computing `a = (y2 - y1) * (x3 - x1)`: `O(1)`
- Computing `b = (y3 - y1) * (x2 - x1)`: `O(1)`
- Comparing `a == b`: `O(1)`

The total number of iterations is approximately `n * n * n / 6` (considering the constraints `i < j < k`), which simplifies to `O(n³)`.

**Space Complexity:** `O(1)`

The algorithm uses only a constant amount of extra space:
- Variables `n`, `ans`: `O(1)`
- Variables `x1, y1, x2, y2, x3, y3`: `O(1)`
- Variables `cnt`, `a`, `b`: `O(1)`

No additional data structures are created that scale with the input size. The space used remains constant regardless of the number of points in the input.

## Common Pitfalls

### 1. **Reusing the Code When Points Can Repeat**
LeetCode guarantees that all points are unique, and the reference solution depends on that. If the same code is reused on input with duplicate points, it breaks: when `points[i]` and `points[j]` are identical, both sides of the cross-product test are `0` for every `k`, so every remaining point is counted as being on the "line".

**Problem Example:**
```python
points = [[1,1], [1,1], [2,3], [5,0]]
# i = 0, j = 1 are the same point, so (2,3) and (5,0) are both counted -> returns 4
# The correct answer is 3 (the two [1,1] points plus either other point)
```

**Solution:**
Fix one anchor point, count exact duplicates of it separately, and group the other points by their reduced direction from the anchor:

```python
from collections import Counter
from math import gcd

def maxPoints(self, points: List[List[int]]) -> int:
    best = 0
    for i, (x1, y1) in enumerate(points):
        duplicates = 1
        slopes = Counter()
        for x2, y2 in points[i + 1:]:
            dx, dy = x2 - x1, y2 - y1
            if dx == 0 and dy == 0:
                duplicates += 1
                continue
            g = gcd(dx, dy)
            dx, dy = dx // g, dy // g
            if dx < 0 or (dx == 0 and dy < 0):
                dx, dy = -dx, -dy  # normalize the sign so opposite directions match
            slopes[(dx, dy)] += 1
        best = max(best, duplicates + max(slopes.values(), default=0))
    return best
```

This version also runs in `O(n²)` time instead of `O(n³)`, at the cost of `O(n)` extra space for the counter.

### 2. **Integer Overflow for Large Coordinates**
When calculating cross products like `(y2 - y1) * (x3 - x1)`, the multiplication of two large integers could overflow in languages with fixed integer sizes. With LeetCode's limits (coordinates between `-10^4` and `10^4`), each difference is at most `2 × 10^4` in size and each product at most `4 × 10^8`, which fits in a 32-bit `int`. Larger coordinate ranges are where the risk appears.

**Problem Example (outside LeetCode's limits):**
```python
points = [[0,0], [10**9, 10**9], [2*10**9, 2*10**9]]  # products near 4 * 10^18
```

**Solution:**
In Python, this isn't typically an issue due to arbitrary precision integers, but in other languages, you might need to:
- Use long long or BigInteger types
- Implement a GCD-based slope comparison to keep numbers smaller
- Use floating-point with careful epsilon comparison (though this introduces precision issues)

### 3. **Edge Cases with Small Input Sizes**
The reference solution already handles one or two points: with one point, the loops never run and `max_points_on_line = 1` is returned; with two points, the `j` loop sets the count to 2. An early return only makes this explicit.

**Problem Example:**
```python
points = [[1,1]]  # Single point
points = [[1,1], [2,2]]  # Two points
```

**Optional:**
Add an early return to make the small cases explicit:

```python
def maxPoints(self, points: List[List[int]]) -> int:
    n = len(points)
  
    # Handle edge cases
    if n <= 2:
        return n
  
    # Rest of the original algorithm...
    max_points_on_line = 1
    # ...
```

### 4. **Vertical and Horizontal Lines**
While the cross-multiplication technique handles vertical lines (where x2 - x1 = 0) correctly without division by zero, developers might be tempted to optimize by calculating slopes directly, which would fail for vertical lines.

**Why the current approach works:**
The formula `(y2 - y1) * (x3 - x1) == (y3 - y1) * (x2 - x1)` correctly handles:
- Vertical lines: When x2 = x1, the formula becomes `(y2 - y1) * (x3 - x1) == (y3 - y1) * 0`, which is true only if x3 = x1
- Horizontal lines: When y2 = y1, the formula becomes `0 * (x3 - x1) == (y3 - y1) * (x2 - x1)`, which is true only if y3 = y1

**Avoid this mistake:**
```python
# DON'T DO THIS - will cause division by zero for vertical lines
slope1 = (y2 - y1) / (x2 - x1)  
slope2 = (y3 - y1) / (x3 - x1)
if slope1 == slope2:  # Also has floating-point precision issues
    points_on_current_line += 1
```

