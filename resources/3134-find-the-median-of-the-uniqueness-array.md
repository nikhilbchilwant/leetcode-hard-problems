# 3134. Find the Median of the Uniqueness Array

> Source: https://algo.monster/liteproblems/3134

## Problem Statement

You are given an integer array `nums`. Your task is to find the median of a special array called the "uniqueness array".

The **uniqueness array** is constructed as follows:
- For every possible subarray of `nums` (including single elements and the entire array), count the number of distinct elements in that subarray
- Collect all these counts into an array
- Sort this array in non-decreasing order

For example, if `nums = [1, 2, 1]`:
- Subarray `[1]` has 1 distinct element
- Subarray `[2]` has 1 distinct element
- Subarray `[1]` has 1 distinct element
- Subarray `[1, 2]` has 2 distinct elements
- Subarray `[2, 1]` has 2 distinct elements
- Subarray `[1, 2, 1]` has 2 distinct elements
- The uniqueness array would be `[1, 1, 1, 2, 2, 2]` after sorting

The **median** of the uniqueness array is:
- The middle element if the array has an odd number of elements
- The smaller of the two middle elements if the array has an even number of elements

Since there are `n * (n + 1) / 2` total subarrays for an array of length `n`, the uniqueness array will have exactly this many elements.

Your goal is to return the median value from this uniqueness array without actually constructing the entire array (which would be inefficient for large inputs).

## Intuition

The key insight is that we don't need to generate the entire uniqueness array to find its median. Instead, we can use [binary search](/problems/binary_search_intro) to directly find the median value.

Think about it this way: if we pick any value `x`, we can count how many elements in the uniqueness array are less than or equal to `x`. For the median, we need the value where exactly `(m + 1) / 2` elements are less than or equal to it, where `m = n * (n + 1) / 2` is the total number of subarrays.

The crucial observation is that as `x` increases, the count of elements ≤ `x` also increases (monotonic property). This makes [binary search](/problems/binary_search_intro) perfect for finding the smallest `x` where at least `(m + 1) / 2` elements are ≤ `x`.

But how do we count subarrays with at most `x` distinct elements efficiently? Here's where the [sliding window](/problems/sliding_window_maximum) technique comes in. We can maintain a window that expands to the right and contracts from the left to ensure it never has more than `x` distinct elements.

