# 3027. Find the Number of Ways to Place People II

> Source: https://algo.monster/liteproblems/3027

## Problem Statement

You are given a 2D array `points` of size `n x 2` representing integer coordinates of some points on a 2D-plane, where `points[i] = [xi, yi]`.

The problem defines directions as follows:
- **Right**: positive x-axis (increasing x-coordinate)
- **Left**: negative x-axis (decreasing x-coordinate)  
- **Up**: positive y-axis (increasing y-coordinate)
- **Down**: negative y-axis (decreasing y-coordinate)

You need to place `n` people, including Alice and Bob, at these points such that there is exactly one person at every point. 

Alice wants to be alone with Bob, so she will build a rectangular fence with:
- Alice's position as the **upper left corner** 
- Bob's position as the **lower right corner**

For Alice to be at the upper left corner and Bob at the lower right corner of the fence:
- Alice's x-coordinate must be ≤ Bob's x-coordinate
- Alice's y-coordinate must be ≥ Bob's y-coordinate

The fence might not enclose any area (it can be a line). If any person other than Alice and Bob is either inside the fence or on the fence boundary, Alice will be sad.

Your task is to return the number of pairs of points where you can place Alice and Bob such that Alice does not become sad when building the fence. In other words, count the valid (Alice, Bob) position pairs where no other person falls within or on the rectangular fence formed by these two positions.

**Note**: Alice can only build a fence with her position as the upper left corner and Bob's position as the lower right corner. The positions cannot be reversed.

## Intuition

To find valid Alice-Bob pairs, we need to ensure no other person is inside or on the rectangular fence formed by their positions.

The key insight is that if we fix Alice's position and look for valid Bob positions, we need Bob to be to the right and below Alice (since Alice is upper-left, Bob is lower-right). This means for a point to be Bob's position relative to Alice at `(x1, y1)`, Bob must be at some `(x2, y2)` where `x2 >= x1` and `y2 <= y1`.

But how do we efficiently check if other people would be inside the fence? A rectangle with corners at `(x1, y1)` and `(x2, y2)` contains a point `(x, y)` if `x1 <= x <= x2` and `y2 <= y <= y1`.

The clever approach is to sort the points in a specific way: by x-coordinate ascending, and for the same x-coordinate, by y-coordinate descending. This ordering `(x, -y)` helps us process points from left to right, and within the same x-coordinate, from top to bottom.

Once sorted this way, for each point as Alice's position, we only need to check points that come after it in the sorted order (since they have `x >= x_alice`). Among these candidates for Bob's position, we need those with `y <= y_alice`.

The brilliant optimization comes from realizing that as we check potential Bob positions from left to right, we can maintain the maximum y-coordinate seen so far among valid Bob positions. If we encounter a point with y-coordinate between this maximum and Alice's y-coordinate, it forms a valid fence because:
1. It satisfies the corner requirements (`x >= x_alice`, `y <= y_alice`)
2. No previously checked points can be inside this fence (they either have smaller x-coordinates or their y-coordinates are outside the range)

This way, we avoid checking all possible pairs and all other points for each pair, reducing the complexity significantly.

**Learn more about [Math](/problems/math-basics) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows these steps:

1. **Sort the points**: We sort the points using a custom key `lambda x: (x[0], -x[1])`. This sorts by:
   - x-coordinate in ascending order (left to right)
   - For same x-coordinate, y-coordinate in descending order (top to bottom)

2. **Iterate through each point as Alice's position**: We use the outer loop to fix Alice's position at index `i` with coordinates `(_, y1)`.

3. **Initialize tracking variable**: For each Alice position, we initialize `max_y = -inf` to track the maximum y-coordinate among valid Bob positions we've seen so far.

4. **Check potential Bob positions**: We iterate through all points after Alice's position (`points[i + 1:]`) as potential Bob positions with coordinates `(_, y2)`.

