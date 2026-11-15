# 2518. Number of Great Partitions

> Source: https://algo.monster/liteproblems/2518

## Problem Statement

You have an array `nums` containing positive integers and an integer `k`. Your task is to split this array into two groups where:

1. **Every element must belong to exactly one group** - you can't leave any element out or put it in both groups
2. **The order of elements within each group must be preserved** - if element at index `i` comes before element at index `j` in the original array, and both are in the same group, then element `i` must come before element `j` in that group
3. **Both groups must have a sum ≥ k** - this is what makes a partition "great"

You need to count how many different ways you can create such great partitions. Two partitions are considered different if at least one element is placed in a different group between them.

For example, if `nums = [1, 2, 3]` and `k = 3`:
- Partition `{[1, 2], [3]}` is great because sum of first group is `1+2=3 ≥ 3` and sum of second group is `3 ≥ 3`
- Partition `{[1], [2, 3]}` is NOT great because sum of first group is `1 < 3`

The answer should be returned modulo `10^9 + 7` since the number of partitions can be very large.

## Intuition

The first observation is that if the total sum of all elements is less than `2*k`, it's impossible to create any great partition since we need both groups to have sum at least `k`. This gives us our base case.

Now, for counting valid partitions, we could try to directly count partitions where both groups have sum ≥ `k`, but this is complex. Instead, let's think inversely - what if we count the invalid partitions and subtract them from the total?

Each element can go into either group 1 or group 2, giving us `2^n` total possible partitions. Among these, the invalid ones are those where at least one group has sum < `k`.

Here's the key insight: if group 1 has sum < `k`, then we can use [dynamic programming](/problems/dynamic_programming_intro) to count how many ways we can select elements to form group 1 with sum in range `[0, k-1]`. Similarly for group 2.

We can use a DP table `f[i][j]` where:
- `i` represents considering the first `i` elements
- `j` represents achieving a sum of exactly `j`
- `f[i][j]` counts the number of ways to select elements from the first `i` elements to get sum `j`

The DP transition is classic subset sum:
- Either we don't include element `i`: `f[i][j] = f[i-1][j]`
- Or we include element `i` (if `j >= nums[i-1]`): `f[i][j] += f[i-1][j-nums[i-1]]`

After building the DP table, `sum(f[n][0] to f[n][k-1])` gives us the number of ways to form a group with sum < `k`. Since either group 1 or group 2 could be the one with sum < `k`, we multiply by 2. But wait - we're double counting the cases where BOTH groups have sum < `k`, so we need to be careful with our subtraction.

The final answer becomes: `total_partitions - 2 * (partitions_with_one_group_sum_less_than_k)`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**Step 1: Check if a great partition is possible**
```python
if sum(nums) < k * 2:
    return 0
```
If the total sum is less than `2*k`, it's impossible to have both groups with sum ≥ `k`, so we return 0.

**Step 2: Initialize variables and DP table**
```python
mod = 10**9 + 7
n = len(nums)
f = [[0] * k for _ in range(n + 1)]
f[0][0] = 1
ans = 1
```
- We create a 2D DP table `f` with dimensions `(n+1) × k`
- `f[i][j]` represents the number of ways to select elements from the first `i` elements to achieve sum exactly `j`
- `f[0][0] = 1` because there's one way to get sum 0 with 0 elements (select nothing)
- `ans` starts at 1 and will track the total number of partitions (`2^n`)

**Step 3: Fill the DP table using subset sum pattern**
```python
for i in range(1, n + 1):
    ans = ans * 2 % mod
    for j in range(k):
        f[i][j] = f[i - 1][j]
        if j >= nums[i - 1]:
            f[i][j] = (f[i][j] + f[i - 1][j - nums[i - 1]]) % mod
```
For each element `nums[i-1]`:
- Update `ans` by multiplying by 2 (each element can go to either group)
- For each possible sum `j` from 0 to `k-1`:
  - First, copy the count without including current element: `f[i][j] = f[i-1][j]`
  - If including current element is valid (`j >= nums[i-1]`), add the count: `f[i][j] += f[i-1][j-nums[i-1]]`

**Step 4: Calculate the final answer**
```python
return (ans - sum(f[-1]) * 2 + mod) % mod
```
- `sum(f[-1])` gives the total number of ways to form a subset with sum in range `[0, k-1]`
- These represent the ways to form group 1 with sum < `k`
- Since either group could have sum < `k`, we multiply by 2
- Subtract from total partitions: `ans - sum(f[-1]) * 2`
- Add `mod` before taking modulo to handle potential negative values

