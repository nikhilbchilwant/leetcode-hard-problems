# 1330. Reverse Subarray To Maximize Array Value

> Source: https://algo.monster/liteproblems/1330

## Problem Statement

You have an integer array `nums`. The **value** of this array is calculated as the sum of `|nums[i] - nums[i + 1]|` for all indices where `0 <= i < nums.length - 1`. In other words, the value is the sum of absolute differences between all consecutive pairs of elements.

You can perform **exactly one operation**: select any contiguous subarray of `nums` and reverse it. Your goal is to find the maximum possible value of the array after performing this operation (or choosing not to reverse any subarray).

For example, if `nums = [2, 3, 1, 5, 4]`:
- The initial value is `|2-3| + |3-1| + |1-5| + |5-4| = 1 + 2 + 4 + 1 = 8`
- If you reverse the subarray from index 1 to 3 (elements `[3, 1, 5]`), the array becomes `[2, 5, 1, 3, 4]`
- The new value would be `|2-5| + |5-1| + |1-3| + |3-4| = 3 + 4 + 2 + 1 = 10`

The problem asks you to determine the maximum value achievable by strategically choosing which subarray to reverse (if any).

## Intuition

When we reverse a subarray, we're essentially changing which elements become neighbors. Let's think about what happens to the array value when we reverse a subarray from index `i` to index `j`.

The key insight is that reversing a subarray only affects the "edges" of that subarray - specifically, the connections at positions `i-1` to `i` and `j` to `j+1`. Inside the reversed subarray, the absolute differences remain the same (just in reverse order), so they don't contribute to any change in the total value.

What changes are:
- The connection between `nums[i-1]` and `nums[i]` becomes a connection between `nums[i-1]` and `nums[j]`
- The connection between `nums[j]` and `nums[j+1]` becomes a connection between `nums[i]` and `nums[j+1]`

This means we're replacing `|nums[i-1] - nums[i]| + |nums[j] - nums[j+1]|` with `|nums[i-1] - nums[j]| + |nums[i] - nums[j+1]|`.

To maximize the array value, we want to maximize the gain from this replacement, which is:
`|nums[i-1] - nums[j]| + |nums[i] - nums[j+1]| - |nums[i-1] - nums[i]| - |nums[j] - nums[j+1]|`

Now we need to consider different cases:

1. **No reversal**: Keep the original array value as a baseline.

2. **Reversal including the first element**: If the reversed subarray starts at index 0, we only have one "edge" change at the right boundary.

3. **Reversal including the last element**: If the reversed subarray ends at the last index, we only have one "edge" change at the left boundary.

4. **Reversal in the middle**: Both edges change, and we need to find the optimal pair of edges to swap.

For the fourth case, the challenge is efficiently finding the maximum gain across all possible pairs of edges. The mathematical trick here is to reformulate the absolute value expressions. For any two pairs `(x₁, y₁)` and `(x₂, y₂)`, the gain can be expressed using the fact that:

`|a - b| + |c - d| = max{(a+c)-(b+d), (a-c)-(b-d), (-a+c)-(-b+d), (-a-c)-(-b-d)}`

By iterating through all adjacent pairs and tracking the maximum and minimum values for different sign combinations (`k₁ * x + k₂ * y` where `k₁, k₂ ∈ {-1, 1}`), we can find the optimal reversal in linear time rather than checking all O(n²) possible subarrays.

**Learn more about [Greedy](/problems/greedy_intro) and [Math](/problems/math-basics) patterns.**

## Solution Approach

Let's implement the solution by handling each case systematically:

**Step 1: Calculate the baseline value**

First, we calculate the sum of absolute differences for the original array:
```python
s = sum(abs(x - y) for x, y in pairwise(nums))
ans = s  # Initialize answer with the original value
```

**Step 2: Handle reversals including the first element**

When the reversed subarray starts at index 0 and ends at index `i`, we're replacing the edge `|nums[i] - nums[i+1]|` with `|nums[0] - nums[i+1]|`:
```python
for x, y in pairwise(nums):  # x = nums[i], y = nums[i+1]
    ans = max(ans, s + abs(nums[0] - y) - abs(x - y))
```

