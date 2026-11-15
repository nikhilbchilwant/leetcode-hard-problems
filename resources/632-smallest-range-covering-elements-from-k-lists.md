# 632. Smallest Range Covering Elements from K Lists

> Source: https://algo.monster/liteproblems/632

## Problem Statement

You are given `k` lists where each list contains sorted integers in non-decreasing order. Your task is to find the smallest range `[a, b]` that includes at least one number from each of the `k` lists.

The problem defines how to compare two ranges:
- Range `[a, b]` is considered smaller than range `[c, d]` if:
  - The width of `[a, b]` is less than the width of `[c, d]` (i.e., `b - a < d - c`), OR
  - If both ranges have the same width (i.e., `b - a == d - c`), then the range with the smaller starting point is considered smaller (i.e., `a < c`)

For example, if you have 3 lists:
- List 1: `[1, 5, 8]`
- List 2: `[2, 9]`  
- List 3: `[3, 6, 10]`

You need to find a range `[a, b]` where:
- The range contains at least one element from List 1
- The range contains at least one element from List 2
- The range contains at least one element from List 3
- Among all such valid ranges, this range is the smallest according to the comparison rules above

The goal is to minimize the range width first, and if there are multiple ranges with the same width, choose the one that starts earliest.

## Intuition

The key insight is that we need to find a continuous range that covers at least one element from each list. Think of this problem as trying to find a window that "touches" all lists simultaneously.

If we merge all the numbers from all lists into a single sorted array while keeping track of which list each number came from, we can use a [sliding window](/problems/sliding_window_maximum) approach. Why does this work?

Consider that any valid range `[a, b]` must have `a` and `b` as actual numbers from our lists (not arbitrary values). The optimal range will have its endpoints as numbers from the lists because we want to minimize the range width.

Once we have all numbers sorted in a single array with their list indices, we can use two pointers to maintain a window. We expand the window by moving the right pointer and include more numbers. As we include numbers, we keep track of which lists are represented in our current window using a counter.

When our window contains at least one number from each of the `k` lists, we have a valid range. At this point, we can try to shrink the window from the left to find a potentially smaller valid range. We keep shrinking as long as the window remains valid (still contains elements from all `k` lists).

The [sliding window](/problems/sliding_window_maximum) technique works here because:
1. Once we have a valid window covering all lists, any smaller valid range must either start later (higher left boundary) or end earlier (lower right boundary)
2. By [sorting](/problems/sorting_summary) all numbers first, we ensure we're checking ranges in a systematic way
3. The two-pointer approach ensures we check all possible valid ranges efficiently without redundant calculations

This approach essentially transforms the problem from dealing with `k` separate lists into dealing with a single sorted sequence where we need to find the smallest window containing all `k` different "colors" (list indices).

**Learn more about [Greedy](/problems/greedy_intro), [Sorting](/problems/sorting_summary), [Sliding Window](/problems/sliding_window_maximum) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution implements the [sorting](/problems/sorting_summary) and [sliding window](/problems/sliding_window_maximum) approach discussed in the intuition. Let's walk through the implementation step by step:

**Step 1: Create a merged sorted array**
```python
t = [(x, i) for i, v in enumerate(nums) for x in v]
t.sort()
```
We create a list `t` where each element is a tuple `(x, i)`:
- `x` is the actual value from the lists
- `i` is the index indicating which list this value came from

After creating this list, we sort it by the values (Python sorts tuples by the first element by default). This effectively merges all `k` sorted lists into one sorted sequence while preserving the origin information.

**Step 2: Initialize data structures**
```python
cnt = Counter()
ans = [-inf, inf]
j = 0
```
- `cnt`: A Counter (hash table) to track how many elements from each list are in our current window
- `ans`: Stores the best range found so far, initialized to `[-inf, inf]` to ensure any valid range will be better
- `j`: Left pointer for the [sliding window](/problems/sliding_window_maximum) (starts at 0)

**Step 3: [Sliding window](/problems/sliding_window_maximum) implementation**
```python
for i, (b, v) in enumerate(t):
    cnt[v] += 1
    while len(cnt) == len(nums):
        # Process valid window
```
We iterate through the sorted array with `i` as the right pointer:
- `b` is the value at the right end of our window
- `v` is which list this value belongs to
- We add this element to our window by incrementing `cnt[v]`

**Step 4: Process valid windows**
When `len(cnt) == len(nums)`, we have elements from all `k` lists in our window:
```python
a = t[j][0]  # Left boundary value
x = b - a - (ans[1] - ans[0])  # Compare range widths
if x < 0 or (x == 0 and a < ans[0]):
    ans = [a, b]
```
- `a` is the value at the left boundary of our window
- We calculate `x` as the difference between current range width and best range width
- Update `ans` if current range is smaller (negative `x`) or same width but starts earlier

