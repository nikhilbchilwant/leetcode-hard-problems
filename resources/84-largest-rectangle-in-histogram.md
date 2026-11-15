# 84. Largest Rectangle in Histogram

> Source: https://algo.monster/liteproblems/84

## Problem Statement

You are given an array `heights` where each element represents the height of a bar in a histogram. Each bar has a width of 1 unit. Your task is to find the area of the largest rectangle that can be formed within the histogram.

The rectangle must be formed by selecting consecutive bars and is limited by the height of the shortest bar in the selected range. For example, if you select bars from index `i` to index `j`, the rectangle's height would be the minimum height among all bars in that range, and its width would be `(j - i + 1)`.

The solution uses a monotonic [stack](/problems/stack_intro) approach to efficiently find, for each bar, the boundaries where it can extend as the minimum height. For each bar at position `i` with height `h`:
- `left[i]` stores the index of the nearest bar to the left that has a height less than `h`
- `right[i]` stores the index of the nearest bar to the right that has a height less than or equal to `h` (because the stack pops on `>=`). For a run of equal heights, this makes earlier bars in the run stop short, but the last bar of the run still extends across the whole run, so the maximum area is unaffected.

The algorithm maintains a [stack](/problems/stack_intro) of indices in increasing order of heights. When processing each bar:
1. It pops bars from the stack that are taller than or equal to the current bar, updating their `right` boundary
2. The top of the stack (if exists) becomes the `left` boundary for the current bar
3. The current bar's index is pushed onto the stack

Finally, for each bar with height `h` at position `i`, the maximum rectangle area with that bar as the minimum height is calculated as `h * (right[i] - left[i] - 1)`. The answer is the maximum among all such areas.

## Intuition

The key insight is that every rectangle in the histogram must have some bar that serves as its "limiting height" - the shortest bar within the rectangle's span. This observation leads us to consider each bar as a potential limiting height and find the maximum rectangle that can be formed with that specific height.

For a bar at position `i` with height `h`, if we want to use it as the limiting height of a rectangle, we need to extend the rectangle as far as possible to both left and right. The rectangle can extend until we hit a bar that is shorter than `h` on either side. Why? Because if we include a shorter bar, it would become the new limiting height, contradicting our assumption that bar `i` is the limiting height.

This means for each bar, we need to find:
- The first bar to its left that is shorter than it
- The first bar to its right that is shorter than it

Once we have these boundaries, the width of the rectangle would be the distance between these two shorter bars (exclusive), and the area would be `height * width`.

A naive approach would check for each bar by scanning left and right, resulting in `O(n²)` time complexity. However, we can optimize this using a monotonic [stack](/problems/stack_intro). The stack maintains bars in increasing order of height. When we encounter a bar that is shorter than the top of the stack, we know that this shorter bar is the right boundary for all taller bars in the stack. Similarly, after popping taller bars, the remaining top of the stack (if any) is the left boundary for the current bar.

This elegant approach processes each bar exactly once (pushed and popped at most once), achieving `O(n)` time complexity while solving what initially seems like a complex geometric problem by breaking it down into finding boundaries for each potential rectangle height.

**Learn more about [Stack](/problems/stack_intro) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

The solution implements a monotonic [stack](/problems/stack_intro) pattern to find, for each bar, the nearest smaller elements on both left and right sides. Here's how the implementation works:

**Data Structures Used:**
- `stk`: A [stack](/problems/stack_intro) that maintains indices of bars in increasing order of their heights
- `left`: Array where `left[i]` stores the index of the nearest bar to the left with height less than `heights[i]`
- `right`: Array where `right[i]` stores the index of the nearest bar to the right with height less than or equal to `heights[i]` (the pop condition is `>=`; for equal heights, the last bar of the run still gets the full width)

**Initialization:**
- `left` is initialized with `-1` (representing no smaller bar on the left)
- `right` is initialized with `n` (representing no smaller bar on the right)

**Main Algorithm:**
```python
for i, h in enumerate(heights):
    while stk and heights[stk[-1]] >= h:
        right[stk[-1]] = i
        stk.pop()
    if stk:
        left[i] = stk[-1]
    stk.append(i)
```

For each bar at position `i` with height `h`:

1. **Update right boundaries**: While the [stack](/problems/stack_intro) is not empty and the top element represents a bar with height ≥ `h`, we've found the right boundary for that bar (which is `i`). We update `right[stk[-1]] = i` and pop it from the stack.

