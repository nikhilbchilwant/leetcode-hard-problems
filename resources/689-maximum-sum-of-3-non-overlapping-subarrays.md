# 689. Maximum Sum of 3 Non-Overlapping Subarrays

> Source: https://algo.monster/liteproblems/689

## Problem Statement

You are given an integer array `nums` and an integer `k`. Your task is to find three non-overlapping subarrays, each of length `k`, such that their total sum is maximized.

The requirements are:
- Each subarray must have exactly `k` consecutive elements
- The three subarrays cannot overlap with each other
- You need to maximize the sum of all three subarrays combined
- Return the starting indices (0-indexed) of these three subarrays as a list

If there are multiple valid solutions with the same maximum sum, return the lexicographically smallest one. This means if you have two solutions `[a1, a2, a3]` and `[b1, b2, b3]` with the same sum, you should return the one that comes first when comparing element by element from left to right.

For example, if `nums = [1,2,1,2,6,7,5,1]` and `k = 2`, you would need to find three subarrays of length 2 that don't overlap and have the maximum possible sum. The solution would be `[0, 3, 5]` representing subarrays starting at indices 0, 3, and 5.

The solution uses a [sliding window](/problems/sliding_window_maximum) technique combined with dynamic tracking. It maintains three sliding windows of size `k` and tracks:
- `s1`, `s2`, `s3`: Current sums of the three windows
- `mx1`: Maximum sum found for the first window
- `mx12`: Maximum sum found for the first two windows combined
- `idx1`, `idx12`: Starting positions for the optimal first window and first two windows

As the algorithm slides through the array, it updates these values and checks if a better combination of three subarrays is found, ensuring the lexicographically smallest solution is kept when sums are equal.

## Intuition

The key insight is that we can think of this problem as placing three fixed-size windows of length `k` in the array. Since we want to maximize the total sum, we need to find the optimal positions for all three windows.

A brute force approach would try all possible combinations of three non-overlapping windows, which would be inefficient. Instead, we can use a more clever approach by fixing one window and optimizing the others.

The breakthrough comes from realizing that if we fix the position of the rightmost (third) window, we only need to find the best positions for the first two windows to its left. This transforms the problem into a series of smaller subproblems.

As we slide the third window from left to right through the array, we can maintain:
1. The best position for a single window in the region before the second window
2. The best combined positions for two windows in the region before the third window

This way, when we're considering a position for the third window, we already know the optimal positions for the first two windows that could pair with it.

The [sliding window](/problems/sliding_window_maximum) technique naturally fits here because:
- We can maintain running sums for each window position in `O(1)` time by adding the new element entering the window and removing the element leaving it
- As we move right, we gradually expand the available space for the first and second windows, allowing us to update our "best so far" values incrementally

The lexicographically smallest requirement is automatically handled by our left-to-right traversal - when we encounter a new maximum, we update our answer. If we encounter the same maximum later, we keep the earlier (lexicographically smaller) solution.

This approach reduces the time complexity from `O(n³)` for trying all combinations to `O(n)` by cleverly maintaining the best solutions for subproblems as we go.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro), [Prefix Sum](/problems/subarray_sum) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

The solution implements a [sliding window](/problems/sliding_window_maximum) technique that maintains three windows of size `k` simultaneously while tracking optimal positions.

**Variables Setup:**
- `s1`, `s2`, `s3`: Running sums for three potential windows at different positions
- `mx1`: Maximum sum found for a single window in the leftmost region
- `mx12`: Maximum sum found for two windows combined
- `idx1`: Starting index of the best single window
- `idx12`: Tuple containing starting indices of the best two windows
- `s`: Overall maximum sum of three windows
- `ans`: Final answer containing three starting indices

**Algorithm Steps:**

1. **Initialize the windows:** Start iterating from index `i = k * 2` (ensuring space for at least two windows before the current position).

