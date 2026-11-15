# 587. Erect the Fence

> Source: https://algo.monster/liteproblems/587

## Problem Statement

You have a garden with trees planted at various coordinates. Each tree's position is given as `[x, y]` coordinates in a 2D plane. Your task is to build a fence around all the trees using the minimum amount of rope possible.

The fence must enclose **all** trees in the garden - no tree can be left outside the fence. Since you want to use the minimum length of rope, the fence will form a convex hull around all the trees.

You need to return the coordinates of all trees that are located exactly on the fence perimeter. These are the trees that the fence rope will touch or pass through. Trees that are inside the fence but not on its boundary should not be included in the output.

For example, if you have trees forming a triangle with one tree in the middle, only the three trees forming the triangle would be on the fence perimeter, while the middle tree would be enclosed but not part of the answer.

The answer can be returned in any order.

**Key Points:**
- Input: An array `trees` where each element `trees[i] = [xi, yi]` represents a tree's coordinates
- Output: Coordinates of all trees that lie on the fence perimeter
- The fence forms the smallest perimeter that encloses all trees (convex hull)
- If there are fewer than 4 trees, all trees will be on the perimeter

## Intuition

The problem is asking us to find the convex hull of a set of points. Think of it like wrapping a rubber band around all the trees - the rubber band will naturally form the smallest perimeter that encloses all points, and only certain trees will actually touch the rubber band.

To find which trees are on this boundary, we can use a technique called the "monotone chain algorithm" or "Andrew's algorithm". The key insight is that we can build the convex hull in two parts: the lower hull and the upper hull.

Imagine walking along the fence from the leftmost point to the rightmost point. As we walk, we should always be making either left turns or going straight - never right turns. If we ever need to make a right turn to include a new point, it means our previous point shouldn't be on the hull.

We can detect whether three points form a left turn or right turn using the cross product. For three points `A`, `B`, and `C`, the cross product `(B - A) × (C - B)` tells us the turn direction:
- Positive value: left turn (counterclockwise)
- Negative value: right turn (clockwise)
- Zero: collinear (straight line)

The algorithm works by:
1. First sorting all points by x-coordinate (and y-coordinate as tiebreaker)
2. Building the lower hull by going from left to right, removing points that would cause right turns
3. Building the upper hull by going from right to left, again removing points that would cause right turns
4. The combination of lower and upper hulls gives us the complete convex hull

The stack-based approach allows us to efficiently backtrack when we discover that a previously added point shouldn't be on the hull. We keep popping points from the stack until adding the new point would no longer create a right turn.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation uses the Monotone Chain Algorithm (Andrew's Algorithm) to find the convex hull. Let's walk through the code step by step:

**1. Cross Product Function:**
```python
def cross(i, j, k):
    a, b, c = trees[i], trees[j], trees[k]
    return (b[0] - a[0]) * (c[1] - b[1]) - (b[1] - a[1]) * (c[0] - b[0])
```
This calculates the cross product of vectors `AB` and `BC` where `A = trees[i]`, `B = trees[j]`, `C = trees[k]`. A negative value indicates a right turn (clockwise), which we want to avoid on the convex hull.

**2. Base Case:**
```python
if n < 4:
    return trees
```
With fewer than 4 points, all points must be on the convex hull perimeter.

**3. Sort the Points:**
```python
trees.sort()
```
Points are sorted lexicographically (first by x-coordinate, then by y-coordinate). This ensures we start from the leftmost point.

