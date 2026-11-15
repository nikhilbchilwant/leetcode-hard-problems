# 1755. Closest Subsequence Sum

> Source: https://algo.monster/liteproblems/1755

## Problem Statement

You are given an integer array `nums` and an integer `goal`. Your task is to find a subsequence of `nums` whose sum is as close as possible to the `goal` value.

A subsequence is formed by selecting some elements from the original array while maintaining their relative order. You can choose any combination of elements, including selecting all elements, no elements, or any subset in between.

The objective is to minimize the absolute difference between the sum of your chosen subsequence and the goal. In other words, if your subsequence has a sum of `sum`, you want to minimize `abs(sum - goal)`.

For example:
- If `nums = [1, 2, 3]` and `goal = 5`, you could choose subsequence `[2, 3]` with sum 5, giving an absolute difference of `abs(5 - 5) = 0`
- If `nums = [7, -9, 15, -2]` and `goal = -5`, you might choose subsequence `[-9, -2]` with sum -11, or just `[-2]` with sum -2, comparing which gives the smaller absolute difference to -5

The function should return the minimum possible value of `abs(sum - goal)` that can be achieved by any valid subsequence of the input array.

## Intuition

The naive approach would be to generate all possible subsequences and check which one gives the minimum absolute difference with the goal. However, with `n` elements, we'd have `2^n` possible subsequences, which becomes computationally infeasible for larger arrays.

The key insight is to use a "meet in the middle" strategy. Instead of generating all `2^n` subsequences at once, we can split the array into two halves and generate subsequences for each half separately. This reduces our problem from `2^n` to `2^(n/2) + 2^(n/2) = 2 * 2^(n/2)` operations, which is a significant improvement.

Here's how we arrive at this approach:

1. **Divide the array**: Split `nums` into two roughly equal halves - left half and right half.

2. **Generate all possible sums**: For each half, generate all possible subsequence sums. The left half gives us `2^(n/2)` possible sums, and similarly for the right half.

3. **The clever part**: Any subsequence of the original array can be represented as a combination of one subsequence from the left half and one from the right half. So if we pick a sum `l` from the left half, we need to find the best matching sum `r` from the right half such that `l + r` is closest to `goal`.

4. **Optimization with binary search**: For each sum `l` from the left half, we need `remaining = goal - l` from the right half. To find the closest value to `remaining` in the right half sums, we can sort the right half sums and use binary search. This allows us to quickly find the two candidates: the smallest value greater than or equal to `remaining`, and the largest value less than `remaining`.

5. **Track the minimum**: For each combination, calculate `abs((l + r) - goal)` and keep track of the minimum difference found.

This approach cleverly reduces an exponential problem to a more manageable complexity of `O(2^(n/2) * log(2^(n/2)))`, making it feasible to solve even for moderately sized arrays.

