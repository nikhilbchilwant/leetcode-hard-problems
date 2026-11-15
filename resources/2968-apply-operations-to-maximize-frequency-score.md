# 2968. Apply Operations to Maximize Frequency Score

> Source: https://algo.monster/liteproblems/2968

## Problem Statement

You have an array of integers `nums` (0-indexed) and an integer `k` representing the maximum number of operations you can perform.

In each operation, you can:
- Select any index `i` in the array
- Either increase or decrease `nums[i]` by 1

Your goal is to maximize the **frequency** of the most frequent element in the final array. The frequency of an element is simply how many times it appears in the array.

For example, if you have `nums = [1, 2, 4]` and `k = 5`:
- You could change the array to `[2, 2, 2]` by decreasing 4 twice (2 operations) and increasing 1 once (1 operation), using 3 operations total
- This gives you a maximum frequency of 3 (the element 2 appears 3 times)

The key insight is that you want to make as many elements equal as possible within your operation budget `k`. Each operation moves an element one step closer to your target value, so elements that are already close to each other will require fewer operations to become equal.

## Intuition

The first key observation is that if we want to make multiple elements equal, it's most efficient to work with elements that are already close to each other in value. This suggests we should sort the array first, as consecutive elements in a sorted array have the smallest differences.

Once sorted, we realize that to maximize frequency, we should focus on making a contiguous segment of the sorted array equal. Why? Because if we try to make non-contiguous elements equal, we'd be wasting operations on elements in between that we're not including.

Now, what value should we change all elements in a segment to? The median is optimal. Consider a segment `[1, 2, 5]`:
- If we change all to 1: operations needed = `0 + 1 + 4 = 5`
- If we change all to 2: operations needed = `1 + 0 + 3 = 4`
- If we change all to 5: operations needed = `4 + 3 + 0 = 7`

The median minimizes the total distance (and thus operations) needed.

The next insight is about the search strategy. If we can achieve a frequency of `x` (meaning we can make `x` elements equal), then we can definitely achieve any frequency less than `x` - we'd just use fewer elements. This monotonic property screams [binary search](/problems/binary_search_intro)!

So our approach becomes:
1. [Binary search](/problems/binary_search_intro) on the answer (the maximum frequency)
2. For each candidate frequency `mid`, check if it's achievable
3. To check if frequency `mid` is achievable, try all possible windows of size `mid` in the sorted array
4. For each window, calculate the cost to make all elements equal to the median
5. If any window has cost ≤ `k`, then frequency `mid` is achievable

The cost calculation uses prefix sums for efficiency. For a window from index `i` to `j`, we need:
- Elements below the median to increase to reach the median
- Elements above the median to decrease to reach the median

This gives us the formula: `left + right` where:
- `left` = sum of differences between median and elements below it
- `right` = sum of differences between elements above median and the median itself

**Learn more about [Binary Search](/problems/binary_search_intro), [Prefix Sum](/problems/subarray_sum), [Sorting](/problems/sorting_summary) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**1. [Sorting](/problems/sorting_summary) and [Prefix Sum](/problems/subarray_sum) Setup**

First, we sort the array to group similar values together:
```python
nums.sort()
```

Then we build a [prefix sum](/problems/subarray_sum) array for efficient range sum queries:
```python
s = list(accumulate(nums, initial=0))
```
This allows us to calculate the sum of any subarray `nums[i:j]` as `s[j] - s[i]` in O(1) time.

**2. [Binary Search](/problems/binary_search_intro) Framework**

We [binary search](/problems/binary_search_intro) on the possible frequency values:
```python
l, r = 0, n
while l < r:
    mid = (l + r + 1) >> 1
```
- `l = 0`: minimum possible frequency
- `r = n`: maximum possible frequency (all elements become equal)
- `mid = (l + r + 1) >> 1`: middle value with upward rounding

**3. Checking Feasibility of a Frequency**

For each candidate frequency `mid`, we check all possible windows of size `mid`:
```python
for i in range(n - mid + 1):
    j = i + mid
    x = nums[(i + j) // 2]  # median element
```

**4. Calculating Operations Needed**

For each window `[i, j)`, we calculate the cost to make all elements equal to the median `x`:

```python
left = ((i + j) // 2 - i) * x - (s[(i + j) // 2] - s[i])
right = (s[j] - s[(i + j) // 2]) - ((j - (i + j) // 2) * x)
```

