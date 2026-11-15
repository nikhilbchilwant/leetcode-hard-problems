# 2945. Find Maximum Non-decreasing Array Length

> Source: https://algo.monster/liteproblems/2945

## Problem Statement

You are given a 0-indexed integer array `nums`.

You can perform any number of operations on this array. In each operation, you:
1. Select a **subarray** (a contiguous non-empty sequence of elements)
2. Replace that subarray with a single element equal to the **sum** of all elements in that subarray

For example, if `nums = [1,3,5,6]` and you select the subarray `[3,5]`, the array becomes `[1,8,6]` (the subarray `[3,5]` is replaced by their sum `8`).

Your goal is to find the **maximum length** of a **non-decreasing** array that can be obtained after performing any number of these operations.

A non-decreasing array means each element is greater than or equal to the previous element (i.e., `nums[i] <= nums[i+1]` for all valid indices).

**Examples:**

- `nums = [5,2,2]`: The original array is not non-decreasing. You can:
  - Replace `[2,2]` with `[4]` to get `[5,4]` - still not non-decreasing
  - Replace `[5,2]` with `[7]` to get `[7,2]` - still not non-decreasing
  - Replace `[5,2,2]` with `[9]` to get `[9]` - this is non-decreasing with length 1
  - Maximum achievable length is 1

- `nums = [1,2,3,4]`: Already non-decreasing, so maximum length is 4

- `nums = [4,3,2,6]`: Replace `[3,2]` with `[5]` to get `[4,5,6]` which is non-decreasing with length 3

The challenge is to strategically choose which subarrays to merge to create the longest possible non-decreasing sequence.

## Intuition

The key insight is that when we merge subarrays into their sum, we're essentially creating segments where each segment has a value (the sum of its elements). To maximize the length of the final non-decreasing array, we want to create as many segments as possible while maintaining the non-decreasing property.

Think about it this way: if we have a segment ending at position `i` with sum `S`, the next segment starting from position `i+1` must have a sum at least `S` to maintain the non-decreasing property. This means we want to find the shortest possible next segment whose sum is at least `S`.

This leads us to a [dynamic programming](/problems/dynamic_programming_intro) approach where:
- `f[i]` represents the maximum number of segments we can create using the first `i` elements
- For each position `i`, we need to track what was the last position where we ended a segment that allows us to create valid segments up to position `i`

The crucial observation is that if we ended a segment at position `pre[i]`, and the sum from `pre[i]` to `i` is `sum1`, then the next segment must have sum at least `sum1`. To find where the next segment can end, we need to find the smallest index `j` where the sum from `i` to `j` is at least `sum1`.

Using prefix sums `s[i]` to quickly calculate range sums:
- Sum from position `a` to `b` is `s[b] - s[a]`
- If the last segment ended at `pre[i]` with sum `s[i] - s[pre[i]]`, we need the next segment to have sum at least this value
- We're looking for the smallest `j` where `s[j] - s[i] >= s[i] - s[pre[i]]`
- Rearranging: `s[j] >= 2*s[i] - s[pre[i]]`

This is why the solution uses `bisect_left` to find the earliest position where we can end the next segment, and maintains `pre[j]` to track the best previous segment ending position for each index.

**Learn more about [Stack](/problems/stack_intro), [Queue](/problems/queue_intro), [Binary Search](/problems/binary_search_intro), [Dynamic Programming](/problems/dynamic_programming_intro), [Monotonic Queue](/problems/mono_stack_intro) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with prefix sums and [binary search](/problems/binary_search_intro) to efficiently find the maximum number of segments (which equals the maximum length of the final non-decreasing array).

**Data Structures Used:**
- `s`: Prefix sum array where `s[i]` = sum of `nums[0]` to `nums[i-1]`
- `f[i]`: Maximum number of segments achievable using the first `i` elements
- `pre[i]`: The best ending position of the last segment before position `i`

**Algorithm Steps:**

1. **Initialize prefix sums**: Create `s` using `accumulate(nums, initial=0)` so that `s[i]` stores the sum of elements from index 0 to i-1.

