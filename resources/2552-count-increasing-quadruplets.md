# 2552. Count Increasing Quadruplets

> Source: https://algo.monster/liteproblems/2552

## Problem Statement

You are given a 0-indexed integer array `nums` of size `n` that contains all numbers from `1` to `n`. Your task is to count the number of increasing quadruplets in this array.

A quadruplet `(i, j, k, l)` is considered "increasing" if it satisfies both of these conditions:
- The indices follow the order: `0 <= i < j < k < l < n`
- The values at these indices follow the specific pattern: `nums[i] < nums[k] < nums[j] < nums[l]`

Note that this is not a standard increasing sequence. The pattern requires that:
1. The value at index `i` is less than the value at index `k`
2. The value at index `k` is less than the value at index `j` 
3. The value at index `j` is less than the value at index `l`

This creates a "132-4" pattern where indices `j` and `k` are swapped in the value comparison compared to their positional order.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with preprocessing. It fixes the middle two indices `j` and `k`, then:
- Uses array `f[j][k]` to store how many indices `l > k` have `nums[l] > nums[j]`
- Uses array `g[j][k]` to store how many indices `i < j` have `nums[i] < nums[k]`

The total count of valid quadruplets is the sum of `f[j][k] * g[j][k]` for all valid pairs of `j` and `k`.

## Intuition

The key insight is recognizing that directly checking all possible quadruplets would require O(n^4) time, which is inefficient. Instead, we can observe that the pattern `nums[i] < nums[k] < nums[j] < nums[l]` has an interesting structure where the middle two elements `j` and `k` are "inverted" in their value relationship compared to their index positions.

This suggests we should fix the middle two indices `j` and `k` as our pivot points. Once we fix these two positions where `j < k` but `nums[j] > nums[k]`, we need to:
1. Find all valid `i` values to the left of `j` where `nums[i] < nums[k]`
2. Find all valid `l` values to the right of `k` where `nums[l] > nums[j]`

The number of valid quadruplets for a fixed `(j, k)` pair equals the product of valid `i` choices and valid `l` choices, since each `i` can be paired with each `l` independently.

The challenge is computing these counts efficiently. We notice that:
- When iterating through different `k` values for a fixed `j`, we can maintain a running count of how many elements to the right of `k` are greater than `nums[j]`
- Similarly, when iterating through different `j` values for a fixed `k`, we can maintain a running count of how many elements to the left of `j` are less than `nums[k]`

This leads to the preprocessing approach where we build two 2D arrays:
- `f[j][k]`: counts elements after position `k` that are greater than `nums[j]`
- `g[j][k]`: counts elements before position `j` that are less than `nums[k]`

By precomputing these values, we reduce the problem to O(n^2) time complexity, making it much more efficient than the naive approach.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The solution uses preprocessing with two 2D arrays to efficiently count the quadruplets. Let's walk through the implementation step by step:

**Step 1: Initialize the arrays**
- Create two `n × n` arrays `f` and `g` initialized with zeros
- `f[j][k]` will store the count of indices `l > k` where `nums[l] > nums[j]`
- `g[j][k]` will store the count of indices `i < j` where `nums[i] < nums[k]`

