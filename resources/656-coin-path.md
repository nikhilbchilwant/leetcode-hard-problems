# 656. Coin Path

> Source: https://algo.monster/liteproblems/656

## Problem Statement

You have an array `coins` of integers where each position represents a cost to visit that position. The array uses 1-based indexing (position 1 is the first element). Some positions may have a value of `-1`, which means you cannot visit them.

Starting from position 1, you need to reach position `n` (the last position) by making jumps. From any position `i`, you can jump to any position between `i+1` and `i+maxJump` (inclusive), as long as:
- The target position is within the array bounds
- The target position doesn't have a value of `-1`

When you visit a position `i`, you must pay `coins[i]` as the cost.

Your goal is to find the path from position 1 to position `n` that minimizes the total cost. The output should be an array of positions (using 1-based indexing) that represents this minimum cost path.

If there are multiple paths with the same minimum cost, you need to return the lexicographically smallest path. A path is lexicographically smaller if at the first position where two paths differ, the lexicographically smaller path has a smaller position number.

For example, path `[1, 2, 5]` is lexicographically smaller than `[1, 3, 5]` because at the second position, 2 < 3.

If it's impossible to reach position `n` (either because position `n` has value `-1` or there's no valid path), return an empty array.

## Intuition

The key insight is that we need to find both the minimum cost AND the lexicographically smallest path. This dual requirement suggests we should work backwards from the destination.

Why backwards? If we work forwards, we'd need to track multiple paths with the same cost to ensure we get the lexicographically smallest one. But working backwards simplifies this - we first find the minimum cost to reach the end from each position, then reconstruct the path going forward.

Think of it like this: once we know the minimum cost from each position to the end, we can make greedy choices when building our path. Starting from position 1, at each step we pick the earliest valid next position that maintains our minimum cost path.

The [dynamic programming](/problems/dynamic_programming_intro) recurrence is straightforward: `f[i] = min(f[j] + coins[i])` for all reachable positions `j` from `i`. Here `f[i]` represents the minimum cost to go from position `i` to position `n`.

The clever part comes in path reconstruction. Since we want the lexicographically smallest path, we iterate through positions from left to right. At each step, we greedily take the first position whose minimum cost equals our remaining budget. This works because:
1. We know the total minimum cost from `f[0]`
2. As we visit each position, we subtract its cost from our budget
3. The first position we encounter with cost equal to our budget must be part of the lexicographically smallest path

For example, if positions 2 and 3 both lead to paths with the same total cost, we'll encounter position 2 first in our left-to-right scan and include it in our answer, ensuring lexicographic minimality.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses [dynamic programming](/problems/dynamic_programming_intro) with backward traversal followed by forward path reconstruction.

**Step 1: Initial Check**
```python
if coins[-1] == -1:
    return []
```
First, we check if the destination position has value `-1`. If it does, it's impossible to reach it, so we return an empty array immediately.

**Step 2: Initialize DP Array**
```python
n = len(coins)
f = [inf] * n
f[-1] = coins[-1]
```
We create a DP array `f` where `f[i]` represents the minimum cost to reach position `n` from position `i`. Initialize all values to infinity except the last position, which costs `coins[-1]` (the cost of being at the destination).

**Step 3: Fill DP Table Backwards**
```python
for i in range(n - 2, -1, -1):
    if coins[i] != -1:
        for j in range(i + 1, min(n, i + maxJump + 1)):
            if f[i] > f[j] + coins[i]:
                f[i] = f[j] + coins[i]
```
Working backwards from position `n-2` to `0`:
- Skip positions with value `-1` (unreachable)
- For each valid position `i`, check all positions we can jump to: `[i+1, i+maxJump]`
- Update `f[i]` with the minimum cost: current position's cost plus the minimum cost from the next position

**Step 4: Check Feasibility**
```python
if f[0] == inf:
    return []
```
If `f[0]` is still infinity, there's no valid path from position 1 to position `n`.

