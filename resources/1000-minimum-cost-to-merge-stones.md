# 1000. Minimum Cost to Merge Stones

> Source: https://algo.monster/liteproblems/1000

## Problem Statement

You are given `n` piles of stones arranged in a row, where the `i-th` pile contains `stones[i]` stones.

In each move, you must merge exactly `k` consecutive piles into a single pile. The cost of this merge operation equals the total number of stones in those `k` piles being merged.

Your goal is to find the minimum total cost to merge all piles into one single pile. If it's impossible to merge all piles into one, return `-1`.

For example, if you have stones `[3, 2, 4, 1]` and `k = 2`:
- You could merge piles at positions 1 and 2 (stones 3 and 2) with cost 5, resulting in `[5, 4, 1]`
- Then merge positions 1 and 2 (stones 5 and 4) with cost 9, resulting in `[9, 1]`
- Finally merge the last two piles with cost 10, resulting in `[10]`
- Total cost would be 5 + 9 + 10 = 24

The key constraint is that you can only merge exactly `k` consecutive piles at a time - no more, no less. This means that not all configurations can be reduced to a single pile. For instance, if you have 4 piles and `k = 3`, you can merge 3 piles to get 2 piles, but then you cannot merge 2 piles when `k = 3`, making it impossible to reach a single pile.

## Intuition

Let's think about when it's possible to merge all stones into one pile. Each merge operation reduces the number of piles by `K - 1` (we take `K` piles and produce 1 pile). Starting with `n` piles and ending with 1 pile, we need to reduce by `n - 1` piles total. This means we need `(n - 1) / (K - 1)` merge operations. For this to be a whole number, `(n - 1) % (K - 1)` must equal 0. If not, it's impossible to merge all piles into one.

Now for the minimum cost calculation. The key insight is that merging stones is a range-based problem - we're always working with consecutive piles. This suggests using [dynamic programming](/problems/dynamic_programming_intro) where we consider all possible ways to merge subarrays of stones.

Let's define `f[i][j][m]` as the minimum cost to merge stones from pile `i` to pile `j` into exactly `m` piles. Why do we need the third dimension `m`? Because intermediate states matter - sometimes we need to first merge subsections into a specific number of piles before we can perform the final merge.

The recurrence relation works as follows:
- To merge stones `[i, j]` into `m` piles, we can split the range at some point `h` and merge `[i, h]` into 1 pile and `[h+1, j]` into `m-1` piles
- To merge stones `[i, j]` into 1 pile, we first need to merge them into `K` piles (which costs `f[i][j][K]`), then perform one final merge of those `K` piles (which costs the sum of all stones in range `[i, j]`)

The base case is `f[i][i][1] = 0` (a single pile is already 1 pile with no cost).

We use prefix sums to quickly calculate the total stones in any range `[i, j]`, which represents the cost of the final merge operation when combining `K` piles into 1.

The answer will be `f[1][n][1]` - the minimum cost to merge all stones from position 1 to `n` into exactly 1 pile.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The implementation follows the [dynamic programming](/problems/dynamic_programming_intro) approach with these key components:

**1. Impossibility Check:**
```python
if (n - 1) % (K - 1):
    return -1
```
First, we verify if merging is possible. If `(n - 1)` is not divisible by `(K - 1)`, we cannot merge all piles into one.

**2. [Prefix Sum](/problems/subarray_sum) Array:**
```python
s = list(accumulate(stones, initial=0))
```
We create a prefix sum array to quickly calculate the sum of stones in any range `[i, j]` using `s[j] - s[i - 1]`. The `initial=0` ensures our prefix sum is 1-indexed for easier calculation.

**3. DP Table Initialization:**
```python
f = [[[inf] * (K + 1) for _ in range(n + 1)] for _ in range(n + 1)]
for i in range(1, n + 1):
    f[i][i][1] = 0
```
We create a 3D DP table `f[i][j][k]` where:
- `i` is the starting position (1-indexed)
- `j` is the ending position (1-indexed)
- `k` is the number of piles to merge into

