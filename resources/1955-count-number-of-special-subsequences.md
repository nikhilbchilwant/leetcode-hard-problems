# 1955. Count Number of Special Subsequences

> Source: https://algo.monster/liteproblems/1955

## Problem Statement

You need to count special subsequences in an array that contains only the integers 0, 1, and 2.

A sequence is **special** if it follows this exact pattern:
- First, one or more `0`s
- Then, one or more `1`s  
- Finally, one or more `2`s

All three parts must be present and appear in this order.

Examples of special sequences:
- `[0,1,2]` - has one 0, one 1, one 2
- `[0,0,1,1,1,2]` - has two 0s, three 1s, one 2

Examples of non-special sequences:
- `[2,1,0]` - wrong order
- `[1]` - missing 0s and 2s
- `[0,1,2,0]` - has a 0 after the 2s

Given an array `nums` containing only 0s, 1s, and 2s, you need to find how many different **subsequences** are special.

A subsequence is formed by deleting some (or no) elements from the original array while keeping the relative order of remaining elements. Two subsequences are considered different if they use different sets of indices from the original array.

Since the answer can be very large, return the result modulo `10^9 + 7`.

For example, if `nums = [0,1,2]`, there is exactly 1 special subsequence: `[0,1,2]` itself.

## Intuition

When we see a problem about counting subsequences, [dynamic programming](/problems/dynamic_programming_intro) often comes to mind. The key insight here is that a special subsequence must have exactly three phases: 0s, then 1s, then 2s.

As we scan through the array, at each position we need to track how many valid subsequences we can form that are currently in each phase:
- Phase 0: subsequences that only contain 0s (waiting for 1s)
- Phase 1: subsequences that have 0s followed by 1s (waiting for 2s)  
- Phase 2: complete special subsequences with 0s, 1s, and 2s

When we encounter a number in the array, we have two choices: include it in our subsequences or skip it.

If we see a `0`:
- We can add it to any existing subsequence ending with 0s (doubling our count)
- We can also start a new subsequence with just this 0 (adding 1)
- This gives us `2 * previous_count_of_0s + 1`

If we see a `1`:
- We can add it to any subsequence ending with 0s (transitioning to phase 1)
- We can add it to any subsequence already in phase 1 (extending the 1s)
- This gives us `previous_count_of_0s + 2 * previous_count_of_1s`

If we see a `2`:
- We can add it to any subsequence ending with 1s (transitioning to phase 2)
- We can add it to any subsequence already in phase 2 (extending the 2s)
- This gives us `previous_count_of_1s + 2 * previous_count_of_2s`

The doubling effect (multiplying by 2) comes from the fact that for each existing subsequence, we can either include or exclude the current element, creating two possibilities.

By tracking these three counts as we iterate through the array, we can efficiently compute the total number of special subsequences. The answer is the count of complete subsequences (phase 2) after processing all elements.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement the [dynamic programming](/problems/dynamic_programming_intro) approach using a 2D array `f[i][j]` where:
- `i` represents the index in the input array (0 to n-1)
- `j` represents the phase/ending digit (0, 1, or 2)
- `f[i][j]` stores the count of special subsequences ending with digit `j` using the first `i+1` elements

**Initialization:**
- Create a 2D array `f` of size `n × 3` initialized with zeros
- If `nums[0] == 0`, set `f[0][0] = 1` (we can start a subsequence with this 0)

**State Transitions:**

For each position `i` from 1 to n-1, we update based on `nums[i]`:

**Case 1: When `nums[i] == 0`**
- `f[i][0] = (2 * f[i-1][0] + 1) % mod`
  - We can extend any subsequence ending with 0 (include or exclude current 0)
  - Plus 1 for starting a new subsequence with just this 0
- `f[i][1] = f[i-1][1]` (no change for subsequences ending with 1)
- `f[i][2] = f[i-1][2]` (no change for subsequences ending with 2)

**Case 2: When `nums[i] == 1`**
- `f[i][0] = f[i-1][0]` (no change for subsequences ending with 0)
- `f[i][1] = (f[i-1][0] + 2 * f[i-1][1]) % mod`
  - Add current 1 to any subsequence ending with 0 (transition from phase 0 to 1)
  - Extend any subsequence ending with 1 (include or exclude current 1)
- `f[i][2] = f[i-1][2]` (no change for subsequences ending with 2)

**Case 3: When `nums[i] == 2`**
- `f[i][0] = f[i-1][0]` (no change for subsequences ending with 0)
- `f[i][1] = f[i-1][1]` (no change for subsequences ending with 1)
- `f[i][2] = (f[i-1][1] + 2 * f[i-1][2]) % mod`
  - Add current 2 to any subsequence ending with 1 (transition from phase 1 to 2)
  - Extend any subsequence ending with 2 (include or exclude current 2)

**Final Answer:**
Return `f[n-1][2]`, which represents the count of complete special subsequences (those that have gone through all three phases: 0→1→2).

