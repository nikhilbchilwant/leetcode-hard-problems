# 1521. Find a Value of a Mysterious Function Closest to Target

> Source: https://algo.monster/liteproblems/1521

## Problem Statement

You are given a mysterious function `func` that takes an array and two indices `l` and `r` as parameters. The function returns the bitwise AND of all elements from index `l` to index `r` (inclusive) in the array. In other words, `func(arr, l, r) = arr[l] & arr[l+1] & ... & arr[r]`.

Given an integer array `arr` and an integer `target`, you need to find indices `l` and `r` such that the absolute difference `|func(arr, l, r) - target|` is minimized.

Return the minimum possible value of `|func(arr, l, r) - target|`.

**Constraints:**
- `0 <= l, r < arr.length`
- Both `l` and `r` must be valid indices in the array

**Example:**
If you have an array and a target value, you need to find a subarray (defined by indices `l` and `r`) where the bitwise AND of all elements in that subarray is as close as possible to the target value. The answer is the minimum absolute difference you can achieve.

## Intuition

The key observation is that the bitwise AND operation has a monotonic property: when we extend a subarray by including more elements, the result either stays the same or decreases (it never increases). This is because AND operation can only turn bits from 1 to 0, never from 0 to 1.

If we fix the right endpoint `r` and vary the left endpoint `l` from `r` down to `0`, the values of `func(arr, l, r)` form a decreasing sequence. Moreover, since each element in the array is at most `10^6` (which requires about 20 bits to represent), there can be at most 20 distinct values as we vary `l`. This is because each bit position can only change from 1 to 0 once as we extend the range leftward.

This suggests we can efficiently track all possible AND values ending at each position. For each position `r`, instead of computing `func(arr, l, r)` for all possible `l` values (which would be O(n²)), we can maintain a set of all distinct AND values that can be obtained with subarrays ending at position `r`.

When we move from position `r` to `r+1`, we can update our set of possible values by:
1. Taking each value in the current set and performing AND with `arr[r+1]` (extending previous subarrays)
2. Adding `arr[r+1]` itself (starting a new subarray at position `r+1`)

Since the set contains at most 20 values at any point (due to the bit limit), this update process is very efficient. For each position, we check all values in our set against the target to find the minimum absolute difference.

This approach cleverly exploits the properties of bitwise AND and the bounded nature of the values to reduce what could be an O(n²) problem to O(n × log M) where M is the maximum value in the array.

