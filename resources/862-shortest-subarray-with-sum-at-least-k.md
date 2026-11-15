# 862. Shortest Subarray with Sum at Least K

> Source: https://algo.monster/liteproblems/862

## Problem Statement

You are given an integer array `nums` and an integer `k`. Your task is to find the length of the shortest contiguous subarray whose sum is at least `k`.

A subarray is a contiguous sequence of elements from the array. For example, if `nums = [1, 2, 3, 4]`, then `[2, 3]` is a subarray, but `[1, 3]` is not (since elements are not contiguous).

The subarray must:
- Be non-empty (contain at least one element)
- Have a sum that is greater than or equal to `k`
- Be as short as possible among all valid subarrays

If no such subarray exists that meets these conditions, return `-1`.

For example:
- If `nums = [2, -1, 2]` and `k = 3`, the shortest subarray with sum ≥ 3 is `[2, -1, 2]` with length 3
- If `nums = [1, 2]` and `k = 4`, no subarray has sum ≥ 4, so return `-1`

The challenge involves efficiently finding the minimum length among all possible contiguous subarrays that satisfy the sum requirement.

## Intuition

The key insight is to use prefix sums to convert the problem from "finding a subarray with sum ≥ k" to "finding two indices where their [prefix sum](/problems/subarray_sum) difference is ≥ k".

If we compute the [prefix sum](/problems/subarray_sum) array `s` where `s[i]` represents the sum of elements from index 0 to i-1, then the sum of any subarray from index `i` to `j` equals `s[j+1] - s[i]`. We need `s[j+1] - s[i] >= k`.

The challenge is that we need the shortest such subarray. A brute force approach checking all pairs would take O(n²) time. We need something more efficient.

The crucial observation is that we can use a monotonic deque to maintain potentially useful starting points. As we iterate through the [prefix sum](/problems/subarray_sum) array:

1. **Why remove elements from the front?** If we're at position `i` with [prefix sum](/problems/subarray_sum) `s[i]`, and there's an earlier position `j` where `s[i] - s[j] >= k`, then we found a valid subarray. We can remove `j` from consideration because any future position would create a longer subarray with `j` as the starting point.

2. **Why maintain monotonicity?** If we have two positions `j1 < j2` with `s[j1] >= s[j2]`, then `j1` is never useful. Why? Because for any future position `i`, if `s[i] - s[j1] >= k`, then `s[i] - s[j2] >= k` as well (since `s[j2] <= s[j1]`), and the subarray starting at `j2` would be shorter.

This leads to maintaining a deque of indices with strictly increasing [prefix sum](/problems/subarray_sum) values. The deque helps us quickly find the best starting point for each ending position while keeping the time complexity at O(n).

