# 2827. Number of Beautiful Integers in the Range

> Source: https://algo.monster/liteproblems/2827

## Problem Statement

You are given three positive integers: `low`, `high`, and `k`.

A number is considered **beautiful** if it satisfies both of these conditions:
1. The count of even digits in the number equals the count of odd digits
2. The number is divisible by `k`

Your task is to find and return the total number of beautiful integers within the range `[low, high]` (inclusive).

For example, if we have the number `1234`:
- Even digits: `2`, `4` (count = 2)
- Odd digits: `1`, `3` (count = 2)
- Since the counts are equal, the first condition is satisfied
- If this number is also divisible by `k`, then it would be beautiful

The solution uses digit [dynamic programming](/problems/dynamic_programming_intro) (Digit DP) to efficiently count beautiful numbers. The approach converts the problem into finding beautiful numbers in `[1, high]` and subtracting beautiful numbers in `[1, low-1]`. The DP function tracks:
- Current position in the digit sequence
- Current modulo value with respect to `k`
- Difference between odd and even digit counts (using `diff = 10` as initial state to handle negative differences)
- Whether we're still in leading zeros
- Whether we've reached the upper limit of digits

The recursion explores all valid digit choices at each position, maintaining the constraints and counting valid beautiful numbers when all digits are processed (where `mod == 0` and `diff == 10`, indicating equal odd/even counts and divisibility by `k`).

## Intuition

When we need to count numbers with specific properties in a range `[low, high]`, a common technique is to transform it into counting numbers in `[0, high]` minus numbers in `[0, low-1]`. This simplifies our problem to just counting from 0 up to some number.

For counting numbers with digit-related properties up to a limit, digit DP is a powerful technique. The key insight is that we can build valid numbers digit by digit, keeping track of the constraints we need to satisfy.

In this problem, we need to track:
1. **Divisibility by k**: As we build the number digit by digit, we can track `(current_number % k)`. When we place a new digit `d` at the end, the new modulo becomes `(old_mod * 10 + d) % k`.

2. **Equal count of odd and even digits**: Instead of tracking two separate counts, we can use a single difference value. When we add an odd digit, we increment the difference; when we add an even digit, we decrement it. A beautiful number will have difference = 0 at the end.

3. **Leading zeros**: Numbers don't actually start with zeros (e.g., 007 is just 7). Leading zeros shouldn't count toward our odd/even digit balance. We need a flag to track whether we're still in the "leading zero" phase.

4. **Upper bound constraint**: When counting up to a number like 523, we can't freely choose any digit at each position. For the first digit, we can only choose 0-5. If we chose 5, then for the second digit we can only choose 0-2. If we chose 52, the third digit can only be 0-3. We need to track whether we're still bounded by the limit.

The trick with the difference value is using an offset (like 10) to avoid negative numbers in our DP state, since having more even digits than odd would give negative differences. Starting at 10 and returning to 10 means we have equal counts.

By memoizing our recursive calls with these states, we avoid recalculating the same subproblems, making the solution efficient even for large ranges.

