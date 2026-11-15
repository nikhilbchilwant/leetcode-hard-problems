# 2263. Make Array Non-decreasing or Non-increasing

> Source: https://algo.monster/liteproblems/2263

## Problem Statement

You have an array of integers `nums` indexed from 0. You can perform operations on this array where each operation allows you to:
- Select any index `i` (where `0 <= i < nums.length`)
- Either increase `nums[i]` by 1 or decrease `nums[i]` by 1

Your goal is to transform the array so that it becomes either:
- **Non-decreasing**: Each element is less than or equal to the next element (`nums[i] <= nums[i+1]` for all valid i)
- **Non-increasing**: Each element is greater than or equal to the next element (`nums[i] >= nums[i+1]` for all valid i)

You need to find the minimum total number of operations required to achieve either of these two conditions.

For example:
- If `nums = [3, 2, 4, 1]`, you could make it non-decreasing by changing it to something like `[2, 2, 2, 2]` or `[1, 1, 4, 4]`
- Or you could make it non-increasing by changing it to something like `[4, 4, 1, 1]` or `[3, 2, 1, 1]`
- The answer would be the minimum operations needed among all possible transformations

## Intuition

The key insight is recognizing that we need to find the optimal target values for each position in the array. When making an array non-decreasing, each element can be adjusted to any value, but there's a constraint: each element must be less than or equal to the next one.

First, let's think about the range of possible values. Since we can only increment or decrement by 1 in each operation, and we want to minimize operations, the optimal target values should be close to the original values. In fact, the optimal solution will always have each final value be one of the original values in the array (or within the range of original values). This is because moving to values outside this range would just add unnecessary operations.

Since the problem constraints limit values to a reasonable range (0-1000 based on the code), we can use [dynamic programming](/problems/dynamic_programming_intro) where we consider all possible values for each position.

For a non-decreasing array, we define `f[i][j]` as the minimum operations needed to make the first `i` elements non-decreasing, where the `i-th` element becomes value `j`. The transition is: for position `i`, if we want it to have value `j`, we need to consider all values `k <= j` for position `i-1`. The cost is the minimum cost to reach value `k` at position `i-1`, plus the cost to change element `i` from its original value to `j` (which is `abs(nums[i] - j)`).

The optimization in the code uses the fact that for non-decreasing arrays, if we fix the value at position `i` to be `j`, we want the minimum cost among all valid previous states where the value was `<= j`. We can maintain this minimum as we iterate through possible values.

For non-increasing arrays, we can reuse the same logic by simply reversing the array. A non-increasing array reversed becomes a non-decreasing array, so we solve the same problem on `nums[::-1]`.

The final answer is the minimum between making the array non-decreasing and making it non-increasing.

