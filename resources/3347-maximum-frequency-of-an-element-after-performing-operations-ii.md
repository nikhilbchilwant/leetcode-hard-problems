# 3347. Maximum Frequency of an Element After Performing Operations II

> Source: https://algo.monster/liteproblems/3347

## Problem Statement

You have an integer array `nums` and two integers `k` and `numOperations`.

You need to perform exactly `numOperations` operations on the array. In each operation:
- Choose an index `i` that hasn't been selected in any previous operation (each index can only be selected once)
- Add any integer value between `-k` and `k` (inclusive) to `nums[i]`

After performing all operations, you want to maximize the frequency of any element in the array. The frequency of an element is the number of times it appears in the array.

Your task is to return the maximum possible frequency of any single element value that can be achieved after performing all the operations optimally.

For example, if you have `nums = [1, 4, 5]`, `k = 2`, and `numOperations = 2`:
- You could change `nums[0]` from 1 to 3 (by adding 2)
- You could change `nums[2]` from 5 to 3 (by adding -2)
- This would give you `[3, 4, 3]` with a maximum frequency of 2 (the value 3 appears twice)

## Intuition

The key observation is that each element `nums[i]` can be transformed to any value in the range `[nums[i] - k, nums[i] + k]` if we choose to apply an operation on it. This means each element has a "reachable range" of values it can become.

To maximize the frequency of any value, we need to find a target value `x` where the most elements can be transformed to reach `x`. But we're limited by `numOperations` - we can only transform at most `numOperations` elements.