**Learn more about [Two Pointers](/problems/two_pointers_intro), [Dynamic Programming](/problems/dynamic_programming_intro), [Bitmask](/problems/bitmask_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows the "meet in the middle" strategy discussed earlier. Let's walk through the code step by step:

**1. Split the Array**
```python
n = len(nums)
left = set()
right = set()

self.getSubSeqSum(0, 0, nums[: n // 2], left)
self.getSubSeqSum(0, 0, nums[n // 2 :], right)
```
- The array is divided into two halves: `nums[:n//2]` and `nums[n//2:]`
- Two sets (`left` and `right`) are used to store all possible subsequence sums from each half
- Sets are used to automatically handle duplicate sums

**2. Generate All Subsequence Sums**
```python
def getSubSeqSum(self, i: int, curr: int, arr: List[int], result: Set[int]):
    if i == len(arr):
        result.add(curr)
        return
  
    self.getSubSeqSum(i + 1, curr, arr, result)
    self.getSubSeqSum(i + 1, curr + arr[i], arr, result)
```
- This recursive function generates all possible subsequence sums
- For each element at index `i`, we have two choices:
  - Don't include it: `getSubSeqSum(i + 1, curr, arr, result)`
  - Include it: `getSubSeqSum(i + 1, curr + arr[i], arr, result)`
- When we reach the end of the array (`i == len(arr)`), we add the current sum to the result set
- This generates `2^(n/2)` sums for each half

**3. Find the Closest Sum Using Binary Search**
```python
result = inf
right = sorted(right)
rl = len(right)

for l in left:
    remaining = goal - l
    idx = bisect_left(right, remaining)
  
    if idx < rl:
        result = min(result, abs(remaining - right[idx]))
  
    if idx > 0:
        result = min(result, abs(remaining - right[idx - 1]))
```
- Sort the right set to enable binary search
- For each sum `l` from the left half:
  - Calculate `remaining = goal - l` (what we need from the right half)
  - Use `bisect_left` to find the insertion point for `remaining` in the sorted right array
  - Check two candidates:
    - `right[idx]`: The smallest value ≥ `remaining` (if it exists)
    - `right[idx - 1]`: The largest value < `remaining` (if it exists)
  - Update the result with the minimum absolute difference found

**Time Complexity Analysis:**
- Generating subsequence sums: `O(2^(n/2))` for each half
- [Sorting](/problems/sorting_summary) the right array: `O(2^(n/2) * log(2^(n/2))) = O(n * 2^(n/2))`
- Binary search for each left sum: `O(2^(n/2) * log(2^(n/2)))`
- Overall: `O(n * 2^(n/2))`

**Space Complexity:** `O(2^(n/2))` to store all the subsequence sums

This approach effectively reduces the problem from `O(2^n)` to `O(n * 2^(n/2))`, making it feasible for arrays with up to approximately 40 elements.

## Example Walkthrough

Let's walk through the solution with a concrete example:

**Input:** `nums = [7, -9, 15, -2]`, `goal = -5`

**Step 1: Split the array into two halves**
- Left half: `[7, -9]`
- Right half: `[15, -2]`

**Step 2: Generate all subsequence sums for each half**

For the left half `[7, -9]`:
- Choose neither: sum = `0`
- Choose only `7`: sum = `7`
- Choose only `-9`: sum = `-9`
- Choose both `7` and `-9`: sum = `7 + (-9) = -2`
- Left sums: `{0, 7, -9, -2}`

For the right half `[15, -2]`:
- Choose neither: sum = `0`
- Choose only `15`: sum = `15`
- Choose only `-2`: sum = `-2`
- Choose both `15` and `-2`: sum = `15 + (-2) = 13`
- Right sums: `{0, 15, -2, 13}`

**Step 3: Sort the right sums for binary search**
- Sorted right sums: `[-2, 0, 13, 15]`

**Step 4: For each left sum, find the best matching right sum**

For `l = 0` from left:
- Need `remaining = goal - l = -5 - 0 = -5` from right
- Binary search for `-5` in `[-2, 0, 13, 15]` gives insertion index 0
- Check candidates:
  - `right[0] = -2`: difference = `|(-5) - (-2)| = 3`
- Total sum = `0 + (-2) = -2`, distance from goal = `|(-2) - (-5)| = 3`

For `l = 7` from left:
- Need `remaining = -5 - 7 = -12` from right
- Binary search for `-12` in `[-2, 0, 13, 15]` gives insertion index 0
- Check candidates:
  - `right[0] = -2`: difference = `|(-12) - (-2)| = 10`
- Total sum = `7 + (-2) = 5`, distance from goal = `|5 - (-5)| = 10`

For `l = -9` from left:
- Need `remaining = -5 - (-9) = 4` from right
- Binary search for `4` in `[-2, 0, 13, 15]` gives insertion index 2
- Check candidates:
  - `right[2] = 13`: difference = `|4 - 13| = 9`
  - `right[1] = 0`: difference = `|4 - 0| = 4`
- Best match is with `0`: Total sum = `-9 + 0 = -9`, distance from goal = `|(-9) - (-5)| = 4`

For `l = -2` from left:
- Need `remaining = -5 - (-2) = -3` from right
- Binary search for `-3` in `[-2, 0, 13, 15]` gives insertion index 0
- Check candidates:
  - `right[0] = -2`: difference = `|(-3) - (-2)| = 1`
- Total sum = `-2 + (-2) = -4`, distance from goal = `|(-4) - (-5)| = 1`

**Step 5: Return the minimum difference**
The minimum difference found is `1`, achieved by selecting the subsequence `[-2, -2]` (both `-2` values from the original array) with sum `-4`.

## Python Solution

```python
class Solution:
    def minAbsDifference(self, nums: List[int], goal: int) -> int:
        """
        Find the minimum absolute difference between the sum of any subsequence and the goal.
        Uses meet-in-the-middle technique to reduce time complexity from O(2^n) to O(2^(n/2) * n).
        """
        n = len(nums)
      
        # Split array into two halves for meet-in-the-middle approach
        left_sums = set()
        right_sums = set()
      
        # Generate all possible subsequence sums for each half
        self.getSubSeqSum(0, 0, nums[:n // 2], left_sums)
        self.getSubSeqSum(0, 0, nums[n // 2:], right_sums)
      
        # Sort right sums for binary search
        sorted_right_sums = sorted(right_sums)
        right_length = len(sorted_right_sums)
      
        min_difference = float('inf')
      
        # For each sum from left half, find the best matching sum from right half
        for left_sum in left_sums:
            # We need to find right_sum such that left_sum + right_sum is closest to goal
            target = goal - left_sum
          
            # Binary search for the closest value to target in sorted_right_sums
            insertion_point = bisect_left(sorted_right_sums, target)
          
            # Check the value at or after insertion point (ceiling value)
            if insertion_point < right_length:
                min_difference = min(min_difference, abs(target - sorted_right_sums[insertion_point]))
          
            # Check the value before insertion point (floor value)
            if insertion_point > 0:
                min_difference = min(min_difference, abs(target - sorted_right_sums[insertion_point - 1]))
      
        return min_difference
  
    def getSubSeqSum(self, index: int, current_sum: int, array: List[int], result_set: Set[int]) -> None:
        """
        Recursively generate all possible subsequence sums.
      
        Args:
            index: Current position in the array
            current_sum: Sum accumulated so far
            array: The array to generate subsequences from
            result_set: Set to store all possible sums
        """
        # Base case: reached end of array
        if index == len(array):
            result_set.add(current_sum)
            return
      
        # Option 1: Exclude current element
        self.getSubSeqSum(index + 1, current_sum, array, result_set)
      
        # Option 2: Include current element
        self.getSubSeqSum(index + 1, current_sum + array[index], array, result_set)
```

## Time and Space Complexity

**Time Complexity:** `O(2^(n/2) * log(2^(n/2)))` which simplifies to `O(2^(n/2) * n)`

The algorithm uses a meet-in-the-middle approach:
- `getSubSeqSum` generates all possible subset sums for each half of the array. For an array of size `n/2`, this creates `2^(n/2)` subsets, taking `O(2^(n/2))` time for each half.
- Sorting the right set takes `O(2^(n/2) * log(2^(n/2)))` = `O(2^(n/2) * n/2)`.
- The final loop iterates through all `2^(n/2)` elements in the left set, and for each element performs a binary search on the sorted right set, which takes `O(log(2^(n/2)))` = `O(n/2)` time.
- Total: `O(2^(n/2))` + `O(2^(n/2))` + `O(2^(n/2) * n/2)` + `O(2^(n/2) * n/2)` = `O(2^(n/2) * n)`

**Space Complexity:** `O(2^(n/2))`

- Both `left` and `right` sets store all possible subset sums from their respective halves of the array.
- Each set can contain at most `2^(n/2)` elements.
- The recursion stack for `getSubSeqSum` goes to depth `n/2`, contributing `O(n/2)` space.
- The sorted list created from the right set also takes `O(2^(n/2))` space.
- Total: `O(2^(n/2))` + `O(2^(n/2))` + `O(n/2)` + `O(2^(n/2))` = `O(2^(n/2))`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow with Large Array Values**
One critical pitfall is not considering integer overflow when array elements or the goal value are very large. In Python, this is handled automatically, but in languages like Java or C++, the sum calculations could overflow.

**Problem Example:**
```python
nums = [10**9, 10**9, 10**9, 10**9]  # Large values
goal = 4 * 10**9
```

**Solution:**
- In Python: No action needed as Python handles arbitrary precision integers
- In Java/C++: Use `long` type instead of `int` for sums and calculations
- Consider using modular arithmetic or careful bounds checking if needed

### 2. **Empty Array Edge Case**
The code doesn't explicitly handle the case when `nums` is empty, which could lead to unexpected behavior.

**Problem Example:**
```python
nums = []
goal = 5
# The code would still work but it's not immediately clear
```

**Solution:**
```python
def minAbsDifference(self, nums: List[int], goal: int) -> int:
    if not nums:
        return abs(goal)  # Empty subsequence has sum 0
  
    # ... rest of the code
```

### 3. **Memory Limit Exceeded for Large Arrays**
When the array size approaches 40, each half generates 2^20 ≈ 1 million subsequence sums. This could cause memory issues.

**Problem Example:**
```python
nums = [1] * 40  # Will generate 2^20 sums for each half
```

**Solution:**
Implement iterative generation with pruning:
```python
def getSubSeqSumIterative(self, arr: List[int]) -> Set[int]:
    sums = {0}
    for num in arr:
        new_sums = set()
        for s in sums:
            new_sum = s + num
            # Optional: Add pruning logic here if possible
            new_sums.add(new_sum)
        sums.update(new_sums)
    return sums
```

### 4. **Duplicate Values Not Being Used**
While using sets eliminates duplicate sums, arrays with many duplicate values still go through all 2^(n/2) recursive calls unnecessarily.

**Problem Example:**
```python
nums = [5, 5, 5, 5, 5, 5]  # Many duplicates
# Still makes 2^3 = 8 recursive calls per half even though many produce same sums
```

**Solution:**
Pre-process to count duplicates and generate sums more efficiently:
```python
def optimizedSubSeqSum(self, arr: List[int]) -> Set[int]:
    from collections import Counter
    counts = Counter(arr)
    sums = {0}
  
    for value, count in counts.items():
        new_sums = set()
        for existing_sum in sums:
            # Add all possible counts of this value (0 to count)
            for i in range(count + 1):
                new_sums.add(existing_sum + value * i)
        sums = new_sums
    return sums
```

### 5. **Not Handling the Case When Goal is Achievable**
The algorithm doesn't short-circuit when it finds an exact match (difference = 0), continuing to check all combinations unnecessarily.

**Solution:**
Add early termination:
```python
for left_sum in left_sums:
    target = goal - left_sum
  
    # Early termination if exact match found
    if target in sorted_right_sums:
        return 0
  
    insertion_point = bisect_left(sorted_right_sums, target)
    # ... rest of the binary search logic
```

