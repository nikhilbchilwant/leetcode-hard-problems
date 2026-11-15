# 446. Arithmetic Slices II - Subsequence

> Source: https://algo.monster/liteproblems/446

## Problem Statement

This problem asks you to count the total number of arithmetic subsequences in an integer array `nums`.

An **arithmetic sequence** must satisfy two conditions:
1. It contains at least 3 elements
2. The difference between any two consecutive elements is the same (constant difference)

For example:
- `[1, 3, 5, 7, 9]` is arithmetic with difference 2
- `[7, 7, 7, 7]` is arithmetic with difference 0  
- `[3, -1, -5, -9]` is arithmetic with difference -4
- `[1, 1, 2, 5, 7]` is NOT arithmetic (differences are 0, 1, 3, 2)

A **subsequence** is formed by selecting elements from the original array while maintaining their relative order. You can skip elements but cannot rearrange them.

For example, from array `[1, 2, 1, 2, 4, 1, 5, 10]`:
- `[2, 5, 10]` is a valid subsequence (taking 2nd, 7th, and 8th elements)

The solution uses dynamic programming with a dictionary at each position. For each index `i`, `f[i][d]` stores the count of arithmetic subsequences ending at position `i` with common difference `d`. 

The algorithm works by:
1. For each position `i` and element `x`
2. Looking at all previous positions `j` with element `y`
3. Calculating the difference `d = x - y`
4. Adding to the answer the number of subsequences ending at `j` with difference `d` (these can be extended by `x` to form valid arithmetic subsequences)
5. Updating `f[i][d]` to include all subsequences that can be formed by extending those from position `j`, plus the new 2-element subsequence `[y, x]`

The final answer is the sum of all valid arithmetic subsequences found.

## Intuition

The key insight is that we need to track arithmetic subsequences as we build them incrementally. When we're at position `i`, we want to know: what arithmetic subsequences can we form that end at this position?

Think about it this way: if we have an arithmetic subsequence ending at position `j` with difference `d`, and we find that `nums[i] - nums[j] = d`, then we can extend that subsequence by adding `nums[i]`. This creates a new arithmetic subsequence ending at position `i`.

But here's the crucial observation: we need to track subsequences of length 2 as well (even though they don't count as arithmetic subsequences yet) because they can become valid arithmetic subsequences when extended by a third element.

For example, if we have `[1, 3]` with difference 2, it's not yet an arithmetic subsequence. But if we later encounter 5, we can form `[1, 3, 5]` which IS valid.

This leads us to use dynamic programming where `f[i][d]` represents the number of arithmetic subsequences (including 2-element ones) ending at index `i` with common difference `d`.

When processing element at index `i`:
- We look at all previous elements at index `j`
- Calculate the difference `d = nums[i] - nums[j]`
- Any subsequence ending at `j` with difference `d` can be extended to `i`
- Those that were already length ≥2 become valid arithmetic subsequences when extended (so we add `f[j][d]` to our answer)
- We also form a new 2-element subsequence `[nums[j], nums[i]]` (the "+1" in the update)

The beauty of this approach is that it naturally handles the "at least 3 elements" requirement: only when we extend a 2+ element subsequence do we count it in our answer, ensuring all counted subsequences have at least 3 elements.

## Solution Approach

The solution uses dynamic programming with hash maps to efficiently track arithmetic subsequences.

**Data Structure:**
- `f`: A list of dictionaries where `f[i]` is a dictionary storing counts of subsequences ending at index `i`
- Each dictionary `f[i]` has keys as differences `d` and values as counts of subsequences with that difference

**Algorithm Steps:**

1. **Initialize**: Create a list `f` where each element is an empty `defaultdict(int)`. This allows us to store counts for different differences at each position.

2. **Iterate through each position** `i` with element `x`:
   ```python
   for i, x in enumerate(nums):
   ```

3. **For each position `i`, check all previous positions** `j` with element `y`:
   ```python
   for j, y in enumerate(nums[:i]):
   ```

4. **Calculate the difference** between current and previous element:
   ```python
   d = x - y
   ```

5. **Count valid arithmetic subsequences**: 
   ```python
   ans += f[j][d]
   ```
   This is the key step - if there are `f[j][d]` subsequences ending at `j` with difference `d`, they can all be extended by adding element `x` to form valid arithmetic subsequences of length ≥3.

