# 2321. Maximum Score Of Spliced Array

> Source: https://algo.monster/liteproblems/2321

## Problem Statement

You have two integer arrays `nums1` and `nums2`, both with the same length `n`. You can perform at most one operation where you select a contiguous subarray from one array and swap it with the corresponding subarray from the other array.

Specifically, you choose indices `left` and `right` (where `0 <= left <= right < n`) and swap:
- Elements from `nums1[left]` to `nums1[right]` with
- Elements from `nums2[left]` to `nums2[right]`

For example, with `nums1 = [1,2,3,4,5]` and `nums2 = [11,12,13,14,15]`, if you choose `left = 1` and `right = 2`:
- `nums1` becomes `[1,12,13,4,5]` (positions 1 and 2 now have values from `nums2`)
- `nums2` becomes `[11,2,3,14,15]` (positions 1 and 2 now have values from `nums1`)

Your goal is to maximize the score, which is defined as the maximum sum between the two arrays after the operation. The score = `max(sum(nums1), sum(nums2))`.

You can choose to either:
1. Perform the swap operation once to maximize the score
2. Not perform any swap at all if it doesn't improve the score

The problem asks you to return the maximum possible score you can achieve.

## Intuition

The key insight is to think about what happens when we swap a subarray. When we swap elements from position `left` to `right`, we're essentially transferring some elements from `nums1` to `nums2` and vice versa.

Let's say initially `sum(nums1) = S1` and `sum(nums2) = S2`. When we swap a subarray, what changes?

For `nums2`, we remove its original subarray and add the subarray from `nums1`. The change in sum is:
- New sum of `nums2` = `S2 - (nums2[left] + ... + nums2[right]) + (nums1[left] + ... + nums1[right])`
- This can be rewritten as: `S2 + sum of (nums1[i] - nums2[i])` for `i` from `left` to `right`

This means the gain for `nums2` when swapping is the sum of differences `(nums1[i] - nums2[i])` over the swapped range. Similarly, the gain for `nums1` would be the sum of `(nums2[i] - nums1[i])`.

Now the problem reduces to: find a contiguous subarray where the sum of differences is maximum. This is exactly the classic maximum subarray sum problem!

We create a difference array `d[i] = nums1[i] - nums2[i]`. Finding the maximum subarray sum in `d` tells us the maximum gain we can add to `S2` by swapping. We use Kadane's algorithm for this.

But wait - we could also be trying to maximize `nums1` instead of `nums2`. So we need to check both directions:
1. Maximize `nums2` by finding max subarray sum in `(nums1[i] - nums2[i])`
2. Maximize `nums1` by finding max subarray sum in `(nums2[i] - nums1[i])`

The final answer is the maximum of these two possibilities, considering we might also choose not to swap at all if it doesn't improve the score.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a helper function `f(nums1, nums2)` that finds the maximum gain possible when transferring elements from `nums1` to `nums2`.

**Step 1: Calculate the difference array**
```python
d = [a - b for a, b in zip(nums1, nums2)]
```
This creates an array where `d[i] = nums1[i] - nums2[i]`. Each element represents how much we gain in `nums2`'s sum if we include position `i` in our swap.

**Step 2: Apply Kadane's Algorithm**
```python
t = mx = d[0]
for v in d[1:]:
    if t > 0:
        t += v
    else:
        t = v
    mx = max(mx, t)
```
This finds the maximum subarray sum in the difference array:
- `t` tracks the current subarray sum
- If `t > 0`, we extend the current subarray by adding the next element `v`
- If `t <= 0`, we start a new subarray from the current element `v`
- `mx` keeps track of the maximum sum seen so far

**Step 3: Calculate initial sums**
```python
s1, s2 = sum(nums1), sum(nums2)
```
We compute the original sums of both arrays.

**Step 4: Find the maximum score**
```python
return max(s2 + f(nums1, nums2), s1 + f(nums2, nums1))
```
We consider two scenarios:
- `s2 + f(nums1, nums2)`: Maximum sum of `nums2` after potentially swapping (transferring elements from `nums1` to `nums2`)
- `s1 + f(nums2, nums1)`: Maximum sum of `nums1` after potentially swapping (transferring elements from `nums2` to `nums1`)

