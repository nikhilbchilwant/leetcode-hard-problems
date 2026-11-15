# 1866. Number of Ways to Rearrange Sticks With K Sticks Visible

> Source: https://algo.monster/liteproblems/1866

## Problem Statement

You have `n` sticks with unique lengths from `1` to `n`. You need to arrange these sticks in a row such that exactly `k` sticks are visible when looking from the left side.

A stick is considered **visible from the left** if there are no longer sticks positioned to its left. In other words, a stick is visible if it's the tallest among all sticks from the leftmost position up to and including itself.

For example, given the arrangement `[1, 3, 2, 5, 4]`:
- Stick with length `1` is visible (it's the first stick)
- Stick with length `3` is visible (it's taller than all sticks to its left, which is just `1`)
- Stick with length `2` is not visible (stick `3` to its left is taller)
- Stick with length `5` is visible (it's taller than all sticks to its left: `1`, `3`, and `2`)
- Stick with length `4` is not visible (stick `5` to its left is taller)

So this arrangement has exactly `3` visible sticks from the left.

Given integers `n` (the number of sticks) and `k` (the desired number of visible sticks), you need to find how many different arrangements of the `n` sticks will result in exactly `k` sticks being visible from the left.

Since the answer can be very large, return the result modulo `10^9 + 7`.

## Intuition

Let's think about this problem step by step. We need to count arrangements where exactly `k` sticks are visible from the left.

The key insight is to consider what happens with the last (rightmost) stick in our arrangement. There are two cases:

**Case 1: The last stick is visible from the left**
For the last stick to be visible, it must be the tallest stick (length `i` when we have `i` sticks total). If we place the tallest stick at the end, it will always be visible because nothing to its left can block it.

In this case, we need the remaining `i-1` sticks in front of it to form an arrangement where exactly `j-1` sticks are visible (since the last stick adds one more visible stick). This gives us `f[i-1][j-1]` ways.

**Case 2: The last stick is NOT visible from the left**
For the last stick to be invisible, there must be a taller stick somewhere to its left. This means the last stick cannot be the tallest one (length `i`). It can be any of the other `i-1` shorter sticks.

When we place one of these `i-1` shorter sticks at the end, it doesn't affect the visibility count from the left - the number of visible sticks remains the same. We need the first `i-1` positions to have exactly `j` visible sticks. For each such arrangement of `i-1` sticks with `j` visible, we can choose any of the `i-1` non-tallest sticks to place at the end. This gives us `f[i-1][j] * (i-1)` ways.

Combining both cases, we get the recurrence relation:
`f[i][j] = f[i-1][j-1] + f[i-1][j] * (i-1)`

The base case is `f[0][0] = 1` (empty arrangement with zero visible sticks), and we build up to `f[n][k]` which gives us our answer.

This [dynamic programming](/problems/dynamic_programming_intro) approach works because each subproblem `f[i][j]` only depends on smaller subproblems with `i-1` sticks, allowing us to build the solution bottom-up.

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

We use [dynamic programming](/problems/dynamic_programming_intro) to solve this problem. Let's define `f[i][j]` as the number of arrangements of `i` sticks where exactly `j` sticks are visible from the left.

**Initialization:**
- Create a 2D DP table `f` with dimensions `(n+1) × (k+1)` initialized with zeros
- Set base case: `f[0][0] = 1` (empty arrangement with zero visible sticks)
- All other states start at 0

**State Transition:**
For each state `f[i][j]` where `i` represents the number of sticks and `j` represents the number of visible sticks:

`f[i][j] = f[i-1][j-1] + f[i-1][j] * (i-1)`

This formula comes from two cases:
- `f[i-1][j-1]`: Place the tallest stick (length `i`) at the end, making it visible
- `f[i-1][j] * (i-1)`: Place any of the `i-1` shorter sticks at the end, keeping it invisible

**Implementation Details:**
```python
def rearrangeSticks(self, n: int, k: int) -> int:
    mod = 10**9 + 7
    f = [[0] * (k + 1) for _ in range(n + 1)]
    f[0][0] = 1
  
    for i in range(1, n + 1):
        for j in range(1, k + 1):
            f[i][j] = (f[i - 1][j - 1] + f[i - 1][j] * (i - 1)) % mod
  
    return f[n][k]
```

**Algorithm Flow:**
1. Iterate through all states from `i = 1` to `n` (number of sticks)
2. For each `i`, iterate through `j = 1` to `k` (number of visible sticks)
3. Apply the transition formula with modulo to prevent overflow
4. Return `f[n][k]` as the final answer

**Time Complexity:** `O(n × k)` - we fill a 2D table of size `n × k`

**Space Complexity:** `O(n × k)` - for storing the DP table

The modulo operation `10^9 + 7` is applied at each step to keep numbers manageable and prevent integer overflow, as the number of arrangements can grow very large.

## Example Walkthrough

Let's walk through a small example with `n = 3` sticks (lengths 1, 2, 3) and we want exactly `k = 2` visible sticks.

**Step 1: Initialize DP Table**
We create a table `f[i][j]` where `i` is the number of sticks and `j` is the number of visible sticks.

```
    j=0  j=1  j=2
i=0  1    0    0
i=1  0    0    0
i=2  0    0    0
i=3  0    0    0
```

**Step 2: Fill the DP Table**

For `i=1` (1 stick of length 1):
- `f[1][1] = f[0][0] + f[0][1] * 0 = 1 + 0 = 1`
  - Only arrangement: [1], which has 1 visible stick

For `i=2` (2 sticks of lengths 1, 2):
- `f[2][1] = f[1][0] + f[1][1] * 1 = 0 + 1 * 1 = 1`
  - Arrangement: [2,1] (only stick 2 is visible)
- `f[2][2] = f[1][1] + f[1][2] * 1 = 1 + 0 = 1`
  - Arrangement: [1,2] (both sticks are visible)

For `i=3` (3 sticks of lengths 1, 2, 3):
- `f[3][1] = f[2][0] + f[2][1] * 2 = 0 + 1 * 2 = 2`
  - Arrangements: [3,1,2] and [3,2,1] (only stick 3 is visible)
- `f[3][2] = f[2][1] + f[2][2] * 2 = 1 + 1 * 2 = 3`
  - Let's verify this with actual arrangements:
    - From `f[2][1]`: Place stick 3 at end of [2,1] → [2,1,3] (sticks 2 and 3 visible)
    - From `f[2][2]`: Place stick 1 or 2 at end of [1,2] → [1,2,3] gives wrong count
    - Actually: [1,3,2] (1 and 3 visible), [2,3,1] (2 and 3 visible), [2,1,3] (2 and 3 visible)

**Final Answer:** `f[3][2] = 3`

The three valid arrangements with exactly 2 visible sticks are:
- [1,3,2]: Sticks 1 and 3 are visible
- [2,1,3]: Sticks 2 and 3 are visible  
- [2,3,1]: Sticks 2 and 3 are visible

The recurrence relation captures both scenarios:
1. Adding the tallest stick at the end (making it visible)
2. Adding any shorter stick at the end (keeping it hidden behind taller sticks to its left)

## Python Solution

```python
class Solution:
    def rearrangeSticks(self, n: int, k: int) -> int:
        """
        Calculate the number of ways to arrange n sticks such that exactly k sticks are visible.
        A stick is visible if all sticks to its left are shorter.
      
        Args:
            n: Total number of sticks (each with unique height from 1 to n)
            k: Number of sticks that should be visible
          
        Returns:
            Number of valid arrangements modulo 10^9 + 7
        """
        MOD = 10**9 + 7
      
        # dp[i][j] represents the number of ways to arrange i sticks 
        # such that exactly j sticks are visible
        dp = [[0] * (k + 1) for _ in range(n + 1)]
      
        # Base case: 0 sticks with 0 visible sticks = 1 way (empty arrangement)
        dp[0][0] = 1
      
        # Fill the DP table
        for num_sticks in range(1, n + 1):
            for num_visible in range(1, k + 1):
                # Case 1: Place the shortest stick at the front (becomes visible)
                # This adds one more visible stick
                place_shortest_first = dp[num_sticks - 1][num_visible - 1]
              
                # Case 2: Place the shortest stick anywhere except the front (not visible)
                # There are (num_sticks - 1) positions where it won't be visible
                place_shortest_not_first = dp[num_sticks - 1][num_visible] * (num_sticks - 1)
              
                # Total ways is the sum of both cases
                dp[num_sticks][num_visible] = (place_shortest_first + place_shortest_not_first) % MOD
      
        return dp[n][k]
```

## Time and Space Complexity

The time complexity of this code is `O(n × k)`. The algorithm uses a nested loop structure where the outer loop iterates from `1` to `n` (n iterations) and the inner loop iterates from `1` to `k` (k iterations). For each combination of `i` and `j`, we perform a constant time operation to calculate `f[i][j]`. Therefore, the total number of operations is proportional to `n × k`.

The space complexity is `O(n × k)`. The algorithm creates a 2D array `f` with dimensions `(n + 1) × (k + 1)` to store the dynamic programming states. This array requires `O((n + 1) × (k + 1))` space, which simplifies to `O(n × k)`. The only other variables used are `mod`, `i`, and `j`, which require constant space `O(1)`. Thus, the overall space complexity is dominated by the 2D array, resulting in `O(n × k)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Interpretation of the Recurrence Relation**

A common mistake is misunderstanding which stick we're adding and where. The code comments mention "shortest stick" but the mathematical formula uses the "tallest stick" perspective. This conceptual mismatch can lead to confusion and errors.

**The Problem:**
The code comments say we're placing the "shortest stick," but the recurrence relation `f[i][j] = f[i-1][j-1] + f[i-1][j] * (i-1)` is actually based on considering the tallest stick among the first `i` sticks.

**Why This Happens:**
When working with permutations of sticks numbered 1 to `i`, we can think about it two ways:
- Adding stick `i` (the tallest) to arrangements of sticks 1 to `i-1`
- Adding stick 1 (the shortest) to arrangements of renumbered sticks 2 to `i`

Both perspectives are valid due to the relative nature of heights, but mixing them causes confusion.

**Solution:**
Be consistent with your mental model. The cleaner approach is to think about adding the tallest stick:

```python
def rearrangeSticks(self, n: int, k: int) -> int:
    MOD = 10**9 + 7
  
    # dp[i][j] = ways to arrange i sticks with j visible
    dp = [[0] * (k + 1) for _ in range(n + 1)]
    dp[0][0] = 1
  
    for i in range(1, n + 1):
        for j in range(1, k + 1):
            # Add tallest stick (height i) at the front -> always visible
            dp[i][j] = dp[i-1][j-1]
          
            # Add tallest stick at any of the i-1 non-front positions -> never visible
            dp[i][j] = (dp[i][j] + dp[i-1][j] * (i-1)) % MOD
  
    return dp[n][k]
```

### 2. **Forgetting to Apply Modulo During Intermediate Calculations**

**The Problem:**
Only applying modulo at the very end or forgetting it in intermediate steps can cause integer overflow, especially for large values of `n` and `k`.

**Incorrect approach:**
```python
# Wrong - can overflow before modulo
dp[i][j] = dp[i-1][j-1] + dp[i-1][j] * (i-1)
return dp[n][k] % MOD
```

**Solution:**
Apply modulo after each arithmetic operation that could produce large numbers:

```python
# Correct - prevents overflow
dp[i][j] = (dp[i-1][j-1] + dp[i-1][j] * (i-1)) % MOD
```

### 3. **Index Out of Bounds When j > i**

**The Problem:**
It's impossible to have more visible sticks than total sticks (j > i), but if we don't handle this case, we might access invalid states or get incorrect results.

**Solution:**
Add a boundary check or ensure the loop limits prevent this:

```python
for i in range(1, n + 1):
    for j in range(1, min(i, k) + 1):  # j cannot exceed i
        dp[i][j] = (dp[i-1][j-1] + dp[i-1][j] * (i-1)) % MOD
```

### 4. **Space Optimization Pitfall**

**The Problem:**
Since we only need the previous row to compute the current row, you might try to optimize space to O(k) using a 1D array. However, updating in-place incorrectly can overwrite values still needed.

**Incorrect space optimization:**
```python
# Wrong - overwrites values still needed
dp = [0] * (k + 1)
dp[0] = 1
for i in range(1, n + 1):
    for j in range(1, k + 1):  # Wrong direction!
        dp[j] = (dp[j-1] + dp[j] * (i-1)) % MOD
```

**Solution:**
If optimizing space, iterate backwards through j to avoid overwriting needed values:

```python
# Correct space-optimized version
dp = [0] * (k + 1)
dp[0] = 1
for i in range(1, n + 1):
    for j in range(min(i, k), 0, -1):  # Iterate backwards
        dp[j] = (dp[j-1] + dp[j] * (i-1)) % MOD
```

