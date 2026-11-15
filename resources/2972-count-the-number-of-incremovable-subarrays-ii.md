# 2972. Count the Number of Incremovable Subarrays II

> Source: https://algo.monster/liteproblems/2972

## Problem Statement

You are given a 0-indexed array of positive integers `nums`.

A subarray is called **incremovable** if removing it from the original array results in the remaining elements forming a strictly increasing sequence. For example, in the array `[5, 3, 4, 6, 7]`, the subarray `[3, 4]` is incremovable because removing it leaves `[5, 6, 7]`, which is strictly increasing.

Your task is to count the total number of incremovable subarrays in the given array `nums`.

Key points to understand:
- A subarray must be contiguous and non-empty
- After removing the subarray, the remaining array (which could be empty) must be strictly increasing
- An empty array is considered strictly increasing
- Strictly increasing means each element must be strictly greater than the previous one (no equal values allowed)

For instance, if we have `nums = [1, 2, 3, 4]`, which is already strictly increasing, any subarray we remove will leave either an empty array or a strictly increasing array, so all possible subarrays are incremovable. The total count would be `n * (n + 1) / 2 = 10` subarrays.

The challenge involves identifying which subarrays, when removed, maintain the strictly increasing property in the remaining elements. The remaining elements after removal could be:
- Only a prefix of the original array
- Only a suffix of the original array
- A combination of prefix and suffix elements
- An empty array (when the entire array is removed)

## Intuition

The key insight is to think about what remains after removing a subarray, rather than what we remove. After removing any subarray, we're left with either:
- Nothing (empty array)
- A prefix of the original array
- A suffix of the original array
- Both a prefix and a suffix concatenated together

Since an empty array and any strictly increasing sequence are valid, we need to count all possible ways to achieve these configurations.

Let's first consider the simplest case: if the entire array is already strictly increasing, then removing any subarray will leave a strictly increasing sequence (or empty). This gives us `n * (n + 1) / 2` total subarrays.

For the general case, we need to identify:
1. The longest strictly increasing prefix ending at index `i`
2. The longest strictly increasing suffix starting from the end

If we can only keep a prefix (removing everything from some point onwards), we can remove:
- `nums[i+1...n-1]` (keep the full increasing prefix)
- `nums[i...n-1]` (keep prefix minus last element)
- `nums[i-1...n-1]` (keep prefix minus last two elements)
- And so on, down to removing the entire array

This gives us `i + 2` possibilities when keeping only a prefix.

For keeping a suffix (with or without a prefix), we iterate through possible starting points `j` of the suffix from right to left. For each valid suffix starting at `j`, we need to find how many elements from the prefix can be kept such that the last prefix element is less than `nums[j]`. We use pointer `i` to track this.

The clever part is moving `i` backwards when needed to maintain `nums[i] < nums[j]`. For each valid suffix position `j`, we add `i + 2` to our count (representing all possible prefixes that can combine with this suffix, including the empty prefix).

We stop when we find `nums[j-1] >= nums[j]` because at that point, the suffix is no longer strictly increasing.

