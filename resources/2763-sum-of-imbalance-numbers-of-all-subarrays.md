# 2763. Sum of Imbalance Numbers of All Subarrays

> Source: https://algo.monster/liteproblems/2763

## Problem Statement

You are given a 0-indexed integer array `nums`. Your task is to find the sum of imbalance numbers across all possible subarrays.

The **imbalance number** of an array is defined as follows:
- First, sort the array to get `sarr`
- Count how many indices `i` (where `0 <= i < n - 1`) satisfy the condition: `sarr[i+1] - sarr[i] > 1`
- This count is the imbalance number

In other words, the imbalance number counts how many "gaps" exist in the sorted array where consecutive elements differ by more than 1.

For example:
- Array `[2, 5, 3]` becomes `[2, 3, 5]` when sorted
- Check consecutive pairs: `3 - 2 = 1` (no gap), `5 - 3 = 2` (gap exists)
- Imbalance number = 1

A **subarray** is any contiguous sequence of elements from the original array. You need to:
1. Consider every possible subarray of `nums`
2. Calculate the imbalance number for each subarray
3. Return the sum of all these imbalance numbers

## Intuition

The key insight is that we don't need to sort each subarray from scratch to calculate its imbalance number. Instead, we can maintain a dynamically sorted structure as we expand our subarray one element at a time.

Think about what happens when we add a new element to an already sorted subarray. The new element will be inserted at some position, and this insertion can only affect the gaps around its insertion point. Specifically:

1. **Before insertion**: We might have a gap between two consecutive elements in our sorted list.
2. **After insertion**: The new element might:
   - Create new gaps with its immediate neighbors
   - Fill an existing gap between two elements

Let's trace through this idea. Suppose we have a sorted list `[2, 5, 8]` with imbalance count of 2 (gaps at 2→5 and 5→8). Now we want to add element `6`:
- Element `6` will be inserted between `5` and `8`
- Check left neighbor: `6 - 5 = 1` (no new gap)
- Check right neighbor: `8 - 6 = 2` (creates a gap)
- But wait! Originally `8 - 5 = 3` was a gap, and now it's split into two parts
- Net effect: We removed one gap (5→8) and added one gap (6→8), so the count remains 2

This observation leads to our approach: For each starting position `i`, we incrementally build subarrays by adding elements one by one. We use a sorted list to maintain order and a counter `cnt` to track the current imbalance number. When adding `nums[j]`:
- Find where it fits in the sorted order (using binary search)
- Check if it creates gaps with its would-be neighbors
- Check if it fills an existing gap
- Update the running count accordingly

This way, we efficiently calculate the imbalance number for all subarrays in `O(n² log n)` time, avoiding the need to sort each subarray independently.

## Solution Approach

We enumerate all possible subarrays by fixing the left endpoint `i` and extending the right endpoint `j` from `i` to `n-1`. For each subarray `[i, j]`, we maintain its elements in a sorted order using a `SortedList` data structure.

**Implementation Steps:**

1. **Initialize for each starting position**: For each left endpoint `i`, create an empty `SortedList` and set the imbalance counter `cnt = 0`.

2. **Extend the subarray**: For each right endpoint `j` from `i` to `n-1`, we add `nums[j]` to our sorted structure and update the imbalance count.

3. **Find insertion position**: Use `bisect_left(nums[j])` to find index `k` where `nums[j]` should be inserted. This gives us:
   - `k`: The index of the first element >= `nums[j]` (or length of list if none exists)
   - `h = k - 1`: The index of the last element < `nums[j]` (or -1 if none exists)

4. **Update imbalance count**: Before inserting `nums[j]`, we check how it affects the gaps:
 
   - **Check left gap**: If element at index `h` exists (`h >= 0`) and `nums[j] - sl[h] > 1`, we're creating a new gap on the left side, so `cnt += 1`
 
   - **Check right gap**: If element at index `k` exists (`k < len(sl)`) and `sl[k] - nums[j] > 1`, we're creating a new gap on the right side, so `cnt += 1`
 
   - **Check filled gap**: If both neighbors exist (`h >= 0` and `k < len(sl)`) and originally `sl[k] - sl[h] > 1`, then inserting `nums[j]` between them fills this existing gap, so `cnt -= 1`

5. **Add element and accumulate**: Insert `nums[j]` into the sorted list, then add the current `cnt` to our answer.

**Example Walkthrough:**

Consider `nums = [2, 5, 3]` and subarray starting at index 0:
- Start with empty `SortedList`, `cnt = 0`
- Add `2`: List becomes `[2]`, no gaps, `cnt = 0`
- Add `5`: 
  - `k = 1`, `h = 0`
  - Check: `5 - 2 = 3 > 1`, so `cnt += 1`
  - List becomes `[2, 5]`, `cnt = 1`
