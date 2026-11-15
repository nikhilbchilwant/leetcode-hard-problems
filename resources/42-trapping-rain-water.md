# 42. Trapping Rain Water

> Source: https://algo.monster/liteproblems/42

## Problem Statement

You are given an array of `n` non-negative integers where each integer represents the height of a bar in an elevation map. Each bar has a width of 1 unit. Your task is to calculate how much rainwater can be trapped between the bars after it rains.

The water gets trapped between bars when there are taller bars on both sides that can act as walls. For any position `i`, water can be trapped above it if there exists a taller bar somewhere to its left and another taller bar somewhere to its right. The amount of water that can be trapped at position `i` is determined by the minimum height of the tallest bars on both sides minus the height of the current bar.

For example, if you have heights `[0,1,0,2,1,0,1,3,2,1,2,1]`, the total trapped water would be 6 units. Water fills in the valleys created by the elevation differences, but only up to the level constrained by the shorter of the two surrounding peaks.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) to pre-compute two arrays:
- `left[i]`: stores the maximum height from the start up to position `i`
- `right[i]`: stores the maximum height from position `i` to the end

For each position, the water level above it is `min(left[i], right[i])`, and the actual water trapped is `min(left[i], right[i]) - height[i]`. The total trapped water is the sum of water at all positions.

## Intuition

To understand how water gets trapped, imagine looking at each bar individually. At any given position, water can only be held if there are taller "walls" on both sides. The water level at that position will be determined by the shorter of these two walls - water would overflow from the shorter side if we tried to fill it higher.

Think of it like filling a bathtub - the water level can't exceed the height of the lowest wall. For each position `i`, we need to know:
1. What's the tallest bar to my left (including myself)?
2. What's the tallest bar to my right (including myself)?

The water level above position `i` would be `min(max_left, max_right)`. Since the bar itself takes up space, the actual water trapped is `min(max_left, max_right) - height[i]`.

The key insight is that we can pre-compute these maximum heights efficiently. Instead of checking all bars to the left and right for each position (which would be inefficient), we can build two arrays in linear time:
- Build `left[]` by scanning from left to right, where `left[i] = max(left[i-1], height[i])`
- Build `right[]` by scanning from right to left, where `right[i] = max(right[i+1], height[i])`

This way, we calculate each maximum only once and reuse the results. The pattern here is that the maximum height up to position `i` is either the bar at position `i` itself or the maximum we've seen so far before position `i`. This [dynamic programming](/problems/dynamic_programming_intro) approach reduces our time complexity from `O(n²)` to `O(n)`.

