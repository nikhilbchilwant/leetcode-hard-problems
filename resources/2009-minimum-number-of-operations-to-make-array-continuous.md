# 2009. Minimum Number of Operations to Make Array Continuous

> Source: https://algo.monster/liteproblems/2009

## Problem Statement

You have an integer array `nums`. You can perform operations where each operation allows you to replace any element in the array with any integer value.

The goal is to make the array **continuous**. An array is considered continuous when it satisfies both conditions:
1. All elements in the array are **unique** (no duplicates)
2. The difference between the maximum and minimum elements equals `nums.length - 1`

In other words, a continuous array of length `n` contains exactly `n` consecutive integers (though not necessarily in sorted order).

For example:
- `nums = [4, 2, 5, 3]` is continuous because:
  - All elements are unique
  - Max (5) - Min (2) = 3, which equals array length (4) - 1
  - This represents consecutive integers: 2, 3, 4, 5

- `nums = [1, 2, 3, 5, 6]` is not continuous because:
  - Max (6) - Min (1) = 5, but array length - 1 = 4
  - The array is missing 4 to be truly consecutive

Your task is to find the **minimum number of operations** needed to transform the given array into a continuous array. Each operation replaces one element with any integer of your choice.

## Intuition

The key insight is that we want to keep as many original elements as possible and only replace the minimum number of elements. Since the final continuous array must have exactly `n` consecutive integers, we need to find which consecutive range would allow us to preserve the most original elements.

Think of it this way: if we decide the final continuous array should be `[x, x+1, x+2, ..., x+n-1]` for some starting value `x`, then we want to choose `x` such that this range contains as many original elements from `nums` as possible.

Here's the crucial observation: instead of trying all possible starting values `x`, we can be smarter. The optimal range must align with at least one existing element in the array. Why? Because if we shift the range slightly to include one more existing element, we reduce the number of operations needed.

So our strategy becomes:
1. Remove duplicates first (since the final array needs unique elements, duplicates must be replaced anyway)
2. Sort the array to make range checking easier
3. For each unique element, consider it as the potential minimum value of our final continuous array
4. If element `nums[i]` is the minimum, then the maximum would be `nums[i] + n - 1`
5. Count how many existing unique elements fall within the range `[nums[i], nums[i] + n - 1]`
6. The elements outside this range need to be replaced

The number of operations needed equals `n` minus the number of elements we can keep. By trying each unique element as the potential minimum and finding which gives us the maximum elements to keep, we minimize the operations needed.

[Binary search](/problems/binary_search_intro) helps us efficiently find how many elements fall within each range since the array is sorted.

**Learn more about [Binary Search](/problems/binary_search_intro) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

The implementation follows a systematic approach using sorting, deduplication, and [binary search](/problems/binary_search_intro):

**Step 1: Initialize and Deduplicate**
```python
ans = n = len(nums)
nums = sorted(set(nums))
```
- Store the original length `n` of the array
- Remove duplicates using `set()` and sort the resulting unique elements
- Initialize `ans` to `n` (worst case: replace all elements)

**Step 2: Enumerate Each Element as Potential Minimum**
```python
for i, v in enumerate(nums):
```
- Iterate through each unique element `v` at index `i`
- Consider `v` as the minimum value of the potential continuous array

**Step 3: Find the Range Upper Bound**
```python
j = bisect_right(nums, v + n - 1)
```
- If `v` is the minimum, the maximum would be `v + n - 1`
- Use `bisect_right` to find the first index `j` where `nums[j] > v + n - 1`
- This gives us the range `[i, j)` containing all elements that fall within `[v, v + n - 1]`

**Step 4: Calculate Operations Needed**
```python
ans = min(ans, n - (j - i))
```
- `j - i` represents the count of existing unique elements within the range `[v, v + n - 1]`
- `n - (j - i)` is the number of elements that need to be replaced
- Update `ans` with the minimum operations found so far

**Step 5: Return Result**
```python
return ans
```

