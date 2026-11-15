# 1840. Maximum Building Height

> Source: https://algo.monster/liteproblems/1840

## Problem Statement

You need to build `n` new buildings in a city, numbered from `1` to `n` and arranged in a line. Each building must follow these rules:

1. **Every building must have a non-negative integer height**
2. **Building 1 must have height 0**
3. **Adjacent buildings can differ in height by at most 1** (the height can increase or decrease by at most 1 between any two consecutive buildings)

Additionally, some buildings have maximum height restrictions given in a 2D array `restrictions`, where `restrictions[i] = [id_i, maxHeight_i]` means building `id_i` cannot exceed height `maxHeight_i`.

Key guarantees:
- Each building appears at most once in the restrictions
- Building 1 is never in the restrictions (since it must be height 0)

Your task is to find the **maximum possible height of the tallest building** while satisfying all the constraints.

For example, if you have 5 buildings and no restrictions, starting from building 1 with height 0, you could have heights like `[0, 1, 2, 3, 4]` making the tallest building have height 4. But if there's a restriction that building 3 can't exceed height 1, you'd need to adjust the heights accordingly, perhaps to something like `[0, 1, 1, 2, 3]` or `[0, 1, 1, 0, 1]`, depending on what maximizes the tallest building's height.

The challenge is to figure out the optimal height assignment that respects both the adjacent height difference constraint and all the specific building restrictions while maximizing the peak height achieved.

## Intuition

Think of the buildings as a mountain range where you can only climb or descend by 1 unit at a time. The restrictions act like "ceiling points" that your mountain cannot exceed at specific locations.

The key insight is that between any two restricted buildings, the height profile forms a "tent" shape - it goes up from the left restriction point and down to the right restriction point (or just up, or just down, depending on the distance and heights). To find the maximum possible height between two points, we need to know the exact height limits at those points first.

But here's the catch: the restrictions influence each other! A restriction at building 10 with max height 5 might actually need to be lower if there's a restriction at building 5 with max height 2 (since we can only increase by 1 per building). This creates a "propagation effect" where restrictions limit not just their own building but nearby buildings too.

So we process the restrictions in two passes:
1. **Left to right**: Each restriction is limited by how high we can climb from the previous restriction. If building `i` is at height `h1` and building `j` is `d` positions away, we can climb at most to `h1 + d`.
2. **Right to left**: Similarly, each restriction is limited by how high we can climb from the next restriction going backwards.

After both passes, we have the true maximum height possible at each restricted building.

Now, between any two consecutive restrictions, we want to build as high as possible. The optimal strategy is to climb as high as we can from both ends and meet at a peak in the middle. If we have heights `h1` and `h2` at positions `p1` and `p2`, and we want to reach maximum height `t` somewhere between them, we need:
- `t - h1` steps to climb from the left
- `t - h2` steps to descend to the right  
- Total steps available: `p2 - p1`

This gives us: `(t - h1) + (t - h2) ≤ p2 - p1`, which solves to `t ≤ (h1 + h2 + p2 - p1) / 2`.

The maximum height of all buildings is the maximum of all these peaks between consecutive restrictions.

**Learn more about [Math](/problems/math-basics) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows these key steps:

**1. Prepare the restrictions list:**
- Add `[1, 0]` to the restrictions (building 1 must have height 0)
- Sort all restrictions by building number
- If the last building `n` isn't restricted, add `[n, n-1]` as its maximum possible height (climbing 1 per building from building 1)

**2. Left-to-right pass to propagate constraints:**
```python
for i in range(1, m):
    r[i][1] = min(r[i][1], r[i-1][1] + r[i][0] - r[i-1][0])
```
For each restriction, we update its maximum height based on the previous restriction. The maximum height we can reach from building `r[i-1][0]` with height `r[i-1][1]` to building `r[i][0]` is `r[i-1][1] + (r[i][0] - r[i-1][0])` since we can increase by at most 1 per building. We take the minimum with the original restriction.

**3. Right-to-left pass to propagate constraints:**
```python
for i in range(m - 2, 0, -1):
    r[i][1] = min(r[i][1], r[i+1][1] + r[i+1][0] - r[i][0])
```
Similar logic but going backwards. Each restriction is limited by what we can reach from the next restriction when going in reverse.

**4. Calculate maximum height between consecutive restrictions:**
```python
for i in range(m - 1):
    t = (r[i][1] + r[i+1][1] + r[i+1][0] - r[i][0]) // 2
    ans = max(ans, t)
```
Between any two consecutive restrictions at positions `r[i][0]` and `r[i+1][0]` with heights `r[i][1]` and `r[i+1][1]`, we calculate the maximum achievable height using the formula:
`t = (r[i][1] + r[i+1][1] + r[i+1][0] - r[i][0]) / 2`

