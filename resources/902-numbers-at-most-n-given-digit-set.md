# 902. Numbers At Most N Given Digit Set

> Source: https://algo.monster/liteproblems/902

## Problem Statement

You are given an array of `digits` sorted in non-decreasing order. You can use each digit from this array as many times as you want to form positive integers.

For example, if `digits = ['1','3','5']`, you can create numbers like:
- `'13'` (using digit 1 and digit 3)
- `'551'` (using digit 5 twice and digit 1 once)
- `'1351315'` (using digits 1, 3, and 5 multiple times)

Your task is to count how many positive integers you can generate using only the given digits that are less than or equal to a given integer `n`.

Key points to understand:
- You can only use the digits provided in the `digits` array
- Each digit can be used multiple times (unlimited usage)
- The numbers you form must be positive integers (no leading zeros, must be > 0)
- The numbers must be ≤ `n`

For instance, if `digits = ['1', '3', '5']` and `n = 100`, you need to count all valid numbers like 1, 3, 5, 11, 13, 15, 31, 33, 35, 51, 53, 55 that can be formed using only digits 1, 3, and 5, and are less than or equal to 100.

## Intuition

When we need to count numbers formed from specific digits that are less than or equal to `n`, we can think about this digit by digit. The key insight is that we're essentially building numbers from left to right, and at each position, we need to make decisions about which digit to place.

Consider `n = 523` and `digits = ['1', '3', '5']`. We can form numbers with:
- 1 digit: 1, 3, 5 (all valid since they're < 523)
- 2 digits: 11, 13, 15, 31, 33, 35, 51, 53, 55 (all valid)
- 3 digits: We need to be careful here - not all combinations will be ≤ 523

This naturally leads us to think about [Dynamic Programming](/problems/dynamic_programming_intro) on digits. The core idea is to traverse the number `n` digit by digit from left to right, and at each position, decide which digits from our available set we can place.

At each position, we face two scenarios:
1. **We're still bounded by `n`**: If we've been matching `n` exactly so far (like placing '5' at the first position when `n = 523`), then at the current position, we can only place digits that don't exceed the corresponding digit in `n`.
2. **We're already smaller than `n`**: If we've already placed a smaller digit earlier (like placing '3' at the first position when `n = 523`), then we're free to place any available digit at subsequent positions.

We also need to handle leading zeros carefully. Since we want positive integers, we can't have numbers that start with 0. However, when we're building numbers digit by digit, we need a way to represent "we haven't started the actual number yet" - this is where the `lead` parameter comes in.

The recursive structure emerges naturally:
- Start from the leftmost position
- For each position, try all valid digits
- Keep track of whether we're still bounded by `n` (the `limit` flag)
- Keep track of whether we're still in the leading zero phase (the `lead` flag)
- When we reach the end, we've successfully formed a valid number (unless it's all leading zeros)

This digit DP approach is powerful because it systematically explores all possibilities while avoiding redundant calculations through memoization. The beauty is that we're not generating all possible numbers explicitly - we're counting them efficiently by recognizing that many subproblems repeat.

**Learn more about [Math](/problems/math-basics), [Binary Search](/problems/binary_search_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a Digit [Dynamic Programming](/problems/dynamic_programming_intro) approach with memoization. Let's break down the implementation step by step:

**1. Initial Setup:**
```python
s = str(n)
nums = {int(x) for x in digits}
```
- Convert the integer `n` to a string `s` to easily access individual digits
- Convert the `digits` array to a set of integers `nums` for O(1) lookup

**2. The Recursive Function:**
The core of the solution is the `dfs` function with three parameters:
- `i`: Current position in the string `s` (0-indexed)
- `lead`: Boolean indicating if we're still in the leading zeros phase (haven't placed any non-zero digit yet)
- `limit`: Boolean indicating if we're still bounded by the upper limit `n`