**Modulo Operation:**
Since the counts can grow very large, we apply modulo `10^9 + 7` at each step to prevent overflow.

**Time Complexity:** O(n) - single pass through the array
**Space Complexity:** O(n) - for the 2D DP array (can be optimized to O(1) using rolling array technique)

## Example Walkthrough

Let's walk through the solution with `nums = [0, 1, 2, 0, 1, 2]`.

We'll track our DP table `f[i][j]` where `f[i][j]` represents the count of subsequences ending with digit `j` after processing the first `i+1` elements.

**Initial Setup:**
- Array: `[0, 1, 2, 0, 1, 2]`
- Initialize `f` as a 6×3 array with zeros
- Since `nums[0] = 0`, set `f[0][0] = 1`

**Step-by-step processing:**

**i = 0:** `nums[0] = 0`
- `f[0][0] = 1` (one subsequence: `[0]`)
- `f[0][1] = 0` (no subsequences ending with 1)
- `f[0][2] = 0` (no subsequences ending with 2)

**i = 1:** `nums[1] = 1`
- `f[1][0] = f[0][0] = 1` (still just `[0]`)
- `f[1][1] = f[0][0] + 2 * f[0][1] = 1 + 0 = 1` (new subsequence: `[0,1]`)
- `f[1][2] = f[0][2] = 0`

**i = 2:** `nums[2] = 2`
- `f[2][0] = f[1][0] = 1` (still just `[0]`)
- `f[2][1] = f[1][1] = 1` (still just `[0,1]`)
- `f[2][2] = f[1][1] + 2 * f[1][2] = 1 + 0 = 1` (complete subsequence: `[0,1,2]`)

**i = 3:** `nums[3] = 0`
- `f[3][0] = 2 * f[2][0] + 1 = 2 * 1 + 1 = 3`
  - Previous `[0]` can include/exclude this 0: gives us `[0]` and `[0,0]`
  - Plus starting fresh with just this 0 at index 3
  - Total: 3 subsequences ending with 0
- `f[3][1] = f[2][1] = 1`
- `f[3][2] = f[2][2] = 1`

**i = 4:** `nums[4] = 1`
- `f[4][0] = f[3][0] = 3`
- `f[4][1] = f[3][0] + 2 * f[3][1] = 3 + 2 * 1 = 5`
  - The 3 subsequences ending with 0 can now add this 1
  - The 1 subsequence ending with 1 can include/exclude this 1
  - Total: 5 subsequences ending with 1
- `f[4][2] = f[3][2] = 1`

**i = 5:** `nums[5] = 2`
- `f[5][0] = f[4][0] = 3`
- `f[5][1] = f[4][1] = 5`
- `f[5][2] = f[4][1] + 2 * f[4][2] = 5 + 2 * 1 = 7`
  - The 5 subsequences ending with 1 can now add this 2
  - The 1 subsequence ending with 2 can include/exclude this 2
  - Total: 7 complete special subsequences

**Final Answer:** `f[5][2] = 7`

The 7 special subsequences are:
1. `[0,1,2]` using indices (0,1,2)
2. `[0,1,2]` using indices (0,1,5)
3. `[0,1,2]` using indices (0,4,5)
4. `[0,1,2]` using indices (3,4,5)
5. `[0,0,1,2]` using indices (0,3,4,5)
6. `[0,1,1,2]` using indices (0,1,4,5)
7. `[0,1,2,2]` using indices (0,1,2,5)

## Python Solution

