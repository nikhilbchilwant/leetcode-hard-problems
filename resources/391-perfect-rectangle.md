# 391. Perfect Rectangle

> Source: https://algo.monster/liteproblems/391

## Problem Statement

You are given an array of rectangles where each rectangle is represented as `rectangles[i] = [x_i, y_i, a_i, b_i]`. Each rectangle is axis-aligned (parallel to the x and y axes) with:
- `(x_i, y_i)` as the bottom-left corner
- `(a_i, b_i)` as the top-right corner

The task is to determine if all these rectangles together form an **exact cover** of a rectangular region. This means:
1. The rectangles must combine to form a single larger rectangle with no gaps
2. There should be no overlapping areas between rectangles
3. Every point inside the larger rectangle should be covered exactly once

The solution uses two key observations to verify if rectangles form an exact cover:

**Area Check**: The sum of all individual rectangle areas must equal the area of the bounding rectangle (formed by the minimum and maximum coordinates).

**Corner Counting**: 
- Each corner point of a rectangle is counted based on how many rectangles share that corner
- For the four corners of the overall bounding rectangle, each should appear exactly once (count = 1)
- For all internal corners (where rectangles meet), each point should appear an even number of times (2 or 4), since internal corners are shared by adjacent rectangles

The algorithm:
1. Calculates the total area and finds the bounding rectangle coordinates `(minX, minY)` to `(maxX, maxY)`
2. Counts how many times each corner point appears across all rectangles
3. Verifies that the total area matches the bounding rectangle area
4. Checks that the four outer corners appear exactly once
5. Confirms all internal corners appear 2 or 4 times (valid sharing between rectangles)

If all conditions are met, the rectangles form an exact cover; otherwise, they don't.

## Intuition

To understand why this solution works, let's think about what happens when rectangles perfectly tile a region.

First, consider the area constraint. If rectangles form an exact cover without gaps or overlaps, the sum of their individual areas must equal the total area of the region they cover. This is straightforward - any gap would mean less area covered, and any overlap would mean counting some area twice.

The clever insight comes from observing corner points. Imagine placing rectangles on a grid and looking at where their corners meet:

1. **Outer corners of the bounding rectangle**: These points can only belong to one rectangle. If a corner of the overall region appeared in multiple rectangles, it would mean rectangles are extending beyond the boundary or overlapping at that corner.

2. **Internal points where rectangles meet**: When rectangles fit together perfectly like puzzle pieces, their corners meet in specific patterns:
   - When two rectangles share an edge, they meet at a point that appears exactly 2 times (once for each rectangle)
   - When four rectangles meet at a cross-junction (like a '+' sign), that point appears exactly 4 times

Why can't an internal point appear an odd number of times? Picture trying to fit rectangles together where a point appears 3 times - you'd either have a gap (incomplete coverage) or an overlap (double coverage). The geometry simply doesn't allow for perfect tiling with odd corner counts at internal points.

This corner-counting technique effectively detects both overlaps and gaps:
- Overlapping rectangles would create internal points with counts like 3, 5, or higher odd numbers
- Gaps would leave some corners appearing only once internally or create mismatched corner counts

By combining the area check (ensures no gaps or overlaps in total) with the corner pattern check (ensures proper local fitting), we can definitively determine if the rectangles form a perfect cover.

## Solution Approach

The implementation follows a systematic approach to verify both the area constraint and corner point patterns:

**1. Initialize tracking variables:**
- `area`: Accumulates the total area of all rectangles
- `minX, minY, maxX, maxY`: Track the bounding rectangle coordinates
- `cnt`: A dictionary to count occurrences of each corner point

**2. Process each rectangle:**
For each rectangle `[x, y, a, b]` in the input:
- Add its area `(a - x) * (b - y)` to the total area
- Update the bounding box by comparing with current min/max values:
  - `minX = min(minX, x)`, `minY = min(minY, y)`
  - `maxX = max(maxX, a)`, `maxY = max(maxY, b)`
- Count all four corners of this rectangle in the dictionary:
  - Bottom-left: `(x, y)`
  - Top-left: `(x, b)`
  - Top-right: `(a, b)`
  - Bottom-right: `(a, y)`

**3. Verify the area condition:**
Check if the sum of individual areas equals the bounding rectangle area:
```python
area == (maxX - minX) * (maxY - minY)
```

**4. Verify the four outer corners:**
The four corners of the bounding rectangle must each appear exactly once:
- `cnt[(minX, minY)] == 1` (bottom-left)
- `cnt[(minX, maxY)] == 1` (top-left)
- `cnt[(maxX, maxY)] == 1` (top-right)
- `cnt[(maxX, minY)] == 1` (bottom-right)

If any of these conditions fail, return `False`.

