# 2454. Next Greater Element IV

> Source: https://algo.monster/liteproblems/2454

## Problem Statement

You are given a 0-indexed array of non-negative integers `nums`. Your task is to find the **second greater** integer for each element in the array.

For an element `nums[i]`, its second greater integer is defined as `nums[j]` where:
- `j > i` (the element must come after `nums[i]` in the array)
- `nums[j] > nums[i]` (the element must be greater than `nums[i]`)
- There exists **exactly one** index `k` such that `i < k < j` and `nums[k] > nums[i]` (there must be exactly one element between them that is also greater than `nums[i]`)

In other words, `nums[j]` is the second element to the right of `nums[i]` that is greater than `nums[i]`.

If no such second greater element exists for `nums[i]`, the result should be `-1`.

**Example:**
For the array `[1, 2, 4, 3]`:
- For `1` (index 0): The first greater element is `2` at index 1, and the second greater element is `4` at index 2. Result: `4`
- For `2` (index 1): The first greater element is `4` at index 2, and the second greater element is `3` at index 3. Result: `3`
- For `4` (index 2): There are no greater elements to its right. Result: `-1`
- For `3` (index 3): There are no elements to its right. Result: `-1`

The function should return an integer array `answer` where `answer[i]` represents the second greater integer of `nums[i]`.

## Intuition

The key insight is to process elements from largest to smallest. Why? Because when we're looking for the "second greater" element for any value, we only care about elements that are greater than it. By processing from largest to smallest, we ensure that when we reach an element, all larger elements have already been processed.

Think about it this way: if we're at element `x`, we know that all elements greater than `x` have already been seen. Now we need to find which of these greater elements appears as the second one after `x`'s position in the original array.

Here's where the ordered set becomes crucial. As we process each element, we maintain an ordered set of indices of all elements we've seen so far (which are all greater than or equal to the current element). For the current element at index `i`, we need to find indices in our set that are greater than `i` (elements that appear after position `i`).

Using [binary search](/problems/binary_search_intro) (`bisect_right(i)`), we can quickly find the position in the ordered set where `i` would be inserted. This gives us the count of indices greater than `i`. If there are at least two such indices (meaning `j + 1 < len(sl)`), then the element at the second index (`sl[j + 1]`) corresponds to our answer - it's the second element after position `i` that has a value greater than `nums[i]`.

The beauty of this approach is that by processing elements in descending order of their values, we naturally filter out smaller elements. When we're looking for the second greater element of `nums[i]`, the ordered set only contains indices of elements with values `≥ nums[i]`, and we just need to find the second index that comes after `i` in the original array order.

**Learn more about [Stack](/problems/stack_intro), [Binary Search](/problems/binary_search_intro), [Sorting](/problems/sorting_summary), [Monotonic Stack](/problems/mono_stack_intro) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation follows a [sorting](/problems/sorting_summary)-based approach with an ordered set to efficiently find the second greater element for each position.

**Step 1: Create and Sort Pairs**
```python
arr = [(x, i) for i, x in enumerate(nums)]
arr.sort(key=lambda x: -x[0])
```
We create pairs of `(value, index)` for each element and sort them in descending order by value. This ensures we process larger elements first.

**Step 2: Initialize Data Structures**
```python
sl = SortedList()
ans = [-1] * n
```
- `SortedList` maintains indices in sorted order as we process elements
- `ans` array is initialized with `-1` (default when no second greater element exists)

**Step 3: Process Elements from Largest to Smallest**
```python
for _, i in arr:
    j = sl.bisect_right(i)
    if j + 1 < len(sl):
        ans[i] = nums[sl[j + 1]]
    sl.add(i)
```

