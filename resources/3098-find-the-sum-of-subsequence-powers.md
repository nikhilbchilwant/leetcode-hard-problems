# 3098. Find the Sum of Subsequence Powers

> Source: https://algo.monster/liteproblems/3098

## Problem Statement

You are given an integer array `nums` of length `n` and a positive integer `k`.

The **power** of a subsequence is defined as the **minimum absolute difference** between **any** two elements in that subsequence.

Your task is to find the **sum of powers** of **all** subsequences of `nums` that have exactly `k` elements.

Since the answer may be very large, return it modulo `10^9 + 7`.

**Key Points:**
- A subsequence is formed by deleting some (possibly zero) elements from the array without changing the order of the remaining elements
- You need to consider all possible subsequences of length exactly `k`
- For each such subsequence, calculate its power (the minimum absolute difference between any two elements in it)
- Sum up all these powers and return the result modulo `10^9 + 7`

For example, if you have a subsequence `[a, b, c]`, its power would be `min(|a-b|, |a-c|, |b-c|)`.

## Intuition

When dealing with minimum differences between elements, [sorting](/problems/sorting_summary) the array first is a natural approach. Once sorted, the minimum difference in any subsequence will always be between two consecutive elements (in their sorted positions within the subsequence). This is because for any two elements in a sorted subsequence, their difference increases as they get further apart.

The key insight is that we need to track not just which elements we've selected for our subsequence, but also what the current minimum difference is. As we build subsequences, this minimum difference might change based on which elements we include.

We can think of this as a [dynamic programming](/problems/dynamic_programming_intro) problem where at each position, we make a choice: either include the current element or skip it. However, when we include an element, we need to know:
1. What was the last element we included (to calculate the new difference)
2. How many more elements we still need to select
3. What is the current minimum difference in our subsequence so far

This leads us to define our state as `(i, j, k, mi)` where:
- `i` is the current position we're considering in the sorted array
- `j` is the index of the last selected element (or `n` if no element has been selected yet)
- `k` is the number of elements we still need to select
- `mi` is the current minimum difference in our subsequence

For each state, we have two choices:
1. Skip the current element: move to the next position without changing other parameters
2. Include the current element: update the last selected element to `i`, decrease `k` by 1, and potentially update the minimum difference (if `j` is not `n`, the new difference would be `nums[i] - nums[j]`)

The power of each complete subsequence (when `k` reaches 0) is its minimum difference `mi`, and we sum all these powers across all valid subsequences.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The solution uses **memoization search (top-down [dynamic programming](/problems/dynamic_programming_intro))** to efficiently compute the sum of powers for all subsequences of length `k`.

**Step 1: Sort the array**
```python
nums.sort()
```
[Sorting](/problems/sorting_summary) ensures that when we calculate differences between selected elements, consecutive selected elements in the sorted order will give us potential minimum differences.

**Step 2: Define the DFS function with memoization**
```python
@cache
def dfs(i: int, j: int, k: int, mi: int) -> int:
```
The function parameters represent:
- `i`: Current index in the sorted array we're considering
- `j`: Index of the last selected element (initialized to `n` meaning no element selected yet)
- `k`: Number of elements still needed to complete the subsequence
- `mi`: Current minimum difference in the subsequence being built

**Step 3: Base cases**
```python
if i >= n:
    return mi if k == 0 else 0
if n - i < k:
    return 0
```
- If we've processed all elements (`i >= n`), return `mi` only if we've selected exactly `k` elements (`k == 0`)
- If remaining elements are insufficient (`n - i < k`), it's impossible to form a valid subsequence, return 0

**Step 4: Recursive transitions**
```python
ans = dfs(i + 1, j, k, mi)  # Skip current element
```
First, calculate the result when we don't select the current element.

```python
if j == n:
    ans += dfs(i + 1, i, k - 1, mi)
else:
    ans += dfs(i + 1, i, k - 1, min(mi, nums[i] - nums[j]))
```
Then add the result when we select the current element:
- If `j == n` (no previous element selected), we start a new subsequence with element at index `i`
- Otherwise, we update the minimum difference to `min(mi, nums[i] - nums[j])` since `nums[i] - nums[j]` is the difference between the current and last selected element

**Step 5: Return the result with modulo**
```python
ans %= mod
return ans
```

**Step 6: Initialize and call the function**
```python
mod = 10**9 + 7
n = len(nums)
return dfs(0, n, k, inf)
```
Start from index 0, with no element selected (`j = n`), needing to select `k` elements, and initial minimum difference as infinity.

