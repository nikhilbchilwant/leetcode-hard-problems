# 2719. Count of Integers

> Source: https://algo.monster/liteproblems/2719

## Problem Statement

You are given two numeric strings `num1` and `num2` representing integers, and two integers `max_sum` and `min_sum`. Your task is to count how many integers `x` are "good" based on the following criteria:

1. `x` must be within the range `[num1, num2]` (inclusive), meaning `num1 <= x <= num2`
2. The sum of digits of `x` must be within the range `[min_sum, max_sum]` (inclusive), meaning `min_sum <= digit_sum(x) <= max_sum`

For example, if `x = 123`, then `digit_sum(x) = 1 + 2 + 3 = 6`.

Since the count of good integers can be very large, you need to return the result modulo `10^9 + 7`.

The challenge involves efficiently counting all integers in a potentially huge range that satisfy the digit sum constraint, without iterating through each number individually. The solution uses digit [dynamic programming](/problems/dynamic_programming_intro) to handle this efficiently by processing the numbers digit by digit from left to right, keeping track of the current digit sum and whether we've reached the upper bound of our range.

## Intuition

When we need to count integers in a range `[num1, num2]` that satisfy certain digit-based conditions, directly iterating through all numbers would be inefficient, especially for large ranges. Instead, we can transform this into a classic digit DP problem.

The key insight is to convert the range problem into a difference of two simpler problems. Rather than counting good integers in `[num1, num2]`, we can:
- Count good integers in `[0, num2]` 
- Count good integers in `[0, num1-1]`
- Subtract the second from the first to get our answer

For counting integers up to a given number with digit sum constraints, we can build the number digit by digit from left to right. At each position, we need to track:
1. **Current position** (`pos`): Which digit we're currently placing
2. **Current digit sum** (`s`): Sum of all digits we've placed so far
3. **Limit flag** (`limit`): Whether we're still bounded by the upper limit number

The `limit` flag is crucial - if we've been placing digits that match the upper bound so far (e.g., if the upper bound is `523` and we've placed `5` and `2`), then at the current position we can only place digits up to the corresponding digit in the upper bound (in this case, `3`). However, if we've already placed a smaller digit earlier (like placing `4` instead of `5` at the first position), then we're free to place any digit `0-9` at subsequent positions.

This recursive approach with memoization allows us to efficiently explore all valid numbers without explicitly generating them. When we reach the end of the number (processed all positions), we simply check if our accumulated digit sum falls within `[min_sum, max_sum]`.

The modulo operation `10^9 + 7` is applied during the computation to prevent integer overflow while maintaining the correct result.