2. **Update window sums:** For each position `i`:
   - Add `nums[i - k * 2]` to `s1`, `nums[i - k]` to `s2`, and `nums[i]` to `s3`
   - Once the windows are complete, `s1` is the sum of the window `[i - k*3 + 1, i - k*2]` (potential first window), `s2` is the sum of `[i - k*2 + 1, i - k]` (potential second window), and `s3` is the sum of `[i - k + 1, i]` (potential third window)

3. **Process when all three windows are complete** (when `i >= k * 3 - 1`):
 
   a. **Update best single window:** If current `s1` is greater than `mx1`, update:
      ```python
      mx1 = s1
      idx1 = i - k * 3 + 1
      ```
 
   b. **Update best two windows:** If `mx1 + s2` exceeds `mx12`, update:
      ```python
      mx12 = mx1 + s2
      idx12 = (idx1, i - k * 2 + 1)
      ```
 
   c. **Update best three windows:** If `mx12 + s3` exceeds current maximum `s`, update:
      ```python
      s = mx12 + s3
      ans = [*idx12, i - k + 1]
      ```
 
   d. **Slide the windows:** Remove the leftmost element from each window sum:
      ```python
      s1 -= nums[i - k * 3 + 1]
      s2 -= nums[i - k * 2 + 1]
      s3 -= nums[i - k + 1]
      ```

**Why this works:**
- At each position, we consider the third window ending at position `i` (starting at `i - k + 1`)
- We already know the best configuration for two windows that could appear before it (stored in `idx12` and `mx12`)
- We also track the best single window that could be the first window (stored in `idx1` and `mx1`)
- The [sliding window](/problems/sliding_window_maximum) technique ensures we compute each sum in `O(1)` time by maintaining running sums

**Lexicographical ordering:** Since we traverse left to right and only update when we find a strictly greater sum, we automatically get the lexicographically smallest solution when there are ties.

**Time Complexity:** `O(n)` where n is the length of the array, as we make a single pass through the array.

**Space Complexity:** `O(1)` as we only use a constant amount of extra space regardless of input size.

## Example Walkthrough

Let's walk through the algorithm with `nums = [1,2,1,2,6,7,5,1]` and `k = 2`. The names below match the explanation: `s1`, `s2`, `s3` are `sum_first`, `sum_second`, `sum_third` in the code; `mx1`, `mx12`, `s` are `max_first`, `max_first_second`, `max_total_sum`; `idx1`, `idx12`, `ans` are `best_first_idx`, `best_first_second_idx`, `result`.

**Initial Setup:**
- The loop starts at `i = k * 2 = 4`
- All sums and maximums start at 0

**Iteration 1: i = 4**
- Add `nums[0] = 1` to `s1`, `nums[2] = 1` to `s2`, `nums[4] = 6` to `s3`
- `s1 = 1`, `s2 = 1`, `s3 = 6`
- `i = 4 < k * 3 - 1 = 5`, so the windows are not complete yet; no comparisons

**Iteration 2: i = 5**
- Add `nums[1] = 2` to `s1`, `nums[3] = 2` to `s2`, `nums[5] = 7` to `s3`
- `s1 = 3` (window [0-1]), `s2 = 3` (window [2-3]), `s3 = 13` (window [4-5])
- Now `i >= 5`, so we compare:
  - `s1 = 3 > mx1 = 0`: update `mx1 = 3`, `idx1 = 0`
  - `mx1 + s2 = 3 + 3 = 6 > mx12 = 0`: update `mx12 = 6`, `idx12 = (0, 2)`
  - `mx12 + s3 = 6 + 13 = 19 > s = 0`: update `s = 19`, `ans = [0, 2, 4]`
- Slide: subtract `nums[0]`, `nums[2]`, `nums[4]` → `s1 = 2`, `s2 = 2`, `s3 = 7`

