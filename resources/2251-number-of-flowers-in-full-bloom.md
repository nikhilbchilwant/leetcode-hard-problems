# 2251. Number of Flowers in Full Bloom

> Source: https://algo.monster/liteproblems/2251

## Problem Statement

You have information about when flowers bloom and when people arrive to see them.

Given:
- A 2D array `flowers` where each `flowers[i] = [start_i, end_i]` represents that the i-th flower blooms from day `start_i` to day `end_i` (both days inclusive)
- An array `people` where `people[i]` represents the day when the i-th person arrives

You need to find out how many flowers are in full bloom when each person arrives.

For example, if a flower blooms from day 3 to day 7, and a person arrives on day 5, that flower would be counted as blooming for that person. If another flower blooms from day 1 to day 4, it would not be counted for this person since it has already wilted by day 5.

The solution uses a clever approach:
1. Create two sorted lists: one with all bloom start times and one with all bloom end times
2. For each person arriving at time `p`:
   - Count how many flowers have started blooming by time `p` using `bisect_right(start, p)`
   - Count how many flowers have already finished blooming before time `p` using `bisect_left(end, p)`
   - The difference gives the number of flowers currently in bloom

## Intuition

Think about what it means for a flower to be blooming at a specific time. A flower is blooming at time `t` if:
- It has already started blooming (its start time ≤ `t`)
- It hasn't finished blooming yet (its end time ≥ `t`)

This naturally leads us to think about counting flowers in two groups:
1. Flowers that have started by time `t`
2. Flowers that have ended before time `t`

The number of flowers blooming at time `t` = (flowers that have started) - (flowers that have already ended)

Instead of checking each flower individually for each person (which would be inefficient), we can preprocess the data. By separating all start times and end times into two sorted arrays, we can quickly answer "how many flowers have started/ended by time `t`" using [binary search](/problems/binary_search_intro).

The trick with using `bisect_right` for starts and `bisect_left` for ends handles the boundary cases correctly:
- `bisect_right(start, p)` counts all flowers where `start_time <= p`
- `bisect_left(end, p)` counts all flowers where `end_time < p`

This is important because a flower that ends on day `p` should still be counted as blooming on day `p` (since the end time is inclusive). By using `bisect_left` for end times, we only exclude flowers that ended strictly before day `p`.

This transforms a potentially complex interval overlap problem into a simple counting problem using sorted arrays and [binary search](/problems/binary_search_intro), achieving an efficient solution.

**Learn more about [Binary Search](/problems/binary_search_intro), [Prefix Sum](/problems/subarray_sum) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The solution uses [sorting](/problems/sorting_summary) and [binary search](/problems/binary_search_intro) to count blooming flowers.

**Step 1: Extract and Sort Start/End Times**

```python
bloom_starts = sorted(start for start, _ in flowers)
bloom_ends = sorted(end for _, end in flowers)
```

**Step 2: Binary Search Using first_true_index Template**

For each person arriving at time `p`, we count:
- Flowers started: count of flowers with `start <= p`
- Flowers ended: count of flowers with `end < p`

Using the first_true_index template to implement `bisect_left`:

```python
def bisect_left(nums, target):
    # Find first index where nums[idx] >= target
    left, right = 0, len(nums) - 1
    first_true_index = len(nums)  # Default: no element >= target

    while left <= right:
        mid = (left + right) // 2
        if nums[mid] >= target:
            first_true_index = mid
            right = mid - 1
        else:
            left = mid + 1

    return first_true_index
```

**Counting Logic:**
- `flowers_started = bisect_left(starts, p + 1)` counts flowers with `start <= p`
- `flowers_ended = bisect_left(ends, p)` counts flowers with `end < p`
- `blooming = flowers_started - flowers_ended`

**Why `p + 1` for starts?**
`bisect_left(starts, p + 1)` finds the first index where `start >= p + 1`, which equals the count of starts where `start <= p`.

**Time Complexity:** `O((m + n) log m)` where `m` = flowers, `n` = people.

**Space Complexity:** `O(m)` for sorted arrays.

## Example Walkthrough

Let's walk through a concrete example to understand how the solution works.

**Given:**
- `flowers = [[1,6], [3,7], [9,12], [4,13]]`
- `people = [2, 3, 7, 11]`

**Step 1: Extract and sort start/end times**

From the flowers array, we extract:
- Start times: `[1, 3, 9, 4]` → Sorted: `[1, 3, 4, 9]`
- End times: `[6, 7, 12, 13]` → Sorted: `[6, 7, 12, 13]`

**Step 2: Process each person**

For **person at day 2**:
- `bisect_right(start=[1,3,4,9], p=2)` = 1
  - Only flower starting at day 1 has begun (1 ≤ 2)
- `bisect_left(end=[6,7,12,13], p=2)` = 0
  - No flowers have ended yet (no end times < 2)
