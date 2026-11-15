# 2801. Count Stepping Numbers in Range

> Source: https://algo.monster/liteproblems/2801

## Problem Statement

You are given two positive integers `low` and `high` represented as strings. Your task is to find the count of **stepping numbers** in the inclusive range `[low, high]`.

A **stepping number** is an integer where all adjacent digits have an absolute difference of exactly `1`. For example:
- `12` is a stepping number because `|1-2| = 1`
- `321` is a stepping number because `|3-2| = 1` and `|2-1| = 1`
- `13` is NOT a stepping number because `|1-3| = 2`

Important constraints:
- Stepping numbers should not have leading zeros
- Since the answer may be very large, return it modulo `10^9 + 7`

The solution uses a digit [dynamic programming](/problems/dynamic_programming_intro) approach. The key idea is to break down the problem into two parts: counting stepping numbers from `1` to `high` and from `1` to `low-1`, then finding the difference.

The `dfs` function tracks:
- `pos`: Current digit position being processed
- `pre`: Previous digit value (or `-1` if no previous digit)
- `lead`: Whether we're still in leading zeros
- `limit`: Whether we're bounded by the upper limit of the current number

For each position, we try placing digits `0` to `9` (or up to the limit), ensuring that:
1. If we're placing a non-zero digit or we're past leading zeros, the absolute difference with the previous digit must be exactly `1`
2. We properly handle leading zeros (they don't count as part of the stepping number check)

## Intuition

When faced with counting problems over a large range of numbers, especially when the constraint involves relationships between digits (like adjacent digits differing by exactly `1`), we should think about digit [dynamic programming](/problems/dynamic_programming_intro).

The key insight is that we can build valid stepping numbers digit by digit from left to right. At each position, we only need to know what the previous digit was to determine what valid choices we have for the current digit. If the previous digit was `5`, we can only place `4` or `6` next. This local constraint makes the problem perfect for [dynamic programming](/problems/dynamic_programming_intro).

Why can't we just iterate through all numbers from `low` to `high`? The range could be enormous (up to `10^100` or more since they're given as strings), making direct enumeration impossible.

The standard technique for range queries `[L, R]` is to transform it into `count(R) - count(L-1)`. This way, we only need to solve the problem of counting stepping numbers from `0` up to some value `N`.

For the digit DP approach, we process the number digit by digit. The crucial observation is that we need to track:
- Whether we're still bounded by the original number (if we've placed a smaller digit earlier, we're free to place any valid digit now)
- Whether we're still in the leading zeros phase (leading zeros don't count for the stepping number property)
- What the previous digit was (to enforce the stepping constraint)

The beauty of this approach is that it explores only valid paths through the digit choices, avoiding the need to check every number in the potentially massive range. By memoizing our recursive calls, we ensure that each unique state `(position, previous_digit, has_leading_zeros, is_limited)` is computed only once, giving us an efficient solution.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements digit DP using a recursive function with memoization. Let's walk through the implementation step by step:

### Core Function: `dfs(pos, pre, lead, limit)`

The function parameters represent:
- `pos`: Current position in the number string (0-indexed from left)
- `pre`: Previous digit placed (-1 if no digit placed yet)
- `lead`: Boolean indicating if we're still in leading zeros
- `limit`: Boolean indicating if we must respect the upper bound of the original number

### Base Case
```python
if pos >= len(num):
    return int(not lead)
```
When we've processed all positions, we return `1` if we've formed a valid number (not all leading zeros), otherwise `0`.

### Recursive Case

First, we determine the upper bound for the current digit:
```python
up = int(num[pos]) if limit else 9
```
If `limit` is `True`, we can only go up to the digit at position `pos` in the original number. Otherwise, we can use any digit from `0` to `9`.

Then we enumerate all possible digits for the current position:
```python
for i in range(up + 1):
```

For each digit `i`, we have two cases:

**Case 1: Leading Zero**
```python
if i == 0 and lead:
    ans += dfs(pos + 1, pre, True, limit and i == up)
```
If we place a `0` and we're still in the leading zeros phase, we continue with leading zeros. The `pre` remains unchanged since this zero doesn't count as part of the number.

**Case 2: Valid Stepping Digit**
```python
elif pre == -1 or abs(i - pre) == 1:
    ans += dfs(pos + 1, i, False, limit and i == up)
```
We can place digit `i` if:
- It's the first non-zero digit (`pre == -1`)
- It differs from the previous digit by exactly 1 (`abs(i - pre) == 1`)

The new state has `lead = False` since we've placed a non-leading digit, and `limit` remains true only if we placed the maximum allowed digit.

### Computing the Final Answer

To find stepping numbers in `[low, high]`:

1. Calculate stepping numbers from `1` to `high`:
```python
num = high
a = dfs(0, -1, True, True)
```

2. Calculate stepping numbers from `1` to `low - 1`:
```python
num = str(int(low) - 1)
b = dfs(0, -1, True, True)
```

3. Return the difference:
```python
return (a - b) % mod
```

### Optimization with Memoization

