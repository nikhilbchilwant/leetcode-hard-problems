# 410. Split Array Largest Sum

> Source: https://algo.monster/liteproblems/410

## Problem Statement

You are given an integer array `nums` and an integer `k`. Your task is to split the array `nums` into exactly `k` non-empty subarrays in a way that minimizes the largest sum among all the subarrays.

A subarray is a contiguous portion of the original array, meaning the elements must appear consecutively in their original order.

The goal is to find a splitting strategy where:
1. The array is divided into exactly `k` parts
2. Each part is a contiguous subarray (elements stay in their original order)
3. Among all possible ways to split the array into `k` parts, you want the one where the maximum sum of any subarray is as small as possible

For example, if you have `nums = [7,2,5,10,8]` and `k = 2`, you could split it as:
- `[7,2,5]` and `[10,8]` with sums 14 and 18, so the largest sum is 18
- `[7,2]` and `[5,10,8]` with sums 9 and 23, so the largest sum is 23
- `[7]` and `[2,5,10,8]` with sums 7 and 25, so the largest sum is 25

Among these options (and others), the first split gives the minimum possible largest sum of 18.

The function should return this minimized largest sum value.

## Intuition

The key insight is recognizing that there's a relationship between the maximum allowed sum for subarrays and the number of subarrays we need to create. If we allow a larger maximum sum per subarray, we can fit more elements into each subarray, resulting in fewer total subarrays. Conversely, if we restrict the maximum sum to be smaller, we'll need more subarrays to accommodate all elements.

Think of it like packing items into boxes with a weight limit - the higher the weight limit per box, the fewer boxes you need.

This relationship is monotonic: as we increase the maximum allowed sum, the number of required subarrays either stays the same or decreases. This monotonic property is perfect for binary search.

We can establish bounds for our search:
- The minimum possible largest sum is `max(nums)` - we can't go lower than this because at least one subarray must contain the largest element
- The maximum possible largest sum is `sum(nums)` - this happens when we put all elements in a single subarray (when `k = 1`)

For any candidate value `mid` between these bounds, we can greedily check if it's possible to split the array into at most `k` subarrays where no subarray sum exceeds `mid`. We traverse the array, accumulating elements into the current subarray. When adding another element would exceed `mid`, we start a new subarray. If we can do this with `k` or fewer subarrays, then `mid` is a valid maximum sum.

Since we want the minimum possible value of this maximum sum, we use binary search to efficiently find the smallest `mid` that allows a valid split into at most `k` subarrays.

## Solution Approach

The solution uses binary search combined with a greedy validation function to find the minimum possible largest sum. We apply the standard binary search template to find the first value where splitting is feasible.

**Binary Search Setup:**
- Initialize `left = max(nums)` as the lower bound (smallest possible maximum sum)
- Initialize `right = sum(nums)` as the upper bound (largest possible maximum sum)
- Initialize `first_true_index = -1` to track the answer

**Feasible Function:**
The `feasible(max_sum)` function determines if we can split the array into at most `k` subarrays where no subarray sum exceeds `max_sum`. This creates a boolean pattern over our search space:

```
max_sum:    10   11   12   ...  17   18   19   ...  32
feasible: false false false ... false true true ... true
```

We want to find the first `true` position.

The checking logic:
1. Initialize `current_sum = 0` and `subarray_count = 1` (we start with one subarray)
2. For each element in `nums`:
   - If adding the element exceeds `max_sum`, start a new subarray
   - Otherwise, add it to the current subarray
3. Return `subarray_count <= k`

**Binary Search Template:**
```python
left, right = max(nums), sum(nums)
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1  # Try to find a smaller valid value
    else:
        left = mid + 1   # Current value is too small

return first_true_index
```

**Why This Works:**
- If a maximum sum `mx` allows splitting into at most `k` subarrays, we record it and try smaller values
- If a maximum sum `mx` requires more than `k` subarrays, we need to increase `mx`
- Binary search efficiently finds the minimum value where splitting is possible

**Time Complexity:** `O(n * log(sum(nums) - max(nums)))` where `n` is the length of the array
**Space Complexity:** `O(1)` as we only use a constant amount of extra space

## Example Walkthrough

Let's walk through the solution with `nums = [7,2,5,10,8]` and `k = 2`.

**Step 1: Set up binary search**
- `left = max(nums) = 10` (minimum possible largest sum)
- `right = sum(nums) = 32` (maximum possible largest sum)
- `first_true_index = -1` (tracks the answer)

**Step 2: Binary search with template**

**Iteration 1:** `left=10, right=32`
- `mid = (10 + 32) // 2 = 21`
- Check `feasible(21)`:
  - `current_sum = 0`, `count = 1`
  - Add 7: `current_sum = 7`
  - Add 2: `current_sum = 9`
  - Add 5: `current_sum = 14`
  - Add 10: `14 + 10 = 24 > 21`, start new subarray: `current_sum = 10`, `count = 2`
  - Add 8: `current_sum = 18`
  - Result: `count = 2 ≤ k = 2` → `True`
- `feasible(21) = True`, so `first_true_index = 21`, `right = 20`