2. **Update left boundary**: After popping all bars taller than or equal to the current bar, if the [stack](/problems/stack_intro) still has elements, the top element is the nearest smaller bar on the left. We set `left[i] = stk[-1]`.

3. **Maintain [stack](/problems/stack_intro) invariant**: Push the current index `i` onto the stack to maintain the increasing height order.

**Calculate Maximum Area:**
```python
return max(h * (right[i] - left[i] - 1) for i, h in enumerate(heights))
```

For each bar at position `i` with height `h`:
- Width of rectangle = `right[i] - left[i] - 1` (distance between boundaries, exclusive)
- Area = `h * width`
- Return the maximum area among all bars

**Why This Works:**
The monotonic [stack](/problems/stack_intro) ensures that:
- When we pop elements due to a smaller height, we're finding their right boundary
- The element remaining at the top after popping is the left boundary for the current element
- Each element is pushed and popped exactly once, ensuring `O(n)` time complexity

The space complexity is `O(n)` for storing the `left`, `right` arrays and the [stack](/problems/stack_intro).

## Example Walkthrough

Let's walk through the algorithm with `heights = [2, 1, 5, 6, 2, 3]`.

**Initialization:**
- `n = 6`
- `left = [-1, -1, -1, -1, -1, -1]`
- `right = [6, 6, 6, 6, 6, 6]`
- `stk = []`

**Processing each bar:**

**i = 0, h = 2:**
- Stack is empty, no elements to pop
- Stack is empty, so `left[0]` remains `-1`
- Push 0 to stack: `stk = [0]`

**i = 1, h = 1:**
- Stack top: heights[0] = 2 ≥ 1, so pop and set `right[0] = 1`
- Stack becomes empty: `stk = []`
- Stack is empty, so `left[1]` remains `-1`
- Push 1 to stack: `stk = [1]`

**i = 2, h = 5:**
- Stack top: heights[1] = 1 < 5, no popping needed
- Stack has element 1, so `left[2] = 1`
- Push 2 to stack: `stk = [1, 2]`

**i = 3, h = 6:**
- Stack top: heights[2] = 5 < 6, no popping needed
- Stack has element 2, so `left[3] = 2`
- Push 3 to stack: `stk = [1, 2, 3]`

**i = 4, h = 2:**
- Stack top: heights[3] = 6 ≥ 2, pop and set `right[3] = 4`
- Stack top: heights[2] = 5 ≥ 2, pop and set `right[2] = 4`
- Stack top: heights[1] = 1 < 2, stop popping
- Stack has element 1, so `left[4] = 1`
- Push 4 to stack: `stk = [1, 4]`

**i = 5, h = 3:**
- Stack top: heights[4] = 2 < 3, no popping needed
- Stack has element 4, so `left[5] = 4`
- Push 5 to stack: `stk = [1, 4, 5]`

**Final arrays:**
- `left = [-1, -1, 1, 2, 1, 4]`
- `right = [1, 6, 4, 4, 6, 6]`

**Calculate areas for each bar:**
- Bar 0 (h=2): width = 1 - (-1) - 1 = 1, area = 2 × 1 = 2
- Bar 1 (h=1): width = 6 - (-1) - 1 = 6, area = 1 × 6 = 6
- Bar 2 (h=5): width = 4 - 1 - 1 = 2, area = 5 × 2 = 10
- Bar 3 (h=6): width = 4 - 2 - 1 = 1, area = 6 × 1 = 6
- Bar 4 (h=2): width = 6 - 1 - 1 = 4, area = 2 × 4 = 8
- Bar 5 (h=3): width = 6 - 4 - 1 = 1, area = 3 × 1 = 3

**Maximum area = 10** (from bar 2 with height 5, extending from index 2 to 3)

The rectangle with area 10 spans bars at indices 2 and 3, with height limited by bar 2's height of 5.

## Python Solution

