# 1067. Digit Count in Range

> Source: https://algo.monster/liteproblems/1067

## Problem Statement

You are given a single-digit integer `d` (from 0 to 9) and two integers `low` and `high`. Your task is to count how many times the digit `d` appears across all integers in the inclusive range `[low, high]`.

For example, if `d = 1`, `low = 1`, and `high = 13`, you need to count how many times the digit `1` appears in the numbers: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13.

- The digit `1` appears once in: 1, 10, 12, 13
- The digit `1` appears twice in: 11
- Total count = 1 + 1 + 2 + 1 + 1 = 6

The solution uses digit [dynamic programming](/problems/dynamic_programming_intro) (digit DP) to efficiently count occurrences. The main function `digitsCount` calculates the answer by finding the count of digit `d` from 0 to `high`, then subtracting the count from 0 to `low - 1`.

The helper function `f(n, d)` counts occurrences of digit `d` in all numbers from 0 to `n`. It uses a recursive `dfs` function with memoization that:
- `pos`: tracks the current digit position being processed
- `cnt`: counts occurrences of digit `d` found so far
- `lead`: indicates if we're still in leading zeros
- `limit`: indicates if we're bounded by the original number's digits

## Intuition

The key insight is that counting digit occurrences in a range `[low, high]` can be transformed into a simpler problem: counting occurrences from `[0, high]` minus occurrences from `[0, low-1]`. This is a common pattern in range-based problems.

Why does this work? Because the count in range `[low, high]` equals all occurrences up to `high` minus all occurrences before `low`.

The challenging part is efficiently counting digit occurrences from 0 to a given number `n`. A brute force approach would iterate through every number and count digits, but this becomes inefficient for large ranges.

This is where digit DP comes in. Instead of generating every number, we can think about building numbers digit by digit. When constructing numbers from 0 to `n`, we observe patterns:

1. **Position matters**: We process digits from most significant to least significant position
2. **Bounded choices**: At each position, our digit choices are limited by whether we're still bounded by `n`'s corresponding digit
3. **Leading zeros**: Numbers like 001, 002 are actually just 1, 2 - we need to handle leading zeros specially
4. **State tracking**: As we build the number, we keep track of how many times we've seen our target digit `d`

The recursive approach naturally fits this problem because at each digit position, we make a choice (which digit to place), and this choice affects our future choices. The memoization (`@cache`) ensures we don't recalculate the same states repeatedly.

For example, when counting digit `1` from 0 to 234:
- At the hundreds place, we can choose 0, 1, or 2
- If we choose 2, at the tens place we can only choose 0-3 (bounded by 234)
- If we choose 1, we've found one occurrence and continue building
- If we choose 0, we're still in leading zeros (building numbers less than 100)

This systematic exploration of all valid number constructions while counting target digit occurrences gives us an efficient `O(log n)` solution.