**Step 5: Shrink the window**
```python
w = t[j][1]  # Which list the left element belongs to
cnt[w] -= 1
if cnt[w] == 0:
    cnt.pop(w)
j += 1
```
We try to shrink the window from the left:
- Decrease the count for the list that the leftmost element belongs to
- If count becomes 0, remove that list from the counter entirely
- Move the left pointer forward

The inner `while` loop continues shrinking as long as we still have all lists represented. Once we lose coverage of any list, we exit the inner loop and continue expanding with the outer loop.

**Time Complexity**: `O(n log n)` where `n` is the total number of elements across all lists, due to [sorting](/problems/sorting_summary).

**Space Complexity**: `O(n)` for storing the merged array `t` and the counter.

## Example Walkthrough

Let's walk through a small example with 3 lists:
- List 1: `[4, 10]`
- List 2: `[1, 9]`  
- List 3: `[5, 12]`

**Step 1: Create merged sorted array**
We create tuples (value, list_index) and sort them:
```
Original tuples: [(4,0), (10,0), (1,1), (9,1), (5,2), (12,2)]
Sorted t: [(1,1), (4,0), (5,2), (9,1), (10,0), (12,2)]
```

**Step 2: Initialize variables**
- `cnt = {}` (empty counter)
- `ans = [-inf, inf]` (initial best range)
- `j = 0` (left pointer)

**Step 3: Sliding window process**

*Iteration i=0:* Right pointer at (1,1)
- Add list 1 to window: `cnt = {1: 1}`
- Window has 1 list (need 3), continue expanding

*Iteration i=1:* Right pointer at (4,0)
- Add list 0 to window: `cnt = {1: 1, 0: 1}`
- Window has 2 lists (need 3), continue expanding

*Iteration i=2:* Right pointer at (5,2)
- Add list 2 to window: `cnt = {1: 1, 0: 1, 2: 1}`
- Window has 3 lists - Valid range found!
- Current range: [1, 5] (from position j=0 to i=2)
- Compare with ans: 5-1=4 < inf-(-inf), update `ans = [1, 5]`
- Try shrinking: Remove (1,1), `cnt = {0: 1, 2: 1}`, j=1
- Only 2 lists now, exit shrinking loop

*Iteration i=3:* Right pointer at (9,1)
- Add list 1 to window: `cnt = {0: 1, 2: 1, 1: 1}`
- Window has 3 lists - Valid range found!
- Current range: [4, 9] (from position j=1 to i=3)
- Compare with ans: 9-4=5 > 5-1=4, keep `ans = [1, 5]`
- Try shrinking: Remove (4,0), `cnt = {2: 1, 1: 1}`, j=2
- Only 2 lists now, exit shrinking loop

*Iteration i=4:* Right pointer at (10,0)
- Add list 0 to window: `cnt = {2: 1, 1: 1, 0: 1}`
- Window has 3 lists - Valid range found!
- Current range: [5, 10] (from position j=2 to i=4)
- Compare with ans: 10-5=5 > 5-1=4, keep `ans = [1, 5]`
- Try shrinking: Remove (5,2), `cnt = {1: 1, 0: 1}`, j=3
- Only 2 lists now, exit shrinking loop

*Iteration i=5:* Right pointer at (12,2)
- Add list 2 to window: `cnt = {1: 1, 0: 1, 2: 1}`
- Window has 3 lists - Valid range found!
- Current range: [9, 12] (from position j=3 to i=5)
- Compare with ans: 12-9=3 < 5-1=4, update `ans = [9, 12]`
- Try shrinking: Remove (9,1), `cnt = {0: 1, 2: 1}`, j=4
- Only 2 lists now, exit shrinking loop

**Final Result:** The smallest range is `[9, 12]` with width 3, which includes:
- 10 from List 1
- 9 from List 2
- 12 from List 3

## Python Solution

