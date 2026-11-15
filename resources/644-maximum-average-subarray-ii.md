# 644. Maximum Average Subarray II

> Source: https://algo.monster/liteproblems/644

## Problem Statement

You are given an integer array `nums` with `n` elements and an integer `k`.

Your task is to find a contiguous subarray whose length is **at least** `k` that has the maximum possible average value. Return this maximum average value.

In other words, among all subarrays with length greater than or equal to `k`, you need to find the one with the highest average (sum of elements divided by the number of elements) and return that average value.

The answer will be accepted if it has a calculation error less than `10^-5` from the correct answer.

For example, if `nums = [1, 12, -5, -6, 50, 3]` and `k = 4`, you need to check all subarrays with length 4, 5, and 6:
- Subarrays of length 4: `[1, 12, -5, -6]` with average 0.5, `[12, -5, -6, 50]` with average 12.75, `[-5, -6, 50, 3]` with average 10.5
- Subarrays of length 5: `[1, 12, -5, -6, 50]` with average 10.4, `[12, -5, -6, 50, 3]` with average 10.8
- Subarray of length 6: `[1, 12, -5, -6, 50, 3]` with average 9.16667

The maximum average among all these subarrays is 12.75.

## Intuition

The key insight is that we can use [binary search](/problems/binary_search_intro) on the answer itself. Instead of trying to find the maximum average directly, we can ask a simpler question: "Is there a subarray of length at least `k` with average value greater than or equal to some value `v`?"

Why does this work? Because if we can find such a subarray with average `≥ v`, then the maximum average must be at least `v`. If we cannot find such a subarray, then the maximum average must be less than `v`. This creates a monotonic property perfect for [binary search](/problems/binary_search_intro).

The search space for our answer is bounded - the minimum possible average is the minimum element in the array (when we take just that element if `k=1`), and the maximum possible average is the maximum element in the array (when we take just that element if `k=1`).

Now, how do we check if there exists a subarray with length `≥ k` and average `≥ v`? Here's the clever transformation:

If a subarray `[a₁, a₂, ..., aⱼ]` has average `≥ v`, then:
- `(a₁ + a₂ + ... + aⱼ) / j ≥ v`
- `a₁ + a₂ + ... + aⱼ ≥ v × j`
- `(a₁ - v) + (a₂ - v) + ... + (aⱼ - v) ≥ 0`

This transforms our problem! If we subtract `v` from each element, we just need to check if there's a subarray of length `≥ k` with non-negative sum.

To efficiently check this, we use a sliding window technique with prefix sums. We maintain the minimum [prefix sum](/problems/subarray_sum) seen so far (at least `k` positions back), and check if the current prefix sum minus this minimum is non-negative. This gives us the maximum sum for any subarray ending at the current position with length `≥ k`.

By combining [binary search](/problems/binary_search_intro) with this efficient checking mechanism, we can find the maximum average in `O(n log(max-min))` time.

