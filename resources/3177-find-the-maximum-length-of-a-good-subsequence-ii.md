# 3177. Find the Maximum Length of a Good Subsequence II

> Source: https://algo.monster/liteproblems/3177

## Problem Statement

You are given an integer array `nums` and a non-negative integer `k`. 

A sequence of integers `seq` is called **good** if there are **at most** `k` indices `i` in the range `[0, seq.length - 2]` where `seq[i] != seq[i + 1]`. In other words, a good sequence can have at most `k` positions where adjacent elements are different.

Your task is to find the **maximum** possible length of a **good subsequence** of `nums`.

A subsequence is a sequence that can be derived from the array by deleting some or no elements without changing the order of the remaining elements.

For example:
- If we have a sequence `[1, 1, 2, 2, 3]` and `k = 2`, there are exactly 2 positions where adjacent elements differ: between the second `1` and first `2`, and between the second `2` and `3`. So this would be a valid good sequence.
- If `k = 0`, then the good sequence must have all elements the same (no position where adjacent elements differ).
- If `k = 1`, then the good sequence can have at most one position where adjacent elements change.

The goal is to select a subsequence from `nums` that satisfies the "good" property while having the maximum possible length.

## Intuition

The key insight is to think about this problem in terms of [dynamic programming](/problems/dynamic_programming_intro) where we track the longest good subsequence ending at each position with a specific number of "changes" used.

Let's think about building a subsequence step by step. When we're at position `i` and want to include `nums[i]` in our subsequence, we need to decide what element came before it. We have two choices:

1. **Same element**: If the previous element in our subsequence is the same as `nums[i]`, we don't use any additional "change" positions. We want to find the longest subsequence ending with the same value that uses at most `h` changes.

2. **Different element**: If the previous element is different from `nums[i]`, we consume one "change" position. So we need to look at subsequences with at most `h-1` changes and add `nums[i]` to them.

This leads us to define `f[i][h]` as the length of the longest good subsequence ending at index `i` with at most `h` changes used.

For each position `i` and change count `h`, we examine all previous positions `j < i`:
- If `nums[i] == nums[j]`, we can extend the subsequence ending at `j` without using an additional change: `f[i][h] = max(f[i][h], f[j][h] + 1)`
- If `nums[i] != nums[j]` and `h > 0`, we can extend the subsequence ending at `j` by using one change: `f[i][h] = max(f[i][h], f[j][h-1] + 1)`

The challenge with this approach is efficiency. Checking all previous positions for each `i` gives us `O(n²k)` complexity, which is too slow for large inputs.

The optimization comes from recognizing patterns in what we're looking for:
- For same elements, we just need the maximum `f[j][h]` where `nums[j] == nums[i]`. We can maintain this using a map for each change count.
- For different elements, we need the maximum `f[j][h-1]` across all different values. Since we only care about the maximum (and sometimes second maximum when the maximum corresponds to `nums[i]`), we can maintain just the top two values for each change count.

This optimization reduces our lookups from `O(n)` to `O(1)` for each state calculation.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements the optimized [dynamic programming](/problems/dynamic_programming_intro) approach using several key data structures:

### Data Structures

1. **`f[i][h]`**: A 2D array where `f[i][h]` represents the length of the longest good subsequence ending at index `i` with at most `h` changes.

2. **`mp[h]`**: An array of dictionaries where `mp[h][x]` stores the maximum length of a good subsequence ending with value `x` using at most `h` changes. This helps us quickly find the best subsequence to extend when we encounter the same value.

3. **`g[h]`**: A 2D array of size `(k+1) × 3` that tracks:
   - `g[h][0]`: The value corresponding to the maximum length
   - `g[h][1]`: The maximum length of subsequences with at most `h` changes
   - `g[h][2]`: The second maximum length (needed when the maximum corresponds to the current value)

### Algorithm Steps

For each element `nums[i]` and each possible change count `h` from 0 to `k`:

1. **Initialize with same value extension**: 
   ```python
   f[i][h] = mp[h][x]
   ```
   This handles extending subsequences that end with the same value `x = nums[i]`.

