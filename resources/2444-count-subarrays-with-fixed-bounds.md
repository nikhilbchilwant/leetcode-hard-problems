# 2444. Count Subarrays With Fixed Bounds

> Source: https://algo.monster/liteproblems/2444

## Problem Statement

You are given an integer array `nums` and two integers `minK` and `maxK`.

A **fixed-bound subarray** is a contiguous portion of the array that satisfies two specific conditions:
1. The minimum value in the subarray must be exactly equal to `minK`
2. The maximum value in the subarray must be exactly equal to `maxK`

Your task is to count how many such fixed-bound subarrays exist in the given array.

For example, if `nums = [1, 3, 5, 2, 7, 5]`, `minK = 1`, and `maxK = 5`:
- The subarray `[1, 3, 5]` is a fixed-bound subarray because its minimum is 1 (equals `minK`) and maximum is 5 (equals `maxK`)
- The subarray `[1, 3, 5, 2]` is also a fixed-bound subarray with minimum 1 and maximum 5
- The subarray `[3, 5]` is NOT a fixed-bound subarray because its minimum is 3 (not equal to `minK = 1`)

The problem requires you to return the total count of all valid fixed-bound subarrays in the array.

Note that a subarray must be contiguous, meaning it consists of consecutive elements from the original array. Also, for a subarray to be valid, it must contain at least one occurrence of `minK` and at least one occurrence of `maxK`, and all elements must be within the range `[minK, maxK]`.

## Intuition

Instead of checking every possible subarray (which would be inefficient), we can think about this problem differently: for each position in the array, how many valid subarrays end at that position?

The key insight is that for a subarray ending at position `i` to be valid:
1. It must contain at least one occurrence of `minK`
2. It must contain at least one occurrence of `maxK`  
3. It cannot contain any element outside the range `[minK, maxK]`

