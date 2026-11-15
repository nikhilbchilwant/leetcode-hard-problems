# 1671. Minimum Number of Removals to Make Mountain Array

> Source: https://algo.monster/liteproblems/1671

## Problem Statement

You need to find the minimum number of elements to remove from an array to make it a "mountain array".

A **mountain array** is defined as an array that:
- Has at least 3 elements
- Has a peak element at some index `i` (where `0 < i < arr.length - 1`)
- All elements before the peak are in strictly increasing order: `arr[0] < arr[1] < ... < arr[i-1] < arr[i]`
- All elements after the peak are in strictly decreasing order: `arr[i] > arr[i+1] > ... > arr[arr.length-1]`

Given an integer array `nums`, you need to determine the minimum number of elements that must be removed to transform it into a valid mountain array.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) to find the longest mountain subsequence. It calculates:
- `left[i]`: the length of the longest increasing subsequence ending at position `i`
- `right[i]`: the length of the longest decreasing subsequence starting at position `i`

For each position `i` that can serve as a peak (where `left[i] > 1` and `right[i] > 1`), the length of the mountain subsequence with peak at `i` is `left[i] + right[i] - 1`. The answer is the total length `n` minus the maximum mountain subsequence length found.

## Intuition

The key insight is to think about what we want to keep rather than what we want to remove. Instead of directly calculating removals, we can find the longest valid mountain subsequence and then subtract its length from the total array length.

A mountain has two parts: an ascending slope and a descending slope. Any valid mountain subsequence must have elements that form an increasing sequence up to a peak, then a decreasing sequence down from that peak.

For each potential peak position `i`, we need to know:
- How long can the ascending part be if it ends at position `i`?
- How long can the descending part be if it starts at position `i`?

This naturally leads us to the Longest Increasing Subsequence (LIS) problem. We can compute:
- `left[i]`: the LIS ending at position `i` (the ascending part)
- `right[i]`: the Longest Decreasing Subsequence starting at position `i` (the descending part)

If position `i` is a valid peak, it must have at least one element before it in the ascending part (`left[i] > 1`) and at least one element after it in the descending part (`right[i] > 1`).

The total length of a mountain with peak at `i` would be `left[i] + right[i] - 1` (we subtract 1 because the peak is counted in both sequences).

By finding the maximum possible mountain length across all valid peaks, we can determine the minimum removals needed: `n - max(mountain_length)`.

**Learn more about [Greedy](/problems/greedy_intro), [Binary Search](/problems/binary_search_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) to compute the longest increasing and decreasing subsequences for each position.

**Step 1: Initialize arrays**
- Create `left[i]` array to store the length of the longest increasing subsequence ending at position `i`
- Create `right[i]` array to store the length of the longest decreasing subsequence starting at position `i`
- Initialize both arrays with 1s (each element is a subsequence of length 1 by itself)

**Step 2: Compute longest increasing subsequences (left to right)**
```python
for i in range(1, n):
    for j in range(i):
        if nums[i] > nums[j]:
            left[i] = max(left[i], left[j] + 1)
```
For each position `i`, check all previous positions `j < i`. If `nums[i] > nums[j]`, we can extend the increasing subsequence ending at `j` by including `nums[i]`. Take the maximum length possible.

**Step 3: Compute longest decreasing subsequences (right to left)**
```python
for i in range(n - 2, -1, -1):
    for j in range(i + 1, n):
        if nums[i] > nums[j]:
            right[i] = max(right[i], right[j] + 1)
```
For each position `i`, check all positions `j > i`. If `nums[i] > nums[j]`, we can extend the decreasing subsequence starting at `j` by including `nums[i]` at the beginning. Take the maximum length possible.

**Step 4: Find the maximum mountain length**
```python
return n - max(a + b - 1 for a, b in zip(left, right) if a > 1 and b > 1)
```
For each position that can be a valid peak (`left[i] > 1` and `right[i] > 1`), calculate the mountain length as `left[i] + right[i] - 1`. The minimum removals equal `n` minus the maximum mountain length found.

The time complexity is `O(n²)` due to the nested loops in computing the subsequences, and the space complexity is `O(n)` for storing the `left` and `right` arrays.