The `@cache` decorator automatically memoizes the results of `dfs` calls. This ensures each unique state is computed only once, dramatically reducing the time complexity. The cache is cleared between computing `a` and `b` using `dfs.cache_clear()` since they work with different target numbers.

### Time and Space Complexity

- **Time Complexity**: `O(log M × |Σ|²)` where `M` is the value of `high` and `|Σ| = 10` (digit set size)
- **Space Complexity**: `O(log M × |Σ|)` for the memoization cache

The digit DP approach elegantly handles the massive range by only exploring valid digit combinations, making it feasible to solve problems with numbers having hundreds of digits.

## Example Walkthrough

Let's walk through finding stepping numbers in the range `[10, 15]`.

**Step 1: Calculate stepping numbers from 1 to 15**

We'll trace through building stepping numbers up to 15 using `dfs(pos, pre, lead, limit)`:

Starting with `num = "15"`, `dfs(0, -1, True, True)`:
- Position 0, we can place digits 0 or 1 (limited by '1' in "15")
  - If we place 0: Continue with leading zeros, `dfs(1, -1, True, False)`
  - If we place 1: First real digit, `dfs(1, 1, False, True)`

Following the path where we placed 1:
- Position 1, previous digit is 1, limit is True (bound by '5')
  - Valid next digits must satisfy `|digit - 1| = 1`, so only 0 or 2
  - We can place 0: `|0-1| = 1` ✓, gives us "10" 
  - We can place 2: `|2-1| = 1` ✓, gives us "12"
  - We cannot place 3,4,5: `|3-1| = 2`, `|4-1| = 3`, `|5-1| = 4` ✗

The complete stepping numbers from 1 to 15 are:
- Single digits: 1, 2, 3, 4, 5, 6, 7, 8, 9 (9 numbers)
- Two digits: 10, 12 (2 numbers)
- Total: 11 stepping numbers

**Step 2: Calculate stepping numbers from 1 to 9 (low - 1)**

With `num = "9"`, `dfs(0, -1, True, True)`:
- Position 0, we can place digits 0 to 9
  - Placing 1-9 gives us the single-digit stepping numbers: 1,2,3,4,5,6,7,8,9
- Total: 9 stepping numbers

**Step 3: Find the answer**

Stepping numbers in `[10, 15]` = Stepping numbers in `[1, 15]` - Stepping numbers in `[1, 9]`
= 11 - 9 = 2

The stepping numbers are: **10** and **12**

Let's verify:
- 10: `|1-0| = 1` ✓
- 11: `|1-1| = 0` ✗ (not a stepping number)
- 12: `|1-2| = 1` ✓ 
- 13: `|1-3| = 2` ✗
- 14: `|1-4| = 3` ✗
- 15: `|1-5| = 4` ✗

The digit DP approach efficiently found both valid stepping numbers without checking every number in the range!

## Python Solution

```python
class Solution:
    def countSteppingNumbers(self, low: str, high: str) -> int:
        """
        Count stepping numbers in range [low, high].
        A stepping number is a number where each digit differs from its adjacent digit by exactly 1.
        Uses digit DP technique to count numbers within bounds.
        """
        from functools import cache
      
        MOD = 10**9 + 7
      
        @cache
        def count_stepping_numbers_up_to(number_str: str) -> int:
            """
            Count stepping numbers from 0 to number_str using digit DP.
            """
            @cache
            def dp(position: int, previous_digit: int, is_leading_zero: bool, is_bounded: bool) -> int:
                """
                Dynamic programming function to count stepping numbers.
              
                Args:
                    position: Current position in the number being formed
                    previous_digit: Previous digit placed (-1 if no digit placed yet)
                    is_leading_zero: True if we're still in leading zeros
                    is_bounded: True if we must respect the upper bound at current position
              
                Returns:
                    Count of valid stepping numbers from this state
                """
                # Base case: reached end of number
                if position >= len(number_str):
                    # Return 1 if we've placed at least one non-zero digit
                    return int(not is_leading_zero)
              
                # Determine the maximum digit we can place at current position
                max_digit = int(number_str[position]) if is_bounded else 9
              
                count = 0
              
                # Try placing each possible digit at current position
                for digit in range(max_digit + 1):
                    # Case 1: Continue with leading zeros
                    if digit == 0 and is_leading_zero:
                        count += dp(
                            position + 1,
                            previous_digit,
                            True,  # Still have leading zeros
                            is_bounded and (digit == max_digit)
                        )
                    # Case 2: Place a valid digit (first digit or stepping constraint satisfied)
                    elif previous_digit == -1 or abs(digit - previous_digit) == 1:
                        count += dp(
                            position + 1,
                            digit,
                            False,  # No longer have leading zeros
                            is_bounded and (digit == max_digit)
                        )
              
                return count % MOD
          
            return dp(0, -1, True, True)
      
        # Count stepping numbers up to high
        count_up_to_high = count_stepping_numbers_up_to(high)
      
        # Clear cache before computing for low-1
        count_stepping_numbers_up_to.cache_clear()
      
        # Count stepping numbers up to (low - 1)
        low_minus_one = str(int(low) - 1)
        count_up_to_low_minus_one = count_stepping_numbers_up_to(low_minus_one)
      
        # Return the difference to get count in range [low, high]
        return (count_up_to_high - count_up_to_low_minus_one) % MOD
```