The clever part is that we only track sums up to `k-1` in our DP table since we only care about counting invalid partitions (those with sum < `k`). This optimization keeps space complexity at `O(n*k)` instead of tracking all possible sums.

## Example Walkthrough

Let's walk through a concrete example with `nums = [1, 2, 3, 4]` and `k = 5`.

**Step 1: Check feasibility**
- Total sum = `1 + 2 + 3 + 4 = 10`
- Since `10 ≥ 2*5`, great partitions are possible

**Step 2: Build DP table to count invalid partitions**

We'll build a DP table `f[i][j]` where rows represent considering first `i` elements and columns represent achieving sum `j` (from 0 to k-1 = 4).

Initial state: `f[0][0] = 1` (one way to get sum 0 with 0 elements)

```
    j: 0  1  2  3  4
i=0:   1  0  0  0  0
```

**Processing element 1 (value = 1):**
- `f[1][0] = f[0][0] = 1` (don't include 1)
- `f[1][1] = f[0][1] + f[0][0] = 0 + 1 = 1` (include 1)
- `f[1][2] = f[0][2] = 0`, `f[1][3] = f[0][3] = 0`, `f[1][4] = f[0][4] = 0`

```
    j: 0  1  2  3  4
i=1:   1  1  0  0  0
```

**Processing element 2 (value = 2):**
- `f[2][0] = f[1][0] = 1`
- `f[2][1] = f[1][1] = 1`
- `f[2][2] = f[1][2] + f[1][0] = 0 + 1 = 1` (include 2 with sum 0)
- `f[2][3] = f[1][3] + f[1][1] = 0 + 1 = 1` (include 2 with sum 1)
- `f[2][4] = f[1][4] + f[1][2] = 0 + 0 = 0`

```
    j: 0  1  2  3  4
i=2:   1  1  1  1  0
```

**Processing element 3 (value = 3):**
- `f[3][0] = f[2][0] = 1`
- `f[3][1] = f[2][1] = 1`
- `f[3][2] = f[2][2] = 1`
- `f[3][3] = f[2][3] + f[2][0] = 1 + 1 = 2` (don't include 3, or include 3 with sum 0)
- `f[3][4] = f[2][4] + f[2][1] = 0 + 1 = 1`

```
    j: 0  1  2  3  4
i=3:   1  1  1  2  1
```

**Processing element 4 (value = 4):**
- `f[4][0] = f[3][0] = 1`
- `f[4][1] = f[3][1] = 1`
- `f[4][2] = f[3][2] = 1`
- `f[4][3] = f[3][3] = 2`
- `f[4][4] = f[3][4] + f[3][0] = 1 + 1 = 2`

```
    j: 0  1  2  3  4
i=4:   1  1  1  2  2
```

**Step 3: Calculate final answer**
- Total partitions = `2^4 = 16`
- Ways to form a group with sum < 5: `sum(f[4]) = 1 + 1 + 1 + 2 + 2 = 7`
- Invalid partitions (at least one group has sum < 5) = `7 * 2 = 14`
- Valid great partitions = `16 - 14 = 2`

**Verification:**
The 2 great partitions are:
1. Group 1: `[1, 4]` (sum = 5), Group 2: `[2, 3]` (sum = 5)
2. Group 1: `[2, 3]` (sum = 5), Group 2: `[1, 4]` (sum = 5)

Note: These correspond to the binary selections:
- `[1,0,0,1]` → element 1 and 4 go to group 1
- `[0,1,1,0]` → element 2 and 3 go to group 1

Both result in groups with sum ≥ 5, confirming our answer of 2 great partitions.

## Python Solution

```python
class Solution:
    def countPartitions(self, nums: List[int], k: int) -> int:
        # If total sum is less than 2k, no valid partition exists
        # (both parts need sum >= k)
        total_sum = sum(nums)
        if total_sum < k * 2:
            return 0
      
        MOD = 10**9 + 7
        n = len(nums)
      
        # dp[i][j] = number of ways to select from first i elements 
        # to get sum j (where j < k)
        dp = [[0] * k for _ in range(n + 1)]
        dp[0][0] = 1  # Base case: one way to get sum 0 with 0 elements
      
        # Total number of possible subsets (2^n)
        total_subsets = 1
      
        # Process each element
        for i in range(1, n + 1):
            # Update total possible subsets
            total_subsets = (total_subsets * 2) % MOD
          
            # Update DP table for current element
            for j in range(k):
                # Option 1: Don't include current element
                dp[i][j] = dp[i - 1][j]
              
                # Option 2: Include current element (if possible)
                if j >= nums[i - 1]:
                    dp[i][j] = (dp[i][j] + dp[i - 1][j - nums[i - 1]]) % MOD
      
        # Count invalid partitions: subsets with sum < k
        # Multiply by 2 because each subset creates two partitions
        # (subset and its complement)
        invalid_partitions = sum(dp[-1]) * 2
      
        # Valid partitions = Total partitions - Invalid partitions
        result = (total_subsets - invalid_partitions + MOD) % MOD
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n * k)`

The algorithm uses dynamic programming with nested loops:
- The outer loop iterates through all `n` elements in the array
- The inner loop iterates through values from `0` to `k-1`
- Inside the nested loops, all operations (array access, addition, modulo) are `O(1)`
- Additionally, calculating `sum(nums)` at the beginning takes `O(n)` time
- Computing `sum(f[-1])` at the end takes `O(k)` time

Therefore, the overall time complexity is `O(n) + O(n * k) + O(k) = O(n * k)`

**Space Complexity:** `O(n * k)`

The algorithm uses a 2D DP table `f` with dimensions `(n+1) × k`:
- The table `f` has `n+1` rows and `k` columns, requiring `O((n+1) * k) = O(n * k)` space
- Other variables (`mod`, `ans`, loop variables) use `O(1)` space

Therefore, the overall space complexity is `O(n * k)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Handle Negative Results in Modular Arithmetic**

**The Pitfall:**
When calculating `(total_subsets - invalid_partitions) % MOD`, if `invalid_partitions > total_subsets`, the result becomes negative. In Python, this isn't usually a problem since Python handles negative modulo correctly, but in many other languages (like Java or C++), this would give incorrect results. Additionally, even in Python, it's better practice to ensure non-negative intermediate results.

**Why It Happens:**
The counting logic might seem counterintuitive - we're subtracting `sum(dp[-1]) * 2` from `2^n`. While mathematically this should always be non-negative for valid inputs, during intermediate calculations or edge cases, the subtraction could temporarily produce negative values.

**The Solution:**
Always add `MOD` before taking the final modulo to ensure a positive result:
```python
result = (total_subsets - invalid_partitions % MOD + MOD) % MOD
```
Or even better, ensure all intermediate calculations are properly modded:
```python
invalid_partitions = (sum(dp[-1]) * 2) % MOD
result = (total_subsets - invalid_partitions + MOD) % MOD
```

### 2. **Incorrectly Counting Invalid Partitions**

**The Pitfall:**
A common mistake is forgetting to multiply `sum(dp[-1])` by 2. The reasoning is subtle: when we find a subset with sum < k, this creates an invalid partition where either the subset OR its complement could be "group 1". Both cases are invalid and need to be counted.

**Example of the Error:**
```python
# WRONG: Only counting one side
invalid_partitions = sum(dp[-1])  # Missing multiplication by 2
```

**Why It's Wrong:**
If subset A has sum < k, then partition (A, B) is invalid. But partition (B, A) where B is the complement might also be invalid if B's sum < k. We need to count both possibilities.

### 3. **Off-by-One Error in DP Array Indexing**

**The Pitfall:**
When accessing `nums[i-1]` in the DP loop, it's easy to accidentally use `nums[i]` instead, causing an index out of bounds error or incorrect calculations.

**The Problem Code:**
```python
# WRONG: Using wrong index
if j >= nums[i]:  # Should be nums[i-1]
    dp[i][j] = (dp[i][j] + dp[i-1][j - nums[i]]) % MOD
```

**The Correct Version:**
```python
if j >= nums[i - 1]:
    dp[i][j] = (dp[i][j] + dp[i - 1][j - nums[i - 1]]) % MOD
```

This happens because the DP table has dimensions `(n+1) × k` where `i` ranges from 0 to n, but the nums array has indices from 0 to n-1.

### 4. **Not Checking for Edge Case Where No Valid Partition Exists**

**The Pitfall:**
Forgetting the initial check `if total_sum < k * 2: return 0` would lead to incorrect counting. The DP would still run, but the mathematical logic breaks down when it's impossible to have both groups with sum ≥ k.

**Why It Matters:**
Without this check, the algorithm might return a positive count even when no valid partition exists, because the subtraction logic assumes at least the possibility of valid partitions existing.