**Iteration 3: i = 6**
- Add `nums[2] = 1`, `nums[4] = 6`, `nums[6] = 5`
- `s1 = 3` (window [1-2]), `s2 = 8` (window [3-4]), `s3 = 12` (window [5-6])
- Compare:
  - `s1 = 3 > mx1 = 3`? No, keep `mx1 = 3`, `idx1 = 0`
  - `mx1 + s2 = 3 + 8 = 11 > mx12 = 6`: update `mx12 = 11`, `idx12 = (0, 3)`
  - `mx12 + s3 = 11 + 12 = 23 > s = 19`: update `s = 23`, `ans = [0, 3, 5]`
- Slide: subtract `nums[1]`, `nums[3]`, `nums[5]` → `s1 = 1`, `s2 = 6`, `s3 = 5`

**Iteration 4: i = 7**
- Add `nums[3] = 2`, `nums[5] = 7`, `nums[7] = 1`
- `s1 = 3` (window [2-3]), `s2 = 13` (window [4-5]), `s3 = 6` (window [6-7])
- Compare:
  - `s1 = 3 > mx1 = 3`? No, keep `idx1 = 0`
  - `mx1 + s2 = 3 + 13 = 16 > mx12 = 11`: update `mx12 = 16`, `idx12 = (0, 4)`
  - `mx12 + s3 = 16 + 6 = 22 > s = 23`? No, keep `ans = [0, 3, 5]`
- Slide (the loop then ends)

**Final Result:** `[0, 3, 5]`
- Subarray 1: `nums[0:2] = [1,2]`, sum = 3
- Subarray 2: `nums[3:5] = [2,6]`, sum = 8
- Subarray 3: `nums[5:7] = [7,5]`, sum = 12
- Total sum = 23

Note that `[1, 3, 5]` also has sum 23, but the first window at index 1 never replaced index 0 because the check `s1 > mx1` is strict. This keeps the lexicographically smallest answer.

## Python Solution

```python
class Solution:
    def maxSumOfThreeSubarrays(self, nums: List[int], k: int) -> List[int]:
        # Initialize variables to track maximum sums and their indices
        max_total_sum = 0  # Maximum sum of all three subarrays
        sum_first = sum_second = sum_third = 0  # Current sums of three windows
        max_first = max_first_second = 0  # Maximum sums for first window and first+second windows
        best_first_idx = 0  # Best starting index for first subarray
        best_first_second_idx = ()  # Best starting indices for first and second subarrays
        result = []  # Final result containing starting indices of three subarrays
      
        # Iterate through the array starting from position where third window can begin
        for i in range(k * 2, len(nums)):
            # Update sliding window sums by adding new elements
            sum_first += nums[i - k * 2]  # Add element to first window
            sum_second += nums[i - k]     # Add element to second window
            sum_third += nums[i]          # Add element to third window
          
            # Process only when all three windows are fully formed
            if i >= k * 3 - 1:
                # Update maximum sum for first subarray
                if sum_first > max_first:
                    max_first = sum_first
                    best_first_idx = i - k * 3 + 1  # Starting index of first subarray
              
                # Update maximum sum for first + second subarrays
                if max_first + sum_second > max_first_second:
                    max_first_second = max_first + sum_second
                    best_first_second_idx = (best_first_idx, i - k * 2 + 1)
              
                # Update maximum sum for all three subarrays
                if max_first_second + sum_third > max_total_sum:
                    max_total_sum = max_first_second + sum_third
                    result = [*best_first_second_idx, i - k + 1]
              
                # Slide windows by removing leftmost elements
                sum_first -= nums[i - k * 3 + 1]   # Remove from first window
                sum_second -= nums[i - k * 2 + 1]  # Remove from second window
                sum_third -= nums[i - k + 1]       # Remove from third window
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n)`, where `n` is the length of the array `nums`.

The algorithm uses a single loop that iterates through the array from index `k * 2` to `len(nums) - 1`. Within each iteration, all operations are constant time:
- Adding elements to running sums `s1`, `s2`, `s3` (`sum_first`, `sum_second`, `sum_third` in the code): `O(1)`
- Comparing and updating maximum values `mx1`, `mx12`: `O(1)`
- Subtracting elements from running sums: `O(1)`
- Updating indices and answer array: `O(1)`