## Example Walkthrough

Let's walk through the solution with a concrete example: `nums = [2, 1, 5, 6, 2, 3, 1]`

**Step 1: Initialize arrays**
- `n = 7`
- `left = [1, 1, 1, 1, 1, 1, 1]` (each element forms a subsequence of length 1)
- `right = [1, 1, 1, 1, 1, 1, 1]`

**Step 2: Compute longest increasing subsequences (left array)**

For `i = 1`: nums[1]=1
- Check j=0: nums[1]=1 not > nums[0]=2, skip
- `left[1] = 1`

For `i = 2`: nums[2]=5
- Check j=0: nums[2]=5 > nums[0]=2, so left[2] = max(1, left[0]+1) = 2
- Check j=1: nums[2]=5 > nums[1]=1, so left[2] = max(2, left[1]+1) = 2
- `left[2] = 2`

For `i = 3`: nums[3]=6
- Check j=0: nums[3]=6 > nums[0]=2, so left[3] = max(1, left[0]+1) = 2
- Check j=1: nums[3]=6 > nums[1]=1, so left[3] = max(2, left[1]+1) = 2
- Check j=2: nums[3]=6 > nums[2]=5, so left[3] = max(2, left[2]+1) = 3
- `left[3] = 3`

Continuing this process:
- `left[4] = 2` (can extend from nums[0]=2 or nums[1]=1)
- `left[5] = 3` (best is 1→2→3 or 2→3)
- `left[6] = 1` (1 is smallest, can't extend any sequence)

Final: `left = [1, 1, 2, 3, 2, 3, 1]`

**Step 3: Compute longest decreasing subsequences (right array)**

For `i = 5`: nums[5]=3
- Check j=6: nums[5]=3 > nums[6]=1, so right[5] = max(1, right[6]+1) = 2
- `right[5] = 2`

For `i = 4`: nums[4]=2
- Check j=5: nums[4]=2 not > nums[5]=3, skip
- Check j=6: nums[4]=2 > nums[6]=1, so right[4] = max(1, right[6]+1) = 2
- `right[4] = 2`

For `i = 3`: nums[3]=6
- Check j=4: nums[3]=6 > nums[4]=2, so right[3] = max(1, right[4]+1) = 3
- Check j=5: nums[3]=6 > nums[5]=3, so right[3] = max(3, right[5]+1) = 3
- Check j=6: nums[3]=6 > nums[6]=1, so right[3] = max(3, right[6]+1) = 3
- `right[3] = 3`

Continuing backwards:
- `right[2] = 4` (can go 5→2→1 or 5→3→1)
- `right[1] = 1` (1 cannot start a decreasing sequence to any element after it)
- `right[0] = 5` (can go 2→1 directly, or through other paths)

Final: `right = [5, 1, 4, 3, 2, 2, 1]`

**Step 4: Find maximum mountain length**

Check each position as a potential peak:
- i=0: left[0]=1, right[0]=5 → Not valid (left[0] ≤ 1)
- i=1: left[1]=1, right[1]=1 → Not valid (both ≤ 1)
- i=2: left[2]=2, right[2]=4 → Valid! Mountain length = 2 + 4 - 1 = 5
- i=3: left[3]=3, right[3]=3 → Valid! Mountain length = 3 + 3 - 1 = 5
- i=4: left[4]=2, right[4]=2 → Valid! Mountain length = 2 + 2 - 1 = 3
- i=5: left[5]=3, right[5]=2 → Valid! Mountain length = 3 + 2 - 1 = 4
- i=6: left[6]=1, right[6]=1 → Not valid (both ≤ 1)

Maximum mountain length = 5

**Answer: 7 - 5 = 2** (minimum removals needed)

One possible mountain subsequence of length 5 would be [2, 5, 6, 2, 1] with peak at 6.

## Python Solution

```python
class Solution:
    def minimumMountainRemovals(self, nums: List[int]) -> int:
        n = len(nums)

        # left[i] stores the length of the longest increasing subsequence ending at index i
        left = [1] * n

        # right[i] stores the length of the longest decreasing subsequence starting at index i
        right = [1] * n

        # Calculate LIS (Longest Increasing Subsequence) for each position from left
        for i in range(1, n):
            for j in range(i):
                if nums[i] > nums[j]:
                    left[i] = max(left[i], left[j] + 1)

        # Calculate LIS from right (which gives us longest decreasing subsequence from left perspective)
        for i in range(n - 2, -1, -1):
            for j in range(i + 1, n):
                if nums[i] > nums[j]:
                    right[i] = max(right[i], right[j] + 1)

        # Find the maximum mountain length
        # A valid mountain must have left[i] > 1 and right[i] > 1 (peak cannot be at endpoints)
        # Mountain length at position i = left[i] + right[i] - 1 (subtract 1 as peak is counted twice)
        max_mountain_length = max(
            left[i] + right[i] - 1
            for i in range(n)
            if left[i] > 1 and right[i] > 1
        )

        # Minimum removals = total elements - maximum mountain length
        return n - max_mountain_length
```

## Time and Space Complexity

The time complexity is `O(n²)`, where `n` is the length of the array `nums`. This is because:
- The first nested loop (computing `left` array) iterates through each element `i` from 1 to n-1, and for each `i`, it checks all previous elements `j` from 0 to i-1, resulting in `O(n²)` operations.
- The second nested loop (computing `right` array) iterates through each element `i` from n-2 to 0, and for each `i`, it checks all subsequent elements `j` from i+1 to n-1, also resulting in `O(n²)` operations.
- The final list comprehension to find the maximum value runs in `O(n)` time.
- Overall: `O(n²) + O(n²) + O(n) = O(n²)`

The space complexity is `O(n)`, where `n` is the length of the array `nums`. This is because:
- The `left` array uses `O(n)` space to store the length of the longest increasing subsequence ending at each position.
- The `right` array uses `O(n)` space to store the length of the longest decreasing subsequence starting at each position.
- The list comprehension creates temporary values but doesn't store them, so it doesn't add to the space complexity.
- Overall: `O(n) + O(n) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to validate peak position constraints**
A common mistake is not checking that `left[i] > 1` and `right[i] > 1` when finding the maximum mountain length. This validation ensures:
- The peak is not at the first or last position (mountain needs at least one element on each side)
- There actually exists an increasing sequence before the peak and a decreasing sequence after it

**Incorrect approach:**
```python
# This would include invalid mountains with peaks at endpoints
max_mountain_length = max(left[i] + right[i] - 1 for i in range(n))
```

**Correct approach:**
```python
max_mountain_length = max(
    left[i] + right[i] - 1
    for i in range(n)
    if left[i] > 1 and right[i] > 1
)
```

### 2. **Double-counting the peak element**
When calculating the mountain length, remember that the peak element is counted in both `left[i]` (as the end of increasing subsequence) and `right[i]` (as the start of decreasing subsequence). Failing to subtract 1 leads to an incorrect mountain length.

**Incorrect:**
```python
mountain_length = left[i] + right[i]  # Peak counted twice!
```

**Correct:**
```python
mountain_length = left[i] + right[i] - 1  # Subtract 1 for the peak
```

### 3. **Using non-strict comparisons**
Mountain arrays require strictly increasing and strictly decreasing sequences. Using `>=` or `<=` instead of `>` would allow duplicate values, violating the mountain array definition.

**Incorrect:**
```python
if nums[i] >= nums[j]:  # This allows equal elements
    left[i] = max(left[i], left[j] + 1)
```

**Correct:**
```python
if nums[i] > nums[j]:  # Strictly greater for valid mountain
    left[i] = max(left[i], left[j] + 1)
```

### 4. **Edge case: No valid mountain exists**
If the array cannot form any valid mountain (e.g., all elements are equal or monotonic), the code might fail. Always ensure the generator expression has valid values:

**Safe implementation:**
```python
valid_mountains = [
    left[i] + right[i] - 1
    for i in range(n)
    if left[i] > 1 and right[i] > 1
]

if not valid_mountains:
    # Handle case where no valid mountain can be formed
    # This might mean removing all but 3 elements in worst case
    return n - 3 if n >= 3 else 0

return n - max(valid_mountains)
```

