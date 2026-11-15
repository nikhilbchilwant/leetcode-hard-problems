# 1420. Build Array Where You Can Find The Maximum Exactly K Comparisons

> Source: https://algo.monster/liteproblems/1420

## Problem Statement

You need to build an array of exactly `n` positive integers where each element is between 1 and `m` (inclusive). The goal is to construct arrays that will have a specific "search cost" of exactly `k` when the following algorithm is applied:

The algorithm tracks a maximum value (initially -1) and counts how many times this maximum gets updated as it scans through the array from left to right. Each time an element is encountered that is strictly greater than the current maximum, the maximum is updated and a counter is incremented. The final counter value is the "search cost".

For example, if the array is `[1, 3, 2, 5, 4]`:
- Start with `maximum = -1`, `search_cost = 0`
- Element 1: Since `1 > -1`, update `maximum = 1`, increment `search_cost = 1`
- Element 3: Since `3 > 1`, update `maximum = 3`, increment `search_cost = 2`
- Element 2: Since `2 < 3`, no update
- Element 5: Since `5 > 3`, update `maximum = 5`, increment `search_cost = 3`
- Element 4: Since `4 < 5`, no update
- Final `search_cost = 3`

Your task is to count how many different arrays of length `n` with elements in range `[1, m]` will produce exactly `k` as the search cost. Since the answer can be very large, return it modulo `10^9 + 7`.

The constraints are:
- The array must have exactly `n` integers
- Each element must be between 1 and `m` (inclusive)
- The search cost after applying the algorithm must equal exactly `k`

## Intuition

The key insight is that the search cost represents the number of times we encounter a new maximum while scanning the array. This happens exactly when we see an element that is strictly greater than all previous elements.

Let's think about what information we need to track. At any position in the array, we need to know:
1. How many positions we've filled so far
2. How many times we've updated the maximum (our current search cost)
3. What the current maximum value is

This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach with state `dp[i][c][j]` representing the number of ways to build the first `i` elements of the array such that:
- We've had exactly `c` updates to the maximum
- The current maximum value is `j`

For the base case, when we have just one element (`i = 1`), we must have exactly one update (`c = 1`), and the maximum can be any value from 1 to `m`. So `dp[1][1][j] = 1` for all valid `j`.

Now, when we add a new element at position `i`, we have two choices:

1. **The new element doesn't update the maximum**: If our current maximum is `j`, we can place any value from 1 to `j` at position `i`. This gives us `j` choices, and the search cost remains the same. So we add `dp[i-1][c][j] * j` to `dp[i][c][j]`.

2. **The new element becomes the new maximum**: If we want the new element to be `j` and update the maximum, the previous maximum must have been some value `j0 < j`. This increases the search cost by 1. So we sum up `dp[i-1][c-1][j0]` for all `j0 < j` and add it to `dp[i][c][j]`.

Finally, the answer is the sum of `dp[n][k][j]` for all possible maximum values `j` from 1 to `m`, since we want exactly `n` elements with exactly `k` updates to the maximum, regardless of what the final maximum value is.

The modulo operation is applied throughout to prevent integer overflow since the counts can become very large.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The solution uses a 3D [dynamic programming](/problems/dynamic_programming_intro) approach to count the number of valid arrays.

**State Definition:**
- `dp[i][c][j]` = number of ways to build the first `i` elements where we have exactly `c` updates to the maximum and the current maximum is `j`

**Initialization:**
```python
dp = [[[0] * (m + 1) for _ in range(k + 1)] for _ in range(n + 1)]
```
We create a 3D DP table with dimensions `(n+1) × (k+1) × (m+1)` initialized to 0.

**Base Case:**
```python
for i in range(1, m + 1):
    dp[1][1][i] = 1
```
When we have only one element, it must be the maximum (causing 1 update). Any value from 1 to `m` can be that single element.

