# 1499. Max Value of Equation

> Source: https://algo.monster/liteproblems/1499

## Problem Statement

You have an array of points on a 2D plane where each point is represented as `[x, y]` coordinates. The points are sorted by their x-values in ascending order, meaning `x₁ < x₂ < x₃ < ... < xₙ`.

Given these points and an integer `k`, you need to find the maximum value of the equation:
`yᵢ + yⱼ + |xᵢ - xⱼ|`

The constraints are:
- You must choose two different points `i` and `j` where `i < j`
- The x-distance between the two points must not exceed `k`: `|xᵢ - xⱼ| ≤ k`
- The problem guarantees at least one valid pair exists

Since the points are sorted by x-values and `i < j`, we know `xⱼ > xᵢ`, so `|xᵢ - xⱼ| = xⱼ - xᵢ`. This simplifies our equation to:
`yᵢ + yⱼ + (xⱼ - xᵢ) = (yᵢ - xᵢ) + (xⱼ + yⱼ)`

The solution uses a priority [queue](/problems/queue_intro) (min-heap) to efficiently track potential point pairs. For each point `(x, y)`:
1. Remove points from the heap where the x-distance exceeds `k`
2. If valid points remain, calculate the maximum value using the current point and the best previous point
3. Add the current point to the heap with key `(x - y)` for future comparisons

The heap stores tuples of `(x - y, x)` where the first element serves as the priority key. By maintaining the minimum `x - y` value at the top of the heap, we can maximize the equation value when combined with future points.

## Intuition

The key insight comes from rewriting the equation. Since points are sorted by x-values and we need `i < j`, we know `xⱼ > xᵢ`, which means `|xᵢ - xⱼ| = xⱼ - xᵢ`.

So our equation becomes:
`yᵢ + yⱼ + (xⱼ - xᵢ) = (yⱼ + xⱼ) + (yᵢ - xᵢ)`

This transformation is crucial! Now we can think of each point as having two components:
- When point `j` is the second point: contribute `(yⱼ + xⱼ)`
- When point `i` is the first point: contribute `(yᵢ - xᵢ)`

As we iterate through points from left to right, for each current point `j`, we want to find the best previous point `i` that:
1. Is within distance `k`: `xⱼ - xᵢ ≤ k`
2. Maximizes `(yᵢ - xᵢ)`

This naturally suggests maintaining a collection of previous points, but we need to handle two operations efficiently:
- Remove points that are now too far away (outside the `k` distance)
- Find the point with maximum `(yᵢ - xᵢ)` value

A min-heap works perfectly here! We store `(xᵢ - yᵢ, xᵢ)` in the heap (negating to get maximum). The heap automatically keeps the best candidate at the top. As we process each new point, we:
- Clean out points beyond distance `k`
- Use the heap's top element (if any) to calculate a potential maximum
- Add the current point for future use

This [sliding window](/problems/sliding_window_maximum) approach with a heap ensures we always consider the best valid pairing while maintaining efficiency.