**Time Complexity:** `O(n log n)` due to sorting and [binary search](/problems/binary_search_intro) operations for each element

**Space Complexity:** `O(n)` for storing the deduplicated sorted array

The algorithm efficiently finds the optimal consecutive range by trying each existing unique element as a potential minimum value and determining which choice requires the fewest replacements.

## Example Walkthrough

Let's walk through the solution with `nums = [1, 10, 100, 10, 1]`.

**Step 1: Initialize and Deduplicate**
- Original array: `[1, 10, 100, 10, 1]` with `n = 5`
- After removing duplicates and sorting: `nums = [1, 10, 100]`
- Initialize `ans = 5` (worst case: replace all elements)

**Step 2: Try Each Element as Potential Minimum**

*Iteration 1: Consider `v = 1` as minimum*
- Range would be `[1, 5]` (since `1 + 5 - 1 = 5`)
- Use binary search: `j = bisect_right([1, 10, 100], 5) = 1`
- Elements in range `[1, 5]`: only `nums[0] = 1`
- Count of elements to keep: `j - i = 1 - 0 = 1`
- Operations needed: `5 - 1 = 4`
- Update: `ans = min(5, 4) = 4`

*Iteration 2: Consider `v = 10` as minimum*
- Range would be `[10, 14]` (since `10 + 5 - 1 = 14`)
- Use binary search: `j = bisect_right([1, 10, 100], 14) = 2`
- Elements in range `[10, 14]`: only `nums[1] = 10`
- Count of elements to keep: `j - i = 2 - 1 = 1`
- Operations needed: `5 - 1 = 4`
- Update: `ans = min(4, 4) = 4`

*Iteration 3: Consider `v = 100` as minimum*
- Range would be `[100, 104]` (since `100 + 5 - 1 = 104`)
- Use binary search: `j = bisect_right([1, 10, 100], 104) = 3`
- Elements in range `[100, 104]`: only `nums[2] = 100`
- Count of elements to keep: `j - i = 3 - 2 = 1`
- Operations needed: `5 - 1 = 4`
- Update: `ans = min(4, 4) = 4`

**Result:** Minimum operations needed = 4

We need to replace 4 elements. For example, if we keep `1` and replace the others with `[2, 3, 4, 5]`, we get the continuous array `[1, 2, 3, 4, 5]`.

## Python Solution

```python
from typing import List
from bisect import bisect_right

class Solution:
    def minOperations(self, nums: List[int]) -> int:
        # Store the original length of the array
        original_length = len(nums)

        # Initialize minimum operations to worst case (changing all elements)
        min_operations = original_length

        # Remove duplicates and sort the array
        # This gives us all unique values in ascending order
        unique_sorted_nums = sorted(set(nums))

        # Iterate through each unique value as a potential start of the continuous range
        for start_index, start_value in enumerate(unique_sorted_nums):
            # Calculate the maximum value that would be in a continuous range
            # starting from start_value with original_length elements
            # Range would be [start_value, start_value + original_length - 1]
            max_value_in_range = start_value + original_length - 1

            # Find the index right after the last element that fits in our range
            # bisect_right returns the insertion point for max_value_in_range
            end_index = bisect_right(unique_sorted_nums, max_value_in_range)

            # Calculate how many unique values from the original array
            # already fall within this continuous range
            elements_in_range = end_index - start_index

            # Calculate operations needed: total elements - elements already in range
            operations_needed = original_length - elements_in_range

            # Update minimum operations if we found a better solution
            min_operations = min(min_operations, operations_needed)

        return min_operations
```

## Time and Space Complexity

The time complexity is `O(n × log n)`, where `n` is the length of the input array. This comes from:
- Converting to set: `O(n)`
- Sorting the unique elements: `O(m × log m)` where `m ≤ n` is the number of unique elements
- Iterating through unique elements: `O(m)` iterations
- Each iteration performs `bisect_right`: `O(log m)` per iteration
- Overall: `O(n) + O(m × log m) + O(m × log m) = O(n × log n)` in the worst case when all elements are unique

