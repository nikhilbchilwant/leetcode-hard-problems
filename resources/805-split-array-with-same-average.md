# 805. Split Array With Same Average

> Source: https://algo.monster/liteproblems/805

## Problem Statement

You are given an integer array `nums`. Your task is to split all elements of `nums` into two non-empty arrays `A` and `B` such that both arrays have the same average value.

The average of an array is calculated as the sum of all its elements divided by the number of elements in the array.

You need to return `true` if it's possible to achieve such a split where `average(A) == average(B)`, and `false` otherwise.

**Key Requirements:**
- Every element from `nums` must be placed into either array `A` or array `B`
- Both `A` and `B` must be non-empty (contain at least one element)
- The average values of `A` and `B` must be exactly equal

**Example scenarios:**
- If `nums = [1,2,3,4,5,6,7,8]`, one possible split could be `A = [1,4,5,8]` and `B = [2,3,6,7]`, where both arrays have an average of 4.5
- If the array cannot be split to satisfy the conditions, return `false`

The challenge lies in finding whether such a partition exists among all possible ways to divide the array elements.

## Intuition

Let's think about what it means for two arrays to have the same average. If array `A` has sum `s1` and `k` elements, and array `B` has sum `s2` and `n-k` elements (where `n` is the total number of elements), then:

`average(A) = s1/k` and `average(B) = s2/(n-k)`

For these to be equal: `s1/k = s2/(n-k)`

Since `s2 = s - s1` (where `s` is the total sum), we can substitute:
`s1/k = (s-s1)/(n-k)`

Cross-multiplying and simplifying, we get:
`s1 * (n-k) = (s-s1) * k`
`s1 * n = s * k`
`s1/k = s/n`

This is a key insight! It means array `A`'s average must equal the overall array's average. In other words, we're looking for a subset whose average equals the total average.

Now, here's where it gets clever. Instead of dealing with averages (which might be fractions), we can transform the problem. If we subtract the overall average from each element, we're essentially asking: "Can we find a subset that sums to 0?"

But wait - the average `s/n` might not be an integer, leading to floating-point issues. To avoid this, we multiply each element by `n` first, giving us `nums[i] * n`. Now our equation becomes `(s1 * n)/k = s`, and after subtracting `s` from each transformed element, we get `nums[i] * n - s`.

With this transformation, finding a subset with the original average is equivalent to finding a subset that sums to 0 in our transformed array.

The final challenge is efficiency. With up to 30 elements, checking all `2^30` subsets would be too slow. This is where "meet in the middle" comes in - we split the array in half, enumerate all possible sums from each half (only `2^15` each), and check if sums from the left and right halves can combine to give 0. This reduces our time complexity from `O(2^n)` to `O(2^(n/2))`.

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

Let's implement the solution step by step based on our intuition. The snippets use short names; in the full solution, `n` is `array_length`, `s` is `total_sum`, `m` is `mid_point`, `vis` is `first_half_sums`, and `t` is `subset_sum`.

**Step 1: Handle Edge Case**
```python
if n == 1:
    return False
```
If there's only one element, we can't split it into two non-empty arrays.

**Step 2: Transform the Array**
```python
s = sum(nums)
for i, v in enumerate(nums):
    nums[i] = v * n - s
```
We transform each element by multiplying it by `n` and subtracting the total sum `s`. This converts our problem to finding a subset that sums to 0.

**Step 3: Split the Array for Meet-in-the-Middle**
```python
m = n >> 1  # m = n // 2
```
We divide the array into two halves - the left half has `m` elements, and the right half has `n-m` elements.

**Step 4: Enumerate All Subset Sums from the Left Half**
```python
vis = set()
for i in range(1, 1 << m):
    t = sum(v for j, v in enumerate(nums[:m]) if i >> j & 1)
    if t == 0:
        return True
    vis.add(t)
```
- We use bit manipulation to enumerate all non-empty subsets of the left half
- `i` ranges from `1` to `2^m - 1`, representing all possible subsets
- For each bit position `j` in `i`, if the bit is set (`i >> j & 1`), we include `nums[j]` in the subset
- If we find a subset sum of 0, we immediately return `True`
- Otherwise, we store all subset sums in the hash set `vis`

**Step 5: Enumerate All Subset Sums from the Right Half**
```python
for i in range(1, 1 << (n - m)):
    t = sum(v for j, v in enumerate(nums[m:]) if i >> j & 1)
    if t == 0 or (i != (1 << (n - m)) - 1 and -t in vis):
        return True
```
- Similarly, we enumerate all non-empty subsets of the right half
- If we find a subset sum of 0, return `True`
- Otherwise, check if `-t` exists in `vis`. If it does, it means we can combine a subset from the left (with sum `-t`) and a subset from the right (with sum `t`) to get a total sum of 0
- **Important**: The condition `i != (1 << (n - m)) - 1` ensures we don't select all elements from the right half. This is because if we select all elements from both halves, array `B` would be empty, violating the problem constraints

