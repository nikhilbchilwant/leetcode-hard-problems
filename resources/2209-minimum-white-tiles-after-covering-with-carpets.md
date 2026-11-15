# 2209. Minimum White Tiles After Covering With Carpets

> Source: https://algo.monster/liteproblems/2209

## Problem Statement

You have a floor represented as a binary string `floor` where:
- `floor[i] = '0'` means the tile at position `i` is black
- `floor[i] = '1'` means the tile at position `i` is white

You are given `numCarpets` black carpets, each with length `carpetLen` tiles. Your goal is to place these carpets on the floor to cover as many white tiles as possible. The carpets can overlap with each other.

The task is to find the minimum number of white tiles that remain visible after optimally placing all available carpets.

For example, if you have a floor `"10110"` with 2 carpets of length 2, you could:
- Place one carpet covering positions [1, 2] to hide the white tile at position 1
- Place another carpet covering positions [2, 3] to hide the white tile at position 3
- This leaves only the white tile at position 4 visible, giving a minimum of 1 visible white tile

## Intuition

When we look at this problem, we need to make decisions about where to place our carpets to minimize visible white tiles. At each position in the floor, we face a choice: should we place a carpet here or not?

The key insight is that this is an optimization problem with choices - a perfect candidate for [dynamic programming](/problems/dynamic_programming_intro). We can think of walking through the floor from left to right, and at each position, we need to decide what to do based on:
1. What tile we're currently looking at (black or white)
2. How many carpets we still have available
3. What happens if we make different choices

If we encounter a black tile, there's no need to cover it - we can simply move to the next position without using any carpet. This is optimal since we want to save our carpets for white tiles.

When we encounter a white tile, we have two options:
- **Skip it**: Leave this white tile uncovered (adding 1 to our count of visible white tiles) and move to the next position
- **Cover it**: Place a carpet starting at this position, which will cover the next `carpetLen` tiles, using up one carpet

The recursive nature becomes clear: after making a choice at position `i` with `j` carpets remaining, we need to solve a smaller subproblem - either at position `i+1` with the same number of carpets (if we skipped), or at position `i+carpetLen` with `j-1` carpets (if we placed a carpet).

To handle the base cases efficiently, we use a [prefix sum](/problems/subarray_sum) array. When we run out of carpets (`j = 0`), we can quickly calculate how many white tiles remain from the current position to the end using `s[n] - s[i]`, where `s` stores the cumulative count of white tiles.

The overlapping subproblems (we might reach the same position with the same number of carpets through different paths) make memoization essential to avoid redundant calculations, transforming this from an exponential to a polynomial time solution.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The solution implements a top-down [dynamic programming](/problems/dynamic_programming_intro) approach with memoization using Python's `@cache` decorator.

**Data Structures Used:**
1. **[Prefix Sum](/problems/subarray_sum) Array `s`**: We build an array where `s[i+1]` stores the count of white tiles from index 0 to i. This allows us to quickly calculate the number of white tiles in any range in O(1) time.
2. **Memoization Cache**: The `@cache` decorator automatically stores results of `dfs(i, j)` to avoid recomputing the same subproblems.

**Implementation Details:**

First, we build the [prefix sum](/problems/subarray_sum) array:
```python
s = [0] * (n + 1)
for i, c in enumerate(floor):
    s[i + 1] = s[i] + int(c == "1")
```
This means `s[i+1] - s[j]` gives us the count of white tiles from index `j` to `i` (inclusive).

The core recursive function `dfs(i, j)` represents the minimum number of uncovered white tiles starting from position `i` with `j` carpets available:

1. **Base Case - Out of bounds**: If `i >= n`, we've processed all tiles, return 0.

2. **Black Tile Case**: If `floor[i] == "0"`, we don't need to cover this tile, so we skip to the next position: `return dfs(i + 1, j)`.

3. **No Carpets Left**: If `j == 0`, we can't cover any more tiles. The remaining white tiles from position `i` to the end is `s[-1] - s[i]` (where `s[-1]` equals `s[n]`, the total count up to position n-1).

4. **White Tile with Carpets Available**: We have two choices:
   - **Don't use a carpet**: Skip this white tile (count it as visible) and move to the next position: `1 + dfs(i + 1, j)`
   - **Use a carpet**: Place a carpet starting at position `i`, covering positions `[i, i+carpetLen)`, and jump to position `i + carpetLen` with one less carpet: `dfs(i + carpetLen, j - 1)`
 
   We take the minimum of these two options: `min(1 + dfs(i + 1, j), dfs(i + carpetLen, j - 1))`

**Time Complexity**: O(n × numCarpets) since we have at most `n` positions and `numCarpets` different values for `j`, and each state is computed only once due to memoization.

**Space Complexity**: O(n × numCarpets) for the memoization cache plus O(n) for the [prefix sum](/problems/subarray_sum) array.

The final answer is obtained by calling `dfs(0, numCarpets)`, which gives the minimum number of white tiles visible when starting from position 0 with all carpets available.