6. **Update the DP state**:
   ```python
   f[i][d] += f[j][d] + 1
   ```
   - `f[j][d]`: Represents all subsequences ending at `j` with difference `d` that can be extended to `i`
   - `+1`: Represents the new 2-element subsequence `[nums[j], nums[i]]`

**Example Walkthrough:**

For `nums = [2, 4, 6, 8]`:
- At `i=1` (value 4): Form `[2,4]` with difference 2
- At `i=2` (value 6): 
  - Extend `[2,4]` to `[2,4,6]` (count this!)
  - Form `[4,6]` with difference 2
- At `i=3` (value 8):
  - Extend `[2,4,6]` to `[2,4,6,8]` (count this!)
  - Extend `[4,6]` to `[4,6,8]` (count this!)
  - Form `[6,8]` with difference 2

**Time Complexity:** `O(n²)` where n is the length of nums, as we have nested loops.

**Space Complexity:** `O(n²)` in worst case, as each position can store up to n different differences.

## Example Walkthrough

Let's walk through `nums = [1, 3, 5, 7]` step by step to see how the algorithm counts arithmetic subsequences.

**Initial Setup:**
- `f = [{}, {}, {}, {}]` (list of empty dictionaries)
- `ans = 0`

**Step 1: i=0, x=1**
- No previous elements to check
- `f[0]` remains empty: `{}`

**Step 2: i=1, x=3**
- Check j=0, y=1:
  - `d = 3 - 1 = 2`
  - `ans += f[0][2] = 0` (no subsequences at position 0 with difference 2)
  - `f[1][2] = f[0][2] + 1 = 0 + 1 = 1` (creates 2-element subsequence [1,3])
- State: `f[1] = {2: 1}`, `ans = 0`

**Step 3: i=2, x=5**
- Check j=0, y=1:
  - `d = 5 - 1 = 4`
  - `ans += f[0][4] = 0`
  - `f[2][4] = 0 + 1 = 1` (creates [1,5])
- Check j=1, y=3:
  - `d = 5 - 3 = 2`
  - `ans += f[1][2] = 1` ✓ (extending [1,3] to [1,3,5] creates our first valid subsequence!)
  - `f[2][2] = f[1][2] + 1 = 1 + 1 = 2` (one from extending [1,3], one new [3,5])
- State: `f[2] = {4: 1, 2: 2}`, `ans = 1`

**Step 4: i=3, x=7**
- Check j=0, y=1:
  - `d = 7 - 1 = 6`
  - `ans += f[0][6] = 0`
  - `f[3][6] = 0 + 1 = 1` (creates [1,7])
- Check j=1, y=3:
  - `d = 7 - 3 = 4`
  - `ans += f[1][4] = 0`
  - `f[3][4] = 0 + 1 = 1` (creates [3,7])
- Check j=2, y=5:
  - `d = 7 - 5 = 2`
  - `ans += f[2][2] = 2` ✓ (two valid extensions: [1,3,5] → [1,3,5,7] and [3,5] → [3,5,7])
  - `f[3][2] = f[2][2] + 1 = 2 + 1 = 3`
- State: `f[3] = {6: 1, 4: 1, 2: 3}`, `ans = 3`

**Final Answer: 3**

The three arithmetic subsequences found are:
1. `[1, 3, 5]` (difference 2)
2. `[1, 3, 5, 7]` (difference 2)
3. `[3, 5, 7]` (difference 2)

The key insight: when we're at position i and find that `nums[i] - nums[j] = d`, the value `f[j][d]` tells us exactly how many subsequences ending at j can be extended to form valid arithmetic subsequences. This is why we add `f[j][d]` to our answer - each represents a newly formed arithmetic subsequence of length ≥3.

## Python Solution

```python
from typing import List
from collections import defaultdict

class Solution:
    def numberOfArithmeticSlices(self, nums: List[int]) -> int:
        # dp[i][diff] = number of arithmetic subsequences ending at index i with difference diff
        # These subsequences have at least 2 elements
        dp = [defaultdict(int) for _ in nums]
      
        # Total count of arithmetic subsequences with at least 3 elements
        total_count = 0
      
        # Iterate through each element as potential end of subsequence
        for i, current_num in enumerate(nums):
            # Check all previous elements as potential second-to-last element
            for j, previous_num in enumerate(nums[:i]):
                # Calculate the common difference
                difference = current_num - previous_num
              
                # Add count of existing subsequences that can be extended
                # dp[j][difference] represents subsequences ending at j with this difference
                # Each of these becomes a valid arithmetic slice when extended with nums[i]
                total_count += dp[j][difference]
              
                # Update dp for position i
                # dp[j][difference] subsequences can be extended with current element
                # Plus 1 for the new 2-element subsequence [nums[j], nums[i]]
                dp[i][difference] += dp[j][difference] + 1
      
        return total_count
```

