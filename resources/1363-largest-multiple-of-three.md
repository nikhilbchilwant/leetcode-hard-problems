# 1363. Largest Multiple of Three

> Source: https://algo.monster/liteproblems/1363

## Problem Statement

You are given an array of digits `digits` where each element is a single digit from 0 to 9. Your task is to form the largest possible number that is a multiple of three by concatenating some (or all) of the given digits in any order you choose.

The key requirements are:
- The resulting number must be divisible by 3
- You want to make this number as large as possible
- You can use any subset of the given digits
- You can arrange the selected digits in any order
- The answer should be returned as a string (since it might be too large for integer data types)
- Leading zeros should be removed from the result (except when the answer is "0")
- If it's impossible to form a multiple of 3, return an empty string

For example, if you're given `digits = [8, 1, 9]`, you could form "981", "918", "891", etc. Since all these numbers are divisible by 3 (because 8+1+9=18, which is divisible by 3), you would return "981" as it's the largest possible arrangement.

The challenge involves two main considerations:
1. **Divisibility by 3**: A number is divisible by 3 if and only if the sum of its digits is divisible by 3
2. **Maximizing the value**: Once you know which digits to use, arrange them in descending order to get the largest number

## Intuition

To build the largest number that's divisible by 3, we need to understand a fundamental property: a number is divisible by 3 if and only if the sum of its digits is divisible by 3.

This means we need to select digits whose sum `≡ 0 (mod 3)`. But we also want the largest possible number, which requires:
1. Using as many digits as possible (more digits = larger number)
2. Arranging selected digits in descending order

The challenge is finding the optimal subset of digits. We can't just greedily take all digits, as their sum might not be divisible by 3. We also can't just take the largest digits, as we might miss a valid combination.

This leads us to think about [dynamic programming](/problems/dynamic_programming_intro). We can track the maximum number of digits we can select for each possible remainder when divided by 3. Specifically, we define `f[i][j]` as the maximum count of digits we can select from the first `i` digits such that their sum modulo 3 equals `j`.

Why track all three remainders (0, 1, 2)? Because as we add more digits, we might transition from one remainder to another. For instance, if we currently have digits summing to remainder 1, and we add a digit with remainder 2, we get remainder 0 (since `(1 + 2) % 3 = 0`).

The state transition follows naturally: for each digit, we either skip it or include it. If we include digit `x`, we transition from remainder `(j - x % 3 + 3) % 3` to remainder `j`, gaining one more digit in our selection.

After computing the DP table, `f[n][0]` tells us the maximum number of digits we can select to form a multiple of 3. To reconstruct which digits to use, we backtrack through our decisions. Starting from `f[n][0]`, we check whether each digit was included by seeing if including it would have led to our current state.

Finally, to maximize the numeric value, we sort the digits initially and select from largest to smallest during backtracking, then handle the edge case of leading zeros.

