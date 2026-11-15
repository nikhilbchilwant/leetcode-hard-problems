# 312. Burst Balloons

> Source: https://algo.monster/liteproblems/312

## Problem Statement

You have `n` balloons arranged in a row, indexed from `0` to `n - 1`. Each balloon has a number written on it, given by the array `nums`.

Your task is to burst all the balloons to collect the maximum number of coins possible.

When you burst balloon `i`, you earn coins equal to `nums[i - 1] * nums[i] * nums[i + 1]`. The key rules are:

- If `i - 1` is out of bounds (i.e., you burst the first balloon), treat it as if there's a balloon with value `1` to the left
- If `i + 1` is out of bounds (i.e., you burst the last balloon), treat it as if there's a balloon with value `1` to the right
- After bursting a balloon, it disappears and the adjacent balloons become neighbors

The challenge is to find the optimal order to burst all balloons to maximize the total coins collected.

For example, if `nums = [3, 1, 5, 8]`:
- If you burst balloon 1 (value 1) first, you get `3 * 1 * 5 = 15` coins
- After bursting, the array becomes `[3, 5, 8]`
- If you then burst balloon 0 (value 3), you get `1 * 3 * 5 = 15` coins (note the virtual 1 on the left)
- And so on...

The goal is to determine the sequence of bursting that yields the maximum total coins.

## Intuition

The key insight is to think about this problem in reverse - instead of deciding which balloon to burst first, we think about which balloon to burst last.

Why does this perspective shift help? When we burst balloons in sequence, the neighbors keep changing, making it extremely difficult to track the state. For example, if we burst balloon `i` first, then balloons `i-1` and `i+1` become neighbors, completely changing the subsequent calculations.

However, if we think about which balloon to burst last in a given range, we can make a crucial observation: when balloon `k` is the last one to burst in range `[i, j]`, all other balloons between `i` and `j` have already been burst. This means:
- The coins we get from bursting `k` last is `arr[i] * arr[k] * arr[j]` (where `arr` includes the virtual 1s at both ends)
- The problem naturally divides into two independent subproblems: bursting all balloons in `[i, k]` and bursting all balloons in `[k, j]`

This "divide and conquer" structure suggests dynamic programming. We can define `f[i][j]` as the maximum coins obtainable by bursting all balloons between positions `i` and `j` (exclusive of `i` and `j` themselves).

For each range `[i, j]`, we try every possible position `k` as the last balloon to burst. The total coins would be:
- Coins from bursting all balloons in the left subrange: `f[i][k]`
- Coins from bursting all balloons in the right subrange: `f[k][j]`  
- Coins from bursting balloon `k` last: `arr[i] * arr[k] * arr[j]`

By trying all possible `k` values and taking the maximum, we ensure we find the optimal solution. The beauty of this approach is that the subproblems are independent - once we decide `k` is the last balloon to burst, the optimal solutions for the left and right subranges don't affect each other.

## Solution Approach

Based on our intuition, let's implement the dynamic programming solution step by step.

**Step 1: Prepare the array with boundaries**

First, we add virtual balloons with value `1` at both ends of the original array:
```python
arr = [1] + nums + [1]
```
This simplifies our calculations - we don't need special cases for boundary conditions.

**Step 2: Initialize the DP table**

We create a 2D table `f` where `f[i][j]` represents the maximum coins we can get by bursting all balloons between indices `i` and `j` (exclusive):
```python
f = [[0] * (n + 2) for _ in range(n + 2)]
```
The table size is `(n+2) × (n+2)` to accommodate the virtual balloons.

**Step 3: Fill the DP table**

The crucial part is the order of computation. We need to ensure that when calculating `f[i][j]`, the subproblems `f[i][k]` and `f[k][j]` have already been computed.

- We iterate `i` from `n-1` down to `0` (right to left)
- For each `i`, we iterate `j` from `i+2` to `n+2` (left to right)
- For each pair `(i, j)`, we try every possible `k` between `i+1` and `j-1` as the last balloon to burst

```python
for i in range(n - 1, -1, -1):
    for j in range(i + 2, n + 2):
        for k in range(i + 1, j):
            f[i][j] = max(f[i][j], f[i][k] + f[k][j] + arr[i] * arr[k] * arr[j])
```

The state transition equation is:
$$f[i][j] = \max(f[i][j], f[i][k] + f[k][j] + arr[i] \times arr[k] \times arr[j])$$

Where:
- `f[i][k]`: Maximum coins from bursting all balloons between `i` and `k`
- `f[k][j]`: Maximum coins from bursting all balloons between `k` and `j`
- `arr[i] * arr[k] * arr[j]`: Coins earned when bursting balloon `k` as the last one