**4. Build Lower Hull:**
```python
vis = [False] * n
stk = [0]
for i in range(1, n):
    while len(stk) > 1 and cross(stk[-2], stk[-1], i) < 0:
        vis[stk.pop()] = False
    vis[i] = True
    stk.append(i)
```
- We use a stack `stk` to maintain the current hull points
- The `vis` array tracks which points are currently on the hull
- For each new point `i`, we check if adding it would create a right turn
- If yes, we pop the previous point from the stack (it's not on the hull)
- We continue popping until adding point `i` would create a left turn or straight line
- Then we add point `i` to the stack

**5. Build Upper Hull:**
```python
m = len(stk)
for i in range(n - 2, -1, -1):
    if vis[i]:
        continue
    while len(stk) > m and cross(stk[-2], stk[-1], i) < 0:
        stk.pop()
    stk.append(i)
```
- `m` stores the size of the lower hull to prevent removing those points
- We traverse points from right to left (indices `n-2` to `0`)
- Skip points already in the lower hull (`vis[i] == True`)
- Apply the same logic: remove points that would cause right turns
- The condition `len(stk) > m` ensures we don't remove lower hull points

**6. Final Cleanup:**
```python
stk.pop()
return [trees[i] for i in stk]
```
- The last point is removed because it's the same as the first point (we've completed the circuit)
- Convert stack indices to actual tree coordinates

The algorithm runs in `O(n log n)` time due to sorting, and the hull construction is `O(n)` since each point is added and removed at most once.

## Example Walkthrough

Let's walk through a concrete example with 5 trees to illustrate how the algorithm finds the convex hull.

**Input:** `trees = [[1,1], [2,2], [2,0], [2,4], [3,3]]`

**Step 1: Sort the points**
After sorting by x-coordinate (then y-coordinate):
`trees = [[1,1], [2,0], [2,2], [2,4], [3,3]]`
Indices:    0      1      2      3      4

**Step 2: Build Lower Hull**
- Start with `stk = [0]` (leftmost point [1,1])
- Add point 1 ([2,0]): 
  - Only one point in stack, so add directly
  - `stk = [0, 1]`
- Add point 2 ([2,2]):
  - Check turn: points [1,1], [2,0], [2,2]
  - Cross product = (1)×(2) - (-1)×(0) = 2 (positive = left turn ✓)
  - Add to stack: `stk = [0, 1, 2]`
- Add point 3 ([2,4]):
  - Check turn: points [2,0], [2,2], [2,4]
  - Cross product = (0)×(2) - (2)×(0) = 0 (collinear ✓)
  - Add to stack: `stk = [0, 1, 2, 3]`
- Add point 4 ([3,3]):
  - Check turn: points [2,2], [2,4], [3,3]
  - Cross product = (0)×(-1) - (2)×(1) = -2 (negative = right turn ✗)
  - Pop point 3, now check [2,0], [2,2], [3,3]
  - Cross product = (0)×(1) - (2)×(1) = -2 (right turn ✗)
  - Pop point 2, now check [1,1], [2,0], [3,3]
  - Cross product = (1)×(3) - (-1)×(1) = 4 (left turn ✓)
  - Add point 4: `stk = [0, 1, 4]`

Lower hull complete: `stk = [0, 1, 4]`, `vis = [T, T, F, F, T]`

**Step 3: Build Upper Hull**
- `m = 3` (size of lower hull). Note that `vis[0]` was never set, and points 2 and 3 were unmarked when they were popped
- Process point 3 ([2,4]):
  - Not visited. The stack has only `m` elements, so no turn check; push it: `stk = [0, 1, 4, 3]`
- Process point 2 ([2,2]):
  - Not visited (it was popped from the lower hull). Check turn: [3,3], [2,4], [2,2]
  - Cross product = (-1)×(-2) - (1)×(0) = 2 (left turn ✓), push it: `stk = [0, 1, 4, 3, 2]`
- Process point 1 ([2,0]): Already visited, skip
- Process point 0 ([1,1]):
  - Not visited. Check turn: [2,4], [2,2], [1,1]
  - Cross product = (0)×(-1) - (-2)×(-1) = -2 (right turn ✗), pop point 2
  - Check turn: [3,3], [2,4], [1,1]: cross product = (-1)×(-3) - (1)×(-1) = 4 (left turn ✓)
  - Push point 0: `stk = [0, 1, 4, 3, 0]`