Since the loop runs approximately `n - 2k` times and each iteration performs `O(1)` operations, the overall time complexity is `O(n)`.

**Space Complexity:** `O(1)`

The algorithm uses only a fixed number of variables regardless of input size:
- Three running sum variables: `s1`, `s2`, `s3`
- Maximum value trackers: `mx1`, `mx12`, `s`
- Index trackers: `idx1`, `idx12`
- Result array: `ans` (which stores exactly 3 indices)

All these variables use constant space that doesn't scale with the input size `n`, resulting in `O(1)` space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Window Initialization and Boundaries**
A common mistake is incorrectly calculating when each window is "complete" and ready for comparison. Developers often confuse the relationship between the loop index `i` and the actual window positions.

**Pitfall Example:**
```python
# WRONG: Processing windows before they're fully formed
for i in range(k * 2, len(nums)):
    sum_first += nums[i - k * 2]
    sum_second += nums[i - k]
    sum_third += nums[i]
  
    # This condition is often written incorrectly
    if i >= k * 2:  # WRONG! Should be k * 3 - 1
        # Process windows...
```

**Solution:**
The third window becomes complete when `i = k * 3 - 1` (after adding `k` elements to each window). Always verify:
- First window: elements from `[i - k*3 + 1]` to `[i - k*2]`
- Second window: elements from `[i - k*2 + 1]` to `[i - k]`
- Third window: elements from `[i - k + 1]` to `[i]`

### 2. **Off-by-One Errors in Index Calculation**
Calculating the starting indices of each window is error-prone, especially when sliding the windows.

**Pitfall Example:**
```python
# WRONG: Incorrect index calculations
best_first_idx = i - k * 3  # Missing +1
second_idx = i - k * 2      # Missing +1
third_idx = i - k            # Missing +1
```

**Solution:**
Use these formulas consistently:
- First window starts at: `i - k * 3 + 1`
- Second window starts at: `i - k * 2 + 1`
- Third window starts at: `i - k + 1`

### 3. **Forgetting to Slide Windows Properly**
Failing to remove the leftmost element from each window sum leads to incorrect cumulative sums.

**Pitfall Example:**
```python
if i >= k * 3 - 1:
    # Process windows...
  
    # WRONG: Forgetting to slide or sliding incorrectly
    sum_first -= nums[i - k * 3]      # Wrong index
    sum_second -= nums[i - k * 2]     # Wrong index
    sum_third -= nums[i - k]           # Wrong index
```

**Solution:**
Always remove the element that's about to leave each window:
```python
sum_first -= nums[i - k * 3 + 1]   # Remove leftmost of first window
sum_second -= nums[i - k * 2 + 1]  # Remove leftmost of second window
sum_third -= nums[i - k + 1]       # Remove leftmost of third window
```

### 4. **Handling Lexicographical Order Incorrectly**
Using `>=` instead of `>` when comparing sums will not guarantee lexicographically smallest result.

**Pitfall Example:**
```python
# WRONG: This doesn't ensure lexicographically smallest solution
if sum_first >= max_first:  # Should use > not >=
    max_first = sum_first
    best_first_idx = i - k * 3 + 1
```

**Solution:**
Always use strict inequality (`>`) when updating maximum values. This ensures we only update when we find a strictly better sum, automatically preserving the leftmost (lexicographically smallest) solution for ties.

### 5. **Not Preserving Previous Best Indices**
When updating `max_first_second`, developers might forget to store the actual indices from when `max_first` was found.

**Pitfall Example:**
```python
# WRONG: Using current position instead of saved best position
if max_first + sum_second > max_first_second:
    max_first_second = max_first + sum_second
    best_first_second_idx = (i - k * 3 + 1, i - k * 2 + 1)  # WRONG!
    # Should use best_first_idx, not recalculate
```

**Solution:**
Always use the saved best index:
```python
best_first_second_idx = (best_first_idx, i - k * 2 + 1)
```
This ensures we're using the position where `max_first` was actually found, not the current position.

