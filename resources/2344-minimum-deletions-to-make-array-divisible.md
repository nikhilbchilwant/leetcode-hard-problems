# 2344. Minimum Deletions to Make Array Divisible

> Source: https://algo.monster/liteproblems/2344

## Problem Statement

You have two arrays of positive integers: `nums` and `numsDivide`.

Your task is to delete elements from `nums` so that the smallest remaining element in `nums` can divide every element in `numsDivide`. You want to minimize the number of deletions.

Here's what you need to find:
- Delete some (possibly zero) elements from `nums`
- After deletion, the smallest element in `nums` must divide all elements in `numsDivide`
- Return the minimum number of deletions needed
- If it's impossible (no element in `nums` can divide all elements in `numsDivide`), return `-1`

Remember that an integer `x` divides `y` if `y % x == 0` (remainder is zero).

For example:
- If `nums = [2, 3, 2, 4, 3]` and `numsDivide = [9, 6, 9, 3, 15]`
- The smallest element that can divide all elements in `numsDivide` is `3` (since `3` divides `9`, `6`, `9`, `3`, and `15`)
- We need to delete the two `2`s from `nums` to make `3` the smallest element
- So the answer is `2` deletions

## Intuition

The key insight is that if a number needs to divide all elements in `numsDivide`, it must divide their Greatest Common Divisor (GCD). This is because any common divisor of a set of numbers must also divide their GCD.

Think about it this way: if a number `x` divides both `a` and `b`, then `x` must also divide `gcd(a, b)`. Conversely, any divisor of `gcd(a, b)` will divide both `a` and `b`. This property extends to multiple numbers - any divisor of the GCD of all numbers will divide each individual number.

So instead of checking if each element in `nums` divides every single element in `numsDivide` (which would be inefficient), we can:
1. First find the GCD of all elements in `numsDivide`
2. Then find the smallest element in `nums` that divides this GCD

To minimize deletions, we want to keep the smallest valid element from `nums`. By [sorting](/problems/sorting_summary) `nums` first, we can iterate through elements from smallest to largest. The first element that divides the GCD is our answer - we delete everything before it.

For example, if `numsDivide = [6, 12, 18]`, their GCD is `6`. Any element from `nums` that we keep must divide `6` (possible values: `1`, `2`, `3`, or `6`). If `nums = [4, 3, 2, 6]`, after [sorting](/problems/sorting_summary) we get `[2, 3, 4, 6]`. The first element that divides `6` is `2`, so we delete `0` elements.

**Learn more about [Math](/problems/math-basics), [Sorting](/problems/sorting_summary) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation follows these steps:

1. **Calculate the GCD of `numsDivide`**: We start by finding the GCD of all elements in `numsDivide`. Initialize `x` with the first element, then iteratively compute the GCD with each subsequent element using `gcd(x, v)`. This gives us the number that any valid element from `nums` must divide.

2. **Sort the `nums` array**: By [sorting](/problems/sorting_summary) `nums` in ascending order, we ensure that we encounter potential candidates from smallest to largest. This is crucial because we want to minimize deletions, which means keeping the smallest valid element.

3. **Find the first valid divisor**: Iterate through the sorted `nums` array with both index `i` and value `v`. For each element, check if it divides the GCD using the condition `x % v == 0`. 
   - If we find such an element, return its index `i` - this represents the number of smaller elements we need to delete
   - The index directly gives us the deletion count because all elements before index `i` need to be removed

4. **Handle the impossible case**: If we iterate through the entire `nums` array without finding any element that divides the GCD, return `-1` to indicate it's impossible.

The algorithm efficiently reduces the problem from checking divisibility against all elements in `numsDivide` to checking against just their GCD. The time complexity is `O(n log n + m)` where `n` is the length of `nums` (for [sorting](/problems/sorting_summary)) and `m` is the length of `numsDivide` (for GCD calculation). The space complexity is `O(1)` if we sort in-place.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Example:** `nums = [4, 3, 6, 2, 3]` and `numsDivide = [12, 9, 6]`

**Step 1: Calculate GCD of numsDivide**
- Start with `x = 12` (first element)
- `x = gcd(12, 9) = 3`
- `x = gcd(3, 6) = 3`
- The GCD of all elements in `numsDivide` is `3`

This means any element we keep from `nums` must divide `3`.

**Step 2: Sort nums array**
- Original: `[4, 3, 6, 2, 3]`
- Sorted: `[2, 3, 3, 4, 6]`