**Step 5: Reconstruct Path**
```python
ans = []
s = f[0]
for i in range(n):
    if f[i] == s:
        s -= coins[i]
        ans.append(i + 1)
```
To build the lexicographically smallest path:
- Start with the total minimum cost `s = f[0]`
- Iterate through positions from left to right
- When we find a position `i` where `f[i] == s`, it means this position is on our optimal path
- Add this position to our answer (converting to 1-based indexing with `i + 1`)
- Subtract the cost of this position from our remaining budget `s`

This greedy approach guarantees the lexicographically smallest path because we always choose the earliest position that maintains the minimum cost. The algorithm ensures that the path we construct has exactly the minimum cost we calculated, and by processing positions in order, we get the lexicographically smallest such path.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Input:** `coins = [1, 2, 4, -1, 2]`, `maxJump = 2`

We need to find the minimum cost path from position 1 to position 5 (using 1-based indexing).

**Step 1: Initial Check**
- `coins[-1] = 2` (not -1), so we can potentially reach the destination.

**Step 2: Initialize DP Array**
- `n = 5`
- `f = [inf, inf, inf, inf, 2]`
- `f[4] = coins[4] = 2` (cost to be at position 5)

**Step 3: Fill DP Table Backwards**

Working from position 4 (index 3) backwards:

*Position 4 (index 3):*
- `coins[3] = -1`, so skip this position (cannot visit)

*Position 3 (index 2):*
- `coins[2] = 4` (valid position)
- Can jump to positions 4 or 5 (indices 3 or 4)
- Position 4 has `coins[3] = -1`, so skip
- Position 5: `f[2] = min(inf, f[4] + coins[2]) = 2 + 4 = 6`
- Update: `f = [inf, inf, 6, inf, 2]`

*Position 2 (index 1):*
- `coins[1] = 2` (valid position)
- Can jump to positions 3 or 4 (indices 2 or 3)
- Position 3: `f[1] = min(inf, f[2] + coins[1]) = 6 + 2 = 8`
- Position 4 has `coins[3] = -1`, so skip
- Update: `f = [inf, 8, 6, inf, 2]`

*Position 1 (index 0):*
- `coins[0] = 1` (valid position)
- Can jump to positions 2 or 3 (indices 1 or 2)
- Position 2: `f[0] = min(inf, f[1] + coins[0]) = 8 + 1 = 9`
- Position 3: `f[0] = min(9, f[2] + coins[0]) = min(9, 6 + 1) = 7`
- Update: `f = [7, 8, 6, inf, 2]`

**Step 4: Check Feasibility**
- `f[0] = 7` (not infinity), so a valid path exists.

**Step 5: Reconstruct Path**

Starting with `s = f[0] = 7`, iterate through positions:

*Position 1 (index 0):*
- `f[0] = 7`, which equals `s`
- Add position 1 to answer: `ans = [1]`
- Update budget: `s = 7 - coins[0] = 7 - 1 = 6`

*Position 2 (index 1):*
- `f[1] = 8`, which doesn't equal `s = 6`
- Skip this position

*Position 3 (index 2):*
- `f[2] = 6`, which equals `s`
- Add position 3 to answer: `ans = [1, 3]`
- Update budget: `s = 6 - coins[2] = 6 - 4 = 2`

*Position 4 (index 3):*
- `f[3] = inf`, which doesn't equal `s = 2`
- Skip this position

*Position 5 (index 4):*
- `f[4] = 2`, which equals `s`
- Add position 5 to answer: `ans = [1, 3, 5]`
- Update budget: `s = 2 - coins[4] = 2 - 2 = 0`

**Final Answer:** `[1, 3, 5]` with total cost 7

This path visits positions 1→3→5 with costs 1+4+2=7. Note that we could also go 1→2→3→5 with costs 1+2+4+2=9, but our algorithm correctly finds the minimum cost path. The greedy reconstruction ensures we get the lexicographically smallest path among all minimum cost paths.

## Python Solution

