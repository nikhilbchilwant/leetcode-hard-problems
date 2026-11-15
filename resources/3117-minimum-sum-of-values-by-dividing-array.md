# 3117. Minimum Sum of Values by Dividing Array

> Source: https://algo.monster/liteproblems/3117

## Problem Statement

You are given two arrays: `nums` of length `n` and `andValues` of length `m`.

The **value** of an array is defined as its **last element**.

Your task is to divide `nums` into exactly `m` disjoint contiguous subarrays with the following constraint:
- For the `i-th` subarray (covering indices from `l_i` to `r_i`), the bitwise AND of all elements in that subarray must equal `andValues[i]`
- In other words: `nums[l_i] & nums[l_i + 1] & ... & nums[r_i] == andValues[i]` for all `1 <= i <= m`

The goal is to find the **minimum possible sum** of the values (last elements) of these `m` subarrays.

**Example breakdown:**
- You must create exactly `m` contiguous, non-overlapping subarrays from `nums`
- Each subarray's bitwise AND result must match the corresponding value in `andValues`
- The "value" of each subarray is its last element
- You want to minimize the sum of these last elements

Return the minimum possible sum if such a division exists, otherwise return `-1`.

**Key points:**
- The subarrays must be contiguous (elements next to each other)
- The subarrays must be disjoint (no overlap)
- All elements of `nums` must be used exactly once
- The bitwise AND operation `&` is applied to all elements within each subarray
- If no valid division exists that satisfies all conditions, return `-1`

## Intuition

To solve this problem, we need to think about how to explore all possible ways to divide the array while keeping track of constraints.

First, let's understand what happens with the bitwise AND operation. When we AND multiple numbers together, the result can only stay the same or decrease (never increase). This is because AND operation can only turn bits from 1 to 0, never from 0 to 1. This property is crucial - once our running AND becomes less than the target `andValues[j]`, we can never reach that target by adding more elements to the current subarray.

The problem asks us to minimize the sum of the last elements of each subarray. This suggests we need to try different cutting points and choose the ones that give us the minimum sum.

We can think of this as a decision problem: at each position in `nums`, we have two choices:
1. **Continue the current subarray** - extend it by including the current element
2. **End the current subarray** - use the current element as the last element of the current subarray and start a new one

This naturally leads to a recursive approach where we explore both options at each position.

The state we need to track includes:
- **Current position `i`** in the `nums` array
- **Number of subarrays `j`** we've already completed
- **Current AND value `a`** of the subarray we're building

At each state `(i, j, a)`:
- We calculate the new AND value by doing `a & nums[i]`
- If this new AND is less than `andValues[j]`, we can't possibly reach our target, so this path is invalid
- If the new AND equals `andValues[j]`, we have the option to end the current subarray here
- We can always try to extend the current subarray (unless it makes the AND too small)

The base cases are:
- If we've successfully divided into `m` subarrays and used all elements, return 0
- If we've run out of elements but haven't created enough subarrays, or vice versa, return infinity (invalid)

Since we'll encounter the same states multiple times through different paths, we use memoization to avoid recalculating the same subproblems. The initial AND value is set to `-1` (all bits set to 1) because `-1` in binary is all 1s, and ANDing with any number gives that number itself.

