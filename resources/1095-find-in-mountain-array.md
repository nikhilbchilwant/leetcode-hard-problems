# 1095. Find in Mountain Array

> Source: https://algo.monster/liteproblems/1095

## Problem Statement

This is an interactive problem where you need to find a target value in a mountain array with limited access.

A **mountain array** is defined as an array that:
- Has at least 3 elements
- Has a peak element at some index `i` (where `0 < i < arr.length - 1`)
- Strictly increases from the start to the peak: `arr[0] < arr[1] < ... < arr[i]`
- Strictly decreases from the peak to the end: `arr[i] > arr[i + 1] > ... > arr[arr.length - 1]`

You're given a `MountainArray` object and a target value. Your task is to find and return the **minimum index** where `mountainArr.get(index) == target`. If the target doesn't exist in the array, return `-1`.

The catch is that you cannot directly access the array. You can only use two methods:
- `MountainArray.get(k)`: Returns the element at index `k` (0-indexed)
- `MountainArray.length()`: Returns the length of the array

There's a constraint: your solution must make **no more than 100 calls** to `MountainArray.get()`, otherwise it will be judged as Wrong Answer.

The solution uses a two-phase approach:
1. **Find the peak**: Uses [binary search](/problems/binary_search_intro) to locate the peak of the mountain array by comparing adjacent elements. If `get(mid) > get(mid + 1)`, the peak is at or before `mid`; otherwise, it's after `mid`.

2. **Search for target**: Once the peak is found, the array is divided into two parts:
   - The ascending part (from start to peak): Regular [binary search](/problems/binary_search_intro)
   - The descending part (from peak+1 to end): Binary search with reversed comparison logic

The clever use of the parameter `k` (1 or -1) in the `search` function allows the same [binary search](/problems/binary_search_intro) logic to work for both ascending and descending portions by multiplying the comparison values by `k`.

## Intuition

The key insight is recognizing that a mountain array has special properties we can exploit. Since we have a strict limit on the number of calls (100), we need an efficient approach - this immediately suggests [binary search](/problems/binary_search_intro), which runs in `O(log n)` time.

The mountain array essentially consists of two sorted sequences joined at a peak: one ascending and one descending. If we can find the peak, we can treat each side as a sorted array and apply [binary search](/problems/binary_search_intro).

Why find the peak first? Without knowing where the peak is, we can't determine which "side" of the mountain we're on when we find an element. The comparison logic differs between the ascending side (where larger indices mean larger values) and the descending side (where larger indices mean smaller values).

To find the peak efficiently, we observe that at any point in the array:
- If `arr[mid] > arr[mid + 1]`, we're on the descending side or at the peak itself, so the peak must be at position `mid` or earlier
- If `arr[mid] < arr[mid + 1]`, we're on the ascending side, so the peak must be after position `mid`

This allows us to [binary search](/problems/binary_search_intro) for the peak in `O(log n)` calls.

Once we have the peak, searching for the target becomes straightforward - we perform [binary search](/problems/binary_search_intro) on the ascending portion first (since we want the minimum index). If not found there, we search the descending portion.

The elegant trick in the code is using a multiplier `k` (1 for ascending, -1 for descending) to unify the [binary search](/problems/binary_search_intro) logic. By multiplying both the current element and target by `k`, we convert the descending search into an ascending one: in the descending portion, smaller values are actually "greater" in terms of their position from the peak, so multiplying by -1 reverses the comparison appropriately.

This approach ensures we use at most `3 * log n` calls to `get()`: one [binary search](/problems/binary_search_intro) to find the peak, and potentially two more to search both sides, well within the 100-call limit.

