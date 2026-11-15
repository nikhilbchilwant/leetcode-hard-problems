# 1340. Jump Game V

> Source: https://algo.monster/liteproblems/1340

## Problem Statement

You are given an array of integers `arr` and an integer `d` representing the maximum jump distance. You need to find the maximum number of indices you can visit by jumping through the array.

From any index `i`, you can jump to another index `j` if the following conditions are met:

1. **Distance constraint**: The jump distance must be within `d`. This means:
   - You can jump forward to `i + x` where `0 < x ≤ d` and `i + x < arr.length`
   - You can jump backward to `i - x` where `0 < x ≤ d` and `i - x ≥ 0`

2. **Height constraint**: You can only jump from a higher value to a lower value. Specifically:
   - `arr[i] > arr[j]` (the destination must have a smaller value)
   - `arr[i] > arr[k]` for all indices `k` between `i` and `j` (all intermediate values must also be smaller than the starting value)

You can start from any index in the array and begin jumping. Each index can be visited at most once in a single path. The goal is to find the maximum number of indices you can visit in a single sequence of jumps.

For example, if `arr = [6,4,14,6,8,13,9,7,10,6,12]` and `d = 2`:
- From index 2 (value 14), you could jump to index 1 (value 4) or index 3 (value 6) or index 4 (value 8), since they are within distance 2 and all have smaller values than 14
- From index 10 (value 12), you could jump to index 9 (value 6) since it's within distance 2 and has a smaller value

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization to explore all possible jumping paths from each starting position and returns the maximum length path found.

## Intuition

The key insight is that this problem has an optimal substructure property - if we know the maximum number of jumps we can make starting from each position, we can build up the solution for other positions.

Think of it this way: when standing at index `i`, we need to find the longest path starting from this position. This path consists of:
1. The current index (count = 1)
2. Plus the longest path we can achieve from any valid jump destination

Since we can only jump to lower values (with all intermediate values also being lower), we're essentially creating a directed acyclic graph (DAG) where edges point from higher values to reachable lower values. This prevents cycles and makes the problem suitable for [dynamic programming](/problems/dynamic_programming_intro).

The recursive nature becomes clear: `maxJumps(i) = 1 + max(maxJumps(j))` for all valid jump destinations `j` from `i`.

We use memoization (`@cache`) because multiple starting positions might jump to the same intermediate position. For instance, if both index 2 and index 5 can jump to index 3, we don't want to recalculate the maximum jumps from index 3 twice.

The algorithm explores two directions from each position:
- **Left direction**: Check positions `i-1, i-2, ..., max(0, i-d)` and stop when we hit a value that's not smaller than `arr[i]` (violating the height constraint)
- **Right direction**: Check positions `i+1, i+2, ..., min(n-1, i+d)` and stop when we hit a value that's not smaller than `arr[i]`

The early stopping is crucial - once we encounter a value that's greater than or equal to `arr[i]`, we know we can't jump over it to reach further positions, so we break out of that direction.

Finally, since we can start from any position, we try all possible starting points and return the maximum path length found.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The solution implements a top-down [dynamic programming](/problems/dynamic_programming_intro) approach with memoization using depth-first search (DFS).

**Core Implementation:**

1. **DFS Function with Memoization**: The `dfs(i)` function calculates the maximum number of indices that can be visited starting from index `i`. The `@cache` decorator automatically memoizes the results to avoid redundant calculations.

2. **Base Case**: Each position can visit at least itself, so we initialize `ans = 1`.

3. **Exploring Left Jumps**:
   ```python
   for j in range(i - 1, -1, -1):
       if i - j > d or arr[j] >= arr[i]:
           break
       ans = max(ans, 1 + dfs(j))
   ```
   - Iterate backwards from `i-1` to `0`
   - Check two stopping conditions:
     - Distance exceeded: `i - j > d`
     - Height constraint violated: `arr[j] >= arr[i]`
   - If both conditions are satisfied, recursively calculate `dfs(j)` and update the maximum

4. **Exploring Right Jumps**:
   ```python
   for j in range(i + 1, n):
       if j - i > d or arr[j] >= arr[i]:
           break
       ans = max(ans, 1 + dfs(j))
   ```
   - Iterate forwards from `i+1` to `n-1`
   - Apply the same stopping conditions for distance and height
   - Update the maximum with `1 + dfs(j)` if valid