**Step 3: Handle reversals including the last element**

When the reversed subarray starts at index `i+1` and ends at the last index, we're replacing the edge `|nums[i] - nums[i+1]|` with `|nums[i] - nums[-1]|`:
```python
for x, y in pairwise(nums):  # x = nums[i], y = nums[i+1]
    ans = max(ans, s + abs(nums[-1] - x) - abs(x - y))
```

**Step 4: Handle middle reversals (the complex case)**

For reversals that don't include the first or last element, we need to find the optimal pair of edges to swap. The gain from swapping edges `(x₁, y₁)` and `(x₂, y₂)` is:

`|x₁ - x₂| + |y₁ - y₂| - |x₁ - y₁| - |x₂ - y₂|`

We can rewrite this using four different sign combinations. For each combination `(k₁, k₂)` where `k₁, k₂ ∈ {-1, 1}`, we track:
- `mx`: maximum value of `k₁ * x + k₂ * y - |x - y|` 
- `mi`: minimum value of `k₁ * x + k₂ * y + |x - y|`

The maximum gain for this sign combination is `mx - mi`.

```python
for k1, k2 in pairwise((1, -1, -1, 1, 1)):  # Covers all four sign combinations
    mx, mi = -inf, inf
    for x, y in pairwise(nums):
        a = k1 * x + k2 * y
        b = abs(x - y)
        mx = max(mx, a - b)  # Track maximum of (k1*x + k2*y - |x-y|)
        mi = min(mi, a + b)  # Track minimum of (k1*x + k2*y + |x-y|)
    ans = max(ans, s + max(mx - mi, 0))  # Add the gain if positive
```

The `pairwise((1, -1, -1, 1, 1))` generates the pairs: `(1, -1)`, `(-1, -1)`, `(-1, 1)`, `(1, 1)`, which covers all four sign combinations needed.

**Time and Space Complexity:**
- **Time Complexity**: `O(n)` - We make a constant number of passes through the array
- **Space Complexity**: `O(1)` - We only use a few variables to track maximum and minimum values

The elegance of this solution lies in transforming what seems like an `O(n²)` problem (checking all possible subarrays) into an `O(n)` problem by recognizing that we only need to consider the edges of the reversed subarray and using mathematical reformulation to efficiently find the optimal edges.

## Example Walkthrough

Let's walk through the solution with `nums = [2, 3, 1, 5, 4]`.

**Step 1: Calculate baseline value**
- Original array: `[2, 3, 1, 5, 4]`
- Consecutive differences: `|2-3| + |3-1| + |1-5| + |5-4| = 1 + 2 + 4 + 1 = 8`
- Initialize `ans = 8`

**Step 2: Check reversals including first element**
We check what happens if we reverse from index 0 to each position:
- Reverse `[0,1]`: Changes edge at position 1. Replace `|3-1|=2` with `|2-1|=1`. Gain = -1. Value = 7.
- Reverse `[0,2]`: Changes edge at position 2. Replace `|1-5|=4` with `|2-5|=3`. Gain = -1. Value = 7.
- Reverse `[0,3]`: Changes edge at position 3. Replace `|5-4|=1` with `|2-4|=2`. Gain = +1. Value = 9.

Current `ans = max(8, 9) = 9`

**Step 3: Check reversals including last element**
We check what happens if we reverse from each position to the end:
- Reverse `[1,4]`: Changes edge at position 0. Replace `|2-3|=1` with `|2-4|=2`. Gain = +1. Value = 9.
- Reverse `[2,4]`: Changes edge at position 1. Replace `|3-1|=2` with `|3-4|=1`. Gain = -1. Value = 7.
- Reverse `[3,4]`: Changes edge at position 2. Replace `|1-5|=4` with `|1-4|=3`. Gain = -1. Value = 7.

Current `ans = max(9, 9) = 9`