```python
class Solution:
    def cheapestJump(self, coins: List[int], maxJump: int) -> List[int]:
        # If the last position is unreachable (marked as -1), no valid path exists
        if coins[-1] == -1:
            return []

        n = len(coins)

        # dp[i] represents the minimum cost to reach the end from position i
        dp = [float('inf')] * n
        dp[-1] = coins[-1]  # Base case: cost at the last position is just its coin value

        # Fill the dp table from right to left (backward)
        for i in range(n - 2, -1, -1):
            # Skip positions marked as unreachable (-1)
            if coins[i] != -1:
                # Try all possible jumps from position i
                for j in range(i + 1, min(n, i + maxJump + 1)):
                    # Update minimum cost if we found a better path
                    if dp[i] > dp[j] + coins[i]:
                        dp[i] = dp[j] + coins[i]

        # If starting position has infinite cost, no valid path exists
        if dp[0] == float('inf'):
            return []

        # Reconstruct the path with lexicographically smallest sequence
        path = []
        remaining_cost = dp[0]

        # Greedily select positions that maintain the optimal cost
        for i in range(n):
            if dp[i] == remaining_cost:
                remaining_cost -= coins[i]
                path.append(i + 1)  # Convert to 1-indexed position

        return path
```

## Time and Space Complexity

**Time Complexity:** `O(n * maxJump)`

The algorithm consists of two main parts:
- The dynamic programming phase: We iterate through all `n` positions from right to left. For each position `i`, we check up to `maxJump` positions ahead (from `i+1` to `min(n, i + maxJump + 1)`). This gives us `O(n * maxJump)` operations.
- The path reconstruction phase: We iterate through the array once more with `O(n)` operations.

Therefore, the overall time complexity is `O(n * maxJump) + O(n) = O(n * maxJump)`.

**Space Complexity:** `O(n)`

The algorithm uses:
- Array `f` of size `n` to store the minimum cost from each position to the end: `O(n)`
- Array `ans` to store the result path, which in the worst case contains all positions: `O(n)`
- A few variables (`s`, `i`, `j`) that use constant space: `O(1)`

The total space complexity is `O(n) + O(n) + O(1) = O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Building the DP Forward Instead of Backward

A forward DP where `g[i]` is the minimum cost to reach position `i` from position 1 finds the right total, but the path it records is not always the lexicographically smallest one. Keeping the best predecessor of each position chooses among predecessors, while lexicographic order is decided by the earliest positions on the path. For `coins = [0, 0, 0]` and `maxJump = 2`, both `[1, 3]` and `[1, 2, 3]` cost 0. A forward DP that records position 1 as the predecessor of position 3 returns `[1, 3]`, but `[1, 2, 3]` is lexicographically smaller.

**Solution:**
Compute `dp[i]` as the minimum cost from position `i` to the end, as the solution does. Then walk forward from position 1 and take the first position whose `dp` value equals the remaining cost. That position is always within `maxJump` of the previous one: the optimal next position `j` satisfies `dp[j] == remaining_cost` and lies within range, so the first match is at or before `j`. Choosing the earliest match at every step gives the lexicographically smallest path among all minimum-cost paths.

### Pitfall 2: Forgetting That Position 1 Can Be Blocked

If `coins[0] == -1`, no path exists. The solution handles this without a special case: the DP loop skips index 0, so `dp[0]` stays infinite and the function returns `[]`. For a single-element array, `dp[0] = coins[0]`, and the reconstruction returns `[1]`. Code that seeds `dp[0]` directly or skips the infinity check before reconstruction loses these cases.

### Pitfall 3: Confusion Between 0-indexed and 1-indexed Positions

The problem uses 1-based indexing for output, but the implementation uses 0-based indexing internally. This can lead to off-by-one errors if not handled consistently throughout the code.

**Solution:**
Be explicit about conversions and add comments to clarify indexing:

```python
# When adding to path, always convert: internal_index + 1
path.append(current_pos + 1)  # Convert 0-indexed to 1-indexed

# When processing jump ranges, remember maxJump is inclusive
for j in range(i + 1, min(n, i + maxJump + 1)):  # i+1 to i+maxJump inclusive
```