**Learn more about [Binary Search](/problems/binary_search_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The solution uses **[binary search](/problems/binary_search_intro)** combined with a **sliding window technique** to find the maximum average.

**[Binary Search](/problems/binary_search_intro) Setup:**
- Initialize left boundary `l = min(nums)` and right boundary `r = max(nums)`
- Set precision `eps = 1e-5` for the binary search termination condition
- While `r - l >= eps`, compute `mid = (l + r) / 2` and check if there exists a subarray with length `≥ k` and average `≥ mid`
- If such a subarray exists, update `l = mid` (the answer could be higher)
- Otherwise, update `r = mid` (the answer must be lower)

**The Check Function:**
The `check(v)` function determines if there's a subarray of length `≥ k` with average `≥ v`.

1. **Transform the problem:** Subtract `v` from each element conceptually. Now we need to find if there's a subarray of length `≥ k` with sum `≥ 0`.

2. **Initial window:** Calculate the sum of the first `k` elements minus `k * v`:
   ```python
   s = sum(nums[:k]) - k * v
   ```
   If `s >= 0`, we found a valid subarray immediately.

3. **Sliding window with [prefix sum](/problems/subarray_sum) optimization:**
   - Maintain `s` as the cumulative sum up to current position (all elements minus `v`)
   - Maintain `t` as the cumulative sum of elements that are now `k` positions behind
   - Maintain `mi` as the minimum prefix sum seen so far (at least `k` positions back)

   For each new element at position `i` (where `i >= k`):
   ```python
   s += nums[i] - v          # Add current element to cumulative sum
   t += nums[i - k] - v      # Add element that's k positions behind
   mi = min(mi, t)           # Update minimum prefix sum
   ```

4. **Check condition:** If `s >= mi`, it means there exists a subarray ending at position `i` with length `≥ k` and sum `≥ 0`. This is because `s - mi` represents the sum of some subarray with length `≥ k`.

**Why this works:**
- `s` represents the sum of all elements from index 0 to `i` (after subtracting `v`)
- `mi` represents the minimum [prefix sum](/problems/subarray_sum) among positions 0 to `i-k`
- `s - mi` gives us the maximum sum of any subarray ending at `i` with length `≥ k`
- If `s - mi >= 0` (equivalent to `s >= mi`), we found a valid subarray

**Time Complexity:** `O(n * log((max-min)/eps))` where `n` is the array length. The [binary search](/problems/binary_search_intro) runs `O(log((max-min)/eps))` iterations, and each check takes `O(n)` time.

**Space Complexity:** `O(1)` as we only use a few variables to track the sums and minimum values.

## Example Walkthrough

Let's walk through a small example with `nums = [1, 12, -5, -6, 50, 3]` and `k = 4`.

**Step 1: Binary Search Setup**
- `l = min(nums) = -6`, `r = max(nums) = 50`
- We'll binary search for the maximum average between -6 and 50

**Step 2: First Binary Search Iteration**
- `mid = (-6 + 50) / 2 = 22`
- Check if there's a subarray of length ≥ 4 with average ≥ 22

**Step 3: Running check(22)**
- Transform array by subtracting 22: `[-21, -10, -27, -28, 28, -19]`
- Need to find if there's a subarray of length ≥ 4 with sum ≥ 0

Initial window (first k=4 elements):
- Sum = -21 + (-10) + (-27) + (-28) = -86
- Since -86 < 0, continue sliding

Sliding to position 4 (index 4):
- `s = -86 + 28 = -58` (cumulative sum from 0 to 4)
- `t = -21` (cumulative sum from 0 to 0)
- `mi = -21` (minimum prefix sum so far)
- Check: Is -58 ≥ -21? No, continue

Sliding to position 5 (index 5):
- `s = -58 + (-19) = -77` (cumulative sum from 0 to 5)
- `t = -21 + (-10) = -31` (cumulative sum from 0 to 1)
- `mi = min(-21, -31) = -31`
- Check: Is -77 ≥ -31? No

No valid subarray found with average ≥ 22, so `r = 22`

**Step 4: Second Binary Search Iteration**
- `mid = (-6 + 22) / 2 = 8`
- Check if there's a subarray of length ≥ 4 with average ≥ 8

**Step 5: Running check(8)**
- Transform array by subtracting 8: `[-7, 4, -13, -14, 42, -5]`

Initial window:
- Sum = -7 + 4 + (-13) + (-14) = -30
- Since -30 < 0, continue sliding

Sliding to position 4:
- `s = -30 + 42 = 12` (cumulative sum from 0 to 4)
- `t = -7` (cumulative sum from 0 to 0)
- `mi = -7`
- Check: Is 12 ≥ -7? **Yes!**
- This means subarray from position 1 to 4 has sum = 12 - (-7) = 19 ≥ 0
- Original subarray `[12, -5, -6, 50]` has average = 51/4 = 12.75 ≥ 8 ✓

Valid subarray found, so `l = 8`

**Step 6: Continue Binary Search**
The process continues, narrowing the range:
- Next `mid = (8 + 22) / 2 = 15`, check fails, so `r = 15`
- Next `mid = (8 + 15) / 2 = 11.5`, check succeeds, so `l = 11.5`
- Next `mid = (11.5 + 15) / 2 = 13.25`, check fails, so `r = 13.25`
- Next `mid = (11.5 + 13.25) / 2 = 12.375`, check succeeds, so `l = 12.375`
- Continue until `r - l < eps`

**Final Answer:** The algorithm converges to approximately 12.75, which is indeed the maximum average (from subarray `[12, -5, -6, 50]`).

## Python Solution

```python
class Solution:
    def findMaxAverage(self, nums: List[int], k: int) -> float:
        def can_achieve_average(target_avg: float) -> bool:
            """
            Check if there exists a subarray of length at least k
            with average >= target_avg.

            We transform the problem by subtracting target_avg from each element.
            If any subarray sum >= 0, then its average >= target_avg.
            """
            # Calculate sum of first k elements after transformation
            current_sum = sum(nums[:k]) - k * target_avg
            if current_sum >= 0:
                return True

            # Use sliding window with prefix sum optimization
            # prefix_sum tracks sum of elements that have left the minimum window
            prefix_sum = 0
            min_prefix_sum = 0  # Minimum prefix sum seen so far

            for i in range(k, len(nums)):
                # Add new element to current sum
                current_sum += nums[i] - target_avg
                # Add element that just left the k-size window to prefix
                prefix_sum += nums[i - k] - target_avg
                # Track minimum prefix sum (best point to start subarray)
                min_prefix_sum = min(min_prefix_sum, prefix_sum)

                # If current_sum - min_prefix_sum >= 0, we found a valid subarray
                # This represents the best subarray ending at position i
                if current_sum >= min_prefix_sum:
                    return True

            return False

        # Binary search on the answer
        epsilon = 1e-5
        left, right = min(nums), max(nums)

        while right - left >= epsilon:
            mid = (left + right) / 2
            if can_achieve_average(mid):
                # If we can achieve this average, try for a higher one
                left = mid
            else:
                # If we cannot achieve this average, try a lower one
                right = mid

        return left
```

## Time and Space Complexity

The time complexity is `O(n × log M)`, where `n` is the length of the array `nums` and `M` is the difference between the maximum and minimum values in the array.

The algorithm uses binary search on the answer space `[min(nums), max(nums)]`. The binary search continues while `r - l >= eps`, where `eps = 1e-5`. The number of iterations is `O(log((max(nums) - min(nums))/eps))`, which simplifies to `O(log M)` when considering `M = max(nums) - min(nums)`.

In each iteration of the binary search, the `check` function is called, which:
- Computes the sum of the first `k` elements: `O(k)`
- Iterates through the remaining `n - k` elements: `O(n - k)`
- Overall takes `O(n)` time

Therefore, the total time complexity is `O(n × log M)`.

The space complexity is `O(1)`. The algorithm only uses a constant amount of extra space for variables like `s`, `t`, `mi`, `l`, `r`, `mid`, and `eps`, regardless of the input size.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Initialization of Minimum Prefix Sum**

**The Pitfall:**
A common mistake is initializing `min_prefix_sum` with a large positive value like `float('inf')` instead of `0`. This breaks the algorithm's logic.

```python
# WRONG - This will fail to find valid subarrays
min_prefix_sum = float('inf')
```

**Why it's wrong:**
- The `min_prefix_sum` represents the minimum sum of a prefix that can be excluded from our total sum
- Initially, we can choose to exclude nothing (empty prefix), which has sum 0
- If we initialize with `float('inf')`, we might miss valid subarrays that use all elements from the start

**The Solution:**
Always initialize `min_prefix_sum = 0` to represent the option of not excluding any prefix:
```python
min_prefix_sum = 0  # Correct initialization
```

### 2. **Off-by-One Error in Sliding Window**

**The Pitfall:**
Incorrectly calculating which element to add to the prefix sum, often using `nums[i-k+1]` instead of `nums[i-k]`.

```python
# WRONG - This adds the wrong element to prefix
prefix_sum += nums[i - k + 1] - target_avg
```

**Why it's wrong:**
- At position `i`, we're considering subarrays of length `k` to `i+1`
- The element at `i-k` is the one that just became eligible to be excluded
- Using `i-k+1` would track the wrong element, leading to incorrect subarray sums

**The Solution:**
Use the correct index `i-k` for the element that's exactly `k` positions behind:
```python
prefix_sum += nums[i - k] - target_avg  # Correct indexing
```

### 3. **Precision Issues in Binary Search**

**The Pitfall:**
Using integer division or incorrect epsilon value, leading to infinite loops or premature termination.

```python
# WRONG - Integer division loses precision
mid = (left + right) // 2

# WRONG - Epsilon too large, might miss the optimal answer
epsilon = 0.01
```

**Why it's wrong:**
- Integer division truncates decimals, causing loss of precision in average calculations
- Too large epsilon might terminate before finding the answer within the required `10^-5` precision

**The Solution:**
Use floating-point division and appropriate epsilon:
```python
mid = (left + right) / 2  # Float division
epsilon = 1e-5  # Or smaller, like 1e-6 for safety
```

### 4. **Forgetting Initial k-Length Window Check**

**The Pitfall:**
Only checking windows in the sliding phase and forgetting to check if the first k elements already form a valid subarray.

```python
# WRONG - Missing initial check
def can_achieve_average(target_avg):
    current_sum = sum(nums[:k]) - k * target_avg
    # Forgot to check if current_sum >= 0 here!

    prefix_sum = 0
    min_prefix_sum = 0
    # ... rest of sliding window
```

**Why it's wrong:**
- The first k elements form a valid subarray of exactly length k
- If their average is already >= target_avg, we should return True immediately
- Missing this check causes incorrect results when the optimal subarray is exactly the first k elements

**The Solution:**
Always check the initial k-length window before sliding:
```python
current_sum = sum(nums[:k]) - k * target_avg
if current_sum >= 0:  # Don't forget this check!
    return True
```