**Transition:**
For each position `i` from 2 to `n`, for each possible search cost `c` from 1 to `min(k, i)` (we can't have more updates than elements), and for each possible maximum value `j` from 1 to `m`:

1. **Case 1 - No update to maximum:**
   ```python
   dp[i][c][j] = dp[i - 1][c][j] * j
   ```
   If the maximum remains `j`, we can place any value from 1 to `j` at position `i`. This gives us `j` choices, multiplied by the number of ways to reach state `(i-1, c, j)`.

2. **Case 2 - Update the maximum:**
   ```python
   for j0 in range(1, j):
       dp[i][c][j] += dp[i - 1][c - 1][j0]
   ```
   If position `i` has value `j` and updates the maximum, the previous maximum must have been some `j0 < j`. We sum all possibilities where the previous state was `(i-1, c-1, j0)`.

**Modulo Operation:**
```python
dp[i][c][j] %= mod
```
Applied after each calculation to prevent overflow.

**Final Answer:**
```python
ans = 0
for i in range(1, m + 1):
    ans += dp[n][k][i]
    ans %= mod
```
Sum up all ways to build `n` elements with exactly `k` updates, regardless of the final maximum value.

**Edge Case:**
```python
if k == 0:
    return 0
```
If `k = 0`, no updates to the maximum are allowed, which is impossible since we start with maximum = -1 and must have at least one positive integer.

**Time Complexity:** `O(n × k × m²)` - We iterate through all states and for each state with maximum `j`, we check all previous maximums `j0 < j`.

**Space Complexity:** `O(n × k × m)` for the DP table.

## Example Walkthrough

Let's walk through a small example with `n = 3`, `m = 3`, `k = 2`. We want to count arrays of length 3 with values in [1,3] that have exactly 2 maximum updates.

**Step 1: Initialize DP table**
- Create `dp[4][3][4]` (dimensions are n+1, k+1, m+1)
- Set base case: `dp[1][1][1] = 1`, `dp[1][1][2] = 1`, `dp[1][1][3] = 1`
  - This means: with 1 element, we have 1 update, and that element can be 1, 2, or 3

**Step 2: Fill position i = 2**

For `c = 1` (only 1 update so far):
- `dp[2][1][1]`: max is 1, no update → `dp[1][1][1] * 1 = 1 * 1 = 1`
  - Array pattern: [1, 1]
- `dp[2][1][2]`: max is 2, no update → `dp[1][1][2] * 2 = 1 * 2 = 2`
  - Array patterns: [2, 1], [2, 2]
- `dp[2][1][3]`: max is 3, no update → `dp[1][1][3] * 3 = 1 * 3 = 3`
  - Array patterns: [3, 1], [3, 2], [3, 3]

For `c = 2` (2 updates):
- `dp[2][2][1]`: Can't happen (max 1 can't be an update from previous)
- `dp[2][2][2]`: Update from max=1 → `dp[1][1][1] = 1`
  - Array pattern: [1, 2]
- `dp[2][2][3]`: Update from max=1 or max=2 → `dp[1][1][1] + dp[1][1][2] = 1 + 1 = 2`
  - Array patterns: [1, 3], [2, 3]

**Step 3: Fill position i = 3**

For `c = 2` (exactly 2 updates - what we want):
- `dp[3][2][1]`: max is 1, no update → `dp[2][2][1] * 1 = 0 * 1 = 0`
- `dp[3][2][2]`: 
  - No update: `dp[2][2][2] * 2 = 1 * 2 = 2`
  - Update from max=1: `dp[2][1][1] = 1`
  - Total: `2 + 1 = 3`
  - Array patterns: [1, 2, 1], [1, 2, 2], [1, 1, 2]
- `dp[3][2][3]`:
  - No update: `dp[2][2][3] * 3 = 2 * 3 = 6`
  - Update from max=1: `dp[2][1][1] = 1`
  - Update from max=2: `dp[2][1][2] = 2`
  - Total: `6 + 1 + 2 = 9`
  - Array patterns include: [1, 3, 1], [1, 3, 2], [1, 3, 3], [2, 3, 1], [2, 3, 2], [2, 3, 3], [1, 1, 3], [2, 1, 3], [2, 2, 3]

**Step 4: Calculate final answer**
```
ans = dp[3][2][1] + dp[3][2][2] + dp[3][2][3]
    = 0 + 3 + 9
    = 12
```

Therefore, there are **12 different arrays** of length 3 with values in [1,3] that have exactly 2 maximum updates.

To verify, here are some example arrays with search cost 2:
- [1, 2, 1]: Updates at positions 1 (max=1) and 2 (max=2)
- [1, 3, 2]: Updates at positions 1 (max=1) and 2 (max=3)
- [2, 3, 1]: Updates at positions 1 (max=2) and 2 (max=3)

## Python Solution

```python
class Solution:
    def numOfArrays(self, n: int, m: int, k: int) -> int:
        # Edge case: if search cost is 0, no valid arrays exist
        if k == 0:
            return 0
      
        # dp[length][cost][max_val] = number of arrays of given length 
        # with given search cost and maximum value
        dp = [[[0] * (m + 1) for _ in range(k + 1)] for _ in range(n + 1)]
        MOD = 10**9 + 7
      
        # Base case: arrays of length 1 have search cost 1
        # and their maximum value is the single element itself
        for max_value in range(1, m + 1):
            dp[1][1][max_value] = 1
      
        # Build up the dp table for arrays of increasing length
        for length in range(2, n + 1):
            # Search cost cannot exceed the array length
            for search_cost in range(1, min(k + 1, length + 1)):
                for max_value in range(1, m + 1):
                    # Case 1: append element <= current max (doesn't increase search cost)
                    # We can append any value from 1 to max_value
                    dp[length][search_cost][max_value] = (
                        dp[length - 1][search_cost][max_value] * max_value
                    ) % MOD
                  
                    # Case 2: append element > previous max (increases search cost by 1)
                    # Sum over all possible previous maximum values less than current
                    for prev_max in range(1, max_value):
                        dp[length][search_cost][max_value] = (
                            dp[length][search_cost][max_value] + 
                            dp[length - 1][search_cost - 1][prev_max]
                        ) % MOD
      
        # Sum up all valid arrays of length n with search cost k
        # across all possible maximum values
        result = 0
        for max_value in range(1, m + 1):
            result = (result + dp[n][k][max_value]) % MOD
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n * k * m^2)`