- Add `3`:
  - `k = 1` (position where 3 should go), `h = 0`
  - Left check: `3 - 2 = 1`, no gap
  - Right check: `5 - 3 = 2 > 1`, so `cnt += 1`
  - Filled gap check: `5 - 2 = 3 > 1` was a gap, so `cnt -= 1`
  - List becomes `[2, 3, 5]`, `cnt = 1`

The time complexity is `O(n² log n)` where `n` is the length of the array, as we have two nested loops and each insertion/search in `SortedList` takes `O(log n)` time.

## Example Walkthrough

Let's walk through `nums = [4, 1, 3, 2]` and calculate the sum of imbalance numbers for all subarrays.

**Starting from index 0 (element 4):**
- Subarray `[4]`: SortedList = `[4]`, no gaps possible with one element, cnt = 0
- Subarray `[4, 1]`: 
  - Find position for 1: k = 0 (1 goes before 4), h = -1
  - Right gap check: `4 - 1 = 3 > 1`, creates gap, cnt = 0 + 1 = 1
  - SortedList = `[1, 4]`, imbalance = 1
- Subarray `[4, 1, 3]`:
  - Find position for 3: k = 1 (3 goes between 1 and 4), h = 0
  - Left gap check: `3 - 1 = 2 > 1`, creates gap, cnt = 1 + 1 = 2
  - Right gap check: `4 - 3 = 1`, no gap
  - Filled gap check: Originally `4 - 1 = 3 > 1` was a gap, now filled, cnt = 2 - 1 = 1
  - SortedList = `[1, 3, 4]`, imbalance = 1
- Subarray `[4, 1, 3, 2]`:
  - Find position for 2: k = 1 (2 goes between 1 and 3), h = 0
  - Left gap check: `2 - 1 = 1`, no gap
  - Right gap check: `3 - 2 = 1`, no gap
  - Filled gap check: Originally `3 - 1 = 2 > 1` was a gap, now filled, cnt = 1 - 1 = 0
  - SortedList = `[1, 2, 3, 4]`, imbalance = 0

**Starting from index 1 (element 1):**
- Subarray `[1]`: SortedList = `[1]`, cnt = 0
- Subarray `[1, 3]`:
  - Find position for 3: k = 1, h = 0
  - Left gap check: `3 - 1 = 2 > 1`, creates gap, cnt = 0 + 1 = 1
  - SortedList = `[1, 3]`, imbalance = 1
- Subarray `[1, 3, 2]`:
  - Find position for 2: k = 1, h = 0
  - Left gap check: `2 - 1 = 1`, no gap
  - Right gap check: `3 - 2 = 1`, no gap
  - Filled gap check: `3 - 1 = 2 > 1` was a gap, now filled, cnt = 1 - 1 = 0
  - SortedList = `[1, 2, 3]`, imbalance = 0

**Starting from index 2 (element 3):**
- Subarray `[3]`: SortedList = `[3]`, cnt = 0
- Subarray `[3, 2]`:
  - Find position for 2: k = 0, h = -1
  - Right gap check: `3 - 2 = 1`, no gap
  - SortedList = `[2, 3]`, imbalance = 0

**Starting from index 3 (element 2):**
- Subarray `[2]`: SortedList = `[2]`, cnt = 0

**Total sum = 0 + 1 + 1 + 0 + 0 + 1 + 0 + 0 + 0 + 0 = 3**

The algorithm efficiently tracks how each new element affects the gaps in the sorted structure, avoiding the need to re-sort and recalculate from scratch for each subarray.

## Python Solution

```python
from typing import List
from sortedcontainers import SortedList

class Solution:
    def sumImbalanceNumbers(self, nums: List[int]) -> int:
        """
        Calculate the sum of imbalance numbers for all subarrays.
        An imbalance in a sorted array is when two adjacent elements differ by more than 1.
      
        Args:
            nums: List of integers
          
        Returns:
            Sum of imbalance counts across all subarrays
        """
        n = len(nums)
        total_imbalance = 0
      
        # Iterate through all possible starting positions of subarrays
        for start_idx in range(n):
            # SortedList maintains elements in sorted order with O(log n) insertion
            sorted_subarray = SortedList()
            current_imbalance_count = 0
          
            # Extend the subarray from start_idx to the right
            for end_idx in range(start_idx, n):
                current_num = nums[end_idx]
              
                # Find where current_num would be inserted in the sorted list
                insert_position = sorted_subarray.bisect_left(current_num)
                left_neighbor_idx = insert_position - 1
              
                # Check if adding current_num creates new imbalances
              
                # Check imbalance with left neighbor
                if left_neighbor_idx >= 0 and current_num - sorted_subarray[left_neighbor_idx] > 1:
                    current_imbalance_count += 1
              
                # Check imbalance with right neighbor
                if insert_position < len(sorted_subarray) and sorted_subarray[insert_position] - current_num > 1:
                    current_imbalance_count += 1
              
                # Check if adding current_num removes an existing imbalance
                # (by filling a gap between two elements that had an imbalance)
                if (left_neighbor_idx >= 0 and 
                    insert_position < len(sorted_subarray) and 
                    sorted_subarray[insert_position] - sorted_subarray[left_neighbor_idx] > 1):
                    current_imbalance_count -= 1
              
                # Add the current number to the sorted list
                sorted_subarray.add(current_num)
              
                # Add current subarray's imbalance count to the total
                total_imbalance += current_imbalance_count
      
        return total_imbalance
```

