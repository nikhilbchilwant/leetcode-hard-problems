# 2281. Sum of Total Strength of Wizards

> Source: https://algo.monster/liteproblems/2281

## Problem Statement

You are given an array `strength` where `strength[i]` represents the strength of the i-th wizard in your kingdom.

For any contiguous subarray of wizards, the **total strength** is calculated as:
- The **minimum strength** among all wizards in the subarray
- **Multiplied by** the **sum of all strengths** in the subarray

For example, if you have a subarray `[3, 1, 5]`:
- The weakest wizard has strength `1`
- The sum of all strengths is `3 + 1 + 5 = 9`
- The total strength is `1 × 9 = 9`

Your task is to find the **sum of total strengths** for **all possible contiguous subarrays** in the given array.

Since the result can be very large, return the answer modulo `10^9 + 7`.

**Example walkthrough:**

If `strength = [1, 3, 2]`, the contiguous subarrays and their total strengths are:
- `[1]`: min = 1, sum = 1, total strength = 1 × 1 = 1
- `[3]`: min = 3, sum = 3, total strength = 3 × 3 = 9  
- `[2]`: min = 2, sum = 2, total strength = 2 × 2 = 4
- `[1, 3]`: min = 1, sum = 4, total strength = 1 × 4 = 4
- `[3, 2]`: min = 2, sum = 5, total strength = 2 × 5 = 10
- `[1, 3, 2]`: min = 1, sum = 6, total strength = 1 × 6 = 6

The sum of all total strengths = 1 + 9 + 4 + 4 + 10 + 6 = 34

## Intuition

The brute force approach would be to check every possible subarray, find its minimum element and sum, then calculate the total strength. This would take `O(n^3)` time, which is too slow for large arrays.

The key insight is to think about the problem from a different angle: instead of iterating through all subarrays, we can focus on each element and ask - "In how many subarrays is this element the minimum?"

For each element `strength[i]`, if we know all the subarrays where it acts as the minimum element, we can calculate its contribution to the final answer. The contribution would be:
`strength[i] × (sum of all subarray sums where strength[i] is the minimum)`

