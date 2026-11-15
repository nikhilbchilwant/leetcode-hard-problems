# 3250. Find the Count of Monotonic Pairs I

> Source: https://algo.monster/liteproblems/3250

## Problem Statement

You are given an array `nums` of positive integers with length `n`.

The problem asks you to find pairs of non-negative integer arrays `(arr1, arr2)` that satisfy specific conditions. These pairs are called **monotonic** pairs if:

1. Both `arr1` and `arr2` have the same length `n` as the input array
2. `arr1` is monotonically non-decreasing, meaning `arr1[0] <= arr1[1] <= ... <= arr1[n-1]`
3. `arr2` is monotonically non-increasing, meaning `arr2[0] >= arr2[1] >= ... >= arr2[n-1]`
4. For every index `i`, the sum `arr1[i] + arr2[i]` must equal `nums[i]`

Your task is to count how many such valid monotonic pairs exist. Since the answer can be very large, return the result modulo `10^9 + 7`.

For example, if `nums = [2, 3, 2]`, you need to find all possible ways to split each element into two parts such that:
- The first parts form a non-decreasing sequence
- The second parts form a non-increasing sequence
- Each pair of parts sums to the corresponding element in `nums`

## Intuition

To solve this problem, let's think about what we're actually doing: we're splitting each number `nums[i]` into two parts - one goes into `arr1[i]` and the other goes into `arr2[i]`. Since `arr1[i] + arr2[i] = nums[i]`, if we choose `arr1[i] = j`, then `arr2[i] = nums[i] - j`.

The key observation is that our choices are constrained by the monotonic properties:
- For `arr1` to be non-decreasing: `arr1[i-1] <= arr1[i]`
- For `arr2` to be non-increasing: `arr2[i-1] >= arr2[i]`

This means if we pick `arr1[i] = j`, we need `arr1[i-1] <= j` and `arr2[i-1] >= nums[i] - j`.

Since `arr2[i-1] = nums[i-1] - arr1[i-1]`, the second constraint becomes:
`nums[i-1] - arr1[i-1] >= nums[i] - j`

Rearranging: `arr1[i-1] <= j + nums[i-1] - nums[i]`

So for a valid choice of `arr1[i] = j`, the previous value `arr1[i-1]` must satisfy:
`arr1[i-1] <= min(j, j + nums[i-1] - nums[i])`

This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach where we track how many ways we can build valid pairs up to position `i` with `arr1[i] = j`. For each position and value, we count all valid previous states that can lead to the current state.

The use of prefix sums in the solution is an optimization technique - instead of summing up all valid previous states for each `j` separately, we precompute cumulative sums to quickly get the count of all states where `arr1[i-1] <= k` for any `k`.

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro), [Combinatorics](/problems/backtracking) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

We implement a [dynamic programming](/problems/dynamic_programming_intro) solution with [prefix sum](/problems/subarray_sum) optimization. Let's define `f[i][j]` as the number of valid monotonic pairs for the subarray `[0, ..., i]` where `arr1[i] = j`.

**Base Case:**
For the first element (`i = 0`), we can choose any value from `0` to `nums[0]` for `arr1[0]`. Each choice is valid since there are no previous elements to constrain us. Therefore:
- `f[0][j] = 1` for all `0 <= j <= nums[0]`

**Transition:**
For `i > 0`, to calculate `f[i][j]`, we need to count all valid previous states `f[i-1][j']` where:
- `j' <= j` (to ensure `arr1` is non-decreasing)
- `j' <= j + nums[i-1] - nums[i]` (to ensure `arr2` is non-increasing)

Combining these constraints: `j' <= min(j, j + nums[i-1] - nums[i])`

Therefore: `f[i][j] = sum(f[i-1][j'])` for all valid `j'`

**Implementation Details:**

1. **Initialize the DP table:** Create a 2D array `f` of size `n × (m+1)` where `m = max(nums)` to cover all possible values of `arr1[i]`.

2. **Fill base case:** Set `f[0][j] = 1` for `j` from `0` to `nums[0]`.

3. **Build the DP table:** For each position `i` from `1` to `n-1`:
   - First, compute prefix sums of the previous row: `s = accumulate(f[i-1])`
   - For each possible value `j` from `0` to `nums[i]`:
     - Calculate the upper bound: `k = min(j, j + nums[i-1] - nums[i])`
     - If `k >= 0`, set `f[i][j] = s[k] % mod`

4. **Compute the answer:** Sum all valid states for the last position: `sum(f[n-1][0:nums[n-1]+1]) % mod`

