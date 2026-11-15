# 1458. Max Dot Product of Two Subsequences

> Source: https://algo.monster/liteproblems/1458

## Problem Statement

You are given two arrays `nums1` and `nums2`.

Your task is to find the maximum dot product between non-empty subsequences of `nums1` and `nums2` that have the same length.

The dot product of two sequences `[a1, a2, ..., ak]` and `[b1, b2, ..., bk]` is calculated as `a1*b1 + a2*b2 + ... + ak*bk`.

A subsequence is formed by deleting some (possibly zero) elements from the original array without changing the relative order of the remaining elements. For example, `[2,3,5]` is a subsequence of `[1,2,3,4,5]`, but `[1,5,3]` is not because it doesn't maintain the relative order.

The key requirements are:
- Both subsequences must be non-empty
- Both subsequences must have the same length
- You want to maximize the dot product

For example, if `nums1 = [2, 1, -2, 5]` and `nums2 = [3, 0, -6]`, you could:
- Choose subsequence `[2, 5]` from `nums1` and `[3, -6]` from `nums2`, giving dot product `2*3 + 5*(-6) = 6 - 30 = -24`
- Choose subsequence `[1]` from `nums1` and `[0]` from `nums2`, giving dot product `1*0 = 0`
- Choose subsequence `[5]` from `nums1` and `[3]` from `nums2`, giving dot product `5*3 = 15`

The maximum dot product would be 18 by choosing `[2, -2]` from `nums1` and `[3, -6]` from `nums2`, giving `2*3 + (-2)*(-6) = 6 + 12 = 18`.

## Intuition

When we need to find optimal subsequences from two arrays, [dynamic programming](/problems/dynamic_programming_intro) is often a natural choice because we can build up the solution by considering smaller subproblems.

Think about what choices we have at each step. For any position `i` in `nums1` and position `j` in `nums2`, we need to decide:
- Should we include both `nums1[i]` and `nums2[j]` in our subsequences?
- Or should we skip one of them?

This leads us to define our state as `f[i][j]` - the maximum dot product we can achieve using elements from the first `i` elements of `nums1` and the first `j` elements of `nums2`.

For each pair of positions `(i, j)`, we have three main choices:

1. **Skip the current element from `nums1`**: We use the best result from `f[i-1][j]`, which means we've already considered all ways to pair the first `i-1` elements of `nums1` with the first `j` elements of `nums2`.

2. **Skip the current element from `nums2`**: We use the best result from `f[i][j-1]`, which means we've already considered all ways to pair the first `i` elements of `nums1` with the first `j-1` elements of `nums2`.

3. **Pair the current elements together**: We multiply `nums1[i-1] * nums2[j-1]` and add it to the best dot product we could achieve with the previous elements. Here's a crucial insight: if the previous best dot product `f[i-1][j-1]` is negative, we might want to start fresh with just the current pair. That's why we use `max(0, f[i-1][j-1]) + nums1[i-1] * nums2[j-1]`.

The reason we take `max(0, f[i-1][j-1])` is because if all previous combinations gave us negative results, we're better off starting a new subsequence with just the current pair rather than adding to a negative sum.

By considering all these options at each step and taking the maximum, we ensure we find the optimal solution. The final answer is stored in `f[m][n]`, representing the maximum dot product using all available elements from both arrays.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement the solution using a 2D [dynamic programming](/problems/dynamic_programming_intro) table `f` where `f[i][j]` represents the maximum dot product between subsequences formed from the first `i` elements of `nums1` and the first `j` elements of `nums2`.

**Initialization:**
- Create a 2D table `f` with dimensions `(m+1) × (n+1)` where `m = len(nums1)` and `n = len(nums2)`
- Initialize all values to negative infinity (`-inf`) since we need to find the maximum and haven't computed any valid dot products yet
- The extra row and column (index 0) serve as base cases representing empty subsequences

**State Transition:**
For each position `(i, j)` where `1 ≤ i ≤ m` and `1 ≤ j ≤ n`, we compute:

```python
v = nums1[i-1] * nums2[j-1]  # Product of current elements
f[i][j] = max(
    f[i-1][j],                 # Skip nums1[i-1]
    f[i][j-1],                 # Skip nums2[j-1]
    max(0, f[i-1][j-1]) + v    # Include both current elements
)
```