**Learn more about [Segment Tree](/problems/segment_tree_intro), [Queue](/problems/queue_intro), [Binary Search](/problems/binary_search_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses **memoization search ([dynamic programming](/problems/dynamic_programming_intro) with recursion)** to explore all valid ways to divide the array.

**Function Design:**
We define `dfs(i, j, a)` where:
- `i` = current position in `nums` array
- `j` = number of subarrays already completed
- `a` = current bitwise AND value of the subarray being built
- Returns: minimum sum of subarray values starting from position `i`

**Implementation Details:**

1. **Pruning Check:**
   ```python
   if n - i < m - j:
       return inf
   ```
   If remaining elements `(n - i)` are less than remaining subarrays needed `(m - j)`, it's impossible to complete the division.

2. **Base Case:**
   ```python
   if j == m:
       return 0 if i == n else inf
   ```
   When we've created `m` subarrays, check if we've used all elements. Return 0 if successful, infinity otherwise.

3. **Update AND Value:**
   ```python
   a &= nums[i]
   ```
   Perform bitwise AND with current element. Initially `a = -1` (all bits set), so first AND gives `nums[i]` itself.

4. **Feasibility Check:**
   ```python
   if a < andValues[j]:
       return inf
   ```
   Since AND can only decrease or stay same, if current AND is already less than target, we can never reach it.

5. **Recursive Choices:**
   - **Extend current subarray:** `dfs(i + 1, j, a)` - continue with same subarray, updated AND value
   - **End current subarray (only if AND matches target):**
     ```python
     if a == andValues[j]:
         ans = min(ans, dfs(i + 1, j + 1, -1) + nums[i])
     ```
     Add `nums[i]` (the last element) to the sum and start new subarray with `a = -1`

6. **Memoization:**
   The `@cache` decorator stores results of `dfs(i, j, a)` to avoid recomputation of identical states.

**Main Function:**
```python
n, m = len(nums), len(andValues)
ans = dfs(0, 0, -1)
return ans if ans < inf else -1
```
Start from position 0, with 0 completed subarrays, and initial AND value of -1. Return the result if valid, otherwise -1.

**Time Complexity:** `O(n * m * k)` where `k` is the number of distinct AND values possible (bounded by the range of values in `nums`)

**Space Complexity:** `O(n * m * k)` for the memoization cache

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Input:** `nums = [2, 3, 5, 7, 7, 7, 5]`, `andValues = [0, 7, 5]`

We need to divide `nums` into exactly 3 subarrays where:
- First subarray's AND must equal 0
- Second subarray's AND must equal 7
- Third subarray's AND must equal 5

Let's trace through the recursive calls starting from `dfs(0, 0, -1)`:

**Step 1:** `dfs(0, 0, -1)` - Start at index 0, 0 subarrays completed, AND = -1
- Update AND: `-1 & 2 = 2`
- Since `2 > 0` (our target), we can extend the subarray
- Call `dfs(1, 0, 2)`

**Step 2:** `dfs(1, 0, 2)` - At index 1, still building first subarray
- Update AND: `2 & 3 = 2` (binary: 10 & 11 = 10)
- Still `2 > 0`, continue extending
- Call `dfs(2, 0, 2)`

**Step 3:** `dfs(2, 0, 2)` - At index 2
- Update AND: `2 & 5 = 0` (binary: 010 & 101 = 000)
- Now AND equals target `andValues[0] = 0`!
- We have two choices:
  - **Choice A:** End subarray here: `dfs(3, 1, -1) + 5` (add value 5)
  - **Choice B:** Extend: `dfs(3, 0, 0)`

**Following Choice A:** `dfs(3, 1, -1)` - Start second subarray at index 3
- Update AND: `-1 & 7 = 7`
- AND equals target `andValues[1] = 7`!
- We can end here: `dfs(4, 2, -1) + 7` (add value 7)

**Continuing:** `dfs(4, 2, -1)` - Start third subarray at index 4
- Update AND: `-1 & 7 = 7`
- Need AND = 5, so extend: `dfs(5, 2, 7)`

**Step:** `dfs(5, 2, 7)`
- Update AND: `7 & 7 = 7`
- Still need 5, extend: `dfs(6, 2, 7)`

**Step:** `dfs(6, 2, 7)`
- Update AND: `7 & 5 = 5` (binary: 111 & 101 = 101)
- AND equals target `andValues[2] = 5`!
- End subarray: `dfs(7, 3, -1) + 5`

**Final:** `dfs(7, 3, -1)`
- `j == m` (3 subarrays completed) and `i == n` (all elements used)
- Return 0

**Backtracking the sum:**
- Third subarray contributes: 5
- Second subarray contributes: 7
- First subarray contributes: 5
- Total sum = 5 + 7 + 5 = 17

The algorithm explores other paths too (like extending the first subarray further), but this path gives us a valid division with sum 17. The memoization ensures we don't recalculate states we've already visited.

**Key insights from this example:**
1. The AND operation progressively reduces values (2 → 2 → 0)
2. We make decisions when AND equals target values
3. Multiple valid divisions may exist; we choose the one with minimum sum
4. The recursive nature allows us to explore all possibilities efficiently

## Python Solution

```python
from typing import List
from functools import cache
from math import inf


class Solution:
    def minimumValueSum(self, nums: List[int], andValues: List[int]) -> int:
        """
        Find the minimum sum of the last elements of each partition where
        the bitwise AND of each partition equals the corresponding value in andValues.

        Args:
            nums: List of integers to partition
            andValues: List of target AND values for each partition

        Returns:
            Minimum sum of last elements, or -1 if impossible
        """

        @cache
        def dfs(current_index: int, partition_index: int, current_and: int) -> int:
            """
            Dynamic programming function to find minimum sum.

            Args:
                current_index: Current position in nums array
                partition_index: Current position in andValues array (which partition we're building)
                current_and: Current AND value of the ongoing partition (-1 means new partition)

            Returns:
                Minimum sum from this state, or inf if impossible
            """

            # Check if we have enough elements left to form remaining partitions
            remaining_nums = n - current_index
            remaining_partitions = m - partition_index
            if remaining_nums < remaining_partitions:
                return inf

            # Base case: all partitions formed
            if partition_index == m:
                # Valid only if we've used all numbers
                return 0 if current_index == n else inf

            # Update current AND value with current number
            current_and &= nums[current_index]

            # If current AND is less than target, this path is invalid
            if current_and < andValues[partition_index]:
                return inf

            # Option 1: Continue current partition (add current number to it)
            result = dfs(current_index + 1, partition_index, current_and)

            # Option 2: End current partition if AND matches target
            if current_and == andValues[partition_index]:
                # End partition here, nums[current_index] is the last element
                next_partition_result = dfs(current_index + 1, partition_index + 1, -1)
                result = min(result, next_partition_result + nums[current_index])

            return result

        # Initialize variables
        n = len(nums)
        m = len(andValues)

        # Start DFS from beginning with no partition formed yet
        answer = dfs(0, 0, -1)

        # Return result or -1 if impossible
        return answer if answer < inf else -1
```

## Time and Space Complexity

The time complexity is `O(n × m × log M)`, where:
- `n` is the length of the `nums` array
- `m` is the length of the `andValues` array
- `M` is the maximum value in the `nums` array (in this problem `M ≤ 10^5`)

The `log M` factor comes from the number of distinct values that the AND operation parameter `a` can take. Since AND operations can only decrease or maintain the value (never increase), and bits can only go from 1 to 0, there are at most `O(log M)` distinct values that `a` can have as we progress through the array.

The space complexity is `O(n × m × log M)` due to the `@cache` decorator memoizing the `dfs` function. The cache stores results for each unique combination of:
- Position `i` in `nums`: `n` possible values
- Position `j` in `andValues`: `m` possible values
- Current AND accumulator `a`: `O(log M)` possible distinct values

Therefore, the maximum number of cached states is `O(n × m × log M)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Initial AND Value**
**Pitfall:** Using `0` as the initial AND value instead of `-1`.

When starting a new subarray, using `0` as the initial value would make all subsequent AND operations result in `0` (since `x & 0 = 0` for any `x`). This would incorrectly calculate the AND values.

**Why `-1` works:** In binary, `-1` is represented as all 1s in two's complement notation. When you AND any number with all 1s, you get the number itself: `x & (-1) = x`. This effectively makes the first element of each subarray the starting AND value.

**Wrong approach:**
```python
# This would always result in 0 for any subarray
dfs(i + 1, j + 1, 0)  # WRONG!
```

**Correct approach:**
```python
# Start with -1 (all bits set to 1)
dfs(i + 1, j + 1, -1)  # CORRECT
```

### 2. **Forgetting to Add the Last Element's Value**
**Pitfall:** When ending a subarray, forgetting to add `nums[i]` (the last element) to the sum.

The problem defines the "value" of a subarray as its last element. When we decide to end a subarray at index `i`, we must add `nums[i]` to our total sum.

**Wrong approach:**
```python
if current_and == andValues[partition_index]:
    result = min(result, dfs(current_index + 1, partition_index + 1, -1))  # WRONG!
```

**Correct approach:**
```python
if current_and == andValues[partition_index]:
    result = min(result, dfs(current_index + 1, partition_index + 1, -1) + nums[current_index])  # CORRECT
```

### 3. **Off-by-One Error in Index Management**
**Pitfall:** Confusion about whether to end the subarray at `current_index` or `current_index - 1`.

When we're at `current_index`, we've already included `nums[current_index]` in the AND calculation. If we decide to end the subarray here, `nums[current_index]` is the last element of this subarray. The next recursive call should start from `current_index + 1`, not `current_index`.

**Wrong approach:**
```python
# Double-counting the current element
result = min(result, dfs(current_index, partition_index + 1, -1) + nums[current_index])  # WRONG!
```

**Correct approach:**
```python
# Move to the next index after ending current subarray
result = min(result, dfs(current_index + 1, partition_index + 1, -1) + nums[current_index])  # CORRECT
```

### 4. **Missing Early Termination Optimization**
**Pitfall:** Not checking if `current_and < andValues[partition_index]` early.

Since the AND operation can only decrease or maintain the value (never increase), once the current AND becomes less than the target, it's impossible to reach the target value. This check prevents unnecessary recursive calls.

**Inefficient approach:**
```python
# Continue recursion even when it's impossible to reach target
def dfs(current_index, partition_index, current_and):
    # ... base cases ...
    current_and &= nums[current_index]
    # Missing the early termination check
    result = dfs(current_index + 1, partition_index, current_and)
    # ...
```

**Optimized approach:**
```python
def dfs(current_index, partition_index, current_and):
    # ... base cases ...
    current_and &= nums[current_index]

    # Early termination when target is unreachable
    if current_and < andValues[partition_index]:
        return inf

    result = dfs(current_index + 1, partition_index, current_and)
    # ...
```