**Learn more about [Queue](/problems/queue_intro), [Sliding Window](/problems/sliding_window_maximum), [Monotonic Queue](/problems/mono_stack_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

We implement the solution using a min-heap to maintain candidate points efficiently:

**Data Structure**: A min-heap (priority [queue](/problems/queue_intro)) storing tuples `(x - y, x)` where:
- First element `(x - y)` serves as the priority key (smaller values have higher priority)
- Second element `x` helps us check the distance constraint

**Algorithm Steps**:

1. **Initialize variables**:
   - `ans = -inf` to track the maximum value found
   - `pq = []` as our min-heap

2. **Process each point** `(x, y)` in order:
 
   a. **Clean expired points**: 
   ```python
   while pq and x - pq[0][1] > k:
       heappop(pq)
   ```
   Remove points from the heap where the x-distance exceeds `k`. Since `pq[0][1]` contains the x-coordinate of the heap's top element, we check if `x - pq[0][1] > k`.

   b. **Calculate maximum with current point**:
   ```python
   if pq:
       ans = max(ans, x + y - pq[0][0])
   ```
   If valid points remain, compute the equation value. Since `pq[0][0]` stores `(xᵢ - yᵢ)` and we need `(yᵢ - xᵢ)`, we use `-pq[0][0]`. The complete equation becomes:
   - `(y + x) + (-pq[0][0])` 
   - `= (y + x) + (yᵢ - xᵢ)`
   - `= yⱼ + xⱼ + yᵢ - xᵢ`

   c. **Add current point to heap**:
   ```python
   heappush(pq, (x - y, x))
   ```
   Store the current point for future iterations with its `(x - y)` value as the priority key.

3. **Return the maximum value** found across all valid pairs.

The min-heap ensures we always have access to the point with minimum `(x - y)` value, which equivalently gives us the maximum `(y - x)` value needed to maximize our equation. The [sliding window](/problems/sliding_window_maximum) constraint is maintained by removing points beyond distance `k`.

## Example Walkthrough

Let's walk through the algorithm with points = [[1,3], [2,0], [5,10], [6,-10]] and k = 3.

**Initial state:**
- ans = -∞
- pq = [] (empty min-heap)

**Processing point [1,3]:**
- x=1, y=3
- Clean expired: pq is empty, nothing to remove
- Calculate max: pq is empty, skip
- Add to heap: pq = [(1-3, 1)] = [(-2, 1)]

**Processing point [2,0]:**
- x=2, y=0
- Clean expired: Check if 2 - 1 > 3? No (1 ≤ 3), keep [(-2, 1)]
- Calculate max: ans = max(-∞, 2 + 0 - (-2)) = max(-∞, 4) = 4
  - This represents pairing points [1,3] and [2,0]: 3 + 0 + |1-2| = 4
- Add to heap: pq = [(-2, 1), (2, 2)]

**Processing point [5,10]:**
- x=5, y=10
- Clean expired: 
  - Check if 5 - 1 > 3? Yes (4 > 3), remove [(-2, 1)]
  - Check if 5 - 2 > 3? Yes (3 ≤ 3), keep [(2, 2)]
- Calculate max: ans = max(4, 5 + 10 - 2) = max(4, 13) = 13
  - This represents pairing points [2,0] and [5,10]: 0 + 10 + |2-5| = 13
- Add to heap: pq = [(-5, 5), (2, 2)]

**Processing point [6,-10]:**
- x=6, y=-10
- Clean expired:
  - Check if 6 - 5 > 3? No (1 ≤ 3), keep [(-5, 5)]
  - Note: (2, 2) would be checked next but (-5, 5) has higher priority
- Calculate max: ans = max(13, 6 + (-10) - (-5)) = max(13, 1) = 13
  - This represents pairing points [5,10] and [6,-10]: 10 + (-10) + |5-6| = 1
- Add to heap: pq = [(-5, 5), (2, 2), (16, 6)]

**Final answer: 13**

The maximum value comes from pairing points [2,0] and [5,10], giving us 0 + 10 + 3 = 13.

## Python Solution

```python
from typing import List
import heapq
from math import inf

class Solution:
    def findMaxValueOfEquation(self, points: List[List[int]], k: int) -> int:
        # Initialize the maximum result as negative infinity
        max_value = -inf
      
        # Min-heap to store tuples of (x - y, x)
        # We want to maximize (y - x) + (current_y + current_x)
        # Since heapq is a min-heap, we store (x - y) to get the minimum (x - y),
        # which corresponds to maximum (y - x)
        min_heap = []
      
        # Iterate through each point
        for current_x, current_y in points:
            # Remove points from heap that are too far away (distance > k)
            while min_heap and current_x - min_heap[0][1] > k:
                heapq.heappop(min_heap)
          
            # If heap is not empty, calculate the value of the equation
            # with the best previous point
            if min_heap:
                # The equation value is: current_x + current_y - (x_i - y_i)
                # which equals: current_x + current_y + y_i - x_i
                max_value = max(max_value, current_x + current_y - min_heap[0][0])
          
            # Add current point to the heap for future comparisons
            # Store (x - y, x) tuple
            heapq.heappush(min_heap, (current_x - current_y, current_x))
      
        return max_value
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)` where `n` is the number of points.

- The code iterates through all `n` points once in the main loop.
- For each point, we perform heap operations:
  - The while loop pops elements from the heap when they violate the distance constraint `x - pq[0][1] > k`. In the worst case, all previously added elements might be popped, but each element is popped at most once throughout the entire execution.
  - Each `heappop` operation takes `O(log n)` time.
  - Each `heappush` operation takes `O(log n)` time.
- Since each point is pushed exactly once and popped at most once, the total number of heap operations is at most `2n`.
- Therefore, the overall time complexity is `O(n log n)`.

**Space Complexity:** `O(n)`

- The priority queue `pq` can contain at most `n` elements in the worst case (when all points satisfy the distance constraint with each other).
- Each element in the heap is a tuple `(x - y, x)` which takes constant space.
- Therefore, the space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Heap Key Selection**
A common mistake is using the wrong value as the heap's priority key. Developers might intuitively store `(y - x, x)` instead of `(x - y, x)`, thinking they want to maximize `(y - x)`.

**Why it's wrong**: Python's heapq is a min-heap, so storing `(y - x)` would give us the minimum `(y - x)` value at the top, which is the opposite of what we need.

**Solution**: Store `(x - y, x)` in the heap. The minimum `(x - y)` value corresponds to the maximum `(y - x)` value, which is what we need to maximize the equation.

### 2. **Forgetting to Clean Expired Points**
Developers might calculate the maximum value before removing points that violate the distance constraint:

```python
# WRONG: Calculating before cleaning
if min_heap:
    max_value = max(max_value, current_x + current_y - min_heap[0][0])
  
while min_heap and current_x - min_heap[0][1] > k:
    heapq.heappop(min_heap)
```

**Why it's wrong**: This could use points that are beyond the allowed distance `k`, leading to incorrect results.

**Solution**: Always clean the heap first, then calculate:
```python
# CORRECT: Clean first, then calculate
while min_heap and current_x - min_heap[0][1] > k:
    heapq.heappop(min_heap)
  
if min_heap:
    max_value = max(max_value, current_x + current_y - min_heap[0][0])
```

### 3. **Using Maximum Heap Instead of Minimum Heap**
Some developers try to use a max-heap by negating values:

```python
# Attempting to use max-heap by negating
heapq.heappush(min_heap, (-(y - x), x))
# Later retrieving with:
max_value = max(max_value, current_x + current_y + min_heap[0][0])
```

**Why it's confusing**: While this can work, it adds unnecessary complexity and makes the code harder to understand. The double negation can lead to sign errors.

**Solution**: Stick with the natural min-heap behavior and adjust the formula accordingly. Store `(x - y)` and use `-min_heap[0][0]` when calculating, which is clearer and less error-prone.

### 4. **Incorrect Distance Calculation**
A subtle error is using the wrong point's x-coordinate when checking the distance constraint:

```python
# WRONG: Using the wrong index or variable
while min_heap and min_heap[0][0] - current_x > k:  # Using priority key instead of x
    heapq.heappop(min_heap)
```

**Why it's wrong**: `min_heap[0][0]` contains `(x - y)`, not the x-coordinate. The distance check needs the actual x-coordinate.

**Solution**: Always use `min_heap[0][1]` for the x-coordinate:
```python
# CORRECT: Using the x-coordinate from the tuple
while min_heap and current_x - min_heap[0][1] > k:
    heapq.heappop(min_heap)
```

### 5. **Not Initializing Maximum Value Properly**
Starting with `max_value = 0` or `max_value = None`:

```python
# WRONG: Could miss negative results
max_value = 0
```

**Why it's wrong**: The maximum value could be negative if all valid pairs produce negative results. Starting with 0 would incorrectly return 0 instead of the actual negative maximum.

**Solution**: Initialize with negative infinity:
```python
max_value = -inf  # or float('-inf')
```

