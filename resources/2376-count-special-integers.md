# 2376. Count Special Integers

> Source: https://algo.monster/liteproblems/2376

## Problem Statement

This problem asks us to count how many "special" positive integers exist within a given range from 1 to n.

A positive integer is considered **special** if all of its digits are distinct (no digit appears more than once).

For example:
- `135` is special because all digits (1, 3, 5) are different
- `121` is not special because the digit 1 appears twice
- `4567` is special because all digits (4, 5, 6, 7) are different

Given a positive integer `n`, we need to return the count of all special integers in the range `[1, n]` inclusive.

The solution uses a technique called Digit [Dynamic Programming](/problems/dynamic_programming_intro) (Digit DP) with state compression. The approach builds numbers digit by digit from left to right, keeping track of which digits have been used through a bitmask. The recursive function `dfs(i, mask, lead, limit)` explores all valid ways to construct special numbers:

- `i`: Current position (digit index) being processed
- `mask`: Bitmask tracking which digits (0-9) have already been used
- `lead`: Boolean indicating if we're still in leading zeros
- `limit`: Boolean indicating if we're bounded by the corresponding digit in `n`

## Intuition

The key insight is that we need to count numbers where digits don't repeat. Instead of checking every number from 1 to n individually (which would be inefficient for large n), we can build valid numbers digit by digit.

Think about how you'd manually count special numbers up to, say, 2357:
- First, you'd consider all 1-digit numbers (1-9): all are special
- Then 2-digit numbers (10-99): you need the two digits to be different
- Then 3-digit numbers (100-999): all three digits must be distinct
- Finally, numbers from 1000-2357: all four digits must be distinct

This naturally leads to a digit-by-digit construction approach. When building a number, at each position we need to:
1. Know which digits we've already used (to avoid repetition)
2. Know if we're still within the bound of n
3. Handle leading zeros correctly (e.g., 0012 is actually 12)

The bitmask is perfect for tracking used digits - if bit `j` is set in `mask`, digit `j` has been used. For example, if we've used digits 2 and 5, our mask would have bits 2 and 5 set: `mask = 0000100100` in binary.

The `limit` flag is crucial for staying within bounds. When `limit` is true, we can only choose digits up to the corresponding digit in n. For instance, if n = 2357 and we're building a number starting with 23__, the third digit can only be 0-5 (not 6-9) to stay ≤ 2357.

The `lead` flag handles the special case of leading zeros. Numbers like 0087 are actually just 87, so we shouldn't mark 0 as "used" when it's a leading zero. This allows us to use 0 as an actual digit later in the number.

By recursively exploring all valid digit choices at each position while maintaining these constraints, we efficiently count all special numbers without explicitly generating each one.

