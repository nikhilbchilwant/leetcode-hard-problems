# 546. Remove Boxes

> Source: https://algo.monster/liteproblems/546

## Problem Statement

You have a row of boxes, where each box has a color represented by a positive number. Your goal is to remove all boxes and maximize your total score.

The game works as follows:
- In each round, you can select a group of consecutive boxes that all have the same color
- When you remove `k` consecutive boxes of the same color, you earn `k * k` points
- After removing boxes, the remaining boxes shift together (maintaining their order)
- You continue removing boxes until none are left

For example, if you have boxes `[1, 3, 2, 2, 2, 3, 4, 3, 1]`:
- You could remove the three consecutive 2's in the middle to get `3 * 3 = 9` points
- Or you might remove boxes strategically in a different order to get a higher total score

The challenge is to find the optimal sequence of removals that maximizes your total points. Since removing boxes causes the remaining boxes to come together, you might want to remove certain boxes first to create larger groups of the same color later.

Your task is to determine the maximum possible points you can achieve by removing all the boxes.

## Intuition

The key insight is that when we remove boxes, we might want to delay removing certain boxes to combine them with others of the same color later. This creates a complex decision tree where the order of removal significantly impacts the final score.

Consider a simple example: `[1, 2, 1]`. If we remove the middle `2` first, we get `1 * 1 = 1` point, then the two `1`s come together and we can remove them for `2 * 2 = 4` points, totaling 5 points. But if we removed each box individually, we'd only get `1 + 1 + 1 = 3` points.

This suggests we need [dynamic programming](/problems/dynamic_programming_intro) with [memoization](/problems/memoization_intro). But what state should we track? The naive approach of tracking which boxes have been removed would be too complex.

The clever observation is that we can define our subproblem as: "What's the maximum score for removing boxes from index `i` to `j`, where we already have `k` boxes **to the right of position `j`** (outside the range `[i, j]`) that share the same color as `boxes[j]` and will be combined with it later?"

Why track boxes outside the current range? When we remove boxes in the middle of the array, previously separated boxes come together. For example, if we have `[1, 2, 2, 1]` and remove the two middle `2`s first, the two `1`s become adjacent. The parameter `k` lets us track that the `1` at position 3 is "waiting" to be combined with the `1` at position 0, even though position 3 is outside our current working range.

![State definition showing where k boxes are located](/liteProblems/remove-boxes-state-definition.svg)

For the recursive case, we have two main strategies:
1. Remove `boxes[j]` along with the `k` boxes of the same color, getting `(k + 1) * (k + 1)` points
2. Look for another box with the same color as `boxes[j]` somewhere in the range `[i, j-1]`, remove the boxes between them first, then combine them for a potentially higher score

The second strategy is powerful because it allows us to "save" boxes of the same color to remove them together later, potentially earning more points than removing them separately.

