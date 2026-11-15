# 2035. Partition Array Into Two Arrays to Minimize Sum Difference

> Source: https://algo.monster/liteproblems/2035

## Problem Statement

You have an integer array `nums` containing exactly `2 * n` integers. Your task is to split this array into two separate arrays, each containing exactly `n` integers. The goal is to minimize the absolute difference between the sums of these two arrays.

Here's what you need to do:
- Take the original array of `2 * n` integers
- Divide all elements into two groups of size `n` each
- Each element must go into exactly one of the two groups
- Calculate the sum of each group
- Find the arrangement that gives the smallest possible absolute difference between these two sums

For example, if `nums = [3, 9, 7, 3]` where `n = 2`, you could partition it as `[3, 9]` and `[7, 3]`. The sums would be `12` and `10`, giving an absolute difference of `|12 - 10| = 2`.

The function should return the minimum possible absolute difference you can achieve through any valid partitioning.

## Intuition

The key insight is that with `2n` elements, trying all possible ways to partition them into two groups of size `n` would require checking `C(2n, n)` combinations, which becomes computationally infeasible for larger values of `n`.

To make this tractable, we can split the problem in half. Instead of looking at the entire array at once, we divide `nums` into two halves - the first `n` elements and the last `n` elements. Now we can think about the problem differently: from the first half, we'll pick some elements for group A and the rest go to group B. Similarly, from the second half, we'll pick some elements for group A and the rest go to group B.

The crucial observation is that if we pick `i` elements from the first half for group A, then we must pick exactly `n - i` elements from the second half for group A (to ensure group A has exactly `n` elements total). This constraint significantly reduces our search space.

For each half of the array, we can precompute all possible "contributions" to the difference between the two groups. If an element goes to group A, it contributes positively to the difference; if it goes to group B, it contributes negatively. By storing these contributions grouped by how many elements we're selecting, we can efficiently combine results from both halves.

The meet-in-the-middle technique allows us to reduce the time complexity from `O(2^(2n))` to `O(2^n * n)`. For each possible contribution from the first half (with `i` elements selected), we look for the best matching contribution from the second half (with `n - i` elements selected) that minimizes the total absolute difference. [Binary search](/problems/binary_search_intro) helps us quickly find the optimal pairing.

This approach transforms an intractable problem into a manageable one by cleverly decomposing it and using precomputation to enable efficient combination of partial results.