This leads us to track three critical positions as we traverse the array:
- The most recent position where we saw `minK` (let's call it `j1`)
- The most recent position where we saw `maxK` (let's call it `j2`)
- The most recent position where we saw an invalid element (outside `[minK, maxK]`), let's call it `k`

For any position `i`, valid subarrays ending at `i` must:
- Start after position `k` (to avoid including invalid elements)
- Start at or before `min(j1, j2)` (to ensure both `minK` and `maxK` are included)

Therefore, the number of valid subarrays ending at position `i` is `min(j1, j2) - k` (assuming this is positive; otherwise it's 0).

Why does this work? Because any starting position between `k + 1` and `min(j1, j2)` will create a valid subarray when paired with ending position `i`. The subarray will contain both required values (`minK` and `maxK`) and no invalid elements.

By iterating through the array once and calculating the count for each position, we efficiently find the total number of fixed-bound subarrays in `O(n)` time.

**Learn more about [Queue](/problems/queue_intro), [Sliding Window](/problems/sliding_window_maximum) and [Monotonic Queue](/problems/mono_stack_intro) patterns.**

## Solution Approach

We enumerate each position as the right endpoint of potential subarrays and count how many valid subarrays end at that position.

**Implementation Details:**

1. **Initialize tracking variables:**
   - `j1 = -1`: tracks the most recent index where we found `minK`
   - `j2 = -1`: tracks the most recent index where we found `maxK`
   - `k = -1`: tracks the most recent index where we found an element outside `[minK, maxK]`
   - `ans = 0`: accumulates the total count

2. **Iterate through the array:** For each element `nums[i]`:
 
   a. **Check if element is invalid:** If `v < minK` or `v > maxK`, update `k = i`. This position marks a boundary - no valid subarray can span across this point.
 
   b. **Update minimum position:** If `v == minK`, update `j1 = i` to record the latest occurrence of `minK`.
 
   c. **Update maximum position:** If `v == maxK`, update `j2 = i` to record the latest occurrence of `maxK`.
 
   d. **Count valid subarrays ending at position i:**
      - The leftmost valid starting position is `k + 1` (just after the last invalid element)
      - The rightmost valid starting position is `min(j1, j2)` (to ensure both `minK` and `maxK` are included)
      - The number of valid subarrays ending at `i` is `max(0, min(j1, j2) - k)`
      - Add this count to `ans`

3. **Return the result:** After processing all elements, `ans` contains the total count of fixed-bound subarrays.

**Why `min(j1, j2) - k` works:**
- `min(j1, j2)` ensures we have seen both `minK` and `maxK` before or at this position
- Subtracting `k` gives us the count of valid starting positions
- If `min(j1, j2) <= k`, it means we haven't seen both required values since the last invalid element, so no valid subarrays exist (hence `max(0, ...)`

**Time Complexity:** `O(n)` - single pass through the array  
**Space Complexity:** `O(1)` - only using a few variables to track positions

## Example Walkthrough

Let's walk through the solution with `nums = [1, 3, 5, 2, 7, 5]`, `minK = 1`, `maxK = 5`.

We'll track three positions as we iterate:
- `j1`: last position of `minK` (1)
- `j2`: last position of `maxK` (5)  
- `k`: last position of invalid element (outside [1,5])
- `ans`: running count of valid subarrays

**Initial state:** `j1 = -1`, `j2 = -1`, `k = -1`, `ans = 0`

**i = 0, nums[0] = 1:**
- 1 is within [1,5] ✓
- 1 == minK, so update `j1 = 0`
- min(j1, j2) = min(0, -1) = -1
- Valid subarrays ending here: max(0, -1 - (-1)) = 0
- `ans = 0`

**i = 1, nums[1] = 3:**
- 3 is within [1,5] ✓
- 3 is neither minK nor maxK
- min(j1, j2) = min(0, -1) = -1
- Valid subarrays ending here: max(0, -1 - (-1)) = 0
- `ans = 0`

**i = 2, nums[2] = 5:**
- 5 is within [1,5] ✓
- 5 == maxK, so update `j2 = 2`
- min(j1, j2) = min(0, 2) = 0
- Valid subarrays ending here: max(0, 0 - (-1)) = 1
- This counts: [1,3,5] starting at index 0
- `ans = 1`

**i = 3, nums[3] = 2:**
- 2 is within [1,5] ✓
- 2 is neither minK nor maxK
- min(j1, j2) = min(0, 2) = 0
- Valid subarrays ending here: max(0, 0 - (-1)) = 1
- This counts: [1,3,5,2] starting at index 0
- `ans = 2`

**i = 4, nums[4] = 7:**
- 7 > maxK, invalid! Update `k = 4`
- min(j1, j2) = min(0, 2) = 0
- Valid subarrays ending here: max(0, 0 - 4) = 0
- No valid subarrays (7 is outside range)
- `ans = 2`

**i = 5, nums[5] = 5:**
- 5 is within [1,5] ✓
- 5 == maxK, so update `j2 = 5`
- min(j1, j2) = min(0, 5) = 0
- Valid subarrays ending here: max(0, 0 - 4) = 0
- No valid subarrays (would need to include index 0 for minK, but index 4 has invalid element 7 in between)
- `ans = 2`

**Final answer: 2**

The two valid fixed-bound subarrays are:
1. [1, 3, 5] (indices 0-2)
2. [1, 3, 5, 2] (indices 0-3)

## Python Solution

```python
class Solution:
    def countSubarrays(self, nums: List[int], minK: int, maxK: int) -> int:
        # Track the most recent positions of key elements
        last_min_pos = -1  # Last position where we found minK
        last_max_pos = -1  # Last position where we found maxK
        last_invalid_pos = -1  # Last position where we found an invalid element (outside [minK, maxK])
      
        total_count = 0
      
        # Iterate through each element in the array
        for current_pos, value in enumerate(nums):
            # Check if current element is outside the valid range [minK, maxK]
            if value < minK or value > maxK:
                last_invalid_pos = current_pos
          
            # Update position if we found minK
            if value == minK:
                last_min_pos = current_pos
          
            # Update position if we found maxK
            if value == maxK:
                last_max_pos = current_pos
          
            # Calculate valid subarrays ending at current position
            # We need both minK and maxK to appear after the last invalid element
            # The number of valid subarrays is determined by the earlier occurrence of minK or maxK
            leftmost_valid_start = min(last_min_pos, last_max_pos)
            valid_subarray_count = max(0, leftmost_valid_start - last_invalid_pos)
          
            total_count += valid_subarray_count
      
        return total_count
```

## Time and Space Complexity

The time complexity is `O(n)`, where `n` is the length of the array `nums`. The algorithm iterates through the array exactly once using a single for loop with `enumerate(nums)`. Each operation inside the loop (comparisons, assignments, and arithmetic operations) takes constant time `O(1)`. Therefore, the overall time complexity is linear.

The space complexity is `O(1)`. The algorithm uses only a fixed number of variables (`j1`, `j2`, `k`, `ans`, `i`, and `v`) regardless of the input size. No additional data structures that grow with the input are created, resulting in constant space usage.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding the Valid Range Check**

**Pitfall:** A common mistake is incorrectly handling elements that equal `minK` or `maxK` when checking for invalid elements. Some might write:
```python
# WRONG: This excludes minK and maxK themselves
if value <= minK or value >= maxK:
    last_invalid_pos = current_pos
```

**Why it's wrong:** Elements equal to `minK` and `maxK` are valid and should NOT update `last_invalid_pos`. Only elements strictly outside the range `[minK, maxK]` are invalid.

**Solution:** Use strict inequality:
```python
# CORRECT: Only mark as invalid if strictly outside [minK, maxK]
if value < minK or value > maxK:
    last_invalid_pos = current_pos
```

### 2. **Forgetting to Handle the Case When minK equals maxK**

**Pitfall:** The code might fail or give incorrect results when `minK == maxK`, which means we're looking for subarrays where all elements are the same value.

**Why it matters:** When `minK == maxK`, both `last_min_pos` and `last_max_pos` get updated to the same value whenever we encounter this element, but the logic still works correctly.

**Solution:** The provided code actually handles this case correctly without modification because:
- When `value == minK == maxK`, both position trackers update to the same index
- The calculation `min(last_min_pos, last_max_pos)` still gives the correct position
- The counting logic remains valid

### 3. **Off-by-One Error in Counting Valid Subarrays**

**Pitfall:** Confusing the number of valid starting positions with indices:
```python
# WRONG: This might seem intuitive but is incorrect
valid_subarray_count = leftmost_valid_start - last_invalid_pos - 1
```

**Why it's wrong:** The number of valid starting positions from `last_invalid_pos + 1` to `leftmost_valid_start` (inclusive) is exactly `leftmost_valid_start - last_invalid_pos`.

**Solution:** Remember that when counting positions from index `a+1` to index `b` (inclusive), the count is `b - a`:
```python
# CORRECT: Direct subtraction gives the count
valid_subarray_count = max(0, leftmost_valid_start - last_invalid_pos)
```

### 4. **Not Initializing Position Trackers to -1**

**Pitfall:** Initializing position trackers to 0:
```python
# WRONG: Starting at 0 can cause incorrect counts
last_min_pos = 0
last_max_pos = 0
last_invalid_pos = 0
```

**Why it's wrong:** If we haven't seen a required element yet, using 0 as the position would incorrectly suggest we've seen it at index 0, leading to wrong subarray counts.

**Solution:** Initialize all position trackers to -1 to indicate "not yet seen":
```python
# CORRECT: -1 indicates we haven't encountered the element yet
last_min_pos = -1
last_max_pos = -1
last_invalid_pos = -1
```
This ensures that `min(last_min_pos, last_max_pos) - last_invalid_pos` correctly evaluates to a negative number (resulting in 0 after `max(0, ...)`) when we haven't seen both required elements yet.

