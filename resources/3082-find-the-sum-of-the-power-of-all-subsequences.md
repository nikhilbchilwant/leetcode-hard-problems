# 3082. Find the Sum of the Power of All Subsequences

> Source: https://algo.monster/liteproblems/3082

## Problem Statement

You are given an integer array `nums` of length `n` and a positive integer `k`.

The problem introduces a concept called the **power** of an array. The power is defined as the total number of subsequences across all possible subsequences of `nums` that have a sum equal to `k`.

To understand this better, let's break it down:
1. First, consider all possible subsequences of the array `nums` (including the empty subsequence and the full array itself)
2. For each of these subsequences, count how many of its own subsequences sum to `k`
3. The power is the sum of all these counts

For example, if `nums = [1, 2, 3]` and `k = 3`:
- The subsequence `[1, 2]` has one subsequence that sums to 3: `[1, 2]` itself
- The subsequence `[3]` has one subsequence that sums to 3: `[3]` itself  
- The subsequence `[1, 2, 3]` has two subsequences that sum to 3: `[1, 2]` and `[3]`
- Other subsequences contribute 0 to the power

Your task is to calculate the sum of power across all subsequences of `nums` and return the result modulo `10^9 + 7` (since the answer can be very large).

The key insight is that for each element in `nums`, we need to consider:
- Whether it's included in the subsequence `S` we're examining
- If it's in `S`, whether it's also part of the subsequence `T` (within `S`) that sums to `k`

This leads to tracking the number of ways to form subsequences with specific sums, accounting for elements that are present but not used in the sum versus elements that actively contribute to reaching the target sum `k`.

## Intuition

The key insight is recognizing that we need to count subsequences within subsequences. This double-counting problem initially seems complex, but we can simplify it by thinking about the contribution of each element.

Consider what happens when we process each element in `nums`. For any element `x`, we need to track:
- All the ways it can be part of a subsequence `S`
- Within each `S` that contains `x`, whether `x` contributes to a sum of `k`

This naturally leads us to [dynamic programming](/problems/dynamic_programming_intro) where we track the number of ways to achieve each possible sum using the elements we've seen so far.

The critical observation is that for each element `x` at position `i`, there are three scenarios:
1. `x` is not included in subsequence `S` at all
2. `x` is in subsequence `S` but doesn't contribute to the sum (it's just "present" but not used)
3. `x` is in subsequence `S` and actively contributes to achieving sum `k`

Why do we care about scenario 2? Because when an element is in `S` but not used for the sum, it still affects the count - it creates a different subsequence `S` that might have its own subsequences summing to `k`.

This realization leads to the pattern: when we don't use element `x` for the sum (scenarios 1 and 2), we're essentially doubling the previous count `f[i-1][j]` because `x` can either be absent or present-but-unused. When we do use `x` (scenario 3), we add the ways to make sum `j-x` from previous elements.

The state transition `f[i][j] = f[i-1][j] * 2 + f[i-1][j-x]` elegantly captures all three scenarios:
- The `f[i-1][j] * 2` term accounts for `x` being either absent from `S` or in `S` but not contributing to the sum
- The `f[i-1][j-x]` term accounts for `x` being in `S` and contributing to the sum

This approach ensures we count every valid configuration exactly once, giving us the total "power" of the array.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement the solution using a 2D [dynamic programming](/problems/dynamic_programming_intro) approach. Let's define `f[i][j]` as the number of ways to form subsequences using the first `i` elements of `nums` such that each subsequence has a sum equal to `j`.

**Initialization:**
- Create a 2D array `f` of size `(n+1) × (k+1)` where `n` is the length of `nums`
- Set `f[0][0] = 1`, representing one way to achieve sum 0 with 0 elements (the empty subsequence)
- All other entries start at 0

**State Transition:**
For each element `x` at position `i` (1-indexed), and for each possible sum `j` from 0 to `k`:

```
f[i][j] = f[i-1][j] × 2 + f[i-1][j-x]  (if j >= x)
f[i][j] = f[i-1][j] × 2                 (if j < x)
```

The transition formula captures three scenarios:
- Element `x` is not in subsequence `S`: contributes `f[i-1][j]`
- Element `x` is in subsequence `S` but not in subsequence `T` (not used for sum): contributes `f[i-1][j]`
- Element `x` is in both `S` and `T` (used for sum): contributes `f[i-1][j-x]`

The first two scenarios together give us `f[i-1][j] × 2`.