**Step 3: Find first valid divisor**
Iterate through sorted `nums` with index and value:
- Index 0, value 2: Check if `3 % 2 == 0` → `1 ≠ 0` (doesn't divide, continue)
- Index 1, value 3: Check if `3 % 3 == 0` → `0 == 0` (divides! Found our answer)

**Result:** Return index `1`

This means we need to delete 1 element (the `2` at index 0) from the sorted array. After deletion, the smallest element would be `3`, which divides all elements in `numsDivide` since it divides their GCD.

**Verification:**
- `12 % 3 = 0` ✓
- `9 % 3 = 0` ✓  
- `6 % 3 = 0` ✓

The minimum number of deletions needed is **1**.

## Python Solution

```python
from typing import List
from math import gcd

class Solution:
    def minOperations(self, nums: List[int], numsDivide: List[int]) -> int:
        # Calculate the GCD of all elements in numsDivide array
        # This gives us the number that all elements in nums must divide
        target_gcd = numsDivide[0]
        for value in numsDivide[1:]:
            target_gcd = gcd(target_gcd, value)
      
        # Sort nums array to check smallest elements first
        nums.sort()
      
        # Find the first element in sorted nums that divides the GCD
        # The index represents minimum operations needed (removing all smaller elements)
        for index, value in enumerate(nums):
            if target_gcd % value == 0:
                return index
      
        # If no element in nums can divide the GCD, return -1
        return -1
```

## Time and Space Complexity

**Time Complexity:** `O(m + n log n)` where `n` is the length of `nums` and `m` is the length of `numsDivide`.

- Computing the GCD of all elements in `numsDivide` takes `O(m * log(min(numsDivide)))` time, where we iterate through `m` elements and each GCD operation takes `O(log(min(a,b)))` time. Since the logarithmic factor is typically much smaller than the array size, we can simplify this to `O(m)`.
- Sorting the `nums` array takes `O(n log n)` time.
- Iterating through the sorted `nums` array takes `O(n)` time in the worst case.
- Overall: `O(m) + O(n log n) + O(n) = O(m + n log n)`

**Space Complexity:** `O(1)` or `O(n)` depending on the sorting algorithm implementation.

- The variable `x` uses `O(1)` space.
- The sorting operation may use `O(1)` space if an in-place sorting algorithm like heapsort is used, or `O(n)` space if the implementation uses a sorting algorithm like mergesort or if Python's Timsort requires auxiliary space.
- In Python's case, the `sort()` method sorts in-place but may use up to `O(n)` auxiliary space in the worst case.
- Overall: `O(n)` in the worst case for Python's implementation.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Understanding Why GCD Works**
A common misconception is thinking you need to check if an element from `nums` divides every single element in `numsDivide`. The key insight is that if a number divides the GCD of all elements in `numsDivide`, it will divide every element in that array. This is because the GCD is the largest common divisor, and any divisor of the GCD is also a divisor of all the original numbers.

**Solution**: Remember that GCD(a, b, c) divides a, b, and c. So if x divides GCD(a, b, c), then x also divides a, b, and c.

### 2. **Modifying the Original Array**
The code sorts `nums` in-place with `nums.sort()`. If the problem requires preserving the original array or if you need to track original indices, this approach will fail.

**Solution**: Create a copy before sorting:
```python
sorted_nums = sorted(nums)
for index, value in enumerate(sorted_nums):
    if target_gcd % value == 0:
        return index
```

### 3. **Edge Case: Single Element in numsDivide**
When `numsDivide` has only one element, the loop `for value in numsDivide[1:]` won't execute, leaving `target_gcd` as just `numsDivide[0]`. While this is actually correct behavior, it might not be immediately obvious.

**Solution**: The current implementation handles this correctly, but for clarity, you could use:
```python
target_gcd = numsDivide[0]
for i in range(1, len(numsDivide)):
    target_gcd = gcd(target_gcd, numsDivide[i])
```

### 4. **Duplicate Values Confusion**
When `nums` contains duplicates of the smallest valid divisor, you might think you need special handling. For example, if `nums = [3, 3, 3, 6]` and the smallest valid divisor is 3, all three 3's are at indices 0, 1, and 2, but returning index 0 is correct since we don't need to delete any of them.

**Solution**: The current approach handles this correctly - it returns the index of the first occurrence, which means all duplicates of that value remain in the array.