The `@cache` decorator automatically memoizes the results, preventing redundant calculations for the same state `(i, j, k, mi)`. This optimization is crucial as there can be many overlapping subproblems in the recursive calls.

## Example Walkthrough

Let's trace through a small example to understand how the solution works.

**Example:** `nums = [1, 3, 5]`, `k = 2`

**Step 1: Sort the array**
After sorting: `nums = [1, 3, 5]` (already sorted)

**Step 2: Initialize and start DFS**
We call `dfs(0, 3, 2, inf)` where:
- `i = 0`: Starting at index 0
- `j = 3`: No element selected yet (using n=3 as sentinel)
- `k = 2`: Need to select 2 elements
- `mi = inf`: Initial minimum difference is infinity

**Step 3: Trace the recursion**

From `dfs(0, 3, 2, inf)`:
- **Option 1: Skip nums[0]=1**
  - Call `dfs(1, 3, 2, inf)`

- **Option 2: Select nums[0]=1**
  - Since `j = 3` (no previous element), don't update mi
  - Call `dfs(1, 0, 1, inf)`

Let's explore Option 2 first: `dfs(1, 0, 1, inf)`
- Current: index 1, last selected was index 0 (value 1), need 1 more element
- **Option 2a: Skip nums[1]=3**
  - Call `dfs(2, 0, 1, inf)`
- **Option 2b: Select nums[1]=3**
  - New difference: `nums[1] - nums[0] = 3 - 1 = 2`
  - Update mi: `min(inf, 2) = 2`
  - Call `dfs(2, 1, 0, 2)`

From `dfs(2, 1, 0, 2)`:
- We need 0 more elements (k=0), so we've completed a subsequence
- Continue to `dfs(3, 1, 0, 2)`
- Since `i >= n`, return `mi = 2` (subsequence [1,3] has power 2)

From `dfs(2, 0, 1, inf)`:
- **Select nums[2]=5** (only valid option as we need 1 more)
  - New difference: `nums[2] - nums[0] = 5 - 1 = 4`
  - Update mi: `min(inf, 4) = 4`
  - Call `dfs(3, 2, 0, 4)`
  - Returns 4 (subsequence [1,5] has power 4)