**Step 4: Check middle reversals**
For reversals not touching the boundaries, we need to swap two edges. Let's check reversing `[1,3]`:
- Original edges: `|2-3|=1` at position 0 and `|5-4|=1` at position 3
- New edges: `|2-5|=3` at position 0 and `|3-4|=1` at position 3
- Gain = `(3 + 1) - (1 + 1) = 2`
- Value = 8 + 2 = 10

To find this efficiently, the algorithm tracks maximum gains across all edge pairs using the sign combination technique:

For sign combination `(k1=1, k2=-1)`:
- Edge at position 0: `(x=2, y=3)`: `1*2 + (-1)*3 - |2-3| = -1 - 1 = -2`
- Edge at position 1: `(x=3, y=1)`: `1*3 + (-1)*1 - |3-1| = 2 - 2 = 0`
- Edge at position 2: `(x=1, y=5)`: `1*1 + (-1)*5 - |1-5| = -4 - 4 = -8`
- Edge at position 3: `(x=5, y=4)`: `1*5 + (-1)*4 - |5-4| = 1 - 1 = 0`

Maximum `mx = 0`, and tracking corresponding minimums, we find the best gain.

After checking all sign combinations, we find that reversing `[1,3]` gives the maximum gain of 2.

**Final Answer: 10**

The optimal strategy is to reverse the subarray `[3, 1, 5]` to get `[2, 5, 1, 3, 4]`, achieving a maximum value of 10.

## Python Solution

```python
class Solution:
    def maxValueAfterReverse(self, nums: List[int]) -> int:
        # Calculate the initial sum of absolute differences between adjacent elements
        initial_sum = sum(abs(nums[i] - nums[i + 1]) for i in range(len(nums) - 1))
        max_sum = initial_sum
      
        # Strategy 1: Try reversing subarrays that start at index 0
        # When we reverse [0, j], the edge at position j changes
        for i in range(len(nums) - 1):
            current_diff = abs(nums[i] - nums[i + 1])
            new_diff = abs(nums[0] - nums[i + 1])
            max_sum = max(max_sum, initial_sum + new_diff - current_diff)
      
        # Strategy 2: Try reversing subarrays that end at the last index
        # When we reverse [i, n-1], the edge at position i-1 changes
        for i in range(len(nums) - 1):
            current_diff = abs(nums[i] - nums[i + 1])
            new_diff = abs(nums[-1] - nums[i])
            max_sum = max(max_sum, initial_sum + new_diff - current_diff)
      
        # Strategy 3: Try reversing middle subarrays [i+1, j]
        # This changes edges at positions i and j
        # We use a mathematical optimization to find the best gain
        sign_patterns = [(1, -1), (-1, 1), (1, 1)]
      
        for sign1, sign2 in sign_patterns:
            max_value = float('-inf')
            min_value = float('inf')
          
            # For each adjacent pair, calculate potential gains
            for i in range(len(nums) - 1):
                x, y = nums[i], nums[i + 1]
              
                # Calculate the linear combination based on sign pattern
                linear_combination = sign1 * x + sign2 * y
                current_diff = abs(x - y)
              
                # Track maximum and minimum values for optimization
                max_value = max(max_value, linear_combination - current_diff)
                min_value = min(min_value, linear_combination + current_diff)
          
            # Calculate the maximum possible gain from this pattern
            gain = max(0, max_value - min_value)
            max_sum = max(max_sum, initial_sum + gain)
      
        return max_sum
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm consists of several parts:
- Initial sum calculation using `pairwise(nums)`: iterates through `n-1` pairs, taking `O(n)` time
- First loop through `pairwise(nums)`: iterates through `n-1` pairs with constant operations per iteration, taking `O(n)` time
- Outer loop through `pairwise((1, -1, -1, 1, 1))`: runs exactly 4 iterations (constant)
  - Inner loop through `pairwise(nums)`: iterates through `n-1` pairs with constant operations per iteration, taking `O(n)` time
  - Since the outer loop runs 4 times, this section takes `4 * O(n) = O(n)` time

Overall time complexity: `O(n) + O(n) + O(n) = O(n)`

**Space Complexity:** `O(1)`

The algorithm uses only a constant amount of extra space:
- Variables `ans`, `s`, `mx`, `mi`, `a`, `b`, `k1`, `k2`, `x`, `y` all store single values
- The `pairwise` iterator generates pairs on-the-fly without creating additional data structures
- No recursive calls or additional data structures proportional to input size are created

Therefore, the space complexity is `O(1)` (constant space).

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Missing the (-1, -1) Sign Pattern**

One of the most common mistakes is forgetting to include all four sign combinations when handling middle reversals. The code shown only includes three patterns: `[(1, -1), (-1, 1), (1, 1)]`, but misses `(-1, -1)`.

**Why this matters:** Each sign pattern captures different scenarios where reversing a subarray could increase the value. Missing one pattern means potentially missing the optimal solution.

**Fix:**
```python
# Incorrect - only 3 patterns
sign_patterns = [(1, -1), (-1, 1), (1, 1)]