**Learn more about [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The implementation uses the binary search template three times: once to find the peak, once to search the ascending portion, and once to search the descending portion.

**Step 1: Find the Peak Using Template**

**Feasible condition:** `get(mid) > get(mid + 1)` - Are we on the descending side or at the peak?

```python
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if mid < n - 1 and mountain_arr.get(mid) > mountain_arr.get(mid + 1):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
peak_index = first_true_index
```

**Step 2: Search Ascending Portion Using Template**

**Feasible condition:** `get(mid) >= target`

Pattern: `[false, false, ..., true, true, true]` (values increase left to right)

```python
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if mountain_arr.get(mid) >= target:
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
# Verify found value equals target
if first_true_index != -1 and mountain_arr.get(first_true_index) == target:
    return first_true_index
```

**Step 3: Search Descending Portion Using Template**

**Feasible condition:** `get(mid) <= target`

Pattern: `[false, false, ..., true, true, true]` (values decrease left to right, so smaller values are on the right)

```python
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if mountain_arr.get(mid) <= target:
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
# Verify found value equals target
```

**Why Different Feasible Conditions?**
- Ascending: Values increase, so first position where `value >= target` gives us the target (or proves it doesn't exist)
- Descending: Values decrease, so first position where `value <= target` gives us the target (or proves it doesn't exist)

**Time Complexity**: `O(log n)` - Three binary searches, each taking `O(log n)` time.

**Space Complexity**: `O(1)` - Only using constant extra space.

**API Calls**: Maximum `3 * log n` calls to `mountain_arr.get()`, well within the 100-call limit.

## Example Walkthrough

Let's walk through the solution with a concrete example:

**Mountain Array**: `[1, 3, 5, 7, 6, 4, 2]`
**Target**: `4`
**Expected Output**: `5` (index where 4 is located)

**Step 1: Find the Peak**

We start with `l = 0`, `r = 6` (array length - 1).

*Iteration 1:*
- `mid = (0 + 6) >> 1 = 3`
- Compare `get(3) = 7` with `get(4) = 6`
- Since `7 > 6`, we're on the descending side or at peak
- Update `r = 3`

*Iteration 2:*
- `l = 0`, `r = 3`
- `mid = (0 + 3) >> 1 = 1`
- Compare `get(1) = 3` with `get(2) = 5`
- Since `3 < 5`, we're on the ascending side
- Update `l = 2`

*Iteration 3:*
- `l = 2`, `r = 3`
- `mid = (2 + 3) >> 1 = 2`
- Compare `get(2) = 5` with `get(3) = 7`
- Since `5 < 7`, we're on the ascending side
- Update `l = 3`

Now `l = r = 3`, so the peak is at index 3 with value 7.

**Step 2: Search for Target = 4**

*Search ascending portion [0, 3] with k = 1:*
- `l = 0`, `r = 3`
- `mid = 1`: `1 * get(1) = 3` vs `1 * 4 = 4`
  - Since `3 < 4`, update `l = 2`
- `l = 2`, `r = 3`
- `mid = 2`: `1 * get(2) = 5` vs `1 * 4 = 4`
  - Since `5 >= 4`, update `r = 2`
- Now `l = r = 2`, check `get(2) = 5 ≠ 4`
- Return `-1` (not found in ascending portion)

*Search descending portion [4, 6] with k = -1:*
- `l = 4`, `r = 6`
- `mid = 5`: `-1 * get(5) = -4` vs `-1 * 4 = -4`
  - Since `-4 >= -4`, update `r = 5`
- Now `l = 4`, `r = 5`
- `mid = 4`: `-1 * get(4) = -6` vs `-1 * 4 = -4`
  - Since `-6 < -4`, update `l = 5`
- Now `l = r = 5`, check `get(5) = 4 = 4` ✓
- Return `5`

**Final Answer**: `5`

The solution efficiently found the target using only 10 calls to `get()`: 3 to find the peak, 3 for the ascending search, and 4 for the descending search.

## Python Solution

```python
# """
# This is MountainArray's API interface.
# You should not implement it, or speculate about its implementation
# """
# class MountainArray:
#    def get(self, index: int) -> int:
#    def length(self) -> int:


class Solution:
    def findInMountainArray(self, target: int, mountain_arr: 'MountainArray') -> int:
        def search_ascending(left: int, right: int) -> int:
            """
            Binary search in ascending portion using template.
            Feasible: mountain_arr.get(mid) >= target
            """
            first_true_index = -1
            while left <= right:
                mid = (left + right) // 2
                if mountain_arr.get(mid) >= target:
                    first_true_index = mid
                    right = mid - 1
                else:
                    left = mid + 1
            # Check if found value equals target
            if first_true_index != -1 and mountain_arr.get(first_true_index) == target:
                return first_true_index
            return -1

        def search_descending(left: int, right: int) -> int:
            """
            Binary search in descending portion using template.
            Feasible: mountain_arr.get(mid) <= target (values decrease, so <= is the boundary)
            """
            first_true_index = -1
            while left <= right:
                mid = (left + right) // 2
                if mountain_arr.get(mid) <= target:
                    first_true_index = mid
                    right = mid - 1
                else:
                    left = mid + 1
            # Check if found value equals target
            if first_true_index != -1 and mountain_arr.get(first_true_index) == target:
                return first_true_index
            return -1

        # Get the length of the mountain array
        array_length = mountain_arr.length()

        # Step 1: Find the peak element using binary search template
        # Feasible: mountain_arr.get(mid) > mountain_arr.get(mid + 1) (on descending side)
        left, right = 0, array_length - 1
        first_true_index = -1
        while left <= right:
            mid = (left + right) // 2
            if mid < array_length - 1 and mountain_arr.get(mid) > mountain_arr.get(mid + 1):
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1
        peak_index = first_true_index

        # Step 2: Search in ascending part (from start to peak)
        result = search_ascending(0, peak_index)

        # Step 3: If not found, search in descending part (from peak+1 to end)
        if result == -1:
            result = search_descending(peak_index + 1, array_length - 1)

        return result
```

## Time and Space Complexity

**Time Complexity:** `O(log n)`

The algorithm consists of three binary search operations:
1. First binary search to find the peak element: `O(log n)` where each iteration makes 2 calls to `mountain_arr.get()`
2. Second binary search on the ascending part (left side): `O(log n)` where each iteration makes 1 call to `mountain_arr.get()`
3. Third binary search on the descending part (right side) - only executed if target not found in ascending part: `O(log n)` where each iteration makes 1 call to `mountain_arr.get()`

Since these searches are performed sequentially (not nested), the total time complexity is `O(log n) + O(log n) + O(log n) = O(log n)`.

The total number of API calls to `mountain_arr.get()` is at most `O(log n)` calls.

**Space Complexity:** `O(1)`

The algorithm uses only a constant amount of extra space:
- A few integer variables (`l`, `r`, `mid`, `n`, `ans`)
- The `search` helper function which doesn't use recursion
- The parameter `k` in the search function

No additional data structures are created that scale with input size, making the space complexity constant.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using the Wrong Binary Search Template**

A common mistake is using `while left < right` with `right = mid` instead of the standard template:

```python
# Alternative template (not recommended)
while left < right:
    mid = (left + right) // 2
    if mountain_arr.get(mid) > mountain_arr.get(mid + 1):
        right = mid
    else:
        left = mid + 1
```

**Solution:** Use the standard template with `while left <= right` and `first_true_index`:

```python
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if mid < n - 1 and mountain_arr.get(mid) > mountain_arr.get(mid + 1):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
```

### 2. **Incorrect Feasible Condition for Descending Search**

Using `>= target` for descending arrays (which have values in decreasing order):

```python
# WRONG for descending arrays
if mountain_arr.get(mid) >= target:
    first_true_index = mid
    right = mid - 1
```

**Why it's wrong:** In a descending array, values decrease from left to right. The boundary where values become <= target is what we're looking for.

**Solution:** Use `<= target` for descending arrays:

```python
# CORRECT for descending arrays
if mountain_arr.get(mid) <= target:
    first_true_index = mid
    right = mid - 1
```

### 3. **Incorrect Binary Search Boundaries for Descending Portion**

Including the peak in both ascending and descending searches:

```python
# WRONG - Including peak twice
result = search_ascending(0, peak_index)
if result == -1:
    result = search_descending(peak_index, array_length - 1)  # Should be peak_index + 1
```

**Solution:** Start descending search from `peak_index + 1`:

```python
result = search_ascending(0, peak_index)
if result == -1:
    result = search_descending(peak_index + 1, array_length - 1)
```

### 4. **Not Validating Found Value Equals Target**

Returning `first_true_index` without checking if the value actually equals target:

```python
# WRONG
return first_true_index
```

**Why it's wrong:** The feasible condition (`>= target` or `<= target`) doesn't guarantee equality. The value at `first_true_index` might be greater or less than target.

**Solution:** Always verify:

```python
if first_true_index != -1 and mountain_arr.get(first_true_index) == target:
    return first_true_index
return -1
```