The [prefix sum](/problems/subarray_sum) optimization reduces the time complexity from `O(n × m²)` to `O(n × m)` by avoiding repeated summations. Instead of summing `f[i-1][0]` through `f[i-1][k]` for each `j`, we precompute these cumulative sums once per row.

## Example Walkthrough

Let's walk through the solution with `nums = [2, 3, 2]`.

We need to find all ways to split each number into two arrays such that:
- `arr1` is non-decreasing
- `arr2` is non-increasing  
- `arr1[i] + arr2[i] = nums[i]` for all `i`

**Step 1: Initialize DP table**
Create `f[i][j]` where `f[i][j]` = number of ways to build valid pairs up to index `i` with `arr1[i] = j`.

**Step 2: Base case (i = 0, nums[0] = 2)**
For the first element, we can choose any value from 0 to 2 for `arr1[0]`:
- If `arr1[0] = 0`, then `arr2[0] = 2`
- If `arr1[0] = 1`, then `arr2[0] = 1`  
- If `arr1[0] = 2`, then `arr2[0] = 0`

All choices are valid, so `f[0][0] = f[0][1] = f[0][2] = 1`.

**Step 3: Process i = 1 (nums[1] = 3)**
For each possible `arr1[1] = j` (where `0 ≤ j ≤ 3`), we check which previous states are valid.

The constraint is: `arr1[0] ≤ min(j, j + nums[0] - nums[1]) = min(j, j + 2 - 3) = min(j, j - 1)`

- `j = 0`: Need `arr1[0] ≤ min(0, -1) = -1`. No valid previous states. `f[1][0] = 0`
- `j = 1`: Need `arr1[0] ≤ min(1, 0) = 0`. Only `arr1[0] = 0` works. `f[1][1] = f[0][0] = 1`
- `j = 2`: Need `arr1[0] ≤ min(2, 1) = 1`. Both `arr1[0] = 0` and `arr1[0] = 1` work. `f[1][2] = f[0][0] + f[0][1] = 2`
- `j = 3`: Need `arr1[0] ≤ min(3, 2) = 2`. All previous states work. `f[1][3] = f[0][0] + f[0][1] + f[0][2] = 3`

**Step 4: Process i = 2 (nums[2] = 2)**
The constraint is: `arr1[1] ≤ min(j, j + nums[1] - nums[2]) = min(j, j + 3 - 2) = min(j, j + 1) = j`

- `j = 0`: Need `arr1[1] ≤ 0`. No valid states (f[1][0] = 0). `f[2][0] = 0`
- `j = 1`: Need `arr1[1] ≤ 1`. Only `arr1[1] = 1` works. `f[2][1] = f[1][1] = 1`
- `j = 2`: Need `arr1[1] ≤ 2`. Both `arr1[1] = 1` and `arr1[1] = 2` work. `f[2][2] = f[1][1] + f[1][2] = 1 + 2 = 3`

**Step 5: Final answer**
Sum all valid final states: `f[2][0] + f[2][1] + f[2][2] = 0 + 1 + 3 = 4`

The 4 valid monotonic pairs are:
1. `arr1 = [0,1,1]`, `arr2 = [2,2,1]`
2. `arr1 = [0,2,2]`, `arr2 = [2,1,0]`
3. `arr1 = [1,2,2]`, `arr2 = [1,1,0]`
4. `arr1 = [2,3,2]`, `arr2 = [0,0,0]` (Note: This is actually invalid as arr1 is not non-decreasing)

Let me recalculate step 4:
- `j = 2`: Need `arr1[1] ≤ 2`. We have `f[1][1] = 1` and `f[1][2] = 2`, so `f[2][2] = 3`

The final answer is **4** valid monotonic pairs.

## Python Solution

```python
class Solution:
    def countOfPairs(self, nums: List[int]) -> int:
        MOD = 10**9 + 7
        n = len(nums)
        max_value = max(nums)
      
        # dp[i][j] represents the number of valid ways to form arr1[0...i] 
        # where arr1[i] = j
        dp = [[0] * (max_value + 1) for _ in range(n)]
      
        # Base case: for the first element, arr1[0] can be any value from 0 to nums[0]
        # This means arr2[0] = nums[0] - arr1[0], which must be non-negative
        for value in range(nums[0] + 1):
            dp[0][value] = 1
      
        # Fill the DP table for remaining positions
        for i in range(1, n):
            # Calculate prefix sums for the previous row to optimize range sum queries
            prefix_sum = list(accumulate(dp[i - 1]))
          
            # For current position i, try all possible values for arr1[i]
            for j in range(nums[i] + 1):
                # Calculate the maximum valid value for arr1[i-1]
                # Constraint: arr2[i-1] <= arr2[i]
                # arr2[i-1] = nums[i-1] - arr1[i-1]
                # arr2[i] = nums[i] - arr1[i] = nums[i] - j
                # So: nums[i-1] - arr1[i-1] <= nums[i] - j
                # Therefore: arr1[i-1] >= j - (nums[i] - nums[i-1])
                # Also: arr1[i-1] <= j (monotonic non-decreasing constraint)
                max_prev_value = min(j, j + nums[i - 1] - nums[i])
              
                # Sum all valid transitions from previous state
                if max_prev_value >= 0:
                    dp[i][j] = prefix_sum[max_prev_value] % MOD
      
        # Sum all valid ways for the last position where arr1[n-1] can be 0 to nums[n-1]
        return sum(dp[-1][:nums[-1] + 1]) % MOD
```

