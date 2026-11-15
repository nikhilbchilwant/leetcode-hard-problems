# 3077. Maximum Strength of K Disjoint Subarrays

> Source: https://algo.monster/liteproblems/3077

## Problem Statement

You have an array of integers `nums` with length `n` and a positive odd integer `k`.

Your task is to select exactly `k` disjoint (non-overlapping) subarrays from `nums`. These subarrays must be ordered such that the last element of subarray `i` appears before the first element of subarray `i+1` in the original array.

The strength of your selection is calculated using an alternating sum formula:

`strength = k * sum(sub₁) - (k-1) * sum(sub₂) + (k-2) * sum(sub₃) - ... - 2 * sum(subₖ₋₁) + sum(subₖ)`

Where:
- `sum(subᵢ)` is the sum of all elements in the i-th subarray
- The coefficients alternate between positive and negative (since `k` is odd, it starts and ends with positive)
- Each subarray's sum is multiplied by a decreasing coefficient starting from `k` down to `1`

The goal is to find the maximum possible strength value by strategically choosing which `k` subarrays to select from the array.

For example, if `k = 3`, the formula would be:
`strength = 3 * sum(sub₁) - 2 * sum(sub₂) + 1 * sum(sub₃)`

Important constraints:
- The subarrays must be disjoint (cannot share elements)
- The subarrays must maintain their relative order in the original array
- You don't need to use all elements from `nums` - some elements can be left unselected

## Intuition

Let's think about what each element contributes to the final strength. If element `nums[i-1]` is selected and belongs to the `j`-th subarray, its contribution to the strength depends on which subarray it's in.

Looking at the strength formula, the `j`-th subarray has a coefficient that follows a pattern:
- 1st subarray: coefficient is `k` (positive)
- 2nd subarray: coefficient is `-(k-1)` (negative)
- 3rd subarray: coefficient is `(k-2)` (positive)
- And so on...

So the coefficient for the `j`-th subarray is `(k - j + 1) * (-1)^(j+1)`. This means each element `nums[i-1]` in the `j`-th subarray contributes `nums[i-1] * (k - j + 1) * (-1)^(j+1)` to the total strength.