This formula comes from the fact that to reach height `t` from both ends:
- We need `t - r[i][1]` steps climbing from the left
- We need `t - r[i+1][1]` steps descending to the right
- Total distance available: `r[i+1][0] - r[i][0]`

The constraint `(t - r[i][1]) + (t - r[i+1][1]) ≤ r[i+1][0] - r[i][0]` gives us the maximum `t`.

**5. Return the maximum height found:**
The answer is the maximum height achievable across all segments between consecutive restrictions.

The time complexity is `O(k log k)` where `k` is the number of restrictions (for [sorting](/problems/sorting_summary)), and space complexity is `O(k)` for storing the modified restrictions list.

## Example Walkthrough

Let's walk through an example with `n = 6` buildings and restrictions `[[3, 2], [5, 2]]`.

**Step 1: Prepare restrictions**
- Add building 1 with height 0: `[[1, 0], [3, 2], [5, 2]]`
- Already sorted by building number
- Add building 6 (last building): `[[1, 0], [3, 2], [5, 2], [6, 5]]`
  - Building 6 can be at most height 5 (distance 5 from building 1)

**Step 2: Left-to-right pass**
- Building 3: Can we reach height 2 from building 1?
  - Distance: 3 - 1 = 2 steps
  - Max height from building 1: 0 + 2 = 2
  - min(2, 2) = 2 ✓ (no change needed)
- Building 5: Can we reach height 2 from building 3?
  - Distance: 5 - 3 = 2 steps
  - Max height from building 3: 2 + 2 = 4
  - min(2, 4) = 2 ✓ (no change needed)
- Building 6: Can we reach height 5 from building 5?
  - Distance: 6 - 5 = 1 step
  - Max height from building 5: 2 + 1 = 3
  - min(5, 3) = 3 (reduced from 5 to 3!)

Restrictions after left-to-right: `[[1, 0], [3, 2], [5, 2], [6, 3]]`

**Step 3: Right-to-left pass**
- Building 5: Can we reach height 2 from building 6?
  - Distance: 6 - 5 = 1 step
  - Max height from building 6: 3 + 1 = 4
  - min(2, 4) = 2 ✓ (no change needed)
- Building 3: Can we reach height 2 from building 5?
  - Distance: 5 - 3 = 2 steps
  - Max height from building 5: 2 + 2 = 4
  - min(2, 4) = 2 ✓ (no change needed)
- Building 1: Already fixed at 0

Final restrictions: `[[1, 0], [3, 2], [5, 2], [6, 3]]`

**Step 4: Calculate maximum heights between consecutive restrictions**

Between buildings 1 and 3:
- Heights: h₁ = 0, h₂ = 2
- Distance: 3 - 1 = 2
- Max height: (0 + 2 + 2) / 2 = 2
- This peak occurs at building 3 itself

Between buildings 3 and 5:
- Heights: h₁ = 2, h₂ = 2
- Distance: 5 - 3 = 2
- Max height: (2 + 2 + 2) / 2 = 3
- This peak occurs at building 4

Between buildings 5 and 6:
- Heights: h₁ = 2, h₂ = 3
- Distance: 6 - 5 = 1
- Max height: (2 + 3 + 1) / 2 = 3
- This peak occurs at building 6 itself

**Step 5: Return maximum**
The maximum height is 3, achieved at building 4 and building 6.

The final building heights could be: `[0, 1, 2, 3, 2, 3]`
- Building 1: height 0 (required)
- Building 2: height 1 (climbing from 0)
- Building 3: height 2 (restricted, climbing from 1)
- Building 4: height 3 (peak between restrictions)
- Building 5: height 2 (restricted, descending from 3)
- Building 6: height 3 (climbing from 2)

## Python Solution