The key insight in the third option is using `max(0, f[i-1][j-1]) + v`:
- If `f[i-1][j-1]` is positive, we extend the previous subsequence by adding the current pair
- If `f[i-1][j-1]` is negative or `-inf`, we start a new subsequence with just the current pair (equivalent to adding `v` to 0)

**Iteration Order:**
We iterate through each element of `nums1` (outer loop) and for each, iterate through all elements of `nums2` (inner loop). This ensures that when we compute `f[i][j]`, the values `f[i-1][j]`, `f[i][j-1]`, and `f[i-1][j-1]` have already been computed.

**Final Result:**
The answer is `f[m][n]`, which represents the maximum dot product considering all elements from both arrays.

**Time Complexity:** `O(m × n)` where `m` and `n` are the lengths of the two arrays
**Space Complexity:** `O(m × n)` for the DP table

## Example Walkthrough

Let's walk through a small example with `nums1 = [2, -1, 3]` and `nums2 = [1, -2]`.

We'll build a DP table `f` of size 4×3 (including base cases). Initially, all values are set to `-inf`.

**Step-by-step computation:**

For `i=1, j=1` (considering nums1[0]=2 and nums2[0]=1):
- Product `v = 2 * 1 = 2`
- `f[0][1]` = -inf (skip nums1[0])
- `f[1][0]` = -inf (skip nums2[0])
- `max(0, f[0][0]) + v = max(0, -inf) + 2 = 0 + 2 = 2`
- `f[1][1] = max(-inf, -inf, 2) = 2`

For `i=1, j=2` (considering nums1[0]=2 and nums2[1]=-2):
- Product `v = 2 * (-2) = -4`
- `f[0][2]` = -inf (skip nums1[0])
- `f[1][1]` = 2 (skip nums2[1])
- `max(0, f[0][1]) + v = max(0, -inf) + (-4) = 0 + (-4) = -4`
- `f[1][2] = max(-inf, 2, -4) = 2`

For `i=2, j=1` (considering nums1[1]=-1 and nums2[0]=1):
- Product `v = -1 * 1 = -1`
- `f[1][1]` = 2 (skip nums1[1])
- `f[2][0]` = -inf (skip nums2[0])
- `max(0, f[1][0]) + v = max(0, -inf) + (-1) = 0 + (-1) = -1`
- `f[2][1] = max(2, -inf, -1) = 2`

For `i=2, j=2` (considering nums1[1]=-1 and nums2[1]=-2):
- Product `v = -1 * (-2) = 2`
- `f[1][2]` = 2 (skip nums1[1])
- `f[2][1]` = 2 (skip nums2[1])
- `max(0, f[1][1]) + v = max(0, 2) + 2 = 2 + 2 = 4`
- `f[2][2] = max(2, 2, 4) = 4`

For `i=3, j=1` (considering nums1[2]=3 and nums2[0]=1):
- Product `v = 3 * 1 = 3`
- `f[2][1]` = 2 (skip nums1[2])
- `f[3][0]` = -inf (skip nums2[0])
- `max(0, f[2][0]) + v = max(0, -inf) + 3 = 0 + 3 = 3`
- `f[3][1] = max(2, -inf, 3) = 3`

For `i=3, j=2` (considering nums1[2]=3 and nums2[1]=-2):
- Product `v = 3 * (-2) = -6`
- `f[2][2]` = 4 (skip nums1[2])
- `f[3][1]` = 3 (skip nums2[1])
- `max(0, f[2][1]) + v = max(0, 2) + (-6) = 2 + (-6) = -4`
- `f[3][2] = max(4, 3, -4) = 4`

**Final DP table:**
```
       j=0   j=1   j=2
i=0   -inf  -inf  -inf
i=1   -inf    2     2
i=2   -inf    2     4
i=3   -inf    3     4
```

The maximum dot product is `f[3][2] = 4`, which corresponds to choosing subsequence `[2, -1]` from nums1 and subsequence `[1, -2]` from nums2, giving us `2*1 + (-1)*(-2) = 2 + 2 = 4`.

## Python Solution