## Example Walkthrough

Let's walk through a concrete example to understand how the solution works.

**Example:** `floor = "10110"`, `numCarpets = 2`, `carpetLen = 2`

**Step 1: Build Prefix Sum Array**
First, we create the prefix sum array `s` to count white tiles:
- floor: `"10110"`
- s: `[0, 1, 1, 2, 3, 3]`
  - s[0] = 0 (no tiles counted yet)
  - s[1] = 1 (floor[0]='1' is white)
  - s[2] = 1 (floor[1]='0' is black, no increment)
  - s[3] = 2 (floor[2]='1' is white)
  - s[4] = 3 (floor[3]='1' is white)
  - s[5] = 3 (floor[4]='0' is black)

**Step 2: Recursive Exploration with dfs(0, 2)**

Starting at position 0 with 2 carpets:

`dfs(0, 2)`: floor[0]='1' (white), we have carpets available
- Option 1: Skip this white tile → `1 + dfs(1, 2)`
  - `dfs(1, 2)`: floor[1]='0' (black) → skip to `dfs(2, 2)`
  - `dfs(2, 2)`: floor[2]='1' (white)
    - Option 1a: Skip → `1 + dfs(3, 2)`
      - `dfs(3, 2)`: floor[3]='1' (white)
        - Skip: `1 + dfs(4, 2)` = 1 + 0 = 1
        - Cover: `dfs(5, 1)` = 0 (out of bounds)
        - Min = 0
      - Result: 1 + 0 = 1
    - Option 1b: Cover positions [2,3] → `dfs(4, 1)`
      - `dfs(4, 1)`: floor[4]='0' (black) → `dfs(5, 1)` = 0
      - Result: 0
    - Min(1, 0) = 0
  - Total for Option 1: 1 + 0 = 1

- Option 2: Cover positions [0,1] with carpet → `dfs(2, 1)`
  - `dfs(2, 1)`: floor[2]='1' (white), 1 carpet left
    - Skip: `1 + dfs(3, 1)` 
      - `dfs(3, 1)`: floor[3]='1' (white)
        - Skip: `1 + dfs(4, 1)` = 1 + 0 = 1
        - Cover: `dfs(5, 0)` = 0
        - Min = 0
      - Result: 1 + 0 = 1
    - Cover: `dfs(4, 0)` (no carpets left)
      - Returns `s[5] - s[4] = 3 - 3 = 0` white tiles remain
    - Min(1, 0) = 0
  - Total for Option 2: 0

Final result: `min(1, 0) = 0`

However, let me recalculate more carefully. When we place carpet at position 0 covering [0,1], we hide the white tile at position 0. Then at position 2, if we place another carpet covering [2,3], we hide both white tiles at positions 2 and 3. This leaves 0 white tiles visible.

Actually, let's trace through with the correct logic:

`dfs(0, 2)`: floor[0]='1', carpets=2
- Skip: 1 + dfs(1, 2) = 1 + dfs(2, 2) = 1 + min(1 + dfs(3, 2), dfs(4, 1))
  - dfs(3, 2): floor[3]='1' → min(1 + dfs(4, 2), dfs(5, 1)) = min(1, 0) = 0
  - dfs(4, 1): floor[4]='0' → dfs(5, 1) = 0
  - So: 1 + min(1 + 0, 0) = 1 + 0 = 1
- Cover: dfs(2, 1) = min(1 + dfs(3, 1), dfs(4, 0))
  - dfs(3, 1): floor[3]='1' → min(1 + 0, 0) = 0
  - dfs(4, 0): s[5] - s[4] = 0
  - So: min(1 + 0, 0) = 0

**Result:** min(1, 0) = 0

The optimal strategy is to place one carpet at position 0 (covering tiles 0-1) and another at position 2 (covering tiles 2-3), leaving 0 white tiles visible.

## Python Solution

```python
class Solution:
    def minimumWhiteTiles(self, floor: str, numCarpets: int, carpetLen: int) -> int:
        from functools import cache
      
        # Helper function to count minimum uncovered white tiles
        # starting from position 'pos' with 'carpets_left' carpets remaining
        @cache
        def dp(pos: int, carpets_left: int) -> int:
            # Base case: if we've gone past the floor, no white tiles remain
            if pos >= floor_length:
                return 0
          
            # If current position is black tile, skip it (no need to cover)
            if floor[pos] == "0":
                return dp(pos + 1, carpets_left)
          
            # If no carpets left, count all remaining white tiles from current position
            if carpets_left == 0:
                return prefix_sum[-1] - prefix_sum[pos]
          
            # Two choices at a white tile:
            # Option 1: Don't place carpet here, count this white tile and continue
            leave_uncovered = 1 + dp(pos + 1, carpets_left)
          
            # Option 2: Place carpet starting at current position, skip carpetLen positions
            place_carpet = dp(pos + carpetLen, carpets_left - 1)
          
            return min(leave_uncovered, place_carpet)
      
        # Initialize variables
        floor_length = len(floor)
      
        # Build prefix sum array to quickly count white tiles in any range
        # prefix_sum[i] = number of white tiles in floor[0:i]
        prefix_sum = [0] * (floor_length + 1)
        for i, tile in enumerate(floor):
            prefix_sum[i + 1] = prefix_sum[i] + (1 if tile == "1" else 0)
      
        # Calculate minimum uncovered white tiles
        result = dp(0, numCarpets)
      
        # Clear cache to free memory
        dp.cache_clear()
      
        return result
```