```python
class Solution:
    def countSpecialSubsequences(self, nums: List[int]) -> int:
        """
        Count the number of subsequences of the form [0...0, 1...1, 2...2]
        where each part contains at least one element.
      
        Args:
            nums: List of integers containing only 0, 1, or 2
          
        Returns:
            Number of valid subsequences modulo 10^9 + 7
        """
        MOD = 10**9 + 7
        n = len(nums)
      
        # dp[i][j] represents the count of valid subsequences ending at index i
        # j=0: subsequences containing only 0s
        # j=1: subsequences containing 0s followed by 1s
        # j=2: subsequences containing 0s, 1s, then 2s (complete subsequences)
        dp = [[0] * 3 for _ in range(n)]
      
        # Base case: first element
        if nums[0] == 0:
            dp[0][0] = 1
      
        # Fill the DP table
        for i in range(1, n):
            current_num = nums[i]
          
            if current_num == 0:
                # Can either start new subsequence with this 0 or extend existing 0-only subsequences
                # Formula: previous 0-only subsequences * 2 (include or exclude) + 1 (new subsequence)
                dp[i][0] = (2 * dp[i - 1][0] + 1) % MOD
                dp[i][1] = dp[i - 1][1]
                dp[i][2] = dp[i - 1][2]
              
            elif current_num == 1:
                # Can either start the 1s part after 0s or extend existing 0-1 subsequences
                # Formula: all 0-only subsequences can transition + existing 0-1 subsequences * 2
                dp[i][0] = dp[i - 1][0]
                dp[i][1] = (dp[i - 1][0] + 2 * dp[i - 1][1]) % MOD
                dp[i][2] = dp[i - 1][2]
              
            else:  # current_num == 2
                # Can either start the 2s part after 0-1s or extend existing complete subsequences
                # Formula: all 0-1 subsequences can transition + existing complete subsequences * 2
                dp[i][0] = dp[i - 1][0]
                dp[i][1] = dp[i - 1][1]
                dp[i][2] = (dp[i - 1][1] + 2 * dp[i - 1][2]) % MOD
      
        # Return count of complete subsequences (containing 0s, 1s, and 2s)
        return dp[n - 1][2]
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm iterates through the array once from index 1 to n-1, where n is the length of the input array. In each iteration, it performs a constant amount of work (checking the value of `nums[i]` and updating at most 3 values in the dp array with simple arithmetic operations). Therefore, the overall time complexity is linear with respect to the input size.

**Space Complexity:** `O(n)`

The algorithm uses a 2D array `f` of size `n × 3`, where n is the length of the input array. This results in `O(3n) = O(n)` space usage. Additionally, only a constant amount of extra variables are used (`mod`, `n`, and loop variable `i`), which doesn't affect the overall space complexity.

Note: The space complexity could be optimized to `O(1)` since each row of the dp array only depends on the previous row. We could use just two 1D arrays of size 3 or even just 3 variables to track the previous state, reducing space usage to constant.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Understanding of Subsequence Multiplication Factor**

**Pitfall:** Many developers mistakenly think that when encountering a digit that matches the current phase, they should simply add 1 or use `dp[i-1][j] + 1` instead of `2 * dp[i-1][j]`.

**Why it's wrong:** When we encounter a digit that can extend existing subsequences of the same type, we have two choices for EACH existing subsequence:
- Include the current element in that subsequence
- Exclude the current element from that subsequence

This creates `2 * count` possibilities, not `count + 1`.

**Example:** If we have 2 subsequences ending with 0: `[0]` and `[0,0]`, and we encounter another 0:
- From `[0]`: we can form `[0]` (exclude) or `[0,0]` (include)
- From `[0,0]`: we can form `[0,0]` (exclude) or `[0,0,0]` (include)
- Plus we can start a new subsequence: `[0]`
- Total: 2 * 2 + 1 = 5 subsequences

**Solution:**
```python
# Correct approach
if current_num == 0:
    dp[i][0] = (2 * dp[i-1][0] + 1) % MOD  # Multiply by 2, not add 1

# Incorrect approach (common mistake)
# dp[i][0] = (dp[i-1][0] + 1) % MOD  # This would undercount
```

### 2. **Forgetting the Modulo Operation During Intermediate Calculations**

**Pitfall:** Applying modulo only at the end instead of during each calculation step, which can cause integer overflow in languages with fixed integer sizes.

**Why it's wrong:** The intermediate values can grow exponentially and exceed the maximum integer value before the final modulo operation.

**Solution:**
```python
# Correct: Apply modulo at each step
dp[i][1] = (dp[i-1][0] + 2 * dp[i-1][1]) % MOD

# Incorrect: Might overflow before modulo
# dp[i][1] = dp[i-1][0] + 2 * dp[i-1][1]
# ... later ...
# return dp[n-1][2] % MOD
```

### 3. **Space Optimization Without Proper Variable Management**

**Pitfall:** When optimizing space from O(n) to O(1) by using three variables instead of a 2D array, developers often update variables in the wrong order, causing them to use already-updated values.

**Why it's wrong:** The transitions depend on the previous state values. If we update a variable too early, subsequent calculations will use the new value instead of the old one.

**Solution:**
```python
# Space-optimized version with correct update order
def countSpecialSubsequences(self, nums: List[int]) -> int:
    MOD = 10**9 + 7
    zeros = ones = twos = 0
  
    for num in nums:
        if num == 0:
            zeros = (2 * zeros + 1) % MOD
        elif num == 1:
            ones = (zeros + 2 * ones) % MOD
        else:  # num == 2
            twos = (ones + 2 * twos) % MOD
  
    return twos
```

Note that this works because each update only modifies one variable at a time, and we don't need the old values after updating.

### 4. **Incorrect Base Case Initialization**

**Pitfall:** Initializing `dp[0][0] = 1` regardless of what `nums[0]` is, or forgetting to handle the base case entirely.

**Why it's wrong:** If `nums[0] = 1` or `nums[0] = 2`, we cannot start a valid subsequence with it (since valid subsequences must start with 0).

**Solution:**
```python
# Correct initialization
if nums[0] == 0:
    dp[0][0] = 1
# dp[0][1] and dp[0][2] remain 0

# Incorrect initialization
# dp[0][0] = 1  # Wrong if nums[0] != 0
```