For any potential target value `x`, we can count:
1. How many elements are already equal to `x` (these don't need operations)
2. How many elements can reach `x` through their range `[nums[i] - k, nums[i] + k]`

The maximum frequency we can achieve for value `x` is the minimum of:
- The total number of elements that can reach `x` (through their ranges)
- The number of elements already at `x` plus `numOperations` (since we can only transform `numOperations` elements)

To efficiently compute this, we use a sweep line technique. For each element `nums[i]`:
- It contributes to the "reachability count" for all values in range `[nums[i] - k, nums[i] + k]`
- We mark the start of this range with `+1` and the end+1 with `-1`

By [sorting](/problems/sorting_summary) these events and sweeping through them, we maintain a running sum `s` that tells us how many elements can reach the current value. At each potential target value, we check if it can achieve a better maximum frequency than what we've seen so far.

The formula `min(s, cnt[x] + numOperations)` captures the constraint: we can either transform all reachable elements to `x` (if we have enough operations), or we keep the existing `cnt[x]` elements and use our `numOperations` to transform additional elements.

**Learn more about [Binary Search](/problems/binary_search_intro), [Prefix Sum](/problems/subarray_sum), [Sorting](/problems/sorting_summary) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

The solution uses a sweep line algorithm with difference arrays to efficiently compute the maximum frequency.

**Step 1: Initialize Data Structures**
- `cnt`: A dictionary to store the original frequency of each element in `nums`
- `d`: A difference array (as a dictionary) to mark range boundaries

**Step 2: Process Each Element**
For each element `x` in `nums`:
- Increment `cnt[x]` to track its original frequency
- Initialize `d[x]` to ensure `x` is in our sweep line events
- Mark the range `[x - k, x + k]` where this element can contribute:
  - `d[x - k] += 1`: Start of the reachable range
  - `d[x + k + 1] -= 1`: End of the reachable range (exclusive)

**Step 3: Sweep Line Processing**
- Sort all the keys in `d` to process events in order
- Maintain a running sum `s` that represents how many elements can reach the current value
- For each value `x` and its difference value `t`:
  - Update the running sum: `s += t`
  - Calculate the maximum frequency possible at value `x`:
    - `s` represents total elements that can reach `x`
    - `cnt[x] + numOperations` represents keeping original `x` values and transforming others
    - Take the minimum of these two values
  - Update the global maximum: `ans = max(ans, min(s, cnt[x] + numOperations))`

**Why This Works:**
- The difference array technique allows us to efficiently track overlapping ranges
- At any point `x`, the value `s` tells us exactly how many elements from the original array can be transformed to reach `x`
- The constraint `min(s, cnt[x] + numOperations)` ensures we don't exceed our operation limit
- By checking all potential target values (all event points in our sweep), we're guaranteed to find the optimal solution

**Time Complexity:** `O(n log n)` due to [sorting](/problems/sorting_summary), where `n` is the length of `nums`
**Space Complexity:** `O(n)` for the dictionaries storing counts and events

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Example:** `nums = [1, 4, 5]`, `k = 2`, `numOperations = 2`

**Step 1: Initialize Data Structures**
- `cnt = {}` (will store original frequencies)
- `d = {}` (difference array for sweep line)

**Step 2: Process Each Element**

Processing `nums[0] = 1`:
- `cnt[1] = 1` (frequency of 1 is now 1)
- `d[1] = 0` (initialize)
- Range where 1 can reach: `[1-2, 1+2] = [-1, 3]`
- `d[-1] += 1` → `d[-1] = 1` (start of range)
- `d[4] -= 1` → `d[4] = -1` (end+1 of range)

Processing `nums[1] = 4`:
- `cnt[4] = 1` (frequency of 4 is now 1)
- `d[4] = -1` (already exists from previous step)
- Range where 4 can reach: `[4-2, 4+2] = [2, 6]`
- `d[2] += 1` → `d[2] = 1` (start of range)
- `d[7] -= 1` → `d[7] = -1` (end+1 of range)

Processing `nums[2] = 5`:
- `cnt[5] = 1` (frequency of 5 is now 1)
- `d[5] = 0` (initialize)
- Range where 5 can reach: `[5-2, 5+2] = [3, 7]`
- `d[3] += 1` → `d[3] = 1` (start of range)
- `d[8] -= 1` → `d[8] = -1` (end+1 of range)

**Current State:**
- `cnt = {1: 1, 4: 1, 5: 1}`
- `d = {-1: 1, 1: 0, 2: 1, 3: 1, 4: -1, 5: 0, 7: -1, 8: -1}`

**Step 3: Sweep Line Processing**

Sort keys of `d`: `[-1, 1, 2, 3, 4, 5, 7, 8]`

Initialize: `s = 0`, `ans = 0`

| Position | Value in d | s (after update) | cnt.get(pos, 0) | min(s, cnt[pos] + numOps) | ans |
|----------|-----------|------------------|-----------------|---------------------------|-----|
| -1       | 1         | 1                | 0               | min(1, 0+2) = 1          | 1   |
| 1        | 0         | 1                | 1               | min(1, 1+2) = 1          | 1   |
| 2        | 1         | 2                | 0               | min(2, 0+2) = 2          | 2   |
| 3        | 1         | 3                | 0               | min(3, 0+2) = 2          | 2   |
| 4        | -1        | 2                | 1               | min(2, 1+2) = 2          | 2   |
| 5        | 0         | 2                | 1               | min(2, 1+2) = 2          | 2   |
| 7        | -1        | 1                | 0               | min(1, 0+2) = 1          | 2   |
| 8        | -1        | 0                | 0               | min(0, 0+2) = 0          | 2   |

**Result:** Maximum frequency = 2

**Verification:**
At position 3, we have `s = 3`, meaning all three elements can reach value 3:
- Element 1 can reach 3 (by adding +2)
- Element 4 can reach 3 (by adding -1)
- Element 5 can reach 3 (by adding -2)

Since we only have 2 operations, we can transform 2 elements to value 3, achieving frequency 2. For instance, transform 1→3 and 5→3, resulting in array `[3, 4, 3]`.

## Python Solution

```python
from collections import defaultdict
from typing import List

class Solution:
    def maxFrequency(self, nums: List[int], k: int, numOperations: int) -> int:
        # Count frequency of each number in the original array
        frequency_count = defaultdict(int)

        # Difference array to track range contributions
        # For each number x, we can change it to any value in [x-k, x+k]
        difference_array = defaultdict(int)

        for num in nums:
            # Count original frequency
            frequency_count[num] += 1

            # Initialize difference array entry for this number
            difference_array[num] += 0

            # Mark the range [num - k, num + k] where this number can contribute
            # +1 at the start of the range
            difference_array[num - k] += 1
            # -1 after the end of the range
            difference_array[num + k + 1] -= 1

        max_frequency = 0
        cumulative_sum = 0

        # Process points in sorted order to calculate maximum possible frequency
        for value, delta in sorted(difference_array.items()):
            # Update cumulative sum (number of elements that can be changed to current value)
            cumulative_sum += delta

            # Maximum frequency at this value is:
            # - Original count plus operations (limited by numOperations)
            # - Cannot exceed total available elements that can reach this value
            max_frequency = max(
                max_frequency,
                min(cumulative_sum, frequency_count[value] + numOperations)
            )

        return max_frequency
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)` where `n` is the length of the input array `nums`.

- Iterating through `nums` to populate `cnt` and `d` dictionaries takes `O(n)` time
- Each element in `nums` creates at most 3 entries in dictionary `d` (at positions `x`, `x-k`, and `x+k+1`), so `d` has at most `O(n)` entries
- Sorting the items in dictionary `d` takes `O(n log n)` time since there are at most `O(n)` unique keys
- The final iteration through sorted items takes `O(n)` time
- Overall complexity is dominated by the sorting step: `O(n) + O(n log n) + O(n) = O(n log n)`

**Space Complexity:** `O(n)` where `n` is the length of the input array `nums`.

- Dictionary `cnt` stores at most `n` unique elements from `nums`, requiring `O(n)` space
- Dictionary `d` stores at most `3n` entries (3 entries per element in `nums`), which is `O(n)` space
- The sorted items list created from `d.items()` requires `O(n)` space
- Additional variables (`ans`, `s`, `x`, `t`) use `O(1)` space
- Total space complexity: `O(n) + O(n) + O(n) + O(1) = O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Initialize Target Values in the Difference Array**

A critical pitfall is forgetting to ensure that original values from `nums` are included as event points in the difference array, even if they don't contribute any delta change initially.

**Why this matters:** If an element `x` exists in `nums` but isn't explicitly added to the difference array, it won't be considered as a potential target value during the sweep line processing. This can lead to missing the optimal solution when the best strategy is to keep some original values unchanged while transforming others to match them.

**Incorrect approach:**
```python
# WRONG: Only adding range boundaries
for num in nums:
    frequency_count[num] += 1
    difference_array[num - k] += 1
    difference_array[num + k + 1] -= 1
```

**Correct approach:**
```python
# RIGHT: Explicitly initialize each num in difference_array
for num in nums:
    frequency_count[num] += 1
    difference_array[num] += 0  # Ensures num is a key in the dictionary
    difference_array[num - k] += 1
    difference_array[num + k + 1] -= 1
```

### 2. **Misunderstanding the Operation Constraint**

Another common mistake is incorrectly applying the `numOperations` constraint. The key insight is that we can perform **at most** `numOperations` changes, and each operation changes exactly one element.

**Incorrect interpretation:**
```python
# WRONG: Thinking we must use all operations
max_frequency = min(cumulative_sum, numOperations)
```

**Correct interpretation:**
```python
# RIGHT: Keep original values and add transformed ones (up to numOperations)
max_frequency = min(cumulative_sum, frequency_count[value] + numOperations)
```

The formula `frequency_count[value] + numOperations` represents:
- `frequency_count[value]`: Elements already at the target value (no operation needed)
- `numOperations`: Maximum additional elements we can transform to this value

### 3. **Off-by-One Error in Range Boundaries**

When marking ranges in the difference array, it's crucial to use `num + k + 1` (not `num + k`) for the end boundary to properly implement the exclusive end of the range `[num - k, num + k]`.

**Wrong:**
```python
difference_array[num + k] -= 1  # This would exclude num + k from the range
```

**Right:**
```python
difference_array[num + k + 1] -= 1  # Correctly includes num + k in the range
```