5. **Early Termination Optimization**: The loops break immediately when encountering a value greater than or equal to `arr[i]`. This is valid because all subsequent positions in that direction would be unreachable (blocked by the taller or equal element).

6. **Global Maximum**: Since we can start from any index, we compute `dfs(i)` for all positions and return the maximum:
   ```python
   return max(dfs(i) for i in range(n))
   ```

**Time Complexity**: `O(n * d)` where `n` is the array length. Each position is computed once (due to memoization), and for each position, we check at most `2d` other positions.

**Space Complexity**: `O(n)` for the memoization cache storing results for each index, plus `O(n)` for the recursion stack in the worst case.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example**: `arr = [7, 6, 5, 4, 3, 2]`, `d = 2`

Let's trace through the DFS calls starting from different positions:

**Starting from index 0 (value 7):**
- From index 0, we can jump within distance 2
- Check left: No positions to the left
- Check right:
  - Index 1 (value 6): Valid! 6 < 7, distance = 1 ≤ 2
  - Index 2 (value 5): Valid! 5 < 7, distance = 2 ≤ 2
- Now we recursively calculate:
  - `dfs(1)`: From index 1 (value 6)
    - Can jump to index 2 (5) or index 3 (4)
    - `dfs(2)` returns 3 (can visit 2→3→4)
    - `dfs(3)` returns 2 (can visit 3→4)
    - So `dfs(1) = 1 + max(3, 2) = 4`
  - `dfs(2)`: From index 2 (value 5)
    - Can jump to index 3 (4) or index 4 (3)
    - `dfs(3)` returns 2 (already computed)
    - `dfs(4)` returns 1 (can only visit 4→5)
    - So `dfs(2) = 1 + max(2, 1) = 3`
- Therefore, `dfs(0) = 1 + max(4, 3) = 5`

**Starting from index 3 (value 4):**
- Check left:
  - Index 2 (value 5): Can't jump (5 ≥ 4)
  - Stop checking left
- Check right:
  - Index 4 (value 3): Valid! 3 < 4
  - Index 5 (value 2): Valid! 2 < 4
- Calculate:
  - `dfs(4) = 2` (can visit 4→5)
  - `dfs(5) = 1` (no valid jumps)
- So `dfs(3) = 1 + max(2, 1) = 3`

The memoization ensures we don't recalculate `dfs(3)`, `dfs(4)`, etc. when they're needed multiple times.

**Final Result**: The maximum path is 5, achieved by starting at index 0 and following the path: 0→1→2→3→4 (visiting indices with values 7→6→5→4→3).

This example demonstrates:
- How the algorithm explores both directions from each position
- The importance of the height constraint (can only jump to lower values)
- How memoization prevents redundant calculations
- The recursive nature of building up solutions from subproblems

## Python Solution

```python
from typing import List
from functools import cache

class Solution:
    def maxJumps(self, arr: List[int], d: int) -> int:
        """
        Find the maximum number of indices you can visit starting from any index.
        You can jump from index i to index j if:
        - i - d <= j <= i + d (j != i)
        - arr[k] < arr[i] for all k between i and j (exclusive)
      
        Args:
            arr: List of integers representing heights
            d: Maximum jump distance
          
        Returns:
            Maximum number of indices that can be visited
        """
        @cache
        def dfs(index: int) -> int:
            """
            Perform depth-first search to find max jumps from given index.
          
            Args:
                index: Current position in the array
              
            Returns:
                Maximum number of indices reachable from current index
            """
            # Start with 1 (counting current index)
            max_jumps = 1
          
            # Try jumping to the left (indices before current)
            for next_index in range(index - 1, -1, -1):
                # Stop if distance exceeds d or we hit a taller/equal element
                if index - next_index > d or arr[next_index] >= arr[index]:
                    break
                # Update max jumps if jumping to next_index gives better result
                max_jumps = max(max_jumps, 1 + dfs(next_index))
          
            # Try jumping to the right (indices after current)
            for next_index in range(index + 1, array_length):
                # Stop if distance exceeds d or we hit a taller/equal element
                if next_index - index > d or arr[next_index] >= arr[index]:
                    break
                # Update max jumps if jumping to next_index gives better result
                max_jumps = max(max_jumps, 1 + dfs(next_index))
          
            return max_jumps
      
        # Store array length for efficiency
        array_length = len(arr)
      
        # Find maximum jumps starting from each index
        return max(dfs(i) for i in range(array_length))
```

