# 220. Contains Duplicate III

> Source: https://algo.monster/liteproblems/220

## Problem Statement

You are given an integer array `nums` and two integer parameters: `indexDiff` and `valueDiff`.

Your task is to determine whether there exists a pair of indices `(i, j)` in the array that satisfies all of the following conditions:

1. The indices must be different: `i != j`
2. The distance between indices must not exceed `indexDiff`: `abs(i - j) <= indexDiff`
3. The difference between the values at these indices must not exceed `valueDiff`: `abs(nums[i] - nums[j]) <= valueDiff`

Return `true` if such a pair exists, otherwise return `false`.

In other words, you need to find two elements in the array that are:
- Close enough in position (within `indexDiff` positions of each other)
- Close enough in value (their difference is at most `valueDiff`)

For example, if `nums = [1, 5, 9, 1, 5, 9]`, `indexDiff = 2`, and `valueDiff = 3`:
- Indices `(0, 3)` won't work because `abs(0 - 3) = 3 > indexDiff`
- Indices `(1, 2)` won't work because `abs(5 - 9) = 4 > valueDiff`
- Indices `(0, 1)` would work because `abs(0 - 1) = 1 <= indexDiff` and `abs(1 - 5) = 4` but this exceeds `valueDiff`, so it doesn't work
- However, indices `(3, 4)` would work because `abs(3 - 4) = 1 <= indexDiff` and `abs(1 - 5) = 4` but we need to check if there's a valid pair

The challenge is to efficiently check all possible pairs within the given constraints.

## Intuition

The naive approach would be to check every pair of indices within `indexDiff` distance, but this could be inefficient with time complexity `O(n * indexDiff)`.

The key insight is that we only care about elements within a sliding window of size `indexDiff + 1`. As we move through the array, we can maintain a window of the most recent `indexDiff + 1` elements. For each new element, we only need to check if any element in our current window satisfies the `valueDiff` condition.

But how do we efficiently check if there's an element in the window that's within `valueDiff` of our current element? If we keep the window elements in a sorted structure, we can use binary search to quickly find elements within the range `[nums[i] - valueDiff, nums[i] + valueDiff]`.

Think of it this way: for a current element `nums[i]`, we want to find if there exists any element `x` in our window such that:
- `nums[i] - valueDiff <= x <= nums[i] + valueDiff`

Using a sorted set (like a balanced BST), we can:
1. Use binary search to find the smallest element that's `>= nums[i] - valueDiff`
2. Check if this element is also `<= nums[i] + valueDiff`
3. If yes, we found a valid pair!

As we slide the window forward:
- Add the current element to the sorted set
- Remove the element that's now outside the window (more than `indexDiff` positions behind)

This approach reduces our time complexity to `O(n log k)` where `k` is the window size (`indexDiff + 1`), because we perform log-time operations (insert, delete, search) for each element in the array.

## Solution Approach

The solution uses a **sliding window technique combined with an ordered set (SortedSet)** to efficiently find pairs that satisfy both the index and value constraints.

Here's the step-by-step implementation:

1. **Initialize a SortedSet**: We create an empty `SortedSet` to maintain elements within our sliding window in sorted order.

2. **Iterate through the array**: For each element `nums[i]` at index `i`:

   a. **Search for a valid pair**: 
      - Use `bisect_left(v - valueDiff)` to find the index of the smallest element in the sorted set that is `>= nums[i] - valueDiff`
      - If such an element exists (i.e., `j < len(s)`), check if it's also `<= nums[i] + valueDiff`
      - If both conditions are met, we've found a valid pair and return `True`

   b. **Add current element**: Insert `nums[i]` into the sorted set

   c. **Maintain window size**: 
      - If we've processed more than `indexDiff` elements (i.e., `i >= indexDiff`), remove the element that's now outside the window
      - The element to remove is `nums[i - indexDiff]`. It is `indexDiff` positions behind `i`, which is still allowed for `i`, but it would be `indexDiff + 1` positions behind the next element, so it must leave before the next search

3. **Return False**: If we complete the iteration without finding a valid pair, return `False`

**Key Implementation Details:**

- The window size is implicitly maintained at most `indexDiff + 1` elements
- `bisect_left` performs binary search in `O(log k)` time where `k` is the window size
- The sorted set ensures we can efficiently find elements within the value range
- By removing elements that fall outside the index window, we ensure we only compare elements that satisfy the `indexDiff` constraint

