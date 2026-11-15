# 1012. Numbers With Repeated Digits

> Source: https://algo.monster/liteproblems/1012

## Problem Statement

Given an integer `n`, you need to count how many positive integers from 1 to `n` contain **at least one repeated digit**.

For example:
- The number `11` has a repeated digit (the digit `1` appears twice)
- The number `121` has a repeated digit (the digit `1` appears twice)  
- The number `123` has no repeated digits (all digits are unique)

The solution uses a digit [dynamic programming](/problems/dynamic_programming_intro) approach with state compression. Instead of directly counting numbers with repeated digits, it calculates numbers with **no repeated digits** and subtracts from the total.

The key idea is to use a bitmask to track which digits (0-9) have already appeared in the number being formed. The `dfs` function explores all valid digit combinations while:
- Tracking used digits via the `mask` parameter (bit `j` is set if digit `j` has been used)
- Handling leading zeros with the `lead` parameter
- Respecting the upper bound `n` with the `limit` parameter

For each position, the algorithm tries placing each valid digit (0-9), skipping those already used (checked via `mask >> j & 1`). The final answer is `n - count_of_numbers_with_unique_digits`.

The time complexity is `O(log n × 2^10 × 10)` where `log n` represents the number of digits, `2^10` represents possible digit masks, and `10` represents digit choices.

## Intuition

When faced with counting numbers with "at least one" repeated digit, a direct approach becomes complicated - we'd need to track various cases like exactly one pair of repeated digits, two pairs, three of the same digit, etc. This quickly becomes unwieldy.

The key insight is to use **complementary counting**: instead of counting what we want directly, count what we don't want and subtract. Numbers with "at least one repeated digit" is the complement of numbers with "all unique digits". So the answer becomes `n - count_of_unique_digit_numbers`.

Counting numbers with all unique digits is much cleaner - we just need to ensure each digit we place hasn't been used before. This naturally leads to using a **bitmask** to track which of the 10 digits (0-9) have been used. If digit `j` has been used, we set bit `j` in our mask.