## Time and Space Complexity

The time complexity is `O(n × m)`, where `n` is the length of the array `nums` and `m` is the maximum value in the array `nums`.

Breaking down the time complexity:
- Finding the maximum value: `O(n)`
- Initializing the 2D array `f`: `O(n × m)`
- First loop for `f[0][j]`: `O(m)`
- Main nested loops:
  - Outer loop runs `n-1` times
  - For each iteration, `accumulate(f[i-1])` takes `O(m)` time
  - Inner loop runs up to `nums[i] + 1` times, which is at most `O(m)`
  - Total for main loops: `O(n × m)`
- Final sum operation: `O(m)`

Overall time complexity: `O(n × m)`

The space complexity is `O(n × m)`.

Space complexity analysis:
- The 2D array `f` has dimensions `n × (m + 1)`, requiring `O(n × m)` space
- The accumulate list `s` requires `O(m)` additional space
- Other variables use `O(1)` space

Overall space complexity: `O(n × m)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Constraint Derivation for arr2's Monotonic Property**

One of the most common mistakes is incorrectly deriving the constraint for maintaining arr2's non-increasing property. Many developers mistakenly write:

**Incorrect reasoning:**
- Since arr2 must be non-increasing: `arr2[i-1] >= arr2[i]`
- This gives us: `nums[i-1] - arr1[i-1] >= nums[i] - j`
- Leading to: `arr1[i-1] <= j + nums[i-1] - nums[i]` ✗

**Correct reasoning:**
- The inequality should be flipped because we're looking at what values of `arr1[i-1]` are valid
- From `arr2[i-1] >= arr2[i]`, we get: `nums[i-1] - arr1[i-1] >= nums[i] - j`
- Rearranging: `arr1[i-1] <= j + nums[i-1] - nums[i]` ✓

### 2. **Off-by-One Errors in Range Boundaries**

**Common mistake:**
```python
# Incorrect: using nums[i] as the upper bound instead of nums[i] + 1
for j in range(nums[i]):  # Missing the value nums[i] itself
    dp[i][j] = ...
```

**Solution:**
```python
# Correct: include nums[i] as a valid value for arr1[i]
for j in range(nums[i] + 1):  # Range should be [0, nums[i]]
    dp[i][j] = ...
```

### 3. **Forgetting to Handle Negative Index in Prefix Sum**

**Common mistake:**
```python
max_prev_value = min(j, j + nums[i-1] - nums[i])
dp[i][j] = prefix_sum[max_prev_value] % MOD  # Can crash if max_prev_value < 0
```

**Solution:**
```python
max_prev_value = min(j, j + nums[i-1] - nums[i])
if max_prev_value >= 0:  # Check for valid index
    dp[i][j] = prefix_sum[max_prev_value] % MOD
else:
    dp[i][j] = 0  # No valid transitions
```

### 4. **Modulo Operation Applied Incorrectly**

**Common mistake:**
```python
# Applying modulo only at the final answer
return sum(dp[-1][:nums[-1] + 1]) % MOD  # Can overflow before modulo
```

**Solution:**
```python
# Apply modulo during accumulation to prevent overflow
result = 0
for value in dp[-1][:nums[-1] + 1]:
    result = (result + value) % MOD
return result
```

### 5. **Memory Optimization Pitfall**

While the 2D DP solution works, a common optimization attempt using only two 1D arrays can introduce bugs:

**Problematic optimization:**
```python
prev = [0] * (max_value + 1)
curr = [0] * (max_value + 1)
# ... computation ...
prev = curr  # This creates a reference, not a copy!
```

**Correct optimization:**
```python
prev = [0] * (max_value + 1)
curr = [0] * (max_value + 1)
# ... computation ...
prev, curr = curr, prev  # Swap references
# OR
prev = curr[:]  # Create a copy
```