**Time Complexity**: `O(n log k)` where `n` is the array length and `k = min(n, indexDiff + 1)`
**Space Complexity**: `O(k)` for storing at most `k` elements in the sorted set

## Example Walkthrough

Let's walk through the solution with a concrete example:

**Input:** `nums = [2, 3, 5, 4, 9]`, `indexDiff = 2`, `valueDiff = 1`

We'll maintain a sliding window of size at most `indexDiff + 1 = 3` using a SortedSet.

**Step-by-step execution:**

**i = 0, nums[0] = 2:**
- SortedSet is empty `[]`
- Search for elements in range `[2-1, 2+1] = [1, 3]`
- No elements in set, so no match found
- Add 2 to set: `[2]`
- Window size (1) ≤ indexDiff, no removal needed

**i = 1, nums[1] = 3:**
- SortedSet: `[2]`
- Search for elements in range `[3-1, 3+1] = [2, 4]`
- Binary search finds 2 at index 0
- Check: Is 2 ≤ 4? Yes! **Valid pair found: (0, 1)**
- Return `True`

The algorithm found that indices 0 and 1 form a valid pair because:
- `abs(0 - 1) = 1 ≤ indexDiff = 2` ✓
- `abs(2 - 3) = 1 ≤ valueDiff = 1` ✓

**Alternative scenario to show window maintenance:**

Let's say we had `nums = [1, 5, 9, 1, 5]`, `indexDiff = 2`, `valueDiff = 3`:

**i = 0:** SortedSet: `[]` → add 1 → `[1]` (no removal, since `0 < indexDiff`)
**i = 1:** SortedSet: `[1]`, range [2, 8] has no match → add 5 → `[1, 5]` (no removal)
**i = 2:** SortedSet: `[1, 5]`, range [6, 12] has no match → add 9 → `[1, 5, 9]`
- Since `i = 2 >= indexDiff`, remove `nums[0] = 1` → `[5, 9]`
**i = 3:** SortedSet: `[5, 9]` (holds only indices 1 and 2)
- Search range [1-3, 1+3] = [-2, 4]
- `bisect_left(-2)` returns index 0, and `5 ≤ 4` is false: no match
- The value 1 at index 0 would have matched, but it was removed at the end of the previous step because `abs(0 - 3) = 3 > indexDiff`
- Add 1 → `[1, 5, 9]`, then remove `nums[1] = 5` → `[1, 9]`
**i = 4:** SortedSet: `[1, 9]`, range [2, 8]
- `bisect_left(2)` returns index 1, and `9 ≤ 8` is false: no match
- The value 5 at index 1 would have matched, but `abs(1 - 4) = 3 > indexDiff`, and it has already been removed
- Add 5, remove `nums[2] = 9` → `[1, 5]`
- Loop ends, return `False`

This demonstrates how the sliding window ensures we only compare elements within `indexDiff` distance.

## Python Solution

```python
from typing import List
from sortedcontainers import SortedSet

class Solution:
    def containsNearbyAlmostDuplicate(
        self, nums: List[int], indexDiff: int, valueDiff: int
    ) -> bool:
        """
        Check if there exist two distinct indices i and j such that:
        - abs(i - j) <= indexDiff
        - abs(nums[i] - nums[j]) <= valueDiff
      
        Args:
            nums: List of integers
            indexDiff: Maximum allowed index difference
            valueDiff: Maximum allowed value difference
          
        Returns:
            True if such pair exists, False otherwise
        """
        # Use a sorted set to maintain a sliding window of values
        # The window size is at most indexDiff + 1 elements
        sorted_window = SortedSet()
      
        for current_index, current_value in enumerate(nums):
            # Find the smallest value in the window that is >= (current_value - valueDiff)
            # This is the lower bound of the valid range [v - valueDiff, v + valueDiff]
            lower_bound_index = sorted_window.bisect_left(current_value - valueDiff)
          
            # Check if there exists a value in the valid range
            # If such value exists, it must be at lower_bound_index position
            if (lower_bound_index < len(sorted_window) and 
                sorted_window[lower_bound_index] <= current_value + valueDiff):
                return True
          
            # Add current value to the sliding window
            sorted_window.add(current_value)
          
            # Maintain sliding window size by removing the element that's now too far
            # Remove the element at index (current_index - indexDiff) when window exceeds size
            if current_index >= indexDiff:
                sorted_window.remove(nums[current_index - indexDiff])
      
        return False
```

