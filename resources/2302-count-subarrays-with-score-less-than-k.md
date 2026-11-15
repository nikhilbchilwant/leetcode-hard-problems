# 2302. Count Subarrays With Score Less Than K

> Source: https://algo.monster/liteproblems/2302

## Problem Statement

You are given a positive integer array `nums` and an integer `k`. The **score** of an array is calculated by multiplying the sum of its elements by its length.

For example, if we have an array `[1, 2, 3, 4, 5]`:
- Sum of elements = `1 + 2 + 3 + 4 + 5 = 15`
- Length = `5`
- Score = `15 × 5 = 75`

Your task is to count how many non-empty subarrays of `nums` have a score that is **strictly less than** `k`.

A subarray is a contiguous sequence of elements from the original array. For instance, if `nums = [1, 2, 3, 4]`, then `[2, 3]` and `[1, 2, 3]` are subarrays, but `[1, 3]` is not (elements are not contiguous).

The function should return the total count of all such subarrays whose score is less than `k`.

## Intuition

When we need to count subarrays with a certain property, a brute force approach would be to check every possible subarray. However, this would be inefficient as there are `O(n²)` subarrays.

Let's think about the structure of the problem. For a subarray from index `i` to `j`, the score is `(sum of elements) × (length)`. If we fix the ending position `j`, we need to find all valid starting positions `i` where the score is less than `k`.

Notice an important property: as we extend a subarray to the left (decreasing the starting index while keeping the ending fixed), both the sum and the length increase. This means the score increases monotonically. In other words, if a subarray of length `L` ending at position `j` has a score ≥ `k`, then all longer subarrays ending at `j` will also have scores ≥ `k`.

This monotonic property is perfect for [binary search](/problems/binary_search_intro)! For each ending position, we can use binary search to find the maximum length `l` such that the subarray of length `l` ending at that position has a score less than `k`. Then all subarrays of length `1, 2, ..., l` ending at that position are valid.

To efficiently calculate the sum of any subarray, we can use prefix sums. With [prefix sum](/problems/subarray_sum) array `s`, the sum of elements from index `i` to `j` is simply `s[j+1] - s[i]`.

So the approach becomes: for each ending position, [binary search](/problems/binary_search_intro) for the maximum valid length, and add that length to our answer. The sum of all these lengths gives us the total count of valid subarrays.

**Learn more about [Binary Search](/problems/binary_search_intro), [Prefix Sum](/problems/subarray_sum) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

The solution uses **[prefix sum](/problems/subarray_sum)** combined with **[binary search](/problems/binary_search_intro)** to efficiently count the valid subarrays.

**Step 1: Build [Prefix Sum](/problems/subarray_sum) Array**

First, we create a [prefix sum](/problems/subarray_sum) array `s` where `s[i]` represents the sum of the first `i` elements of `nums`. We initialize it with `0` at index `0` for easier calculation. This allows us to compute the sum of any subarray in `O(1)` time: the sum from index `i` to `j` is `s[j+1] - s[i]`.

```python
s = list(accumulate(nums, initial=0))
```

**Step 2: Binary Search Using first_true_index Template**

For each ending position `i`, we find the **maximum** valid length where score < k. This is a "last true" pattern, so we invert the condition: find the first length where score >= k.

Using the first_true_index template:
```python
def find_max_valid_length(end_idx):
    left, right = 1, end_idx
    first_true_index = end_idx + 1  # Default: all lengths are valid

    while left <= right:
        mid = (left + right) // 2
        subarray_sum = prefix_sums[end_idx] - prefix_sums[end_idx - mid]
        if subarray_sum * mid >= k:  # Inverted: score is TOO LARGE
            first_true_index = mid
            right = mid - 1
        else:
            left = mid + 1

    # Maximum valid length = first_true_index - 1
    return first_true_index - 1
```