**Step 4: Remove duplicate and return**
- Pop the last element (it is the starting point 0 again)
- `stk = [0, 1, 4, 3]`
- Return coordinates: `[[1,1], [2,0], [3,3], [2,4]]`

**Visualization:**
```
4 |     [2,4]
3 |         [3,3]
2 |     [2,2]     <- Inside hull, not on perimeter
1 | [1,1]
0 |     [2,0]
  +---------------
    1   2   3
```

The fence connects [1,1] → [2,0] → [3,3] → [2,4] → back to [1,1], with [2,2] enclosed inside but not touching the fence.

## Python Solution

```python
class Solution:
    def outerTrees(self, trees: List[List[int]]) -> List[List[int]]:
        """
        Find the convex hull of a set of points (trees) using Graham's scan algorithm.
        Returns all points on the perimeter of the convex hull.
      
        Args:
            trees: List of [x, y] coordinates representing tree positions
          
        Returns:
            List of points that form the convex hull
        """
      
        def calculate_cross_product(index_a: int, index_b: int, index_c: int) -> int:
            """
            Calculate the cross product of vectors AB and BC.
          
            Positive value: C is to the left of line AB (counter-clockwise turn)
            Negative value: C is to the right of line AB (clockwise turn)
            Zero: A, B, C are collinear
          
            Args:
                index_a: Index of point A in trees array
                index_b: Index of point B in trees array
                index_c: Index of point C in trees array
              
            Returns:
                Cross product value
            """
            point_a, point_b, point_c = trees[index_a], trees[index_b], trees[index_c]
            vector_ab_x = point_b[0] - point_a[0]
            vector_ab_y = point_b[1] - point_a[1]
            vector_bc_x = point_c[0] - point_b[0]
            vector_bc_y = point_c[1] - point_b[1]
          
            return vector_ab_x * vector_bc_y - vector_ab_y * vector_bc_x
      
        num_trees = len(trees)
      
        # If there are 3 or fewer trees, they all form the convex hull
        if num_trees < 4:
            return trees
      
        # Sort trees by x-coordinate first, then by y-coordinate
        trees.sort()
      
        # Track which points have been visited in the lower hull
        is_visited = [False] * num_trees
      
        # Stack to store indices of points in the convex hull
        hull_stack = [0]
      
        # Build lower hull (left to right)
        for current_index in range(1, num_trees):
            # Remove points that create a clockwise turn (not part of convex hull)
            while len(hull_stack) > 1 and calculate_cross_product(
                hull_stack[-2], hull_stack[-1], current_index
            ) < 0:
                removed_index = hull_stack.pop()
                is_visited[removed_index] = False
          
            is_visited[current_index] = True
            hull_stack.append(current_index)
      
        # Mark the size of lower hull to know when to stop popping during upper hull construction
        lower_hull_size = len(hull_stack)
      
        # Build upper hull (right to left)
        for current_index in range(num_trees - 2, -1, -1):
            # Skip points already in the lower hull
            if is_visited[current_index]:
                continue
          
            # Remove points that create a clockwise turn
            while len(hull_stack) > lower_hull_size and calculate_cross_product(
                hull_stack[-2], hull_stack[-1], current_index
            ) < 0:
                hull_stack.pop()
          
            hull_stack.append(current_index)
      
        # Remove the last point as it's the same as the first point (closes the hull)
        hull_stack.pop()
      
        # Convert indices back to coordinates
        return [trees[index] for index in hull_stack]
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)`

The time complexity is dominated by the sorting operation `trees.sort()` which takes `O(n log n)` time where `n` is the number of points. After sorting, the algorithm performs two passes through the points:

1. **First pass (building lower hull):** Iterates through all `n` points once. For each point, it may pop elements from the stack. Since each element can be pushed and popped at most once, this pass takes `O(n)` time.

2. **Second pass (building upper hull):** Similarly iterates through all `n` points in reverse order. Each point is processed at most once, and stack operations are amortized `O(1)` per point, resulting in `O(n)` time.

