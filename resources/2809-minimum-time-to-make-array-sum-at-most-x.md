# 2809. Minimum Time to Make Array Sum At Most x

> Source: https://algo.monster/liteproblems/2809

## Problem Statement

You have two arrays `nums1` and `nums2` of equal length. The problem involves a time-based simulation where:

1. **Every second**: All elements in `nums1` are automatically incremented by their corresponding elements in `nums2`. That is, for all indices `i`, `nums1[i]` becomes `nums1[i] + nums2[i]`.

2. **After each increment**: You can optionally choose one index `i` and set `nums1[i] = 0`.

Your goal is to find the **minimum time** (in seconds) needed to make the sum of all elements in `nums1` less than or equal to a given value `x`. If it's impossible to achieve this, return `-1`.

**Key Points:**
- At time `t`, if you haven't reset an element at index `i`, its value will be `nums1[i] + t * nums2[i]`
- You can reset at most one element per second (after the automatic increment)
- Once an element is reset to 0, it will continue growing from 0 in subsequent seconds

**Example scenario**: If `nums1 = [3, 2]`, `nums2 = [1, 2]`, and `x = 10`:
- At `t=0`: Sum is `3 + 2 = 5` (≤ 10, so answer could be 0)
- At `t=1`: Elements become `[4, 4]` before any operation, then you could reset one to get `[0, 4]` or `[4, 0]`
- At `t=2`: Elements that weren't reset continue growing, and you can reset another element

The challenge is determining the optimal strategy for which elements to reset and when, in order to keep the sum at or below `x` in minimum time.

## Intuition

Let's think about what happens when we reset an element at different times. If we reset `nums1[i]` at time `t`, we eliminate a value of `nums1[i] + t * nums2[i]` from the sum. This is a crucial observation - the later we reset an element, the more value we eliminate.

Now consider if we reset the same element multiple times. The second reset would only eliminate the growth since the first reset, which is less effective than resetting a different element that has been growing since the beginning. Therefore, **each element should be reset at most once**.

Since we can perform at most `n` operations (one per element), and each second allows at most one operation, the answer must be between `0` and `n` seconds.

Here's the key insight: If we decide to reset exactly `j` elements, which elements should we choose and in what order? 

- The value eliminated when resetting element `i` at time `t` is `nums1[i] + t * nums2[i]`
- If we're resetting `j` elements at times `1, 2, ..., j`, the total reduction is:
  - Element reset at time 1: `nums1[i₁] + 1 * nums2[i₁]`
  - Element reset at time 2: `nums1[i₂] + 2 * nums2[i₂]`
  - ...
  - Element reset at time j: `nums1[iⱼ] + j * nums2[iⱼ]`