**Counting Logic:**
- `first_true_index` gives the first length where score >= k (invalid)
- `first_true_index - 1` gives the maximum length where score < k (valid)
- All subarrays of length 1, 2, ..., (first_true_index - 1) are valid

**Time Complexity:** `O(n log n)` - binary search per position.
**Space Complexity:** `O(n)` - prefix sum array.

## Example Walkthrough

Let's walk through the solution with `nums = [2, 1, 4, 3, 5]` and `k = 10`.

**Step 1: Build Prefix Sum Array**
- Original array: `[2, 1, 4, 3, 5]`
- Prefix sum `s`: `[0, 2, 3, 7, 10, 15]`
  - `s[0] = 0` (initial value)
  - `s[1] = 0 + 2 = 2`
  - `s[2] = 2 + 1 = 3`
  - `s[3] = 3 + 4 = 7`
  - `s[4] = 7 + 3 = 10`
  - `s[5] = 10 + 5 = 15`

**Step 2-4: Process Each Ending Position**

**Position i = 1** (subarrays ending at index 0 of original array):
- Binary search for max length where score < 10
- Try length 1: sum = `s[1] - s[0] = 2 - 0 = 2`, score = `2 × 1 = 2 < 10` ✓
- Maximum valid length = 1
- Valid subarrays: `[2]`
- Count: 1

**Position i = 2** (subarrays ending at index 1):
- Binary search range: length 0 to 2
- Try length 2: sum = `s[2] - s[0] = 3 - 0 = 3`, score = `3 × 2 = 6 < 10` ✓
- Maximum valid length = 2
- Valid subarrays: `[1]` (length 1), `[2,1]` (length 2)
- Count: 2

**Position i = 3** (subarrays ending at index 2):
- Binary search range: length 0 to 3
- Try length 2: sum = `s[3] - s[1] = 7 - 2 = 5`, score = `5 × 2 = 10 ≥ 10` ✗
- Try length 1: sum = `s[3] - s[2] = 7 - 3 = 4`, score = `4 × 1 = 4 < 10` ✓
- Maximum valid length = 1
- Valid subarrays: `[4]`
- Count: 1

**Position i = 4** (subarrays ending at index 3):
- Binary search range: length 0 to 4
- Try length 2: sum = `s[4] - s[2] = 10 - 3 = 7`, score = `7 × 2 = 14 ≥ 10` ✗
- Try length 1: sum = `s[4] - s[3] = 10 - 7 = 3`, score = `3 × 1 = 3 < 10` ✓
- Maximum valid length = 1
- Valid subarrays: `[3]`
- Count: 1

**Position i = 5** (subarrays ending at index 4):
- Binary search range: length 0 to 5
- Try length 3: sum = `s[5] - s[2] = 15 - 3 = 12`, score = `12 × 3 = 36 ≥ 10` ✗
- Try length 1: sum = `s[5] - s[4] = 15 - 10 = 5`, score = `5 × 1 = 5 < 10` ✓
- Maximum valid length = 1
- Valid subarrays: `[5]`
- Count: 1

**Step 5: Total Count**
Total = 1 + 2 + 1 + 1 + 1 = **6**

The 6 valid subarrays are: `[2]`, `[1]`, `[2,1]`, `[4]`, `[3]`, `[5]`

## Python Solution