**Learn more about [Two Pointers](/problems/two_pointers_intro) and [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The implementation uses a two-pointer approach to efficiently count all incremovable subarrays.

**Step 1: Find the longest strictly increasing prefix**

```python
i, n = 0, len(nums)
while i + 1 < n and nums[i] < nums[i + 1]:
    i += 1
```

We iterate from the start to find the last index `i` where `nums[0] < nums[1] < ... < nums[i]`. This gives us the longest strictly increasing prefix.

**Step 2: Handle the special case**

```python
if i == n - 1:
    return n * (n + 1) // 2
```

If `i == n - 1`, the entire array is strictly increasing. Any subarray removal will result in a strictly increasing sequence, so we return the total number of subarrays: `n * (n + 1) // 2`.

**Step 3: Count subarrays when keeping only a prefix**

```python
ans = i + 2
```

We initialize our answer with `i + 2`, which represents all the ways to remove a suffix and keep only a prefix (or nothing):
- Remove `nums[i+1...n-1]` (keep full prefix)
- Remove `nums[i...n-1]` (keep prefix except last element)
- Remove `nums[i-1...n-1]` (keep prefix except last 2 elements)
- ... down to removing the entire array

**Step 4: Count subarrays when keeping a suffix (with or without prefix)**

```python
j = n - 1
while j:
    while i >= 0 and nums[i] >= nums[j]:
        i -= 1
    ans += i + 2
    if nums[j - 1] >= nums[j]:
        break
    j -= 1
```

We iterate `j` from right to left, treating `nums[j]` as the first element of the suffix we want to keep:

- For each position `j`, we adjust pointer `i` backwards until we find `nums[i] < nums[j]` (or `i` becomes -1)
- This ensures that if we keep prefix `nums[0...i]` and suffix `nums[j...n-1]`, the concatenation is strictly increasing
- We add `i + 2` to the answer:
  - `i + 1` ways to choose different prefix lengths (from empty to full prefix up to index `i`)
  - Plus 1 for removing everything before `j` (keeping only the suffix)
- We stop when `nums[j-1] >= nums[j]` because the suffix starting at `j-1` wouldn't be strictly increasing

The algorithm efficiently counts all valid configurations in `O(n)` time by cleverly reusing the pointer `i` as we move `j` leftward, avoiding redundant comparisons.

## Example Walkthrough

Let's walk through the solution with `nums = [2, 1, 3, 5, 4, 6]`.

**Step 1: Find longest strictly increasing prefix**
- Start with `i = 0`
- Check: `nums[0] = 2` < `nums[1] = 1`? No, so stop
- Longest prefix ends at `i = 0` (just element `[2]`)

**Step 2: Check if entire array is increasing**
- Since `i = 0` and `n - 1 = 5`, the array is not fully increasing
- Continue to count incremovable subarrays

**Step 3: Count subarrays keeping only prefix**
- `ans = i + 2 = 0 + 2 = 2`
- This counts:
  - Removing `[1, 3, 5, 4, 6]` → leaves `[2]` (strictly increasing)
  - Removing `[2, 1, 3, 5, 4, 6]` → leaves `[]` (empty, valid)

**Step 4: Count subarrays keeping suffix (with/without prefix)**

Start with `j = 5` (last element):
- `nums[j] = 6`
- Check prefix: `nums[0] = 2 < 6`? Yes, so `i` stays at 0
- Add `i + 2 = 0 + 2 = 2` to answer → `ans = 4`
  - This counts removing `[1, 3, 5, 4]` leaving `[2] + [6]`
  - And removing `[2, 1, 3, 5, 4]` leaving just `[6]`
- Check: `nums[4] = 4 < nums[5] = 6`? Yes, continue

Move to `j = 4`:
- `nums[j] = 4`
- Check prefix: `nums[0] = 2 < 4`? Yes, so `i` stays at 0
- Add `i + 2 = 2` → `ans = 6`
  - This counts removing `[1, 3, 5]` leaving `[2] + [4, 6]`
  - And removing `[2, 1, 3, 5]` leaving just `[4, 6]`
- Check: `nums[3] = 5 > nums[4] = 4`? Yes, STOP!
  - The suffix `[5, 4, 6]` is not strictly increasing

**Final answer: 6 incremovable subarrays**

These are:
1. `[2, 1, 3, 5, 4, 6]` → removes entire array → `[]`
2. `[1, 3, 5, 4, 6]` → removes suffix → `[2]`
3. `[1, 3, 5, 4]` → leaves `[2, 6]`
4. `[2, 1, 3, 5, 4]` → leaves `[6]`
5. `[1, 3, 5]` → leaves `[2, 4, 6]`
6. `[2, 1, 3, 5]` → leaves `[4, 6]`

## Python Solution

```python
class Solution:
    def incremovableSubarrayCount(self, nums: List[int]) -> int:
        # Find the longest strictly increasing prefix
        left_boundary = 0
        n = len(nums)

        # Extend the left boundary while array is strictly increasing
        while left_boundary + 1 < n and nums[left_boundary] < nums[left_boundary + 1]:
            left_boundary += 1

        # If entire array is strictly increasing, any subarray can be removed
        # Total subarrays = n * (n + 1) / 2
        if left_boundary == n - 1:
            return n * (n + 1) // 2

        # Count subarrays that can be removed
        # Start with subarrays that include everything from index (left_boundary + 1) onwards
        # Plus the empty removal and full array removal
        result = left_boundary + 2

        # Process from the right side
        right_pointer = n - 1

        while right_pointer > 0:
            # Find valid left boundaries that can connect with current right element
            # Move left boundary back while it's greater than or equal to right element
            while left_boundary >= 0 and nums[left_boundary] >= nums[right_pointer]:
                left_boundary -= 1

            # Add count of valid removals ending just before right_pointer
            # (left_boundary + 1) positions from start, plus empty prefix
            result += left_boundary + 2

            # Stop if right side is no longer strictly increasing
            if nums[right_pointer - 1] >= nums[right_pointer]:
                break

            right_pointer -= 1

        return result
```

## Time and Space Complexity

**Time Complexity: `O(n)`**

The algorithm consists of three main parts:

1. The first while loop iterates from the beginning of the array to find the longest strictly increasing prefix. This takes at most `O(n)` iterations.

2. If the entire array is strictly increasing (checked by `if i == n - 1`), we return immediately with a constant time calculation.

3. The second while loop starts from the end of the array and moves backwards. In the worst case, variable `j` iterates through all positions from `n-1` to `1`. For each position of `j`, the inner while loop adjusts pointer `i` backwards. The key observation is that pointer `i` only moves backwards throughout the entire execution - it starts at some position and can only decrease. Therefore, across all iterations of the outer loop, the inner while loop performs at most `O(n)` operations in total, not `O(n)` per iteration of the outer loop.

Since each element is visited at most a constant number of times across all operations, the overall time complexity is `O(n)`.

**Space Complexity: `O(1)`**

The algorithm only uses a fixed number of variables (`i`, `j`, `n`, and `ans`) regardless of the input size. No additional data structures like arrays, lists, or hash maps are created. All operations are performed in-place using only these constant extra variables, resulting in `O(1)` space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Boundary Condition When Connecting Prefix and Suffix**

**The Pitfall:**
A common mistake is using the wrong comparison operator when checking if a prefix can connect with a suffix. Developers might incorrectly write:

```python
while left_boundary >= 0 and nums[left_boundary] > nums[right_pointer]:
    left_boundary -= 1
```

This would check for `nums[left_boundary] > nums[right_pointer]` instead of `nums[left_boundary] >= nums[right_pointer]`. The issue is that we need strict inequality for a valid connection (the last element of the prefix must be strictly less than the first element of the suffix).

**Why It Happens:**
The confusion arises because we're looking for strictly increasing sequences, so developers might think "strictly greater" is the right condition. However, we're actually moving the left boundary backwards to find where it becomes strictly less than the right element.

**The Fix:**
Use the correct condition:
```python
while left_boundary >= 0 and nums[left_boundary] >= nums[right_pointer]:
    left_boundary -= 1
```
After this loop, `left_boundary` points to the rightmost position where `nums[left_boundary] < nums[right_pointer]`, ensuring a valid strictly increasing connection.

### 2. **Not Handling the Edge Case of Already Strictly Increasing Array**

**The Pitfall:**
Forgetting to handle the special case where the entire array is already strictly increasing can lead to incorrect counting. Without this check, the algorithm might undercount valid subarrays.

```python
# Missing this crucial check:
if left_boundary == n - 1:
    return n * (n + 1) // 2
```

**Why It Happens:**
When the array is entirely strictly increasing, every single subarray is incremovable (removing any subarray leaves either an empty array or a strictly increasing array). The general algorithm logic doesn't naturally handle this case.

**The Fix:**
Always check if `left_boundary == n - 1` after finding the longest strictly increasing prefix. If true, return the total number of subarrays immediately.

### 3. **Resetting the Left Boundary for Each Right Position**

**The Pitfall:**
Some might reset `left_boundary` to its original value for each new `right_pointer` position:

```python
right_pointer = n - 1
while right_pointer > 0:
    left_boundary = original_left_boundary  # Wrong! Don't reset
    while left_boundary >= 0 and nums[left_boundary] >= nums[right_pointer]:
        left_boundary -= 1
    # ...
```

**Why It Happens:**
It seems logical to start fresh for each suffix position, but this creates O(n²) time complexity and is unnecessary.

**The Fix:**
Reuse the `left_boundary` position from the previous iteration. As `right_pointer` moves left, the values typically decrease (in a strictly increasing suffix), so the valid `left_boundary` can only move left or stay the same. This optimization maintains O(n) time complexity:

```python
# Don't reset left_boundary; let it continue from where it was
while right_pointer > 0:
    while left_boundary >= 0 and nums[left_boundary] >= nums[right_pointer]:
        left_boundary -= 1
    # ...
```

### 4. **Off-by-One Error in Counting**

**The Pitfall:**
Incorrectly calculating the number of valid removals for each configuration:

```python
result = left_boundary + 1  # Wrong! Should be left_boundary + 2
# or later:
result += left_boundary + 1  # Wrong! Should be left_boundary + 2
```

**Why It Happens:**
The confusion comes from counting the different ways to remove subarrays:
- When `left_boundary = i`, we can keep prefixes of length 0, 1, 2, ..., i, which is `i + 1` options
- Plus we need to count the case where we keep only the suffix (no prefix), adding 1 more
- Total: `i + 2` options

**The Fix:**
Always use `left_boundary + 2` when adding to the result, accounting for all valid prefix lengths (including empty) plus the suffix-only option.