The function returns the maximum of these two values, which represents the best possible score we can achieve with at most one swap operation.

The time complexity is `O(n)` where `n` is the length of the arrays, and the space complexity is `O(n)` for storing the difference array.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- `nums1 = [2, 8, 3, 1]`
- `nums2 = [5, 4, 6, 7]`

**Step 1: Calculate initial sums**
- `s1 = sum(nums1) = 2 + 8 + 3 + 1 = 14`
- `s2 = sum(nums2) = 5 + 4 + 6 + 7 = 22`
- Current maximum score without swapping = `max(14, 22) = 22`

**Step 2: Try to maximize nums2 by transferring from nums1**

Calculate difference array `d[i] = nums1[i] - nums2[i]`:
- `d[0] = 2 - 5 = -3`
- `d[1] = 8 - 4 = 4`
- `d[2] = 3 - 6 = -3`
- `d[3] = 1 - 7 = -6`
- Difference array: `[-3, 4, -3, -6]`

Apply Kadane's algorithm to find maximum subarray sum:
- Start with `t = mx = -3`
- Process `d[1] = 4`: Since `t = -3 < 0`, start new subarray: `t = 4`, `mx = max(-3, 4) = 4`
- Process `d[2] = -3`: Since `t = 4 > 0`, extend: `t = 4 + (-3) = 1`, `mx = max(4, 1) = 4`
- Process `d[3] = -6`: Since `t = 1 > 0`, extend: `t = 1 + (-6) = -5`, `mx = max(4, -5) = 4`

Maximum gain for nums2 = 4
New score if we maximize nums2 = `22 + 4 = 26`

This gain of 4 comes from swapping only index 1:
- After swap: `nums1 = [2, 4, 3, 1]`, `nums2 = [5, 8, 6, 7]`
- New sums: `sum(nums1) = 10`, `sum(nums2) = 26`

**Step 3: Try to maximize nums1 by transferring from nums2**

Calculate difference array `d[i] = nums2[i] - nums1[i]`:
- `d[0] = 5 - 2 = 3`
- `d[1] = 4 - 8 = -4`
- `d[2] = 6 - 3 = 3`
- `d[3] = 7 - 1 = 6`
- Difference array: `[3, -4, 3, 6]`

Apply Kadane's algorithm:
- Start with `t = mx = 3`
- Process `d[1] = -4`: Since `t = 3 > 0`, extend: `t = 3 + (-4) = -1`, `mx = max(3, -1) = 3`
- Process `d[2] = 3`: Since `t = -1 < 0`, start new: `t = 3`, `mx = max(3, 3) = 3`
- Process `d[3] = 6`: Since `t = 3 > 0`, extend: `t = 3 + 6 = 9`, `mx = max(3, 9) = 9`

Maximum gain for nums1 = 9
New score if we maximize nums1 = `14 + 9 = 23`

This gain of 9 comes from swapping indices 2 to 3:
- After swap: `nums1 = [2, 8, 6, 7]`, `nums2 = [5, 4, 3, 1]`
- New sums: `sum(nums1) = 23`, `sum(nums2) = 13`

**Step 4: Final answer**
- Maximum score = `max(26, 23) = 26`

The optimal strategy is to swap index 1, transferring the value 8 from nums1 to nums2, achieving a maximum score of 26.

## Python Solution

```python
class Solution:
    def maximumsSplicedArray(self, nums1: List[int], nums2: List[int]) -> int:
        def find_max_subarray_sum(arr1: List[int], arr2: List[int]) -> int:
            """
            Find the maximum subarray sum of the difference array (arr1 - arr2).
            Uses Kadane's algorithm to find the maximum sum of a contiguous subarray.
          
            Args:
                arr1: First array
                arr2: Second array
          
            Returns:
                Maximum sum of any contiguous subarray in the difference array
            """
            # Create difference array where each element is arr1[i] - arr2[i]
            difference = [a - b for a, b in zip(arr1, arr2)]
          
            # Initialize current sum and maximum sum with the first element
            current_sum = max_sum = difference[0]
          
            # Apply Kadane's algorithm starting from the second element
            for value in difference[1:]:
                # If current sum is positive, continue the subarray
                if current_sum > 0:
                    current_sum += value
                else:
                    # Otherwise, start a new subarray from current element
                    current_sum = value
              
                # Update maximum sum found so far
                max_sum = max(max_sum, current_sum)
          
            return max_sum
      
        # Calculate the sum of both arrays
        sum1, sum2 = sum(nums1), sum(nums2)
      
        # Two scenarios:
        # 1. Replace a subarray of nums2 with corresponding subarray from nums1
        #    Result: sum2 + maximum gain from swapping (nums1[i] - nums2[i])
        # 2. Replace a subarray of nums1 with corresponding subarray from nums2
        #    Result: sum1 + maximum gain from swapping (nums2[i] - nums1[i])
        return max(
            sum2 + find_max_subarray_sum(nums1, nums2),
            sum1 + find_max_subarray_sum(nums2, nums1)
        )
```