For each position `r` as the right endpoint:
- We expand the window to include `nums[r]`
- If the window has more than `x` distinct elements, we shrink from the left until it's valid again
- Once valid, all subarrays ending at `r` with starting positions from `l` to `r` are valid (that's `r - l + 1` subarrays)

This two-pointer approach lets us count all valid subarrays in `O(n)` time for a given `x`. Combined with [binary search](/problems/binary_search_intro) over possible values of `x` (which ranges from 1 to `n`), we get an efficient solution without explicitly constructing the massive uniqueness array.

The beauty of this approach is that we transform the problem from "find the median of a huge array" to "find the first value where enough subarrays satisfy a condition" - a much more manageable task.

**Learn more about [Binary Search](/problems/binary_search_intro) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

The solution uses **[binary search](/problems/binary_search_intro)** combined with a **[sliding window](/problems/sliding_window_maximum) technique** to efficiently find the median without constructing the entire uniqueness array.

### Binary Search Template

We use the standard binary search template to find the smallest uniqueness value where at least half of subarrays have that many or fewer distinct elements:

```python
left, right = 1, n
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

return first_true_index
```

The `feasible(mid)` function returns `True` if at least `(total_subarrays + 1) / 2` subarrays have at most `mid` distinct elements.

### The Feasible Function

The feasible function creates a boolean pattern over the search space:
```
false, false, false, ..., true, true, true, ...
```
We find the first `true` position, which is exactly the median uniqueness value.

### Counting Subarrays with Sliding Window

For a given `max_unique`, we count subarrays with at most `max_unique` distinct elements using a sliding window:

1. **Expand the window:** For each position `right`, add `nums[right]` to the frequency map

2. **Shrink if needed:** While the window has more than `max_unique` distinct elements:
   - Remove elements from the left
   - If count becomes 0, delete from the map
   - Move left pointer forward

3. **Count valid subarrays:** All subarrays ending at `right` with start positions from `left` to `right` are valid, contributing `right - left + 1` subarrays

### Time Complexity

- Binary search: `O(log n)` iterations
- Each feasible check: `O(n)` time (sliding window traverses array once)
- Overall: `O(n log n)`

### Space Complexity

- `O(n)` for the frequency map in the worst case (all elements distinct)

## Example Walkthrough

Let's walk through the solution with `nums = [1, 2, 1]`.

**Step 1: Setup**
- Array length `n = 3`
- Total subarrays: `m = 3 * 4 / 2 = 6`
- Median position: `(6 + 1) / 2 = 3.5`, so we need the 4th smallest element (ceiling of 3.5)
- Binary search range: `l = 0`, `r = 3`

**Step 2: Binary Search Iterations**

**First iteration:** `mid = (0 + 3) / 2 = 1`
- Check if at least 4 subarrays have ≤ 1 distinct element
- Using sliding window with `mx = 1`:
  - Window `[1]`: 1 distinct element, valid → count = 1
  - Window `[2]`: 1 distinct element, valid → count = 2
  - Window `[1]`: 1 distinct element, valid → count = 3
  - Total: 3 subarrays with ≤ 1 distinct element
- Since 3 < 4, `check(1) = False`
- Update: `l = 2` (search in upper half)

**Second iteration:** `mid = (2 + 3) / 2 = 2`
- Check if at least 4 subarrays have ≤ 2 distinct elements
- Using sliding window with `mx = 2`:
  - `r = 0`: Window `[1]` → 1 distinct, add 1 subarray → count = 1
  - `r = 1`: Window `[1,2]` → 2 distinct, add 2 subarrays (`[2]` and `[1,2]`) → count = 3
  - `r = 2`: Window `[1,2,1]` → 2 distinct, add 3 subarrays (`[1]`, `[2,1]`, `[1,2,1]`) → count = 6
  - Total: 6 subarrays with ≤ 2 distinct elements
- Since 6 ≥ 4, `check(2) = True`
- Update: `r = 2` (search in lower half to find smallest valid value)

**Third iteration:** Binary search converges
- We've found that 2 is the smallest value where at least 4 subarrays have that many or fewer distinct elements
- This means the median is 2

**Verification:**
The actual uniqueness array (sorted) would be:
- `[1]` → 1 distinct
- `[2]` → 1 distinct
- `[1]` → 1 distinct
- `[1,2]` → 2 distinct
- `[2,1]` → 2 distinct
- `[1,2,1]` → 2 distinct

Sorted: `[1, 1, 1, 2, 2, 2]`
The 4th element (median for even-length array) is indeed 2.

The algorithm correctly found the median without constructing the entire array!

## Python Solution

```python
from typing import List
from collections import defaultdict

class Solution:
    def medianOfUniquenessArray(self, nums: List[int]) -> int:
        n = len(nums)
        total_subarrays = n * (n + 1) // 2

        def count_subarrays_with_at_most_k_distinct(max_unique: int) -> int:
            """Count subarrays with at most max_unique distinct elements."""
            element_count = defaultdict(int)
            subarrays_count = 0
            left = 0

            for right, current_num in enumerate(nums):
                element_count[current_num] += 1

                while len(element_count) > max_unique:
                    left_num = nums[left]
                    element_count[left_num] -= 1
                    if element_count[left_num] == 0:
                        del element_count[left_num]
                    left += 1

                subarrays_count += right - left + 1

            return subarrays_count

        def feasible(mid: int) -> bool:
            """Check if at least half of subarrays have uniqueness <= mid."""
            return count_subarrays_with_at_most_k_distinct(mid) >= (total_subarrays + 1) // 2

        # Binary search using the standard template
        left, right = 1, n
        first_true_index = -1

        while left <= right:
            mid = (left + right) // 2
            if feasible(mid):
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1

        return first_true_index
```

## Time and Space Complexity

The time complexity is `O(n × log n)`, where `n` is the length of the array `nums`.

**Time Complexity Analysis:**
- The outer binary search using `bisect_left` searches over the range `[0, n)`, which takes `O(log n)` iterations.
- For each binary search iteration, the `check` function is called.
- Inside the `check` function:
  - There's a sliding window approach with two pointers `l` and `r` that traverse the array once.
  - Each element is visited at most twice (once when `r` moves right, once when `l` moves right).
  - The operations inside the loop (dictionary updates, comparisons) are `O(1)` on average.
  - Therefore, each call to `check` takes `O(n)` time.
- Total time complexity: `O(log n) × O(n) = O(n × log n)`.

The space complexity is `O(n)`.

**Space Complexity Analysis:**
- The `cnt` dictionary (defaultdict) stores the frequency of distinct elements in the current window.
- In the worst case, all elements in the array are distinct, so the dictionary can contain up to `n` entries.
- The binary search and other variables use `O(1)` additional space.
- Total space complexity: `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Using Wrong Binary Search Template Variant

**Pitfall:** Using the `while left < right` with `right = mid` pattern instead of the standard template:

```python
# WRONG - Non-standard template variant
while left < right:
    mid = (left + right) // 2
    if feasible(mid):
        right = mid
    else:
        left = mid + 1
return left
```

**Correct approach using the standard template:**
```python
left, right = 1, n
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

return first_true_index
```

The standard template explicitly tracks the answer with `first_true_index`, uses `while left <= right`, and always moves bounds by exactly 1 (`right = mid - 1`).

### 2. Incorrect Median Position Calculation

**Pitfall:** Using `total_subarrays // 2` instead of `(total_subarrays + 1) // 2` for the median position. This leads to finding the wrong element in the sorted uniqueness array.

**Why it happens:** For even-length arrays, we need the smaller of the two middle elements. The position `(total_subarrays + 1) // 2` correctly gives us this position.

**Solution:** Always use `(total_subarrays + 1) // 2` for the median position check.

### 3. Memory Leak with defaultdict

**Pitfall:** Not removing elements from the dictionary when their count reaches 0, causing incorrect distinct element counting.

```python
# Wrong approach
while len(element_count) > max_unique:
    element_count[nums[left]] -= 1
    # Missing: del element_count[nums[left]] when count is 0
    left += 1
```

**Solution:** Always remove dictionary entries when their count reaches 0:
```python
while len(element_count) > max_unique:
    element_count[nums[left]] -= 1
    if element_count[nums[left]] == 0:
        del element_count[nums[left]]
    left += 1
```

### 4. Misunderstanding the Binary Search Target

**Pitfall:** Thinking we're searching for the exact median value rather than the minimum number of distinct elements that satisfies our condition.

**Solution:** Remember that we find the first (smallest) value where the condition becomes True, which correctly gives us the median of the sorted uniqueness array.