Why digit DP? When building numbers digit by digit from left to right, we need to:
1. Respect the upper bound `n` (we can't exceed it)
2. Handle leading zeros properly (they don't count as "used" digits)
3. Track our state efficiently across recursive calls

The recursive structure emerges naturally: at each position, try all valid digits (0-9), mark them as used in the mask, and continue building the rest of the number. The `limit` flag ensures we don't exceed `n` - if we're at the limit and place the maximum allowed digit, the next position remains limited. The `lead` flag handles the special case where we haven't placed any non-zero digit yet.

The memoization with `@cache` is crucial because many different paths lead to the same state (same position, same used digits, same flags), allowing us to avoid redundant calculations.

**Learn more about [Math](/problems/math-basics) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements digit [dynamic programming](/problems/dynamic_programming_intro) with state compression. Let's break down the implementation step by step:

**1. State Definition**

We define a recursive function `dfs(i, mask, lead, limit)` with four parameters:
- `i`: Current digit position (0-indexed from left)
- `mask`: Binary representation of used digits (bit `j` is 1 if digit `j` has been used)
- `lead`: Boolean indicating if we're still in leading zeros
- `limit`: Boolean indicating if we're bounded by the original number `n`

**2. Base Case**

```python
if i >= len(s):
    return lead ^ 1
```
When we've processed all digits, we return 1 if we've formed a valid number (not just leading zeros), otherwise 0. The expression `lead ^ 1` returns 0 if `lead` is True (all zeros), and 1 if `lead` is False (valid number).

**3. Calculating Upper Bound**

```python
up = int(s[i]) if limit else 9
```
If we're limited by the original number, we can only place digits up to `s[i]`. Otherwise, we can use any digit from 0 to 9.

**4. Digit Enumeration**

For each position, we try all valid digits from 0 to `up`:

```python
for j in range(up + 1):
    if lead and j == 0:
        ans += dfs(i + 1, mask, True, False)
    elif mask >> j & 1 ^ 1:
        ans += dfs(i + 1, mask | 1 << j, False, limit and j == up)
```

- **Leading zeros case**: If we're still in leading zeros (`lead = True`) and place another 0, we continue with leading zeros without marking 0 as used. The limit becomes False since leading zeros are always less than any positive number.

- **Valid digit case**: The condition `mask >> j & 1 ^ 1` checks if digit `j` hasn't been used yet (equivalent to checking if bit `j` in mask is 0). If the digit is available:
  - We set bit `j` in the mask: `mask | 1 << j`
  - We mark `lead` as False since we've placed a non-zero digit
  - We update `limit`: it remains True only if we were limited AND placed the maximum digit

**5. Memoization**

The `@cache` decorator automatically memoizes the results, avoiding recalculation for identical states.

**6. Final Calculation**

```python
s = str(n)
return n - dfs(0, 0, True, True)
```

We convert `n` to string for easy digit access, then subtract the count of numbers with unique digits from `n` to get numbers with at least one repeated digit.

The initial call uses:
- `i = 0`: Start from the leftmost digit
- `mask = 0`: No digits used yet
- `lead = True`: Initially in leading zeros
- `limit = True`: Initially bounded by `n`

## Example Walkthrough

Let's trace through the algorithm with `n = 25` to understand how it counts numbers with unique digits.

**Goal**: Count numbers from 1 to 25 with at least one repeated digit.
**Strategy**: Count numbers with ALL unique digits, then compute `25 - count`.

Let's trace a few paths through the DFS to see how it builds valid numbers:

**Initial Call**: `dfs(0, 0, True, True)` with string "25"
- Position 0, no digits used (mask=0000000000), in leading zeros, limited by "25"

**Building "12"**:
1. At position 0: Can place 0, 1, or 2 (limited by '2')
   - Choose 1: `dfs(1, 0000000010, False, False)`
   - Mask becomes 0000000010 (bit 1 set)
   - No longer in leading zeros, no longer limited (1 < 2)

2. At position 1: Can place 0-9 except 1 (already used)
   - Choose 2: `dfs(2, 0000000110, False, False)`
   - Mask becomes 0000000110 (bits 1 and 2 set)
 
3. At position 2: Reached end, return 1 (valid number formed)

**Building "22"** (will be rejected):
1. At position 0: Choose 2: `dfs(1, 0000000100, False, True)`
   - Still limited because we placed the maximum digit '2'

2. At position 1: Can only place 0-2 (limited by '2' in "25")
   - Try to place 2: Check `mask >> 2 & 1` = 1 (digit 2 already used)
   - This path is blocked! Cannot reuse digit 2

**Building "7"** (single digit):
1. At position 0: Choose 0 (leading zero): `dfs(1, 0, True, False)`
   - Remains in leading zeros, mask unchanged

2. At position 1: Choose 7: `dfs(2, 0010000000, False, False)`
   - Exits leading zeros, marks digit 7 as used

3. At position 2: Reached end, return 1

**Complete Count**:
The algorithm systematically explores all paths, counting valid numbers:
- Single digit numbers with unique digits: 1,2,3,4,5,6,7,8,9 (9 numbers)
- Two digit numbers 10-25 with unique digits: 10,12,13,14,15,16,17,18,19,20,21,23,24,25 (14 numbers)
- Total with unique digits: 23

**Final Answer**: `25 - 23 = 2`

The two numbers with repeated digits are: 11 and 22


Let's work through `n = 25` to see how the algorithm counts numbers with at least one repeated digit.

**Setup**: Convert 25 to string "25" and call `dfs(0, 0, True, True)`

The algorithm will count numbers from 1-25 that have ALL unique digits, then subtract from 25.

**Key paths through the recursion:**

**Path 1: Building number "12"**
- Position 0: Place digit 1
  - `mask = 0000000010` (bit 1 set), `lead = False`, `limit = False` (1 < 2)
- Position 1: Place digit 2 
  - Check: Is digit 2 used? `(mask >> 2) & 1 = 0` ✓ Available
  - `mask = 0000000110` (bits 1,2 set)
- Position 2: End reached, return 1 (valid number with unique digits)

**Path 2: Attempting "22"**
- Position 0: Place digit 2
  - `mask = 0000000100` (bit 2 set), `lead = False`, `limit = True` (2 = 2)
- Position 1: Try digit 2
  - Check: Is digit 2 used? `(mask >> 2) & 1 = 1` ✗ Already used!
  - This branch returns 0 (cannot form valid number)

**Path 3: Building "7" (single digit)**
- Position 0: Place digit 0 (leading zero)
  - `mask = 0` (unchanged), `lead = True`, `limit = False`
- Position 1: Place digit 7
  - `mask = 0010000000` (bit 7 set), `lead = False`
- Position 2: End reached, return 1

**Results Summary:**
- Numbers 1-25 with unique digits: 1,2,3,4,5,6,7,8,9,10,12,13,14,15,16,17,18,19,20,21,23,24,25
- Count = 23
- Numbers with repeated digits = 25 - 23 = 2 (which are 11 and 22)

The algorithm efficiently explores all valid combinations using memoization to avoid recalculating identical states.

## Python Solution

```python
class Solution:
    def numDupDigitsAtMostN(self, n: int) -> int:
        """
        Count how many positive integers <= n have at least one repeated digit.
      
        Strategy: Count numbers without repeated digits, then subtract from n.
        Uses digit DP with states tracking position, used digits, leading zeros, and upper bound.
        """
        from functools import cache
      
        # Convert n to string for digit-by-digit processing
        digits_str = str(n)
      
        @cache
        def count_unique_digit_numbers(
            position: int, 
            used_digits_mask: int, 
            has_leading_zeros: bool, 
            is_bounded: bool
        ) -> int:
            """
            Count numbers with unique digits using digit DP.
          
            Args:
                position: Current digit position being processed
                used_digits_mask: Bitmask representing which digits (0-9) have been used
                has_leading_zeros: True if we haven't placed a non-zero digit yet
                is_bounded: True if we must respect the upper bound (digits of n)
          
            Returns:
                Count of valid numbers from this state
            """
            # Base case: processed all digits
            if position >= len(digits_str):
                # Return 1 if we've formed a valid number (not just leading zeros)
                return 0 if has_leading_zeros else 1
          
            # Determine the maximum digit we can place at this position
            max_digit = int(digits_str[position]) if is_bounded else 9
          
            total_count = 0
          
            # Try each possible digit at current position
            for digit in range(max_digit + 1):
                # Handle leading zeros case
                if has_leading_zeros and digit == 0:
                    # Continue with leading zeros, no longer bounded
                    total_count += count_unique_digit_numbers(
                        position + 1, 
                        used_digits_mask, 
                        True, 
                        False
                    )
                # Check if digit hasn't been used yet (bit is 0 in mask)
                elif (used_digits_mask >> digit) & 1 == 0:
                    # Mark digit as used and continue
                    new_mask = used_digits_mask | (1 << digit)
                    # Stay bounded only if we're currently bounded AND using max digit
                    new_bounded = is_bounded and (digit == max_digit)
                  
                    total_count += count_unique_digit_numbers(
                        position + 1, 
                        new_mask, 
                        False, 
                        new_bounded
                    )
          
            return total_count
      
        # Count numbers without repeated digits from 1 to n
        unique_digit_count = count_unique_digit_numbers(0, 0, True, True)
      
        # Numbers with repeated digits = total numbers - numbers without repeated digits
        return n - unique_digit_count
```

## Time and Space Complexity

**Time Complexity:** `O(D * 2^10 * D)` where `D` is the number of digits in `n`.

The time complexity can be analyzed through the memoization states and transitions:
- Parameter `i` ranges from `0` to `D-1` (number of digits)
- Parameter `mask` can have at most `2^10` different values (representing which digits 0-9 have been used)
- Parameter `lead` has 2 possible values (True/False)
- Parameter `limit` has 2 possible values (True/False)

This gives us at most `D * 2^10 * 2 * 2 = O(D * 2^10)` unique states. For each state, we iterate through at most 10 digits (0-9), making the total time complexity `O(D * 2^10 * 10)`. Since `2^10 = 1024` and we multiply by 10, this simplifies to `O(D * 10240)` or `O(D * 2^10 * D)` in terms of digit operations.

**Space Complexity:** `O(D * 2^10)`

The space complexity comes from:
- The recursion stack depth which is at most `D` (the number of digits)
- The memoization cache storing up to `D * 2^10 * 2 * 2` states
- The string representation of `n` which takes `O(D)` space

The dominant factor is the memoization cache, giving us `O(D * 2^10)` space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Leading Zeros in the Bitmask**

**The Pitfall:** A common mistake is to mark the digit `0` as "used" when it appears as a leading zero. This would incorrectly prevent valid numbers like `101`, `202`, etc., from being counted.

**Why it happens:** When we encounter leading zeros, we might intuitively think to mark `0` as used in the bitmask. However, leading zeros don't actually represent the digit `0` in the number - they're just placeholders.

**Incorrect approach:**
```python
for digit in range(max_digit + 1):
    if has_leading_zeros and digit == 0:
        # WRONG: marking 0 as used for leading zeros
        total_count += count_unique_digit_numbers(
            position + 1, 
            used_digits_mask | (1 << 0),  # ❌ Incorrectly setting bit 0
            True, 
            False
        )
```

**Correct approach:**
```python
for digit in range(max_digit + 1):
    if has_leading_zeros and digit == 0:
        # CORRECT: Don't mark 0 as used for leading zeros
        total_count += count_unique_digit_numbers(
            position + 1, 
            used_digits_mask,  # ✓ Keep mask unchanged
            True, 
            False
        )
```

### 2. **Misunderstanding the Limit Flag Update**

**The Pitfall:** Incorrectly updating the `is_bounded` flag, particularly forgetting that once we place a digit smaller than the maximum allowed, all subsequent positions become unbounded.

**Common mistakes:**
- Setting `is_bounded = True` for all recursive calls when currently bounded
- Not considering that leading zeros automatically make the number unbounded

**Incorrect approach:**
```python
# WRONG: Always passing the current bounded state
new_bounded = is_bounded  # ❌ Doesn't account for digit choice
total_count += count_unique_digit_numbers(
    position + 1, new_mask, False, new_bounded
)
```

**Correct approach:**
```python
# CORRECT: Only stay bounded if we use the maximum digit
new_bounded = is_bounded and (digit == max_digit)  # ✓
total_count += count_unique_digit_numbers(
    position + 1, new_mask, False, new_bounded
)
```

### 3. **Off-by-One Error in Final Calculation**

**The Pitfall:** The problem asks for numbers from `1` to `n` with repeated digits, but the digit DP counts all valid numbers including `0`. This can lead to confusion about whether to include or exclude zero.

**Why it matters:** The function `count_unique_digit_numbers` doesn't generate the number `0` when starting with `has_leading_zeros = True` because the base case returns `0` for pure leading zeros. This is correct behavior, but it's easy to misunderstand and try to "fix" it.

**Incorrect attempts to handle this:**
```python
# WRONG: Trying to adjust for zero
return n - unique_digit_count + 1  # ❌ Overcounting

# WRONG: Starting count from 1
return (n - 1) - unique_digit_count  # ❌ Undercounting
```

**Correct approach:**
```python
# CORRECT: Direct subtraction works because:
# - n represents count of numbers from 1 to n
# - unique_digit_count represents numbers from 1 to n with unique digits
return n - unique_digit_count  # ✓
```

### 4. **Bit Manipulation Confusion**

**The Pitfall:** Misunderstanding the bit operations for checking and setting digits in the mask.

**Common mistakes:**
```python
# WRONG: Incorrect check for unused digit
if used_digits_mask & (1 << digit) == 0:  # Works but less clear
  
# WRONG: Using wrong operator precedence
if used_digits_mask >> digit & 1 ^ 1:  # Can be confusing without parentheses

# WRONG: Checking for used digit instead of unused
if (used_digits_mask >> digit) & 1 == 1:  # ❌ Logic reversed
```

**Correct and clear approach:**
```python
# CORRECT: Check if digit is NOT used (bit is 0)
if (used_digits_mask >> digit) & 1 == 0:  # ✓ Clear and correct
    # Mark digit as used
    new_mask = used_digits_mask | (1 << digit)
```

These pitfalls often lead to subtle bugs that can be hard to debug because they only affect certain edge cases or ranges of numbers.