2. **Consider different value extension** (if `h > 0`):
   ```python
   if g[h-1][0] != nums[i]:
       f[i][h] = max(f[i][h], g[h-1][1])
   else:
       f[i][h] = max(f[i][h], g[h-1][2])
   ```
   - If the best subsequence with `h-1` changes doesn't end with `nums[i]`, we can use it directly
   - Otherwise, we use the second-best to avoid counting the same value twice

3. **Add current element**: 
   ```python
   f[i][h] += 1
   ```

4. **Update the map for future same-value lookups**:
   ```python
   mp[h][nums[i]] = max(mp[h][nums[i]], f[i][h])
   ```

5. **Update the global maximum tracker `g[h]`**:
   - If `x != g[h][0]` (different from current maximum value):
     - If `f[i][h] >= g[h][1]`, update both maximum and second maximum
     - Otherwise, just update second maximum if needed
   - If `x == g[h][0]` (same as current maximum value):
     - Just update the maximum length

6. **Track the overall answer**:
   ```python
   ans = max(ans, f[i][h])
   ```

### State Transition

The core state transition follows:

$$
f[i][h]=
\begin{cases}
\max(f[i][h], f[j][h] + 1), & \text{if } nums[i] = nums[j] \\
\max(f[i][h], f[j][h - 1] + 1), & \text{if } h > 0 \text{ and } nums[i] \neq nums[j]
\end{cases}
$$

But instead of checking all `j < i`, we use the optimized lookups through `mp` and `g`.

### Time and Space Complexity

- **Time Complexity**: `O(n × k)` where `n` is the length of the array and `k` is the maximum allowed changes
- **Space Complexity**: `O(n × k)` for the DP table, plus `O(k × m)` for the maps where `m` is the number of distinct values

This optimization reduces the time complexity from `O(n² × k)` to `O(n × k)`, making it efficient enough to handle large inputs.

## Example Walkthrough

Let's walk through the solution with `nums = [1, 2, 1, 1, 3]` and `k = 2`.

We want to find the longest subsequence where adjacent elements can differ at most 2 times.

**Initialization:**
- `f[i][h]`: DP table (5×3 since we have 5 elements and k+1=3 possible change counts)
- `mp[h]`: Array of 3 dictionaries for quick same-value lookups
- `g[h]`: Array tracking [best_value, max_length, second_max_length] for each h

**Processing each element:**

**i=0, nums[0]=1:**
- For h=0,1,2: 
  - No previous elements, so `f[0][h] = 0 + 1 = 1`
  - Update `mp[h][1] = 1`
  - Update `g[h] = [1, 1, 0]` (value 1 has length 1)