**Learn more about [Segment Tree](/problems/segment_tree_intro) and [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The implementation uses a hash table (set) to maintain all possible bitwise AND values ending at the current position.

**Algorithm Steps:**

1. **Initialize**: Start with the answer as `abs(arr[0] - target)` and create a set `s` containing just `arr[0]`.

2. **Iterate through the array**: For each element `x` in `arr`:
   - **Update the set**: Create a new set containing:
     - All values from the previous set ANDed with `x`: `{x & y for y in s}`
     - The current element itself: `{x}`
     - This is done with the expression: `s = {x & y for y in s} | {x}`

3. **Update the minimum**: For each value `y` in the updated set `s`, calculate `abs(y - target)` and update the answer with the minimum value found.

**Implementation Details:**

```python
class Solution:
    def closestToTarget(self, arr: List[int], target: int) -> int:
        ans = abs(arr[0] - target)  # Initialize with first element
        s = {arr[0]}  # Set to store all possible AND values ending at current position

        for x in arr:
            # Update set: AND all previous values with x, and add x itself
            s = {x & y for y in s} | {x}
            # Find minimum difference with target among all values in set
            ans = min(ans, min(abs(y - target) for y in s))

        return ans
```

**Why this works:**

- The set `s` at each iteration contains all possible values of `func(arr, l, i)` where `i` is the current index and `l` ranges from `0` to `i`.
- When we process element `arr[i]`, we extend all subarrays ending at `i-1` by including `arr[i]` (hence `x & y for y in s`).
- We also start a new subarray at position `i` (hence adding `{x}` to the set).
- The set automatically handles duplicates, keeping only unique values.
- Since the set size is bounded by approximately 20 (the number of bits), each iteration is efficient.

**Time Complexity**: `O(n × log M)` where `n` is the array length and `M` is the maximum value in the array. The `log M` factor comes from the maximum number of distinct AND values possible.

**Space Complexity**: `O(log M)` for storing the set of distinct AND values.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example:** `arr = [9, 12, 3, 7, 15]`, `target = 5`

We need to find a subarray where the bitwise AND of all elements is closest to 5.

**Initial State:**
- `ans = |9 - 5| = 4`
- `s = {9}` (only considering subarray [9])

**Iteration 1: Process arr[1] = 12**
- Calculate new set: `s = {12 & 9} | {12} = {8} | {12} = {8, 12}`
  - `12 & 9 = 1100 & 1001 = 1000 = 8` (extending subarray [9] to [9,12])
  - `12` itself (new subarray starting at index 1)
- Update answer:
  - `|8 - 5| = 3`
  - `|12 - 5| = 7`
  - `ans = min(4, 3) = 3`

**Iteration 2: Process arr[2] = 3**
- Calculate new set: `s = {3 & 8, 3 & 12} | {3} = {0, 0} | {3} = {0, 3}`
  - `3 & 8 = 0011 & 1000 = 0000 = 0` (extending [9,12] to [9,12,3])
  - `3 & 12 = 0011 & 1100 = 0000 = 0` (extending [12] to [12,3])
  - `3` itself (new subarray [3])
- Update answer:
  - `|0 - 5| = 5`
  - `|3 - 5| = 2`
  - `ans = min(3, 2) = 2`

**Iteration 3: Process arr[3] = 7**
- Calculate new set: `s = {7 & 0, 7 & 3} | {7} = {0, 3} | {7} = {0, 3, 7}`
  - `7 & 0 = 0` (extending subarrays ending with AND value 0)
  - `7 & 3 = 0111 & 0011 = 0011 = 3` (extending [3] to [3,7])
  - `7` itself (new subarray [7])
- Update answer:
  - `|0 - 5| = 5`
  - `|3 - 5| = 2`
  - `|7 - 5| = 2`
  - `ans = min(2, 2) = 2`

**Iteration 4: Process arr[4] = 15**
- Calculate new set: `s = {15 & 0, 15 & 3, 15 & 7} | {15} = {0, 3, 7} | {15} = {0, 3, 7, 15}`
  - `15 & 0 = 0`
  - `15 & 3 = 1111 & 0011 = 0011 = 3`
  - `15 & 7 = 1111 & 0111 = 0111 = 7` (extending [7] to [7,15])
  - `15` itself
- Update answer:
  - `|0 - 5| = 5`
  - `|3 - 5| = 2`
  - `|7 - 5| = 2`
  - `|15 - 5| = 10`
  - `ans = min(2, 2) = 2`

**Final Answer: 2**

The closest we can get to target 5 is either:
- Subarray [3] with AND value = 3 (difference = 2)
- Subarray [7] with AND value = 7 (difference = 2)
- Subarray [3,7] with AND value = 3 (difference = 2)

This example demonstrates how the set maintains all possible AND values ending at each position, efficiently tracking only distinct values (notice how duplicate 0s are automatically handled), and how we continuously update our minimum difference.

## Python Solution

```python
class Solution:
    def closestToTarget(self, arr: List[int], target: int) -> int:
        # Initialize the minimum difference with the first element
        min_difference = abs(arr[0] - target)

        # Set to store all possible AND values ending at current position
        current_and_values = {arr[0]}

        # Iterate through each element in the array
        for num in arr:
            # Create new set of AND values by:
            # 1. AND-ing current number with all previous AND values
            # 2. Including the current number itself as a new subsequence start
            current_and_values = {num & prev_value for prev_value in current_and_values} | {num}

            # Update minimum difference by checking all current AND values
            min_difference = min(min_difference, min(abs(and_value - target) for and_value in current_and_values))

        return min_difference
```

## Time and Space Complexity

**Time Complexity:** `O(n * log(max(arr)))`

The algorithm iterates through each element in the array once. For each element `x`, it performs bitwise AND operations with all elements in set `s`. The key insight is that the set `s` contains at most `O(log(max(arr)))` distinct values at any point.

This is because when we perform bitwise AND operations starting from any number, the result can only decrease or stay the same (never increase). Since bitwise AND can only turn bits from 1 to 0 (never from 0 to 1), and each number has at most `log(max(arr))` bits, there can be at most `O(log(max(arr)))` distinct values in the set after consecutive AND operations.

Therefore:
- Outer loop: `O(n)` iterations
- Inner set comprehension: `O(log(max(arr)))` operations per iteration
- Total: `O(n * log(max(arr)))`

**Space Complexity:** `O(log(max(arr)))`

The space is primarily used by the set `s`. As explained above, the set `s` can contain at most `O(log(max(arr)))` distinct values at any point during the execution, since we're tracking the results of cumulative bitwise AND operations which can only produce a limited number of distinct values based on the bit representation of the numbers.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Forgetting to Initialize with the First Element

**Issue**: A common mistake is to start with an empty set or not properly initialize the answer with the first element's difference from the target.

**Wrong Implementation:**
```python
def closestToTarget(self, arr: List[int], target: int) -> int:
    min_difference = float('inf')  # Wrong: might miss single-element subarrays
    current_and_values = set()     # Wrong: empty set loses first element

    for num in arr:
        current_and_values = {num & prev_value for prev_value in current_and_values} | {num}
        min_difference = min(min_difference, min(abs(and_value - target) for and_value in current_and_values))

    return min_difference
```

**Correct Approach:**
```python
def closestToTarget(self, arr: List[int], target: int) -> int:
    min_difference = abs(arr[0] - target)  # Initialize with first element
    current_and_values = {arr[0]}          # Start with first element in set

    for num in arr[1:]:  # Process remaining elements
        current_and_values = {num & prev_value for prev_value in current_and_values} | {num}
        min_difference = min(min_difference, min(abs(and_value - target) for and_value in current_and_values))

    return min_difference
```

### Pitfall 2: Not Including Single Elements as Valid Subarrays

**Issue**: Forgetting to add `{num}` when updating the set, which would miss all single-element subarrays starting at the current position.

**Wrong Implementation:**
```python
def closestToTarget(self, arr: List[int], target: int) -> int:
    min_difference = abs(arr[0] - target)
    current_and_values = {arr[0]}

    for num in arr:
        # Wrong: only extends previous subarrays, doesn't start new ones
        current_and_values = {num & prev_value for prev_value in current_and_values}
        min_difference = min(min_difference, min(abs(and_value - target) for and_value in current_and_values))

    return min_difference
```

**Correct Approach:**
```python
def closestToTarget(self, arr: List[int], target: int) -> int:
    min_difference = abs(arr[0] - target)
    current_and_values = {arr[0]}

    for num in arr:
        # Correct: includes both extended subarrays AND new subarray starting at current position
        current_and_values = {num & prev_value for prev_value in current_and_values} | {num}
        min_difference = min(min_difference, min(abs(and_value - target) for and_value in current_and_values))

    return min_difference
```

### Pitfall 3: Processing the First Element Twice

**Issue**: If you initialize the set with `arr[0]` and then iterate through the entire array starting from index 0, you'll process the first element twice, which could lead to incorrect results.

**Wrong Implementation:**
```python
def closestToTarget(self, arr: List[int], target: int) -> int:
    min_difference = abs(arr[0] - target)
    current_and_values = {arr[0]}

    # Wrong: starts from index 0, processing arr[0] again
    for num in arr:
        current_and_values = {num & prev_value for prev_value in current_and_values} | {num}
        min_difference = min(min_difference, min(abs(and_value - target) for and_value in current_and_values))

    return min_difference
```

**Correct Approaches:**

Option 1: Start iteration from index 1
```python
def closestToTarget(self, arr: List[int], target: int) -> int:
    min_difference = abs(arr[0] - target)
    current_and_values = {arr[0]}

    for num in arr[1:]:  # Start from index 1
        current_and_values = {num & prev_value for prev_value in current_and_values} | {num}
        min_difference = min(min_difference, min(abs(and_value - target) for and_value in current_and_values))

    return min_difference
```

Option 2: Don't initialize, process everything in loop
```python
def closestToTarget(self, arr: List[int], target: int) -> int:
    min_difference = float('inf')
    current_and_values = set()

    for num in arr:
        current_and_values = {num & prev_value for prev_value in current_and_values} | {num}
        min_difference = min(min_difference, min(abs(and_value - target) for and_value in current_and_values))

    return min_difference
```

Note: While the original solution processes the first element twice, it still works correctly because `arr[0] & arr[0] = arr[0]`, making it idempotent. However, it's cleaner to avoid the redundancy.