Initialize all values to infinity except `f[i][i][1] = 0` (single pile requires no merging).

**4. Main DP Loop:**
```python
for l in range(2, n + 1):  # length of subarray
    for i in range(1, n - l + 2):  # starting position
        j = i + l - 1  # ending position
```
We iterate through all possible subarray lengths from 2 to `n`, and for each length, we consider all valid starting positions.

**5. Calculate Minimum Cost for k Piles:**
```python
for k in range(1, K + 1):
    for h in range(i, j):
        f[i][j][k] = min(f[i][j][k], f[i][h][1] + f[h + 1][j][k - 1])
```
For merging range `[i, j]` into `k` piles, we try split points `h`. We merge `[i, h]` into 1 pile and `[h+1, j]` into `k-1` piles, taking the minimum cost among all splits. The case `k = 1` never produces a finite value here (`f[h+1][j][0]` stays infinity), so the full solution starts `k` at 2. It also steps `h` by `K - 1` (`range(i, j, K - 1)`): the left part `[i, h]` can become 1 pile only when its length is `1, K, 2K - 1, ...`, so the other split points always give infinity and can be skipped.

In the full solution, `f` is named `dp`, `s` is `prefix_sum`, `i`/`j` are `start`/`end`, `k` is `num_piles`, and `h` is `split`.

**6. Final Merge to One Pile:**
```python
f[i][j][1] = f[i][j][K] + s[j] - s[i - 1]
```
To merge range `[i, j]` into 1 pile, we first merge it into `K` piles (cost: `f[i][j][K]`), then perform the final merge of those `K` piles into 1. The cost of this final merge equals the total stones in the range, calculated as `s[j] - s[i - 1]`.

**7. Return Result:**
```python
return f[1][n][1]
```
The answer is the minimum cost to merge all stones (from position 1 to n) into exactly 1 pile.

The time complexity is `O(n³)` and space complexity is `O(n² * K)`, where `n` is the number of stone piles (see the complexity section for why stepping by `K - 1` removes the factor `K`).

## Example Walkthrough

Let's walk through a concrete example with `stones = [3, 2, 4, 1]` and `K = 2`.

**Step 1: Check if merging is possible**
- We have `n = 4` piles
- Check: `(n - 1) % (K - 1) = (4 - 1) % (2 - 1) = 3 % 1 = 0` ✓
- Since the remainder is 0, we can merge all piles into one

**Step 2: Build prefix sum array**
- `s = [0, 3, 5, 9, 10]`
- This helps us quickly calculate range sums. For example, sum of stones[2:4] = s[4] - s[1] = 10 - 3 = 7

**Step 3: Initialize DP table**
- Create 3D array `f[i][j][m]` initialized to infinity
- Set base cases: `f[1][1][1] = 0`, `f[2][2][1] = 0`, `f[3][3][1] = 0`, `f[4][4][1] = 0`
- (Each single pile is already 1 pile with zero cost)

**Step 4: Fill DP table for increasing lengths**

*Length 2 subarrays:*
- For `[3, 2]` (i=1, j=2):
  - To get 2 piles: `f[1][2][2] = f[1][1][1] + f[2][2][1] = 0 + 0 = 0`
  - To get 1 pile: `f[1][2][1] = f[1][2][2] + (s[2] - s[0]) = 0 + 5 = 5`
- For `[2, 4]` (i=2, j=3):
  - To get 2 piles: `f[2][3][2] = 0`
  - To get 1 pile: `f[2][3][1] = 0 + 6 = 6`
- For `[4, 1]` (i=3, j=4):
  - To get 2 piles: `f[3][4][2] = 0`
  - To get 1 pile: `f[3][4][1] = 0 + 5 = 5`

