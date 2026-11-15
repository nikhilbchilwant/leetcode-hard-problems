# 1691. Maximum Height by Stacking Cuboids

> Source: https://algo.monster/liteproblems/1691

## Problem Statement

You are given `n` cuboids (3D rectangular boxes), where each cuboid has three dimensions: width, length, and height. The dimensions of the `i-th` cuboid are given as `cuboids[i] = [width_i, length_i, height_i]`.

Your task is to select a subset of these cuboids and stack them on top of each other to achieve the maximum possible height.

The stacking rules are:
- Cuboid `i` can be placed on top of cuboid `j` if and only if:
  - `width_i ≤ width_j`
  - `length_i ≤ length_j`  
  - `height_i ≤ height_j`

- You can rotate any cuboid before placing it. This means you can rearrange its three dimensions in any order (e.g., you can make any dimension the "height", "width", or "length").

The goal is to find the maximum total height you can achieve by stacking a valid subset of cuboids.

For example, if you have a cuboid with dimensions `[1, 2, 3]`, you can rotate it to have dimensions like `[2, 3, 1]` or `[3, 1, 2]`, etc., choosing whichever orientation helps you build the tallest stack.

## Intuition

The key observation is that we're allowed to rotate cuboids, which gives us flexibility in choosing which dimension becomes the height. Since we want to maximize the total height, we should orient each cuboid to use its largest dimension as the height whenever possible.

Think about it this way: if we have a cuboid with dimensions `[1, 2, 3]`, and we can choose any orientation, why not put `3` as the height to maximize our contribution? But there's a constraint - we need to ensure valid stacking.

For any two cuboids that can be stacked, if we sort the dimensions of each cuboid individually (so `[1, 2, 3]` stays `[1, 2, 3]` but `[3, 1, 2]` becomes `[1, 2, 3]`), we can always use the largest dimension as height while maintaining the stacking validity. This is because if cuboid A can be placed on cuboid B in any orientation, then cuboid A with sorted dimensions can be placed on cuboid B with sorted dimensions, using the largest values as heights.

Once we've sorted each cuboid's dimensions, we need to determine the optimal stacking order. If we sort all cuboids by their dimensions (comparing them lexicographically), we create a natural ordering where smaller cuboids come before larger ones. This ordering ensures that if cuboid `i` comes before cuboid `j` in our sorted list, and `i` can be placed on `j`, we'll consider this possibility.

Now the problem transforms into finding the best subset and their stacking order. This resembles the longest increasing subsequence problem, but instead of finding the longest sequence, we're finding the sequence with maximum height sum. We can use [dynamic programming](/problems/dynamic_programming_intro) where `f[i]` represents the maximum height achievable with cuboid `i` at the bottom. For each cuboid `i`, we check all previous cuboids `j` that could potentially go on top of it (where `j`'s width and length are both less than or equal to `i`'s), and take the best option.

The rotation flexibility combined with [sorting](/problems/sorting_summary) essentially converts a complex 3D stacking problem into a more manageable [dynamic programming](/problems/dynamic_programming_intro) problem on sorted sequences.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows these key steps:

**Step 1: Sort each cuboid's dimensions**
```python
for c in cuboids:
    c.sort()
```
We sort each cuboid's three dimensions in ascending order. After this operation, each cuboid has its dimensions arranged as `[smallest, middle, largest]`. This ensures that we'll use the largest dimension as the height when stacking.

**Step 2: Sort all cuboids**
```python
cuboids.sort()
```
We sort the entire array of cuboids lexicographically. This creates an ordering where smaller cuboids appear before larger ones. For example, `[1, 2, 3]` comes before `[1, 2, 4]` and `[2, 3, 4]`. This ordering is crucial because it guarantees that if cuboid `j` can be placed on cuboid `i`, then `j` appears before `i` in the sorted array.

**Step 3: [Dynamic Programming](/problems/dynamic_programming_intro)**
```python
n = len(cuboids)
f = [0] * n
```
We create a DP array `f` where `f[i]` represents the maximum height achievable when cuboid `i` is at the bottom of the stack.

**Step 4: Fill the DP table**
```python
for i in range(n):
    for j in range(i):
        if cuboids[j][1] <= cuboids[i][1] and cuboids[j][2] <= cuboids[i][2]:
            f[i] = max(f[i], f[j])
    f[i] += cuboids[i][2]
```

For each cuboid `i`, we:
- Check all previous cuboids `j` (where `j < i`)
- Verify if cuboid `j` can be placed on top of cuboid `i` by checking:
  - `cuboids[j][1] <= cuboids[i][1]` (middle dimension comparison)
  - `cuboids[j][2] <= cuboids[i][2]` (largest dimension comparison)
  - We don't need to check `cuboids[j][0] <= cuboids[i][0]` because the lexicographic [sorting](/problems/sorting_summary) already ensures this condition