**Learn more about [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements digit [dynamic programming](/problems/dynamic_programming_intro) with memoization. Let's break down the implementation:

**Main Function**: `digitsCount(d, low, high)`
- Returns `f(high, d) - f(low - 1, d)`
- This uses the prefix sum principle to get the count in range `[low, high]`

**Helper Function**: `f(n, d)`
This function counts occurrences of digit `d` from 0 to `n`:

1. **Digit Extraction**: First, we extract digits of `n` into array `a`:
   ```python
   a = [0] * 11  # Array to store digits
   l = 0         # Length counter
   while n:
       l += 1
       a[l] = n % 10
       n //= 10
   ```
   For `n = 234`, we get `a = [0, 4, 3, 2, ...]` (stored in reverse, 1-indexed)

2. **Recursive DFS with Memoization**: The core logic is in the `dfs` function:
   ```python
   @cache
   def dfs(pos, cnt, lead, limit)
   ```

   **Parameters**:
   - `pos`: Current digit position (from most significant to least)
   - `cnt`: Count of target digit `d` found so far
   - `lead`: Boolean indicating if we're still in leading zeros
   - `limit`: Boolean indicating if we're bounded by the original number's digits

3. **Base Case**:
   ```python
   if pos <= 0:
       return cnt
   ```
   When all positions are processed, return the accumulated count.

4. **Upper Bound Determination**:
   ```python
   up = a[pos] if limit else 9
   ```
   - If `limit` is True, we can only use digits up to `a[pos]`
   - Otherwise, we can use any digit from 0 to 9

5. **Digit Choice Iteration**:
   ```python
   for i in range(up + 1):
       if i == 0 and lead:
           ans += dfs(pos - 1, cnt, lead, limit and i == up)
       else:
           ans += dfs(pos - 1, cnt + (i == d), False, limit and i == up)
   ```

   For each valid digit choice `i`:
   - **Leading Zero Case**: If `i == 0` and we're still in leading zeros (`lead == True`), we continue with leading zeros without counting this 0 as a digit
   - **Normal Case**: Otherwise, we:
     - Increment `cnt` if `i == d` (found our target digit)
     - Set `lead` to False (no longer in leading zeros)
     - Update `limit` based on whether we chose the maximum allowed digit

**Example Walkthrough** for counting digit `1` from 0 to 13:
- Extract digits: `a = [0, 3, 1]`, `l = 2`
- Start with `dfs(2, 0, True, True)`
- At position 2 (tens place):
  - Can choose 0 or 1 (limited by 1 in 13)
  - If choose 1: `dfs(1, 1, False, True)` (found one `1`, still limited)
  - If choose 0: `dfs(1, 0, True, False)` (leading zero, no longer limited)
- Continue recursively until all positions are processed

The `@cache` decorator memoizes results, preventing redundant calculations for repeated states, making the algorithm efficient with time complexity `O(log n × 10 × 2 × 2)` for each call to `f`.

## Example Walkthrough

Let's walk through counting digit `2` in the range `[15, 25]`.

**Step 1: Transform the Problem**
- We need `digitsCount(2, 15, 25)`
- This becomes: `f(25, 2) - f(14, 2)`
- Let's calculate each part

**Step 2: Calculate f(25, 2) - Count digit 2 from 0 to 25**

Extract digits of 25: `a = [0, 5, 2]`, length = 2

Start with `dfs(pos=2, cnt=0, lead=True, limit=True)`:

Position 2 (tens place):
- Upper bound is `a[2] = 2` (since limit=True)
- Try digit 0: Leading zero → `dfs(1, 0, True, False)`
  - This path will count 2's in numbers 0-9
  - At position 1, can use 0-9 (no limit)
  - When we choose 2: adds 1 to count
  - Result: 1 (just the number 2)

- Try digit 1: Not leading zero → `dfs(1, 0, False, False)`
  - This path counts 2's in numbers 10-19
  - At position 1, can use 0-9 (no limit)
  - When we choose 2: adds 1 to count
  - Result: 1 (just the number 12)

- Try digit 2: Found our target! → `dfs(1, 1, False, True)`
  - This path counts 2's in numbers 20-25
  - Already found one 2 (cnt=1)
  - At position 1, limited by `a[1] = 5`
  - Can choose 0,1,2,3,4,5
  - When we choose 2: adds another 1 (number 22 has two 2's)
  - Result: 1 + 6 = 7 (one 2 in tens place for all 20-25, plus one more in 22)

Total for f(25, 2) = 1 + 1 + 7 = 9

**Step 3: Calculate f(14, 2) - Count digit 2 from 0 to 14**

Extract digits of 14: `a = [0, 4, 1]`, length = 2

Start with `dfs(pos=2, cnt=0, lead=True, limit=True)`:

Position 2 (tens place):
- Upper bound is `a[2] = 1` (since limit=True)
- Try digit 0: Leading zero → `dfs(1, 0, True, False)`
  - Counts 2's in numbers 0-9
  - Result: 1 (just the number 2)

- Try digit 1: Not leading zero → `dfs(1, 0, False, True)`
  - Counts 2's in numbers 10-14
  - At position 1, limited by `a[1] = 4`
  - Can choose 0,1,2,3,4
  - When we choose 2: adds 1 to count
  - Result: 1 (just the number 12)

Total for f(14, 2) = 1 + 1 = 2

**Step 4: Final Answer**
- `digitsCount(2, 15, 25) = f(25, 2) - f(14, 2) = 9 - 2 = 7`

The digit 2 appears 7 times in the range [15, 25]:
- Once each in: 20, 21, 23, 24, 25
- Twice in: 22

This demonstrates how digit DP efficiently counts occurrences by building numbers digit by digit, tracking whether we're bounded by the original number and handling leading zeros appropriately.

## Python Solution

```python
from functools import cache
from typing import List

class Solution:
    def digitsCount(self, d: int, low: int, high: int) -> int:
        """
        Count how many times digit 'd' appears in all numbers from 'low' to 'high' (inclusive).
        Uses the principle: count(low, high) = count(0, high) - count(0, low-1)
        """
        return self.count_digit_occurrences(high, d) - self.count_digit_occurrences(low - 1, d)

    def count_digit_occurrences(self, upper_bound: int, target_digit: int) -> int:
        """
        Count occurrences of 'target_digit' in all numbers from 0 to 'upper_bound'.
        Uses digit DP technique to efficiently count without iterating through all numbers.
        """

        @cache
        def digit_dp(position: int, digit_count: int, has_leading_zeros: bool, is_bounded: bool) -> int:
            """
            Dynamic programming function to count digit occurrences.

            Args:
                position: Current digit position (1-indexed from right)
                digit_count: Count of target digit found so far
                has_leading_zeros: True if we haven't placed any non-zero digit yet
                is_bounded: True if we're still bounded by the original number's digits

            Returns:
                Total count of target digit occurrences for all valid numbers from this state
            """
            # Base case: finished processing all digits
            if position <= 0:
                return digit_count

            # Determine the maximum digit we can place at this position
            max_digit = digits_array[position] if is_bounded else 9

            total_count = 0

            # Try each possible digit at current position
            for current_digit in range(max_digit + 1):
                if current_digit == 0 and has_leading_zeros:
                    # Still in leading zeros, don't count this zero
                    total_count += digit_dp(
                        position - 1,
                        digit_count,
                        True,  # Still has leading zeros
                        is_bounded and (current_digit == max_digit)
                    )
                else:
                    # Either non-zero digit or zero after first non-zero digit
                    new_count = digit_count + (1 if current_digit == target_digit else 0)
                    total_count += digit_dp(
                        position - 1,
                        new_count,
                        False,  # No more leading zeros
                        is_bounded and (current_digit == max_digit)
                    )

            return total_count

        # Extract digits from the number (stored in reverse order for easier access)
        digits_array: List[int] = [0] * 11  # Support up to 10-digit numbers
        num_digits = 0
        temp_num = upper_bound

        while temp_num > 0:
            num_digits += 1
            digits_array[num_digits] = temp_num % 10
            temp_num //= 10

        # Handle edge case where upper_bound is 0
        if upper_bound == 0:
            return 1 if target_digit == 0 else 0

        # Start the digit DP from the most significant digit
        return digit_dp(num_digits, 0, True, True)
```

## Time and Space Complexity

**Time Complexity:** `O(log(high) * 10 * log(high))` which simplifies to `O(log²(high))`

The time complexity is determined by the digit DP (dynamic programming) approach:
- The number of digits in the maximum number is `O(log(high))`
- For each position `pos` from 1 to the number of digits, we iterate through at most 10 possible digits (0-9)
- Each state `(pos, cnt, lead, limit)` is computed at most once due to memoization
- The number of unique states is bounded by:
  - `pos`: `O(log(high))` positions
  - `cnt`: `O(log(high))` possible count values (at most the number of digits)
  - `lead`: 2 possible values (True/False)
  - `limit`: 2 possible values (True/False)
- Total states: `O(log(high) * log(high) * 2 * 2)` = `O(log²(high))`
- Each state does `O(10)` work iterating through digits
- Overall: `O(log²(high))` considering the constant factor of 10

**Space Complexity:** `O(log²(high))`

The space complexity consists of:
- Recursion stack depth: `O(log(high))` - maximum depth equals the number of digits
- Memoization cache: `O(log²(high))` - storing all possible states as analyzed above
- Array `a` for storing digits: `O(11)` = `O(1)` constant space
- The dominant factor is the memoization cache, giving us `O(log²(high))` space complexity

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Leading Zeros When Counting Zero**

**The Pitfall**: When counting occurrences of digit `0`, leading zeros should NOT be counted, but zeros that appear after the first non-zero digit SHOULD be counted. Many implementations incorrectly handle this distinction.

**Example Problem**: Count occurrences of `0` from 1 to 105
- Number `105` has one `0` (the middle digit)
- Number `10` has one `0`
- Numbers like `001`, `002` don't exist (leading zeros don't form valid numbers)

**Incorrect Implementation**:
```python
# WRONG: This might count leading zeros
if current_digit == 0:
    total_count += digit_dp(position - 1, digit_count + 1, ...)  # Always counting zeros
```

**Correct Implementation**:
```python
if current_digit == 0 and has_leading_zeros:
    # Leading zero - don't count it
    total_count += digit_dp(position - 1, digit_count, True, ...)
else:
    # Either non-zero OR zero after first non-zero digit - count normally
    new_count = digit_count + (1 if current_digit == target_digit else 0)
    total_count += digit_dp(position - 1, new_count, False, ...)
```

### 2. **Edge Case: When Upper Bound is 0**

**The Pitfall**: The digit extraction loop `while temp_num > 0` won't execute when the upper bound is 0, leaving `num_digits = 0`. This causes the DP to return 0 even when counting occurrences of digit `0` from 0 to 0 (which should return 1).

**Incorrect Handling**:
```python
def count_digit_occurrences(self, upper_bound: int, target_digit: int) -> int:
    # ... digit extraction ...
    while temp_num > 0:
        num_digits += 1
        digits_array[num_digits] = temp_num % 10
        temp_num //= 10

    # If upper_bound is 0, num_digits stays 0, causing incorrect result
    return digit_dp(num_digits, 0, True, True)
```

**Correct Solution**:
```python
def count_digit_occurrences(self, upper_bound: int, target_digit: int) -> int:
    # ... digit extraction ...

    # Handle edge case where upper_bound is 0
    if upper_bound == 0:
        return 1 if target_digit == 0 else 0

    return digit_dp(num_digits, 0, True, True)
```

### 3. **Off-by-One Error in Range Calculation**

**The Pitfall**: Forgetting to subtract 1 when calculating the lower bound, resulting in counting the range `[0, low]` instead of `[0, low-1]`.

**Incorrect**:
```python
def digitsCount(self, d: int, low: int, high: int) -> int:
    return self.f(high, d) - self.f(low, d)  # WRONG: excludes 'low' from the range
```

**Correct**:
```python
def digitsCount(self, d: int, low: int, high: int) -> int:
    return self.f(high, d) - self.f(low - 1, d)  # Correct: includes 'low' in the range
```

### 4. **Incorrect Limit Flag Propagation**

**The Pitfall**: Not properly updating the `limit` flag when recursing. The limit should only remain `True` if we choose the maximum allowed digit at the current position.

**Incorrect**:
```python
# WRONG: Always passing the same limit flag
total_count += digit_dp(position - 1, new_count, False, is_bounded)
```

**Correct**:
```python
# Correct: Update limit based on whether we chose the maximum allowed digit
total_count += digit_dp(
    position - 1,
    new_count,
    False,
    is_bounded and (current_digit == max_digit)
)
```

### 5. **Array Size and Indexing Issues**

**The Pitfall**: Using 0-based indexing when the algorithm expects 1-based indexing for digit positions, or allocating insufficient array size for large numbers.

**Prevention**:
- Always allocate enough space: `digits_array = [0] * 11` for 10-digit numbers
- Use 1-based indexing consistently: digits are stored from index 1 to `num_digits`
- The digit at position `pos` represents the `pos`-th digit from the right (1-indexed)