```python
class Solution:
    def maxBuilding(self, n: int, restrictions: List[List[int]]) -> int:
        # Create a mutable copy of restrictions to avoid modifying the input
        height_restrictions = restrictions.copy()
      
        # Add the starting position: building 1 must have height 0
        height_restrictions.append([1, 0])
      
        # Sort restrictions by position
        height_restrictions.sort()
      
        # Add the last position if not already present
        # The maximum possible height at position n is n-1 (starting from 0 at position 1)
        if not height_restrictions or height_restrictions[-1][0] != n:
            height_restrictions.append([n, n - 1])
      
        num_restrictions = len(height_restrictions)
      
        # Forward pass: propagate height constraints from left to right
        # Each building's height is limited by the previous building's height plus the distance
        for i in range(1, num_restrictions):
            max_height_from_left = height_restrictions[i - 1][1] + (height_restrictions[i][0] - height_restrictions[i - 1][0])
            height_restrictions[i][1] = min(height_restrictions[i][1], max_height_from_left)
      
        # Backward pass: propagate height constraints from right to left
        # Each building's height is also limited by the next building's height plus the distance
        for i in range(num_restrictions - 2, 0, -1):
            max_height_from_right = height_restrictions[i + 1][1] + (height_restrictions[i + 1][0] - height_restrictions[i][0])
            height_restrictions[i][1] = min(height_restrictions[i][1], max_height_from_right)
      
        # Find the maximum possible height between consecutive restriction points
        max_height = 0
        for i in range(num_restrictions - 1):
            # Calculate the maximum height that can be achieved between two consecutive points
            # The formula finds the peak height when growing from both endpoints
            left_height = height_restrictions[i][1]
            right_height = height_restrictions[i + 1][1]
            distance = height_restrictions[i + 1][0] - height_restrictions[i][0]
          
            # Maximum height between two points with given endpoint heights
            peak_height = (left_height + right_height + distance) // 2
            max_height = max(max_height, peak_height)
      
        return max_height
```

## Time and Space Complexity

The time complexity is `O(m × log m)`, where `m` is the number of restrictions (constraints).

Breaking down the time complexity:
- Adding restrictions to the list: `O(1)` for each append operation
- Sorting the restrictions: `O(m × log m)` where `m` is the total number of restrictions including the added ones
- First forward pass (propagating constraints left to right): `O(m)` iterations
- Second backward pass (propagating constraints right to left): `O(m)` iterations  
- Final loop to calculate maximum height: `O(m)` iterations

The dominant operation is the sorting step, giving us an overall time complexity of `O(m × log m)`.

The space complexity is `O(m)`, where `m` is the number of restrictions.

Space complexity analysis:
- The restrictions list `r` stores at most `m + 2` elements (original restrictions plus two boundary conditions)
- No additional data structures are used that scale with input size
- Only constant extra variables are used (`ans`, `t`, loop indices)

Therefore, the space complexity is `O(m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Modifying the Original Input Array**
A critical pitfall is directly modifying the input `restrictions` array when adding new constraints. This can cause issues if the function is called multiple times or if the caller expects the input to remain unchanged.

**Problem Code:**
```python
def maxBuilding(self, n: int, restrictions: List[List[int]]) -> int:
    restrictions.append([1, 0])  # Directly modifying input!
    restrictions.sort()
    # ... rest of code
```

**Solution:**
Always create a copy of the input array before modifying it:
```python
def maxBuilding(self, n: int, restrictions: List[List[int]]) -> int:
    height_restrictions = restrictions.copy()  # Create a copy first
    height_restrictions.append([1, 0])
    height_restrictions.sort()
    # ... rest of code
```

### 2. **Integer Division vs Float Division**
When calculating the peak height between two restrictions, using float division can lead to incorrect results since building heights must be integers.

**Problem Code:**
```python
peak_height = (left_height + right_height + distance) / 2  # Float division!
max_height = max(max_height, peak_height)  # Comparing int with float
```

**Solution:**
Use integer division to ensure the result is an integer:
```python
peak_height = (left_height + right_height + distance) // 2  # Integer division
max_height = max(max_height, peak_height)
```

### 3. **Forgetting to Check if Last Building Already Has a Restriction**
Adding a restriction for building `n` without checking if it already exists in the restrictions can create duplicate entries and incorrect calculations.

**Problem Code:**
```python
height_restrictions.append([n, n - 1])  # Always adding, even if n is already restricted!
```

**Solution:**
Check if the last building is already in the sorted restrictions before adding:
```python
if not height_restrictions or height_restrictions[-1][0] != n:
    height_restrictions.append([n, n - 1])
```

### 4. **Incorrect Backward Pass Range**
The backward pass should skip index 0 (building 1 with height 0) since it's fixed and shouldn't be updated.

**Problem Code:**
```python
for i in range(num_restrictions - 2, -1, -1):  # Goes all the way to index 0!
    height_restrictions[i][1] = min(height_restrictions[i][1], ...)
```

**Solution:**
Stop at index 1 to avoid updating the fixed building 1:
```python
for i in range(num_restrictions - 2, 0, -1):  # Stops at index 1
    height_restrictions[i][1] = min(height_restrictions[i][1], ...)
```

### 5. **Not Handling Edge Case of Empty Restrictions**
If the original restrictions list is empty, the code might fail when checking `height_restrictions[-1][0]`.

**Problem Code:**
```python
if height_restrictions[-1][0] != n:  # IndexError if height_restrictions is empty!
    height_restrictions.append([n, n - 1])
```

**Solution:**
Add a check for empty list:
```python
if not height_restrictions or height_restrictions[-1][0] != n:
    height_restrictions.append([n, n - 1])
```