## Time and Space Complexity

The time complexity is `O(n × m)`, where `n` is the length of the string `floor` and `m` is the value of `numCarpets`.

The recursive function `dfs(i, j)` uses memoization with `@cache`. The function has two parameters: `i` representing the current position in the floor (ranging from 0 to n-1), and `j` representing the number of carpets remaining (ranging from 0 to numCarpets). This creates at most `n × (numCarpets + 1)` unique states. Each state is computed only once due to memoization, and each computation takes `O(1)` time (comparing values, arithmetic operations, and looking up precomputed prefix sums). Therefore, the overall time complexity is `O(n × m)`.

The space complexity is `O(n × m)`. This comes from two main sources:
1. The memoization cache stores up to `n × (numCarpets + 1)` states, which is `O(n × m)`.
2. The recursion stack depth can go up to `O(n)` in the worst case when we don't place any carpets and traverse tile by tile.
3. The prefix sum array `s` uses `O(n)` space.

Since `O(n × m)` dominates, the overall space complexity is `O(n × m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Carpet Placement Boundaries**

One of the most common mistakes is not properly handling what happens when a carpet extends beyond the floor's end. When placing a carpet at position `i`, it covers positions `[i, i+carpetLen)`. If `i + carpetLen > floor_length`, the carpet extends past the floor boundary.

**Incorrect approach:**
```python
# This might cause issues if not handled properly
place_carpet = dp(pos + carpetLen, carpets_left - 1)
```

**Why it's problematic:** 
- If `pos + carpetLen > floor_length`, we jump beyond the floor, which is actually valid (the base case handles it)
- However, we might miss counting white tiles between `pos` and `floor_length` if we don't think about this carefully

**Correct approach:**
The solution handles this correctly by:
1. Allowing the jump to `pos + carpetLen` even if it exceeds floor_length
2. The base case `if pos >= floor_length: return 0` properly handles positions beyond the floor
3. When a carpet partially extends beyond the floor, it still covers all tiles from its starting position to the floor's end

### 2. **Off-by-One Errors in Prefix Sum Calculation**

**Common mistake:**
```python
# Incorrect: might mix up indices
prefix_sum[i] = prefix_sum[i-1] + (1 if floor[i] == "1" else 0)
```

**Correct approach:**
```python
# Maintain clear indexing: prefix_sum[i+1] represents sum up to and including floor[i]
prefix_sum[i + 1] = prefix_sum[i] + (1 if floor[i] == "1" else 0)
```

This ensures:
- `prefix_sum[0] = 0` (no tiles counted)
- `prefix_sum[i+1]` = count of white tiles in `floor[0:i+1]`
- Range sum from index `a` to `b` (inclusive) = `prefix_sum[b+1] - prefix_sum[a]`

### 3. **Forgetting to Skip Black Tiles**

**Incorrect approach:**
```python
def dp(pos, carpets_left):
    if pos >= floor_length:
        return 0
    if carpets_left == 0:
        return prefix_sum[-1] - prefix_sum[pos]
  
    # Missing the check for black tiles!
    leave_uncovered = 1 + dp(pos + 1, carpets_left)  # Wrong: counts black tiles as 1
    place_carpet = dp(pos + carpetLen, carpets_left - 1)
    return min(leave_uncovered, place_carpet)
```

**Correct approach:**
Always check if the current tile is black before making decisions:
```python
if floor[pos] == "0":
    return dp(pos + 1, carpets_left)  # Skip black tiles without counting
```

### 4. **Memory Limit Issues with Large Inputs**

For very large inputs, the cache might consume excessive memory. While the provided solution includes `dp.cache_clear()`, forgetting this step in repeated test cases could cause memory issues.

**Best practice:**
```python
# Always clear cache after getting the result
result = dp(0, numCarpets)
dp.cache_clear()  # Important for multiple test cases
return result
```

### 5. **Suboptimal Choice Logic**

**Common mistake:** Placing carpets greedily at every white tile:
```python
# Wrong: Always try to place carpet at first white tile encountered
if floor[pos] == "1" and carpets_left > 0:
    return dp(pos + carpetLen, carpets_left - 1)
```

**Correct approach:** Consider both options (place or skip) and choose the minimum:
```python
leave_uncovered = 1 + dp(pos + 1, carpets_left)
place_carpet = dp(pos + carpetLen, carpets_left - 1)
return min(leave_uncovered, place_carpet)
```

This ensures we find the globally optimal solution rather than a locally optimal one.