```python
class Solution:
    def largestRectangleArea(self, heights: List[int]) -> int:
        """
        Find the largest rectangle area in a histogram.
      
        Uses a monotonic stack to find the nearest smaller elements on both sides
        for each bar, which determines the maximum width for rectangles with
        that bar's height.
      
        Args:
            heights: List of non-negative integers representing histogram bar heights
          
        Returns:
            The area of the largest rectangle in the histogram
        """
        n = len(heights)
      
        # Stack to maintain indices of bars in increasing height order
        stack = []
      
        # left[i] stores the index of the nearest smaller element to the left of i
        # Initialize to -1 (no smaller element on the left)
        left_boundaries = [-1] * n
      
        # right[i] stores the index of the nearest smaller element to the right of i
        # Initialize to n (no smaller element on the right)
        right_boundaries = [n] * n
      
        # Single pass to find both left and right boundaries
        for i, current_height in enumerate(heights):
            # Pop elements from stack that are >= current height
            # These elements have found their right boundary (current index)
            while stack and heights[stack[-1]] >= current_height:
                right_boundaries[stack[-1]] = i
                stack.pop()
          
            # The remaining top of stack (if exists) is the left boundary for current element
            if stack:
                left_boundaries[i] = stack[-1]
          
            # Add current index to stack
            stack.append(i)
      
        # Calculate maximum rectangle area
        # For each bar, the rectangle width is (right_boundary - left_boundary - 1)
        # and height is the bar's height
        max_area = max(
            height * (right_boundaries[i] - left_boundaries[i] - 1) 
            for i, height in enumerate(heights)
        )
      
        return max_area
```

## Time and Space Complexity

**Time Complexity:** `O(n)` where `n` is the length of the heights array.

- The algorithm iterates through the heights array once with the main for loop: `O(n)`
- While there is a nested while loop inside, each element is pushed onto the stack exactly once and popped from the stack at most once throughout the entire execution
- This means the total number of operations across all iterations is bounded by `2n` (each element is processed at most twice - once when pushed, once when popped)
- The final max calculation iterates through all elements once: `O(n)`
- Total time complexity: `O(n) + O(n) = O(n)`

**Space Complexity:** `O(n)` where `n` is the length of the heights array.

- The stack `stk` can contain at most `n` elements in the worst case (when heights are in increasing order): `O(n)`
- The `left` array stores `n` elements: `O(n)`
- The `right` array stores `n` elements: `O(n)`
- Total space complexity: `O(n) + O(n) + O(n) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Handling Equal Heights Inconsistently**

**The Pitfall:**
Equal heights need a consistent boundary convention. Some implementations pop while `heights[stack[-1]] >= current_height`; others pop only while `heights[stack[-1]] > current_height` and handle equal bars with a different left/right boundary convention. The important part is that the chosen convention stays consistent.

**Why It Matters:**
If the pop condition and boundary formulas do not match, equal-height bars can receive boundaries that are too narrow or too wide.

**Example:**
For consecutive equal bars such as `heights = [2, 2, 2]`, all bars represent the same height plateau. The implementation must ensure at least one of those bars can span the entire plateau when computing area.

**Solution:**
For this boundary-array formulation, use `>=` in the while loop condition:
```python
while stack and heights[stack[-1]] >= current_height:  # Correct: use >=
    right_boundaries[stack[-1]] = i
    stack.pop()
```

### 2. **Forgetting to Handle Remaining Stack Elements**

**The Pitfall:**
After the main loop, some indices might remain in the stack. These represent bars that never found a smaller bar to their right. If you don't handle this case, their `right_boundaries` remain at the initialized value.

**Why It Matters:**
The initialization `right_boundaries = [n] * n` actually handles this correctly by default. However, if someone modifies the algorithm or tries to optimize it differently, they might forget this crucial detail.

**Solution:**
The current implementation handles this correctly by initializing `right_boundaries` to `n`. Alternatively, you could explicitly process remaining stack elements:
```python
# After the main loop
while stack:
    right_boundaries[stack[-1]] = n  # Already set, but makes it explicit
    stack.pop()
```

### 3. **Off-by-One Error in Width Calculation**

**The Pitfall:**
Calculating the width as `right[i] - left[i]` instead of `right[i] - left[i] - 1`.

**Why It's Wrong:**
- `left[i]` is the index of the nearest smaller bar to the left
- `right[i]` is the index of the nearest smaller bar to the right
- The rectangle extends from `left[i] + 1` to `right[i] - 1` (inclusive)
- Therefore, width = `(right[i] - 1) - (left[i] + 1) + 1 = right[i] - left[i] - 1`

**Example:**
For `heights = [2, 1, 5, 6, 2, 3]` and the bar at index 2 (height 5):
- `left[2] = 1` (bar at index 1 has height 1 < 5)
- `right[2] = 4` (bar at index 4 has height 2 < 5)
- Correct width: `4 - 1 - 1 = 2` (bars at indices 2 and 3)
- Incorrect width: `4 - 1 = 3` (one bar too wide; the rectangle would have to include a bar shorter than 5)

**Solution:**
Always subtract 1 when calculating width:
```python
width = right_boundaries[i] - left_boundaries[i] - 1  # Correct
```