# Correct - all 4 patterns
sign_patterns = [(1, -1), (-1, 1), (1, 1), (-1, -1)]
```

### 2. **Off-by-One Errors in Index Handling**

When reversing a subarray from index `i` to `j`, it's easy to confuse which edges are affected:
- Reversing `[i, j]` affects the edges at positions `i-1` (if `i > 0`) and `j` (if `j < n-1`)
- The code needs to handle the relationship between array indices and edge positions carefully

**Common mistake:**
```python
# Wrong - confusing array index with edge index
new_diff = abs(nums[-1] - nums[i+1])  # Should be nums[i]
```

### 3. **Not Considering the "Do Nothing" Option**

The problem allows choosing not to reverse any subarray. Forgetting to initialize the answer with the original sum means the solution might return a value less than the original when no reversal is beneficial.

**Fix:**
```python
# Always start with the original sum as a baseline
max_sum = initial_sum  # Don't forget this initialization!
```

### 4. **Integer Overflow with Float Infinity**

Using `float('-inf')` and `float('inf')` can cause issues in some environments or when dealing with very large integers. Python handles this well, but it's cleaner to use bounded values.

**Better approach:**
```python
# Instead of float('-inf') and float('inf')
max_value = -10**9  # Use a reasonable bound based on constraints
min_value = 10**9
```

### 5. **Inefficient Recalculation of Initial Sum**

While not incorrect, using a generator expression in `sum()` creates unnecessary overhead. For better performance with large arrays:

**More efficient:**
```python
# Instead of generator expression
initial_sum = 0
for i in range(len(nums) - 1):
    initial_sum += abs(nums[i] - nums[i + 1])
```

### Complete Corrected Solution:

```python
class Solution:
    def maxValueAfterReverse(self, nums: List[int]) -> int:
        n = len(nums)
        if n <= 1:
            return 0
          
        # Calculate initial sum efficiently
        initial_sum = 0
        for i in range(n - 1):
            initial_sum += abs(nums[i] - nums[i + 1])
      
        max_sum = initial_sum
      
        # Strategy 1: Reversals starting at index 0
        for i in range(n - 1):
            current_diff = abs(nums[i] - nums[i + 1])
            new_diff = abs(nums[0] - nums[i + 1])
            max_sum = max(max_sum, initial_sum + new_diff - current_diff)
      
        # Strategy 2: Reversals ending at last index
        for i in range(n - 1):
            current_diff = abs(nums[i] - nums[i + 1])
            new_diff = abs(nums[-1] - nums[i])
            max_sum = max(max_sum, initial_sum + new_diff - current_diff)
      
        # Strategy 3: Middle reversals - ALL FOUR sign patterns
        sign_patterns = [(1, -1), (-1, 1), (1, 1), (-1, -1)]
      
        for sign1, sign2 in sign_patterns:
            max_value = -10**9
            min_value = 10**9
          
            for i in range(n - 1):
                x, y = nums[i], nums[i + 1]
                linear_combination = sign1 * x + sign2 * y
                current_diff = abs(x - y)
              
                max_value = max(max_value, linear_combination - current_diff)
                min_value = min(min_value, linear_combination + current_diff)
          
            gain = max(0, max_value - min_value)
            max_sum = max(max_sum, initial_sum + gain)
      
        return max_sum
```