2. **[Dynamic Programming](/problems/dynamic_programming_intro) Loop**: For each position `i` from 1 to n:

   a. **Inherit the best previous position**: `pre[i] = max(pre[i], pre[i-1])`
      - This ensures we're using the optimal ending position from previous computations

   b. **Calculate maximum segments**: `f[i] = f[pre[i]] + 1`
      - We can form one more segment than what we had at position `pre[i]`
      - The new segment spans from `pre[i]` to `i`

   c. **Find next valid position**: Use [binary search](/problems/binary_search_intro) to find the smallest index `j` where we can end the next segment
      - The sum of the current segment is `s[i] - s[pre[i]]`
      - The next segment starting at `i` must have sum ≥ `s[i] - s[pre[i]]`
      - We need `s[j] - s[i] >= s[i] - s[pre[i]]`, which gives us `s[j] >= 2*s[i] - s[pre[i]]`
      - `j = bisect_left(s, 2*s[i] - s[pre[i]])`

   d. **Update future position**: `pre[j] = i`
      - Record that position `i` is a good ending position for creating segments up to position `j`

3. **Return Result**: `f[n]` contains the maximum number of segments using all n elements.

**Why this works:**
- By maintaining `pre[i]`, we always know the optimal way to partition elements up to position `i`
- The [binary search](/problems/binary_search_intro) ensures we find the earliest possible position to end the next segment while maintaining the non-decreasing property
- The DP recurrence `f[i] = f[pre[i]] + 1` builds up the solution optimally from smaller subproblems

**Time Complexity**: O(n log n) due to the [binary search](/problems/binary_search_intro) operation for each of the n positions.
**Space Complexity**: O(n) for the auxiliary arrays.

## Example Walkthrough

Let's walk through the solution with `nums = [4,3,2,6]`.

**Step 1: Initialize prefix sums**
- `nums = [4,3,2,6]`
- `s = [0,4,7,9,15]` (cumulative sums)
- `n = 4`
- `f = [0,0,0,0,0]` (max segments for each position)
- `pre = [0,0,0,0,0]` (best previous segment ending)

**Step 2: Process each position**

**i = 1:**
- `pre[1] = max(0, 0) = 0`
- `f[1] = f[0] + 1 = 0 + 1 = 1` (one segment from 0 to 1)
- Current segment sum: `s[1] - s[0] = 4 - 0 = 4`
- Find next position j where sum ≥ 4:
  - Need `s[j] >= 2*4 - 0 = 8`
  - `bisect_left(s, 8)` returns 3 (since s[3] = 9 ≥ 8)
- `pre[3] = 1` (position 1 is good for segments up to position 3)

**i = 2:**
- `pre[2] = max(0, 0) = 0`
- `f[2] = f[0] + 1 = 1` (one segment from 0 to 2)
- Current segment sum: `s[2] - s[0] = 7 - 0 = 7`
- Find next position j where sum ≥ 7:
  - Need `s[j] >= 2*7 - 0 = 14`
  - `bisect_left(s, 14)` returns 4 (since s[4] = 15 ≥ 14)
- `pre[4] = 2` (position 2 is good for segments up to position 4)

**i = 3:**
- `pre[3] = max(1, 0) = 1` (from earlier update)
- `f[3] = f[1] + 1 = 1 + 1 = 2` (two segments: [4] and [3,2])
- Current segment sum: `s[3] - s[1] = 9 - 4 = 5`
- Find next position j where sum ≥ 5:
  - Need `s[j] >= 2*9 - 4 = 14`
  - `bisect_left(s, 14)` returns 4
- `pre[4] = max(2, 3) = 3` (update to better position)

**i = 4:**
- `pre[4] = max(3, 0) = 3` (from earlier updates)
- `f[4] = f[3] + 1 = 2 + 1 = 3` (three segments)
- Current segment sum: `s[4] - s[3] = 15 - 9 = 6`
- Find next position j where sum ≥ 6:
  - Need `s[j] >= 2*15 - 9 = 21`
  - `bisect_left(s, 21)` returns 5 (beyond array)
- No update needed (j > n)

**Result:** `f[4] = 3`

This means we can create 3 segments:
- Segment 1: `[4]` with sum 4
- Segment 2: `[3,2]` with sum 5
- Segment 3: `[6]` with sum 6

The final array `[4,5,6]` is non-decreasing with length 3.

## Python Solution