The space complexity is `O(n)`, not `O(log n)` as stated in the reference. This comes from:
- Creating a set and sorted list of unique elements: `O(m)` where `m ≤ n`, which is `O(n)` in the worst case
- The sorting algorithm's recursive stack space: `O(log m)` which is `O(log n)` in the worst case
- Overall: `O(n)` dominates

Note: The reference answer states `O(log n)` for space complexity, which would only account for the sorting stack space and assumes the sorted unique array doesn't count as extra space (perhaps considering it replaces the original array reference). However, since we create a new data structure from `set(nums)` and then sort it, the more accurate space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using the Wrong Binary Search Template Variant**

A common mistake is using `while left < right` with `right = mid` instead of the standard boundary-finding template. The correct template uses `while left <= right` with `right = mid - 1` and tracks the answer with `firstTrueIndex`.

**Incorrect Implementation:**
```java
while (left < right) {
    int mid = (left + right) >> 1;
    if (nums[mid] > target) {
        right = mid;  // Wrong pattern
    } else {
        left = mid + 1;
    }
}
return left;  // Returns left directly
```

**Correct Implementation:**
```java
int firstTrueIndex = rightBound;  // Default if not found
while (left <= right) {
    int mid = left + (right - left) / 2;
    if (nums[mid] > target) {
        firstTrueIndex = mid;  // Track the answer
        right = mid - 1;
    } else {
        left = mid + 1;
    }
}
return firstTrueIndex;
```

### 2. **Forgetting to Handle Duplicates**
**Pitfall:** Working with the original array without removing duplicates leads to incorrect calculations. Duplicate values would be counted multiple times when determining elements within a range, causing the algorithm to underestimate the number of operations needed.

**Example:**
```python
# Incorrect approach - not removing duplicates
nums = [1, 1, 2, 2, 3]
# If we don't deduplicate, we might incorrectly count 5 elements in range [1,5]
# But we actually only have 3 unique values (1, 2, 3)
```

**Solution:** Always deduplicate the array before processing:
```python
unique_sorted_nums = sorted(set(nums))  # Removes duplicates and sorts
```

### 2. **Using Binary Search Incorrectly**
**Pitfall:** Using `bisect_left` instead of `bisect_right` or calculating the wrong boundary value can lead to off-by-one errors.

**Example:**
```python
# Incorrect - using bisect_left
end_index = bisect_left(unique_sorted_nums, max_value_in_range)
# This would miss elements equal to max_value_in_range

# Incorrect - wrong boundary calculation
max_value_in_range = start_value + original_length  # Should be original_length - 1
```

**Solution:** Use `bisect_right` to include all elements up to and including the maximum value:
```python
max_value_in_range = start_value + original_length - 1
end_index = bisect_right(unique_sorted_nums, max_value_in_range)
```

### 3. **Misunderstanding the Continuous Array Definition**
**Pitfall:** Confusing "continuous" with "sorted" or thinking elements need to be consecutive in their current positions rather than just existing as consecutive values.

**Example:**
```python
# Array [4, 2, 5, 3] is continuous even though not sorted
# It contains consecutive integers: 2, 3, 4, 5
# Some might incorrectly think this needs operations to sort it
```

**Solution:** Remember that continuous means:
- All unique elements
- Max - Min = length - 1
- Position in array doesn't matter, only the values present

### 4. **Not Considering All Possible Ranges**
**Pitfall:** Only checking ranges starting from the minimum value in the array, missing potentially better solutions.

**Example:**
```python
nums = [1, 10, 100, 200, 300]
# If we only consider range starting at 1: [1, 2, 3, 4, 5]
# We'd need to replace 4 elements
# But range [100, 101, 102, 103, 104] only needs 2 replacements
```

**Solution:** Iterate through every unique value as a potential starting point:
```python
for start_index, start_value in enumerate(unique_sorted_nums):
    # Check range starting from each unique value
```

