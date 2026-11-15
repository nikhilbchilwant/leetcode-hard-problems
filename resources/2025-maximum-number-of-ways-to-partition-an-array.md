# 2025. Maximum Number of Ways to Partition an Array

> Source: https://algo.monster/liteproblems/2025

## Problem Statement

You are given a 0-indexed integer array `nums` of length `n`. A **pivot** index is a position that divides the array into two parts with equal sums. Specifically, a pivot index must satisfy:

1. The index is between 1 and n-1 (inclusive): `1 <= pivot < n`
2. The sum of elements before the pivot equals the sum of elements from the pivot onwards: `nums[0] + nums[1] + ... + nums[pivot - 1] == nums[pivot] + nums[pivot + 1] + ... + nums[n - 1]`

Additionally, you are given an integer `k`. You have the option to:
- Change exactly **one** element in `nums` to the value `k`, OR
- Leave the array unchanged

Your task is to find the **maximum** possible number of valid pivot indices (ways to partition the array) after optionally changing at most one element to `k`.

For example, if changing a certain element to `k` creates 3 valid pivot indices, while leaving the array unchanged only has 1 valid pivot index, you should return 3.

The problem asks you to explore all possibilities - either keeping the array as is, or changing each element one at a time to `k` - and determine which scenario gives you the maximum number of valid partitions.

## Intuition

Let's think about what happens when we have a valid pivot. At a pivot index `i`, the sum of elements before it equals the sum of elements from position `i` onwards. Using prefix sums, this means: `prefix[i] = total_sum - prefix[i]`, which simplifies to `prefix[i] = total_sum / 2`. This tells us that for a valid pivot to exist without any modifications, the total sum must be even, and we need to find positions where the [prefix sum](/problems/subarray_sum) equals half of the total.

Now, what happens when we change one element? If we change `nums[j]` from its original value to `k`, the difference is `d = k - nums[j]`. This change affects the total sum and the prefix sums differently depending on where the change occurs relative to potential pivot points.

For pivots to the left of position `j` (pivot < j), the left side sum remains unchanged, but the right side sum increases by `d`. The condition becomes: `prefix[pivot] = (total_sum + d) - prefix[pivot]`, which means `prefix[pivot] = (total_sum + d) / 2`.

For pivots to the right of position `j` (pivot > j), the left side sum increases by `d`. The [prefix sum](/problems/subarray_sum) at that pivot also increases by `d`. The condition becomes: `prefix[pivot] + d = (total_sum + d) - (prefix[pivot] + d)`, which simplifies to `prefix[pivot] = (total_sum - d) / 2`.

The key insight is that we can precompute how many times each [prefix sum](/problems/subarray_sum) value appears to the left and right of any position. As we iterate through each potential modification position, we maintain two hash tables: one tracking prefix sum frequencies to the left, and another for the right. This allows us to quickly count how many valid pivots we'd create by making that specific change.

By checking both the unchanged scenario and all possible single-element modifications, we can determine the maximum number of valid partitions achievable.