**Learn more about [Two Pointers](/problems/two_pointers_intro), [Binary Search](/problems/binary_search_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The implementation uses a meet-in-the-middle strategy with the following steps:

**Step 1: Split and Precompute**

We divide the array into two halves of size `n` each. For each half, we generate all possible subset sums with their corresponding element counts:

```python
n = len(nums) >> 1  # n = len(nums) // 2
f = defaultdict(set)  # stores contributions from first half
g = defaultdict(set)  # stores contributions from second half
```

**Step 2: Generate All Possible Contributions**

For each possible subset (represented by [bitmask](/problems/bitmask_intro) `i` from `0` to `2^n - 1`):
- We iterate through all `n` positions
- If bit `j` is set in `i`, we add the element to our subset (contributes positively)
- If bit `j` is not set, we subtract the element (contributes negatively)

```python
for i in range(1 << n):  # iterate through all 2^n subsets
    s = cnt = 0          # s: contribution sum, cnt: number of selected elements
    s1 = cnt1 = 0        # for second half
    for j in range(n):
        if (i & (1 << j)) != 0:
            s += nums[j]         # add to group A
            cnt += 1
            s1 += nums[n + j]
            cnt1 += 1
        else:
            s -= nums[j]         # add to group B (negative contribution)
            s1 -= nums[n + j]
```

The dictionaries `f[cnt]` and `g[cnt]` store all possible contribution values when selecting exactly `cnt` elements from their respective halves.

**Step 3: Combine Results Using [Binary Search](/problems/binary_search_intro)**

For each possible split configuration:
- If we select `i` elements from the first half, we must select `n - i` from the second half
- We need to find pairs `(a, b)` where `a ∈ f[i]` and `b ∈ g[n-i]` that minimize `|a + b|`

```python
for i in range(n + 1):
    fi, gi = sorted(list(f[i])), sorted(list(g[n - i]))
    for a in fi:
        b = -a  # ideal target value to make sum zero
        # [Binary search](/problems/binary_search_intro) for value closest to b in gi
        left, right = 0, len(gi) - 1
        while left < right:
            mid = (left + right) >> 1
            if gi[mid] >= b:
                right = mid
            else:
                left = mid + 1
```

For each value `a` from the first half, we use [binary search](/problems/binary_search_intro) to find the value in the second half closest to `-a` (which would give us a sum close to zero). We check both `gi[left]` and `gi[left-1]` to ensure we find the minimum absolute difference.

**Time Complexity**: `O(2^n * n * log n)` where:
- `O(2^n * n)` for generating all subsets
- `O(2^n * log n)` for the [binary search](/problems/binary_search_intro) operations

**Space Complexity**: `O(2^n)` to store all possible subset sums for both halves.

## Example Walkthrough

Let's walk through the solution with `nums = [1, 2, 3, 4]` where `n = 2`.

**Step 1: Split the array**
- First half: `[1, 2]`
- Second half: `[3, 4]`

**Step 2: Generate all contributions for the first half `[1, 2]`**

For each subset, we calculate the contribution (elements in group A are positive, group B are negative):

| Bitmask | Selected for A | Selected for B | Contribution | Count |
|---------|---------------|----------------|--------------|-------|
| 00 | {} | {1, 2} | 0 - 1 - 2 = -3 | 0 |
| 01 | {1} | {2} | 1 - 2 = -1 | 1 |
| 10 | {2} | {1} | 2 - 1 = 1 | 1 |
| 11 | {1, 2} | {} | 1 + 2 = 3 | 2 |

So `f[0] = {-3}`, `f[1] = {-1, 1}`, `f[2] = {3}`

**Step 3: Generate all contributions for the second half `[3, 4]`**

| Bitmask | Selected for A | Selected for B | Contribution | Count |
|---------|---------------|----------------|--------------|-------|
| 00 | {} | {3, 4} | 0 - 3 - 4 = -7 | 0 |
| 01 | {3} | {4} | 3 - 4 = -1 | 1 |
| 10 | {4} | {3} | 4 - 3 = 1 | 1 |
| 11 | {3, 4} | {} | 3 + 4 = 7 | 2 |

So `g[0] = {-7}`, `g[1] = {-1, 1}`, `g[2] = {7}`

**Step 4: Combine results**

We need exactly `n = 2` elements in group A. So if we pick `i` from first half, we need `2 - i` from second half:

- **Case i = 0**: Pick 0 from first half, 2 from second half
  - From `f[0] = {-3}` and `g[2] = {7}`
  - Combination: -3 + 7 = 4
  - This represents: Group A = {3, 4}, Group B = {1, 2}
  - Sums: A = 7, B = 3, difference = |7 - 3| = 4

- **Case i = 1**: Pick 1 from first half, 1 from second half
  - From `f[1] = {-1, 1}` and `g[1] = {-1, 1}`
  - For a = -1: Find closest to -(-1) = 1 in g[1]. We have 1 exactly!
    - Combination: -1 + 1 = 0
    - This represents: Group A = {1, 4}, Group B = {2, 3}
    - Sums: A = 5, B = 5, difference = |5 - 5| = 0
  - For a = 1: Find closest to -1 in g[1]. We have -1 exactly!
    - Combination: 1 + (-1) = 0
    - This represents: Group A = {2, 3}, Group B = {1, 4}
    - Sums: A = 5, B = 5, difference = |5 - 5| = 0

- **Case i = 2**: Pick 2 from first half, 0 from second half
  - From `f[2] = {3}` and `g[0] = {-7}`
  - Combination: 3 + (-7) = -4
  - This represents: Group A = {1, 2}, Group B = {3, 4}
  - Sums: A = 3, B = 7, difference = |3 - 7| = 4

**Result**: The minimum absolute difference is **0**, achieved by partitioning into `{1, 4}` and `{2, 3}` (or equivalently `{2, 3}` and `{1, 4}`).

## Python Solution

```python
class Solution:
    def minimumDifference(self, nums: List[int]) -> int:
        # Split the array into two halves
        half_size = len(nums) // 2

        # Dictionary to store possible differences for each subset size in left half
        left_half_differences = defaultdict(set)
        # Dictionary to store possible differences for each subset size in right half
        right_half_differences = defaultdict(set)

        # Generate all possible subset combinations for the left half
        for mask in range(1 << half_size):
            difference = 0
            subset_size = 0

            # Calculate difference (sum of selected - sum of unselected) for left half
            for bit_position in range(half_size):
                if (mask & (1 << bit_position)) != 0:
                    # Element is selected for first subset
                    difference += nums[bit_position]
                    subset_size += 1
                else:
                    # Element is selected for second subset
                    difference -= nums[bit_position]

            left_half_differences[subset_size].add(difference)

        # Generate all possible subset combinations for the right half
        for mask in range(1 << half_size):
            difference = 0
            subset_size = 0

            # Calculate difference (sum of selected - sum of unselected) for right half
            for bit_position in range(half_size):
                if (mask & (1 << bit_position)) != 0:
                    # Element is selected for first subset
                    difference += nums[half_size + bit_position]
                    subset_size += 1
                else:
                    # Element is selected for second subset
                    difference -= nums[half_size + bit_position]

            right_half_differences[subset_size].add(difference)

        minimum_difference = float('inf')

        # Try all possible combinations where left contributes i elements to first subset
        # and right contributes (n-i) elements to first subset
        for left_subset_size in range(half_size + 1):
            right_subset_size = half_size - left_subset_size

            # Convert sets to sorted lists for binary search
            left_differences_sorted = sorted(list(left_half_differences[left_subset_size]))
            right_differences_sorted = sorted(list(right_half_differences[right_subset_size]))

            # For each difference in left half, find the closest matching difference in right half
            for left_diff in left_differences_sorted:
                # We want to find right_diff closest to -left_diff
                # so that left_diff + right_diff is minimized
                target = -left_diff

                # Binary search using template: find first index where value >= target
                left_idx = 0
                right_idx = len(right_differences_sorted) - 1
                first_true_index = len(right_differences_sorted)  # Default: not found

                while left_idx <= right_idx:
                    mid_index = (left_idx + right_idx) // 2
                    if right_differences_sorted[mid_index] >= target:
                        first_true_index = mid_index
                        right_idx = mid_index - 1
                    else:
                        left_idx = mid_index + 1

                # Check the found position and its neighbor for minimum difference
                if first_true_index < len(right_differences_sorted):
                    minimum_difference = min(minimum_difference,
                                           abs(left_diff + right_differences_sorted[first_true_index]))

                if first_true_index > 0:
                    minimum_difference = min(minimum_difference,
                                           abs(left_diff + right_differences_sorted[first_true_index - 1]))

        return minimum_difference
```

## Time and Space Complexity

**Time Complexity:** `O(2^n * n + n * 2^n * log(2^n))` which simplifies to `O(n * 2^n)`

The analysis breaks down as follows:
- The first loop iterates through all `2^n` possible subsets of the first half of the array. For each subset, it performs `n` operations to calculate sums and counts. This gives `O(2^n * n)`.
- The second part iterates through `n+1` possible values of `i`. For each `i`:
  - Sorting `f[i]` and `g[n-i]` takes `O(2^n * log(2^n))` in the worst case (when all elements are in one set)
  - For each element in `fi` (up to `2^n` elements), binary search is performed on `gi` which takes `O(log(2^n))` time
  - This gives `O(2^n * log(2^n))` for each value of `i`
- Total for the second part: `O(n * 2^n * log(2^n))` = `O(n * 2^n * n)` = `O(n^2 * 2^n)`
- The dominant term is `O(n^2 * 2^n)`, but since `n` is typically small compared to `2^n`, this can be simplified to `O(n * 2^n)`

**Space Complexity:** `O(2^n)`

The space analysis:
- The dictionaries `f` and `g` each store at most `n+1` keys
- For each key, the set can contain up to `C(n, k)` elements where `k` is the key value
- The total number of unique sums across all keys is bounded by `2^n` (total number of subsets)
- Each dictionary therefore uses `O(2^n)` space
- The sorted lists `fi` and `gi` use additional space but are bounded by `O(2^n)`
- Total space complexity: `O(2^n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Using the Wrong Binary Search Template Variant**

A common mistake is using `while left < right` with `right = mid` instead of the standard boundary-finding template. The correct template uses `while left <= right` with `right = mid - 1` and tracks the answer with `firstTrueIndex`.

**Incorrect Implementation:**
```python
while left < right:
    mid = (left + right) // 2
    if right_differences_sorted[mid] >= target:
        right = mid  # Wrong pattern
    else:
        left = mid + 1
# Uses left directly without tracking
minimum_difference = min(minimum_difference, abs(left_diff + right_differences_sorted[left]))
```

**Correct Implementation:**
```python
first_true_index = len(right_differences_sorted)  # Default if not found
while left <= right:
    mid = (left + right) // 2
    if right_differences_sorted[mid] >= target:
        first_true_index = mid  # Track the answer
        right = mid - 1
    else:
        left = mid + 1
# Check bounds before accessing
if first_true_index < len(right_differences_sorted):
    minimum_difference = min(minimum_difference, abs(left_diff + right_differences_sorted[first_true_index]))
```

### 2. **Incorrect Interpretation of Subset Contribution**

**Pitfall**: A common misunderstanding is thinking that the "difference" stored represents the sum of selected elements only. In reality, it represents `sum(selected) - sum(unselected)`, which is the contribution to the overall difference between the two partition sums.

**Why this matters**: If you only store sums of selected elements without considering the unselected ones, you can't correctly calculate the final difference between partitions.

**Solution**: Always calculate the contribution as:
```python
if element is selected for first subset:
    difference += element  # positive contribution
else:
    difference -= element  # negative contribution (goes to second subset)
```

### 2. **Off-by-One Error in Binary Search**

**Pitfall**: The binary search implementation can easily have off-by-one errors, especially when searching for the closest value rather than an exact match. A common mistake is:
```python
# Incorrect - might miss the optimal value
while left <= right:  # Using <= instead of <
    mid = (left + right) // 2
    if right_differences_sorted[mid] >= target:
        right = mid - 1  # This could skip the optimal value
    else:
        left = mid + 1
```

**Solution**: Use the correct binary search pattern for finding the leftmost position where value >= target:
```python
while left < right:
    mid = (left + right) // 2
    if right_differences_sorted[mid] >= target:
        right = mid  # Keep mid as a candidate
    else:
        left = mid + 1
# After loop, check both positions left and left-1
```

### 3. **Missing Edge Cases in Binary Search Result**

**Pitfall**: Only checking the position found by binary search without considering adjacent positions:
```python
# Incorrect - only checks one position
minimum_difference = min(minimum_difference,
                        abs(left_diff + right_differences_sorted[left_index]))
```

**Solution**: Always check both the found position and its neighbor:
```python
# Check the found position
minimum_difference = min(minimum_difference,
                        abs(left_diff + right_differences_sorted[left_index]))
# Also check the previous position if it exists
if left_index > 0:
    minimum_difference = min(minimum_difference,
                            abs(left_diff + right_differences_sorted[left_index - 1]))
```

### 4. **Incorrect Subset Size Pairing**

**Pitfall**: Forgetting that when selecting `i` elements from the left half for the first subset, you must select exactly `n - i` elements from the right half for the first subset (not `i` elements):
```python
# Incorrect pairing
for i in range(half_size + 1):
    left_diffs = left_half_differences[i]
    right_diffs = right_half_differences[i]  # Wrong! Should be half_size - i
```

**Solution**: Ensure complementary subset sizes:
```python
for left_subset_size in range(half_size + 1):
    right_subset_size = half_size - left_subset_size  # Complementary size
    left_diffs = left_half_differences[left_subset_size]
    right_diffs = right_half_differences[right_subset_size]
```

### 5. **Integer Overflow in Large Sums**

**Pitfall**: For arrays with large values, the sum calculations might overflow in languages with fixed integer sizes. While Python handles arbitrary precision integers, this could be an issue when porting to other languages.

**Solution**: In Python, this isn't an issue, but when implementing in other languages, consider:
- Using long/int64 data types
- Normalizing values by subtracting the mean from all elements first
- Working with differences directly rather than absolute sums