To find which subarrays have `strength[i]` as their minimum, we need to find:
- The nearest smaller element to the left (let's call its index `left[i]`)
- The nearest smaller or equal element to the right (let's call its index `right[i]`)

Any subarray that includes `strength[i]` and spans from index `l` (where `left[i] < l <= i`) to index `r` (where `i <= r < right[i]`) will have `strength[i]` as its minimum element.

The next challenge is efficiently calculating the sum of all subarray sums that include `strength[i]` as the minimum. This is where prefix sums come in handy. By using a double [prefix sum](/problems/subarray_sum) (prefix sum of prefix sums), we can calculate in `O(1)` time:
- The sum of all subarray sums that start from any position between `left[i]+1` to `i` and end at any position between `i` to `right[i]-1`

The monotonic [stack](/problems/stack_intro) technique helps us efficiently find the nearest smaller elements on both sides in `O(n)` time. We use the stack to maintain elements in increasing order, popping elements when we find a smaller one.

This approach reduces the overall time complexity to `O(n)`, making it efficient even for large arrays.

**Learn more about [Stack](/problems/stack_intro), [Prefix Sum](/problems/subarray_sum) and [Monotonic Stack](/problems/mono_stack_intro) patterns.**

## Solution Approach

The solution implements the intuition using monotonic stacks and prefix sums. Let's walk through each component:

**Step 1: Find boundaries using monotonic stacks**

First, we find for each element the range where it acts as the minimum:

```python
left = [-1] * n  # nearest smaller element to the left
right = [n] * n   # nearest smaller or equal element to the right
```

For the left boundaries:
- Iterate through the array from left to right
- Maintain a [stack](/problems/stack_intro) of indices in increasing order of their values
- When we encounter `strength[i]`, pop all elements from the stack that are `>= strength[i]`
- The top of the stack (if exists) gives us `left[i]`

For the right boundaries:
- Iterate through the array from right to left  
- Similar process, but we pop elements that are `> strength[i]` (strictly greater)
- This ensures we handle equal elements correctly (leftmost one is chosen as minimum)

**Step 2: Calculate prefix sums of prefix sums**

```python
ss = list(accumulate(list(accumulate(strength, initial=0)), initial=0))
```

This creates a double [prefix sum](/problems/subarray_sum) array where:
- First `accumulate` gives us regular prefix sums: `ps[i] = strength[0] + ... + strength[i-1]`
- Second `accumulate` gives us prefix sums of prefix sums: `ss[i] = ps[0] + ps[1] + ... + ps[i-1]`

**Step 3: Calculate contribution of each element**

For each element at index `i` with value `v`:
- It's the minimum in all subarrays from `[l, r]` where `l ∈ [left[i]+1, i]` and `r ∈ [i, right[i]-1]`

The sum of all subarray sums where `strength[i]` is minimum can be calculated as:
```python
a = (ss[r + 2] - ss[i + 1]) * (i - l + 1)  # sum of subarrays ending after i
b = (ss[i + 1] - ss[l]) * (r - i + 1)      # sum of subarrays starting before i
total_sum = a - b
```

This formula uses the double [prefix sum](/problems/subarray_sum) to efficiently compute:
- `a`: Sum of all subarrays that end at positions from `i` to `r`, multiplied by the number of possible starting positions
- `b`: Sum of all subarrays that start at positions from `l` to `i`, multiplied by the number of possible ending positions

The contribution of element `i` is then: `(a - b) * v`

**Step 4: Sum all contributions**

```python
ans = (ans + (a - b) * v) % mod
```

We add each element's contribution to the final answer, taking modulo at each step to prevent overflow.

The time complexity is `O(n)` as we make a constant number of passes through the array, and space complexity is `O(n)` for the auxiliary arrays.

## Example Walkthrough

Let's walk through the solution approach with `strength = [2, 1, 3]`.

**Step 1: Find boundaries using monotonic stacks**

Finding `left[]` (nearest smaller element to the left):
- i=0, strength[0]=2: stack is empty, so left[0]=-1. Push 0 to stack. Stack: [0]
- i=1, strength[1]=1: Pop 0 (since 2≥1). Stack empty, so left[1]=-1. Push 1. Stack: [1]  
- i=2, strength[2]=3: 1<3, so left[2]=1. Push 2. Stack: [1,2]
- Result: `left = [-1, -1, 1]`

Finding `right[]` (nearest smaller or equal to the right):
- i=2, strength[2]=3: stack empty, so right[2]=3. Push 2. Stack: [2]
- i=1, strength[1]=1: Pop 2 (since 3>1). Stack empty, so right[1]=3. Push 1. Stack: [1]
- i=0, strength[0]=2: 1<2, so right[0]=1. Push 0. Stack: [1,0]
- Result: `right = [1, 3, 3]`

**Step 2: Calculate prefix sums of prefix sums**
- Original: `[2, 1, 3]`
- First prefix sum: `[0, 2, 3, 6]` (ps[i] = sum of elements before index i)
- Second prefix sum: `[0, 0, 2, 5, 11]` (ss[i] = sum of ps[0] to ps[i-1])

**Step 3: Calculate contribution of each element**

For element at index 0 (value=2):
- It's minimum in subarrays from [l=0, r=0] (since left[0]=-1, right[0]=1)
- l = left[0]+1 = 0, r = right[0]-1 = 0
- a = (ss[0+2] - ss[0+1]) × (0-0+1) = (2-0) × 1 = 2
- b = (ss[0+1] - ss[0]) × (0-0+1) = (0-0) × 1 = 0
- Contribution = (2-0) × 2 = 4

For element at index 1 (value=1):
- It's minimum in subarrays from [l=0, r=2] (since left[1]=-1, right[1]=3)
- l = left[1]+1 = 0, r = right[1]-1 = 2
- a = (ss[2+2] - ss[1+1]) × (1-0+1) = (11-2) × 2 = 18
- b = (ss[1+1] - ss[0]) × (2-1+1) = (2-0) × 2 = 4
- Contribution = (18-4) × 1 = 14

For element at index 2 (value=3):
- It's minimum in subarrays from [l=2, r=2] (since left[2]=1, right[2]=3)
- l = left[2]+1 = 2, r = right[2]-1 = 2
- a = (ss[2+2] - ss[2+1]) × (2-2+1) = (11-5) × 1 = 6
- b = (ss[2+1] - ss[2]) × (2-2+1) = (5-2) × 1 = 3
- Contribution = (6-3) × 3 = 9

**Step 4: Sum all contributions**
Total = 4 + 14 + 9 = 27

Let's verify:
- [2]: min=2, sum=2, strength=2×2=4 ✓
- [1]: min=1, sum=1, strength=1×1=1
- [3]: min=3, sum=3, strength=3×3=9 ✓
- [2,1]: min=1, sum=3, strength=1×3=3
- [1,3]: min=1, sum=4, strength=1×4=4
- [2,1,3]: min=1, sum=6, strength=1×6=6

Total = 4+1+9+3+4+6 = 27 ✓

## Python Solution

```python
class Solution:
    def totalStrength(self, strength: List[int]) -> int:
        """
        Calculate the sum of total strength across all subarrays.
        For each subarray, total strength = min(subarray) * sum(subarray)
        """
        n = len(strength)
      
        # Find the previous smaller element for each position
        # left[i] = index of the nearest element to the left that is strictly smaller than strength[i]
        # -1 if no such element exists
        left_smaller = [-1] * n
        stack = []
      
        for i, current_strength in enumerate(strength):
            # Pop elements that are greater than or equal to current
            while stack and strength[stack[-1]] >= current_strength:
                stack.pop()
          
            if stack:
                left_smaller[i] = stack[-1]
            stack.append(i)
      
        # Find the next smaller or equal element for each position
        # right[i] = index of the nearest element to the right that is smaller than or equal to strength[i]
        # n if no such element exists
        right_smaller = [n] * n
        stack = []
      
        for i in range(n - 1, -1, -1):
            # Pop elements that are strictly greater than current
            while stack and strength[stack[-1]] > strength[i]:
                stack.pop()
          
            if stack:
                right_smaller[i] = stack[-1]
            stack.append(i)
      
        # Build prefix sum array and prefix sum of prefix sums
        # This helps calculate range sums efficiently
        # prefix_sum_of_prefix_sums[i] = sum of all prefix sums up to index i-1
        prefix_sum_of_prefix_sums = list(accumulate(list(accumulate(strength, initial=0)), initial=0))
      
        MOD = 10**9 + 7
        total_strength_sum = 0
      
        # For each element as the minimum of subarrays
        for i, min_strength in enumerate(strength):
            # Range where strength[i] is the minimum
            # All subarrays from index left_bound to right_bound that include i
            left_bound = left_smaller[i] + 1
            right_bound = right_smaller[i] - 1
          
            # Calculate contribution of all subarrays where strength[i] is minimum
            # Using the formula for sum of subarray sums
          
            # Sum of all subarrays ending after i and starting at or before i
            positive_contribution = (prefix_sum_of_prefix_sums[right_bound + 2] - prefix_sum_of_prefix_sums[i + 1]) * (i - left_bound + 1)
          
            # Sum of all subarrays starting before i and ending at or before i
            negative_contribution = (prefix_sum_of_prefix_sums[i + 1] - prefix_sum_of_prefix_sums[left_bound]) * (right_bound - i + 1)
          
            # Add the contribution of this element as minimum
            contribution = (positive_contribution - negative_contribution) * min_strength
            total_strength_sum = (total_strength_sum + contribution) % MOD
      
        return total_strength_sum
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm consists of several linear passes through the array:
- First monotonic stack traversal to compute `left` array: `O(n)` - each element is pushed and popped at most once
- Second monotonic stack traversal to compute `right` array: `O(n)` - each element is pushed and popped at most once  
- Computing prefix sum array using `accumulate` twice: `O(n)` for each accumulate operation
- Final loop to calculate the answer: `O(n)` - iterates through each element once with `O(1)` operations per iteration

Total time complexity: `O(n) + O(n) + O(n) + O(n) + O(n) = O(n)`

**Space Complexity:** `O(n)`

The algorithm uses the following additional space:
- `left` array: `O(n)`
- `right` array: `O(n)`
- `stk` (reused twice): `O(n)` in worst case when all elements are in increasing/decreasing order
- `ss` array (double prefix sum): `O(n+1)` = `O(n)`

Total space complexity: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Equal Elements in Monotonic Stack**

One of the most critical pitfalls in this problem is incorrectly handling equal elements when finding the boundaries where each element serves as the minimum.

**The Pitfall:**
If you use the same comparison operator (either `>=` or `>`) for both left and right boundary calculations, you'll count some subarrays multiple times when there are duplicate values.

For example, with `strength = [2, 2, 2]`:
- If you use `>=` for both directions, all three elements would claim to be the minimum for the entire array
- The subarray `[2, 2, 2]` would be counted three times instead of once

**The Solution:**
Use asymmetric comparisons:
- For left boundaries: Pop elements that are `>= current` (greater than or equal)
- For right boundaries: Pop elements that are `> current` (strictly greater)

This ensures that when there are equal elements, only the leftmost one is chosen as the minimum for subarrays containing multiple equal values.

```python
# Correct approach:
# Left boundary - use >=
while stack and strength[stack[-1]] >= current_strength:
    stack.pop()

# Right boundary - use > (strictly greater)
while stack and strength[stack[-1]] > strength[i]:
    stack.pop()
```

### 2. **Integer Overflow with Large Numbers**

**The Pitfall:**
The multiplication and addition operations can produce very large numbers before taking the modulo, potentially causing integer overflow in languages with fixed integer sizes.

**The Solution:**
In Python, this isn't an issue due to arbitrary precision integers, but it's good practice to apply modulo operations frequently:

```python
# Instead of calculating everything then taking modulo once:
# total = (a - b) * v
# ans = (ans + total) % MOD

# Apply modulo at intermediate steps:
contribution = ((positive_contribution - negative_contribution) % MOD * min_strength) % MOD
total_strength_sum = (total_strength_sum + contribution) % MOD
```

### 3. **Off-by-One Errors in Prefix Sum Calculations**

**The Pitfall:**
The double prefix sum array indexing can be confusing. It's easy to use wrong indices when calculating the sum contributions, especially with the `+1` and `+2` offsets.

**The Solution:**
Remember that `prefix_sum_of_prefix_sums` has two extra levels of padding:
- First `accumulate` adds one element at the beginning (initial=0)
- Second `accumulate` adds another element at the beginning

So `prefix_sum_of_prefix_sums[i+2]` corresponds to the sum of prefix sums up to and including index `i` in the original array.

Always verify your indexing with small examples:
```python
# For strength = [1, 2, 3]
# After first accumulate: [0, 1, 3, 6]
# After second accumulate: [0, 0, 1, 4, 10]
# prefix_sum_of_prefix_sums[i+2] gives sum of prefix sums up to index i
```

### 4. **Misunderstanding the Contribution Formula**

**The Pitfall:**
The formula `(a - b) * v` for calculating each element's contribution is not intuitive and can be easily misimplemented.

**The Solution:**
Break down what each part represents:
- `a = (ss[r + 2] - ss[i + 1]) * (i - l + 1)`: Sum of all subarrays that END at positions from `i` to `r`, multiplied by the number of possible starting positions
- `b = (ss[i + 1] - ss[l]) * (r - i + 1)`: Sum of all subarrays that START at positions from `l` to `i`, multiplied by the number of possible ending positions
- The difference `(a - b)` gives the total sum of all subarrays where element `i` is the minimum

Test your understanding with a simple example before implementing.