5. **Validate Bob's position**: For each potential Bob position, we check if `max_y < y2 <= y1`:
   - `y2 <= y1`: Ensures Bob is below or at the same height as Alice (valid lower-right corner)
   - `y2 > max_y`: Ensures no previously considered point lies within the new fence
 
   If both conditions are met:
   - Update `max_y = y2` to track this as the new maximum y-coordinate
   - Increment `ans` counter as we found a valid Alice-Bob pair

6. **Return the result**: After checking all possible pairs, return the total count `ans`.

The algorithm works because:
- The [sorting](/problems/sorting_summary) ensures we only check points to the right of Alice (satisfying the x-coordinate constraint)
- By maintaining `max_y`, we ensure that any point between Alice and Bob either:
  - Has x-coordinate less than Alice's (outside the fence on the left), or
  - Has y-coordinate greater than Alice's or less than Bob's (outside the fence vertically)
- This guarantees no other person is inside or on the fence boundary

Time complexity: `O(n^2)` where n is the number of points, as we have nested loops checking pairs.
Space complexity: `O(1)` excluding the [sorting](/problems/sorting_summary) space.

## Example Walkthrough

Let's walk through a small example with points: `[[1,1], [2,2], [3,3]]`

**Step 1: Sort the points**
Using the key `(x, -y)`, we sort the points:
- `[1,1]` → key: `(1, -1)`
- `[2,2]` → key: `(2, -2)`  
- `[3,3]` → key: `(3, -3)`

Sorted order: `[[1,1], [2,2], [3,3]]`

**Step 2: Process each point as Alice's position**

*Alice at [1,1] (index 0):*
- Initialize `max_y = -inf`
- Check Bob at `[2,2]`: Is `-inf < 2 ≤ 1`? No (2 > 1), skip
- Check Bob at `[3,3]`: Is `-inf < 3 ≤ 1`? No (3 > 1), skip
- Found 0 valid pairs

*Alice at [2,2] (index 1):*
- Initialize `max_y = -inf`
- Check Bob at `[3,3]`: Is `-inf < 3 ≤ 2`? No (3 > 2), skip
- Found 0 valid pairs

*Alice at [3,3] (index 2):*
- No points after this one
- Found 0 valid pairs

**Total: 0 valid pairs**

This makes sense! Since the points form an ascending diagonal line, no point is both to the right AND below another point, so no valid Alice-Bob fence can be formed.

---

Let's try another example with points: `[[2,3], [3,1], [4,2]]`

**Step 1: Sort the points**
- `[2,3]` → key: `(2, -3)`
- `[3,1]` → key: `(3, -1)`
- `[4,2]` → key: `(4, -2)`

Sorted order: `[[2,3], [3,1], [4,2]]`

**Step 2: Process each point as Alice's position**

*Alice at [2,3] (index 0):*
- Initialize `max_y = -inf`
- Check Bob at `[3,1]`: Is `-inf < 1 ≤ 3`? Yes!
  - Update `max_y = 1`, increment count → 1 valid pair
  - Fence: upper-left (2,3) to lower-right (3,1), no other points inside
- Check Bob at `[4,2]`: Is `1 < 2 ≤ 3`? Yes!
  - Update `max_y = 2`, increment count → 2 valid pairs
  - Fence: upper-left (2,3) to lower-right (4,2), point [3,1] is outside (y=1 < 2)

*Alice at [3,1] (index 1):*
- Initialize `max_y = -inf`
- Check Bob at `[4,2]`: Is `-inf < 2 ≤ 1`? No (2 > 1), skip
- Found 0 valid pairs

*Alice at [4,2] (index 2):*
- No points after this one
- Found 0 valid pairs

**Total: 2 valid pairs**
- Alice at [2,3], Bob at [3,1]
- Alice at [2,3], Bob at [4,2]

The `max_y` tracking ensures that when we place Bob at [4,2], the point [3,1] doesn't violate our fence because its y-coordinate (1) is less than Bob's y-coordinate (2), placing it outside the rectangular fence.

## Python Solution