**Step 4: Return the result**

The answer is `f[0][n+1]`, which represents the maximum coins from bursting all balloons between the two virtual boundaries (i.e., all the original balloons).

**Time and Space Complexity:**
- Time: `O(n³)` - three nested loops iterating through the ranges
- Space: `O(n²)` - for the DP table

## Example Walkthrough

Let's walk through a small example with `nums = [3, 1, 5]` to illustrate the solution approach.

**Step 1: Add virtual boundaries**
```
Original: [3, 1, 5]
With boundaries: arr = [1, 3, 1, 5, 1]
                 index: 0  1  2  3  4
```

**Step 2: Initialize DP table**
We create a 5×5 table (n+2 = 5) filled with zeros. `f[i][j]` represents max coins from bursting all balloons between indices i and j (exclusive).

**Step 3: Fill the DP table**

We iterate with `i` from 2 down to 0, and for each `i`, `j` from `i+2` to 4.

Let's trace through key calculations:

**When i=2, j=4:** (bursting balloons between index 2 and 4)
- Range contains only balloon at index 3 (value 5)
- Try k=3 as the last balloon to burst
- `f[2][4] = f[2][3] + f[3][4] + arr[2]*arr[3]*arr[4]`
- `f[2][4] = 0 + 0 + 1*5*1 = 5`

**When i=1, j=3:** (bursting balloons between index 1 and 3)
- Range contains only balloon at index 2 (value 1)
- Try k=2 as the last balloon to burst
- `f[1][3] = f[1][2] + f[2][3] + arr[1]*arr[2]*arr[3]`
- `f[1][3] = 0 + 0 + 3*1*5 = 15`

**When i=1, j=4:** (bursting balloons between index 1 and 4)
- Range contains balloons at indices 2 and 3
- Try k=2: `f[1][4] = f[1][2] + f[2][4] + arr[1]*arr[2]*arr[4]`
  - `= 0 + 5 + 3*1*1 = 8`
- Try k=3: `f[1][4] = f[1][3] + f[3][4] + arr[1]*arr[3]*arr[4]`
  - `= 15 + 0 + 3*5*1 = 30`
- Maximum: `f[1][4] = 30`

**When i=0, j=2:** (bursting balloons between index 0 and 2)
- Range contains only balloon at index 1 (value 3)
- Try k=1: `f[0][2] = 0 + 0 + 1*3*1 = 3`

**When i=0, j=3:** (bursting balloons between index 0 and 3)
- Try k=1: `f[0][3] = 0 + 15 + 1*3*5 = 30`
- Try k=2: `f[0][3] = 3 + 0 + 1*1*5 = 8`
- Maximum: `f[0][3] = 30`

**When i=0, j=4:** (bursting all original balloons)
- Try k=1: `f[0][4] = 0 + 30 + 1*3*1 = 33`
- Try k=2: `f[0][4] = 3 + 5 + 1*1*1 = 9`
- Try k=3: `f[0][4] = 30 + 0 + 1*5*1 = 35`
- Maximum: `f[0][4] = 35`

**Result:** The maximum coins obtainable is `f[0][4] = 35`

This corresponds to the optimal bursting order:
1. Burst balloon with value 1 first: earn 3*1*5 = 15 coins
2. Burst balloon with value 3 next: earn 1*3*5 = 15 coins  
3. Burst balloon with value 5 last: earn 1*5*1 = 5 coins
Total: 15 + 15 + 5 = 35 coins

## Python Solution

```python
class Solution:
    def maxCoins(self, nums: List[int]) -> int:
        """
        Calculate maximum coins obtainable by bursting all balloons.
      
        When balloon i is burst, you get nums[left] * nums[i] * nums[right] coins,
        where left and right are adjacent balloons after previous bursts.
      
        Args:
            nums: List of integers representing balloon values
          
        Returns:
            Maximum coins obtainable
        """
        n = len(nums)
      
        # Add virtual balloons with value 1 at both ends
        # This simplifies boundary conditions
        balloons = [1] + nums + [1]
      
        # dp[left][right] = maximum coins obtainable from bursting 
        # all balloons between left and right (exclusive)
        dp = [[0] * (n + 2) for _ in range(n + 2)]
      
        # Iterate through all possible intervals from smallest to largest
        # Start from the right side and move left
        for left in range(n - 1, -1, -1):
            # For each left boundary, consider all valid right boundaries
            for right in range(left + 2, n + 2):
                # Try bursting each balloon k as the last balloon in interval (left, right)
                for last_burst in range(left + 1, right):
                    # When balloon k is the last to burst in (left, right):
                    # - All balloons in (left, k) are already burst: dp[left][last_burst]
                    # - All balloons in (k, right) are already burst: dp[last_burst][right]
                    # - Bursting k gives: balloons[left] * balloons[k] * balloons[right]
                    coins = (dp[left][last_burst] + 
                            dp[last_burst][right] + 
                            balloons[left] * balloons[last_burst] * balloons[right])
                  
                    dp[left][right] = max(dp[left][right], coins)
      
        # Return maximum coins for bursting all balloons between index 0 and n+1
        return dp[0][n + 1]
```

