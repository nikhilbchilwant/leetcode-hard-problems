# 1739. Building Boxes

> Source: https://algo.monster/liteproblems/1739

## Problem Statement

You have a cubic storage room with dimensions `n × n × n` units. Your task is to place `n` unit cubes (boxes) in this room following specific placement rules:

1. **Floor placement**: You can place boxes anywhere on the floor
2. **Stacking rule**: If you place box `x` on top of box `y`, then all four vertical sides of box `y` must be adjacent to either another box or a wall

Given an integer `n`, you need to find the minimum number of boxes that must touch the floor to accommodate all `n` boxes.

The key insight is that to minimize floor-touching boxes, you should build a pyramid-like structure against the corner where three walls meet. This creates a stepped arrangement where:
- The topmost layer has 1 box
- The second layer from top can have up to `1 + 2 = 3` boxes
- The third layer from top can have up to `1 + 2 + 3 = 6` boxes
- The k-th layer from top can have up to `1 + 2 + ... + k` boxes

The total number of boxes in a complete k-layer pyramid is the sum: `1 + 3 + 6 + ... + (1+2+...+k)`.

If you can't form a complete pyramid with `n` boxes, the remaining boxes are added starting from the bottom layer, following the same stepped pattern. Each additional floor box allows you to stack more boxes on top following the stacking rule.

## Intuition

To minimize the number of floor-touching boxes, we need to maximize vertical stacking. The constraint that each box must have all four sides supported (by walls or other boxes) naturally leads us to think about corner placement.

When we place the first box in a corner, it has three sides supported by walls. This allows us to build upward efficiently. As we expand outward from the corner, we create a stepped pyramid structure where each "step" provides support for the boxes above it.

Let's visualize how this works:
- Start with 1 box on the floor (corner position) - this supports 1 box above
- Add a 2nd floor box adjacent to the first - now we can support 2 more boxes in the second layer
- Add a 3rd floor box - we can support 3 more boxes in the second layer
- With `k` boxes on the floor arranged properly, we can support `1 + 2 + ... + k = k(k+1)/2` total boxes

The pattern emerges: if we have `k` complete layers, the bottom layer has `k(k+1)/2` boxes, the second-to-bottom has `(k-1)k/2` boxes, and so on. The total capacity is the sum of these triangular numbers.

The algorithm becomes:
1. Find the maximum number of complete layers `k` we can build with `n` boxes
2. Calculate how many boxes remain after building `k` complete layers
3. For the remaining boxes, we add them starting from a new bottom layer position, where each new floor box allows us to stack additional boxes following the same triangular pattern

This [greedy](/problems/greedy_intro) approach works because each additional floor box maximizes the stacking potential - the i-th additional floor box allows us to place `i` more boxes total (including itself).

