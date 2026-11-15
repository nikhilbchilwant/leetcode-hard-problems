# 548. Split Array with Equal Sum

> Source: https://algo.monster/liteproblems/548

## Problem Statement

You are given an integer array `nums` of length `n`. You need to determine if you can find three indices `i`, `j`, and `k` that split the array into exactly 4 subarrays with equal sums.

The indices must satisfy these constraints:
- `0 < i < j < k < n - 1` (all indices are valid and in increasing order)
- There must be at least one element gap between consecutive indices: `i + 1 < j` and `j + 1 < k`
- The indices split the array into 4 parts with ranges: `[0, i-1]`, `[i+1, j-1]`, `[j+1, k-1]`, and `[k+1, n-1]`
- All four subarrays must have the same sum

The elements at positions `i`, `j`, and `k` themselves are not included in any of the four subarrays - they act as separators.

For example, if `nums = [1, 2, 1, 2, 1, 2, 1]` and we choose `i=1`, `j=3`, `k=5`:
- First subarray: `[0, 0]` → `[1]` with sum 1
- Second subarray: `[2, 2]` → `[1]` with sum 1
- Third subarray: `[4, 4]` → `[1]` with sum 1
- Fourth subarray: `[6, 6]` → `[1]` with sum 1

Since all four subarrays have equal sum (1), this would return `true`.

The function should return `true` if such a valid split exists, and `false` otherwise.

## Intuition

The key insight is that we need to find three indices that create four subarrays with equal sums. Instead of checking all possible combinations of `(i, j, k)` which would be `O(n³)`, we can optimize by fixing one index and cleverly using the equal sum property.

Let's think about what happens when we fix `j` (the middle index). If a valid split exists with this `j`, then:
- The sum of subarray `[0, i-1]` must equal the sum of subarray `[i+1, j-1]`
- The sum of subarray `[j+1, k-1]` must equal the sum of subarray `[k+1, n-1]`
- All four subarrays must have the same sum

This means if we denote the common sum as `target`, then:
- `sum[0, i-1] = target`
- `sum[i+1, j-1] = target`
- `sum[j+1, k-1] = target`
- `sum[k+1, n-1] = target`

Using prefix sums, we can rewrite the first two conditions. If `s[x]` represents the [prefix sum](/problems/subarray_sum) up to index `x-1`, then:
- `s[i] = target`
- `s[j] - s[i+1] = target`

From these two equations: `s[i] = s[j] - s[i+1]`, which means `2 * s[i] = s[j]`.

Similarly, for the last two subarrays:
- `s[k] - s[j+1] = target`
- `s[n] - s[k+1] = target`

This gives us: `s[k] - s[j+1] = s[n] - s[k+1]`, which means `2 * s[k] = s[n] + s[j+1]`.

The strategy becomes:
1. Fix `j` and find all valid `i` values where the first two subarrays have equal sum
2. Store these equal sum values in a set
3. Then find all valid `k` values where the last two subarrays have equal sum
4. If this sum matches any sum we found in step 2, we have found a valid split

This reduces the time complexity from `O(n³)` to `O(n²)` by avoiding the need to check all three indices simultaneously.