## Time and Space Complexity

**Time Complexity:** `O(n * d)`

The analysis breaks down as follows:
- The `dfs` function is called for each index in the array, which gives us `n` unique calls due to memoization with `@cache`
- For each `dfs(i)` call, we explore up to `d` positions to the left and up to `d` positions to the right
- In the worst case, we check `O(d)` positions in each direction before either hitting the distance limit or finding a taller/equal element
- Since each state is computed only once due to caching, the total work is `O(n)` states × `O(d)` work per state = `O(n * d)`

**Space Complexity:** `O(n)`

The space usage consists of:
- The recursion stack depth can go up to `O(n)` in the worst case (when we can jump from index 0 to 1 to 2... to n-1 in a chain)
- The `@cache` decorator stores up to `n` different results, one for each index, using `O(n)` space
- Total space complexity is `O(n)` for the cache plus `O(n)` for the recursion stack = `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Height Constraint Check Between Indices**

One of the most common mistakes is not properly checking that ALL intermediate values between the starting and destination indices are smaller than the starting value. Many developers incorrectly only check the destination value:

**Incorrect Implementation:**
```python
# WRONG: Only checks destination, ignores intermediate values
for j in range(i - 1, max(i - d - 1, -1), -1):
    if arr[j] < arr[i]:  # Missing check for intermediate values!
        ans = max(ans, 1 + dfs(j))
```

**Why it's wrong:** According to the problem, you can only jump from `i` to `j` if `arr[k] < arr[i]` for ALL indices `k` between `i` and `j`. The above code would incorrectly allow jumping over taller intermediate elements.

**Correct Approach:**
```python
# CORRECT: Breaks immediately when hitting a taller/equal element
for j in range(i - 1, -1, -1):
    if i - j > d or arr[j] >= arr[i]:
        break  # Can't jump past this point
    ans = max(ans, 1 + dfs(j))
```

### 2. **Off-by-One Errors in Distance Calculation**

Another frequent mistake is incorrectly calculating the jump distance or using wrong comparison operators:

**Incorrect Examples:**
```python
# WRONG: Uses >= instead of >
if i - j >= d or arr[j] >= arr[i]:  # Should be i - j > d
    break

# WRONG: Includes current index in distance calculation
for j in range(i - d, i):  # Should ensure j != i and proper bounds
```

**Correct Implementation:**
The distance between indices should be strictly compared: `i - j > d` (not `>=`), and we must ensure we never jump to the same index.

### 3. **Forgetting to Handle Edge Cases**

**Common Edge Case Mistakes:**
- Not handling single-element arrays properly
- Incorrectly setting loop boundaries that could cause index out of bounds
- Not considering that you can start from ANY index (not just index 0)

**Prevention:**
Always validate loop boundaries:
```python
# Safe boundary checking
for j in range(i - 1, -1, -1):  # Automatically stops at 0
    if i - j > d:  # Distance check prevents going too far
        break
```

### 4. **Missing or Incorrect Memoization**

Without proper memoization, the solution becomes exponentially slow:

**Incorrect:**
```python
def maxJumps(self, arr, d):
    def dfs(i):  # No memoization - will cause TLE!
        # ... rest of logic
    return max(dfs(i) for i in range(len(arr)))
```

**Correct:**
Use `@cache` decorator or manual memoization to ensure each state is computed only once.

### 5. **Breaking Too Early in the Loop**

Some might think they can optimize by breaking when finding any valid jump:

**Incorrect:**
```python
for j in range(i - 1, -1, -1):
    if i - j > d or arr[j] >= arr[i]:
        break
    ans = max(ans, 1 + dfs(j))
    break  # WRONG: Stops after first valid jump, missing better options
```

**Why it's wrong:** You need to explore ALL valid jumps from each position to find the maximum path length. Breaking after the first valid jump would miss potentially longer paths.