**Learn more about [Stack](/problems/stack_intro), [Two Pointers](/problems/two_pointers_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**


### Watch it run

Step through the three passes below: tallest wall to the left, tallest to the right, then the water each bar holds.

<iframe src="/widgets/trapping-rain-water-arrays.html" width="100%" height="560" frameborder="0" style="border:1px solid #e5e7eb;border-radius:12px;width:100%;min-height:160px;touch-action:pan-y;" title="Trapping Rain Water walkthrough"></iframe>

## Solution Approach

The implementation uses [dynamic programming](/problems/dynamic_programming_intro) with two auxiliary arrays to efficiently compute the trapped water.

**Step 1: Initialize the arrays**
- Create `left` array initialized with `height[0]` repeated `n` times
- Create `right` array initialized with `height[-1]` repeated `n` times
- These arrays will store the maximum heights from left and right respectively

**Step 2: Build the maximum height arrays**
The algorithm fills both arrays in a single loop from `i = 1` to `n-1`:
- For `left[i]`: Calculate `max(left[i-1], height[i])` 
  - This gives us the maximum height from position 0 to position `i`
  - We build this left-to-right since each position depends on the previous one

- For `right[n-i-1]`: Calculate `max(right[n-i], height[n-i-1])`
  - This gives us the maximum height from position `n-i-1` to position `n-1`
  - We build this right-to-left by using index manipulation `n-i-1`

**Step 3: Calculate trapped water**
Iterate over every index `i`:
- For each position, calculate `min(left[i], right[i]) - height[i]`
- `min(left[i], right[i])` gives the water level at position `i`
- Subtracting `height[i]` gives the actual water trapped above the bar
- Sum all these values to get the total trapped water

**Time Complexity**: `O(n)` - We make two passes through the array (one for building the arrays, one for calculating the sum)

**Space Complexity**: `O(n)` - We use two additional arrays of size `n` to store the maximum heights

The elegance of this solution lies in pre-computing the required information, allowing us to calculate the trapped water at each position in constant time during the final summation.

## Example Walkthrough

<details>
<summary>Prefer text? Read the step-by-step walkthrough.</summary>

Let's walk through a small example with heights `[3, 0, 2, 0, 4]` to see how the solution works.

**Initial Setup:**
- Array: `[3, 0, 2, 0, 4]`
- n = 5
- Initialize `left = [3, 3, 3, 3, 3]` (all set to height[0])
- Initialize `right = [4, 4, 4, 4, 4]` (all set to height[-1])

**Step 1: Build the maximum height arrays**

Building `left[]` (max height from left up to position i):
- i = 1: `left[1] = max(left[0], height[1]) = max(3, 0) = 3`
- i = 2: `left[2] = max(left[1], height[2]) = max(3, 2) = 3`
- i = 3: `left[3] = max(left[2], height[3]) = max(3, 0) = 3`
- i = 4: `left[4] = max(left[3], height[4]) = max(3, 4) = 4`
- Result: `left = [3, 3, 3, 3, 4]`

Building `right[]` (max height from position i to the end):
- i = 1, pos = 3: `right[3] = max(right[4], height[3]) = max(4, 0) = 4`
- i = 2, pos = 2: `right[2] = max(right[3], height[2]) = max(4, 2) = 4`
- i = 3, pos = 1: `right[1] = max(right[2], height[1]) = max(4, 0) = 4`
- i = 4, pos = 0: `right[0] = max(right[1], height[0]) = max(4, 3) = 4`
- Result: `right = [4, 4, 4, 4, 4]`

**Step 2: Calculate trapped water at each position**

For each position, water trapped = `min(left[i], right[i]) - height[i]`:

| Position | height | left | right | min(left, right) | Water Trapped |
|----------|--------|------|-------|------------------|---------------|
| 0        | 3      | 3    | 4     | 3                | 3 - 3 = 0     |
| 1        | 0      | 3    | 4     | 3                | 3 - 0 = 3     |
| 2        | 2      | 3    | 4     | 3                | 3 - 2 = 1     |
| 3        | 0      | 3    | 4     | 3                | 3 - 0 = 3     |
| 4        | 4      | 4    | 4     | 4                | 4 - 4 = 0     |

**Total water trapped = 0 + 3 + 1 + 3 + 0 = 7 units**

Visual representation:
```
        #
# ≈ ≈ ≈ #
# ≈ # ≈ #
# ≈ # ≈ #
3 0 2 0 4
```
Where `#` is a bar and `≈` represents trapped water: 3 units above index 1, 1 unit above index 2, and 3 units above index 3. The water fills the valleys between the bars, with the level capped by the minimum of the tallest bars on each side.


</details>

## Python Solution

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        """
        Calculate the amount of water that can be trapped after raining.
      
        Args:
            height: List of non-negative integers representing elevation map
          
        Returns:
            Total amount of trapped rainwater
        """
        n = len(height)
      
        # Initialize arrays to store maximum height to the left and right of each position
        left_max = [0] * n
        right_max = [0] * n
      
        # Base cases: first element for left_max, last element for right_max
        left_max[0] = height[0]
        right_max[n - 1] = height[n - 1]
      
        # Fill left_max array: for each position, store the maximum height seen so far from the left
        for i in range(1, n):
            left_max[i] = max(left_max[i - 1], height[i])
      
        # Fill right_max array: for each position, store the maximum height seen so far from the right
        for i in range(n - 2, -1, -1):
            right_max[i] = max(right_max[i + 1], height[i])
      
        # Calculate trapped water at each position
        # Water level at position i = min(left_max[i], right_max[i])
        # Trapped water = water level - height of bar at position i
        total_water = 0
        for i in range(n):
            water_level = min(left_max[i], right_max[i])
            total_water += water_level - height[i]
      
        return total_water
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the length of the input array `height`. This is because:
- The initialization of `left` and `right` arrays takes `O(n)` time
- The for loop iterates through the array once with `n-1` iterations, performing constant time operations (`max` comparisons and assignments) in each iteration, resulting in `O(n)` time
- The final loop iterates through all `n` elements once, performing constant time operations (`min` comparison and subtraction) for each element, which is `O(n)` time
- Overall: `O(n) + O(n) + O(n) = O(n)`

The space complexity is `O(n)`, where `n` is the length of the input array `height`. This is because:
- Two auxiliary arrays `left` and `right` are created, each of size `n`, requiring `O(n)` space each
- Total auxiliary space: `O(n) + O(n) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Edge Case: Empty or Single Element Array**
A frequent oversight is not handling arrays with length 0 or 1, which can cause index out of bounds errors when initializing `left_max[0]` or `right_max[n-1]`.

**Problem Code:**
```python
def trap(self, height: List[int]) -> int:
    n = len(height)
    left_max = [0] * n
    right_max = [0] * n
    left_max[0] = height[0]  # IndexError if height is empty
    right_max[n - 1] = height[n - 1]
```

**Solution:**
Add an early return for arrays that cannot trap water:
```python
def trap(self, height: List[int]) -> int:
    n = len(height)
    if n <= 2:  # Cannot trap water with fewer than 3 bars
        return 0
  
    left_max = [0] * n
    right_max = [0] * n
    # ... rest of the code
```

### 2. **Incorrect Loop Bounds When Building Arrays**
Mixing up the loop boundaries or indices when filling the `right_max` array is a common mistake, especially when trying to build both arrays in a single loop.

**Problem Code:**
```python
# Attempting to fill both arrays in one loop incorrectly
for i in range(1, n):
    left_max[i] = max(left_max[i-1], height[i])
    right_max[n-i] = max(right_max[n-i+1], height[n-i])  # Off-by-one error
```

**Solution:**
Keep the loops separate for clarity, or carefully manage indices:
```python
# Separate loops (clearer)
for i in range(1, n):
    left_max[i] = max(left_max[i-1], height[i])

for i in range(n-2, -1, -1):
    right_max[i] = max(right_max[i+1], height[i])

# Or combined with correct indices
for i in range(1, n):
    left_max[i] = max(left_max[i-1], height[i])
    right_max[n-i-1] = max(right_max[n-i], height[n-i-1])
```

### 3. **Memory Optimization Confusion**
When attempting the two-pointer optimization to reduce space complexity from O(n) to O(1), developers often incorrectly update the pointers or water calculation logic.

**Problem Code:**
```python
# Incorrect two-pointer implementation
left, right = 0, n - 1
left_max = right_max = 0
total = 0

while left < right:
    if height[left] < height[right]:
        total += left_max - height[left]  # May add negative water
        left_max = max(left_max, height[left])
        left += 1
```

**Solution:**
Update the maximum before calculating water, and ensure non-negative water:
```python
while left < right:
    if height[left] < height[right]:
        left_max = max(left_max, height[left])
        total += max(0, left_max - height[left])  # Or update after max calculation
        left += 1
    else:
        right_max = max(right_max, height[right])
        total += max(0, right_max - height[right])
        right -= 1
```