```python
class Solution:
    def maxDotProduct(self, nums1: List[int], nums2: List[int]) -> int:
        # Get the lengths of both arrays
        m, n = len(nums1), len(nums2)
      
        # Initialize DP table with negative infinity
        # dp[i][j] represents the maximum dot product using first i elements from nums1
        # and first j elements from nums2
        dp = [[float('-inf')] * (n + 1) for _ in range(m + 1)]
      
        # Iterate through each element in nums1
        for i in range(1, m + 1):
            # Iterate through each element in nums2
            for j in range(1, n + 1):
                # Calculate the product of current elements
                current_product = nums1[i - 1] * nums2[j - 1]
              
                # Take maximum of three cases:
                # 1. Skip current element from nums1: dp[i-1][j]
                # 2. Skip current element from nums2: dp[i][j-1]
                # 3. Include current pair: max(0, dp[i-1][j-1]) + current_product
                #    (we take max with 0 to optionally start a new subsequence)
                dp[i][j] = max(
                    dp[i - 1][j],                                    # Skip nums1[i-1]
                    dp[i][j - 1],                                    # Skip nums2[j-1]
                    max(0, dp[i - 1][j - 1]) + current_product      # Include both elements
                )
      
        # Return the maximum dot product using all available elements
        return dp[m][n]
```

## Time and Space Complexity

The time complexity is `O(m × n)`, where `m` is the length of `nums1` and `n` is the length of `nums2`. This is because the code uses two nested loops: the outer loop iterates through all elements of `nums1` (from index 1 to `m`), and the inner loop iterates through all elements of `nums2` (from index 1 to `n`). Each cell `f[i][j]` is computed exactly once with constant time operations (multiplication and max comparisons), resulting in `m × n` total operations.

The space complexity is `O(m × n)`. The code creates a 2D dynamic programming table `f` with dimensions `(m + 1) × (n + 1)` to store the intermediate results. Each cell stores a single value representing the maximum dot product for the corresponding subproblems. The additional `+1` in each dimension accounts for the base cases (empty subsequences), but this doesn't change the asymptotic complexity, which remains `O(m × n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall: Incorrect Initialization Leading to Wrong Results for All-Negative Arrays

**The Problem:**
The current solution initializes the DP table with `float('-inf')` and uses the recurrence relation:
```python
dp[i][j] = max(
    dp[i - 1][j],
    dp[i][j - 1],
    max(0, dp[i - 1][j - 1]) + current_product
)
```

This works correctly in most cases, but there's a subtle issue. When `dp[i-1][j-1]` is `-inf` (which happens when we haven't selected any pairs yet), the expression `max(0, dp[i-1][j-1]) + current_product` evaluates to `0 + current_product = current_product`, which correctly starts a new subsequence.

However, the pitfall occurs when someone tries to "optimize" or modify this initialization. A common mistake is to initialize `dp[0][0] = 0` thinking it represents "no elements selected, so dot product is 0". This breaks the logic because:
- The problem requires **non-empty** subsequences
- Setting `dp[0][0] = 0` would allow the algorithm to return 0 even when all possible products are negative
- For example, with `nums1 = [-1]` and `nums2 = [-1]`, the only valid subsequence pair gives dot product = 1, but incorrect initialization might return 0

**Example of Incorrect Code:**
```python
# WRONG: This initialization can lead to incorrect results
dp = [[0] * (n + 1) for _ in range(m + 1)]  # Initializing with 0 instead of -inf
```

**The Solution:**
Maintain the initialization with negative infinity for all positions except when computing actual products. This ensures:
1. Empty subsequences are never considered valid (they have value `-inf`)
2. The first product included will always be taken regardless of its sign
3. The algorithm correctly handles all-negative or all-positive cases

**Correct Approach:**
```python
# Initialize everything with -inf
dp = [[float('-inf')] * (n + 1) for _ in range(m + 1)]

# In the recurrence, max(0, dp[i-1][j-1]) handles both:
# - Starting a new subsequence (when dp[i-1][j-1] is -inf)
# - Extending an existing subsequence (when dp[i-1][j-1] is a valid value)
```

**Alternative Clear Solution:**
If the logic seems confusing, you can make it more explicit:
```python
for i in range(1, m + 1):
    for j in range(1, n + 1):
        current_product = nums1[i - 1] * nums2[j - 1]
      
        # Start with skipping options
        dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
      
        # Include current pair
        if dp[i - 1][j - 1] == float('-inf'):
            # First pair in subsequence
            dp[i][j] = max(dp[i][j], current_product)
        else:
            # Extend existing subsequence
            dp[i][j] = max(dp[i][j], dp[i - 1][j - 1] + current_product)
```

This makes it clear that we're handling the "first pair" case separately from the "extending subsequence" case.