**Learn more about [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The implementation uses prefix sums and a hash set to efficiently find valid splits:

**Step 1: Build [Prefix Sum](/problems/subarray_sum) Array**
```python
s = [0] * (n + 1)
for i, v in enumerate(nums):
    s[i + 1] = s[i] + v
```
We create a prefix sum array where `s[i]` represents the sum of elements from index `0` to `i-1`. This allows us to calculate any subarray sum in `O(1)` time using the formula: `sum[l, r] = s[r+1] - s[l]`.

**Step 2: Fix Middle Index j**
```python
for j in range(3, n - 3):
```
We iterate through valid positions for `j`. The range `[3, n-3)` ensures:
- At least 1 element before `i` (so `i` can be at least 1)
- A gap between `i` and `j` (so `i` can be at most `j-2`)
- A gap between `j` and `k` (so `k` can be at least `j+2`)
- At least 1 element after `k` (so `k` can be at most `n-2`)

**Step 3: Find Valid i Values**
```python
seen = set()
for i in range(1, j - 1):
    if s[i] == s[j] - s[i + 1]:
        seen.add(s[i])
```
For each `j`, we find all valid `i` positions where:
- The sum of `[0, i-1]` equals the sum of `[i+1, j-1]`
- This condition translates to: `s[i] == s[j] - s[i+1]`

When this condition is met, we store the common sum value `s[i]` in a set called `seen`.

**Step 4: Find Valid k Values and Check**
```python
for k in range(j + 2, n - 1):
    if s[n] - s[k + 1] == s[k] - s[j + 1] and s[n] - s[k + 1] in seen:
        return True
```
For each valid `k` position:
- We check if the sum of `[j+1, k-1]` equals the sum of `[k+1, n-1]`
- This condition is: `s[k] - s[j+1] == s[n] - s[k+1]`
- If true, the common sum for these two subarrays is `s[n] - s[k+1]`
- We then check if this sum exists in our `seen` set
- If it does, all four subarrays have the same sum, so we return `True`

**Time Complexity:** `O(n²)` - For each of `O(n)` values of `j`, we check `O(n)` values of `i` and `k`.

**Space Complexity:** `O(n)` - For the [prefix sum](/problems/subarray_sum) array and the hash set.

## Example Walkthrough

Let's walk through the solution with `nums = [1, 2, 1, 2, 1, 2, 1]`.

**Step 1: Build prefix sum array**
```
nums: [1, 2, 1, 2, 1, 2, 1]
s:    [0, 1, 3, 4, 6, 7, 9, 10]
```
Where `s[i]` represents the sum of elements from index 0 to i-1.

**Step 2: Fix j = 3 (nums[3] = 2 is the separator)**

**Step 3: Find valid i values where first two subarrays have equal sum**

Check i = 1:
- First subarray [0, 0]: sum = s[1] = 1
- Second subarray [2, 2]: sum = s[3] - s[2] = 4 - 3 = 1
- Condition: s[1] == s[3] - s[2]? → 1 == 1 ✓
- Add 1 to seen set: `seen = {1}`

Check i = 2:
- First subarray [0, 1]: sum = s[2] = 3
- Second subarray would be empty (invalid since i+1 = 3 = j)
- Skip

**Step 4: Find valid k values and check against seen**

Check k = 5:
- Third subarray [4, 4]: sum = s[5] - s[4] = 7 - 6 = 1
- Fourth subarray [6, 6]: sum = s[7] - s[6] = 10 - 9 = 1
- Condition: s[5] - s[4] == s[7] - s[6]? → 1 == 1 ✓
- Common sum is 1, check if 1 ∈ seen? → Yes! ✓
- Return `True`

The split with i=1, j=3, k=5 creates four subarrays:
- [1] with sum 1
- [1] with sum 1
- [1] with sum 1
- [1] with sum 1

All four subarrays have equal sum, so the function returns `True`.

### Complexity Analysis

**Time Complexity:** O(n²)
- We iterate through O(n) possible values of j
- For each j, we check O(n) values of i and O(n) values of k
- Each check operation uses prefix sums and set lookup, both O(1)
- Total: O(n) × O(n) = O(n²)

**Space Complexity:** O(n)
- Prefix sum array requires O(n) space
- Hash set can store at most O(n) unique sums
- Total auxiliary space: O(n)

## Python Solution

```python
from typing import List

class Solution:
    def splitArray(self, nums: List[int]) -> bool:
        """
        Check if the array can be split into 4 subarrays with equal sums.
        The array is split at indices i, j, k where:
        - nums[0:i], nums[i+1:j], nums[j+1:k], nums[k+1:n] all have equal sums
        - Elements at indices i, j, k are excluded from the subarrays
        """
        n = len(nums)

        # Build prefix sum array for efficient range sum queries
        # prefix_sum[i] = sum of nums[0:i]
        prefix_sum = [0] * (n + 1)
        for index, value in enumerate(nums):
            prefix_sum[index + 1] = prefix_sum[index] + value

        # Fix middle split point j (must leave room for i before and k after)
        for j in range(3, n - 3):
            # Store valid sums from left side splits
            valid_left_sums = set()

            # Try all possible left split points i
            for i in range(1, j - 1):
                # Check if left two parts have equal sum
                left_part_sum = prefix_sum[i]  # sum of nums[0:i]
                middle_left_sum = prefix_sum[j] - prefix_sum[i + 1]  # sum of nums[i+1:j]

                if left_part_sum == middle_left_sum:
                    valid_left_sums.add(left_part_sum)

            # Try all possible right split points k
            for k in range(j + 2, n - 1):
                # Check if right two parts have equal sum
                middle_right_sum = prefix_sum[k] - prefix_sum[j + 1]  # sum of nums[j+1:k]
                right_part_sum = prefix_sum[n] - prefix_sum[k + 1]  # sum of nums[k+1:n]

                # If right parts are equal and match a valid left sum, we found a solution
                if right_part_sum == middle_right_sum and right_part_sum in valid_left_sums:
                    return True

        return False
```

## Time and Space Complexity

**Time Complexity:** `O(n²)`

The algorithm uses three nested iterations:
- The outer loop iterates through `j` from position 3 to `n-3`, which is `O(n)` iterations
- For each fixed `j`, there are two inner loops:
  - The first inner loop iterates through `i` from 1 to `j-2`, which is at most `O(n)` iterations
  - The second inner loop iterates through `k` from `j+2` to `n-2`, which is at most `O(n)` iterations
- Each operation inside the loops (calculating prefix sums differences, set operations) takes `O(1)` time

Since the two inner loops are sequential (not nested) for each value of `j`, the total time complexity is `O(n) × O(n) = O(n²)`.

**Space Complexity:** `O(n)`

The space usage consists of:
- The prefix sum array `s` of size `n+1`: `O(n)`
- The `seen` set which can store at most `O(n)` elements in the worst case (though typically much fewer)
- A few constant variables: `O(1)`

Therefore, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Index Boundary Confusion

**Pitfall:** One of the most common mistakes is misunderstanding the index boundaries and constraints. Developers often get confused about:
- Whether indices `i`, `j`, `k` are inclusive or exclusive in the subarrays
- The requirement for gaps between consecutive indices (`i+1 < j` and `j+1 < k`)
- The correct range calculations for each subarray

**Example of incorrect implementation:**
```python
# WRONG: This allows adjacent indices without gaps
for j in range(2, n - 2):  # Should be range(3, n - 3)
    for i in range(1, j):   # Should be range(1, j - 1) to ensure gap
        for k in range(j + 1, n - 1):  # Should be range(j + 2, n - 1) to ensure gap
```

**Solution:** Always verify that:
- `i` can range from 1 to `j-2` (ensuring at least one element gap before `j`)
- `j` can range from 3 to `n-4` (ensuring room for all constraints)
- `k` can range from `j+2` to `n-2` (ensuring at least one element gap after `j`)

### 2. Prefix Sum Calculation Errors

**Pitfall:** Incorrectly calculating subarray sums using prefix sums, especially when dealing with excluded elements at indices `i`, `j`, and `k`.

**Example of incorrect calculation:**
```python
# WRONG: Including the element at index i in the first subarray
first_sum = prefix_sum[i + 1]  # This includes nums[i]

# CORRECT: Excluding the element at index i
first_sum = prefix_sum[i]  # Sum of nums[0:i], excludes nums[i]
```

**Solution:** Remember that for a subarray from index `l` to `r` (inclusive), the sum is `prefix_sum[r + 1] - prefix_sum[l]`. When elements at split points are excluded:
- First subarray `[0, i-1]`: `prefix_sum[i] - prefix_sum[0] = prefix_sum[i]`
- Second subarray `[i+1, j-1]`: `prefix_sum[j] - prefix_sum[i + 1]`
- Third subarray `[j+1, k-1]`: `prefix_sum[k] - prefix_sum[j + 1]`
- Fourth subarray `[k+1, n-1]`: `prefix_sum[n] - prefix_sum[k + 1]`

### 3. Edge Case: Minimum Array Length

**Pitfall:** Not checking if the array is long enough to accommodate the split requirements. The minimum array length needed is 7.

**Why 7?** We need:
- At least 1 element in each of the 4 subarrays (4 elements)
- 3 separator elements at indices `i`, `j`, `k` (3 elements)
- Total minimum: 7 elements

**Example of missing edge case check:**
```python
def splitArray(self, nums: List[int]) -> bool:
    n = len(nums)
    # MISSING: Should check if n < 7 and return False early
    prefix_sum = [0] * (n + 1)
    # ... rest of the code
```

In this solution the loop `for j in range(3, n - 3)` is empty when `n < 7`, so the function already returns `False`. The pitfall matters for implementations that index into the array before looping, such as reading `prefix_sum[j + 2]` for a fixed `j`.

**Solution:** Add an early return for arrays that are too short, which also makes the requirement explicit:
```python
def splitArray(self, nums: List[int]) -> bool:
    n = len(nums)
    if n < 7:
        return False
    # ... rest of the code
```

### 4. Optimization Opportunity Missed

**Pitfall:** Not pruning the search space when the total sum makes equal splits impossible.

**Example scenario:** If the total sum minus the three separator elements cannot be divided evenly by 4, it's impossible to have four equal-sum subarrays.

**Solution:** While not strictly necessary for correctness, you can add an early optimization:
```python
def splitArray(self, nums: List[int]) -> bool:
    n = len(nums)
    if n < 7:
        return False

    total_sum = sum(nums)

    # For each combination of i, j, k, check if remaining sum is divisible by 4
    # This is more complex since separator values vary, but can be considered
    # in the inner loops for additional pruning
```

### 5. Hash Set Usage Error

**Pitfall:** Forgetting to clear or reinitialize the `valid_left_sums` set for each new value of `j`.

**Example of incorrect implementation:**
```python
valid_left_sums = set()  # WRONG: Declared outside the j loop
for j in range(3, n - 3):
    for i in range(1, j - 1):
        # ... adding to valid_left_sums
    # The set keeps accumulating values from previous j iterations!
```

**Solution:** Always declare the set inside the `j` loop to ensure it's fresh for each middle split point:
```python
for j in range(3, n - 3):
    valid_left_sums = set()  # CORRECT: Fresh set for each j
    for i in range(1, j - 1):
        # ... rest of the code
```

