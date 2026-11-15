# 891. Sum of Subsequence Widths

> Source: https://algo.monster/liteproblems/891

## Problem Statement

You are given an array of integers `nums`. For any subsequence of this array, the **width** is defined as the difference between the maximum and minimum elements in that subsequence.

Your task is to find the sum of widths of all possible non-empty subsequences of `nums`. Since this sum can be very large, return the result modulo `10^9 + 7`.

A **subsequence** is formed by selecting some (or all) elements from the original array while maintaining their relative order. Elements don't need to be consecutive. For example, if `nums = [0,3,1,6,2,2,7]`, then `[3,6,2,7]` is a valid subsequence.

**Example breakdown:**
- If `nums = [2,1,3]`, the non-empty subsequences are:
  - `[2]` with width = 0 (max=2, min=2)
  - `[1]` with width = 0 (max=1, min=1)  
  - `[3]` with width = 0 (max=3, min=3)
  - `[2,1]` with width = 1 (max=2, min=1)
  - `[2,3]` with width = 1 (max=3, min=2)
  - `[1,3]` with width = 2 (max=3, min=1)
  - `[2,1,3]` with width = 2 (max=3, min=1)
  - Total sum = 0 + 0 + 0 + 1 + 1 + 2 + 2 = 6

## Intuition

The key insight is that instead of enumerating all subsequences (which would be exponentially expensive), we can think about the contribution of each element to the total sum.

First, let's consider what happens when we sort the array. After [sorting](/problems/sorting_summary), for any subsequence, the width is simply `max_element - min_element`. 

Now, here's the crucial observation: for each element in the sorted array, we need to figure out:
1. In how many subsequences does this element appear as the maximum?
2. In how many subsequences does this element appear as the minimum?

Let's say we have a sorted array and we're looking at element at index `i`:
- For this element to be the **maximum** in a subsequence, we can only include elements from indices `0` to `i-1` (all smaller elements). We have `2^i` ways to form such subsequences (each of the `i` elements before can either be included or not).
- For this element to be the **minimum** in a subsequence, we can only include elements from indices `i+1` to `n-1` (all larger elements). We have `2^(n-1-i)` ways to form such subsequences.