For each element at original index `i`:
- **Find position in sorted list**: `sl.bisect_right(i)` returns the position where `i` would be inserted to maintain sorted order. This gives us the count of indices already in the set that are greater than `i`.
- **Check for second greater**: If `j + 1 < len(sl)`, it means there are at least two indices in the set that are greater than `i`. The element at `sl[j + 1]` is the second index after `i` in the original array.
- **Update answer**: Set `ans[i] = nums[sl[j + 1]]` to record the second greater element.
- **Add current index**: Add `i` to the sorted list for processing future elements.

**Why This Works:**
When processing an element with value `x` at index `i`, the sorted list contains indices of all elements with values `≥ x` (since we process in descending order). Among these indices, we need exactly the second one that appears after position `i` in the original array. The `bisect_right` operation efficiently finds where `i` fits in the sorted order of indices, and `sl[j + 1]` gives us the second index greater than `i`.

**Time Complexity:** `O(n log n)` for [sorting](/problems/sorting_summary) and `O(n log n)` for n insertions and binary searches in the sorted list.
**Space Complexity:** `O(n)` for the additional data structures.

## Example Walkthrough

Let's walk through the array `[2, 4, 0, 9, 6]` step by step.

**Initial Setup:**
- Create pairs: `[(2,0), (4,1), (0,2), (9,3), (6,4)]`
- Sort by value (descending): `[(9,3), (6,4), (4,1), (2,0), (0,2)]`
- Initialize: `sl = []`, `ans = [-1, -1, -1, -1, -1]`

**Processing each element (from largest to smallest):**

**Step 1:** Process `(9, 3)` - value 9 at index 3
- `sl.bisect_right(3)` = 0 (no indices > 3 in sl yet)
- Since 0 + 1 = 1 is not < 0, no second greater element
- Add 3 to sl: `sl = [3]`
- `ans = [-1, -1, -1, -1, -1]`

**Step 2:** Process `(6, 4)` - value 6 at index 4
- `sl.bisect_right(4)` = 1 (all indices in sl are ≤ 4)
- Since 1 + 1 = 2 is not < 1, no second greater element
- Add 4 to sl: `sl = [3, 4]`
- `ans = [-1, -1, -1, -1, -1]`

**Step 3:** Process `(4, 1)` - value 4 at index 1
- `sl.bisect_right(1)` = 0 (indices 3 and 4 are both > 1)
- Since 0 + 1 = 1 < 2, we have a second greater element!
- `ans[1] = nums[sl[1]] = nums[4] = 6`
- Add 1 to sl: `sl = [1, 3, 4]`
- `ans = [-1, 6, -1, -1, -1]`

**Step 4:** Process `(2, 0)` - value 2 at index 0
- `sl.bisect_right(0)` = 0 (indices 1, 3, 4 are all > 0)
- Since 0 + 1 = 1 < 3, we have a second greater element!
- `ans[0] = nums[sl[1]] = nums[3] = 9`
- Add 0 to sl: `sl = [0, 1, 3, 4]`
- `ans = [9, 6, -1, -1, -1]`

**Step 5:** Process `(0, 2)` - value 0 at index 2
- `sl.bisect_right(2)` = 2 (indices 3 and 4 are > 2)
- Since 2 + 1 = 3 < 4, we have a second greater element!
- `ans[2] = nums[sl[3]] = nums[4] = 6`
- Add 2 to sl: `sl = [0, 1, 2, 3, 4]`
- `ans = [9, 6, 6, -1, -1]`

**Final Result:** `[9, 6, 6, -1, -1]`

**Verification:**
- For `nums[0]=2`: Greater elements to the right are 4(index 1), 9(index 3), 6(index 4). The second one is 9. ✓
- For `nums[1]=4`: Greater elements to the right are 9(index 3), 6(index 4). The second one is 6. ✓
- For `nums[2]=0`: Greater elements to the right are 9(index 3), 6(index 4). The second one is 6. ✓
- For `nums[3]=9`: No greater elements exist. Result is -1. ✓
- For `nums[4]=6`: No elements to the right. Result is -1. ✓

## Python Solution