## Time and Space Complexity

The time complexity of this code is `O(n^3)`, where `n` is the length of the array `nums`.

This is determined by the three nested loops:
- The outer loop iterates from `n-1` down to `0`, giving `n` iterations
- The middle loop iterates from `i+2` to `n+2`, which in the worst case gives approximately `n` iterations
- The inner loop iterates from `i+1` to `j`, which in the worst case gives approximately `n` iterations
- The operation inside the innermost loop (`max` comparison and arithmetic) takes `O(1)` time

Therefore, the overall time complexity is `O(n) × O(n) × O(n) × O(1) = O(n^3)`.

The space complexity is `O(n^2)`, where `n` is the length of the array `nums`.

This is due to:
- The array `arr` which has size `n+2`, contributing `O(n)` space
- The 2D DP table `f` which has dimensions `(n+2) × (n+2)`, contributing `O((n+2)^2) = O(n^2)` space

The dominant term is `O(n^2)`, so the overall space complexity is `O(n^2)`.

## Common Pitfalls

### 1. **Incorrect Iteration Order in Dynamic Programming**

One of the most common mistakes is iterating through the DP table in the wrong order, which leads to using uncomputed subproblems.

**Wrong approach:**
```python
# INCORRECT - Forward iteration
for left in range(n):
    for right in range(left + 2, n + 2):
        # This won't work because we need f[left][k] and f[k][right]
        # to be already computed, but they might not be
```

**Why it fails:** When computing `dp[left][right]`, we need `dp[left][k]` and `dp[k][right]` for all `k` between `left` and `right`. With forward iteration, these values haven't been computed yet.

**Correct approach:**
```python
# CORRECT - Backward iteration for left, forward for right
for left in range(n - 1, -1, -1):
    for right in range(left + 2, n + 2):
        # Now all required subproblems are available
```

### 2. **Misunderstanding the Problem - Thinking About "Which Balloon to Burst First"**

Many people initially approach this problem by trying to decide which balloon to burst first, leading to incorrect greedy or recursive solutions.

**Wrong mental model:**
```python
# INCORRECT thinking: "Let me find the best balloon to burst first"
def maxCoins(nums):
    if not nums:
        return 0
    max_coins = 0
    for i in range(len(nums)):
        left = nums[i-1] if i > 0 else 1
        right = nums[i+1] if i < len(nums)-1 else 1
        coins = left * nums[i] * right
        remaining = nums[:i] + nums[i+1:]
        max_coins = max(max_coins, coins + maxCoins(remaining))
    return max_coins
```

**Why it fails:** This approach has overlapping subproblems that are hard to memoize because the array keeps changing shape. After removing elements, the same subproblem can appear in many different forms.

**Correct mental model:** Think about which balloon to burst **last** in each interval. This way, when we burst the last balloon `k` in interval `(i, j)`, we know exactly what its neighbors are: `balloons[i]` and `balloons[j]`.

### 3. **Off-by-One Errors with Virtual Balloons**

**Wrong indexing:**
```python
# INCORRECT - Forgetting to account for virtual balloons
balloons = [1] + nums + [1]
dp = [[0] * n for _ in range(n)]  # Wrong size!
# Should be (n+2) × (n+2)
```

**Correct indexing:**
```python
balloons = [1] + nums + [1]
dp = [[0] * (n + 2) for _ in range(n + 2)]  # Correct size
# Final answer is dp[0][n + 1], not dp[0][n]
```

### 4. **Incorrect Base Cases or Range Boundaries**

**Common mistakes:**
- Starting `right` from `left + 1` instead of `left + 2` (there must be at least one balloon between boundaries)
- Using wrong range for the last balloon to burst: should be `range(left + 1, right)`, not `range(left, right + 1)`

**Verification tip:** Always check that your intervals make sense:
- `dp[i][i+1] = 0` (no balloons between adjacent positions)
- `dp[i][i+2]` represents bursting exactly one balloon (at position `i+1`)