- If valid, update `f[i]` to be the maximum of its current value and `f[j]` (the best height achievable with `j` on top)
- Finally, add cuboid `i`'s height (`cuboids[i][2]`) to `f[i]`

The state transition equation is:
`f[i] = max{f[j]} + height[i]` for all valid `j < i`

**Step 5: Return the result**
```python
return max(f)
```
The answer is the maximum value in the DP array, representing the tallest possible stack among all possible configurations.

The time complexity is `O(n²)` for the DP iterations plus `O(n log n)` for [sorting](/problems/sorting_summary), resulting in `O(n²)` overall. The space complexity is `O(n)` for the DP array.

## Example Walkthrough

Let's walk through the solution with a concrete example. Suppose we have the following cuboids:
```
cuboids = [[2,3,1], [1,1,3], [2,2,1]]
```

**Step 1: Sort each cuboid's dimensions**

After sorting each cuboid individually:
- `[2,3,1]` becomes `[1,2,3]`
- `[1,1,3]` becomes `[1,1,3]`
- `[2,2,1]` becomes `[1,2,2]`

Now we have: `cuboids = [[1,2,3], [1,1,3], [1,2,2]]`

Each cuboid now has its largest dimension as the last element, which will be used as height.

**Step 2: Sort all cuboids lexicographically**

Sorting the entire array:
- `[1,1,3]` comes first (smallest second dimension)
- `[1,2,2]` comes second
- `[1,2,3]` comes third

After sorting: `cuboids = [[1,1,3], [1,2,2], [1,2,3]]`

**Step 3 & 4: Dynamic Programming**

Initialize DP array: `f = [0, 0, 0]`

Process each cuboid:

**i = 0:** cuboid = `[1,1,3]`
- No previous cuboids to check
- `f[0] = 0 + 3 = 3`
- DP array: `f = [3, 0, 0]`

**i = 1:** cuboid = `[1,2,2]`
- Check j = 0: cuboid `[1,1,3]`
  - Can `[1,1,3]` be placed on `[1,2,2]`?
  - Check: `1 ≤ 2` (middle) ✓ and `3 ≤ 2` (height) ✗
  - Cannot stack, so skip
- `f[1] = 0 + 2 = 2`
- DP array: `f = [3, 2, 0]`

**i = 2:** cuboid = `[1,2,3]`
- Check j = 0: cuboid `[1,1,3]`
  - Can `[1,1,3]` be placed on `[1,2,3]`?
  - Check: `1 ≤ 2` (middle) ✓ and `3 ≤ 3` (height) ✓
  - Valid! Update: `f[2] = max(0, f[0]) = max(0, 3) = 3`
- Check j = 1: cuboid `[1,2,2]`
  - Can `[1,2,2]` be placed on `[1,2,3]`?
  - Check: `2 ≤ 2` (middle) ✓ and `2 ≤ 3` (height) ✓
  - Valid! Update: `f[2] = max(3, f[1]) = max(3, 2) = 3`
- Add current height: `f[2] = 3 + 3 = 6`
- DP array: `f = [3, 2, 6]`

**Step 5: Return maximum**

The maximum value in `f` is 6, which represents stacking:
- Bottom: `[1,2,3]` (oriented from original `[2,3,1]`)
- Top: `[1,1,3]` (oriented from original `[1,1,3]`)
- Total height: 3 + 3 = 6

This is indeed the maximum possible height for this set of cuboids.

## Python Solution

```python
class Solution:
    def maxHeight(self, cuboids: List[List[int]]) -> int:
        # Sort each cuboid's dimensions in ascending order
        # This ensures we can use any dimension as height while maintaining validity
        for cuboid in cuboids:
            cuboid.sort()
      
        # Sort all cuboids lexicographically
        # This helps establish a potential stacking order
        cuboids.sort()
      
        num_cuboids = len(cuboids)
      
        # dp[i] represents the maximum height achievable with cuboid i at the top
        dp = [0] * num_cuboids
      
        # Process each cuboid as a potential top of the stack
        for i in range(num_cuboids):
            # Check all previous cuboids as potential bases
            for j in range(i):
                # A cuboid j can be placed below cuboid i if:
                # - width of j <= width of i (cuboids[j][1] <= cuboids[i][1])
                # - depth of j <= depth of i (cuboids[j][2] <= cuboids[i][2])
                # Note: length is already satisfied due to sorting
                if (cuboids[j][1] <= cuboids[i][1] and 
                    cuboids[j][2] <= cuboids[i][2]):
                    # Update maximum height if stacking on cuboid j is better
                    dp[i] = max(dp[i], dp[j])
          
            # Add current cuboid's height (largest dimension after sorting)
            dp[i] += cuboids[i][2]
      
        # Return the maximum height among all possible configurations
        return max(dp)
```