```python
from sortedcontainers import SortedList
from typing import List

class Solution:
    def secondGreaterElement(self, nums: List[int]) -> List[int]:
        # Create pairs of (value, index) for sorting
        value_index_pairs = [(value, index) for index, value in enumerate(nums)]

        # Sort pairs by value in descending order (largest first)
        value_index_pairs.sort(key=lambda pair: -pair[0])

        # SortedList to maintain indices of processed elements in sorted order
        processed_indices = SortedList()

        # Initialize result array with -1 (default when no second greater element exists)
        n = len(nums)
        result = [-1] * n

        # Process elements from largest to smallest value
        for value, current_index in value_index_pairs:
            # Find position where current_index would be inserted (indices greater than current_index)
            position = processed_indices.bisect_right(current_index)

            # Check if there are at least 2 elements to the right of current_index
            if position + 1 < len(processed_indices):
                # The element at position+1 is the second element to the right
                second_greater_index = processed_indices[position + 1]
                result[current_index] = nums[second_greater_index]

            # Add current index to the sorted list for future iterations
            processed_indices.add(current_index)

        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The time complexity is dominated by three main operations:
1. Creating the array of tuples `arr`: `O(n)`
2. Sorting `arr` by value in descending order: `O(n × log n)`
3. Iterating through `arr` and performing operations with `SortedList`: `O(n × log n)`
   - Each `bisect_right()` operation takes `O(log n)` time
   - Each `add()` operation takes `O(log n)` time to maintain sorted order
   - We perform both operations `n` times

Since these operations are sequential, the overall time complexity is `O(n) + O(n × log n) + O(n × log n) = O(n × log n)`.

**Space Complexity:** `O(n)`

The space complexity comes from:
1. The `arr` list containing `n` tuples: `O(n)`
2. The `SortedList` which can contain up to `n` elements: `O(n)`
3. The `ans` list of size `n`: `O(n)`

All these data structures use linear space, resulting in a total space complexity of `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall: Misunderstanding the Index Relationship in SortedList

**The Problem:**
A common mistake is misinterpreting what `bisect_right(current_index)` returns and how it relates to finding the "second greater" element. Developers often confuse:
1. The position in the SortedList (what bisect_right returns)
2. The actual indices stored in the SortedList
3. Which element represents the "second greater"

**Example of the Mistake:**
```python
# INCORRECT interpretation
position = processed_indices.bisect_right(current_index)
# Wrong: Thinking position itself is an index in nums
result[current_index] = nums[position + 1]  # This would cause IndexError or wrong result
```

**Why This Happens:**
When `bisect_right(current_index)` returns a value like 2, it means there are 2 indices in the SortedList that are greater than `current_index`. However, these indices might be something like [5, 7] - not [0, 1]. The confusion arises because:
- `position` is just a count/position in the SortedList
- `processed_indices[position]` gives you the actual array index
- You need `processed_indices[position + 1]` for the second greater element's index

**The Correct Solution:**
```python
# CORRECT implementation
position = processed_indices.bisect_right(current_index)
if position + 1 < len(processed_indices):
    # Get the actual index from the SortedList at position+1
    second_greater_index = processed_indices[position + 1]
    # Use that index to get the value from nums
    result[current_index] = nums[second_greater_index]
```

**Visual Example:**
For `nums = [2, 4, 0, 9, 6]`, when processing element `0` at index 2:
- `processed_indices` might be `[1, 3, 4]` (indices of elements 4, 9, and 6)
- `bisect_right(2)` returns `1` (index 2 would go after position 0 but before position 1)
- `processed_indices[1]` = 3 (first greater element's index)
- `processed_indices[2]` = 4 (second greater element's index)
- So `result[2] = nums[4] = 6`

**Key Takeaway:**
Always remember that SortedList operations return positions within the list, not the actual values stored in the list. You must use these positions to access the actual indices, then use those indices to access values in the original array.