**Learn more about [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements digit [dynamic programming](/problems/dynamic_programming_intro) with memoization. Here's how the implementation works:

**Main Function Structure:**
The `count` function handles the range calculation by computing the answer for `[0, num2]` and subtracting the answer for `[0, num1-1]`.

**Core DP Function - `dfs(pos, s, limit)`:**
- `pos`: Current digit position being processed (from left to right)
- `s`: Current sum of digits accumulated so far
- `limit`: Boolean flag indicating if we're still bounded by the upper limit

**Base Case:**
When `pos >= len(num)`, we've formed a complete number. We return `1` if the digit sum `s` is within `[min_sum, max_sum]`, otherwise return `0`.

**Recursive Case:**
1. Determine the upper bound for the current digit:
   - If `limit` is `True`: upper bound is `int(num[pos])` (the digit at current position in the limit number)
   - If `limit` is `False`: upper bound is `9` (we can use any digit)

2. Try all possible digits from `0` to `up`:
   - For each digit `i`, recursively call `dfs(pos + 1, s + i, limit and i == up)`
   - The new limit is `True` only if both current `limit` is `True` AND we chose the maximum possible digit `i == up`

3. Sum all possibilities and apply modulo `10^9 + 7`

**Memoization with `@cache`:**
The `@cache` decorator stores results of `dfs` calls to avoid redundant calculations. This is crucial for efficiency as many subproblems repeat.

**Computing the Final Answer:**
```python
num = num2
a = dfs(0, 0, True)  # Count good integers in [0, num2]
dfs.cache_clear()    # Clear cache before next computation
num = str(int(num1) - 1)  # Convert to num1-1
b = dfs(0, 0, True)  # Count good integers in [0, num1-1]
return (a - b) % mod  # Return the difference
```

The cache is cleared between computations because the `num` variable changes, and cached results from the first computation would be invalid for the second.

**Time Complexity:** `O(10 × n × max_sum)` where `n` is the length of the number string. For each position (n), each possible digit sum (up to max_sum), we might try up to 10 digits.

**Space Complexity:** `O(n × max_sum)` for the memoization cache storing unique states.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example:** Count good integers where `num1 = "10"`, `num2 = "15"`, `min_sum = 3`, `max_sum = 5`

We need to find integers in range [10, 15] whose digit sums are in [3, 5].

**Step 1: Transform the problem**
Instead of directly counting in [10, 15], we compute:
- Count of good integers in [0, 15]
- Count of good integers in [0, 9] (which is num1-1)
- Answer = first count - second count

**Step 2: Count good integers in [0, 15] using dfs(pos, s, limit)**

Let's trace through building numbers digit by digit up to "15":

Starting with `dfs(0, 0, True)`:
- Position 0 (tens place), sum=0, limit=True
- Upper bound is '1' (first digit of "15")
- Try digit 0: leads to numbers 00-09
  - `dfs(1, 0, False)` - limit becomes False since 0 < 1
  - At position 1, can use any digit 0-9
  - Results: 03(sum=3✓), 04(sum=4✓), 05(sum=5✓) → 3 good numbers
- Try digit 1: leads to numbers 10-15
  - `dfs(1, 1, True)` - limit stays True since 1 == 1
  - At position 1, upper bound is '5' (second digit of "15")
  - Try digits 0-5:
    - 10: sum=1+0=1 ✗
    - 11: sum=1+1=2 ✗
    - 12: sum=1+2=3 ✓
    - 13: sum=1+3=4 ✓
    - 14: sum=1+4=5 ✓
    - 15: sum=1+5=6 ✗
  - Results: 3 good numbers

Total for [0, 15]: 3 + 3 = 6 good numbers

**Step 3: Count good integers in [0, 9]**

Starting with `dfs(0, 0, True)`:
- Position 0 (ones place), sum=0, limit=True
- Upper bound is '9' (the only digit of "9")
- Try digits 0-9:
  - 0: sum=0 ✗
  - 1: sum=1 ✗
  - 2: sum=2 ✗
  - 3: sum=3 ✓
  - 4: sum=4 ✓
  - 5: sum=5 ✓
  - 6: sum=6 ✗
  - 7: sum=7 ✗
  - 8: sum=8 ✗
  - 9: sum=9 ✗

Total for [0, 9]: 3 good numbers

**Step 4: Calculate final answer**
Answer = 6 - 3 = 3

The good integers in [10, 15] are: 12 (sum=3), 13 (sum=4), 14 (sum=5)

**Key Observations:**
1. The `limit` flag controls whether we can use any digit (0-9) or are restricted by the upper bound
2. Once we place a digit smaller than the bound, all subsequent positions become unrestricted
3. Memoization prevents recalculating the same (pos, s, limit) states multiple times
4. The digit sum constraint is only checked when we've built a complete number (base case)

## Python Solution

```python
class Solution:
    def count(self, num1: str, num2: str, min_sum: int, max_sum: int) -> int:
        """
        Count integers between num1 and num2 (inclusive) whose digit sum is between min_sum and max_sum.
        Uses digit DP technique to efficiently count valid numbers.
      
        Args:
            num1: Lower bound as string
            num2: Upper bound as string
            min_sum: Minimum allowed digit sum
            max_sum: Maximum allowed digit sum
          
        Returns:
            Count of valid integers modulo 10^9 + 7
        """
        from functools import cache
      
        MOD = 10**9 + 7
      
        @cache
        def digit_dp(position: int, digit_sum: int, is_limit: bool, current_number: str) -> int:
            """
            Dynamic programming function to count valid numbers.
          
            Args:
                position: Current position in the number being formed
                digit_sum: Sum of digits chosen so far
                is_limit: Whether we're still bounded by the original number's digits
                current_number: The upper bound number we're comparing against
              
            Returns:
                Count of valid numbers from this state
            """
            # Base case: reached end of number
            if position >= len(current_number):
                # Check if digit sum is within valid range
                return 1 if min_sum <= digit_sum <= max_sum else 0
          
            # Determine the maximum digit we can place at current position
            # If limited by original number, use its digit; otherwise use 9
            max_digit = int(current_number[position]) if is_limit else 9
          
            # Try all possible digits from 0 to max_digit
            total_count = 0
            for digit in range(max_digit + 1):
                # Recurse to next position
                # Update digit_sum and check if we're still at limit
                total_count += digit_dp(
                    position + 1,
                    digit_sum + digit,
                    is_limit and (digit == max_digit),
                    current_number
                )
                total_count %= MOD
          
            return total_count
      
        # Count valid numbers from 0 to num2
        count_up_to_num2 = digit_dp(0, 0, True, num2)
      
        # Clear cache before computing for num1
        digit_dp.cache_clear()
      
        # Count valid numbers from 0 to (num1 - 1)
        # This excludes num1 from the count
        num1_minus_one = str(int(num1) - 1)
        count_up_to_num1_minus_one = digit_dp(0, 0, True, num1_minus_one)
      
        # Return difference to get count in range [num1, num2]
        return (count_up_to_num2 - count_up_to_num1_minus_one) % MOD
```

## Time and Space Complexity

**Time Complexity:** `O(n * S * d)` where `n` is the length of the longer number between `num1` and `num2`, `S` is the maximum possible sum of digits (which is `9 * n`), and `d` is the number of possible digits (10).

The DFS function explores states defined by:
- `pos`: position in the number (0 to n-1)
- `s`: current sum of digits (0 to 9*n)
- `limit`: boolean flag

For each state, we iterate through at most 10 digits (0-9). Due to memoization with `@cache`, each unique state is computed only once. The total number of unique states is `n * (9n + 1) * 2 = O(n²)`. Since we iterate through up to 10 digits for each state, the overall time complexity is `O(10 * n²) = O(n²)`.

However, more precisely, the time complexity is `O(n * S)` where `S` represents the range of possible sums, which is bounded by `9n`. Since we call the function twice (once for `num2` and once for `num1 - 1`), the complexity remains `O(n * S)`.

**Space Complexity:** `O(n * S)` where `n` is the length of the number and `S` is the maximum possible sum.

The space is used for:
- The memoization cache storing results for each unique `(pos, s, limit)` combination
- The recursion stack depth which is at most `n`

The cache stores at most `n * (9n + 1) * 2` states, which simplifies to `O(n²)` or more precisely `O(n * S)` space. The recursion stack adds `O(n)` space, but this is dominated by the cache size.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of num1 - 1 Leading to Edge Cases**

**The Pitfall:**
When computing `str(int(num1) - 1)`, there's a critical edge case when `num1 = "0"`. This results in `int("0") - 1 = -1`, which becomes `"-1"` as a string. The digit DP function isn't designed to handle negative numbers, leading to incorrect results or unexpected behavior.

**Example:**
```python
num1 = "0"
num1_minus_one = str(int(num1) - 1)  # Results in "-1"
# The DP function will process "-1" incorrectly
```

**Solution:**
Add a special check for when `num1 = "0"`:
```python
if num1 == "0":
    # Special case: when num1 is 0, we don't need to subtract anything
    count_up_to_num1_minus_one = 0
else:
    num1_minus_one = str(int(num1) - 1)
    count_up_to_num1_minus_one = digit_dp(0, 0, True, num1_minus_one)
```

### 2. **Cache Pollution Between Different Problem Instances**

**The Pitfall:**
The `@cache` decorator maintains state between different calls to the `count` method. If the Solution class instance is reused for multiple test cases, the cache from previous computations can interfere with new ones, especially since the parameters `min_sum` and `max_sum` are used inside the cached function but aren't part of the cache key.

**Example:**
```python
sol = Solution()
# First call with min_sum=5, max_sum=10
result1 = sol.count("1", "100", 5, 10)
# Second call with different min_sum/max_sum
result2 = sol.count("1", "100", 3, 8)  # May use cached values with wrong sum bounds!
```

**Solution:**
Either include all relevant parameters in the function signature or clear the cache at the beginning of each `count` call:
```python
def count(self, num1: str, num2: str, min_sum: int, max_sum: int) -> int:
    from functools import cache
  
    MOD = 10**9 + 7
  
    @cache
    def digit_dp(position: int, digit_sum: int, is_limit: bool, 
                 current_number: str, min_s: int, max_s: int) -> int:
        # Include min_s and max_s as parameters
        if position >= len(current_number):
            return 1 if min_s <= digit_sum <= max_s else 0
        # ... rest of the function
  
    # Pass min_sum and max_sum explicitly
    count_up_to_num2 = digit_dp(0, 0, True, num2, min_sum, max_sum)
```

### 3. **Integer Overflow in Modulo Operation**

**The Pitfall:**
When computing `(count_up_to_num2 - count_up_to_num1_minus_one) % MOD`, if `count_up_to_num1_minus_one > count_up_to_num2`, the subtraction results in a negative number. In Python, this isn't typically an issue as the modulo operation handles negatives correctly, but in other languages or when porting this solution, it can cause problems.

**Example:**
```python
# If count_up_to_num2 = 5 and count_up_to_num1_minus_one = 8
result = (5 - 8) % MOD  # In Python: gives correct positive result
# In some other languages: might give negative result
```

**Solution:**
Ensure the result is always positive by adding MOD before taking modulo:
```python
return ((count_up_to_num2 - count_up_to_num1_minus_one) % MOD + MOD) % MOD
```
Or more elegantly:
```python
return (count_up_to_num2 - count_up_to_num1_minus_one + MOD) % MOD
```