## Time and Space Complexity

**Time Complexity:** `O(n^2)`

The time complexity is dominated by the following operations:
- Sorting each cuboid's dimensions: `O(n)` iterations, each taking `O(1)` time since each cuboid has only 3 dimensions
- Sorting all cuboids: `O(n log n)`
- Nested loops for dynamic programming: `O(n^2)`, where the outer loop runs `n` times and the inner loop runs up to `i` times for each iteration

The overall time complexity is `O(n) + O(n log n) + O(n^2) = O(n^2)`, where `n` is the number of cuboids.

**Space Complexity:** `O(n)`

The space complexity comes from:
- The array `f` of size `n` used for dynamic programming: `O(n)`
- The in-place sorting doesn't require additional space beyond constant variables

Therefore, the total space complexity is `O(n)`, where `n` is the number of cuboids.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrect Dimension Comparison After Sorting

**The Problem:**
A common mistake is assuming that after sorting cuboids lexicographically, we only need to check if one cuboid can be placed on another by comparing their sorted dimensions directly. However, the sorting alone doesn't guarantee all three dimensions satisfy the stacking constraints.

**Incorrect Approach:**
```python
# WRONG: Only checking if cuboids[j] <= cuboids[i] lexicographically
for i in range(n):
    for j in range(i):
        # This is insufficient!
        if cuboids[j] <= cuboids[i]:  
            f[i] = max(f[i], f[j])
    f[i] += cuboids[i][2]
```

**Why It Fails:**
Consider cuboids `[1, 3, 5]` and `[2, 2, 4]`. After sorting, we have:
- Cuboid A: `[1, 3, 5]`
- Cuboid B: `[2, 2, 4]`

Lexicographically, A < B (since 1 < 2), but B cannot be placed on A because B's width (2) > A's width (1).

**The Solution:**
Explicitly check all relevant dimensions after the first one:
```python
for i in range(n):
    for j in range(i):
        # Must check all dimensions explicitly
        if (cuboids[j][0] <= cuboids[i][0] and 
            cuboids[j][1] <= cuboids[i][1] and 
            cuboids[j][2] <= cuboids[i][2]):
            f[i] = max(f[i], f[j])
    f[i] += cuboids[i][2]
```

### Pitfall 2: Forgetting to Consider Each Cuboid as a Potential Base

**The Problem:**
Some might think they need to find a single continuous chain of cuboids and forget that any cuboid can serve as the base of the stack, not just the "smallest" one.

**Incorrect Approach:**
```python
# WRONG: Starting from a fixed base
def maxHeight(self, cuboids):
    for c in cuboids:
        c.sort()
    cuboids.sort()
  
    # Assuming we must start from cuboids[0]
    height = cuboids[0][2]
    for i in range(1, len(cuboids)):
        if can_stack(cuboids[i-1], cuboids[i]):
            height += cuboids[i][2]
    return height
```

**Why It Fails:**
This approach forces us to use consecutive cuboids and start from a specific base, missing optimal solutions where we skip certain cuboids or start from a different base.

**The Solution:**
The DP approach correctly considers each cuboid as a potential base and finds the maximum among all possibilities:
```python
# Correct: Consider all possible configurations
return max(dp)  # Returns the best result among all possible bases
```

### Pitfall 3: Not Sorting Individual Cuboid Dimensions First

**The Problem:**
Forgetting to sort each cuboid's dimensions before processing leads to suboptimal solutions because we're not maximizing the height contribution of each cuboid.

**Incorrect Approach:**
```python
# WRONG: Not sorting individual cuboid dimensions
def maxHeight(self, cuboids):
    # Missing: for c in cuboids: c.sort()
    cuboids.sort()  # Only sorting the array of cuboids
    # ... rest of the DP logic
```

**Why It Fails:**
Without sorting individual dimensions, a cuboid `[5, 1, 2]` would use 2 as its height instead of the optimal 5 (after rotation to `[1, 2, 5]`).

**The Solution:**
Always sort each cuboid's dimensions first to ensure the largest dimension becomes the height:
```python
# Correct: Sort each cuboid's dimensions first
for cuboid in cuboids:
    cuboid.sort()  # Now each cuboid is [smallest, middle, largest]
```