```python
from typing import List
from math import inf

class Solution:
    def numberOfPairs(self, points: List[List[int]]) -> int:
        # Sort points by x-coordinate (ascending), then by y-coordinate (descending)
        # This ensures we process points from left to right, with higher points first
        points.sort(key=lambda point: (point[0], -point[1]))
      
        # Initialize counter for valid pairs
        pair_count = 0
      
        # Iterate through each point as the potential left point of a pair
        for i, (x1, y1) in enumerate(points):
            # Track the maximum y-coordinate seen so far for valid right points
            # This helps avoid counting pairs where another point blocks the view
            max_y_seen = -inf
          
            # Check all points to the right of current point
            for x2, y2 in points[i + 1:]:
                # A valid pair must satisfy:
                # 1. y2 is not higher than y1 (y2 <= y1)
                # 2. y2 is higher than any previously seen valid point (y2 > max_y_seen)
                # This ensures no other point blocks the rectangular region
                if max_y_seen < y2 <= y1:
                    max_y_seen = y2
                    pair_count += 1
      
        return pair_count
```

## Time and Space Complexity

The time complexity of this code is `O(n^2)`, where `n` is the number of points. The sorting operation takes `O(n log n)` time, and the nested loops iterate through all pairs of points with `i < j`, resulting in `O(n^2)` comparisons. Since `O(n^2)` dominates `O(n log n)`, the overall time complexity is `O(n^2)`.

The space complexity is `O(1)` if we don't count the space used by the sorting algorithm (which might use `O(log n)` for in-place sorting). The algorithm only uses a constant amount of extra space for variables `ans`, `max_y`, `i`, `y1`, and `y2`, regardless of the input size.

Note: The reference answer stating `O(1)` for time complexity appears to be incorrect, as the algorithm clearly has nested loops that depend on the input size.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Incorrect Sorting Logic
**Pitfall**: Using the wrong sorting order, particularly sorting y-coordinates in ascending order instead of descending order.

```python
# WRONG - sorts y in ascending order
points.sort(key=lambda point: (point[0], point[1]))
```

This breaks the algorithm because when we process points with the same x-coordinate, we need to consider higher points first to correctly maintain the `max_y_seen` invariant.

**Solution**: Always sort with `(x, -y)` to ensure proper ordering:
```python
# CORRECT - sorts x ascending, y descending
points.sort(key=lambda point: (point[0], -point[1]))
```

### 2. Misunderstanding the Fence Constraints
**Pitfall**: Incorrectly assuming Alice must be strictly above and to the left of Bob, using strict inequalities:

```python
# WRONG - uses strict inequality
if max_y_seen < y2 < y1:  # Missing edge case where y2 == y1
    max_y_seen = y2
    pair_count += 1
```

This misses valid cases where Alice and Bob can be at the same height (forming a horizontal line fence).

**Solution**: Use `<=` for the y-coordinate comparison:
```python
# CORRECT - allows Alice and Bob at same height
if max_y_seen < y2 <= y1:
    max_y_seen = y2
    pair_count += 1
```

### 3. Not Updating max_y_seen for Invalid Pairs
**Pitfall**: Only updating `max_y_seen` when finding a valid pair:

```python
# WRONG - can miss blocking points
for x2, y2 in points[i + 1:]:
    if max_y_seen < y2 <= y1:
        max_y_seen = y2
        pair_count += 1
    # Should we update max_y_seen for points with y2 > y1?
```

While the current solution is correct (we only need to track max_y for valid Bob positions), some might mistakenly think they need to track ALL points, which would lead to:

```python
# UNNECESSARY - tracking all points
for x2, y2 in points[i + 1:]:
    if y2 <= y1 and y2 > max_y_seen:
        pair_count += 1
    max_y_seen = max(max_y_seen, y2)  # This would be wrong!
```

**Solution**: Only update `max_y_seen` for valid Bob positions, as points above Alice are already outside the fence.

### 4. Forgetting Edge Cases
**Pitfall**: Not considering that the fence can be a single point or a line:

- When Alice and Bob are at the same position (if allowed)
- When the fence forms a vertical or horizontal line

**Solution**: The condition `max_y_seen < y2 <= y1` correctly handles these cases:
- Horizontal line: when `y2 == y1`
- Vertical line: when points have the same x-coordinate (handled by sorting)