## Time and Space Complexity

**Time Complexity:** `O(n * 10 * 2 * 2)` = `O(n)` where `n` is the number of digits in the input number.

The analysis breaks down as follows:
- The DFS function explores states defined by four parameters: `(pos, pre, lead, limit)`
- `pos` ranges from 0 to `n-1` (length of the number string)
- `pre` can take values from -1 to 9 (11 possible values)
- `lead` is a boolean (2 possible values)
- `limit` is a boolean (2 possible values)
- Total possible states: `n * 11 * 2 * 2` = `44n`
- Each state is computed once due to memoization with `@cache`
- Within each state, we iterate through at most 10 digits (0-9)
- Therefore, total operations: `O(n * 11 * 2 * 2 * 10)` = `O(440n)` = `O(n)`

Since the function is called twice (once for `high` and once for `low-1`), the overall time complexity remains `O(n)`.

**Space Complexity:** `O(n * 11 * 2 * 2)` = `O(n)`

The space is consumed by:
- The memoization cache stores all possible states: `n * 11 * 2 * 2` = `44n` states
- The recursion stack depth is at most `n` (the length of the number)
- Therefore, the total space complexity is `O(44n + n)` = `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of the Lower Bound Subtraction**

**The Pitfall:**
When calculating stepping numbers in range `[low, high]`, developers often make the mistake of directly converting `low` to an integer for subtraction without considering edge cases:

```python
# WRONG: This fails when low = "10"
low_minus_one = str(int(low) - 1)  # Results in "9"
```

The problem occurs when `low - 1` results in a number with fewer digits. For example, if `low = "10"`, then `low - 1 = 9`. This can cause issues if the digit DP implementation doesn't properly handle numbers with different lengths, or when `low = "1"` (resulting in `"0"`), which needs special treatment.

**Solution:**
Handle the edge case where `low` equals `"1"` separately:

```python
if low == "1":
    # All stepping numbers from 1 to high
    return count_stepping_numbers_up_to(high)
else:
    low_minus_one = str(int(low) - 1)
    return (count_stepping_numbers_up_to(high) - 
            count_stepping_numbers_up_to(low_minus_one)) % MOD
```

### 2. **Cache Pollution Between Different Number Calculations**

**The Pitfall:**
Using the same memoization cache for different target numbers (`high` and `low-1`) without clearing it:

```python
# WRONG: Using same cache for different numbers
@cache
def dfs(pos, pre, lead, limit):
    # ... implementation ...

# Computing for 'high'
num = high
a = dfs(0, -1, True, True)

# Computing for 'low-1' with polluted cache
num = str(int(low) - 1)
b = dfs(0, -1, True, True)  # Wrong results!
```

The cache contains results specific to the first number, which will give incorrect results for the second number since the `limit` parameter depends on the actual digits of the target number.

**Solution:**
Either clear the cache between calculations or use separate functions:

```python
# Option 1: Clear cache
dfs.cache_clear()

# Option 2: Use number as parameter
@cache
def dfs(num_str, pos, pre, lead, limit):
    up = int(num_str[pos]) if limit else 9
    # ... rest of implementation
```

### 3. **Forgetting the Modulo Operation**

**The Pitfall:**
Not applying modulo at intermediate steps, potentially causing integer overflow in languages with fixed integer sizes, or forgetting the final modulo:

```python
# WRONG: Only applying modulo at the end
count = 0
for digit in range(max_digit + 1):
    count += dp(...)  # Can overflow
return count % MOD

# WRONG: Negative results from subtraction
return (a - b) % MOD  # If b > a, this gives wrong answer in some languages
```

**Solution:**
Apply modulo consistently and handle potential negative results:

```python
# Correct approach
count = 0
for digit in range(max_digit + 1):
    count = (count + dp(...)) % MOD

# For subtraction, ensure positive result
return (a - b + MOD) % MOD
```

### 4. **Misunderstanding the Leading Zero Logic**

**The Pitfall:**
Incorrectly treating leading zeros as part of the stepping number validation:

```python
# WRONG: Checking stepping constraint for leading zeros
if digit == 0 and is_leading_zero:
    if previous_digit == -1 or abs(digit - previous_digit) == 1:
        count += dp(...)  # Wrong! Leading zeros shouldn't be validated
```

Leading zeros are not part of the actual number and shouldn't participate in the stepping number constraint check. The number "0123" is actually just "123".

**Solution:**
Handle leading zeros separately without checking the stepping constraint:

```python
if digit == 0 and is_leading_zero:
    # Continue with leading zeros, no stepping check needed
    count += dp(position + 1, previous_digit, True, ...)
elif previous_digit == -1 or abs(digit - previous_digit) == 1:
    # Only check stepping constraint for actual digits
    count += dp(position + 1, digit, False, ...)
```