**Learn more about [Greedy](/problems/greedy_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a [dynamic programming](/problems/dynamic_programming_intro) approach with space optimization. Let's walk through the implementation step by step:

**1. Main Function Structure:**
The solution defines a helper function `solve(nums)` that finds the minimum operations to make the array non-decreasing. It then calls this function twice - once with the original array and once with the reversed array - and returns the minimum of the two results.

**2. [Dynamic Programming](/problems/dynamic_programming_intro) Setup:**
```python
n = len(nums)
f = [[0] * 1001 for _ in range(n + 1)]
```
We create a 2D DP table `f` where `f[i][j]` represents the minimum operations needed to make the first `i` elements non-decreasing, with the `i-th` element having value `j`. The table has dimensions `(n+1) × 1001` to handle all possible values from 0 to 1000.

**3. DP Transition:**
```python
for i, x in enumerate(nums, 1):
    mi = inf
    for j in range(1001):
        if mi > f[i - 1][j]:
            mi = f[i - 1][j]
        f[i][j] = mi + abs(x - j)
```

For each element at position `i` (1-indexed) with original value `x`:
- We iterate through all possible target values `j` from 0 to 1000
- We maintain `mi` as the minimum cost to reach position `i-1` with any value `<= j`
- As we iterate through `j`, we update `mi` whenever we find a smaller cost at `f[i-1][j]`
- The cost for position `i` to have value `j` is: `mi + abs(x - j)`
  - `mi`: minimum cost to make the first `i-1` elements non-decreasing with the last element `<= j`
  - `abs(x - j)`: cost to change the current element from `x` to `j`

**4. Key Optimization:**
The clever part is maintaining the running minimum `mi` as we iterate through values. Since we want a non-decreasing array, if element `i` has value `j`, then element `i-1` can have any value from 0 to `j`. Instead of checking all these values for each `j`, we maintain the minimum incrementally.

**5. Final Result:**
```python
return min(f[n])
```
After processing all elements, `f[n][j]` contains the minimum operations to make the entire array non-decreasing with the last element being `j`. We return the minimum among all possible final values.

**6. Handling Both Cases:**
```python
return min(solve(nums), solve(nums[::-1]))
```
To handle both non-decreasing and non-increasing cases:
- `solve(nums)` finds the cost for non-decreasing
- `solve(nums[::-1])` finds the cost for non-increasing (by reversing the array)
- We return the minimum of the two

The time complexity is `O(n × m)` where `n` is the array length and `m` is the range of values (1001), and the space complexity is also `O(n × m)`.

## Example Walkthrough

Let's walk through the solution with `nums = [3, 2, 4, 1]`.

**Finding minimum operations for non-decreasing:**

We'll build a DP table where `f[i][j]` = minimum operations to make first `i` elements non-decreasing with element `i` having value `j`.

For element 1 (value=3):
- `f[1][0] = |3-0| = 3` (change 3 to 0)
- `f[1][1] = |3-1| = 2` (change 3 to 1)
- `f[1][2] = |3-2| = 1` (change 3 to 2)
- `f[1][3] = |3-3| = 0` (keep 3 as 3)
- `f[1][4] = |3-4| = 1` (change 3 to 4)

For element 2 (value=2):
- To make element 2 have value `j`, element 1 must be ≤ `j`
- For `j=2`: minimum of `f[1][0]`, `f[1][1]`, `f[1][2]` = min(3,2,1) = 1
  - `f[2][2] = 1 + |2-2| = 1`
- For `j=3`: minimum of `f[1][0]`, `f[1][1]`, `f[1][2]`, `f[1][3]` = min(3,2,1,0) = 0
  - `f[2][3] = 0 + |2-3| = 1`
- For `j=4`: minimum includes all previous = 0
  - `f[2][4] = 0 + |2-4| = 2`

For element 3 (value=4):
- For `j=4`: minimum of all `f[2][k]` where `k≤4`
  - Best previous state is `f[2][2]=1` or `f[2][3]=1`
  - `f[3][4] = 1 + |4-4| = 1`

For element 4 (value=1):
- For `j=4`: minimum of all `f[3][k]` where `k≤4` = 1
  - `f[4][4] = 1 + |1-4| = 4`
- For `j=1`: minimum of `f[3][0]`, `f[3][1]` (higher costs)
- For `j=2`: minimum includes `f[3][2]` = 3
  - `f[4][2] = 3 + |1-2| = 4`

Minimum for non-decreasing: min(f[4]) = 4 (achieved when final array is [2,2,2,2] or similar)

**Finding minimum operations for non-increasing:**

Reverse the array: `[1, 4, 2, 3]` and apply the same algorithm.

After similar calculations, we'd find that making [1,4,2,3] non-decreasing (which corresponds to making [3,2,4,1] non-increasing) requires 3 operations.

One optimal non-increasing result would be [3,2,1,1] with operations:
- Keep 3 as 3 (0 ops)
- Keep 2 as 2 (0 ops)
- Change 4 to 1 (3 ops)
- Keep 1 as 1 (0 ops)
- Total: 3 operations

**Final answer:** min(4, 3) = 3

## Python Solution

```python
class Solution:
    def convertArray(self, nums: List[int]) -> int:
        def solve(nums):
            """
            Calculate minimum operations to make array non-decreasing.
            Uses dynamic programming where dp[i][j] represents the minimum cost
            to make the first i elements non-decreasing with the i-th element being j.
            """
            n = len(nums)
            # dp[i][j] = minimum cost to make first i elements non-decreasing
            # with the i-th element having value j
            dp = [[0] * 1001 for _ in range(n + 1)]

            # Process each element in the array
            for i, current_value in enumerate(nums, 1):
                min_cost = float('inf')

                # Try all possible values for the current position (0 to 1000)
                for target_value in range(1001):
                    # Update minimum cost from previous row
                    # This ensures non-decreasing property
                    if min_cost > dp[i - 1][target_value]:
                        min_cost = dp[i - 1][target_value]

                    # Cost to change current element to target_value
                    # plus the minimum cost to reach this state
                    dp[i][target_value] = min_cost + abs(current_value - target_value)

            # Return the minimum cost among all possible final values
            return min(dp[n])

        # Try both non-decreasing and non-increasing transformations
        # Non-increasing is achieved by reversing the array and finding non-decreasing
        return min(solve(nums), solve(nums[::-1]))
```

## Time and Space Complexity

**Time Complexity:** `O(n * m)` where `n` is the length of the input array and `m = 1001` (the range of possible values).

The algorithm calls the `solve` function twice - once with the original array and once with the reversed array. Inside `solve`:
- There are two nested loops: the outer loop iterates through `n` elements of the array
- The inner loop iterates through 1001 possible values (0 to 1000)
- For each combination of `i` and `j`, we perform constant time operations (comparison, assignment, and absolute value calculation)
- Finding the minimum in `f[i-1][j]` is done incrementally with the `mi` variable, so it doesn't add extra complexity
- The final `min(f[n])` operation takes `O(m)` time

Total time: `2 * O(n * m) = O(n * m) = O(1001n) = O(n)`

**Space Complexity:** `O(n * m)` where `n` is the length of the input array and `m = 1001`.

The space is dominated by:
- The 2D DP array `f` with dimensions `(n+1) × 1001`, requiring `O(n * m)` space
- The reversed array `nums[::-1]` creates a copy of the input array, requiring `O(n)` space
- Other variables like `mi`, `i`, `j`, `x` use constant space

Total space: `O(n * m) + O(n) = O(n * m) = O(1001n) = O(n)`

Note: Since `m = 1001` is a constant, both complexities can be simplified to `O(n)` in terms of the input size.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Range Assumption**
One of the most common pitfalls is assuming the wrong range for array values. The solution uses `[0, 1000]` as the range, but if the input contains negative numbers or values greater than 1000, the solution will fail.

**Example of the problem:**
```python
nums = [-5, 10, 2000]  # Contains negative and values > 1000
# The current solution will crash or give incorrect results
```

**Solution:**
Dynamically determine the range from the input array:
```python
def solve(nums):
    n = len(nums)
    min_val = min(nums)
    max_val = max(nums)

    # Create DP table with actual range
    dp = [[0] * (max_val - min_val + 1) for _ in range(n + 1)]

    for i, current_value in enumerate(nums, 1):
        min_cost = float('inf')

        # Iterate through the actual range
        for target_value in range(min_val, max_val + 1):
            idx = target_value - min_val  # Adjust index for DP table
            if min_cost > dp[i - 1][idx]:
                min_cost = dp[i - 1][idx]
            dp[i][idx] = min_cost + abs(current_value - target_value)

    return min(dp[n])
```

### 2. **Memory Overflow with Large Ranges**
If the array contains values with a very large range (e.g., `[-10^9, 10^9]`), creating a DP table for all possible values will cause memory issues.

**Solution:**
Use coordinate compression - only consider values that actually appear in the array:
```python
def solve(nums):
    n = len(nums)
    # Only consider values that appear in the array
    unique_values = sorted(set(nums))
    m = len(unique_values)

    # Map values to indices
    value_to_idx = {v: i for i, v in enumerate(unique_values)}

    dp = [[0] * m for _ in range(n + 1)]

    for i, current_value in enumerate(nums, 1):
        min_cost = float('inf')

        for j, target_value in enumerate(unique_values):
            if min_cost > dp[i - 1][j]:
                min_cost = dp[i - 1][j]
            dp[i][j] = min_cost + abs(current_value - target_value)

    return min(dp[n])
```

### 3. **Space Optimization Opportunity Missed**
The current solution uses O(n × m) space, but since we only need the previous row to compute the current row, we can optimize to O(m) space.

**Solution:**
Use rolling arrays:
```python
def solve(nums):
    n = len(nums)
    min_val, max_val = min(nums), max(nums)
    range_size = max_val - min_val + 1

    # Use only two rows instead of n+1 rows
    prev = [0] * range_size
    curr = [0] * range_size

    for current_value in nums:
        min_cost = float('inf')

        for target_value in range(min_val, max_val + 1):
            idx = target_value - min_val
            if min_cost > prev[idx]:
                min_cost = prev[idx]
            curr[idx] = min_cost + abs(current_value - target_value)

        prev, curr = curr, prev

    return min(prev)
```

### 4. **Not Handling Edge Cases**
The solution might not properly handle edge cases like empty arrays or single-element arrays.

**Solution:**
Add edge case checks:
```python
def convertArray(self, nums: List[int]) -> int:
    # Handle edge cases
    if not nums or len(nums) <= 1:
        return 0

    def solve(nums):
        # ... rest of the implementation

    return min(solve(nums), solve(nums[::-1]))
```