**3. Base Case:**
```python
if i >= len(s):
    return lead ^ 1
```
When we've processed all positions:
- If `lead` is True (all leading zeros), return 0 (not a valid positive integer)
- If `lead` is False (we've placed at least one non-zero digit), return 1 (found one valid number)
- The XOR operation `lead ^ 1` elegantly returns 0 when `lead=True` and 1 when `lead=False`

**4. Calculating the Upper Bound:**
```python
up = int(s[i]) if limit else 9
```
- If `limit` is True, we can only place digits up to `s[i]` (the digit at position `i` in `n`)
- If `limit` is False, we can place any digit from 0 to 9

**5. Trying All Valid Digits:**
```python
for j in range(up + 1):
    if j == 0 and lead:
        ans += dfs(i + 1, 1, limit and j == up)
    elif j in nums:
        ans += dfs(i + 1, 0, limit and j == up)
```
For each possible digit `j` from 0 to `up`:
- **Case 1: Leading Zero** (`j == 0 and lead`):
  - We're still in the leading zeros phase
  - Move to next position with `lead=True`
  - Update `limit`: remains True only if we're at the limit AND placed the maximum allowed digit

- **Case 2: Valid Digit** (`j in nums`):
  - We can only place digits that exist in our available set
  - Move to next position with `lead=False` (we've placed a non-zero digit)
  - Update `limit`: remains True only if we're at the limit AND placed the maximum allowed digit

**6. Memoization:**
The `@cache` decorator automatically memoizes the results, preventing redundant calculations for the same `(i, lead, limit)` state.

**7. Final Answer:**
```python
return dfs(0, 1, True)
```
Start the recursion from position 0, with leading zeros allowed initially (`lead=True`), and bounded by the limit (`limit=True`).

**Example Walkthrough:**
For `n = 100` and `digits = ['1', '3']`:
- Position 0: Can place 0 (leading zero) or 1 (valid digit, at limit)
- If we place 1 at position 0:
  - Position 1: Can only place 0 (since `limit=True` and `s[1]='0'`)
  - But 0 is not in our digits, so this path ends
- If we place 0 at position 0 (leading zero):
  - Continue with `lead=True` to position 1
  - Can try placing 1 or 3 (both create 2-digit numbers < 100)

The algorithm efficiently counts all valid combinations without explicitly generating each number, achieving a time complexity of `O(10 × log n) = O(log n)`, since each of the `O(log n)` states loops over at most 10 candidate digits.

## Example Walkthrough

Let's walk through a concrete example with `n = 35` and `digits = ['1', '5']`.

We need to count all positive integers ≤ 35 that can be formed using only digits 1 and 5.

**Setup:**
- Convert `n = 35` to string `s = "35"`
- Convert digits to set: `nums = {1, 5}`
- Start with `dfs(0, lead=True, limit=True)`

**Step-by-step recursion:**

**Position 0 (tens place):**
- Current state: `i=0, lead=True, limit=True`
- Upper bound: `up = int(s[0]) = 3` (since `limit=True`)
- Try digits 0 to 3:
  - `j=0`: Leading zero, continue with `dfs(1, True, False)`
  - `j=1`: Valid digit (1 ∈ nums), continue with `dfs(1, False, False)`
  - `j=2`: Not in nums, skip
  - `j=3`: Not in nums, skip

**Branch 1: Placed nothing yet (leading zero at position 0)**
- Current state: `i=1, lead=True, limit=False`
- Upper bound: `up = 9` (no limit anymore)
- Try digits 0 to 9:
  - `j=0`: Still leading zero, would reach base case returning 0
  - `j=1`: Valid digit, forms number "1", reaches base case returning 1
  - `j=2-4`: Not in nums, skip
  - `j=5`: Valid digit, forms number "5", reaches base case returning 1
  - `j=6-9`: Not in nums, skip
- This branch contributes: 2 numbers (1 and 5)

**Branch 2: Placed '1' at position 0**
- Current state: `i=1, lead=False, limit=False`
- Upper bound: `up = 9` (not at limit since we placed 1 < 3)
- Try digits 0 to 9:
  - `j=0`: Not in nums, skip
  - `j=1`: Valid digit, forms "11", reaches base case returning 1
  - `j=2-4`: Not in nums, skip
  - `j=5`: Valid digit, forms "15", reaches base case returning 1
  - `j=6-9`: Not in nums, skip
- This branch contributes: 2 numbers (11 and 15)

**Total count:** 2 + 2 = 4

The valid numbers are: **1, 5, 11, 15**

**Key observations from this walkthrough:**
1. The `lead` parameter prevents counting empty numbers or those with only zeros
2. The `limit` parameter becomes False as soon as we place a digit smaller than the corresponding digit in `n`, giving us freedom in subsequent positions
3. We only count combinations using digits from our available set
4. The recursion efficiently explores all valid paths without generating actual numbers

## Python Solution

```python
class Solution:
    def atMostNGivenDigitSet(self, digits: List[str], n: int) -> int:
        from functools import cache

        # Convert n to string for digit-by-digit processing
        n_str = str(n)
        # Convert digit strings to a set of integers for O(1) lookup
        available_digits = {int(digit) for digit in digits}

        @cache
        def count_valid_numbers(position: int, is_leading_zero: bool, is_bounded: bool) -> int:
            """
            Count valid numbers using digit DP.

            Args:
                position: Current position in the number (0-indexed from left)
                is_leading_zero: True if we haven't placed any non-zero digit yet
                is_bounded: True if we must stay within the upper bound of n

            Returns:
                Count of valid numbers from this state
            """
            # Base case: reached the end of the number
            if position >= len(n_str):
                # Return 1 if we've placed at least one non-zero digit (valid number formed)
                # Return 0 if we only have leading zeros (no number formed)
                return 0 if is_leading_zero else 1

            # Determine the maximum digit we can place at this position
            max_digit = int(n_str[position]) if is_bounded else 9

            total_count = 0

            # Try each possible digit from 0 to max_digit
            for digit in range(max_digit + 1):
                if digit == 0 and is_leading_zero:
                    # Skip this position (extend leading zeros)
                    # Stay bounded if we were bounded and placed the max digit
                    total_count += count_valid_numbers(
                        position + 1,
                        True,
                        is_bounded and (digit == max_digit)
                    )
                elif digit in available_digits:
                    # Place this digit (it's in our allowed set)
                    # No longer have leading zeros after placing a non-zero digit
                    # Stay bounded if we were bounded and placed the max digit
                    total_count += count_valid_numbers(
                        position + 1,
                        False,
                        is_bounded and (digit == max_digit)
                    )

            return total_count

        # Start from position 0, with leading zeros, and bounded by n
        return count_valid_numbers(0, True, True)
```

## Time and Space Complexity

**Time Complexity:** `O(10 * log(n))`, which is `O(log(n))`

The algorithm uses digit dynamic programming (digit DP) to count valid numbers. The recursive function `dfs` processes each digit position of the number `n`:
- The string representation of `n` has `log₁₀(n)` digits, which determines the depth of recursion
- At each position, the loop `for digit in range(max_digit + 1)` tries up to 10 candidate digits, and each one needs only an `O(1)` set lookup, so each recursive call does `O(10)` work outside its recursive calls
- The `@cache` decorator memoizes results, ensuring each unique state `(i, lead, limit)` is computed only once
- Total unique states: `O(log(n) * 2 * 2)` = `O(log(n))`
- Total time: `O(log(n) * 2 * 2 * 10)` = `O(log(n))`

**Space Complexity:** `O(log(n))`

The space usage comes from:
- The string conversion `s = str(n)` takes `O(log(n))` space
- The set `nums` stores at most `|digits|` elements, which is `O(|digits|)` but typically `|digits| ≤ 9`
- The recursion stack depth is at most `len(s)` = `O(log(n))`
- The memoization cache stores at most `O(log(n) * 2 * 2)` = `O(log(n))` states
- Overall space complexity: `O(log(n) + |digits|)` = `O(log(n))` (since `|digits|` is bounded by 9)

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Handling the Leading Zero State**

**Pitfall:** A common mistake is misunderstanding when and how to handle leading zeros. Developers often fail to treat a leading zero as "skip this position", which is how the algorithm counts numbers shorter than `n`.

**Incorrect Implementation:**
```python
# WRONG: This allows 0 to be treated as a valid digit when it's in the digits array
if digit in available_digits:
    total_count += count_valid_numbers(position + 1, False, is_bounded and (digit == max_digit))
```

**Why it's wrong:** The problem guarantees that `digits` contains only `'1'` to `'9'`, so `0` is never in `available_digits`. Without the separate leading-zero branch, the code can never skip a position, so it only counts numbers with exactly as many digits as `n`. For `digits = ['1', '3', '5']` and `n = 100`, it returns `0` instead of `12`, because every 1-digit and 2-digit number is missed.

**Correct Implementation:**
```python
# Handle leading zeros separately from regular digits
if digit == 0 and is_leading_zero:
    # Continue with leading zeros, don't place any digit
    total_count += count_valid_numbers(position + 1, True, is_bounded and (digit == max_digit))
elif digit in available_digits:
    # Only place non-zero digits or zero after a non-zero digit
    total_count += count_valid_numbers(position + 1, False, is_bounded and (digit == max_digit))
```

### 2. **Forgetting to Update the Bounded State Correctly**

**Pitfall:** Failing to properly update the `is_bounded` parameter when making recursive calls, especially forgetting that it should only remain `True` when both conditions are met: we were already bounded AND we placed the maximum allowed digit.

**Incorrect Implementation:**
```python
# WRONG: Always passing the same bounded state
total_count += count_valid_numbers(position + 1, False, is_bounded)

# OR WRONG: Only checking if digit equals max_digit
total_count += count_valid_numbers(position + 1, False, digit == max_digit)
```

**Correct Implementation:**
```python
# Bounded remains true only if BOTH conditions are met
total_count += count_valid_numbers(
    position + 1,
    False,
    is_bounded and (digit == max_digit)
)
```

### 3. **Not Handling Numbers with Fewer Digits Than n**

**Pitfall:** The algorithm naturally handles numbers with fewer digits through the leading zero mechanism, but developers sometimes try to add special logic for this case, which can lead to double counting or missed cases.

**Incorrect Approach:**
```python
# WRONG: Trying to handle shorter numbers separately
def count_valid_numbers(position, is_leading_zero, is_bounded):
    if position >= len(n_str):
        return 0 if is_leading_zero else 1

    # Trying to add special case for shorter numbers
    if not is_leading_zero:
        # This creates problems with counting
        total_count = 1  # Count current number as valid
```

**Why it works correctly without special handling:**
The leading zero mechanism naturally handles shorter numbers. For example, with `n = 100`:
- "5" is represented as "005" with two leading zeros
- "13" is represented as "013" with one leading zero
- The algorithm correctly counts these by allowing positions to be skipped via leading zeros

### 4. **Using the Wrong Data Structure for Digit Lookup**

**Pitfall:** Using a list instead of a set for checking if a digit is available, leading to O(D) lookup time instead of O(1).

**Inefficient Implementation:**
```python
# WRONG: Keep digits as a list
available_digits = [int(digit) for digit in digits]

# Later in the code - O(D) lookup
if digit in available_digits:  # Linear search
    # process
```

**Efficient Implementation:**
```python
# CORRECT: Convert to set for O(1) lookup
available_digits = {int(digit) for digit in digits}

# O(1) lookup
if digit in available_digits:
    # process
```

### 5. **Incorrect Base Case Return Value**

**Pitfall:** Returning the wrong value in the base case, particularly confusing when to return 0 vs 1.

**Incorrect Implementation:**
```python
# WRONG: Always returning 1
if position >= len(n_str):
    return 1

# OR WRONG: Using wrong logic
if position >= len(n_str):
    return 1 if is_leading_zero else 0  # Backwards!
```

**Correct Implementation:**
```python
if position >= len(n_str):
    # Return 1 only if we've formed a valid number (not all leading zeros)
    return 0 if is_leading_zero else 1
    # Or equivalently: return is_leading_zero ^ 1
```

**Key Insight:** The base case should return 1 only when we've successfully formed a valid positive integer (at least one non-zero digit was placed), and 0 when we've only encountered leading zeros (no number was formed).