## Time and Space Complexity

**Time Complexity:** `O(n × log k)`, where `n` is the length of the array `nums` and `k` is the `indexDiff` parameter.

- The outer loop iterates through all `n` elements in the array.
- For each element, we perform:
  - `bisect_left()` operation on the SortedSet: `O(log k)` time, since the set maintains at most `k+1` elements due to the sliding window constraint.
  - Comparison and potential return: `O(1)` time.
  - `add()` operation to insert the current element: `O(log k)` time.
  - `remove()` operation when `i >= indexDiff`: `O(log k)` time to maintain the sliding window of size `k+1`.
- Since we perform `O(log k)` operations for each of the `n` elements, the total time complexity is `O(n × log k)`.

**Space Complexity:** `O(min(n, k))` or more precisely `O(min(n, indexDiff + 1))`

- The SortedSet `s` maintains a sliding window of elements.
- At any point, the set contains at most `indexDiff + 1` elements (elements within the current window).
- In the worst case where `indexDiff >= n-1`, the set could contain all `n` elements.
- Therefore, the space complexity is `O(min(n, k))` where `k` represents `indexDiff`.

## Common Pitfalls

### 1. **Integer Overflow in Value Comparisons**
When dealing with large integers, calculating `current_value - valueDiff` or `current_value + valueDiff` can cause integer overflow in languages with fixed integer sizes. In Python this isn't an issue due to arbitrary precision integers, but in languages like Java or C++, this can lead to incorrect results.

**Example problematic scenario:**
- `current_value = Integer.MAX_VALUE`
- `valueDiff = 10`
- `current_value + valueDiff` would overflow

**Solution:**
Instead of computing `sorted_window[j] <= current_value + valueDiff`, rearrange to avoid overflow:
```python
# Instead of: sorted_window[j] <= current_value + valueDiff
# Use: sorted_window[j] - current_value <= valueDiff
```

### 2. **Off-by-One Error in Window Size Management**
A frequent mistake is incorrectly managing when to remove elements from the sliding window. The condition should be `current_index >= indexDiff`, not `current_index > indexDiff`.

**Why this matters:**
- At index `indexDiff`, we have `indexDiff + 1` elements in our window
- The element at index `0` has already been compared with index `indexDiff` (distance exactly `indexDiff`, which is allowed)
- The next element, at index `indexDiff + 1`, would be `indexDiff + 1` away from it, so it must be removed now. With `>` instead of `>=`, index 0 stays one step too long and a pair at distance `indexDiff + 1` can be reported

**Correct implementation:**
```python
# Remove element when we've processed indexDiff + 1 elements
if current_index >= indexDiff:
    sorted_window.remove(nums[current_index - indexDiff])
```

### 3. **Using Regular Set Instead of SortedSet**
Using a regular Python `set` or `HashSet` won't work because we need to efficiently find elements within a value range. Binary search requires sorted order.

**Wrong approach:**
```python
window = set()  # Regular set doesn't maintain order
# Can't efficiently find elements in range [v - valueDiff, v + valueDiff]
```

**Correct approach:**
```python
from sortedcontainers import SortedSet
sorted_window = SortedSet()  # Maintains sorted order for binary search
```

### 4. **Incorrect Boundary Check After Binary Search**
After finding the lower bound index, forgetting to check if it's within bounds can cause index errors.

**Problematic code:**
```python
lower_bound_index = sorted_window.bisect_left(current_value - valueDiff)
# Missing bounds check!
if sorted_window[lower_bound_index] <= current_value + valueDiff:
    return True
```

**Correct code:**
```python
lower_bound_index = sorted_window.bisect_left(current_value - valueDiff)
if (lower_bound_index < len(sorted_window) and 
    sorted_window[lower_bound_index] <= current_value + valueDiff):
    return True
```

### 5. **Edge Case: indexDiff = 0**
When `indexDiff = 0`, we're looking for duplicate values at the same index, which is impossible since `i != j`. The algorithm handles this correctly, but it's worth noting that it will always return `False` unless there's an implementation error.

**Test case to verify:**
```python
nums = [1, 1, 1]
indexDiff = 0
valueDiff = 0
# Should return False (can't have i != j with abs(i - j) <= 0)
```