Breaking down the formulas:
- **Left cost**: Elements from index `i` to `(i+j)//2 - 1` need to increase to `x`
  - Number of elements: `(i + j) // 2 - i`
  - Target sum if all were `x`: `((i + j) // 2 - i) * x`
  - Current sum: `s[(i + j) // 2] - s[i]`
  - Operations needed: target sum - current sum

- **Right cost**: Elements from index `(i+j)//2 + 1` to `j-1` need to decrease to `x`
  - Current sum: `s[j] - s[(i + j) // 2]`
  - Target sum if all were `x`: `(j - (i + j) // 2) * x`
  - Operations needed: current sum - target sum

**5. [Binary Search](/problems/binary_search_intro) Decision**

If any window can be transformed with at most `k` operations:
```python
if left + right <= k:
    ok = True
    break
```

Based on feasibility:
- If feasible (`ok = True`): This frequency works, try larger (`l = mid`)
- If not feasible: This frequency is too large, try smaller (`r = mid - 1`)

**Time Complexity**: O(n² log n)
- [Binary search](/problems/binary_search_intro): O(log n) iterations
- Each iteration checks O(n) windows
- Each window check: O(1) using prefix sums

**Space Complexity**: O(n) for the [prefix sum](/problems/subarray_sum) array

## Example Walkthrough

Let's walk through a concrete example with `nums = [1, 4, 8, 13]` and `k = 5`.

**Step 1: Sort and Build Prefix Sum**
- Array is already sorted: `[1, 4, 8, 13]`
- Prefix sum array: `s = [0, 1, 5, 13, 26]`
  - This allows us to quickly calculate range sums

**Step 2: Binary Search on Frequency**

We'll binary search between frequency 1 and 4 (array length).

**First iteration: mid = 2**
- Can we make 2 elements equal with ≤ 5 operations?
- Check all windows of size 2:

  Window `[1, 4]` (indices 0-1):
  - Median element: `nums[0] = 1`
  - Cost to make both equal to 1: We need to decrease 4 to 1
  - Operations = 3 ✓ (≤ 5, so frequency 2 is achievable)

**Second iteration: mid = 3**
- Can we make 3 elements equal with ≤ 5 operations?
- Check all windows of size 3:

  Window `[1, 4, 8]` (indices 0-2):
  - Median element: `nums[1] = 4`
  - Cost calculation:
    - Left side: Change 1 to 4 = 3 operations
    - Right side: Change 8 to 4 = 4 operations
    - Total = 7 operations ✗ (> 5)

  Window `[4, 8, 13]` (indices 1-3):
  - Median element: `nums[2] = 8`
  - Cost calculation:
    - Left side: Change 4 to 8 = 4 operations
    - Right side: Change 13 to 8 = 5 operations
    - Total = 9 operations ✗ (> 5)

  Frequency 3 is not achievable with k=5.

**Binary Search Continues:**
- Since frequency 3 failed, we search lower
- We already know frequency 2 works
- Binary search converges to answer: 2

**Verification of the Answer:**
With k=5 operations, we can transform `[1, 4, 8, 13]` to have at most frequency 2:
- One possibility: Change 4 to 1 (3 operations) → `[1, 1, 8, 13]`
- Another possibility: Change 8 to 4 (4 operations) → `[1, 4, 4, 13]`

The maximum frequency achievable is 2.

### Template Solution (Last True Pattern)

```python
def maxFrequencyScore(self, nums: List[int], k: int) -> int:
    nums.sort()
    n = len(nums)
    prefix_sum = list(accumulate(nums, initial=0))

    def feasible(window_size: int) -> bool:
        for i in range(n - window_size + 1):
            j = i + window_size
            median_idx = (i + j) // 2
            median = nums[median_idx]

            left_cost = (median_idx - i) * median - (prefix_sum[median_idx] - prefix_sum[i])
            right_cost = (prefix_sum[j] - prefix_sum[median_idx]) - (j - median_idx) * median

            if left_cost + right_cost <= k:
                return True
        return False

    # Binary search: find last true (maximum feasible frequency)
    left, right = 1, n
    last_true_index = 0

    while left <= right:
        mid = (left + right) // 2
        if feasible(mid):
            last_true_index = mid
            left = mid + 1  # Try larger
        else:
            right = mid - 1  # Try smaller

    return last_true_index
```

## Python Solution