**Implementation Details:**
```python
class Solution:
    def sumOfPower(self, nums: List[int], k: int) -> int:
        mod = 10**9 + 7
        n = len(nums)
        f = [[0] * (k + 1) for _ in range(n + 1)]
        f[0][0] = 1
      
        for i, x in enumerate(nums, 1):
            for j in range(k + 1):
                # Cases where x is not used for the sum
                f[i][j] = f[i - 1][j] * 2 % mod
              
                # Case where x is used for the sum
                if j >= x:
                    f[i][j] = (f[i][j] + f[i - 1][j - x]) % mod
      
        return f[n][k]
```

**Time Complexity:** `O(n × k)` where `n` is the length of the array and `k` is the target sum. We iterate through each element and for each element, we update all possible sums from 0 to `k`.

**Space Complexity:** `O(n × k)` for the 2D DP table. This can be optimized to `O(k)` using rolling arrays since we only need the previous row to compute the current row.

The final answer `f[n][k]` represents the total number of subsequence pairs `(S, T)` where `S` is a subsequence of `nums` and `T` is a subsequence of `S` with sum equal to `k`, which is exactly the "power" of the array as defined in the problem.

## Example Walkthrough

Let's walk through a small example with `nums = [1, 2, 3]` and `k = 3`.

We'll build our DP table `f[i][j]` where `f[i][j]` represents the number of ways to form subsequences using the first `i` elements with sum `j`.

**Initial State:**
```
f[0][0] = 1 (empty subsequence)
f[0][1] = f[0][2] = f[0][3] = 0
```

**Processing element 1 (value = 1):**
For each sum j from 0 to 3:
- j = 0: `f[1][0] = f[0][0] × 2 + f[0][-1]` = 1 × 2 + 0 = 2
  - Two ways: {} (1 not in S), {1} with sum 0 (1 in S but not used)
- j = 1: `f[1][1] = f[0][1] × 2 + f[0][0]` = 0 × 2 + 1 = 1
  - One way: {1} with sum 1 (1 used for sum)
- j = 2: `f[1][2] = f[0][2] × 2` = 0 × 2 = 0
- j = 3: `f[1][3] = f[0][3] × 2` = 0 × 2 = 0

**Processing element 2 (value = 2):**
- j = 0: `f[2][0] = f[1][0] × 2` = 2 × 2 = 4
  - Four ways: {}, {1}, {2}, {1,2} all with sum 0
- j = 1: `f[2][1] = f[1][1] × 2` = 1 × 2 = 2
  - Two ways: {1} with sum 1, {1,2} with sum 1 (only 1 used)
- j = 2: `f[2][2] = f[1][2] × 2 + f[1][0]` = 0 × 2 + 2 = 2
  - Two ways: {2} with sum 2, {1,2} with sum 2 (only 2 used)
- j = 3: `f[2][3] = f[1][3] × 2 + f[1][1]` = 0 × 2 + 1 = 1
  - One way: {1,2} with sum 3 (both used)

**Processing element 3 (value = 3):**
- j = 0: `f[3][0] = f[2][0] × 2` = 4 × 2 = 8
- j = 1: `f[3][1] = f[2][1] × 2` = 2 × 2 = 4
- j = 2: `f[3][2] = f[2][2] × 2` = 2 × 2 = 4
- j = 3: `f[3][3] = f[2][3] × 2 + f[2][0]` = 1 × 2 + 4 = 6

**Final Answer:** `f[3][3] = 6`

This means there are 6 total subsequence pairs (S, T) where S is a subsequence of [1,2,3] and T is a subsequence of S with sum 3:
1. S = {1,2}, T = {1,2}
2. S = {3}, T = {3}
3. S = {1,3}, T = {3}
4. S = {2,3}, T = {3}
5. S = {1,2,3}, T = {1,2}
6. S = {1,2,3}, T = {3}

The DP approach correctly counts all these contributions by tracking how elements can be included in S but not used for the sum versus being used to achieve the target sum k.

## Python Solution

```python
class Solution:
    def sumOfPower(self, nums: List[int], k: int) -> int:
        # Define modulo for preventing integer overflow
        MOD = 10**9 + 7
      
        # Get the length of input array
        n = len(nums)
      
        # Initialize DP table
        # dp[i][j] represents the sum of power considering first i elements
        # where subsequences sum to j
        dp = [[0] * (k + 1) for _ in range(n + 1)]
      
        # Base case: empty subsequence with sum 0 has power 1
        dp[0][0] = 1
      
        # Iterate through each element in nums
        for i, num in enumerate(nums, 1):
            # For each possible sum from 0 to k
            for target_sum in range(k + 1):
                # Case 1: Don't include current element
                # All subsequences from previous state contribute with doubled power
                # (since current element can be either included or excluded in other positions)
                dp[i][target_sum] = dp[i - 1][target_sum] * 2 % MOD
              
                # Case 2: Include current element if it doesn't exceed target sum
                if target_sum >= num:
                    # Add the power from subsequences that sum to (target_sum - num)
                    dp[i][target_sum] = (dp[i][target_sum] + dp[i - 1][target_sum - num]) % MOD
      
        # Return the total power for subsequences that sum to k using all n elements
        return dp[n][k]
```