So element `nums[i]` contributes:
- `+nums[i] * 2^i` to the sum (when it's the maximum)
- `-nums[i] * 2^(n-1-i)` to the sum (when it's the minimum)

The clever trick in the solution is to pair up the positive and negative contributions. Notice that `nums[i]` as a maximum (contributing positively) can be paired with `nums[n-1-i]` as a minimum (contributing negatively). Both have the same power of 2 coefficient: `2^i`.

Therefore, we can iterate through the array once and for each position `i`, calculate: `(nums[i] - nums[n-1-i]) * 2^i`. This gives us the net contribution for this pair of positions, eliminating the need to calculate positive and negative contributions separately.

**Learn more about [Math](/problems/math-basics) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows the mathematical insight we developed:

1. **Sort the array**: First, we sort `nums` in ascending order. This allows us to easily identify which elements can be maximums and minimums in subsequences.

2. **Initialize variables**:
   - `mod = 10^9 + 7` for the modulo operation
   - `ans = 0` to accumulate the total sum
   - `p = 1` to represent `2^i` (starts at `2^0 = 1`)

3. **Single pass calculation**: We iterate through each index `i` and its corresponding value `v`:
   ```python
   for i, v in enumerate(nums):
       ans = (ans + (v - nums[-i - 1]) * p) % mod
       p = (p * 2) % mod
   ```
 
   Let's break down what happens in each iteration:
   - `v` is `nums[i]` (current element)
   - `nums[-i - 1]` is `nums[n-1-i]` (element from the opposite end)
   - `(v - nums[-i - 1]) * p` calculates the net contribution:
     - When `v` acts as maximum in subsequences: contributes `+v * 2^i`
     - When `nums[n-1-i]` acts as minimum in subsequences: contributes `-nums[n-1-i] * 2^i`
     - Combined: `(nums[i] - nums[n-1-i]) * 2^i`
 
4. **Power of 2 update**: After each iteration, we update `p` by multiplying it by 2 (the code names this variable `power_of_two`; `ans` is `result`). This maintains `p = 2^(i+1)` for the next iteration.

5. **Modulo operations**: We apply modulo at each step to prevent integer overflow and get the final answer modulo `10^9 + 7`.

**Time Complexity**: `O(n log n)` due to [sorting](/problems/sorting_summary), where `n` is the length of the array.

**Space Complexity**: `O(n)`, because Python's sort (Timsort) can use `O(n)` auxiliary space. Apart from sorting, the loop uses `O(1)` extra space.

The elegance of this solution lies in avoiding the exponential complexity of enumerating all subsequences by instead calculating each element's contribution mathematically based on its position in the sorted array.

## Example Walkthrough

Let's walk through the solution with `nums = [2, 1, 3]`.

**Step 1: Sort the array**
- Original: `[2, 1, 3]`
- Sorted: `[1, 2, 3]`

**Step 2: Initialize variables**
- `mod = 10^9 + 7`
- `ans = 0`
- `p = 1` (represents 2^0)

**Step 3: Iterate through the sorted array**

**Iteration 1** (i=0, v=1):
- Current element: `v = 1`
- Opposite element: `nums[-0-1] = nums[-1] = nums[2] = 3`
- Contribution: `(1 - 3) * 1 = -2 * 1 = -2`
- Update answer: `ans = (0 + (-2)) % mod = 1000000005` (Python's `%` with a positive modulus never returns a negative value, so `-2` wraps to `10^9 + 5`)
- Update power: `p = 1 << 1 = 2` (now represents 2^1)

**Iteration 2** (i=1, v=2):
- Current element: `v = 2`
- Opposite element: `nums[-1-1] = nums[-2] = nums[1] = 2`
- Contribution: `(2 - 2) * 2 = 0 * 2 = 0`
- Update answer: `ans = (1000000005 + 0) % mod = 1000000005`
- Update power: `p = 2 << 1 = 4` (now represents 2^2)

**Iteration 3** (i=2, v=3):
- Current element: `v = 3`
- Opposite element: `nums[-2-1] = nums[-3] = nums[0] = 1`
- Contribution: `(3 - 1) * 4 = 2 * 4 = 8`
- Update answer: `ans = (1000000005 + 8) % mod = 6`
- Update power: `p = 4 << 1 = 8` (now represents 2^3)

**Final answer: 6**

Let's verify this makes sense:
- Element 1: appears as min in 4 subsequences ([1], [1,2], [1,3], [1,2,3]) → contributes -1×4 = -4
- Element 1: appears as max in 1 subsequence ([1]) → contributes +1×1 = +1
- Element 2: appears as min in 2 subsequences ([2], [2,3]) → contributes -2×2 = -4
- Element 2: appears as max in 2 subsequences ([2], [1,2]) → contributes +2×2 = +4
- Element 3: appears as min in 1 subsequence ([3]) → contributes -3×1 = -3
- Element 3: appears as max in 4 subsequences ([3], [1,3], [2,3], [1,2,3]) → contributes +3×4 = +12

Total: -4 + 1 + (-4) + 4 + (-3) + 12 = 6 ✓

The algorithm cleverly pairs these contributions using the sorted array positions to compute the same result efficiently.

## Python Solution

```python
class Solution:
    def sumSubseqWidths(self, nums: List[int]) -> int:
        """
        Calculate the sum of widths of all non-empty subsequences.
        Width is defined as the difference between max and min elements.
      
        Key insight: After sorting, for each element at position i:
        - It will be the maximum in 2^i subsequences (all subsets of elements before it)
        - It will be the minimum in 2^(n-i-1) subsequences (all subsets of elements after it)
        """
        MOD = 10**9 + 7
      
        # Sort the array to easily identify min/max contributions
        nums.sort()
      
        # Initialize result and power of 2
        result = 0
        power_of_two = 1
      
        # Process each element and its contribution
        for index, value in enumerate(nums):
            # For element at index i:
            # - As maximum: contributes +value * 2^i times
            # - As minimum: contributes -value * 2^(n-i-1) times
            # nums[-index-1] gives the element from the end (symmetric position)
            contribution = (value - nums[-index - 1]) * power_of_two
            result = (result + contribution) % MOD
          
            # Update power of 2 for next iteration (2^(i+1))
            power_of_two = (power_of_two * 2) % MOD
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)`

The time complexity is dominated by the sorting operation `nums.sort()`, which takes `O(n log n)` time where `n` is the length of the input array. After sorting, there is a single loop that iterates through all `n` elements, performing constant-time operations in each iteration (arithmetic operations and modulo). Therefore, the overall time complexity is `O(n log n) + O(n) = O(n log n)`.

**Space Complexity:** `O(1)` or `O(n)`

The space complexity depends on the sorting algorithm implementation:
- If we consider the sorting to be in-place (like Python's Timsort which modifies the input list), the additional space used is `O(1)` for the variables `mod`, `ans`, `p`, `i`, and `v`.
- However, Python's sort() method actually uses `O(n)` space in the worst case due to the temporary arrays created during the merge operations in Timsort.
- If we strictly consider only the extra space used by our algorithm (excluding the sort), it's `O(1)`.

The most accurate answer would be `O(n)` space complexity due to the sorting algorithm's space requirements.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow Without Proper Modulo Application**

**The Pitfall:**
A common mistake is forgetting to apply modulo operations at intermediate steps, especially when calculating powers of 2. Since we're dealing with potentially large arrays (up to 10^5 elements), `2^i` can quickly exceed integer limits. For example, `2^100` is astronomically large and will cause overflow even in languages with 64-bit integers.

**Incorrect Implementation:**
```python
def sumSubseqWidths(self, nums: List[int]) -> int:
    MOD = 10**9 + 7
    nums.sort()
    result = 0
    power_of_two = 1
  
    for index, value in enumerate(nums):
        contribution = (value - nums[-index - 1]) * power_of_two
        result += contribution  # Missing modulo here!
        power_of_two *= 2       # Missing modulo here!
  
    return result % MOD  # Only applying modulo at the end
```

**Why This Fails:**
- In Java, C++, or TypeScript, `power_of_two` overflows a 64-bit integer after about 63 doublings (and loses precision in a JavaScript number after about 53), so the result is wrong
- The intermediate `result` can also overflow before the final modulo in those languages
- Python's big integers keep this version correct, but `power_of_two` grows to about `n` bits, so each multiplication gets slower and the loop takes roughly `O(n^2)` time for `n = 10^5`

**Correct Solution:**
Apply modulo at every arithmetic operation that could potentially overflow:
```python
for index, value in enumerate(nums):
    contribution = (value - nums[-index - 1]) * power_of_two
    result = (result + contribution) % MOD  # Apply modulo after addition
    power_of_two = (power_of_two * 2) % MOD  # Apply modulo after multiplication
```

### 2. **Misunderstanding the Index Symmetry**

**The Pitfall:**
Incorrectly calculating the symmetric element from the opposite end of the sorted array. The formula `nums[-index - 1]` might be confusing and lead to off-by-one errors.

**Incorrect Implementation:**
```python
for index, value in enumerate(nums):
    # Wrong: Using nums[-index] instead of nums[-index - 1]
    contribution = (value - nums[-index]) * power_of_two
    # This would incorrectly pair nums[0] with nums[0] instead of nums[n-1]
```

**Why This Matters:**
- For an array of length `n`, when we're at index `i`, we want to pair it with index `n-1-i`
- Using negative indexing: `nums[-1]` is the last element, `nums[-2]` is second-to-last, etc.
- So for index `i`, we need `nums[-(i+1)]` or `nums[-i-1]` to get `nums[n-1-i]`

**Verification Example:**
For `nums = [1, 2, 3, 4]` (already sorted):
- When `i=0`: We want `nums[0]` paired with `nums[3]` → `nums[-0-1] = nums[-1] = nums[3]` ✓
- When `i=1`: We want `nums[1]` paired with `nums[2]` → `nums[-1-1] = nums[-2] = nums[2]` ✓

### 3. **Not Sorting the Array First**

**The Pitfall:**
Forgetting to sort the array or assuming it's already sorted can completely break the algorithm.

**Why Sorting is Critical:**
The entire mathematical approach relies on the fact that after sorting:
- Elements to the left of position `i` are all smaller (can form subsequences where `nums[i]` is maximum)
- Elements to the right of position `i` are all larger (can form subsequences where `nums[i]` is minimum)

Without sorting, the counting logic falls apart and produces incorrect results.

**Prevention:**
Always include `nums.sort()` at the beginning of the solution, even if test cases appear sorted.

