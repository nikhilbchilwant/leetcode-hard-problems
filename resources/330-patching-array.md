# 330. Patching Array

> Source: https://algo.monster/liteproblems/330

## Problem Statement

You are given a sorted integer array `nums` and an integer `n`. Your task is to add/patch the minimum number of elements to the array so that any number in the range `[1, n]` inclusive can be formed by summing some elements in the array.

The goal is to find the minimum number of patches (additions) required to achieve this coverage.

For example, if you have `nums = [1, 3]` and `n = 6`:
- Initially, you can form: 1 (using 1), 3 (using 3), 4 (using 1+3)
- You cannot form 2, 5, or 6
- By adding 2 to the array, you get `[1, 2, 3]`
- Now you can form all numbers from 1 to 6: 1 (using 1), 2 (using 2), 3 (using 3), 4 (using 1+3), 5 (using 2+3), 6 (using 1+2+3)
- Therefore, the minimum number of patches needed is 1

## Intuition

Let's think about this problem step by step. If we can form all numbers from `[1, x-1]`, what happens when we add a new number to our collection?

Suppose we add a number `y` to our array. Since we can already form all numbers from `[1, x-1]`, we can now also form `y, y+1, y+2, ..., y+(x-1)` by adding `y` to each of the numbers we could form before. This extends our range to `[1, x+y-1]`.

The key insight is: **when we need to form the number `x` but can't, what's the best number to add?**

If we add a number greater than `x`, say `x+k`, we still can't form `x` itself. We'd only be able to form numbers starting from `x+k`.

If we add a number less than or equal to `x`, we can form `x` (either directly if we add `x`, or by combining the added number with existing sums). The question becomes: which number maximizes our new range?

- Adding `x` gives us range `[1, x + (x-1)] = [1, 2x-1]`
- Adding any `y < x` gives us range `[1, x + y - 1]` where `x + y - 1 < 2x - 1`

Therefore, **greedily adding `x` itself maximizes our coverage** whenever we encounter a gap.