## Time and Space Complexity

The time complexity is `O(n × k)`, where `n` is the length of the array `nums` and `k` is the given positive integer. This is because we have two nested loops: the outer loop iterates through all `n` elements of `nums` (from index 1 to n), and the inner loop iterates through all possible sums from 0 to `k`. Each operation inside the nested loops takes constant time `O(1)`.

The space complexity is `O(n × k)`. We create a 2D DP table `f` with dimensions `(n + 1) × (k + 1)` to store the intermediate results. This table requires `O((n + 1) × (k + 1)) = O(n × k)` space. The additional variables used (`mod`, `n`, `i`, `j`, `x`) only require constant space `O(1)`, so the overall space complexity is dominated by the DP table.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding the Problem Definition**
The most common pitfall is misinterpreting what "power" means. Many initially think it's about counting subsequences that sum to `k`, but it's actually about counting subsequence pairs (S, T) where S is a subsequence of nums and T is a subsequence of S that sums to k.

**Wrong interpretation:**
```python
# This only counts subsequences that sum to k
def wrongApproach(nums, k):
    count = 0
    # Generate all subsequences and check if sum equals k
    for subsequence in all_subsequences(nums):
        if sum(subsequence) == k:
            count += 1
    return count
```

**Correct understanding:** Each element can be in three states:
- Not in subsequence S at all
- In subsequence S but not used in the sum (not in T)
- In subsequence S and used in the sum (in T)

### 2. **Incorrect DP State Transition**
A common mistake is forgetting to multiply by 2 when an element is not used for the sum, missing the fact that the element can either be present in S but not in T, or not present in S at all.

**Incorrect:**
```python
# Missing the factor of 2
for i, num in enumerate(nums, 1):
    for target_sum in range(k + 1):
        dp[i][target_sum] = dp[i - 1][target_sum]  # Wrong! Missing multiplication by 2
        if target_sum >= num:
            dp[i][target_sum] = (dp[i][target_sum] + dp[i - 1][target_sum - num]) % MOD
```

**Correct:**
```python
for i, num in enumerate(nums, 1):
    for target_sum in range(k + 1):
        dp[i][target_sum] = dp[i - 1][target_sum] * 2 % MOD  # Correct: multiply by 2
        if target_sum >= num:
            dp[i][target_sum] = (dp[i][target_sum] + dp[i - 1][target_sum - num]) % MOD
```

### 3. **Forgetting Modulo Operations**
Since the result can be very large, forgetting to apply modulo at each step can cause integer overflow or incorrect results.

**Incorrect:**
```python
# Missing modulo operations
dp[i][target_sum] = dp[i - 1][target_sum] * 2 + dp[i - 1][target_sum - num]
```

**Correct:**
```python
dp[i][target_sum] = dp[i - 1][target_sum] * 2 % MOD
if target_sum >= num:
    dp[i][target_sum] = (dp[i][target_sum] + dp[i - 1][target_sum - num]) % MOD
```

### 4. **Space Optimization Pitfall**
When optimizing space from O(n×k) to O(k) using a 1D array, a common mistake is updating values in the wrong order, causing previously computed values to be overwritten before they're used.

**Incorrect space optimization:**
```python
# Wrong: iterating forward causes values to be overwritten prematurely
dp = [0] * (k + 1)
dp[0] = 1
for num in nums:
    for target_sum in range(k + 1):  # Wrong direction!
        dp[target_sum] = dp[target_sum] * 2 % MOD
        if target_sum >= num:
            dp[target_sum] = (dp[target_sum] + dp[target_sum - num]) % MOD
```

**Correct space optimization:**
```python
# Correct: iterate backward to avoid overwriting needed values
dp = [0] * (k + 1)
dp[0] = 1
for num in nums:
    for target_sum in range(k, -1, -1):  # Iterate backward
        dp[target_sum] = dp[target_sum] * 2 % MOD
        if target_sum >= num:
            dp[target_sum] = (dp[target_sum] + dp[target_sum - num]) % MOD
```

### 5. **Off-by-One Errors in Indexing**
Using 0-based vs 1-based indexing incorrectly can lead to accessing wrong array positions.

**Solution:** Be consistent with indexing. The provided solution uses 1-based indexing for the DP table rows (with `enumerate(nums, 1)`) to make the logic clearer, but you could also use 0-based indexing throughout as long as you're consistent.