```python
from typing import List
from collections import Counter
from math import inf

class Solution:
    def smallestRange(self, nums: List[List[int]]) -> List[int]:
        # Create a list of (value, list_index) tuples from all lists
        # This flattens all lists while keeping track of which list each element came from
        elements = [(value, list_index) for list_index, list_values in enumerate(nums) for value in list_values]
      
        # Sort elements by their values
        elements.sort()
      
        # Counter to track how many lists are represented in current window
        list_counter = Counter()
      
        # Initialize the answer with the widest possible range
        min_range = [-inf, inf]
      
        # Left pointer for sliding window
        left = 0
      
        # Right pointer iterates through sorted elements
        for right, (right_value, list_index) in enumerate(elements):
            # Add current element's list to the counter
            list_counter[list_index] += 1
          
            # Try to shrink window from left while all lists are represented
            while len(list_counter) == len(nums):
                # Get the left boundary value
                left_value = elements[left][0]
              
                # Calculate range difference
                current_range_diff = right_value - left_value - (min_range[1] - min_range[0])
              
                # Update answer if we found a smaller range
                # or same range but with smaller starting point
                if current_range_diff < 0 or (current_range_diff == 0 and left_value < min_range[0]):
                    min_range = [left_value, right_value]
              
                # Remove leftmost element from window
                left_list_index = elements[left][1]
                list_counter[left_list_index] -= 1
              
                # Remove list from counter if no elements remain from it
                if list_counter[left_list_index] == 0:
                    list_counter.pop(left_list_index)
              
                # Move left pointer forward
                left += 1
      
        return min_range
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)` where `n` is the total number of elements across all arrays in `nums`.

- Creating list `t` by iterating through all elements: `O(n)` 
- Sorting list `t`: `O(n × log n)` - This is the dominant operation
- The sliding window loop iterates through all elements once: `O(n)`
- Inside the loop, Counter operations (`cnt[v] += 1`, `cnt[w] -= 1`, `cnt.pop(w)`) are `O(1)` on average
- The while loop moves pointer `j` at most `n` times total across all iterations

Therefore, the overall time complexity is `O(n × log n)` due to the sorting step.

**Space Complexity:** `O(n)` where `n` is the total number of elements across all arrays.

- List `t` stores tuples for all elements: `O(n)`
- Counter `cnt` stores at most `k` entries (where `k` is the number of arrays), but `k ≤ n`: `O(k)` ⊆ `O(n)`
- Variable `ans` and other variables: `O(1)`

The total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Range Comparison Logic**

When two ranges have the same width, the one with the smaller starting point wins. A common mistake is using a non-strict comparison, which replaces an earlier range with a later one of the same width.

**Incorrect Implementation:**
```python
# Wrong: <= replaces a range with a later range of equal width
if right_value - left_value <= min_range[1] - min_range[0]:
    min_range = [left_value, right_value]
```

**Why it's wrong:** The sliding window visits windows in increasing order of their right endpoint, so among windows with the same width, later ones start later. For `nums = [[1, 4], [2, 5]]`, the windows `[1, 2]` and `[4, 5]` both have width 1. With `<=`, the answer becomes `[4, 5]` instead of `[1, 2]`.

**Correct Implementation:**
```python
current_range_diff = right_value - left_value - (min_range[1] - min_range[0])
if current_range_diff < 0 or (current_range_diff == 0 and left_value < min_range[0]):
    min_range = [left_value, right_value]
```
Because of the visiting order, a strict `<` on the width alone would also be correct here; the explicit tie-break makes the rule from the problem statement visible and keeps the code correct if the windows were visited in a different order.

### 2. **Premature Window Shrinking**

Another pitfall is shrinking the window too aggressively or incorrectly managing when to stop shrinking.

**Incorrect Implementation:**
```python
# Wrong: Shrinking without checking if all lists are still covered
while left < right:  # Should be: while len(list_counter) == len(nums)
    # shrink window logic
```

**Why it's wrong:** The window should only be shrunk while we still have at least one element from each list. Once we lose coverage of any list, we must stop shrinking and continue expanding.

### 3. **Not Handling Counter Cleanup**

Forgetting to remove lists from the counter when their count reaches zero can lead to incorrect window validity checks.

**Incorrect Implementation:**
```python
# Wrong: Just decrementing without removing empty entries
list_counter[left_list_index] -= 1
# Missing: list_counter.pop(left_list_index) when count is 0
```

**Why it's wrong:** If we don't remove lists with zero count from the counter, `len(list_counter)` will give incorrect results, making us think we still have elements from all lists when we don't.

**Correct Implementation:**
```python
list_counter[left_list_index] -= 1
if list_counter[left_list_index] == 0:
    list_counter.pop(left_list_index)
```

### 4. **Using Wrong Initial Values**

Initializing the answer range incorrectly can cause issues with the first valid range found.

**Incorrect Implementation:**
```python
# Wrong: Using arbitrary values
min_range = [0, 0]  # or min_range = []
```

**Why it's wrong:** Using `[0, 0]` might accidentally be better than actual valid ranges, and using an empty list will cause comparison errors.

**Correct Implementation:**
```python
min_range = [-inf, inf]  # Guarantees any valid range will be better
```

This ensures that the first valid range found will always replace the initial value since any finite range will be smaller than an infinite one.