**Step 6: Return False if No Valid Split Found**
```python
return False
```

**Time Complexity**: `O(2^(n/2) * n)` - We enumerate `2^(n/2)` subsets for each half, and calculating each subset sum takes `O(n)` time.

**Space Complexity**: `O(2^(n/2))` - The hash set `vis` stores at most `2^(n/2)` subset sums.

## Example Walkthrough

Let's walk through a small example with `nums = [1, 2, 3, 4]`.

**Step 1: Initial Setup**
- Total sum `s = 1 + 2 + 3 + 4 = 10`
- Number of elements `n = 4`
- Overall average = `10/4 = 2.5`

**Step 2: Transform the Array**
We transform each element: `nums[i] * n - s`
- `1 * 4 - 10 = -6`
- `2 * 4 - 10 = -2`
- `3 * 4 - 10 = 2`
- `4 * 4 - 10 = 6`

Transformed array: `[-6, -2, 2, 6]`

**Step 3: Split for Meet-in-the-Middle**
- Left half: `[-6, -2]` (indices 0, 1)
- Right half: `[2, 6]` (indices 2, 3)

**Step 4: Enumerate Left Half Subsets**
Using bit manipulation (where bit position indicates element inclusion):
- `i = 1` (binary: 01): Include element 0 → subset `[-6]`, sum = `-6`
- `i = 2` (binary: 10): Include element 1 → subset `[-2]`, sum = `-2`
- `i = 3` (binary: 11): Include elements 0,1 → subset `[-6, -2]`, sum = `-8`

Store in hash set: `vis = {-6, -2, -8}`

**Step 5: Enumerate Right Half Subsets**
- `i = 1` (binary: 01): Include element 0 → subset `[2]`, sum = `2`
  - Check if `-2` is in `vis`: YES! This means we found a match.
  - Left subset with sum `-2` is `[-2]` (original element `2`)
  - Right subset with sum `2` is `[2]` (original element `3`)
  - Combined sum = `-2 + 2 = 0` ✓

**Step 6: Verify the Original Split**
- Array A: elements that contributed to sum 0 → `[2, 3]`
- Array B: remaining elements → `[1, 4]`
- Average of A: `(2 + 3) / 2 = 2.5`
- Average of B: `(1 + 4) / 2 = 2.5`

Both averages equal 2.5, so we return `true`!

The key insight: By transforming the array, we converted the "equal average" problem into a "subset sum to 0" problem, which we efficiently solved using meet-in-the-middle to avoid checking all 2^n possible subsets.

## Python Solution

```python
class Solution:
    def splitArraySameAverage(self, nums: List[int]) -> bool:
        # Get array length
        array_length = len(nums)
      
        # Edge case: cannot split array of size 1
        if array_length == 1:
            return False
      
        # Transform the problem: if we can split into two sets A and B where
        # sum(A)/len(A) = sum(B)/len(B) = total_sum/array_length
        # This is equivalent to: sum(A) * array_length = len(A) * total_sum
        # Rearranging: sum(A * array_length - total_sum) = 0 for each element in A
        total_sum = sum(nums)
        for index, value in enumerate(nums):
            nums[index] = value * array_length - total_sum
      
        # Split array into two halves for meet-in-the-middle approach
        mid_point = array_length >> 1  # equivalent to array_length // 2
      
        # Store all possible sums from the first half
        first_half_sums = set()
      
        # Generate all non-empty subsets of the first half using bit manipulation
        for subset_mask in range(1, 1 << mid_point):
            # Calculate sum of elements in current subset
            subset_sum = sum(
                value 
                for bit_position, value in enumerate(nums[:mid_point]) 
                if subset_mask >> bit_position & 1
            )
          
            # If we found a subset with sum 0, we have a valid split
            if subset_sum == 0:
                return True
          
            # Store this sum for later checking
            first_half_sums.add(subset_sum)
      
        # Generate all non-empty subsets of the second half
        second_half_size = array_length - mid_point
        for subset_mask in range(1, 1 << second_half_size):
            # Calculate sum of elements in current subset
            subset_sum = sum(
                value 
                for bit_position, value in enumerate(nums[mid_point:]) 
                if subset_mask >> bit_position & 1
            )
          
            # Check if this subset alone sums to 0
            if subset_sum == 0:
                return True
          
            # Check if we can combine with a subset from first half to get sum 0
            # But avoid the case where we select all elements (no valid split)
            if subset_mask != (1 << second_half_size) - 1 and -subset_sum in first_half_sums:
                return True
      
        # No valid split found
        return False
```