## Time and Space Complexity

**Time Complexity:** `O(n² × log n)`

The code uses two nested loops:
- The outer loop runs from `i = 0` to `n-1`, iterating `n` times
- The inner loop runs from `j = i` to `n-1`, iterating up to `n` times in the worst case

Within the inner loop, the following operations are performed:
- `sl.bisect_left(nums[j])`: Binary search operation taking `O(log n)` time in the worst case when the SortedList contains up to `n` elements
- `sl.add(nums[j])`: Insertion into a SortedList taking `O(log n)` time
- Array access operations `sl[h]` and `sl[k]`: `O(1)` time each
- Arithmetic operations and comparisons: `O(1)` time

The total number of iterations across both loops is `n + (n-1) + (n-2) + ... + 1 = n(n+1)/2 = O(n²)`

For each iteration, we perform `O(log n)` operations (bisect_left and add), resulting in a total time complexity of `O(n² × log n)`.

**Space Complexity:** `O(n)`

The space usage comes from:
- The SortedList `sl` which can store up to `n` elements in the worst case (when processing the entire array from index 0)
- A few integer variables (`i`, `j`, `k`, `h`, `cnt`, `ans`, `n`) which take `O(1)` space

Therefore, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Duplicate Elements Handling**
The current solution doesn't account for duplicate elements in the array. When duplicates exist, the `SortedList` will maintain them, but the imbalance calculation logic assumes all elements are unique.

**Problem Example:**
For array `[2, 3, 3, 5]`, when sorted, we have consecutive identical elements. The gap between `3` and `3` is 0, which shouldn't count as an imbalance. However, the current logic might miscount imbalances when inserting a duplicate.

**Solution:**
Check if the element already exists before updating the imbalance count:

```python
def sumImbalanceNumbers(self, nums: List[int]) -> int:
    n = len(nums)
    total_imbalance = 0
  
    for start_idx in range(n):
        sorted_subarray = SortedList()
        current_imbalance_count = 0
      
        for end_idx in range(start_idx, n):
            current_num = nums[end_idx]
          
            # Check if this is a duplicate
            if current_num in sorted_subarray:
                # Duplicates don't create new imbalances
                sorted_subarray.add(current_num)
                total_imbalance += current_imbalance_count
                continue
          
            insert_position = sorted_subarray.bisect_left(current_num)
            left_neighbor_idx = insert_position - 1
          
            # Rest of the logic remains the same...
```

### 2. **Edge Case with Single Element Subarrays**
When dealing with subarrays of size 1, there are no pairs to check for imbalances, so the imbalance count should always be 0. The current solution handles this correctly, but it's easy to overlook when modifying the code.

**Verification:**
Ensure that when `sorted_subarray` has only one element, `current_imbalance_count` remains 0, which it does since no conditions for incrementing will be met.

### 3. **Integer Overflow for Large Inputs**
For very large arrays with many subarrays having high imbalance counts, the sum might exceed typical integer limits in some languages (though Python handles arbitrary precision integers).

**Solution for other languages:**
Use appropriate data types (e.g., `long long` in C++) or modulo arithmetic if required by the problem constraints.

### 4. **Misunderstanding Gap Definition**
A common mistake is confusing the gap condition. The imbalance occurs when `sarr[i+1] - sarr[i] > 1`, not `>= 1`. This means consecutive numbers like `[2, 3]` do NOT create an imbalance.

**Debugging tip:**
Add assertions or debug prints to verify the imbalance count for known test cases:

```python
# Debug helper
def verify_imbalance(arr):
    sorted_arr = sorted(arr)
    count = 0
    for i in range(len(sorted_arr) - 1):
        if sorted_arr[i+1] - sorted_arr[i] > 1:
            count += 1
    return count
```