**Learn more about [Queue](/problems/queue_intro), [Binary Search](/problems/binary_search_intro), [Prefix Sum](/problems/subarray_sum), [Sliding Window](/problems/sliding_window_maximum), [Monotonic Queue](/problems/mono_stack_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation uses a monotonic deque with prefix sums to efficiently find the shortest subarray:

**Step 1: Build [Prefix Sum](/problems/subarray_sum) Array**
```python
prefix_sums = list(accumulate(nums, initial=0))
```
This creates a prefix sum array where `prefix_sums[i]` represents the sum of elements from index 0 to i-1. The `initial=0` parameter ensures `prefix_sums[0] = 0`, making it easier to handle subarrays starting from index 0.

**Step 2: Initialize Data Structures**
```python
indices_deque = deque()
min_length = float('inf')
```
- `indices_deque`: A deque that stores indices of the [prefix sum](/problems/subarray_sum) array
- `min_length`: Tracks the minimum length found so far, initialized to infinity

**Step 3: Process Each [Prefix Sum](/problems/subarray_sum)**
For each position `current_index` with prefix sum value `current_sum`:

```python
while indices_deque and current_sum - prefix_sums[indices_deque[0]] >= k:
    min_length = min(min_length, current_index - indices_deque.popleft())
```
This checks if the current position forms a valid subarray with the leftmost index in the deque. If `current_sum - prefix_sums[indices_deque[0]] >= k`, we found a valid subarray of length `current_index - indices_deque[0]`. We remove `indices_deque[0]` because any future position would create a longer subarray with this starting point.

```python
while indices_deque and prefix_sums[indices_deque[-1]] >= current_sum:
    indices_deque.pop()
```
This maintains the monotonic property of the deque. If the last element in the deque has a [prefix sum](/problems/subarray_sum) greater than or equal to the current value `current_sum`, we remove it. This is because the current position `current_index` would always be a better starting point (shorter subarray with smaller or equal prefix sum).

```python
indices_deque.append(current_index)
```
Add the current index to the deque as a potential starting point for future positions.

**Step 4: Return Result**
```python
return -1 if min_length == float('inf') else min_length
```
If `min_length` remains infinity, no valid subarray was found, so return -1. Otherwise, return the minimum length found.

The algorithm processes each element once, and each index enters and leaves the deque at most once, resulting in O(n) time complexity with O(n) space complexity.

## Example Walkthrough

Let's trace through the algorithm with `nums = [2, -1, 2, 1]` and `k = 3`.

**Step 1: Build prefix sum array**
- `s = [0, 2, 1, 3, 4]`
- This represents cumulative sums: s[0]=0, s[1]=0+2=2, s[2]=2+(-1)=1, s[3]=1+2=3, s[4]=3+1=4

**Step 2: Process each prefix sum**

*Iteration i=0, v=0:*
- Deque `q = []`
- No elements to check for valid subarrays
- No elements to maintain monotonicity
- Add index 0: `q = [0]`
- `ans = inf`

*Iteration i=1, v=2:*
- Check front: `2 - 0 = 2 < 3`, no valid subarray
- Check monotonicity: `s[0] = 0 < 2`, keep it
- Add index 1: `q = [0, 1]`
- `ans = inf`

*Iteration i=2, v=1:*
- Check front: `1 - 0 = 1 < 3`, no valid subarray
- Check monotonicity: `s[1] = 2 >= 1`, remove index 1
- Now `q = [0]`
- Add index 2: `q = [0, 2]`
- `ans = inf`

*Iteration i=3, v=3:*
- Check front: `3 - 0 = 3 >= 3`, found valid subarray!
  - Length = `3 - 0 = 3`
  - Update `ans = 3`
  - Remove index 0: `q = [2]`
- Check front again: `3 - 1 = 2 < 3`, no more valid subarrays
- Check monotonicity: `s[2] = 1 < 3`, keep it
- Add index 3: `q = [2, 3]`

*Iteration i=4, v=4:*
- Check front: `4 - 1 = 3 >= 3`, found valid subarray!
  - Length = `4 - 2 = 2`
  - Update `ans = min(3, 2) = 2`
  - Remove index 2: `q = [3]`
- Check front again: `4 - 3 = 1 < 3`, no more valid subarrays
- Check monotonicity: `s[3] = 3 < 4`, keep it
- Add index 4: `q = [3, 4]`

**Result:** The shortest subarray with sum ≥ 3 has length 2 (the subarray `[2, 1]` from indices 2 to 3).

The deque maintains indices with increasing prefix sums, allowing us to efficiently find the shortest valid subarray by checking from the front (oldest, smallest prefix sum) and maintaining monotonicity from the back.

## Python Solution

```python
from collections import deque
from itertools import accumulate
from typing import List

class Solution:
    def shortestSubarray(self, nums: List[int], k: int) -> int:
        # Create prefix sum array with initial value 0
        # prefix_sums[i] represents sum of nums[0:i]
        prefix_sums = list(accumulate(nums, initial=0))

        # Deque to maintain indices in increasing order of prefix sum values
        # This helps us find the shortest valid subarray
        indices_deque = deque()

        # Initialize minimum length to infinity
        min_length = float('inf')

        # Iterate through each prefix sum value
        for current_index, current_sum in enumerate(prefix_sums):
            # Check if we can form a valid subarray ending at current_index
            # Remove indices from front while the subarray sum >= k
            while indices_deque and current_sum - prefix_sums[indices_deque[0]] >= k:
                # Update minimum length if current subarray is shorter
                min_length = min(min_length, current_index - indices_deque.popleft())

            # Maintain monotonic increasing property of deque
            # Remove indices from back if their prefix sum >= current sum
            # (They won't give us shorter subarrays in the future)
            while indices_deque and prefix_sums[indices_deque[-1]] >= current_sum:
                indices_deque.pop()

            # Add current index to the deque
            indices_deque.append(current_index)

        # Return -1 if no valid subarray found, otherwise return minimum length
        return -1 if min_length == float('inf') else min_length
```

## Time and Space Complexity

**Time Complexity:** `O(n)` where `n` is the length of the input array `nums`.

- Computing the prefix sum array using `accumulate` takes `O(n)` time.
- The main loop iterates through each element of the prefix sum array once, which is `O(n+1) = O(n)`.
- Within the loop:
  - Each element is added to the deque exactly once via `indices_deque.append(current_index)`.
  - Each element can be removed from the front of the deque at most once via `indices_deque.popleft()`.
  - Each element can be removed from the back of the deque at most once via `indices_deque.pop()`.
- Since each element enters and leaves the deque at most once, the total operations across all iterations is `O(n)`.
- Therefore, the overall time complexity is `O(n) + O(n) = O(n)`.

**Space Complexity:** `O(n)` where `n` is the length of the input array `nums`.

- The prefix sum array `prefix_sums` stores `n+1` elements, requiring `O(n)` space.
- The deque `indices_deque` can store at most `n+1` indices in the worst case, requiring `O(n)` space.
- Other variables (`min_length`, `current_index`, `current_sum`) use constant space `O(1)`.
- Therefore, the overall space complexity is `O(n) + O(n) + O(1) = O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Attempting Sliding Window with Negative Numbers**

The most common pitfall is trying to use a standard two-pointer sliding window approach when the array contains negative numbers. This approach works perfectly for arrays with only non-negative numbers but fails with negative values.

**Why it fails:** With negative numbers, the subarray sum doesn't monotonically increase as we expand the window. Adding more elements might actually decrease the sum, breaking the fundamental assumption of the sliding window technique.

**Example of incorrect approach:**
```python
# INCORRECT - This won't work with negative numbers!
def shortestSubarray_wrong(nums, k):
    left = 0
    current_sum = 0
    min_length = float('inf')

    for right in range(len(nums)):
        current_sum += nums[right]

        while current_sum >= k:
            min_length = min(min_length, right - left + 1)
            current_sum -= nums[left]
            left += 1

    return -1 if min_length == float('inf') else min_length
```

**Test case where it fails:**
```python
nums = [-2, 3]
k = 3
# The correct answer is 1 (the subarray [3])
# The window sum is -2, then 1. It never reaches k, so the window never
# shrinks and the negative prefix [-2] is never dropped. Returns -1.
```

### 2. **Not Maintaining Monotonic Property Correctly**

Another pitfall is forgetting to maintain the monotonic increasing property of the deque. Without this step, the algorithm becomes inefficient and might not find the optimal solution.

**Why we need it:** If we have two indices i < j where prefix_sum[i] >= prefix_sum[j], then j is always a better starting point for any future position because:
- It comes later (creating shorter subarrays)
- It has a smaller or equal prefix sum (making it easier to reach the target k)

### 3. **Incorrect Prefix Sum Initialization**

Using `accumulate(nums)` without `initial=0` creates a prefix sum array where the first element is nums[0] instead of 0. This makes it harder to handle subarrays starting from index 0.

**Incorrect:**
```python
prefix_sums = list(accumulate(nums))  # First element is nums[0]
```

**Correct:**
```python
prefix_sums = list(accumulate(nums, initial=0))  # First element is 0
```

### 4. **Not Handling Edge Cases**

Forgetting to check if the deque is empty before accessing elements can cause index errors:

```python
# Always check if deque is not empty before accessing
while indices_deque and current_sum - prefix_sums[indices_deque[0]] >= k:
    # Process...
```

**Solution:** The provided implementation correctly addresses all these pitfalls by:
1. Using a monotonic deque approach that handles negative numbers
2. Maintaining the deque's monotonic property by removing larger prefix sums from the back
3. Using `initial=0` in accumulate for proper prefix sum initialization
4. Always checking if the deque is non-empty before accessing elements