**Learn more about [Memoization](/problems/memoization_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with [memoization](/problems/memoization_intro) to solve this problem. Let's break down the implementation:

**State Definition:**
- `dfs(i, j, k)` returns the maximum points we can get from removing boxes in the range `[i, j]`, where we have `k` additional boxes **to the right of position `j`** (outside the current range) that share the same color as `boxes[j]` and will eventually be combined with it when `boxes[j]` is removed.

**Base Case:**
- If `i > j`, there are no boxes to remove, so return 0.

**Optimization Step:**
Before processing, we consolidate consecutive boxes of the same color at the end:
```python
while i < j and boxes[j] == boxes[j - 1]:
    j, k = j - 1, k + 1
```
This merges adjacent boxes of the same color as `boxes[j]` into our `k` counter, reducing redundant calculations.

**Recursive Cases:**

1. **Direct Removal Strategy:**
   ```python
   ans = dfs(i, j - 1, 0) + (k + 1) * (k + 1)
   ```
   Remove `boxes[j]` along with the `k` boxes of the same color. This gives us `(k + 1) * (k + 1)` points, plus whatever we get from removing the remaining boxes `[i, j-1]`.

2. **Merge Strategy:**
   ```python
   for h in range(i, j):
       if boxes[h] == boxes[j]:
           ans = max(ans, dfs(h + 1, j - 1, 0) + dfs(i, h, k + 1))
   ```
   For each box at position `h` in the range `[i, j-1]` that has the same color as `boxes[j]`:
   - First, completely remove all boxes between positions `h` and `j`: `dfs(h + 1, j - 1, 0)`
   - Now `boxes[h]` and `boxes[j]` (plus the `k` boxes to the right of `j`) can all be grouped together
   - Solve the subproblem `dfs(i, h, k + 1)` where `boxes[h]` now has `k + 1` boxes waiting to be combined with it: the original `k` boxes that were to the right of `j`, plus `boxes[j]` itself

**[Memoization](/problems/memoization_intro):**
The `@cache` decorator automatically memoizes the results, preventing redundant calculations for the same `(i, j, k)` state.

**Time Complexity:** `O(n^4)` where n is the number of boxes. We have `O(n^3)` possible states and each state takes `O(n)` time to compute.

**Space Complexity:** `O(n^3)` for the [memoization](/problems/memoization_intro) cache.

## Example Walkthrough

Let's walk through a small example with boxes `[2, 1, 2, 1]` to illustrate the solution approach.

**Initial Call:** `dfs(0, 3, 0)` - process boxes from index 0 to 3 with 0 additional boxes.

**Step 1: Processing `dfs(0, 3, 0)`**
- Current range: `[2, 1, 2, 1]`, `boxes[3] = 1`, `k = 0`
- No consecutive same-color boxes at the end, so no optimization step needed

**Option 1 - Direct Removal:**
- Remove `boxes[3]` (the `1` at index 3) immediately
- Score: `dfs(0, 2, 0) + (0 + 1)² = dfs(0, 2, 0) + 1`

**Option 2 - Merge Strategy:**
- Look for boxes with same color as `boxes[3]` (which is `1`)
- Found `boxes[1] = 1`
- Remove boxes between index 1 and 3 first: `dfs(2, 2, 0)`
  - This removes the `2` at index 2, giving us `1` point
- Then handle `dfs(0, 1, 1)` where the `1` parameter means we have 1 box waiting outside the range `[0, 1]` (specifically, `boxes[3]` which we're saving to merge later)
  - This processes `[2, 1]` where the `1` at index 1 can eventually be combined with the `1` at index 3 (the one we're tracking with k=1)
  - We can remove the `2` first: `dfs(0, 0, 0) = 1` point
  - Then remove both `1`s together (the one at index 1 plus the one at index 3 that we've been tracking): `(1 + 1)² = 4` points
- Total for this path: `1 + 1 + 4 = 6` points

**Continuing the Direct Removal path from Step 1:**
- `dfs(0, 2, 0)` processes `[2, 1, 2]`
- Direct removal: Remove the `2` at index 2 for `1` point, then solve `dfs(0, 1, 0)`
- Merge strategy: `boxes[0] = 2` matches `boxes[2]`
  - Remove middle box: `dfs(1, 1, 0) = 1` point (remove the `1`)
  - Then `dfs(0, 0, 1)`: Remove two `2`s together for `(1 + 1)² = 4` points
  - Total: `1 + 4 = 5` points
- Best for `dfs(0, 2, 0)` is `5` points
- So direct removal total: `5 + 1 = 6` points

**Final Result:** Both strategies give us `6` points as the maximum.

The key insight demonstrated here is how the merge strategy allows us to "save" boxes of the same color (the two `1`s) to remove them together later for more points (`2² = 4`) instead of removing them separately (`1² + 1² = 2`). This dynamic programming approach explores all possible removal orders efficiently through memoization.

## Python Solution

```python
class Solution:
    def removeBoxes(self, boxes: List[int]) -> int:
        """
        Remove boxes to maximize points. When removing k consecutive boxes of the same color,
        you get k * k points.
      
        Args:
            boxes: List of integers representing box colors
          
        Returns:
            Maximum points obtainable by removing all boxes
        """
        from functools import cache
      
        @cache
        def dfs(left: int, right: int, extra_count: int) -> int:
            """
            Dynamic programming function to find maximum points for boxes[left:right+1]
            with extra_count additional boxes of the same color as boxes[right] that can be
            merged with boxes[right].

            Args:
                left: Left boundary index (inclusive)
                right: Right boundary index (inclusive)
                extra_count: Number of boxes to the right of position 'right' (outside
                            the current range) that share the same color as boxes[right]
                            and will be merged with it when removed

            Returns:
                Maximum points obtainable from this subproblem
            """
            # Base case: no boxes left
            if left > right:
                return 0
          
            # Optimization: merge consecutive boxes of same color at the right end
            # This reduces redundant states in memoization
            while left < right and boxes[right] == boxes[right - 1]:
                right -= 1
                extra_count += 1
          
            # Option 1: Remove boxes[right] along with extra_count boxes of same color
            max_points = dfs(left, right - 1, 0) + (extra_count + 1) * (extra_count + 1)
          
            # Option 2: Try to merge boxes[right] with boxes[mid] where they have same color
            # Split the problem into two subproblems
            for mid in range(left, right):
                if boxes[mid] == boxes[right]:
                    # Remove boxes between mid and right first, then merge boxes[mid] with boxes[right]
                    points = (dfs(mid + 1, right - 1, 0) + 
                             dfs(left, mid, extra_count + 1))
                    max_points = max(max_points, points)
          
            return max_points
      
        # Calculate result for entire array
        n = len(boxes)
        result = dfs(0, n - 1, 0)
      
        # Clear cache to free memory
        dfs.cache_clear()
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n^4)`

The time complexity analysis involves understanding the memoization pattern:
- The `dfs` function has three parameters: `i`, `j`, and `k`
- `i` and `j` range from `0` to `n-1`, giving us `O(n^2)` possible combinations
- `k` represents the count of consecutive boxes equal to `boxes[j]` that follow position `j`. In the worst case, `k` can be at most `n`
- This gives us `O(n^3)` possible states to memoize

For each state `(i, j, k)`:
- The while loop at the beginning runs in `O(n)` time in the worst case
- The for loop iterates through positions from `i` to `j-1`, which is `O(n)` iterations
- Each iteration makes recursive calls that are already memoized or will be memoized

Therefore, the total time complexity is `O(n^3)` states × `O(n)` work per state = `O(n^4)`

**Space Complexity:** `O(n^3)`

The space complexity consists of:
- The memoization cache stores up to `O(n^3)` states (all possible combinations of `i`, `j`, and `k`)
- The recursion stack depth can go up to `O(n)` in the worst case when we recurse through all boxes
- Since `O(n^3)` dominates `O(n)`, the overall space complexity is `O(n^3)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect State Definition - Missing the "Extra Count" Parameter**

One of the most common mistakes is trying to solve this problem with a simpler DP state like `dp[i][j]` representing the maximum points from removing boxes in range `[i, j]`. This approach fails because it doesn't account for boxes outside the current range that could be merged later.

**Why this fails:**
Consider boxes `[1, 2, 1, 2]`. If we just use `dp[i][j]`:
- We might remove the middle `[2, 1]` first, getting some points
- But we lose the opportunity to merge the two 1's together for a better score

**Solution:**
The third parameter `k` (or `extra_count`) is crucial. It tracks boxes of the same color as `boxes[j]` that are **to the right of position `j`** (outside the current range `[i, j]`) and will be merged with `boxes[j]` when it's removed. This allows the algorithm to consider strategies where we remove intermediate boxes first, bringing distant same-colored boxes together to create larger groups.

### 2. **Forgetting the Optimization Step**

Without the consolidation step that merges consecutive same-colored boxes:
```python
while left < right and boxes[right] == boxes[right - 1]:
    right -= 1
    extra_count += 1
```

**Why this matters:**
- Without this optimization, you'd have many redundant states in your memoization
- For example, `dfs(0, 5, 2)` and `dfs(0, 4, 3)` might represent the same actual problem if `boxes[4] == boxes[5]`
- Without the loop, the solution is still correct and still has `O(n^3)` states, but it stores more of them and does more work for inputs with long runs of the same color

**Solution:**
Consolidate consecutive boxes of the same color at the boundary. Equivalent subproblems then map to the same memoized state, which reduces the number of distinct states that are actually computed.

### 3. **Incorrect Merging Logic**

A subtle but critical mistake is incorrectly handling the merge strategy:

**Wrong approach:**
```python
# INCORRECT: This doesn't properly account for the merging
for h in range(i, j):
    if boxes[h] == boxes[j]:
        ans = max(ans, dfs(i, h, k) + dfs(h + 1, j, 0))  # Wrong!
```

**Why this fails:**
This approach doesn't correctly model what happens when we remove the middle section. When we remove boxes `[h+1, j-1]`, we're left with `boxes[h]` that should now be adjacent to `boxes[j]` and the `k` extra boxes.

**Solution:**
The correct formulation is:
```python
ans = max(ans, dfs(h + 1, j - 1, 0) + dfs(i, h, k + 1))
```
- First remove the middle section `[h+1, j-1]` completely
- Then solve for `[i, h]` where `boxes[h]` now has `k+1` additional boxes to merge with

### 4. **Memory Limit Issues with Large Inputs**

The memoization cache can grow very large (`O(n^3)` entries), potentially causing memory issues.

**Solution:**
- Clear the cache after getting the result: `dfs.cache_clear()`
- Consider using an iterative DP approach with arrays if memory is critically constrained
- Use appropriate data types (avoid unnecessary large integers if the problem constraints allow)