## Time and Space Complexity

**Time Complexity:** `O(n)`, where `n` is the length of the input arrays.

The algorithm performs the following operations:
- Creating the difference array `d`: `O(n)` time using `zip` and list comprehension
- Finding the maximum subarray sum in array `d`: `O(n)` time with a single pass through the array
- Computing `sum(nums1)` and `sum(nums2)`: `O(n)` time each
- Calling function `f` twice: `2 * O(n) = O(n)`

Overall, all operations are linear, resulting in `O(n)` time complexity.

**Space Complexity:** `O(n)`

The space usage includes:
- The difference array `d` created in function `f`: `O(n)` space
- Variables `t`, `mx`, `s1`, `s2`: `O(1)` space
- The function is called twice, but not recursively, so only one difference array exists at any given time

The dominant factor is the difference array, giving us `O(n)` space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Consider No Swap Scenario**

A common mistake is assuming that we must perform a swap operation. However, if all elements in `nums1` are smaller than their corresponding elements in `nums2` (or vice versa), the maximum subarray sum in the difference array could be negative. In this case, not swapping at all gives a better result.

**Pitfall Example:**
```python
# Incorrect approach - doesn't handle negative maximum properly
def find_max_subarray_sum(arr1, arr2):
    difference = [a - b for a, b in zip(arr1, arr2)]
    # ... Kadane's algorithm ...
    return max_sum  # Could be negative!

# If max_sum is negative, adding it decreases the total
result = sum2 + find_max_subarray_sum(nums1, nums2)  # Wrong if max_sum < 0
```

**Solution:**
The code correctly handles this by using Kadane's algorithm which will return 0 when the best option is to not swap anything (empty subarray). However, to make this more explicit, you could add:
```python
max_sum = max(max_sum, 0)  # Ensure we consider "no swap" option
```

### 2. **Initializing Kadane's Algorithm Incorrectly**

Another pitfall is incorrectly initializing the maximum sum to 0 or negative infinity instead of the first element of the difference array.

**Pitfall Example:**
```python
# Wrong initialization
current_sum = max_sum = 0  # Incorrect - misses single element subarrays
# or
max_sum = float('-inf')  # Also problematic for edge cases
```

**Solution:**
Always initialize both `current_sum` and `max_sum` with the first element of the difference array:
```python
current_sum = max_sum = difference[0]
```

### 3. **Not Considering Both Swap Directions**

A critical mistake is only considering swapping from `nums1` to `nums2` but forgetting that swapping from `nums2` to `nums1` might yield a better result.

**Pitfall Example:**
```python
# Only considers one direction
def maximumsSplicedArray(self, nums1, nums2):
    sum2 = sum(nums2)
    max_gain = find_max_subarray_sum(nums1, nums2)
    return sum2 + max_gain  # Missing the other direction!
```

**Solution:**
Always check both directions and return the maximum:
```python
return max(
    sum2 + find_max_subarray_sum(nums1, nums2),
    sum1 + find_max_subarray_sum(nums2, nums1)
)
```

### 4. **Edge Case: Arrays with Single Element**

When arrays have only one element, the algorithm still works but could be inefficient or unclear.

**Pitfall Example:**
```python
# Potential issue with single element arrays
for value in difference[1:]:  # This loop won't execute if len(difference) == 1
    # ...
```

**Solution:**
The current implementation handles this correctly since when `difference[1:]` is empty, the loop doesn't execute and `max_sum` remains as `difference[0]`, which is the correct answer for single-element arrays.