- Blooming flowers: 1 - 0 = **1**

For **person at day 3**:
- `bisect_right(start=[1,3,4,9], p=3)` = 2
  - Flowers starting at days 1 and 3 have begun (both ≤ 3)
- `bisect_left(end=[6,7,12,13], p=3)` = 0
  - No flowers have ended yet (no end times < 3)
- Blooming flowers: 2 - 0 = **2**

For **person at day 7**:
- `bisect_right(start=[1,3,4,9], p=7)` = 3
  - Flowers starting at days 1, 3, and 4 have begun (all ≤ 7)
- `bisect_left(end=[6,7,12,13], p=7)` = 1
  - Only the flower ending at day 6 has finished (6 < 7)
  - Note: The flower ending at day 7 is still blooming!
- Blooming flowers: 3 - 1 = **2**

For **person at day 11**:
- `bisect_right(start=[1,3,4,9], p=11)` = 4
  - All flowers have started (all start times ≤ 11)
- `bisect_left(end=[6,7,12,13], p=11)` = 2
  - Flowers ending at days 6 and 7 have finished (both < 11)
- Blooming flowers: 4 - 2 = **2**

**Result:** `[1, 2, 2, 2]`

The key insight is how `bisect_left` for end times ensures flowers ending on the same day as a person's arrival are still counted as blooming, while `bisect_right` for start times includes flowers that start on the arrival day.

## Python Solution

```python
from typing import List
from bisect import bisect_right, bisect_left

class Solution:
    def fullBloomFlowers(
        self, flowers: List[List[int]], people: List[int]
    ) -> List[int]:
        """
        Calculate the number of flowers in bloom for each person's arrival time.

        Args:
            flowers: List of [start_time, end_time] representing bloom periods
            people: List of arrival times for each person

        Returns:
            List of flower counts for each person
        """
        # Extract and sort all bloom start times
        bloom_starts = sorted(start_time for start_time, _ in flowers)

        # Extract and sort all bloom end times
        bloom_ends = sorted(end_time for _, end_time in flowers)

        # For each person, calculate flowers in bloom at their arrival time
        result = []
        for arrival_time in people:
            # Count flowers that have started blooming by this time
            flowers_started = bisect_right(bloom_starts, arrival_time)

            # Count flowers that have already finished blooming before this time
            flowers_ended = bisect_left(bloom_ends, arrival_time)

            # The difference gives us flowers currently in bloom
            flowers_in_bloom = flowers_started - flowers_ended
            result.append(flowers_in_bloom)

        return result
```

## Time and Space Complexity

**Time Complexity:** `O((m + n) × log n)`

The time complexity breaks down as follows:
- Sorting the start times: `O(n log n)` where `n` is the number of flowers
- Sorting the end times: `O(n log n)` where `n` is the number of flowers
- For each person in `people`, we perform:
  - `bisect_right(start, p)`: `O(log n)` binary search
  - `bisect_left(end, p)`: `O(log n)` binary search
  - Since there are `m` people, this step takes `O(m × log n)`
- Total: `O(n log n + n log n + m × log n)` = `O((m + n) × log n)`

**Space Complexity:** `O(n)`

The space complexity consists of:
- `start` array: `O(n)` space to store all flower start times
- `end` array: `O(n)` space to store all flower end times
- The result list uses `O(m)` space but is not counted as auxiliary space since it's the output
- Total auxiliary space: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template**

The problem uses `bisect_left` which finds the first index where element >= target.

**Wrong approach:**
```python
while left < right:
    mid = (left + right) >> 1
    if nums[mid] >= target:
        right = mid
    else:
        left = mid + 1
return left  # Returns left directly
```

**Correct approach (first_true_index template):**
```python
first_true_index = len(nums)  # Default
while left <= right:
    mid = (left + right) // 2
    if nums[mid] >= target:
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
return first_true_index
```

### 2. **Confusing bisect_left vs bisect_right Logic**

To count flowers with `start <= p`, use `bisect_left(starts, p + 1)` (not `bisect_right`).
To count flowers with `end < p`, use `bisect_left(ends, p)`.

The key insight:
- `bisect_left(arr, x)` returns count of elements < x
- `bisect_left(arr, x + 1)` returns count of elements <= x

### 3. **Not Handling Inclusive Bloom Periods**

Bloom periods are inclusive: a flower blooming [3, 5] is still blooming on day 5.

The formula `bisect_left(ends, p)` correctly counts only flowers with `end < p`, so flowers ending exactly on arrival day are still counted as blooming.

### 4. **Incorrect Default for first_true_index**

When no element >= target exists, `first_true_index` should default to `len(nums)` to correctly return "all elements are less than target".

**Wrong:** `first_true_index = -1`
**Correct:** `first_true_index = len(nums)`