**5. Check internal corners:**
After removing the four outer corners from the dictionary:
```python
del cnt[(minX, minY)], cnt[(minX, maxY)], cnt[(maxX, maxY)], cnt[(maxX, minY)]
```

All remaining points (internal corners) must have counts of either 2 or 4:
```python
all(c == 2 or c == 4 for c in cnt.values())
```

The algorithm runs in `O(n)` time where `n` is the number of rectangles, as we iterate through the list once and perform constant-time operations for each rectangle. The space complexity is also `O(n)` for storing the corner points in the dictionary.

## Example Walkthrough

Let's walk through a small example where we have 3 rectangles that form an exact cover:
- Rectangle 1: `[1, 1, 3, 3]` (bottom-left at (1,1), top-right at (3,3))
- Rectangle 2: `[3, 1, 4, 2]` (bottom-left at (3,1), top-right at (4,2))  
- Rectangle 3: `[3, 2, 4, 3]` (bottom-left at (3,2), top-right at (4,3))

**Step 1: Process each rectangle and track information**

For Rectangle 1 `[1, 1, 3, 3]`:
- Area = (3-1) × (3-1) = 4
- Update bounds: minX=1, minY=1, maxX=3, maxY=3
- Count corners: (1,1)→1, (1,3)→1, (3,3)→1, (3,1)→1

For Rectangle 2 `[3, 1, 4, 2]`:
- Area total = 4 + (4-3) × (2-1) = 4 + 1 = 5
- Update bounds: minX=1, minY=1, maxX=4, maxY=3
- Count corners: (3,1)→2, (3,2)→1, (4,2)→1, (4,1)→1

For Rectangle 3 `[3, 2, 4, 3]`:
- Area total = 5 + (4-3) × (3-2) = 5 + 1 = 6
- Update bounds: minX=1, minY=1, maxX=4, maxY=3
- Count corners: (3,2)→2, (3,3)→2, (4,3)→1, (4,2)→2

**Step 2: Verify area condition**
- Total area of rectangles: 6
- Bounding rectangle area: (4-1) × (3-1) = 3 × 2 = 6 ✓

**Step 3: Check corner counts**
Our corner count dictionary contains:
- (1,1)→1, (1,3)→1, (4,3)→1, (4,1)→1 (outer corners)
- (3,1)→2, (3,2)→2, (3,3)→2, (4,2)→2 (internal corners)

**Step 4: Verify outer corners**
The four corners of the bounding rectangle all appear exactly once:
- (1,1): count = 1 ✓
- (1,3): count = 1 ✓
- (4,3): count = 1 ✓
- (4,1): count = 1 ✓

**Step 5: Verify internal corners**
After removing outer corners, remaining corners all have count 2:
- (3,1): count = 2 ✓ (where Rectangle 1 and 2 meet)
- (3,2): count = 2 ✓ (where Rectangle 2 and 3 meet)
- (3,3): count = 2 ✓ (where Rectangle 1 and 3 meet)
- (4,2): count = 2 ✓ (where Rectangle 2 and 3 meet)

All conditions are satisfied, so these rectangles form an exact cover!

Visualized:
```
3 |[1]|[3]|
2 |[1]|[2]|
1 |[1]|[2]|
  +---------
    1 2 3 4
```
Where [1], [2], [3] represent the three rectangles perfectly tiling the region.

## Python Solution

```python
class Solution:
    def isRectangleCover(self, rectangles: List[List[int]]) -> bool:
        """
        Determines if given rectangles form an exact cover of a rectangular region.
      
        The algorithm checks two conditions:
        1. Total area of all rectangles equals the area of the bounding rectangle
        2. Corner points appear with correct frequencies:
           - The 4 corners of the bounding rectangle appear exactly once
           - All other corners appear 2 or 4 times (shared by adjacent rectangles)
      
        Args:
            rectangles: List of rectangles, each as [x1, y1, x2, y2]
      
        Returns:
            True if rectangles form an exact cover, False otherwise
        """
        from collections import defaultdict
      
        # Initialize variables to track total area and bounding box
        total_area = 0
        min_x, min_y = rectangles[0][0], rectangles[0][1]
        max_x, max_y = rectangles[0][2], rectangles[0][3]
      
        # Dictionary to count occurrences of each corner point
        corner_count = defaultdict(int)
      
        # Process each rectangle
        for rectangle in rectangles:
            x1, y1, x2, y2 = rectangle
          
            # Add rectangle's area to total
            total_area += (x2 - x1) * (y2 - y1)
          
            # Update bounding box coordinates
            min_x = min(min_x, x1)
            min_y = min(min_y, y1)
            max_x = max(max_x, x2)
            max_y = max(max_y, y2)
          
            # Count occurrences of each corner of current rectangle
            corner_count[(x1, y1)] += 1  # Bottom-left corner
            corner_count[(x1, y2)] += 1  # Top-left corner
            corner_count[(x2, y2)] += 1  # Top-right corner
            corner_count[(x2, y1)] += 1  # Bottom-right corner
      
        # Check if total area matches bounding box area
        # and if the four corners of bounding box appear exactly once
        if (total_area != (max_x - min_x) * (max_y - min_y) or
            corner_count[(min_x, min_y)] != 1 or
            corner_count[(min_x, max_y)] != 1 or
            corner_count[(max_x, max_y)] != 1 or
            corner_count[(max_x, min_y)] != 1):
            return False
      
        # Remove the four outer corners from consideration
        del corner_count[(min_x, min_y)]
        del corner_count[(min_x, max_y)]
        del corner_count[(max_x, max_y)]
        del corner_count[(max_x, min_y)]
      
        # Check that all internal corners appear 2 or 4 times
        # 2 times: corner shared by 2 rectangles (edge point)
        # 4 times: corner shared by 4 rectangles (internal point)
        return all(count == 2 or count == 4 for count in corner_count.values())
```