*Length 3 subarrays:*
- For `[3, 2, 4]` (i=1, j=3):
  - To get 2 piles (split at h=1): `f[1][3][2] = f[1][1][1] + f[2][3][1] = 0 + 6 = 6`
  - To get 2 piles (split at h=2): `f[1][3][2] = min(6, f[1][2][1] + f[3][3][1]) = min(6, 5 + 0) = 5`
  - To get 1 pile: `f[1][3][1] = f[1][3][2] + (s[3] - s[0]) = 5 + 9 = 14`
- For `[2, 4, 1]` (i=2, j=4):
  - To get 2 piles: `f[2][4][2] = min(f[2][2][1] + f[3][4][1], f[2][3][1] + f[4][4][1]) = min(0 + 5, 6 + 0) = 5`
  - To get 1 pile: `f[2][4][1] = 5 + 7 = 12`

*Length 4 (entire array):*
- For `[3, 2, 4, 1]` (i=1, j=4):
  - To get 2 piles:
    - Split at h=1: `f[1][1][1] + f[2][4][1] = 0 + 12 = 12`
    - Split at h=2: `f[1][2][1] + f[3][4][1] = 5 + 5 = 10`
    - Split at h=3: `f[1][3][1] + f[4][4][1] = 14 + 0 = 14`
    - Minimum: `f[1][4][2] = 10`
  - To get 1 pile: `f[1][4][1] = f[1][4][2] + (s[4] - s[0]) = 10 + 10 = 20`

**Step 5: Return result**
The minimum cost to merge all stones into one pile is `f[1][4][1] = 20`.

This represents the optimal merging sequence:
1. Merge [3, 2] → [5] with cost 5
2. Merge [4, 1] → [5] with cost 5  
3. Merge [5, 5] → [10] with cost 10
Total cost: 5 + 5 + 10 = 20

## Python Solution

```python
class Solution:
    def mergeStones(self, stones: List[int], K: int) -> int:
        n = len(stones)
      
        # Check if it's possible to merge all stones into one pile
        # We need exactly (n-1)/(K-1) merge operations
        if (n - 1) % (K - 1) != 0:
            return -1
      
        # Build prefix sum array for quick range sum calculation
        # prefix_sum[i] = sum of stones[0] to stones[i-1]
        prefix_sum = [0]
        for stone in stones:
            prefix_sum.append(prefix_sum[-1] + stone)
      
        # dp[i][j][m] = minimum cost to merge stones[i-1] to stones[j-1] into m piles
        # Initialize with infinity (impossible states)
        dp = [[[float('inf')] * (K + 1) for _ in range(n + 1)] for _ in range(n + 1)]
      
        # Base case: single stone forms 1 pile with cost 0
        for i in range(1, n + 1):
            dp[i][i][1] = 0
      
        # Iterate over all possible subarray lengths
        for length in range(2, n + 1):
            # Iterate over all possible starting positions
            for start in range(1, n - length + 2):
                end = start + length - 1
              
                # Try to form different number of piles
                for num_piles in range(2, K + 1):
                    # Try different split points
                    for split in range(start, end, K - 1):
                        # Merge left part into 1 pile and right part into (num_piles - 1) piles
                        dp[start][end][num_piles] = min(
                            dp[start][end][num_piles],
                            dp[start][split][1] + dp[split + 1][end][num_piles - 1]
                        )
              
                # Merge K piles into 1 pile
                # The cost is the sum of all stones in the range
                dp[start][end][1] = dp[start][end][K] + prefix_sum[end] - prefix_sum[start - 1]
      
        # Return minimum cost to merge all stones into 1 pile
        return dp[1][n][1]
```

## Time and Space Complexity

**Time Complexity:** `O(n³)`

The algorithm uses dynamic programming with four nested loops:
- The outermost loop iterates through lengths from 2 to n: `O(n)` iterations
- The second loop iterates through starting positions: `O(n)` iterations
- The third loop iterates through `num_piles` from 2 to K: `O(K)` iterations
- The innermost loop steps `split` from `start` to `end - 1` in steps of `K - 1`: `O(n / K)` iterations