Notice that elements with larger `nums2[i]` values benefit more from being reset later (since they're multiplied by the time). This suggests a **greedy strategy**: sort elements by `nums2` values in ascending order, so elements with larger growth rates are reset later to maximize the reduction.

After [sorting](/problems/sorting_summary), we need to determine which subset of elements to reset and in what order. This becomes a [dynamic programming](/problems/dynamic_programming_intro) problem: for the first `i` elements (after sorting), what's the maximum reduction we can achieve with exactly `j` operations? We can either skip element `i` or reset it at time `j`, leading to the recurrence relation.

Finally, for each possible number of operations `j` from `0` to `n`, we check if the total sum after `j` seconds minus the maximum possible reduction is at most `x`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

Based on our intuition, we implement the solution using [sorting](/problems/sorting_summary) and [dynamic programming](/problems/dynamic_programming_intro).

**Step 1: [Sorting](/problems/sorting_summary)**
First, we sort the pairs `(nums1[i], nums2[i])` based on `nums2[i]` values in ascending order. This ensures that elements with larger growth rates will be considered for later time slots, maximizing the reduction value.

```python
sorted(zip(nums1, nums2), key=lambda z: z[1])
```

**Step 2: [Dynamic Programming](/problems/dynamic_programming_intro) Setup**
We define `f[i][j]` as the maximum reduction in sum achievable when considering the first `i` elements (after [sorting](/problems/sorting_summary)) with exactly `j` operations.

- Initialize a 2D array `f` of size `(n+1) × (n+1)` with all zeros
- `f[0][j] = 0` for all `j` (no elements means no reduction possible)

**Step 3: DP Transition**
For each element `i` (1-indexed) and each possible number of operations `j`:

```
f[i][j] = max(
    f[i-1][j],                              # Skip element i
    f[i-1][j-1] + nums1[i] + nums2[i] * j  # Reset element i at time j
)
```

The second option represents resetting the current element at time `j`, which eliminates `nums1[i] + nums2[i] * j` from the sum.

**Step 4: Finding the Answer**
After filling the DP table, we calculate:
- `s1 = sum(nums1)`: initial sum
- `s2 = sum(nums2)`: total growth per second

For each possible time `j` from `0` to `n`:
- The sum at time `j` without any operations would be `s1 + s2 * j`
- The maximum reduction possible with `j` operations is `f[n][j]`
- The actual sum after optimal operations is `s1 + s2 * j - f[n][j]`

We return the smallest `j` where `s1 + s2 * j - f[n][j] <= x`. If no such `j` exists, return `-1`.

**Time Complexity**: `O(n²)` for the DP computation
**Space Complexity**: `O(n²)` for the DP table

## Example Walkthrough

Let's walk through a concrete example with `nums1 = [3, 2, 5]`, `nums2 = [2, 1, 3]`, and `x = 15`.

**Step 1: Sort by nums2 values**
We sort the pairs by `nums2` in ascending order:
- Original pairs: `(3,2), (2,1), (5,3)`
- After sorting: `(2,1), (3,2), (5,3)`

**Step 2: Build DP table**
We create a DP table `f[i][j]` where `i` represents the first `i` elements considered and `j` represents using exactly `j` operations.

Let's fill the table step by step:

For element 1 (2,1):
- `f[1][0] = 0` (no operations, no reduction)
- `f[1][1] = max(0, 0 + 2 + 1*1) = 3` (reset at time 1, eliminates 2 + 1 = 3)

For element 2 (3,2):
- `f[2][0] = 0` (no operations)
- `f[2][1] = max(f[1][1], f[1][0] + 3 + 2*1) = max(3, 5) = 5` (better to reset element 2 at time 1)
- `f[2][2] = max(f[1][2], f[1][1] + 3 + 2*2) = max(0, 3 + 7) = 10` (reset element 1 at time 1, element 2 at time 2)

For element 3 (5,3):
- `f[3][0] = 0`
- `f[3][1] = max(f[2][1], f[2][0] + 5 + 3*1) = max(5, 8) = 8` (reset element 3 at time 1)
- `f[3][2] = max(f[2][2], f[2][1] + 5 + 3*2) = max(10, 5 + 11) = 16` (reset another element at time 1, element 3 at time 2)
- `f[3][3] = max(f[2][3], f[2][2] + 5 + 3*3) = max(0, 10 + 14) = 24` (reset elements 1,2 at times 1,2, element 3 at time 3)

**Step 3: Calculate sums and find answer**
- Initial sum `s1 = 3 + 2 + 5 = 10`
- Growth per second `s2 = 2 + 1 + 3 = 6`

Check each time:
- Time 0: Sum = `10 + 6*0 - 0 = 10` (≤ 15) ✓
- Time 1: Sum = `10 + 6*1 - 8 = 8` (≤ 15) ✓
- Time 2: Sum = `10 + 6*2 - 16 = 6` (≤ 15) ✓

The answer is **0** seconds since the initial sum of 10 is already ≤ 15.

Let's verify with a different target `x = 8`:
- Time 0: Sum = 10 (> 8) ✗
- Time 1: Sum = `10 + 6*1 - 8 = 8` (≤ 8) ✓

The answer would be **1** second.

The key insight illustrated here is that by sorting elements with smaller `nums2` values first, we ensure that elements with higher growth rates (like the element with `nums2=3`) are available for resetting at later times, maximizing the reduction value when multiplied by the time factor.

## Python Solution

```python
class Solution:
    def minimumTime(self, nums1: List[int], nums2: List[int], x: int) -> int:
        n = len(nums1)
      
        # dp[i][j] represents the maximum reduction we can achieve
        # by selecting j items from the first i items
        dp = [[0] * (n + 1) for _ in range(n + 1)]
      
        # Sort items by nums2 values (ascending order) to optimize the selection order
        # Each item contributes nums1[i] + nums2[i] * time when selected at a given time
        sorted_pairs = sorted(zip(nums1, nums2), key=lambda pair: pair[1])
      
        # Build the DP table
        for i in range(1, n + 1):
            value1, value2 = sorted_pairs[i - 1]
          
            for operations_count in range(n + 1):
                # Option 1: Don't select the current item
                dp[i][operations_count] = dp[i - 1][operations_count]
              
                # Option 2: Select the current item (if we have operations available)
                if operations_count > 0:
                    # When selected at time 'operations_count', this item contributes:
                    # value1 + value2 * operations_count to the reduction
                    reduction_with_current = dp[i - 1][operations_count - 1] + value1 + value2 * operations_count
                    dp[i][operations_count] = max(dp[i][operations_count], reduction_with_current)
      
        # Calculate initial sums
        sum1 = sum(nums1)
        sum2 = sum(nums2)
      
        # Check each possible number of operations (time steps)
        for time in range(n + 1):
            # Total sum at time t = initial_sum + sum2 * t - maximum_reduction
            total_sum_at_time = sum1 + sum2 * time - dp[n][time]
          
            if total_sum_at_time <= x:
                return time
      
        # If no valid time found, return -1
        return -1
```

## Time and Space Complexity

**Time Complexity:** `O(n^2)`

The code first sorts the zipped arrays based on the second element of each pair, which takes `O(n log n)` time. Then it uses a nested loop structure where the outer loop runs `n` times (iterating through sorted pairs) and the inner loop runs up to `n+1` times (iterating through `j` values). Within the nested loops, all operations are `O(1)`. Therefore, the dominant operation is the double nested loop which gives us `O(n * n) = O(n^2)`. The final loop to check conditions runs `O(n)` times. Since `O(n^2)` dominates `O(n log n)` and `O(n)`, the overall time complexity is `O(n^2)`.

**Space Complexity:** `O(n^2)`

The code creates a 2D array `f` with dimensions `(n+1) × (n+1)`, which requires `O(n^2)` space. The sorted operation creates a new list of tuples which takes `O(n)` space. Other variables like `s1`, `s2`, and loop variables use `O(1)` space. The dominant space usage comes from the 2D array `f`, making the overall space complexity `O(n^2)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Sorting Direction**
A critical pitfall is sorting the pairs by `nums2` in descending order instead of ascending order. This seems intuitive since we might want to "eliminate" the fastest-growing elements first.

**Why it's wrong:** When we reset an element at time `t`, we eliminate `nums1[i] + nums2[i] * t` from the sum. If we process elements with larger `nums2[i]` values first (at earlier times), we lose potential reduction value since `t` is smaller. By sorting in ascending order, elements with larger growth rates are reset at later times, maximizing the product `nums2[i] * t`.

**Example:** 
- If `nums2 = [3, 1]` and we reset both elements
- Ascending order: Reset index 1 at t=1 (saves 1×1=1), reset index 0 at t=2 (saves 3×2=6). Total savings: 7
- Descending order: Reset index 0 at t=1 (saves 3×1=3), reset index 1 at t=2 (saves 1×2=2). Total savings: 5

### 2. **Misunderstanding the DP State Definition**
Another common mistake is defining `dp[i][j]` as "the minimum sum achievable" rather than "the maximum reduction achievable". This leads to incorrect transitions and boundary conditions.

**Solution:** Always define DP in terms of maximum reduction. This makes the transition clearer: we're accumulating the total amount we can subtract from the ever-growing sum.

### 3. **Incorrect Time Calculation in DP Transition**
When calculating the reduction for resetting element `i` at operation `j`, using `j-1` instead of `j` as the time multiplier is a frequent error.

**Wrong:** `dp[i-1][j-1] + value1 + value2 * (j-1)`
**Correct:** `dp[i-1][j-1] + value1 + value2 * j`

**Why:** When we perform the j-th operation, it happens at time `j` (assuming 0-indexed operations but 1-indexed in this context). The element has grown for `j` seconds before being reset.

### 4. **Off-by-One Errors in Array Indexing**
Mixing 0-based and 1-based indexing when accessing `sorted_pairs` can cause array out-of-bounds errors or incorrect values.

**Solution:** Be consistent with indexing. In the code above, `dp` uses 1-based indexing for items (rows) while `sorted_pairs` uses 0-based indexing, hence we use `sorted_pairs[i-1]` when processing the i-th item in the DP table.