Back to Option 1: `dfs(1, 3, 2, inf)`
- **Skip nums[1]=3**: Call `dfs(2, 3, 2, inf)` → Returns 0 (can't select 2 from 1 element)
- **Select nums[1]=3**: Call `dfs(2, 1, 1, inf)`
  - Must select nums[2]=5
  - New difference: `5 - 3 = 2`
  - Returns 2 (subsequence [3,5] has power 2)

**Step 4: Sum all powers**
- Subsequence [1,3]: power = 2
- Subsequence [1,5]: power = 4  
- Subsequence [3,5]: power = 2
- Total sum = 2 + 4 + 2 = 8

The algorithm correctly identifies all subsequences of length 2 and calculates their powers (minimum absolute differences), then returns their sum modulo 10^9 + 7.

## Python Solution

```python
from typing import List
from functools import cache
from math import inf

class Solution:
    def sumOfPowers(self, nums: List[int], k: int) -> int:
        MOD = 10**9 + 7
        n = len(nums)
      
        # Sort the array to make it easier to calculate differences
        nums.sort()
      
        @cache
        def dfs(current_index: int, last_picked_index: int, remaining_count: int, min_difference: int) -> int:
            """
            Dynamic programming function to calculate sum of powers.
          
            Args:
                current_index: Current position in nums array
                last_picked_index: Index of the last element picked in subsequence
                remaining_count: Number of elements still needed for subsequence
                min_difference: Minimum difference seen so far in current subsequence
          
            Returns:
                Sum of powers for all valid subsequences
            """
            # Base case: reached end of array
            if current_index >= n:
                # Return min_difference if we've selected exactly k elements
                return min_difference if remaining_count == 0 else 0
          
            # Pruning: not enough elements left to form subsequence of size k
            if n - current_index < remaining_count:
                return 0
          
            # Option 1: Skip current element
            result = dfs(current_index + 1, last_picked_index, remaining_count, min_difference)
          
            # Option 2: Include current element
            if last_picked_index == n:
                # First element in subsequence
                result += dfs(current_index + 1, current_index, remaining_count - 1, min_difference)
            else:
                # Calculate new minimum difference when adding current element
                new_min_difference = min(min_difference, nums[current_index] - nums[last_picked_index])
                result += dfs(current_index + 1, current_index, remaining_count - 1, new_min_difference)
          
            # Apply modulo operation
            result %= MOD
          
            return result
      
        # Start DFS with initial parameters
        # current_index=0, last_picked_index=n (sentinel for no element picked yet),
        # remaining_count=k, min_difference=inf
        return dfs(0, n, k, inf)
```

## Time and Space Complexity

The time complexity is `O(n^3 × k)` and the space complexity is `O(n^3 × k)`.

Let me analyze the recursive function `dfs(i, j, k, mi)`:
- Parameter `i` ranges from `0` to `n-1` (n possible values)
- Parameter `j` ranges from `0` to `n` (n+1 possible values, including the initial value `n`)
- Parameter `k` ranges from `0` to `k` (k+1 possible values)
- Parameter `mi` can take at most `n^2` different values (all possible differences between pairs of elements in the sorted array, plus infinity)

However, upon closer examination, `mi` represents the minimum difference encountered so far. In a sorted array of length `n`, there are at most `n-1` consecutive differences, and when considering all possible subsequences, the number of unique minimum differences is bounded by `O(n^2)` (all pairwise differences).

Therefore, the total number of unique states in the memoization is `O(n × n × k × n^2) = O(n^4 × k)`.

Each state computation involves constant time operations (excluding recursive calls), so the time complexity is `O(n^4 × k)`.

The space complexity is also `O(n^4 × k)` due to the memoization cache storing all possible states.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Using Infinity as Initial Minimum Difference

**Problem:** 
Using `float('inf')` as the initial minimum difference can cause issues with hashing in the `@cache` decorator. Python's `functools.cache` uses the arguments as dictionary keys, and `float('inf')` may not hash consistently across different Python implementations or could lead to unexpected behavior in the memoization.

**Solution:**
Use a large integer value instead of infinity as the initial minimum difference:

```python
# Instead of:
return dfs(0, n, k, inf)

# Use:
MAX_DIFF = 10**9  # or max(nums) - min(nums) + 1
return dfs(0, n, k, MAX_DIFF)
```

### Pitfall 2: Incorrect Handling of Minimum Difference Updates

**Problem:**
A common mistake is updating the minimum difference incorrectly when selecting elements. Since the array is sorted, we only need to consider the difference between consecutive selected elements (not all pairs), but developers might try to compare the current element with all previously selected elements.

**Incorrect approach:**
```python
# Wrong: Trying to maintain all selected elements and compute all pairwise differences
selected_elements.append(nums[current_index])
new_min = min(abs(nums[current_index] - elem) for elem in selected_elements)
```

**Solution:**
The correct approach only tracks the last selected element and computes the difference with it:
```python
# Correct: Only compute difference with last selected element
new_min_difference = min(min_difference, nums[current_index] - nums[last_picked_index])
```

### Pitfall 3: Memory Limit Exceeded Due to Excessive State Space

**Problem:**
If the minimum difference can take too many unique values, the memoization cache can grow exponentially, leading to memory issues. This happens when `min_difference` has a large range of possible values.

**Solution:**
Optimize the state representation by discretizing the minimum differences:

```python
def sumOfPowers(self, nums: List[int], k: int) -> int:
    MOD = 10**9 + 7
    n = len(nums)
    nums.sort()
  
    # Collect all possible differences
    all_diffs = set()
    for i in range(n):
        for j in range(i + 1, n):
            all_diffs.add(nums[j] - nums[i])
    all_diffs.add(float('inf'))  # sentinel value
  
    # Create mapping for discretization
    diff_to_idx = {diff: idx for idx, diff in enumerate(sorted(all_diffs))}
  
    @cache
    def dfs(current_index: int, last_picked_index: int, remaining_count: int, min_diff_idx: int) -> int:
        # Use min_diff_idx instead of actual min_difference value
        min_difference = sorted(all_diffs)[min_diff_idx]
        # ... rest of the logic remains similar
```

### Pitfall 4: Forgetting to Apply Modulo at Each Step

**Problem:**
Only applying modulo at the final return can cause integer overflow in languages with fixed-size integers, or unnecessary large number computations in Python.

**Solution:**
Apply modulo operation after each addition to keep numbers manageable:
```python
result = dfs(current_index + 1, last_picked_index, remaining_count, min_difference)
result = (result + dfs(current_index + 1, current_index, remaining_count - 1, new_min_difference)) % MOD
return result
```