## Time and Space Complexity

**Time Complexity:** `O(n²)`

The algorithm uses two nested loops:
- The outer loop iterates through all elements in `nums`, running `n` times
- The inner loop iterates through all elements before index `i`, which on average runs `i/2` times
- The total number of iterations is `1 + 2 + 3 + ... + (n-1) = n(n-1)/2`
- Inside the nested loops, all operations (dictionary access, arithmetic operations, and updates) take `O(1)` time
- Therefore, the overall time complexity is `O(n²)`

**Space Complexity:** `O(n²)`

The space usage comes from:
- The list `f` contains `n` dictionaries, one for each element in `nums`
- Each dictionary `f[i]` can potentially store up to `i` different arithmetic differences (one for each previous element)
- In the worst case where all differences are unique, the total space used is `0 + 1 + 2 + ... + (n-1) = n(n-1)/2`
- This results in `O(n²)` space complexity for storing all the difference counts
- Additional space usage includes the loop variables and temporary variables, which are `O(1)`
- Therefore, the overall space complexity is `O(n²)`

## Common Pitfalls

### 1. **Confusing Subsequences with Subarrays**
A common mistake is treating this problem as counting arithmetic subarrays (contiguous elements) rather than subsequences (non-contiguous elements that maintain order).

**Wrong Approach:**
```python
# This only counts contiguous arithmetic sequences
def countArithmeticSubarrays(nums):
    count = 0
    for i in range(len(nums) - 2):
        diff = nums[i+1] - nums[i]
        for j in range(i+2, len(nums)):
            if nums[j] - nums[j-1] == diff:
                count += 1
            else:
                break
    return count
```

**Solution:** Remember that subsequences can skip elements. The DP approach correctly handles this by considering all previous positions, not just adjacent ones.

### 2. **Integer Overflow with Large Differences**
When dealing with large numbers in the array, the difference `d = x - y` might exceed integer limits in some languages.

**Problematic Example:**
```python
nums = [2147483647, -2147483648, 0]  # Max and min integers
# The difference could overflow in languages with fixed integer sizes
```

**Solution:** In Python, this isn't an issue due to arbitrary precision integers. In other languages, use appropriate data types (e.g., `long long` in C++) or check for overflow conditions.

### 3. **Forgetting to Count Only Length ≥3 Subsequences**
The DP state `f[i][d]` includes 2-element subsequences, but we only want to count subsequences with at least 3 elements.

**Wrong Implementation:**
```python
def numberOfArithmeticSlices(nums):
    dp = [defaultdict(int) for _ in nums]
    total = 0
  
    for i in range(len(nums)):
        for j in range(i):
            d = nums[i] - nums[j]
            dp[i][d] += dp[j][d] + 1
            total += dp[i][d]  # WRONG: This counts 2-element subsequences too
  
    return total
```

**Solution:** Only add `dp[j][d]` to the answer, not the full `dp[i][d]`, because `dp[j][d]` represents subsequences that already have at least 2 elements and will have at least 3 when extended.

### 4. **Memory Issues with Dense Arrays**
For arrays with many unique differences, the space complexity can become problematic.

**Problematic Case:**
```python
nums = [1, 2, 4, 8, 16, 32, ...]  # Many unique differences
# Each pair creates a unique difference, leading to O(n²) space usage
```

**Solution:** While the algorithm is correct, be aware of memory constraints. For very large inputs with many unique differences, consider:
- Using regular dictionaries instead of defaultdict to save some overhead
- Clearing dictionary entries with value 0 to save space
- Implementing early termination if memory becomes critical

### 5. **Incorrect Loop Boundary**
Using `enumerate(nums[:i])` creates a new list slice each iteration, which is inefficient.

**Better Implementation:**
```python
for i in range(len(nums)):
    for j in range(i):  # More efficient than enumerate(nums[:i])
        d = nums[i] - nums[j]
        total_count += dp[j][d]
        dp[i][d] += dp[j][d] + 1
```

This avoids creating unnecessary list slices and improves performance.