Now, how do we decide which elements to include in which subarrays? This is a classic [dynamic programming](/problems/dynamic_programming_intro) problem because:
1. We need to make decisions for each element (include it or not, and if included, which subarray it belongs to)
2. These decisions affect future choices (once we start a new subarray, we can't go back)
3. We want to optimize (maximize) the total strength

The key insight is to track two states for each position:
- Whether the current element is selected or not
- How many complete subarrays we've formed so far

We use `f[i][j][0]` to represent the maximum strength when we've processed the first `i` elements, formed `j` subarrays, and the `i`-th element is NOT selected.

We use `f[i][j][1]` to represent the maximum strength when we've processed the first `i` elements, formed `j` subarrays, and the `i`-th element IS selected.

For each element, we have choices:
- Skip it entirely (contributes nothing)
- Include it as part of the current subarray (if we're building one)
- Start a new subarray with it (if we haven't reached `k` subarrays yet)

The transitions between states naturally follow: if we select element `i`, it either extends the current subarray or starts a new one. If we don't select it, we can potentially end the current subarray and prepare to start a new one later.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

We implement a 3D [dynamic programming](/problems/dynamic_programming_intro) solution where `f[i][j][s]` represents the maximum strength achievable using the first `i` elements, selecting exactly `j` subarrays, where `s` indicates whether the `i`-th element is selected (`s=1`) or not (`s=0`).

**Initialization:**
- Create a 3D DP array with dimensions `(n+1) × (k+1) × 2`, initialized to negative infinity
- Set `f[0][0][0] = 0` as the base case (no elements processed, no subarrays selected, nothing selected)

**State Transitions:**

For each position `i` from 1 to `n` and each number of subarrays `j` from 0 to `k`:

1. **When the `i`-th element is NOT selected** (`f[i][j][0]`):
   - We take the maximum of the previous state where element `i-1` was either selected or not selected
   - `f[i][j][0] = max(f[i-1][j][0], f[i-1][j][1])`

2. **When the `i`-th element IS selected** (`f[i][j][1]`):
   - First, calculate the sign: `sign = 1` if `j` is odd, else `sign = -1`
   - The contribution of this element is `sign × nums[i-1] × (k - j + 1)`
 
   Two cases to consider:
   - **Extending the current subarray**: If element `i-1` was also selected and in the same subarray:
     - `f[i][j][1] = f[i-1][j][1] + sign × nums[i-1] × (k - j + 1)`
 
   - **Starting a new subarray**: If this element starts the `j`-th subarray (requires `j > 0`):
     - `f[i][j][1] = max(f[i-1][j-1][0], f[i-1][j-1][1]) + sign × nums[i-1] × (k - j + 1)`
     - We take the maximum of both cases where the previous element was selected or not

**Final Answer:**
After processing all elements, the maximum strength is `max(f[n][k][0], f[n][k][1])`, which represents the maximum value when we've selected exactly `k` subarrays from all `n` elements.

**Time Complexity:** `O(n × k)` where `n` is the length of the array and `k` is the number of subarrays to select.

**Space Complexity:** `O(n × k)` for the 3D DP array.

## Example Walkthrough

Let's walk through a small example with `nums = [1, 2, -1, 4]` and `k = 3`.

We need to select exactly 3 disjoint subarrays. The strength formula is:
`strength = 3 × sum(sub₁) - 2 × sum(sub₂) + 1 × sum(sub₃)`

**DP Table Setup:**
- We create a 3D array `f[i][j][s]` where `i` ranges from 0 to 4 (length of nums + 1), `j` ranges from 0 to 3 (k), and `s` is 0 or 1.
- Initialize all values to negative infinity except `f[0][0][0] = 0`.

**Key State Transitions:**

Let's trace through some important states:

**i=1 (element 1):**
- `f[1][1][1]`: Start first subarray with element 1
  - Sign for j=1: positive (odd)
  - Contribution: 1 × 3 = 3
  - `f[1][1][1] = 0 + 3 = 3`

**i=2 (element 2):**
- `f[2][1][1]`: Two options for first subarray
  - Extend: `f[1][1][1] + 2 × 3 = 3 + 6 = 9` (subarray [1,2])
  - Start new: `f[1][0][0] + 2 × 3 = 0 + 6 = 6` (subarray [2])
  - Take max: `f[2][1][1] = 9`
- `f[2][2][1]`: Start second subarray with element 2
  - Must come after first subarray ended
  - Sign for j=2: negative (even)
  - `f[2][2][1] = f[1][1][0] + 2 × (-2) = 3 + (-4) = -1`

**i=3 (element -1):**
- `f[3][2][1]`: Options for second subarray
  - Extend from `f[2][2][1]`: `-1 + (-1) × (-2) = -1 + 2 = 1`
  - Start new from `f[2][1][0]`: `9 + (-1) × (-2) = 9 + 2 = 11`
  - Take max: `f[3][2][1] = 11`

**i=4 (element 4):**
- `f[4][3][1]`: Options for third subarray
  - Sign for j=3: positive (odd)
  - Start new from `f[3][2][0]`: `11 + 4 × 1 = 15`
  - This gives us the selection: [1,2], [-1], [4]

**Verification:**
- Subarray 1: [1,2], sum = 3, contribution = 3 × 3 = 9
- Subarray 2: [-1], sum = -1, contribution = -2 × (-1) = 2  
- Subarray 3: [4], sum = 4, contribution = 1 × 4 = 4
- Total strength = 9 + 2 + 4 = 15

The maximum strength is 15, achieved by selecting subarrays [1,2], [-1], and [4].

## Python Solution

```python
class Solution:
    def maximumStrength(self, nums: List[int], k: int) -> int:
        """
        Find the maximum strength by selecting k disjoint subarrays from nums.
        Each subarray contributes to the strength based on its position and alternating sign.
      
        Args:
            nums: Input array of integers
            k: Number of subarrays to select
          
        Returns:
            Maximum possible strength value
        """
        n = len(nums)
      
        # Initialize DP table
        # dp[i][j][state] represents the maximum strength using first i elements,
        # with j subarrays selected, where:
        # state=0: not currently in a subarray
        # state=1: currently in a subarray
        dp = [[[-float('inf'), -float('inf')] for _ in range(k + 1)] for _ in range(n + 1)]
      
        # Base case: no elements used, no subarrays selected, not in a subarray
        dp[0][0][0] = 0
      
        # Process each element
        for i, current_value in enumerate(nums, 1):
            # Try all possible numbers of subarrays
            for subarray_count in range(k + 1):
                # Determine sign based on subarray position (alternating pattern)
                # Even positions (0, 2, 4...) get positive sign
                # Odd positions (1, 3, 5...) get negative sign
                sign = 1 if subarray_count & 1 else -1
              
                # Calculate strength multiplier for current subarray
                strength_multiplier = k - subarray_count + 1
              
                # State 0: Not currently in a subarray
                # Can either skip current element or end previous subarray
                dp[i][subarray_count][0] = max(
                    dp[i - 1][subarray_count][0],  # Was not in subarray, stay out
                    dp[i - 1][subarray_count][1]   # Was in subarray, end it
                )
              
                # State 1: Currently in a subarray
                # Option 1: Continue existing subarray by including current element
                dp[i][subarray_count][1] = max(
                    dp[i][subarray_count][1],
                    dp[i - 1][subarray_count][1] + sign * current_value * strength_multiplier
                )
              
                # Option 2: Start a new subarray (if we haven't used all k subarrays yet)
                if subarray_count > 0:
                    # Take the best state from previous position with one less subarray
                    # and start a new subarray with current element
                    previous_best = max(dp[i - 1][subarray_count - 1])
                    dp[i][subarray_count][1] = max(
                        dp[i][subarray_count][1],
                        previous_best + sign * current_value * strength_multiplier
                    )
      
        # Return the maximum strength after using all n elements and k subarrays
        # Can be in either state (not in subarray or just ended a subarray)
        return max(dp[n][k])
```

## Time and Space Complexity

The time complexity of this code is `O(n × k)`, where `n` is the length of the array `nums` and `k` is the number of subarrays to select.

This is because:
- The outer loop iterates through all elements in `nums`, which takes `O(n)` time
- For each element, we have a nested loop that iterates from 0 to `k`, which takes `O(k)` time
- Inside the nested loops, all operations (comparisons, assignments, arithmetic) are `O(1)`
- Therefore, the total time complexity is `O(n) × O(k) = O(n × k)`

The space complexity is `O(n × k)` as well.

This is because:
- We create a 3D array `f` with dimensions `(n + 1) × (k + 1) × 2`
- The first dimension has size `n + 1`
- The second dimension has size `k + 1` 
- The third dimension has fixed size 2
- Therefore, the total space used is `O((n + 1) × (k + 1) × 2) = O(n × k)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Sign Calculation for Alternating Pattern**

A critical pitfall is miscalculating the sign for the alternating sum formula. The problem states that for odd `k`, the pattern starts and ends with positive coefficients.

**The Pitfall:**
```python
# INCORRECT: Using subarray_count directly for sign
sign = 1 if subarray_count & 1 else -1  # This is wrong!
```

This incorrectly assigns signs based on the current count of subarrays being processed, but the formula requires signs based on the **position** in the final selection.

**The Correct Approach:**
```python
# CORRECT: Sign should alternate starting with positive for first subarray
sign = 1 if (subarray_count % 2) == 1 else -1
```

Since we're counting subarrays from 1 to k, odd positions (1st, 3rd, 5th...) get positive signs, and even positions (2nd, 4th...) get negative signs.

### 2. **Initialization Issues with Negative Infinity**

**The Pitfall:**
Not properly handling the initialization can lead to incorrect propagation of invalid states.

```python
# Potential issue: Some Python versions or environments might not handle -inf properly
dp = [[[-float('inf'), -float('inf')] for _ in range(k + 1)] for _ in range(n + 1)]
```

**Better Solution:**
```python
# Use a very large negative number instead
MIN_VAL = -10**15  # Large enough to act as negative infinity for this problem
dp = [[[MIN_VAL, MIN_VAL] for _ in range(k + 1)] for _ in range(n + 1)]
dp[0][0][0] = 0
```

### 3. **Off-by-One Error in Coefficient Calculation**

**The Pitfall:**
The coefficient for each subarray should decrease from `k` to `1`, but it's easy to miscalculate this.

```python
# INCORRECT: Wrong coefficient calculation
strength_multiplier = subarray_count  # This would give 1, 2, 3... instead of k, k-1, k-2...
```

**The Correct Approach:**
```python
# CORRECT: Coefficient decreases as we select more subarrays
strength_multiplier = k - subarray_count + 1
```

### 4. **Not Handling the Transition for Starting New Subarrays Properly**

**The Pitfall:**
When starting a new subarray, forgetting to check if we have subarrays left to allocate:

```python
# INCORRECT: Not checking if subarray_count > 0
dp[i][subarray_count][1] = max(
    dp[i][subarray_count][1],
    max(dp[i - 1][subarray_count - 1]) + contribution  # Could access negative index!
)
```

**The Correct Approach:**
```python
# CORRECT: Only start new subarray if we haven't started all k subarrays yet
if subarray_count > 0:
    previous_best = max(dp[i - 1][subarray_count - 1])
    dp[i][subarray_count][1] = max(
        dp[i][subarray_count][1],
        previous_best + sign * current_value * strength_multiplier
    )
```

### 5. **Memory Optimization Opportunity Missed**

While not a bug, the 3D DP array uses O(n×k×2) space. Since we only need the previous row, we can optimize:

**Space-Optimized Solution:**
```python
def maximumStrength(self, nums: List[int], k: int) -> int:
    n = len(nums)
    MIN_VAL = -10**15
  
    # Use only two rows instead of n+1 rows
    prev = [[MIN_VAL, MIN_VAL] for _ in range(k + 1)]
    curr = [[MIN_VAL, MIN_VAL] for _ in range(k + 1)]
  
    prev[0][0] = 0
  
    for i, val in enumerate(nums, 1):
        curr[0][0] = 0  # Reset base case
      
        for j in range(k + 1):
            sign = 1 if j & 1 else -1
            coef = k - j + 1
          
            # Not in subarray
            curr[j][0] = max(prev[j][0], prev[j][1])
          
            # In subarray - continue
            curr[j][1] = prev[j][1] + sign * val * coef
          
            # In subarray - start new
            if j > 0:
                curr[j][1] = max(curr[j][1], 
                                max(prev[j-1]) + sign * val * coef)
      
        prev, curr = curr, prev
  
    return max(prev[k])
```

This reduces space complexity from O(n×k) to O(k).