```python
from typing import List
from itertools import accumulate
from bisect import bisect_left


class Solution:
    def findMaximumLength(self, nums: List[int]) -> int:
        """
        Find the maximum length of a non-decreasing sequence that can be formed
        by merging consecutive elements of the input array.

        Args:
            nums: List of integers representing the input array

        Returns:
            Maximum length of the resulting sequence
        """
        n = len(nums)

        # Calculate prefix sums for quick range sum queries
        # prefix_sums[i] = sum of nums[0:i]
        prefix_sums = list(accumulate(nums, initial=0))

        # dp[i] represents the maximum length of sequence using nums[0:i]
        dp = [0] * (n + 1)

        # last_segment_start[i] stores the optimal starting position of the last segment
        # when considering elements up to position i
        last_segment_start = [0] * (n + 2)

        # Process each position from 1 to n
        for i in range(1, n + 1):
            # Update the best starting position seen so far
            last_segment_start[i] = max(last_segment_start[i], last_segment_start[i - 1])

            # Calculate maximum length at position i
            # We extend from the optimal previous segment
            dp[i] = dp[last_segment_start[i]] + 1

            # Find the next position where we can form a valid segment
            # The segment from last_segment_start[i] to i has sum:
            # prefix_sums[i] - prefix_sums[last_segment_start[i]]
            # We need the next segment to have at least this sum
            segment_sum = prefix_sums[i] - prefix_sums[last_segment_start[i]]
            target_prefix_sum = prefix_sums[i] + segment_sum
            next_position = bisect_left(prefix_sums, target_prefix_sum)

            # Update the optimal starting position for the next valid position
            last_segment_start[next_position] = i

        return dp[n]
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)`

The algorithm iterates through the array once with a loop from 1 to n, which takes `O(n)` time. Within each iteration:
- Accessing and updating arrays `pre` and `f` takes `O(1)` time
- The `bisect_left` operation performs binary search on the prefix sum array `s`, which takes `O(log n)` time

Since we perform the binary search operation for each of the n iterations, the overall time complexity is `O(n) * O(log n) = O(n log n)`.

**Space Complexity:** `O(n)`

The algorithm uses several auxiliary arrays:
- `s`: prefix sum array of size `n + 1`
- `f`: DP array of size `n + 1`
- `pre`: array of size `n + 2`

Each array uses linear space proportional to the input size n. Therefore, the total space complexity is `O(n) + O(n) + O(n) = O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Off-by-One Errors with Index Management**

The most common pitfall in this solution is confusion around the indexing conventions, particularly with the `last_segment_start` array and how it relates to the prefix sums.

**The Problem:**
- The prefix sum array has length `n+1` where `prefix_sums[i]` represents the sum of elements from index 0 to i-1
- The `last_segment_start` array needs to handle positions beyond `n` (hence size `n+2`)
- When using `bisect_left`, the returned index might be `n+1`, which would cause an index out of bounds error if not handled properly

**Example of the pitfall:**
```python
# Incorrect: Array too small
last_segment_start = [0] * (n + 1)  # This will cause IndexError
# When bisect_left returns n+1, last_segment_start[n+1] will fail
```

**Solution:**
Always ensure the `last_segment_start` array has size `n+2` to accommodate all possible positions returned by `bisect_left`.

### 2. **Incorrect Prefix Sum Calculation**

**The Problem:**
Using the wrong initial value or incorrect accumulation can lead to wrong segment sums.

**Example of the pitfall:**
```python
# Incorrect: No initial value
prefix_sums = list(accumulate(nums))  # Length is n, not n+1
# This makes prefix_sums[0] = nums[0] instead of 0
```

**Solution:**
Always use `accumulate(nums, initial=0)` to ensure `prefix_sums[0] = 0` and the array has length `n+1`.

### 3. **Misunderstanding the Binary Search Target**

**The Problem:**
The binary search looks for the minimum position where we can end the next segment while maintaining the non-decreasing property. The calculation `2 * prefix_sums[i] - prefix_sums[last_segment_start[i]]` might seem arbitrary.

**Example of the pitfall:**
```python
# Incorrect understanding might lead to:
next_position = bisect_left(prefix_sums, prefix_sums[i] + segment_sum, i+1)
# Adding a start parameter to bisect_left, which changes the behavior
```

**Solution:**
Remember that we're searching for the first position `j` where:
- The sum from position `i` to `j-1` is at least `segment_sum`
- This translates to: `prefix_sums[j] >= prefix_sums[i] + segment_sum`
- Which simplifies to: `prefix_sums[j] >= 2 * prefix_sums[i] - prefix_sums[last_segment_start[i]]`

### 4. **Not Propagating the Best Previous Position**

**The Problem:**
Forgetting to update `last_segment_start[i]` with the maximum of itself and the previous position.

**Example of the pitfall:**
```python
# Incorrect: Missing the max operation
for i in range(1, n + 1):
    # last_segment_start[i] = max(last_segment_start[i], last_segment_start[i - 1])  # Missing!
    dp[i] = dp[last_segment_start[i]] + 1  # Will use 0 instead of optimal position
```

**Solution:**
Always include the line `last_segment_start[i] = max(last_segment_start[i], last_segment_start[i - 1])` to ensure we're using the best ending position discovered so far.