```python
class Solution:
    def maxFrequencyScore(self, nums: List[int], k: int) -> int:
        # Sort the array for efficient median-based calculations
        nums.sort()

        # Create prefix sum array for range sum queries
        # prefix_sum[i] represents sum of elements from nums[0] to nums[i-1]
        prefix_sum = list(accumulate(nums, initial=0))
        n = len(nums)

        def feasible(window_size: int) -> bool:
            """Check if we can make window_size elements equal with at most k operations."""
            # Try all possible windows of the given size
            for window_start in range(n - window_size + 1):
                window_end = window_start + window_size

                # Find the median element's index in current window
                median_index = (window_start + window_end) // 2
                median_value = nums[median_index]

                # Calculate cost to change all elements left of median to median value
                left_count = median_index - window_start
                left_sum = prefix_sum[median_index] - prefix_sum[window_start]
                left_cost = left_count * median_value - left_sum

                # Calculate cost to change all elements right of median to median value
                right_count = window_end - median_index
                right_sum = prefix_sum[window_end] - prefix_sum[median_index]
                right_cost = right_sum - right_count * median_value

                # Check if total cost is within budget k
                if left_cost + right_cost <= k:
                    return True
            return False

        # Binary search for maximum feasible frequency
        # Pattern: true, true, ..., true, false, false (find last true)
        left, right = 1, n
        last_true_index = 0

        while left <= right:
            mid = (left + right) // 2
            if feasible(mid):
                last_true_index = mid
                left = mid + 1  # Try larger frequency
            else:
                right = mid - 1  # Try smaller frequency

        return last_true_index
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The time complexity consists of two main parts:
- Sorting the array takes `O(n log n)` time
- Binary search on the answer space: The binary search runs for `O(log n)` iterations (searching from 0 to n for the maximum frequency). In each iteration, we check all possible subarrays of length `mid`, which takes `O(n)` time. Therefore, this part contributes `O(n × log n)` time.

Since both operations are `O(n × log n)`, the overall time complexity is `O(n × log n)`.

**Space Complexity:** `O(n)`

The space complexity comes from:
- The prefix sum array `s` which stores `n+1` elements, requiring `O(n)` space
- The sorting operation may require `O(log n)` space for the recursion stack in the worst case (depending on the sorting algorithm implementation)
- Other variables like `l`, `r`, `mid`, `i`, `j`, `x`, `left`, `right`, and `ok` use `O(1)` space

The dominant term is `O(n)` from the prefix sum array, so the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using Wrong Binary Search Template Variant**

This is a maximization problem (find largest feasible frequency), which requires the "last true" pattern. Using the "first true" pattern will give incorrect results.

**The Issue:**
```python
# WRONG: "first true" pattern (for minimization)
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        first_true_index = mid
        right = mid - 1  # Wrong direction!
    else:
        left = mid + 1
```

**Solution:**
Use the "last true" pattern for maximization:
```python
# CORRECT: "last true" pattern (for maximization)
last_true_index = 0
while left <= right:
    mid = (left + right) // 2
    if feasible(mid):
        last_true_index = mid
        left = mid + 1  # Try larger values
    else:
        right = mid - 1  # Try smaller values
```

### 2. **Incorrect Median Index Calculation for Even-Length Windows**

One of the most common mistakes is mishandling the median index calculation when the window has an even number of elements.

**The Issue:**
For even-length windows, there are two middle elements. The formula `(i + j) // 2` gives us the upper median index. This inconsistency can lead to incorrect cost calculations.

**Solution:**
Be explicit about which median you're using and ensure consistency:
```python
# For a window [window_start, window_end)
median_index = (window_start + window_end) // 2
```

### 3. **Confusion with Prefix Sum Boundaries**

The prefix sum array uses `initial=0`, making `prefix_sum[i]` represent the sum of elements from index 0 to i-1 (exclusive of i).

**The Issue:**
```python
# Wrong: sum from i to j inclusive
wrong_sum = prefix_sum[j] - prefix_sum[i]  # This actually gives sum from i to j-1

# Correct: sum from i to j inclusive
correct_sum = prefix_sum[j + 1] - prefix_sum[i]
```

### 4. **Not Defining a Clear Feasible Function**

Mixing the feasibility check into the binary search loop makes the code harder to reason about and more error-prone.

**Solution:**
Extract the feasibility check into a separate function:
```python
def feasible(window_size: int) -> bool:
    """Check if we can make window_size elements equal with at most k operations."""
    for window_start in range(n - window_size + 1):
        # ... check each window
        if cost <= k:
            return True
    return False
```

### 5. **Integer Overflow in Cost Calculations**

For large arrays with large values, the cost calculations might overflow in languages with fixed integer sizes.

**Solution (for languages with overflow concerns):**
Use `long long` in C++ or `long` in Java for storing sums and products.