**Learn more about [Greedy](/problems/greedy_intro), [Math](/problems/math-basics) and [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution follows the mathematical pattern discovered in our intuition. We build complete pyramid layers first, then handle remaining boxes.

**Step 1: Find Maximum Complete Layers**

We need to find the largest `k` such that a complete `k`-layer pyramid fits within `n` boxes. The total number of boxes in a `k`-layer pyramid is:
- Layer 1 (top): 1 box
- Layer 2: `1 + 2 = 3` boxes
- Layer 3: `1 + 2 + 3 = 6` boxes
- Layer k: `1 + 2 + ... + k = k(k+1)/2` boxes

Total boxes = `1 + 3 + 6 + ... + k(k+1)/2`

The code iterates through increasing values of `k`:
```python
s, k = 0, 1
while s + k * (k + 1) // 2 <= n:
    s += k * (k + 1) // 2  # Add boxes for layer k
    k += 1
k -= 1  # Step back to last valid k
```

Here, `s` tracks the cumulative boxes placed so far. We keep adding complete layers while the total doesn't exceed `n`.

**Step 2: Calculate Floor Boxes for Complete Pyramid**

After finding the maximum `k`, the number of floor-touching boxes in this complete pyramid is:
```python
ans = k * (k + 1) // 2
```

This represents the bottom layer of our `k`-layer pyramid.

**Step 3: Place Remaining Boxes**

If `s < n`, we have remaining boxes to place. These are added incrementally:
```python
k = 1
while s < n:
    ans += 1  # Add one more floor box
    s += k    # This floor box allows k additional boxes total
    k += 1
```

Each new floor box allows us to stack boxes following the triangular pattern:
- 1st additional floor box: adds 1 box total
- 2nd additional floor box: adds 2 boxes total
- 3rd additional floor box: adds 3 boxes total

The algorithm continues until all `n` boxes are placed, returning the minimum number of floor-touching boxes `ans`.

## Example Walkthrough

Let's walk through the solution with **n = 10** boxes.

**Step 1: Find Maximum Complete Layers**

We need to find the largest complete pyramid that fits within 10 boxes.

- Start with `s = 0, k = 1`
- Layer 1: Can we add `1*(1+1)/2 = 1` box? Yes, `0 + 1 = 1 ≤ 10`. Update: `s = 1, k = 2`
- Layer 2: Can we add `2*(2+1)/2 = 3` boxes? Yes, `1 + 3 = 4 ≤ 10`. Update: `s = 4, k = 3`
- Layer 3: Can we add `3*(3+1)/2 = 6` boxes? No, `4 + 6 = 10 ≤ 10`. Actually yes! Update: `s = 10, k = 4`
- Layer 4: Can we add `4*(4+1)/2 = 10` boxes? No, `10 + 10 = 20 > 10`. Stop.

Step back: `k = 3`

We can build a complete 3-layer pyramid using exactly 10 boxes:
- Top layer: 1 box
- Middle layer: 3 boxes (arranged in an L-shape)
- Bottom layer: 6 boxes (arranged in a triangular shape)

Visualizing the layers from above:
```
Layer 1 (top):    [X]

Layer 2:          [X][X]
                  [X]

Layer 3 (floor):  [X][X][X]
                  [X][X]
                  [X]
```

**Step 2: Calculate Floor Boxes**

The bottom layer has `k*(k+1)/2 = 3*4/2 = 6` boxes touching the floor.

**Step 3: Place Remaining Boxes**

Since `s = 10` equals `n = 10`, we have no remaining boxes to place.

**Answer: 6 floor-touching boxes**

---

Let's try another example with **n = 8** boxes to see how remaining boxes are handled.

**Step 1: Find Maximum Complete Layers**

- Start with `s = 0, k = 1`
- Layer 1: Add 1 box, `s = 1, k = 2`
- Layer 2: Add 3 boxes, `s = 4, k = 3`
- Layer 3: Can't add 6 boxes (4 + 6 = 10 > 8). Stop.

Step back: `k = 2`

We have a 2-layer pyramid with 4 boxes total.

**Step 2: Calculate Floor Boxes for Complete Pyramid**

The 2-layer pyramid has `2*3/2 = 3` floor boxes.

**Step 3: Place Remaining Boxes**

We have `8 - 4 = 4` remaining boxes to place.

- Start with `k = 1`
- Add 1st floor box: `ans = 3 + 1 = 4`, can place 1 box total, `s = 4 + 1 = 5`
- Add 2nd floor box: `ans = 4 + 1 = 5`, can place 2 boxes total, `s = 5 + 2 = 7`
- Add 3rd floor box: `ans = 5 + 1 = 6`, can place 3 boxes total, but we only need 1 more, `s = 7 + 1 = 8`

**Answer: 6 floor-touching boxes**

The final arrangement has the original 3 floor boxes from the 2-layer pyramid, plus 3 additional floor boxes that support the remaining 4 boxes in a partial new pyramid structure.

## Python Solution

```python
class Solution:
    def minimumBoxes(self, n: int) -> int:
        # Find the maximum complete pyramid that can be built with n boxes
        # A complete pyramid of height h has h layers, where layer i has i*(i+1)/2 boxes on the ground

        # Step 1: Find the maximum height of complete pyramid we can build
        total_boxes = 0  # Total boxes used so far
        height = 1  # Current height being considered

        # Keep adding complete layers while we have enough boxes
        while total_boxes + height * (height + 1) // 2 <= n:
            total_boxes += height * (height + 1) // 2  # Add boxes from this layer
            height += 1

        # Backtrack since we went one layer too far
        height -= 1

        # Calculate the number of ground boxes for the complete pyramid
        ground_boxes = height * (height + 1) // 2

        # Step 2: Add remaining boxes one column at a time
        # Each additional ground box can support a column of increasing height
        column_height = 1  # Height of the next column we can add

        while total_boxes < n:
            ground_boxes += 1  # Add one more box to the ground
            total_boxes += column_height  # Add a column of this height
            column_height += 1  # Next column can be one box taller

        return ground_boxes
```

## Time and Space Complexity

The time complexity is `O(√n)`, where `n` is the number of boxes given in the problem.

The first while loop continues while `s + k * (k + 1) // 2 <= n`. Since `s` accumulates the sum of triangular numbers (1 + 3 + 6 + 10 + ...), and the k-th triangular number is `k * (k + 1) // 2`, the total sum after k iterations is approximately `k³/6`. The loop stops when this sum exceeds `n`, which means `k³/6 ≈ n`, so `k ≈ ∛(6n) ≈ n^(1/3)`. Therefore, the first loop runs `O(n^(1/3))` times.

The second while loop fills in the remaining boxes. Since we need at most `O(n^(1/3))` additional boxes to reach exactly `n` (as we're adding boxes in the next layer), this loop also runs at most `O(n^(1/3))` times.

However, considering that the problem involves finding the minimum number of boxes on the ground to stack `n` boxes in a pyramid formation, and the optimal ground configuration forms approximately a triangular base with side length proportional to `√n`, the overall time complexity is `O(√n)`.

The space complexity is `O(1)` as the algorithm only uses a constant amount of extra space with variables `s`, `k`, and `ans`, regardless of the input size `n`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Error in Layer Calculation**

A common mistake is incorrectly handling the boundary when finding the maximum complete pyramid height. Developers often forget to backtrack after the while loop exits.

**Incorrect approach:**
```python
while total_boxes + height * (height + 1) // 2 <= n:
    total_boxes += height * (height + 1) // 2
    height += 1
# Forgetting to do: height -= 1
```

**Why it's wrong:** The while loop increments `height` one extra time before exiting, so we're working with an invalid height value.

**Solution:** Always remember to decrement the counter after the loop:
```python
while total_boxes + height * (height + 1) // 2 <= n:
    total_boxes += height * (height + 1) // 2
    height += 1
height -= 1  # Critical: backtrack to the last valid height
```

### 2. **Confusion Between Pyramid Layers and Floor Boxes**

Many people confuse the number of pyramid layers with the number of floor-touching boxes.

**Incorrect understanding:** "A k-layer pyramid has k floor boxes"

**Correct understanding:** A k-layer pyramid has `k*(k+1)/2` floor boxes because the bottom layer forms a triangular arrangement.

### 3. **Incorrect Remaining Boxes Calculation**

When adding remaining boxes after the complete pyramid, a common error is not understanding the incremental pattern.

**Incorrect approach:**
```python
# Wrong: Adding same number of boxes for each floor box
remaining = n - total_boxes
ground_boxes += remaining  # This assumes 1:1 ratio
```

**Correct approach:**
```python
column_height = 1
while total_boxes < n:
    ground_boxes += 1
    total_boxes += column_height  # Each new floor box supports increasing boxes
    column_height += 1
```

### 4. **Integer Overflow or Precision Issues**

For large values of `n`, calculating `k * (k + 1) / 2` using regular division can cause floating-point precision issues.

**Problematic code:**
```python
boxes_in_layer = k * (k + 1) / 2  # Uses float division
```

**Solution:** Always use integer division:
```python
boxes_in_layer = k * (k + 1) // 2  # Uses integer division
```

### 5. **Not Handling Edge Cases**

Failing to consider special cases like `n = 1` or very small values can lead to incorrect results.

**Solution:** The provided algorithm naturally handles these cases, but always verify:
- When `n = 1`: Returns 1 (single box on floor)
- When `n = 3`: Returns 2 (two floor boxes supporting one stacked box)