**Iteration 2:** `left=10, right=20`
- `mid = (10 + 20) // 2 = 15`
- Check `feasible(15)`:
  - After processing: needs 3 subarrays (14, 10, 8)
  - Result: `count = 3 > k = 2` → `False`
- `feasible(15) = False`, so `left = 16`

**Iteration 3:** `left=16, right=20`
- `mid = (16 + 20) // 2 = 18`
- Check `feasible(18)`:
  - Subarrays: `[7,2,5]` (sum=14), `[10,8]` (sum=18)
  - Result: `count = 2 ≤ k = 2` → `True`
- `feasible(18) = True`, so `first_true_index = 18`, `right = 17`

**Iteration 4:** `left=16, right=17`
- `mid = (16 + 17) // 2 = 16`
- `feasible(16) = False` (needs 3 subarrays)
- `left = 17`

**Iteration 5:** `left=17, right=17`
- `mid = 17`
- `feasible(17) = False` (needs 3 subarrays)
- `left = 18`

**Loop ends:** `left = 18 > right = 17`

**Answer:** `first_true_index = 18`

The final split: `[7,2,5]` (sum = 14) and `[10,8]` (sum = 18).

## Python Solution

```python
class Solution:
    def splitArray(self, nums: List[int], k: int) -> int:
        def feasible(max_sum):
            """
            Check if we can split the array into at most k subarrays
            where each subarray sum doesn't exceed max_sum.
            """
            current_sum = 0
            subarray_count = 1  # Start with one subarray

            for num in nums:
                if current_sum + num > max_sum:
                    # Start a new subarray
                    current_sum = num
                    subarray_count += 1
                else:
                    current_sum += num

            return subarray_count <= k

        # Binary search bounds
        left, right = max(nums), sum(nums)
        first_true_index = -1

        while left <= right:
            mid = (left + right) // 2
            if feasible(mid):
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1

        return first_true_index
```

## Time and Space Complexity

**Time Complexity:** `O(n × log m)`

The algorithm uses binary search on the range `[max(nums), sum(nums)]`. The search space has a size of `sum(nums) - max(nums) + 1`, which is at most `m` (where `m = sum(nums)`). Binary search over this range takes `O(log m)` iterations.

For each binary search iteration, the `check` function is called, which iterates through all `n` elements of the array once, taking `O(n)` time.

Therefore, the overall time complexity is `O(n × log m)`, where `n` is the length of the array and `m` is the sum of all elements in the array.

**Space Complexity:** `O(1)`

The algorithm only uses a constant amount of extra space:
- The `check` function uses variables `s`, `cnt`, and `x` which require `O(1)` space
- The binary search uses variables `left` and `right` which require `O(1)` space
- The `bisect_left` function with a range object doesn't create the entire range in memory but generates values on-the-fly, using `O(1)` space

Therefore, the space complexity is `O(1)`.

## Common Pitfalls

### 1. **Mixing Binary Search Template Variants**
Two templates are common: `while left < right` with `right = mid`, returning `left`; and `while left <= right` with `right = mid - 1`, recording the answer in `first_true_index`. Both are correct here, because `sum(nums)` is always feasible. Mixing them is not.

**Incorrect:**
```python
while left < right:
    mid = (left + right) // 2
    if feasible(mid):
        right = mid - 1  # WRONG with left < right: can step past the answer
    else:
        left = mid + 1
return left
```

For `nums = [7,2,5,10,8]` and `k = 2`, this checks 21 (true), 15 (false), 18 (true, so `right = 17`), and 16 (false, so `left = 17`), then stops with `left = 17`. The answer is 18, and 17 is not even feasible.

**Correct (template-compliant):**
```python
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
return first_true_index
```

Recording each feasible `mid` before moving `right` past it keeps the answer from being lost.

### 2. **Incorrect Binary Search Bounds**
A common mistake is setting incorrect bounds:
- Using `left = 0` or `left = min(nums)` instead of `left = max(nums)`
- The minimum possible maximum sum is the largest single element (each subarray must have at least one element)

**Why this matters:** The greedy check assumes every single element fits within `max_sum`. When `max_sum` is smaller than some element, the check starts a new subarray for that element but still lets its sum exceed the limit, so it can report `True` for an impossible limit. For `nums = [1, 100]` and `k = 2`, `feasible(1)` puts 1 in the first subarray and 100 in the second, counts 2 subarrays, and returns `True`. With `left = min(nums) = 1`, the search returns 1 instead of 100. Starting from `left = max(nums)` keeps every tested limit at least as large as each element.

### 3. **Confusion Between "At Most k" vs "Exactly k"**
The feasible function checks for "at most k subarrays" (`subarray_count <= k`), not exactly k.

**Incorrect:**
```python
return subarray_count == k
```

**Why "at most" works:** If we can split into fewer than k subarrays with a given maximum sum, we can always further split to create exactly k subarrays without increasing the maximum sum.

### 4. **Greedy Algorithm Correctness**
The greedy approach (starting a new subarray only when sum exceeds limit) is optimal for a fixed maximum sum limit. Delaying the split allows more elements per subarray, minimizing the total count.