**i=1, nums[1]=2:**
- For h=0: 
  - Same value: `mp[0][2] = 0` (no previous 2's)
  - Different value: Can't use (need h>0 for changes)
  - `f[1][0] = 0 + 1 = 1`
- For h=1:
  - Same value: `mp[1][2] = 0`
  - Different value: Best with h=0 is `g[0][1] = 1` (from value 1)
  - `f[1][1] = max(0, 1) + 1 = 2` (subsequence: [1,2])
- For h=2:
  - Similar logic: `f[1][2] = 2`
- Update maps and g arrays accordingly

**i=2, nums[2]=1:**
- For h=0:
  - Same value: `mp[0][1] = 1` (from i=0)
  - `f[2][0] = 1 + 1 = 2` (subsequence: [1,1])
- For h=1:
  - Same value: `mp[1][1] = 1`
  - Different value: Best with h=0 is from value 2, `g[0][1] = 1`
  - `f[2][1] = max(1, 1) + 1 = 2`
- For h=2:
  - Same value: `mp[2][1] = 1`
  - Different value: Best with h=1 is `g[1][1] = 2` (from value 2)
  - `f[2][2] = max(1, 2) + 1 = 3` (subsequence: [1,2,1])

**i=3, nums[3]=1:**
- For h=0:
  - Same value: `mp[0][1] = 2` (updated from i=2)
  - `f[3][0] = 2 + 1 = 3` (subsequence: [1,1,1])
- For h=1:
  - Same value: `mp[1][1] = 2`
  - Different value: Best with h=0 from non-1 value is 1
  - `f[3][1] = max(2, 1) + 1 = 3`
- For h=2:
  - Same value: `mp[2][1] = 3` (updated from i=2)
  - Different value: Best with h=1 from non-1 value is 2
  - `f[3][2] = max(3, 2) + 1 = 4` (subsequence: [1,2,1,1])

**i=4, nums[4]=3:**
- For h=0:
  - Same value: `mp[0][3] = 0`
  - `f[4][0] = 0 + 1 = 1`
- For h=1:
  - Same value: `mp[1][3] = 0`
  - Different value: Best with h=0 is `g[0][1] = 3` (from value 1)
  - `f[4][1] = max(0, 3) + 1 = 4` (subsequence: [1,1,1,3])
- For h=2:
  - Same value: `mp[2][3] = 0`
  - Different value: Best with h=1 is `g[1][1] = 3` (from value 1)
  - `f[4][2] = max(0, 3) + 1 = 4`

**Final Answer:** Maximum across all f[i][h] values = 4

The optimal subsequence is [1,2,1,1] or [1,1,1,3], both having length 4 with at most 2 changes between adjacent elements.

## Python Solution

```python
class Solution:
    def maximumLength(self, nums: List[int], k: int) -> int:
        n = len(nums)
      
        # dp[i][j] = maximum length of subsequence ending at index i with at most j adjacent differences
        dp = [[0] * (k + 1) for _ in range(n)]
      
        # max_len_by_value[j][value] = maximum length of subsequence ending with 'value' using at most j differences
        max_len_by_value = [defaultdict(int) for _ in range(k + 1)]
      
        # top_two[j] stores the top two maximum lengths for j differences:
        # top_two[j][0] = value that achieves the maximum length
        # top_two[j][1] = maximum length
        # top_two[j][2] = second maximum length (for different value)
        top_two = [[0] * 3 for _ in range(k + 1)]
      
        max_length = 0
      
        for i, current_value in enumerate(nums):
            for num_differences in range(k + 1):
                # Start with the best length ending with the same value
                dp[i][num_differences] = max_len_by_value[num_differences][current_value]
              
                # If we can use one more difference, consider transitioning from a different value
                if num_differences > 0:
                    # If the best value from (num_differences - 1) is different from current value
                    if top_two[num_differences - 1][0] != current_value:
                        # Use the best length from previous difference count
                        dp[i][num_differences] = max(dp[i][num_differences], 
                                                     top_two[num_differences - 1][1])
                    else:
                        # Current value matches the best, so use second best
                        dp[i][num_differences] = max(dp[i][num_differences], 
                                                     top_two[num_differences - 1][2])
              
                # Add current element to the subsequence
                dp[i][num_differences] += 1
              
                # Update the maximum length for current value with num_differences
                max_len_by_value[num_differences][current_value] = max(
                    max_len_by_value[num_differences][current_value], 
                    dp[i][num_differences]
                )
              
                # Update top two maximum lengths for num_differences
                if top_two[num_differences][0] != current_value:
                    # Current value is different from the best value
                    if dp[i][num_differences] >= top_two[num_differences][1]:
                        # New best found, shift previous best to second
                        top_two[num_differences][2] = top_two[num_differences][1]
                        top_two[num_differences][1] = dp[i][num_differences]
                        top_two[num_differences][0] = current_value
                    else:
                        # Update second best if necessary
                        top_two[num_differences][2] = max(top_two[num_differences][2], 
                                                         dp[i][num_differences])
                else:
                    # Current value matches the best value, update the best length
                    top_two[num_differences][1] = max(top_two[num_differences][1], 
                                                      dp[i][num_differences])
              
                # Update global maximum
                max_length = max(max_length, dp[i][num_differences])
      
        return max_length
```

## Time and Space Complexity

**Time Complexity:** `O(n × k)`

The algorithm iterates through each element in the array `nums` (n iterations). For each element, it performs operations across all possible values from 0 to k (k+1 iterations). Within each iteration, all operations are constant time:
- Dictionary lookups and updates in `mp[h][x]` are `O(1)` on average
- Array accesses and comparisons in `f[i][h]` and `g[h]` are `O(1)`
- The max operations and conditional updates are `O(1)`

Therefore, the total time complexity is `O(n × (k + 1))` = `O(n × k)`.

**Space Complexity:** `O(n × k)`

The space usage consists of:
- `f`: A 2D array of size `n × (k + 1)`, contributing `O(n × k)` space
- `mp`: A list of `k + 1` dictionaries. In the worst case, each dictionary could store up to `n` unique elements, giving `O(n × k)` space
- `g`: A 2D array of size `(k + 1) × 3`, contributing `O(k)` space
- Other variables like `ans`, `i`, `x`, `h` use `O(1)` space

The dominant space complexity is `O(n × k)` from the `f` array and `mp` dictionaries.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Incorrect Initialization of the Top-Two Tracker

**Pitfall**: The most critical issue is initializing `top_two[j][0]` to 0, which can conflict with actual values in the array that are also 0. This causes incorrect comparisons when checking if `top_two[num_differences - 1][0] != current_value`.

**Example of the Bug**:
```python
nums = [0, 1, 0, 2], k = 2
# When processing the first 0, top_two[0][0] is already 0 (initial value)
# This incorrectly makes the algorithm think there's already a subsequence ending with 0
```

**Solution**: Initialize the value field with a sentinel value that cannot appear in the input:
```python
# Instead of:
top_two = [[0] * 3 for _ in range(k + 1)]

# Use:
top_two = [[None, 0, 0] for _ in range(k + 1)]
# Or use a value outside the input range like -float('inf') or -1 if nums contains only non-negative integers
```

Then update the comparison logic:
```python
if top_two[num_differences - 1][0] is None or top_two[num_differences - 1][0] != current_value:
    dp[i][num_differences] = max(dp[i][num_differences], 
                                 top_two[num_differences - 1][1])
```

### 2. Misunderstanding the Second Maximum Update Logic

**Pitfall**: When updating `top_two`, the code only updates the second maximum when the current value differs from the best value. However, there could be multiple distinct values achieving different lengths, and we need to track the true second maximum across all values.

**Example Issue**:
```python
# Consider sequence: [1, 2, 3, 1] with k = 2
# After processing indices 0, 1, 2:
# top_two[2] might be [3, 3, 2] (value 3 with length 3, second best is 2)
# When processing index 3 (value 1), we might miss updating the second maximum correctly
```

**Solution**: Maintain proper second maximum tracking:
```python
if top_two[num_differences][0] != current_value:
    if dp[i][num_differences] >= top_two[num_differences][1]:
        # New maximum found
        top_two[num_differences][2] = top_two[num_differences][1]
        top_two[num_differences][1] = dp[i][num_differences]
        top_two[num_differences][0] = current_value
    elif dp[i][num_differences] > top_two[num_differences][2]:
        # Not the best, but better than second best
        top_two[num_differences][2] = dp[i][num_differences]
```

### 3. Edge Case with k = 0

**Pitfall**: When `k = 0`, the subsequence must have all identical elements. The transition logic for different values should never be triggered, but the code structure might still attempt it if not careful.

**Solution**: Ensure the condition `if num_differences > 0:` properly guards the different-value transition, which the current code already does correctly.

### 4. Memory Optimization Opportunity Missed

**Pitfall**: The code maintains a full `n × (k+1)` DP table, but only the current row is actually needed for the final result.

**Solution for Space Optimization**:
```python
# Instead of dp[i][num_differences], use a 1D array that gets updated:
current_dp = [0] * (k + 1)
# Update in place for each element
```

However, this optimization trades off code clarity for space efficiency and should be considered based on constraints.