**Learn more about [Greedy](/problems/greedy_intro), [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The solution implements [dynamic programming](/problems/dynamic_programming_intro) with backtracking to find the largest multiple of 3. Here's the step-by-step implementation:

**1. [Sorting](/problems/sorting_summary) the Input**
```python
digits.sort()
```
We sort the digits in ascending order first. This helps during backtracking - we'll process larger digits first when reconstructing the answer.

**2. [Dynamic Programming](/problems/dynamic_programming_intro) Table Setup**
```python
f = [[-inf] * 3 for _ in range(n + 1)]
f[0][0] = 0
```
We create a 2D DP table `f[i][j]` where:
- `i` ranges from 0 to `n` (number of digits considered)
- `j` ranges from 0 to 2 (possible remainders when divided by 3)
- `f[i][j]` stores the maximum count of digits we can select from the first `i` digits such that their sum `mod 3 = j`

Initialize `f[0][0] = 0` (selecting 0 digits gives sum 0, which has remainder 0), and all other states to `-inf` (impossible states).

**3. Filling the DP Table**
```python
for i, x in enumerate(digits, 1):
    for j in range(3):
        f[i][j] = max(f[i - 1][j], f[i - 1][(j - x % 3 + 3) % 3] + 1)
```
For each digit `x` at position `i`, and each remainder `j`:
- Option 1: Don't select digit `x`, so `f[i][j] = f[i-1][j]`
- Option 2: Select digit `x`, transitioning from state where previous remainder was `(j - x % 3 + 3) % 3`

The formula `(j - x % 3 + 3) % 3` calculates the previous remainder needed. If we want remainder `j` after adding `x`, we need the previous sum to have remainder `(j - x % 3)`. The `+3` and `% 3` handle negative values correctly.

**4. Check for Valid Solution**
```python
if f[n][0] <= 0:
    return ""
```
If `f[n][0] ≤ 0`, no valid selection exists that forms a multiple of 3.

**5. Backtracking to Find Selected Digits**
```python
arr = []
j = 0
for i in range(n, 0, -1):
    k = (j - digits[i - 1] % 3 + 3) % 3
    if f[i - 1][k] + 1 == f[i][j]:
        arr.append(digits[i - 1])
        j = k
```
Starting from `f[n][0]`, we trace back our decisions:
- For each position `i`, calculate what the previous remainder `k` would have been if we selected `digits[i-1]`
- If `f[i-1][k] + 1 == f[i][j]`, then we selected this digit
- Add selected digits to `arr` and update `j = k` for the next iteration
- Since we process from largest to smallest indices and digits were sorted, `arr` will contain digits in descending order

**6. Handle Leading Zeros**
```python
i = 0
while i < len(arr) - 1 and arr[i] == 0:
    i += 1
return "".join(map(str, arr[i:]))
```
Remove unnecessary leading zeros, keeping at least one digit if all selected digits are zeros. Convert the result to a string and return.

The time complexity is `O(n log n + 3n) = O(n log n)` for [sorting](/problems/sorting_summary) and DP computation. The space complexity is `O(n)` for the DP table.

## Example Walkthrough

Let's walk through the solution with `digits = [8, 6, 7, 1, 0]`.

**Step 1: Sort the digits**
After sorting: `[0, 1, 6, 7, 8]`

**Step 2: Initialize DP table**
We create `f[i][j]` where `i` is from 0 to 5 (number of digits) and `j` is from 0 to 2 (remainders).
Initial state: `f[0][0] = 0`, all others = `-inf`

**Step 3: Fill DP table**
Process each digit and update the table:

For digit `0` (i=1):
- `f[1][0] = max(f[0][0], f[0][0] + 1) = max(0, 1) = 1`
- `f[1][1] = max(f[0][1], f[0][1] + 1) = max(-inf, -inf) = -inf`
- `f[1][2] = max(f[0][2], f[0][2] + 1) = max(-inf, -inf) = -inf`

For digit `1` (i=2):
- `f[2][0] = max(f[1][0], f[1][2] + 1) = max(1, -inf) = 1`
- `f[2][1] = max(f[1][1], f[1][0] + 1) = max(-inf, 2) = 2`
- `f[2][2] = max(f[1][2], f[1][1] + 1) = max(-inf, -inf) = -inf`

For digit `6` (i=3):
- `f[3][0] = max(f[2][0], f[2][0] + 1) = max(1, 2) = 2`
- `f[3][1] = max(f[2][1], f[2][1] + 1) = max(2, 3) = 3`
- `f[3][2] = max(f[2][2], f[2][2] + 1) = max(-inf, -inf) = -inf`

For digit `7` (i=4):
- `f[4][0] = max(f[3][0], f[3][2] + 1) = max(2, -inf) = 2`
- `f[4][1] = max(f[3][1], f[3][0] + 1) = max(3, 3) = 3`
- `f[4][2] = max(f[3][2], f[3][1] + 1) = max(-inf, 4) = 4`

For digit `8` (i=5):
- `f[5][0] = max(f[4][0], f[4][1] + 1) = max(2, 4) = 4`
- `f[5][1] = max(f[4][1], f[4][2] + 1) = max(3, 5) = 5`
- `f[5][2] = max(f[4][2], f[4][0] + 1) = max(4, 3) = 4`

**Step 4: Check validity**
`f[5][0] = 4 > 0`, so a valid solution exists with 4 digits.

**Step 5: Backtrack to find selected digits**
Starting from `f[5][0] = 4`, trace back:

- At i=5 (digit 8): Check if we used it
  - Previous remainder needed: `(0 - 8%3 + 3) % 3 = (0 - 2 + 3) % 3 = 1`
  - Check: `f[4][1] + 1 = 3 + 1 = 4 = f[5][0]` ✓
  - Select digit 8, move to state `f[4][1]`

- At i=4 (digit 7): Check if we used it
  - Previous remainder needed: `(1 - 7%3 + 3) % 3 = (1 - 1 + 3) % 3 = 0`
  - Check: `f[3][0] + 1 = 2 + 1 = 3 = f[4][1]` ✓
  - Select digit 7, move to state `f[3][0]`

- At i=3 (digit 6): Check if we used it
  - Previous remainder needed: `(0 - 6%3 + 3) % 3 = (0 - 0 + 3) % 3 = 0`
  - Check: `f[2][0] + 1 = 1 + 1 = 2 = f[3][0]` ✓
  - Select digit 6, move to state `f[2][0]`

- At i=2 (digit 1): Check if we used it
  - Previous remainder needed: `(0 - 1%3 + 3) % 3 = (0 - 1 + 3) % 3 = 2`
  - Check: `f[1][2] + 1 = -inf + 1 ≠ 1` ✗
  - Don't select digit 1, stay at state `f[2][0]`

- At i=1 (digit 0): Check if we used it
  - Previous remainder needed: `(0 - 0%3 + 3) % 3 = 0`
  - Check: `f[0][0] + 1 = 0 + 1 = 1 = f[1][0]` ✓
  - Select digit 0, move to state `f[0][0]`

Selected digits in order: `[8, 7, 6, 0]`

**Step 6: Handle leading zeros**
No leading zeros in our result. The final answer is **"8760"**.

Verification: 8 + 7 + 6 + 0 = 21, which is divisible by 3. ✓

## Python Solution

```python
class Solution:
    def largestMultipleOfThree(self, digits: List[int]) -> str:
        # Sort digits in ascending order for later reconstruction
        digits.sort()
        num_digits = len(digits)
      
        # Dynamic programming table
        # dp[i][j] represents the maximum number of digits we can use 
        # from the first i digits to form a number with remainder j when divided by 3
        dp = [[-float('inf')] * 3 for _ in range(num_digits + 1)]
      
        # Base case: using 0 digits gives remainder 0 with 0 digits used
        dp[0][0] = 0
      
        # Fill the DP table
        for i, digit in enumerate(digits, 1):
            for remainder in range(3):
                # Option 1: Don't include current digit
                dont_include = dp[i - 1][remainder]
              
                # Option 2: Include current digit
                # Calculate what remainder we need before adding this digit
                prev_remainder = (remainder - digit % 3 + 3) % 3
                include = dp[i - 1][prev_remainder] + 1
              
                # Take the maximum of both options
                dp[i][remainder] = max(dont_include, include)
      
        # If we can't form a number divisible by 3, return empty string
        if dp[num_digits][0] <= 0:
            return ""
      
        # Reconstruct the solution by backtracking through the DP table
        result_digits = []
        current_remainder = 0
      
        # Traverse from the last digit to the first
        for i in range(num_digits, 0, -1):
            current_digit = digits[i - 1]
          
            # Calculate what the remainder would have been before including this digit
            prev_remainder = (current_remainder - current_digit % 3 + 3) % 3
          
            # Check if this digit was included in the optimal solution
            if dp[i - 1][prev_remainder] + 1 == dp[i][current_remainder]:
                # This digit was included, add it to result
                result_digits.append(current_digit)
                current_remainder = prev_remainder
      
        # Remove leading zeros, keeping at least one digit
        leading_zero_index = 0
        while leading_zero_index < len(result_digits) - 1 and result_digits[leading_zero_index] == 0:
            leading_zero_index += 1
      
        # Convert digits to string and return
        return "".join(map(str, result_digits[leading_zero_index:]))
```

## Time and Space Complexity

**Time Complexity:** `O(n × log n)`

The time complexity is dominated by the following operations:
- `digits.sort()` takes `O(n × log n)` time where `n` is the length of the digits array
- The nested loop for dynamic programming fills a `(n+1) × 3` table, which takes `O(n × 3) = O(n)` time
- The backtracking loop to construct the result array runs in `O(n)` time
- The final loop to skip leading zeros takes `O(n)` time in the worst case
- Converting the array to string with `"".join(map(str, arr[i:]))` takes `O(n)` time

Since `O(n × log n) + O(n) + O(n) + O(n) = O(n × log n)`, the overall time complexity is `O(n × log n)`.

**Space Complexity:** `O(n)`

The space complexity consists of:
- The 2D DP table `f` with dimensions `(n+1) × 3`, which uses `O(n × 3) = O(n)` space
- The result array `arr` which can contain up to `n` elements, using `O(n)` space
- The sorted digits array (if not sorting in-place) would use `O(n)` space
- Other variables like `i`, `j`, `k` use `O(1)` space

Therefore, the total space complexity is `O(n) + O(n) + O(1) = O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Reconstruction Order Leading to Non-Optimal Results**

The most critical pitfall in this solution is that the backtracking reconstruction doesn't guarantee the largest possible number. The current approach adds digits to `result_digits` in the order they appear during backtracking (from largest index to smallest), but this doesn't ensure the digits are arranged to form the maximum value.

**The Problem:**
- When we sort digits in ascending order initially and backtrack from `i = n` to `i = 1`, we're adding digits to our result based on DP decisions, not based on maximizing the final number.
- For example, with `digits = [3, 1, 4, 1, 5, 9]`, the backtracking might select `[9, 5, 4]` but arrange them as `[9, 4, 5]` instead of `[9, 5, 4]`.

**The Solution:**
After collecting all selected digits during backtracking, sort them in descending order before constructing the final string:

```python
# After backtracking and collecting result_digits
result_digits.sort(reverse=True)  # Sort in descending order for maximum value

# Then handle leading zeros
leading_zero_index = 0
while leading_zero_index < len(result_digits) - 1 and result_digits[leading_zero_index] == 0:
    leading_zero_index += 1
```

### 2. **Edge Case: All Zeros**

Another pitfall is improperly handling the case where all selected digits are zeros.

**The Problem:**
- If `digits = [0, 0, 0]`, the sum is 0 (divisible by 3), so we should return "0"
- The current leading zero removal could accidentally return an empty string if not careful

**The Solution:**
Ensure at least one digit remains after removing leading zeros:

```python
# The current code handles this correctly with:
while leading_zero_index < len(result_digits) - 1 and result_digits[leading_zero_index] == 0:
    leading_zero_index += 1
# The condition `< len(result_digits) - 1` ensures at least one digit remains
```

### 3. **Unnecessary Initial Sorting**

The initial `digits.sort()` is actually counterproductive if we're going to sort the selected digits again later.

**The Problem:**
- Sorting the input doesn't help with the DP computation (order doesn't matter for calculating remainders)
- It might confuse the reconstruction logic

**The Solution:**
Remove the initial sort and only sort the final selected digits:

```python
def largestMultipleOfThree(self, digits: List[int]) -> str:
    # Don't sort initially
    num_digits = len(digits)
  
    # ... DP computation remains the same ...
  
    # After collecting result_digits in backtracking:
    result_digits.sort(reverse=True)  # Only sort here for maximum value
```

### 4. **Floating Point Infinity vs Integer Constants**

Using `-float('inf')` for impossible states works but can be less efficient than using a large negative integer.

**The Problem:**
- Floating-point operations are generally slower than integer operations
- Mixed float/int comparisons can lead to unexpected behavior in edge cases

**The Solution:**
Use a large negative integer constant instead:

```python
# Instead of:
dp = [[-float('inf')] * 3 for _ in range(num_digits + 1)]

# Use:
NEG_INF = -10**9  # Or -(num_digits + 1) since max count is num_digits
dp = [[NEG_INF] * 3 for _ in range(num_digits + 1)]
```