The algorithm uses dynamic programming with three nested loops:
- The outermost loop iterates through array positions from `2` to `n`: `O(n)` iterations
- The second loop iterates through the number of search costs from `1` to `min(k, i)`: `O(k)` iterations in the worst case
- The third loop iterates through maximum values from `1` to `m`: `O(m)` iterations
- Inside the third loop, there's another loop that iterates from `1` to `j-1` to accumulate values from previous states: `O(m)` iterations in the worst case

Therefore, the total time complexity is `O(n * k * m * m) = O(n * k * m^2)`.

**Space Complexity:** `O(n * k * m)`

The algorithm uses a 3D DP array `dp[n+1][k+1][m+1]` where:
- The first dimension represents the array length: `n+1` elements
- The second dimension represents the search cost: `k+1` elements  
- The third dimension represents the maximum value: `m+1` elements

The total space required is `O((n+1) * (k+1) * (m+1)) = O(n * k * m)`.

Note that the space complexity could be optimized to `O(k * m)` since each state `dp[i][c][j]` only depends on `dp[i-1][*][*]`, allowing us to use only two 2D arrays instead of a full 3D array.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Base Case Initialization**
A common mistake is initializing the base case incorrectly or incompletely. Some developers might write:
```python
# Wrong: Only initializing dp[1][1][1] = 1
dp[1][1][1] = 1
```
This assumes only value 1 can be the first element, but actually any value from 1 to m can be the first element.

**Solution:** Initialize all possible maximum values for length 1:
```python
for max_value in range(1, m + 1):
    dp[1][1][max_value] = 1
```

### 2. **Forgetting Modulo Operations**
Since the result can be very large, forgetting to apply modulo at intermediate steps can cause integer overflow:
```python
# Wrong: Only applying modulo at the end
dp[length][search_cost][max_value] = (
    dp[length - 1][search_cost][max_value] * max_value + 
    sum_of_previous_states
)
# Missing modulo here!
```

**Solution:** Apply modulo after every arithmetic operation:
```python
dp[length][search_cost][max_value] = (
    dp[length - 1][search_cost][max_value] * max_value
) % MOD
# ... later additions also need modulo
dp[length][search_cost][max_value] = (
    dp[length][search_cost][max_value] + 
    dp[length - 1][search_cost - 1][prev_max]
) % MOD
```

### 3. **Off-by-One Errors in Loop Bounds**
Getting the loop bounds wrong, especially for the search cost:
```python
# Wrong: search_cost can go up to k+1
for search_cost in range(1, k + 2):  # This will cause index out of bounds
```
Also, search cost cannot exceed the array length:
```python
# Wrong: not considering that search_cost <= length
for search_cost in range(1, k + 1):  # Should be min(k + 1, length + 1)
```

**Solution:** Use correct bounds that respect both the maximum search cost k and the array length:
```python
for search_cost in range(1, min(k + 1, length + 1)):
```

### 4. **Misunderstanding the State Transition Logic**
A subtle error is miscounting when adding an element that doesn't update the maximum:
```python
# Wrong: Adding 1 for each way instead of multiplying by max_value
dp[length][search_cost][max_value] = dp[length - 1][search_cost][max_value] + max_value
```
When the maximum stays the same, we can choose any value from 1 to max_value for the new position, so we multiply, not add.

**Solution:** Multiply by the number of valid choices:
```python
dp[length][search_cost][max_value] = (
    dp[length - 1][search_cost][max_value] * max_value
) % MOD
```

### 5. **Not Handling Edge Cases**
Failing to check if k = 0 or if k > n:
```python
# Missing edge case handling
def numOfArrays(self, n: int, m: int, k: int) -> int:
    # Directly starting DP without checking invalid cases
    dp = [[[0] * (m + 1) for _ in range(k + 1)] for _ in range(n + 1)]
```

**Solution:** Add edge case checks at the beginning:
```python
if k == 0:
    return 0  # No valid arrays with 0 search cost
if k > n:
    return 0  # Can't have more updates than elements
```