**Learn more about [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

We implement the solution using prefix sums and hash tables to efficiently count valid pivots.

**Step 1: Build [Prefix Sum](/problems/subarray_sum) Array**
First, we create a prefix sum array `s` where `s[i]` represents the sum of elements from index 0 to i. The total sum of the array is `s[n-1]`.

**Step 2: Initialize Right Hash Table**
We populate a hash table `right` with the frequency of each [prefix sum](/problems/subarray_sum) value. Initially, all prefix sums (except `s[0]`) are considered to be on the "right side" of our current position.

**Step 3: Handle the Unchanged Case**
If we don't modify any element, a valid pivot exists at position `i` when `s[i] = s[n-1] / 2`. This only works when the total sum `s[n-1]` is even. We count how many prefix sums equal `s[n-1] / 2` using `right[s[n-1] // 2]`.

**Step 4: Iterate Through Each Potential Modification**
For each position `i` where we might change `nums[i]` to `k`:

- Calculate the difference: `d = k - nums[i]`
- The new total sum would be: `s[n-1] + d`

For this modification to create valid pivots:
- Pivots to the left of `i` need: `prefix = (s[n-1] + d) / 2`
- Pivots to the right of `i` need: `prefix = (s[n-1] - d) / 2`

We check if `(s[n-1] + d)` is even (otherwise no valid pivots exist). If it is:
- Count pivots on the left: `left[(s[n-1] + d) // 2]`
- Count pivots on the right: `right[(s[n-1] - d) // 2]`
- Total valid pivots for this modification: `left[(s[n-1] + d) // 2] + right[(s[n-1] - d) // 2]`

**Step 5: Update Hash Tables**
After processing position `i`, we move its [prefix sum](/problems/subarray_sum) `s[i]` from the "right" hash table to the "left" hash table:
- `left[s[i]] += 1`
- `right[s[i]] -= 1`

This maintains the invariant that `left` contains prefix sums before our current position and `right` contains those after.

**Step 6: Track Maximum**
Throughout the process, we keep track of the maximum number of valid pivots found across all scenarios (unchanged or any single modification).

The time complexity is `O(n)` as we iterate through the array once, and hash table operations are `O(1)` on average. The space complexity is `O(n)` for storing the prefix sums and hash tables.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example:** `nums = [1, 2, 3, 4]`, `k = 5`

**Step 1: Build prefix sum array**
- `s[0] = 1`
- `s[1] = 1 + 2 = 3`
- `s[2] = 1 + 2 + 3 = 6`
- `s[3] = 1 + 2 + 3 + 4 = 10` (total sum)

**Step 2: Initialize hash tables**
- `left = {}` (empty initially)
- `right = {3: 1, 6: 1}` (contains s[1] and s[2], which are potential pivot positions)

**Step 3: Check unchanged case**
- Total sum = 10, so we need prefix sum = 10/2 = 5 for a valid pivot
- Check `right[5]` = 0 (no prefix sum equals 5)
- Unchanged case: 0 valid pivots

**Step 4: Iterate through modification positions**

*Position i=0: Change nums[0] from 1 to 5*
- `d = 5 - 1 = 4`
- New total = 10 + 4 = 14 (even, so valid pivots possible)
- Need left pivots with prefix = 14/2 = 7: `left[7]` = 0
- Need right pivots with prefix = 6/2 = 3: `right[3]` = 1
- Total for this modification: 0 + 1 = 1 valid pivot

*Update hash tables:*
- Move s[0]=1 from right to left: `left = {1: 1}`, `right = {3: 1, 6: 1}`

*Position i=1: Change nums[1] from 2 to 5*
- `d = 5 - 2 = 3`
- New total = 10 + 3 = 13 (odd, no valid pivots possible)
- Total for this modification: 0 valid pivots

*Update hash tables:*
- Move s[1]=3 from right to left: `left = {1: 1, 3: 1}`, `right = {3: 0, 6: 1}`

*Position i=2: Change nums[2] from 3 to 5*
- `d = 5 - 3 = 2`
- New total = 10 + 2 = 12 (even, so valid pivots possible)
- Need left pivots with prefix = 12/2 = 6: `left[6]` = 0
- Need right pivots with prefix = 8/2 = 4: `right[4]` = 0
- Total for this modification: 0 + 0 = 0 valid pivots

*Update hash tables:*
- Move s[2]=6 from right to left: `left = {1: 1, 3: 1, 6: 1}`, `right = {3: 0, 6: 0}`

*Position i=3: Change nums[3] from 4 to 5*
- `d = 5 - 4 = 1`
- New total = 10 + 1 = 11 (odd, no valid pivots possible)
- Total for this modification: 0 valid pivots

**Step 5: Return maximum**
- Maximum across all scenarios: max(0, 1, 0, 0, 0) = 1

The answer is 1, achieved by changing nums[0] to 5, which creates array [5, 2, 3, 4]. This has one valid pivot at index 1, where the left sum (5) equals the right sum (2 + 3 = 5).

## Python Solution

```python
class Solution:
    def waysToPartition(self, nums: List[int], k: int) -> int:
        n = len(nums)
      
        # Build prefix sum array
        prefix_sum = [nums[0]] * n
        for i in range(1, n):
            prefix_sum[i] = prefix_sum[i - 1] + nums[i]
      
        # Count occurrences of each prefix sum value (excluding the last one)
        # These represent potential pivot points on the right side
        right_pivots = defaultdict(int)
        for i in range(n - 1):
            right_pivots[prefix_sum[i]] += 1
      
        # Calculate initial answer without changing any element
        max_ways = 0
        total_sum = prefix_sum[-1]
        if total_sum % 2 == 0:
            # Count how many pivots create equal partitions
            max_ways = right_pivots[total_sum // 2]
      
        # Try changing each element to k and count valid partitions
        left_pivots = defaultdict(int)
        for i in range(n):
            current_prefix = prefix_sum[i]
            original_value = nums[i]
          
            # Calculate the difference if we change nums[i] to k
            difference = k - original_value
            new_total_sum = total_sum + difference
          
            # Check if new total sum allows equal partitions
            if new_total_sum % 2 == 0:
                target_sum = new_total_sum // 2
              
                # Count valid pivots:
                # - Left pivots that already equal target_sum
                # - Right pivots that would equal target_sum after adjustment
                valid_partitions = left_pivots[target_sum] + right_pivots[target_sum - difference]
              
                # Update maximum ways
                if max_ways < valid_partitions:
                    max_ways = valid_partitions
          
            # Move current prefix from right to left for next iteration
            left_pivots[current_prefix] += 1
            right_pivots[current_prefix] -= 1
      
        return max_ways
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm consists of several sequential operations:
- Initial prefix sum calculation loop iterates through `n-1` elements: `O(n)`
- Building the `right` dictionary during prefix sum calculation: `O(n)` 
- Checking if total sum is even and accessing dictionary: `O(1)`
- Main loop iterates through all `n` elements, where each iteration performs:
  - Calculate difference `d`: `O(1)`
  - Check divisibility and compute potential answer: `O(1)`
  - Dictionary lookups and updates for `left` and `right`: `O(1)` average case
  - Update maximum answer: `O(1)`

Since all operations are either `O(1)` or sequential `O(n)` loops, the overall time complexity is `O(n)`.

**Space Complexity:** `O(n)`

The space usage includes:
- Prefix sum array `s` of size `n`: `O(n)`
- Dictionary `right` storing at most `n-1` unique prefix sums: `O(n)` worst case
- Dictionary `left` storing at most `n` unique prefix sums: `O(n)` worst case
- Other variables (`n`, `ans`, `d`, `t`, `v`, `x`): `O(1)`

The dominant space usage comes from the arrays and dictionaries, resulting in a total space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall: Incorrectly Handling the Pivot Point Movement

A critical issue in this problem is understanding how changing an element affects different pivot points. When we change `nums[i]` to `k`, the effect on prefix sums differs based on whether the pivot is before or after position `i`:

**The Problem:**
- For pivots **before** position `i`: The left sum remains unchanged, but the right sum changes by `difference = k - nums[i]`
- For pivots **after** position `i`: Both left and right sums change by the difference

Many implementations incorrectly assume all pivots are affected the same way, leading to wrong counts.

**Why This Happens:**
When we change `nums[i]` to `k`:
- All prefix sums from position `i` onwards increase by `difference`
- Prefix sums before position `i` remain unchanged
- This creates an asymmetric effect on the partition validity

**Correct Solution:**
The code handles this by maintaining two separate hash tables (`left_pivots` and `right_pivots`) and applying different logic:

```python
# For pivots to the left of i (already processed)
# Their prefix sum stays the same, but total changes
# So they need: prefix_sum = (new_total_sum) / 2
valid_left = left_pivots[new_total_sum // 2]

# For pivots to the right of i (not yet processed)
# Their prefix sum will increase by difference
# So currently they need: prefix_sum = (new_total_sum / 2) - difference
valid_right = right_pivots[(new_total_sum // 2) - difference]
```

**Example to Illustrate:**
Consider `nums = [2, 3, 1, 4]` and we want to change `nums[2] = 1` to `k = 7`:
- Original prefix sums: `[2, 5, 6, 10]`
- After change, prefix sums would be: `[2, 5, 12, 16]` (positions 2 and 3 increase by 6)
- For pivot at position 1: Left sum = 2 (unchanged), Right sum = 14 (changed from 8)
- For pivot at position 2: Left sum = 5 (unchanged), Right sum = 11 (changed from 5)
- For pivot at position 3: Left sum = 12 (changed from 6), Right sum = 4 (unchanged)

The asymmetric nature of these changes is why we need to track pivots on the left and right separately and apply different calculations to each group.