**Learn more about [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements digit DP using a recursive function with memoization. Let's walk through the implementation step by step:

### Main Function Structure

The main function computes two values:
- `a`: Count of beautiful numbers in `[1, high]`
- `b`: Count of beautiful numbers in `[1, low-1]`
- Final answer: `a - b`

### DFS Function Parameters

The recursive function `dfs(pos, mod, diff, lead, limit)` tracks five states:

1. **`pos`**: Current position in the digit string (0-indexed)
2. **`mod`**: Current number modulo `k` (tracks divisibility)
3. **`diff`**: Difference between odd and even digit counts + 10 (offset to avoid negatives)
4. **`lead`**: Boolean flag for leading zeros (1 if still in leading zeros, 0 otherwise)
5. **`limit`**: Boolean flag for upper bound constraint (1 if bounded by input number, 0 otherwise)

### Base Case

When `pos >= len(s)`, we've built a complete number:
- Return 1 if `mod == 0` (divisible by `k`) and `diff == 10` (equal odd/even counts)
- Return 0 otherwise

### Recursive Case

For each position, we determine the maximum digit we can place:
```python
up = int(s[pos]) if limit else 9
```

Then we try each digit from 0 to `up`:

**Case 1: Leading Zero** (`i == 0` and `lead == 1`)
- Continue with leading zeros
- Don't modify `mod` or `diff`
- Recursive call: `dfs(pos + 1, mod, diff, 1, limit and i == up)`

**Case 2: Regular Digit**
- Update difference: 
  - Add 1 if digit is odd (`i % 2 == 1`)
  - Subtract 1 if digit is even
- Update modulo: `(mod * 10 + i) % k`
- Set `lead = 0` (no longer in leading zeros)
- Update `limit`: remains true only if we chose the maximum allowed digit

### Memoization with @cache

The `@cache` decorator automatically memoizes the function results. For each unique combination of `(pos, mod, diff, lead, limit)`, the result is computed only once.

### Algorithm Flow

1. Convert `high` to string `s`
2. Call `dfs(0, 0, 10, 1, 1)` to count beautiful numbers up to `high`
3. Clear the cache with `dfs.cache_clear()`
4. Convert `low - 1` to string `s`
5. Call `dfs(0, 0, 10, 1, 1)` to count beautiful numbers up to `low - 1`
6. Return the difference

### Time and Space Complexity

- **Time Complexity**: `O((log M)² × k × 10)` where `M` is the value of `high`
  - `log M` positions to fill
  - `log M` possible values for `diff` (ranging from 0 to 20 in practice)
  - `k` possible values for `mod`
  - Up to 10 digits to try at each position

- **Space Complexity**: `O((log M)² × k)` for the memoization cache

The digit DP approach efficiently handles the large range by avoiding enumeration of all numbers, instead building them digit by digit while maintaining the constraints.

## Example Walkthrough

Let's walk through finding beautiful numbers in the range `[1, 22]` with `k = 2`.

**Step 1: Transform the problem**
- Count beautiful numbers in `[1, 22]`
- This equals: beautiful numbers in `[1, 22]` minus beautiful numbers in `[1, 0]`

**Step 2: Count beautiful numbers up to 22**

We'll use our digit DP function `dfs(pos, mod, diff, lead, limit)` with string `s = "22"`.

Starting state: `dfs(0, 0, 10, 1, 1)`
- `pos = 0`: We're at the first digit
- `mod = 0`: Current number mod 2 is 0
- `diff = 10`: No digits yet (10 is our neutral state)
- `lead = 1`: Still in leading zeros
- `limit = 1`: Bounded by input "22"

**First digit (position 0):**
- Maximum digit allowed: 2 (because `s[0] = '2'` and `limit = 1`)
- Try digit 0: Leading zero, recurse with `dfs(1, 0, 10, 1, 0)`
- Try digit 1: Odd digit, recurse with `dfs(1, 1, 11, 0, 0)`
  - `mod = (0 * 10 + 1) % 2 = 1`
  - `diff = 10 + 1 = 11` (one more odd than even)
- Try digit 2: Even digit, recurse with `dfs(1, 0, 9, 0, 1)`
  - `mod = (0 * 10 + 2) % 2 = 0`
  - `diff = 10 - 1 = 9` (one more even than odd)
  - `limit` stays 1 because we chose the max digit

**Following path for number 12:**
- First digit = 1: `dfs(1, 1, 11, 0, 0)`
- Second digit options: 0-9 (no limit)
  - Choose 2: `dfs(2, 0, 10, 0, 0)`
    - `mod = (1 * 10 + 2) % 2 = 0` ✓
    - `diff = 11 - 1 = 10` ✓
  - Base case reached: Returns 1 (12 is beautiful!)

**Beautiful numbers found in [1, 22]:**
- Single digit: None (can't have equal odd/even counts)
- Two digits: 10, 12, 14, 16, 18 (all have 1 odd, 1 even, divisible by 2)
- Count = 5

**Step 3: Count beautiful numbers up to 0**
- String `s = "0"`
- `dfs(0, 0, 10, 1, 1)` returns 0 (no beautiful numbers)

**Final answer:** 5 - 0 = 5

The algorithm efficiently explores all valid digit combinations while maintaining:
- Divisibility tracking through `mod`
- Digit count balance through `diff`
- Valid number construction through `lead` and `limit` flags

This avoids checking all 22 numbers individually, instead building valid numbers digit by digit with memoization preventing redundant calculations.

## Python Solution

```python
from functools import cache
from typing import List

class Solution:
    def numberOfBeautifulIntegers(self, low: int, high: int, k: int) -> int:
        """
        Count beautiful integers in range [low, high] that are divisible by k.
        A beautiful integer has equal count of odd and even digits.
        """
      
        @cache
        def count_beautiful_up_to_n(
            position: int, 
            remainder: int, 
            odd_even_difference: int, 
            is_leading_zero: int, 
            is_bounded: int
        ) -> int:
            """
            Digit DP to count beautiful integers up to current number.
          
            Args:
                position: Current digit position being processed
                remainder: Current remainder when divided by k
                odd_even_difference: Difference between odd and even digit counts (offset by 10)
                is_leading_zero: 1 if still processing leading zeros, 0 otherwise
                is_bounded: 1 if current number is still bounded by original limit, 0 otherwise
          
            Returns:
                Count of beautiful integers satisfying all conditions
            """
            # Base case: processed all digits
            if position >= len(number_string):
                # Check if divisible by k and has equal odd/even digits (difference = 10 means 0)
                return 1 if remainder == 0 and odd_even_difference == 10 else 0
          
            # Determine upper bound for current digit
            upper_bound = int(number_string[position]) if is_bounded else 9
          
            result = 0
          
            # Try all possible digits at current position
            for digit in range(upper_bound + 1):
                if digit == 0 and is_leading_zero:
                    # Skip leading zeros - they don't affect odd/even count
                    result += count_beautiful_up_to_n(
                        position + 1, 
                        remainder, 
                        odd_even_difference, 
                        1,  # Still have leading zeros
                        is_bounded and (digit == upper_bound)
                    )
                else:
                    # Process actual digit
                    # Update odd/even difference: +1 for odd, -1 for even
                    new_difference = odd_even_difference + (1 if digit % 2 == 1 else -1)
                  
                    result += count_beautiful_up_to_n(
                        position + 1,
                        (remainder * 10 + digit) % k,  # Update remainder
                        new_difference,
                        0,  # No longer have leading zeros
                        is_bounded and (digit == upper_bound)
                    )
          
            return result
      
        # Count beautiful integers from 0 to high
        number_string = str(high)
        count_up_to_high = count_beautiful_up_to_n(0, 0, 10, 1, 1)
      
        # Clear cache before processing lower bound
        count_beautiful_up_to_n.cache_clear()
      
        # Count beautiful integers from 0 to (low - 1)
        number_string = str(low - 1)
        count_up_to_low_minus_1 = count_beautiful_up_to_n(0, 0, 10, 1, 1)
      
        # Return count in range [low, high]
        return count_up_to_high - count_up_to_low_minus_1
```

## Time and Space Complexity

**Time Complexity:** `O(log(high) * k * log(high) * 10)`

The time complexity is determined by the digit DP (dynamic programming) approach:
- The function processes numbers digit by digit, with at most `log(high)` digits
- For each position `pos`, we have at most `log(high)` states (from 0 to length of string)
- The `mod` parameter can have `k` different values (0 to k-1)
- The `diff` parameter represents the difference between odd and even digit counts, which ranges from `-log(high)` to `log(high)`, giving approximately `2*log(high)` possible values
- The `lead` parameter has 2 possible values (0 or 1)
- The `limit` parameter has 2 possible values (0 or 1)
- For each state, we iterate through at most 10 digits (0-9)

Total states: `log(high) * k * 2*log(high) * 2 * 2 = O(log²(high) * k)`
Since each state processes up to 10 digits, the overall time complexity is `O(log²(high) * k * 10) = O(k * log²(high))`

**Space Complexity:** `O(log²(high) * k)`

The space complexity comes from:
- The recursion stack depth: `O(log(high))` for the maximum number of digits
- The memoization cache storing all possible states: `O(log(high) * k * 2*log(high) * 2 * 2) = O(k * log²(high))`

The cache dominates the space complexity, resulting in `O(k * log²(high))`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of the Odd-Even Difference Offset**

**Pitfall**: The most common mistake is forgetting why we use `diff = 10` as the initial value and offset. Since we're tracking the difference between odd and even digit counts, this value can go negative. For example, if we have 3 even digits and 1 odd digit, the difference would be -2. Without an offset, negative values would cause issues with memoization and array indexing.

**Incorrect approach**:
```python
# Starting with diff = 0 without offset
def dfs(pos, mod, diff, lead, limit):
    if pos >= len(s):
        return 1 if mod == 0 and diff == 0 else 0
    # ...
    new_diff = diff + (1 if i % 2 == 1 else -1)  # Can go negative!
```

**Correct approach**:
```python
# Start with diff = 10 as offset
def dfs(pos, mod, diff, lead, limit):
    if pos >= len(s):
        return 1 if mod == 0 and diff == 10 else 0  # 10 represents balanced
    # ...
    new_diff = diff + (1 if i % 2 == 1 else -1)  # Safe with offset
```

### 2. **Forgetting to Clear Cache Between Calls**

**Pitfall**: When using `@cache` decorator, the memoization persists between function calls. If you don't clear the cache between counting for `high` and `low-1`, you'll get incorrect results because the cached values from the first call will be used in the second call, even though the input string `s` has changed.

**Incorrect approach**:
```python
s = str(high)
a = dfs(0, 0, 10, 1, 1)
# Missing cache clear!
s = str(low - 1)
b = dfs(0, 0, 10, 1, 1)  # Will use cached values from previous s!
```

**Correct approach**:
```python
s = str(high)
a = dfs(0, 0, 10, 1, 1)
dfs.cache_clear()  # Essential!
s = str(low - 1)
b = dfs(0, 0, 10, 1, 1)
```

### 3. **Incorrect Modulo Update for Leading Zeros**

**Pitfall**: When encountering a leading zero, some might incorrectly update the modulo value. Leading zeros should not affect the number's value or its remainder when divided by k.

**Incorrect approach**:
```python
if i == 0 and lead == 1:
    # Wrong: updating mod even for leading zeros
    result += dfs(pos + 1, (mod * 10) % k, diff, 1, limit and i == up)
```

**Correct approach**:
```python
if i == 0 and lead == 1:
    # Correct: mod stays unchanged for leading zeros
    result += dfs(pos + 1, mod, diff, 1, limit and i == up)
```

### 4. **Mishandling the Limit Flag Update**

**Pitfall**: The limit flag should only remain true if we choose the maximum allowed digit at the current position. A common mistake is to always pass the same limit value or incorrectly update it.

**Incorrect approaches**:
```python
# Wrong: Always passing the same limit
result += dfs(pos + 1, new_mod, new_diff, 0, limit)

# Wrong: Setting limit to 0 unconditionally
result += dfs(pos + 1, new_mod, new_diff, 0, 0)
```

**Correct approach**:
```python
# Limit remains true only if we chose the upper bound digit
result += dfs(pos + 1, new_mod, new_diff, 0, limit and (i == up))
```

### 5. **Edge Case: When low = 0**

**Pitfall**: The problem states that low, high, and k are positive integers, but if the constraints were different and `low` could be 0, computing `str(low - 1)` would give "-1", causing the algorithm to fail.

**Solution for edge case**:
```python
if low == 0:
    # Handle separately if needed
    count_up_to_low_minus_1 = 0
else:
    number_string = str(low - 1)
    count_up_to_low_minus_1 = count_beautiful_up_to_n(0, 0, 10, 1, 1)
```

### 6. **Not Considering Single-Digit Numbers Properly**

**Pitfall**: For single-digit numbers, there's only one digit which is either odd or even, so they can never have equal counts of odd and even digits. The algorithm handles this correctly through the difference mechanism, but it's important to understand that no single-digit number can be beautiful (except if we counted leading zeros, which we don't).

These pitfalls highlight the importance of carefully tracking state variables, properly handling edge cases, and understanding the mathematical properties of the problem when implementing digit DP solutions.