The `cross` function performs constant time operations `O(1)` for each call.

Overall: `O(n log n) + O(n) + O(n) = O(n log n)`

**Space Complexity:** `O(n)`

The space complexity consists of:

1. **Visited array `vis`:** Takes `O(n)` space to track which points are already in the convex hull.

2. **Stack `stk`:** In the worst case (when all points are on the convex hull), the stack can contain all `n` point indices, requiring `O(n)` space.

3. **Output list:** The final list comprehension creates a new list that can contain up to `n` points in the worst case, using `O(n)` space.

4. **Other variables:** Variables like `i`, `m`, etc., use `O(1)` space.

Total space complexity: `O(n) + O(n) + O(n) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Handling Collinear Points on the Hull Edges**

The most significant pitfall in this problem is dropping collinear points (points that lie on the same straight line along an edge of the fence). Every tree on the fence must be returned, including trees in the middle of an edge.

**Example Problem:**
If you have trees at coordinates `[[0,0], [0,2], [0,4], [2,2]]`, the trees at `[0,0]`, `[0,2]`, and `[0,4]` are collinear. A textbook convex hull that keeps only the corner vertices returns `[0,0]`, `[0,4]`, and `[2,2]`, missing `[0,2]`, which also lies on the fence.

**Why This Happens:**
The cross product is 0 for collinear points. Many convex hull implementations pop on `cross(...) <= 0`, which removes collinear points. The solution pops only on `cross(...) < 0` (a strict right turn), so collinear points stay on the stack.

**Solution:**
Pop only on a strict right turn. An equivalent version without the `vis` array builds the lower and upper hulls separately with the same `< 0` test and merges them with a set:

```python
def outerTrees(self, trees: List[List[int]]) -> List[List[int]]:
    def cross(i, j, k):
        a, b, c = trees[i], trees[j], trees[k]
        return (b[0] - a[0]) * (c[1] - b[1]) - (b[1] - a[1]) * (c[0] - b[0])
  
    n = len(trees)
    if n < 4:
        return trees
  
    trees.sort()
  
    # Build lower hull
    lower = []
    for i in range(n):
        while len(lower) > 1 and cross(lower[-2], lower[-1], i) < 0:
            lower.pop()
        lower.append(i)
  
    # Build upper hull
    upper = []
    for i in range(n - 1, -1, -1):
        while len(upper) > 1 and cross(upper[-2], upper[-1], i) < 0:
            upper.pop()
        upper.append(i)
  
    # Remove duplicates and combine
    hull_indices = set(lower + upper)
    return [trees[i] for i in hull_indices]
```

### 2. **Incorrect Handling of Duplicate Points**

**Problem:**
If the input contains duplicate tree coordinates, the algorithm might include the same point multiple times in the result or fail to handle them properly.

**Solution:**
Either deduplicate the input first or use a set to track unique points:

```python
# Option 1: Deduplicate input
unique_trees = []
seen = set()
for tree in trees:
    tree_tuple = tuple(tree)
    if tree_tuple not in seen:
        seen.add(tree_tuple)
        unique_trees.append(tree)
trees = unique_trees

# Option 2: Use set for final result
hull_points = list({tuple(trees[i]) for i in hull_stack})
return [list(point) for point in hull_points]
```

### 3. **Stack Index Management Issues**

**Problem:**
The `vis` array prevents points of the lower hull from being pushed again during the upper pass. It must be cleared when a point is popped from the lower hull (`vis[removed_index] = False`); otherwise a point that was removed from the lower hull, such as an interior point, would be skipped in the upper pass even if it lies on the upper boundary. Note that the starting point (index 0) is never marked, so the upper pass ends by pushing it again, and the final `hull_stack.pop()` removes that duplicate.

**Solution:**
Keep the `vis` updates exactly as in the solution, or use the separate lower/upper approach from the collinear section above, which does not need `vis` and removes duplicates with a set.