Now, how do we use the existing sorted array? As we iterate through `nums`, if the current number `nums[i]` is less than or equal to our current gap `x`, we should use it (it's free!). This extends our range to `[1, x + nums[i] - 1]`. If `nums[i]` is too large (greater than `x`), we have no choice but to patch the array with `x` to fill the gap.

This greedy strategy ensures we add the minimum number of patches because each patch maximally extends our range, and we only patch when absolutely necessary.

## Solution Approach

We implement the greedy strategy using two key variables:
- `x`: tracks the smallest positive integer that cannot be represented (initially 1)
- `i`: index to traverse through the sorted array `nums`
- `ans`: counter for the number of patches added

The algorithm works as follows:

1. **Initialize state**: Start with `x = 1` (the first number we need to form), `i = 0` (array index), and `ans = 0` (patch count).

2. **Main loop**: Continue while `x <= n` (we need to cover all numbers up to `n`):
 
   **Case 1: Can use existing array element**
   - If `i < len(nums)` and `nums[i] <= x`:
     - We can use `nums[i]` to extend our range
     - Since we can form `[1, x-1]`, adding `nums[i]` lets us form `[1, x + nums[i] - 1]`
     - Update: `x = x + nums[i]` (new smallest uncovered number)
     - Move to next array element: `i += 1`
 
   **Case 2: Need to patch**
   - If `nums[i] > x` or we've exhausted the array:
     - We must patch the array with number `x`
     - This extends our range from `[1, x-1]` to `[1, 2x-1]`
     - Update: `x = 2 * x` (or `x <<= 1` using bit shift)
     - Increment patch count: `ans += 1`

3. **Return result**: Once `x > n`, we've covered all numbers in `[1, n]`, so return `ans`.

**Example walkthrough** with `nums = [1, 5, 10]`, `n = 20`:
- Initially: `x = 1`, can't form any number
- `nums[0] = 1 <= x`: use it, `x = 1 + 1 = 2`, range `[1, 1]`
- `nums[1] = 5 > x = 2`: patch with 2, `x = 2 * 2 = 4`, range `[1, 3]`, patches = 1
- `nums[1] = 5 > x = 4`: patch with 4, `x = 4 * 2 = 8`, range `[1, 7]`, patches = 2
- `nums[1] = 5 <= x = 8`: use it, `x = 8 + 5 = 13`, range `[1, 12]`
- `nums[2] = 10 <= x = 13`: use it, `x = 13 + 10 = 23`, range `[1, 22]`
- Now `x = 23 > n = 20`, done with 2 patches

The time complexity is `O(m + log n)` where `m` is the length of `nums`, since we traverse the array once and potentially patch `O(log n)` times (each patch doubles our range).

## Example Walkthrough

Let's walk through a detailed example with `nums = [1, 3]` and `n = 6`.

**Goal**: Ensure we can form all numbers from 1 to 6 using elements from the array (with patches if needed).

**Initial State**:
- `x = 1` (smallest number we can't form yet)
- `i = 0` (array index)
- `ans = 0` (patches added)

**Step 1**: `x = 1`, checking `nums[0] = 1`
- Since `nums[0] = 1 <= x = 1`, we can use this element
- Currently can form: `[0]` (empty), adding 1 gives us: `[1]`
- Update: `x = 1 + 1 = 2` (next number to check)
- Move to next element: `i = 1`
- Range covered: `[1, 1]`

**Step 2**: `x = 2`, checking `nums[1] = 3`
- Since `nums[1] = 3 > x = 2`, we cannot use this element yet (it would leave a gap at 2)
- Must patch with 2 to fill the gap
- Currently can form `[1]`, adding 2 gives us: `[1, 2, 3]` (1+2=3)
- Update: `x = 2 * 2 = 4` (doubling our range)
- Increment patches: `ans = 1`
- Range covered: `[1, 3]`

**Step 3**: `x = 4`, checking `nums[1] = 3`
- Since `nums[1] = 3 <= x = 4`, we can now use this element
- Currently can form `[1, 2, 3]`, adding element 3 gives us: `[1, 2, 3, 4, 5, 6]`
  - Previous: 1, 2, 3
  - New: 3+1=4, 3+2=5, 3+3=6
- Update: `x = 4 + 3 = 7`
- Move to next element: `i = 2` (now out of array bounds)
- Range covered: `[1, 6]`

**Step 4**: `x = 7`, but `7 > n = 6`
- We've covered all numbers up to 6, so we're done!

**Result**: Minimum patches needed = 1 (we added the number 2)

**Verification**: With array `[1, 2, 3]`:
- 1 = 1
- 2 = 2  
- 3 = 3
- 4 = 1 + 3
- 5 = 2 + 3
- 6 = 1 + 2 + 3

The key insight: We greedily patch with the smallest uncovered number to maximize the range extension each time.

## Python Solution

```python
from typing import List


class Solution:
    def minPatches(self, nums: List[int], n: int) -> int:
        # max_reachable represents the maximum sum we can form with current numbers
        # All numbers in range [1, max_reachable) can be formed
        max_reachable = 1
      
        # patches_count tracks the number of patches (additions) needed
        patches_count = 0
      
        # index to traverse through the nums array
        index = 0
      
        # Continue until we can form all numbers up to n
        while max_reachable <= n:
            # If current number in nums can extend our range without gaps
            if index < len(nums) and nums[index] <= max_reachable:
                # Add current number to extend the reachable range
                max_reachable += nums[index]
                index += 1
            else:
                # Need to patch: add max_reachable itself to the array
                # This doubles our reachable range
                patches_count += 1
                max_reachable *= 2  # Equivalent to max_reachable <<= 1
      
        return patches_count
```

## Time and Space Complexity

The time complexity is `O(m + log n)`, where `m` is the length of the array `nums` and `n` is the target value.

- The loop continues while `x <= n`. In each iteration, either:
  - We consume an element from `nums` (happens at most `m` times), or
  - We patch by doubling `x` (via `x <<= 1`), which means `x` grows exponentially
- When patching, `x` doubles each time, so it takes at most `log n` patches to reach `n`
- Therefore, the total iterations is bounded by `m + log n`

The space complexity is `O(1)`.

- The algorithm only uses a constant amount of extra space for variables `x`, `ans`, and `i`
- No additional data structures are created that depend on the input size

## Common Pitfalls

### 1. **Off-by-One Error with Range Coverage**

A common mistake is misunderstanding what `max_reachable` represents. Many developers incorrectly assume it represents the maximum number we can form, but it actually represents the first number we CANNOT form yet.

**Incorrect interpretation:**
```python
# WRONG: Thinking max_reachable is the largest number we can form
while max_reachable < n:  # This would stop too early!
    if index < len(nums) and nums[index] <= max_reachable:
        max_reachable += nums[index]
        index += 1
    else:
        patches_count += 1
        max_reachable *= 2
```

**Why it fails:** The loop must keep going while `n` itself is still unreachable, which is exactly when `max_reachable == n`. With `nums = [1, 2]` and `n = 4`, the array covers `[1, 3]`, so `max_reachable = 4`. The condition `4 < 4` is false, the loop stops, and the function returns 0 even though 4 cannot be formed. The correct answer is 1 (patch with 4).

**Correct approach:** Use `max_reachable <= n` because when `max_reachable = n + 1`, we know we can form all numbers in `[1, n]`.

### 2. **Integer Overflow in Large Cases**

When dealing with large values of `n` (like `n = 2^31 - 1`), the multiplication `max_reachable *= 2` can cause integer overflow in languages with fixed integer sizes.

**Problematic scenario:**
```python
# When max_reachable is very large
max_reachable *= 2  # Could overflow in some languages
```

**Solution:** In Python, integers have arbitrary precision so overflow isn't an issue. However, for optimization or when porting to other languages, consider:
- Using bit shift: `max_reachable <<= 1` (slightly faster)
- Adding overflow checks in languages like C++ or Java
- Using long/int64 data types when appropriate

### 3. **Forgetting to Check Array Bounds**

Another pitfall is accessing `nums[index]` without first checking if `index < len(nums)`.

**Incorrect code:**
```python
# WRONG: Checking nums[index] before bounds check
while max_reachable <= n:
    if nums[index] <= max_reachable:  # IndexError when index >= len(nums)
        max_reachable += nums[index]
        index += 1
    else:
        patches_count += 1
        max_reachable *= 2
```

**Correct approach:** Always check `index < len(nums)` before accessing `nums[index]`:
```python
if index < len(nums) and nums[index] <= max_reachable:
    # Safe to access nums[index]
```

### 4. **Misunderstanding the Greedy Choice**

Some might think we should always use array elements when possible, even if they're much larger than needed.

**Example mistake:** With `nums = [1, 100]` and `n = 50`, after using 1 (covering [1,1]), someone might think:
- "Since 100 is in the array, let's wait to use it"
- This leads to unnecessary patches

**Correct understanding:** We only use an array element when `nums[index] <= max_reachable`. If `nums[index] > max_reachable`, using it would create gaps in our coverage. The optimal strategy is to patch with `max_reachable` itself to maximize range extension.