```python
from typing import List
from itertools import accumulate

class Solution:
    def countSubarrays(self, nums: List[int], k: int) -> int:
        # Create prefix sum array with initial 0
        # prefix_sums[i] represents sum of nums[0:i]
        prefix_sums = list(accumulate(nums, initial=0))

        total_count = 0

        # For each ending position i (1-indexed in prefix_sums)
        for end_idx in range(1, len(prefix_sums)):
            # Binary search using first_true_index template
            # Find first length where score >= k (inverted condition)
            left, right = 1, end_idx
            first_true_index = end_idx + 1  # Default: all lengths are valid

            while left <= right:
                mid = (left + right) // 2

                # Calculate sum of subarray with length mid ending at end_idx-1
                subarray_sum = prefix_sums[end_idx] - prefix_sums[end_idx - mid]

                # Inverted condition: check if score is TOO LARGE (>= k)
                if subarray_sum * mid >= k:
                    first_true_index = mid
                    right = mid - 1
                else:
                    left = mid + 1

            # Maximum valid length = first_true_index - 1
            # All subarrays of length 1, 2, ..., (first_true_index - 1) are valid
            total_count += first_true_index - 1

        return total_count
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The algorithm uses a prefix sum array and then iterates through each position `i` from 1 to `n` (where `n` is the length of the array). For each position `i`, it performs a binary search to find the maximum length `l` of a subarray ending at position `i-1` that satisfies the condition `(s[i] - s[i - mid]) * mid < k`.

- The outer loop runs `n` times
- For each iteration, the binary search performs `O(log i)` operations, which in the worst case is `O(log n)`
- Therefore, the overall time complexity is `O(n × log n)`

**Space Complexity:** `O(n)`

The algorithm creates a prefix sum array `s` using `accumulate` with `initial=0`, which has length `n + 1`. This is the only additional data structure that scales with the input size. The other variables (`ans`, `i`, `l`, `r`, `mid`) use constant space.

- The prefix sum array `s` requires `O(n)` space
- All other variables use `O(1)` space
- Therefore, the overall space complexity is `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template**

This problem finds the **maximum** valid length where score < k. The correct approach inverts the condition to use the first_true_index template.

**Wrong approach:**
```python
left, right = 0, end_idx
while left < right:
    mid = (left + right + 1) >> 1
    if subarray_sum * mid < k:  # Valid
        left = mid
    else:
        right = mid - 1
return left
```

**Correct approach (first_true_index template):**
```python
left, right = 1, end_idx
first_true_index = end_idx + 1  # Default: all lengths are valid

while left <= right:
    mid = (left + right) // 2
    if subarray_sum * mid >= k:  # Inverted: find first INVALID length
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

return first_true_index - 1  # Maximum valid length
```

The key insight is to invert the condition: instead of finding where length IS valid, we find the first length where score >= k (invalid), then return `first_true_index - 1`.

### 2. Score Calculation Overflow

**The Pitfall:**
For large arrays with large values, calculating `subarray_sum * length` might cause integer overflow in languages with fixed integer sizes. While Python handles arbitrary precision integers automatically, this is a critical issue in languages like Java or C++.

**Solution:**
- In Python: No action needed due to automatic handling
- In other languages: Use long/int64 types or check for overflow before multiplication
- Alternative approach: Rearrange the inequality to avoid multiplication: instead of `sum * length < k`, check `sum < k / length` (being careful with integer division)

### 3. Off-by-One Error in Prefix Sum Indexing

**The Pitfall:**
Confusing the indexing between the original array and the prefix sum array. The prefix sum array has length `n+1` where `prefix_sums[i]` represents the sum of the first `i` elements of the original array.

**Why it happens:**
- `prefix_sums[0] = 0` (sum of zero elements)
- `prefix_sums[i] - prefix_sums[j]` gives the sum of elements from index `j` to `i-1` in the original array
- It's easy to miscalculate which elements are included in the subarray

**Solution:**
- Always remember that `prefix_sums[end] - prefix_sums[start]` gives the sum of `nums[start:end]`
- The length of this subarray is `end - start`, not `end - start + 1`
- Draw out small examples to verify the indexing

### 4. Incorrect Default for first_true_index

**The Pitfall:**
When no length satisfies score >= k (all lengths are valid), `first_true_index` should default to `end_idx + 1` so that `first_true_index - 1 = end_idx` gives the correct maximum length.

**Wrong:** `first_true_index = -1` or `first_true_index = end_idx`
**Correct:** `first_true_index = end_idx + 1`

This default ensures that when all subarrays are valid, we correctly count all of them.