## Time and Space Complexity

**Time Complexity:** `O(n × 2^(n/2))`

The algorithm uses a meet-in-the-middle approach by splitting the array into two halves. For the first half of size `m = n/2`, it generates all possible subsets using bit manipulation from `1` to `2^m - 1`, which takes `O(2^(n/2))` iterations. For each subset, it calculates the sum by iterating through at most `m` elements, contributing `O(n/2)` operations per subset. Similarly, for the second half of size `n - m`, it generates all subsets from `1` to `2^(n-m) - 1`, which is also `O(2^(n/2))` iterations, with each subset sum calculation taking `O(n/2)` time. The initial transformation of the array takes `O(n)` time. Therefore, the overall time complexity is `O(n × 2^(n/2))`.

**Space Complexity:** `O(2^(n/2))`

The algorithm stores all possible subset sums from the first half in the set `first_half_sums`. Since there are at most `2^(n/2) - 1` non-empty subsets in the first half, the space required for the set is `O(2^(n/2))`. The input array modification is done in-place, requiring no additional space. Thus, the total space complexity is `O(2^(n/2))`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Check for Empty Array B**

**The Pitfall:**
When using the meet-in-the-middle approach, a critical edge case occurs when we select all elements from both halves. This would mean array A contains all elements from `nums`, leaving array B empty, which violates the constraint that both arrays must be non-empty.

**Why It Happens:**
In the second loop, when `subset_mask = (1 << second_half_size) - 1`, all bits are set to 1, meaning we're selecting all elements from the second half. If we find a matching negative sum in `first_half_sums`, we might incorrectly return `True` even though this would result in selecting all elements for array A.

**The Bug:**
```python
# Incorrect version - missing the check
if -subset_sum in first_half_sums:  # This could select all elements!
    return True
```

**The Solution:**
```python
# Correct version - ensures we don't select all elements
if subset_mask != (1 << second_half_size) - 1 and -subset_sum in first_half_sums:
    return True
```

### 2. **Integer Overflow in Transformation**

**The Pitfall:**
The transformation `value * array_length - total_sum` multiplies every element by `n`, and subset sums of the transformed values grow further. In languages with fixed-size integers it is worth checking that these values fit.

**Example:**
Under the LeetCode constraints (`1 <= nums.length <= 30`, `0 <= nums[i] <= 10^4`), each transformed value lies between `-300000` and `300000`, and a subset sum of up to 15 such values stays within `±4,500,000`, so 32-bit `int` is enough. If the constraints were larger (for example values near `10^9`), `value * array_length` alone would exceed the 32-bit range.

**The Solution:**
In Python, this isn't an issue due to arbitrary precision integers. In other languages, check the bounds implied by the constraints and switch to long/int64 when the products or subset sums can exceed the 32-bit range.

### 3. **Incorrect Subset Enumeration**

**The Pitfall:**
Using incorrect bit manipulation logic when enumerating subsets can lead to missing valid combinations or including invalid ones.

**Common Mistakes:**
```python
# Wrong: Starting from 0 includes empty subset
for subset_mask in range(0, 1 << mid_point):  # Includes empty set!

# Wrong: Incorrect bit checking (shifts the wrong operand)
if bit_position >> subset_mask & 1:  # Should be: subset_mask >> bit_position & 1
```

Starting from 0 adds the empty subset, whose sum is 0, so the first loop returns `True` immediately for every input.

**The Solution:**
Always start from 1 to exclude empty subsets and test bit `bit_position` of the mask (`subset_mask >> bit_position & 1` and `subset_mask & (1 << bit_position)` are equivalent ways to do this):
```python
for subset_mask in range(1, 1 << mid_point):  # Start from 1
    if subset_mask >> bit_position & 1:  # Correct bit checking
```

### 4. **Missing Early Termination Opportunities**

**The Pitfall:**
Not checking for impossible cases early can lead to unnecessary computation.

**Example:**
If the total sum cannot be evenly distributed (considering the mathematical constraints), we should return `False` immediately rather than performing expensive subset enumeration.

**The Solution:**
Add mathematical feasibility checks before the main algorithm:
```python
# Check if it's mathematically possible
# For a valid split with k elements in array A:
# sum(A)/k = total_sum/n
# This means sum(A) = k * total_sum / n must be an integer
for k in range(1, n):
    if (k * total_sum) % n == 0:
        # At least one valid k exists, proceed with algorithm
        break
else:
    return False  # No valid k exists
```