This gives `O(n * n * K * n / K) = O(n³)`. Without the step of `K - 1` (trying every split point), the bound would be `O(n³ * K)`.

Additionally, the prefix sum calculation takes `O(n)` time, but this doesn't affect the overall complexity.

**Space Complexity:** `O(n² * K)`

The algorithm uses a 3D DP array `dp` with dimensions `(n+1) × (n+1) × (K+1)`, which requires `O(n² * K)` space.

The prefix sum array `prefix_sum` uses `O(n)` additional space, but this is dominated by the DP array.

Therefore, the total space complexity is `O(n² * K)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Split Point Iteration: Every Split vs. Steps of K-1**

**Pitfall:** Assuming that the split loop must try every position, or, the other way round, assuming that trying every position gives wrong answers. Both loops below return the same result.

**Every split (correct, slower):**
```python
for split in range(start, end):  # Checking every possible split
    dp[start][end][num_piles] = min(
        dp[start][end][num_piles],
        dp[start][split][1] + dp[split + 1][end][num_piles - 1]
    )
```

**Steps of K-1 (correct, used in the solution):**
```python
for split in range(start, end, K - 1):  # Step by K-1
    dp[start][end][num_piles] = min(
        dp[start][end][num_piles],
        dp[start][split][1] + dp[split + 1][end][num_piles - 1]
    )
```

The left part `[start, split]` must be merged into 1 pile, which is only possible when its length is `1 + t * (K - 1)` for some `t >= 0`. For every other split, `dp[start][split][1]` is infinity and the candidate is ignored. Stepping by `K - 1` skips exactly those useless splits, which removes a factor of `K` from the running time. Starting the step at any position other than `start` would skip valid splits and give wrong answers.

### 2. **Misunderstanding the Impossibility Condition**

**Pitfall:** Developers might check `n % K == 0` or other incorrect conditions to determine if merging is possible.

**Why it's wrong:**
```python
# INCORRECT checks:
if n % K != 0:  # Wrong!
    return -1

if n % K == 1:  # Also wrong!
    return -1
```

**Correct approach:**
```python
# CORRECT check:
if (n - 1) % (K - 1) != 0:
    return -1
```

**Explanation:** Each merge operation takes `K` piles and produces 1 pile, reducing the total by `K-1`. To go from `n` piles to 1 pile, we need exactly `(n-1)/(K-1)` merge operations. This is only possible if `(n-1)` is divisible by `(K-1)`.

### 3. **Off-by-One Errors in Prefix Sum Calculation**

**Pitfall:** Using 0-indexed prefix sums with 1-indexed DP table leads to index confusion.

**Why it's wrong:**
```python
# INCORRECT: Mixing 0-indexed and 1-indexed
prefix_sum = list(accumulate(stones))  # 0-indexed
# Later in code:
cost = prefix_sum[end] - prefix_sum[start - 1]  # Off by one; IndexError when end = n
```

**Correct approach:**
```python
# CORRECT: Consistent 1-indexed approach
prefix_sum = [0]
for stone in stones:
    prefix_sum.append(prefix_sum[-1] + stone)
# Or using accumulate:
prefix_sum = list(accumulate(stones, initial=0))

# Now safe to use:
cost = prefix_sum[end] - prefix_sum[start - 1]
```

### 4. **Forgetting the Final Merge Cost**

**Pitfall:** Only calculating the cost to arrange stones into `K` piles without adding the final merge cost.

**Why it's wrong:**
```python
# INCORRECT: Missing the final merge cost
dp[start][end][1] = dp[start][end][K]  # Forgot to add merge cost!
```

**Correct approach:**
```python
# CORRECT: Include the cost of the final merge
dp[start][end][1] = dp[start][end][K] + prefix_sum[end] - prefix_sum[start - 1]
```

The cost to merge `K` piles into 1 includes both:
1. The cost to arrange into `K` piles (`dp[start][end][K]`)
2. The cost of merging those `K` piles (sum of all stones in the range)