**Learn more about [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements Digit [Dynamic Programming](/problems/dynamic_programming_intro) with memoization using a recursive depth-first search function. Let's walk through the implementation step by step:

**1. String Conversion:**
First, we convert the number `n` to a string `s` to easily access individual digits. This allows us to process the number digit by digit from left to right.

**2. DFS Function Parameters:**
The recursive function `dfs(i, mask, lead, limit)` uses four parameters:
- `i`: Current digit position (0-indexed from the left)
- `mask`: Bitmask where bit `j` being 1 means digit `j` has been used
- `lead`: True if we're still in leading zeros
- `limit`: True if we must respect the upper bound from `n`

**3. Base Case:**
```python
if i >= len(s):
    return int(lead ^ 1)
```
When we've processed all positions, we return 1 if we've formed a valid number (not just leading zeros), otherwise 0. The expression `lead ^ 1` returns 0 when `lead` is True and 1 when `lead` is False.

**4. Determining Upper Bound:**
```python
up = int(s[i]) if limit else 9
```
If `limit` is True, we can only use digits up to the current digit in `n`. Otherwise, we can use any digit from 0 to 9.

**5. Trying Each Digit:**
For each position, we iterate through all possible digits from 0 to `up`:
```python
for j in range(up + 1):
    if mask >> j & 1:
        continue
```
We skip digit `j` if it's already been used (checked by seeing if bit `j` is set in `mask`).

**6. Handling Leading Zeros:**
```python
if lead and j == 0:
    ans += dfs(i + 1, mask, True, limit and j == up)
```
If we're still in leading zeros and choose 0, we don't mark 0 as used (mask remains unchanged) and continue with `lead = True`.

**7. Regular Digit Placement:**
```python
else:
    ans += dfs(i + 1, mask | 1 << j, False, limit and j == up)
```
For non-leading digits, we:
- Mark digit `j` as used by setting bit `j` in mask: `mask | 1 << j`
- Set `lead = False` since we've placed a non-zero digit
- Update `limit` based on whether we chose the maximum allowed digit

**8. Memoization:**
The `@cache` decorator automatically memoizes the results, preventing redundant calculations for the same state `(i, mask, lead, limit)`.

**9. Initial Call:**
```python
return dfs(0, 0, True, True)
```
We start from position 0, with no digits used (mask = 0), in leading zeros (lead = True), and bounded by n (limit = True).

The algorithm efficiently counts all special numbers by exploring only valid digit combinations while pruning invalid branches early. The time complexity is `O(m × 2^D × D)` where `m` is the number of digits in `n` and `D = 10` (the number of possible digits), with space complexity `O(m × 2^D)` for memoization.

## Example Walkthrough

Let's trace through the algorithm with `n = 25` to understand how it counts special integers.

**Setup:** Convert `n = 25` to string `s = "25"`. We need to count all special integers from 1 to 25.

**Initial call:** `dfs(0, 0, True, True)`
- Position 0 (first digit), no digits used (mask=0), in leading zeros, limited by n

**At position 0:**
- Upper bound is 2 (first digit of 25)
- Try digits 0, 1, 2:

  **j = 0:** Leading zero case
  - Call `dfs(1, 0, True, True)` - continue with leading zeros, mask unchanged

  **j = 1:** Place digit 1
  - Mark bit 1 in mask: `mask = 0000000010`
  - Call `dfs(1, 2, False, False)` - no longer leading, not at limit (1 < 2)

  **j = 2:** Place digit 2  
  - Mark bit 2 in mask: `mask = 0000000100`
  - Call `dfs(1, 4, False, True)` - no longer leading, still at limit (2 = 2)

**Following branch where first digit is 2 (limit maintained):**
`dfs(1, 4, False, True)` - Position 1, digit 2 used (mask=4), limit=True
- Upper bound is 5 (second digit of 25)
- Try digits 0, 1, 3, 4, 5 (skip 2 as it's used):

  **j = 0:** Valid, creates number 20
  - Call `dfs(2, 5, False, False)` → returns 1 (base case)

  **j = 1:** Valid, creates number 21
  - Call `dfs(2, 6, False, False)` → returns 1

  **j = 3:** Valid, creates number 23
  - Call `dfs(2, 12, False, False)` → returns 1

  **j = 4:** Valid, creates number 24
  - Call `dfs(2, 20, False, False)` → returns 1

  **j = 5:** Valid, creates number 25
  - Call `dfs(2, 36, False, True)` → returns 1

**Following branch where first digit is 1 (no limit):**
`dfs(1, 2, False, False)` - Position 1, digit 1 used (mask=2), limit=False
- Upper bound is 9 (no limit)
- Try digits 0, 2-9 (skip 1 as it's used):
  Each creates a valid 2-digit number (10, 12-19) → 9 special numbers

**Following leading zero branch:**
`dfs(1, 0, True, True)` - Position 1, no digits used, still leading
- This eventually generates single-digit numbers 1-9 → 9 special numbers

**Final count:**
- Single-digit: 1, 2, 3, 4, 5, 6, 7, 8, 9 → 9 numbers
- Two-digit starting with 1: 10, 12-19 → 9 numbers  
- Two-digit starting with 2: 20, 21, 23, 24, 25 → 5 numbers

**Total: 23 special integers from 1 to 25**

The algorithm efficiently explores all valid combinations while the bitmask prevents digit repetition, the limit flag ensures we don't exceed n, and the lead flag correctly handles numbers with fewer digits than n.

## Python Solution

```python
class Solution:
    def countSpecialNumbers(self, n: int) -> int:
        """
        Count numbers from 1 to n that have all distinct digits.
        Uses digit DP with states for position, used digits mask, leading zeros, and limit.
        """
        from functools import cache
      
        @cache
        def digit_dp(pos: int, used_mask: int, has_leading_zero: bool, is_limited: bool) -> int:
            """
            Dynamic programming function to count special numbers.
          
            Args:
                pos: Current position in the number (0-indexed from left)
                used_mask: Bitmask representing which digits have been used
                has_leading_zero: True if we haven't placed any non-zero digit yet
                is_limited: True if we must respect the upper bound at current position
          
            Returns:
                Count of valid numbers that can be formed from current state
            """
            # Base case: reached end of number
            if pos >= len(str_n):
                # Return 1 if we've placed at least one non-zero digit (valid number)
                # Return 0 if we only have leading zeros (not a valid number)
                return 0 if has_leading_zero else 1
          
            # Determine the maximum digit we can place at current position
            max_digit = int(str_n[pos]) if is_limited else 9
          
            result = 0
          
            # Try placing each possible digit at current position
            for digit in range(max_digit + 1):
                # Skip if this digit has already been used
                if (used_mask >> digit) & 1:
                    continue
              
                # Handle leading zeros specially
                if has_leading_zero and digit == 0:
                    # Continue with leading zero, don't mark 0 as used
                    result += digit_dp(
                        pos + 1, 
                        used_mask, 
                        True,  # Still have leading zeros
                        is_limited and (digit == max_digit)
                    )
                else:
                    # Place a non-zero digit or a zero after non-zero digits
                    result += digit_dp(
                        pos + 1,
                        used_mask | (1 << digit),  # Mark this digit as used
                        False,  # No longer have leading zeros
                        is_limited and (digit == max_digit)  # Update limit constraint
                    )
          
            return result
      
        # Convert number to string for digit-by-digit processing
        str_n = str(n)
      
        # Start DP from position 0, no digits used, with leading zeros, and limited by n
        return digit_dp(0, 0, True, True)
```

## Time and Space Complexity

**Time Complexity:** `O(D * 2^10 * D)` where `D` is the number of digits in `n`.

The time complexity is determined by the number of unique states in the memoized recursion:
- Position `i`: ranges from `0` to `D-1`, giving us `D` possible values
- Mask: represents which digits (0-9) have been used, resulting in `2^10 = 1024` possible states
- Lead flag: can be `True` or `False` (2 states)
- Limit flag: can be `True` or `False` (2 states)

Total unique states: `D * 2^10 * 2 * 2 = D * 2^12`

However, for each recursive call, we iterate through at most 10 digits (0-9), so the actual time complexity is `O(D * 2^10 * 10)` which simplifies to `O(D * 2^10 * D)` or approximately `O(D^2 * 1024)`.

**Space Complexity:** `O(D * 2^10)`

The space complexity consists of:
- Recursion call stack depth: `O(D)` as we recurse to a maximum depth equal to the number of digits
- Memoization cache: stores unique states which is `O(D * 2^10 * 2 * 2) = O(D * 2^12)`, but this simplifies to `O(D * 2^10)` as the constant factors are absorbed
- String conversion: `O(D)` to store the string representation of `n`

The dominant factor is the memoization cache, giving us a total space complexity of `O(D * 2^10)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Leading Zeros**

One of the most common mistakes is incorrectly handling leading zeros, particularly marking `0` as "used" when it appears as a leading zero. This would incorrectly prevent valid numbers like `102`, `203`, etc.

**Incorrect Implementation:**
```python
for digit in range(max_digit + 1):
    if (used_mask >> digit) & 1:
        continue
    # WRONG: Always marking digit as used, even for leading zeros
    result += digit_dp(
        pos + 1,
        used_mask | (1 << digit),  # This marks 0 as used even for leading zeros!
        digit != 0,
        is_limited and (digit == max_digit)
    )
```

**Correct Implementation:**
```python
if has_leading_zero and digit == 0:
    # Don't mark 0 as used when it's a leading zero
    result += digit_dp(pos + 1, used_mask, True, is_limited and (digit == max_digit))
else:
    # Only mark digit as used when it's not a leading zero
    result += digit_dp(pos + 1, used_mask | (1 << digit), False, is_limited and (digit == max_digit))
```

### 2. **Wrong Base Case Return Value**

Another frequent error is returning the wrong value in the base case, particularly not accounting for the case where we only have leading zeros (which represents the number 0 or an invalid number).

**Incorrect Implementation:**
```python
if pos >= len(str_n):
    return 1  # WRONG: This counts leading zeros as valid numbers
```

**Correct Implementation:**
```python
if pos >= len(str_n):
    return 0 if has_leading_zero else 1  # Only count if we've placed actual digits
```

### 3. **Forgetting to Update the Limit Flag**

The limit flag must be properly propagated. It should remain `True` only if we're still limited AND we choose the maximum allowed digit at the current position.

**Incorrect Implementation:**
```python
# WRONG: Always passing the same limit value
result += digit_dp(pos + 1, new_mask, new_lead, is_limited)

# OR WRONG: Only checking if digit equals max_digit
result += digit_dp(pos + 1, new_mask, new_lead, digit == max_digit)
```

**Correct Implementation:**
```python
# Must be limited AND choosing the max digit to remain limited
result += digit_dp(pos + 1, new_mask, new_lead, is_limited and (digit == max_digit))
```

### 4. **Off-by-One Error with Bit Operations**

When checking or setting bits in the mask, ensure you're using the correct bit position for each digit.

**Incorrect Implementation:**
```python
# WRONG: Using 1-indexed instead of 0-indexed
if (used_mask >> (digit + 1)) & 1:  # Off by one!
    continue
used_mask | (1 << (digit + 1))  # Off by one!
```

**Correct Implementation:**
```python
# Digit d corresponds to bit position d (0-indexed)
if (used_mask >> digit) & 1:
    continue
used_mask | (1 << digit)
```

### 5. **Not Using Memoization or Using It Incorrectly**

Without memoization, the solution will have exponential time complexity and timeout for large inputs.

**Incorrect Implementation:**
```python
def digit_dp(pos, used_mask, has_leading_zero, is_limited):
    # No memoization - will recalculate same states multiple times
    ...
```

**Correct Implementation:**
```python
from functools import cache

@cache  # Essential for performance
def digit_dp(pos, used_mask, has_leading_zero, is_limited):
    ...
```

**Note:** If manually implementing memoization with a dictionary, be careful not to cache states where `is_limited = True` incorrectly, as these states are specific to the input number and shouldn't be reused across different inputs.