**Step 2: Build array `f`**
For each valid `j` position (from `1` to `n-3`):
- First, count all elements after `j` that are greater than `nums[j]`: `cnt = sum(nums[l] > nums[j] for l in range(j + 1, n))`
- Then iterate through each `k` position after `j` (from `j+1` to `n-2`):
  - If `nums[j] > nums[k]` (the required condition), set `f[j][k] = cnt`
  - Otherwise, decrement `cnt` by 1 (since `nums[k]` itself was counted but doesn't satisfy the condition anymore as we move `k` forward)

The key optimization here is maintaining a running count. As `k` increases, if `nums[k] >= nums[j]`, then `nums[k]` was previously counted in `cnt` but shouldn't be counted for positions after `k`, so we decrement it.

**Step 3: Build array `g`**
For each valid `k` position (from `2` to `n-2`):
- First, count all elements before `k` that are less than `nums[k]`: `cnt = sum(nums[i] < nums[k] for i in range(k))`
- Then iterate through each `j` position before `k` in reverse (from `k-1` down to `1`):
  - If `nums[j] > nums[k]` (the required condition), set `g[j][k] = cnt`
  - Otherwise, decrement `cnt` by 1 (since `nums[j]` itself was counted but doesn't satisfy the condition as we move `j` backward)

**Step 4: Calculate the final answer**
The total count of quadruplets is:
```python
sum(f[j][k] * g[j][k] for j in range(1, n - 2) for k in range(j + 1, n - 1))
```

For each valid `(j, k)` pair where `j < k` and `nums[j] > nums[k]`, multiply:
- The number of valid `l` values (stored in `f[j][k]`)
- The number of valid `i` values (stored in `g[j][k]`)

This gives us all possible quadruplets for that specific `(j, k)` pair.

**Time Complexity**: O(n²) for preprocessing both arrays and computing the final sum
**Space Complexity**: O(n²) for storing the two 2D arrays

## Example Walkthrough

Let's walk through the solution with a concrete example: `nums = [4, 1, 3, 2, 5]`

We need to find quadruplets `(i, j, k, l)` where `i < j < k < l` and `nums[i] < nums[k] < nums[j] < nums[l]`.

**Step 1: Initialize arrays**
- Create 5×5 arrays `f` and `g` (all zeros initially)

**Step 2: Build array `f[j][k]` (counts of `l > k` where `nums[l] > nums[j]`)**

For `j = 1` (`nums[1] = 1`):
- Count elements after j=1 that are > 1: positions 2,3,4 have values 3,2,5, all > 1, so `cnt = 3`
- For `k = 2` (`nums[2] = 3`): Since `nums[1]=1` NOT > `nums[2]=3`, skip and decrement `cnt` to 2
- For `k = 3` (`nums[3] = 2`): Since `nums[1]=1` NOT > `nums[3]=2`, skip and decrement `cnt` to 1

For `j = 2` (`nums[2] = 3`):
- Count elements after j=2 that are > 3: only position 4 has value 5 > 3, so `cnt = 1`
- For `k = 3` (`nums[3] = 2`): Since `nums[2]=3` > `nums[3]=2` ✓, set `f[2][3] = 1`

**Step 3: Build array `g[j][k]` (counts of `i < j` where `nums[i] < nums[k]`)**

For `k = 2` (`nums[2] = 3`):
- Count elements before k=2 that are < 3: positions 0,1 have values 4,1, only 1 < 3, so `cnt = 1`
- For `j = 1` (`nums[1] = 1`): Since `nums[1]=1` NOT > `nums[2]=3`, skip and decrement `cnt` to 0

For `k = 3` (`nums[3] = 2`):
- Count elements before k=3 that are < 2: only position 1 has value 1 < 2, so `cnt = 1`
- For `j = 2` (`nums[2] = 3`): Since `nums[2]=3` > `nums[3]=2` ✓, set `g[2][3] = 1`
- For `j = 1` (`nums[1] = 1`): Since `nums[1]=1` NOT > `nums[3]=2`, skip

**Step 4: Calculate total quadruplets**

Sum up `f[j][k] * g[j][k]` for all valid (j,k) pairs:
- Only `(j=2, k=3)` has both `f[2][3] = 1` and `g[2][3] = 1`
- Result: `1 * 1 = 1`

**Verification**: The quadruplet is `(i=1, j=2, k=3, l=4)` with values `(1, 3, 2, 5)`.
- Check: `nums[1]=1` < `nums[3]=2` < `nums[2]=3` < `nums[4]=5` ✓

The algorithm correctly finds 1 increasing quadruplet.

## Python Solution

```python
class Solution:
    def countQuadruplets(self, nums: List[int]) -> int:
        n = len(nums)
      
        # f[j][k] stores count of elements > nums[j] that come after k
        # where j < k and nums[j] > nums[k]
        greater_after = [[0] * n for _ in range(n)]
      
        # g[j][k] stores count of elements < nums[k] that come before j
        # where j < k and nums[j] > nums[k]
        less_before = [[0] * n for _ in range(n)]
      
        # Build greater_after matrix
        # For each j, count elements greater than nums[j] that appear after position k
        for j in range(1, n - 2):
            # Initially count all elements after j that are greater than nums[j]
            count = sum(nums[l] > nums[j] for l in range(j + 1, n))
          
            for k in range(j + 1, n - 1):
                if nums[j] > nums[k]:
                    # If nums[j] > nums[k], store the count
                    greater_after[j][k] = count
                else:
                    # If nums[j] <= nums[k], nums[k] was counted but shouldn't be
                    # Decrement count as we move forward
                    count -= 1
      
        # Build less_before matrix
        # For each k, count elements less than nums[k] that appear before position j
        for k in range(2, n - 1):
            # Initially count all elements before k that are less than nums[k]
            count = sum(nums[i] < nums[k] for i in range(k))
          
            for j in range(k - 1, 0, -1):
                if nums[j] > nums[k]:
                    # If nums[j] > nums[k], store the count
                    less_before[j][k] = count
                else:
                    # If nums[j] <= nums[k], nums[j] was counted but shouldn't be
                    # Decrement count as we move backward
                    count -= 1
      
        # Calculate total quadruplets
        # For each valid (j, k) pair where nums[j] > nums[k]:
        # - greater_after[j][k] gives count of valid l positions
        # - less_before[j][k] gives count of valid i positions
        # Multiply them to get all valid (i, j, k, l) quadruplets
        total = sum(
            greater_after[j][k] * less_before[j][k] 
            for j in range(1, n - 2) 
            for k in range(j + 1, n - 1)
        )
      
        return total
```

## Time and Space Complexity

The time complexity is `O(n²)` and the space complexity is `O(n²)`, where `n` is the length of the array.

**Time Complexity Analysis:**
- The algorithm uses two main preprocessing loops:
  - First loop: For each `j` from `1` to `n-2`, we iterate through `k` from `j+1` to `n-1`. The initial counting `sum(nums[l] > nums[j] for l in range(j + 1, n))` takes `O(n)` time, but this is done once per `j`. The inner loop runs `O(n)` times with `O(1)` operations each. Total for this section: `O(n²)`
  - Second loop: For each `k` from `2` to `n-1`, we iterate through `j` from `k-1` down to `1`. Similar to the first loop, the initial counting takes `O(n)` time once per `k`, and the inner loop runs `O(n)` times with `O(1)` operations. Total for this section: `O(n²)`
- The final summation iterates through all valid `(j, k)` pairs where `j` ranges from `1` to `n-2` and `k` ranges from `j+1` to `n-1`, which is `O(n²)` pairs with `O(1)` work per pair.
- Overall time complexity: `O(n²) + O(n²) + O(n²) = O(n²)`

**Space Complexity Analysis:**
- Two 2D arrays `f` and `g` are created, each of size `n × n`
- Total space used: `2 × n² = O(n²)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Count Updates When Building the Arrays**

A critical pitfall is misunderstanding when to decrement the count while building the `f` and `g` arrays. Many developers make the mistake of decrementing the count in the wrong condition.

**Incorrect Implementation:**
```python
# WRONG: Decrementing when nums[j] > nums[k]
for k in range(j + 1, n - 1):
    if nums[j] > nums[k]:
        greater_after[j][k] = count
        count -= 1  # WRONG: Should not decrement here
```

**Why it's wrong:** When `nums[j] > nums[k]`, we want to keep all elements greater than `nums[j]` in our count. We only decrement when `nums[k] >= nums[j]` because `nums[k]` itself was initially counted but won't be available for future `k` positions.

**Correct Implementation:**
```python
for k in range(j + 1, n - 1):
    if nums[j] > nums[k]:
        greater_after[j][k] = count
    else:
        # Only decrement when nums[k] >= nums[j]
        count -= 1
```

### 2. **Off-by-One Errors in Loop Boundaries**

The loop boundaries are crucial for this problem. Common mistakes include:

**Incorrect Loop Ranges:**
```python
# WRONG: Missing valid positions
for j in range(n - 2):  # Should start from 1, not 0
    count = sum(nums[l] > nums[j] for l in range(j + 1, n))
    for k in range(j + 1, n):  # Should end at n-1, not n
```

**Correct Loop Ranges:**
```python
# j must be at least index 1 (need room for i before it)
# j must be at most n-3 (need room for k and l after it)
for j in range(1, n - 2):
    count = sum(nums[l] > nums[j] for l in range(j + 1, n))
    # k must be at most n-2 (need room for l after it)
    for k in range(j + 1, n - 1):
```

### 3. **Confusing the Pattern Requirements**

The pattern `nums[i] < nums[k] < nums[j] < nums[l]` is NOT a standard increasing sequence. It's easy to mistakenly check for `nums[i] < nums[j] < nums[k] < nums[l]`.

**Wrong Understanding:**
```python
# WRONG: Checking standard increasing order
if nums[i] < nums[j] and nums[j] < nums[k] and nums[k] < nums[l]:
    count += 1
```

**Correct Understanding:**
The pattern specifically requires:
- `nums[i] < nums[k]` (comparing positions i and k)
- `nums[k] < nums[j]` (k value is less than j value, despite k > j in position)
- `nums[j] < nums[l]` (comparing positions j and l)

### 4. **Inefficient Initial Count Calculation**

While the provided solution is correct, a common inefficiency is recalculating the initial count from scratch for every j or k value:

**Less Efficient:**
```python
for j in range(1, n - 2):
    # Recalculating from scratch every time
    count = sum(nums[l] > nums[j] for l in range(j + 1, n))
```

**More Efficient Alternative:**
```python
# Pre-compute suffix counts for all positions
suffix_greater = [0] * n
for j in range(n - 2, -1, -1):
    suffix_greater[j] = sum(1 for l in range(j + 1, n) if nums[l] > nums[j])
```

### 5. **Memory Optimization Overlooked**

The solution uses O(n²) space for two 2D arrays. For very large inputs, this might cause memory issues. Consider that many entries in the arrays remain zero (when `nums[j] <= nums[k]`).

**Space-Optimized Alternative Using Dictionary:**
```python
greater_after = {}  # Use sparse representation
less_before = {}

# Only store non-zero values
if nums[j] > nums[k]:
    greater_after[(j, k)] = count
  
# Later when calculating result
total = sum(
    greater_after.get((j, k), 0) * less_before.get((j, k), 0)
    for j in range(1, n - 2)
    for k in range(j + 1, n - 1)
)
```

This optimization is particularly useful when the array has many positions where `nums[j] <= nums[k]`, resulting in sparse matrices.