## Time and Space Complexity

**Time Complexity:** `O(n)` where `n` is the number of rectangles in the input list.

- The algorithm iterates through all rectangles once in the main loop: `O(n)`
- For each rectangle, it performs constant time operations:
  - Area calculation: `O(1)`
  - Min/max updates: `O(1)` 
  - Adding 4 corner points to the hashmap: `O(1)` for each insertion
- After the loop, checking the 4 outer corners: `O(1)`
- Deleting 4 corners from the hashmap: `O(1)`
- The final validation iterates through all remaining points in the hashmap. In the worst case, there are `4n` corner points total, minus the 4 outer corners, giving us `O(n)` points to check
- Overall: `O(n) + O(n) = O(n)`

**Space Complexity:** `O(n)`

- The `cnt` dictionary stores corner points. Each rectangle contributes 4 corners, but corners can be shared between adjacent rectangles
- In the worst case (no overlapping corners except at boundaries), we have approximately `4n` unique corner points
- Therefore, the space complexity is `O(n)` for storing the corner point frequencies in the hashmap

## Common Pitfalls

### 1. **Integer Overflow in Area Calculation**
When calculating areas with large coordinate values, the multiplication `(x2 - x1) * (y2 - y1)` can cause integer overflow in languages with fixed integer sizes. While Python handles arbitrary precision integers automatically, this is a critical issue in languages like Java or C++.

**Solution:** Use appropriate data types (e.g., `long` in Java) or check for potential overflow before multiplication:
```python
# For other languages, ensure using appropriate data type
# Java: use long for area calculations
# C++: use long long or check for overflow
```

### 2. **Incorrect Corner Identification for Overlapping Rectangles**
A subtle pitfall occurs when trying to detect overlaps through corner counting alone. If two rectangles overlap perfectly (one completely inside another), the corner counting method might not detect this as an invalid case if the counts happen to work out.

**Example of edge case:**
```python
# These rectangles overlap: two small squares sit inside the larger square
rectangles = [[0,0,2,2], [0,0,1,1], [1,1,2,2]]
```

**Solution:** The algorithm rejects this through both checks. The total area is `4 + 1 + 1 = 6`, larger than the bounding area of 4, because overlapping regions are counted twice. The corner `(0,0)` also appears twice instead of once. The two checks are needed together: the area check catches overlaps that happen to leave the corner counts valid, and the corner check catches a gap and an overlap of the same size, which leave the total area unchanged.

### 3. **Floating Point Precision Issues**
If the problem involved floating-point coordinates instead of integers, comparing areas with `==` would be problematic due to floating-point precision errors.

**Solution:** For floating-point comparisons, use an epsilon value:
```python
EPSILON = 1e-9
if abs(total_area - (max_x - min_x) * (max_y - min_y)) > EPSILON:
    return False
```

### 4. **Not Handling Edge Case of Single Rectangle**
The code might seem to have issues with a single rectangle input, but it actually handles it correctly. A single rectangle has four corners that each appear once, which satisfies both the outer corner condition and leaves no internal corners to check.

**Solution:** The current implementation handles this correctly, but it's worth adding a comment or explicit check for clarity:
```python
# Single rectangle is always a valid exact cover of itself
if len(rectangles) == 1:
    return True
```

### 5. **Memory Issues with Large Coordinate Spaces**
If coordinates can be very large but sparse, storing them as tuples in a dictionary is memory-efficient. However, attempting to use a 2D array for counting would cause memory issues.

**Solution:** The current approach using a dictionary with tuple keys is optimal for sparse coordinate spaces. Avoid array-based solutions unless coordinates are guaranteed to be small and dense.

