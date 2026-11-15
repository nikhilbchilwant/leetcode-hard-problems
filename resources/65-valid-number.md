# 65. Valid Number

> Source: https://algo.monster/liteproblems/65

## Problem Statement

Given a string `s`, you need to determine whether it represents a valid number.

A **valid number** can be either:
1. An **integer number** followed by an **optional exponent**
2. A **decimal number** followed by an **optional exponent**

The components are defined as follows:

**Integer number**: An optional sign (`+` or `-`) followed by one or more digits.

**Decimal number**: An optional sign (`+` or `-`) followed by one of these formats:
- Digits followed by a dot `.` (e.g., `"3."`)
- Digits followed by a dot `.` followed by more digits (e.g., `"3.14"`)
- A dot `.` followed by digits (e.g., `".5"`)

**Exponent**: The letter `e` or `E` followed by an integer number (which can have its own sign).

**Digits**: One or more numeric characters (0-9).

Examples of **valid numbers**:
- `"2"` - simple integer
- `"-0.1"` - negative decimal
- `"4."` - decimal with trailing dot
- `"-.9"` - decimal starting with dot
- `"2e10"` - integer with exponent
- `"3e+7"` - exponent with explicit positive sign
- `"53.5e93"` - decimal with exponent

Examples of **invalid numbers**:
- `"abc"` - contains letters
- `"1e"` - exponent without integer
- `"e3"` - starts with exponent
- `"99e2.5"` - exponent must be integer
- `"--6"` - double sign
- `"95a54e53"` - contains invalid character

The task is to return `true` if the string represents a valid number according to these rules, and `false` otherwise.

## Intuition

To validate whether a string represents a valid number, we need to carefully check if it follows the grammar rules for valid numbers. The key insight is to process the string character by character while tracking what we've seen so far.

Think of it like parsing the string from left to right and checking if each character is valid given what came before it. We need to enforce several rules:

1. **Sign handling**: A sign (`+` or `-`) can only appear at the beginning of the number or right after an exponent notation (`e` or `E`). We can't have multiple signs or signs in the wrong positions.

2. **Decimal point rules**: We can have at most one decimal point, and it must appear before any exponent notation. The tricky part is that a decimal point alone (without any digits) is invalid, but `.5` or `5.` are both valid.

3. **Exponent rules**: The exponent notation (`e` or `E`) can appear at most once, and it must be followed by a valid integer (which can have its own sign). The exponent cannot appear at the beginning or end of the string.

4. **Digit requirements**: We need at least some digits in the string. Pure signs, dots, or exponent notations without digits are invalid.

The approach is to use a state-based validation: we track whether we've seen a decimal point (`dot`) and whether we've seen an exponent (`e`). As we scan through each character, we check if it's valid based on our current state. For example, if we encounter a decimal point but we've already seen one before or we've already seen an exponent, we know the string is invalid.

The special cases to handle are:
- A string that's just a sign is invalid
- A decimal point must have digits either before or after it (or both)
- After an exponent notation, we need a valid integer (potentially with a sign)
- Any non-numeric character (except valid special characters like `.`, `e`, `E`, `+`, `-`) makes the string invalid

## Solution Approach

The implementation uses a character-by-character scanning approach with state tracking. Here's how the solution works:

**Step 1: Handle the optional sign at the beginning**
```python
if s[i] in '+-':
    i += 1
```
We first check if the string starts with a sign. If it does, we advance the pointer `i`. If after this step we've reached the end of the string (meaning the string was just a sign), we return `false`.

**Step 2: Special case for decimal point**
```python
if s[i] == '.' and (i + 1 == n or s[i + 1] in 'eE'):
    return False
```
We check if the current position is a decimal point with either nothing after it or immediately followed by an exponent. This handles invalid cases like `"."` or `".e5"`.

**Step 3: Main scanning loop with state tracking**
We use two counters:
- `dot`: tracks if we've seen a decimal point
- `e`: tracks if we've seen an exponent notation

Starting from position `j = i`, we scan through each remaining character:

**Case 1: Decimal point**
```python
if s[j] == '.':
    if e or dot:
        return False
    dot += 1
```
If we encounter a decimal point:
- Check if we've already seen an exponent (decimal points can't appear after exponents)
- Check if we've already seen a decimal point (only one allowed)
- If both checks pass, increment the `dot` counter

**Case 2: Exponent notation**
```python
elif s[j] in 'eE':
    if e or j == i or j == n - 1:
        return False
    e += 1
    if s[j + 1] in '+-':
        j += 1
        if j == n - 1:
            return False
```
If we encounter `e` or `E`:
- Check if we've already seen an exponent (only one allowed)
- Check if it's at the beginning (need digits before exponent)
- Check if it's at the end (need integer after exponent)
- Increment the `e` counter
- Check if the next character is a sign (allowed after exponent)
- If there's a sign, advance the pointer and ensure we're not at the end

**Case 3: Non-numeric character**
```python
elif not s[j].isnumeric():
    return False
```
Any character that's not a digit, decimal point, exponent, or properly placed sign is invalid.

**Step 4: Return result**
After scanning all characters without finding any violations, the string represents a valid number, so we return `true`.

This approach ensures all the grammar rules are enforced: proper sign placement, single decimal point before any exponent, single exponent with valid integer following it, and at least some digits present in the string.

## Example Walkthrough

Let's walk through the validation of `"-.9e2"` step by step:

**Initial Setup:**
- String: `"-.9e2"`, length n = 5
- i = 0 (starting position)

**Step 1: Handle optional sign**
- s[0] = '-' is a sign
- Increment i to 1
- String is not just a sign (i < n), continue

**Step 2: Check special decimal point case**
- s[1] = '.' is a decimal point
- Check if nothing after or immediately followed by 'e'/'E'
- s[2] = '9' (not 'e' or 'E'), so this check passes

**Step 3: Main scanning loop**
- Initialize: j = 1, dot = 0, e = 0

**Iteration 1 (j = 1):**
- s[1] = '.'
- Is it a decimal? Yes
- Have we seen an exponent (e > 0)? No
- Have we seen a decimal (dot > 0)? No
- Valid! Increment dot to 1
- j++ → j = 2

**Iteration 2 (j = 2):**
- s[2] = '9'
- Is it numeric? Yes
- Valid! Continue
- j++ → j = 3

**Iteration 3 (j = 3):**
- s[3] = 'e'
- Is it an exponent? Yes
- Have we seen an exponent (e > 0)? No
- Is it at the beginning (j == i)? No (3 ≠ 1)
- Is it at the end (j == n-1)? No (3 ≠ 4)
- Valid! Increment e to 1
- Check next character: s[4] = '2' (not a sign)
- Don't advance extra
- j++ → j = 4

**Iteration 4 (j = 4):**
- s[4] = '2'
- Is it numeric? Yes
- Valid! Continue
- j++ → j = 5

**Step 4: Loop ends (j = 5 = n)**
- No violations found
- Return `true`

The string `"-.9e2"` represents a valid number: a negative decimal (.9) with exponent (e2).

---

Now let's see an invalid example: `"e3"`

**Initial Setup:**
- String: `"e3"`, length n = 2
- i = 0

**Step 1: Handle optional sign**
- s[0] = 'e' is not a sign
- i remains 0

**Step 2: Check special decimal point case**
- s[0] = 'e' is not a decimal point
- Skip this check

**Step 3: Main scanning loop**
- Initialize: j = 0, dot = 0, e = 0

**Iteration 1 (j = 0):**
- s[0] = 'e'
- Is it an exponent? Yes
- Check conditions:
  - Have we seen an exponent (e > 0)? No
  - Is it at the beginning (j == i)? Yes (0 == 0) ❌
- This violates the rule: exponent cannot be at the beginning
- Return `false`

The string `"e3"` is invalid because it starts with an exponent notation without any preceding digits.

## Python Solution

```python
class Solution:
    def isNumber(self, s: str) -> bool:
        """
        Validates if a string represents a valid number.
        Valid numbers include integers, decimals, and scientific notation.
      
        Args:
            s: Input string to validate
          
        Returns:
            bool: True if string is a valid number, False otherwise
        """
        n = len(s)
        index = 0
      
        # Check for optional sign at the beginning
        if s[index] in '+-':
            index += 1
      
        # String cannot be just a sign
        if index == n:
            return False
      
        # Check for invalid cases: lone decimal point or decimal point followed by e/E
        if s[index] == '.' and (index + 1 == n or s[index + 1] in 'eE'):
            return False
      
        # Initialize flags for decimal point and exponent
        has_decimal = 0
        has_exponent = 0
        current_index = index
      
        # Parse the rest of the string
        while current_index < n:
            if s[current_index] == '.':
                # Decimal point cannot appear after exponent or if already present
                if has_exponent or has_decimal:
                    return False
                has_decimal += 1
              
            elif s[current_index] in 'eE':
                # Check for invalid exponent placement
                # Cannot have multiple exponents, at the start, or at the end
                if has_exponent or current_index == index or current_index == n - 1:
                    return False
                has_exponent += 1
              
                # Check for optional sign after exponent
                if s[current_index + 1] in '+-':
                    current_index += 1
                    # Sign cannot be the last character
                    if current_index == n - 1:
                        return False
                      
            elif not s[current_index].isnumeric():
                # Any non-numeric character (except . and e/E) is invalid
                return False
              
            current_index += 1
          
        return True
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm iterates through the string once using a while loop that goes from index `j = i` to `n`. Each character is visited at most once during this iteration. Inside the loop, all operations are constant time:
- Checking if a character is `.`, `e`, or `E`: `O(1)`
- Checking if a character is numeric using `isnumeric()`: `O(1)` 
- Incrementing counters and indices: `O(1)`

Even when we encounter `e` or `E` and potentially skip one additional character for the sign (`+` or `-`), each character is still processed at most once. Therefore, the overall time complexity is `O(n)`, where `n` is the length of the string.

**Space Complexity:** `O(1)`

The algorithm uses only a fixed amount of extra space:
- Integer variables: `n`, `i`, `j`, `dot`, `e`
- No additional data structures that grow with input size

All variables use constant space regardless of the input string length, resulting in `O(1)` space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Not Requiring Digits Before or After Decimal Point

**The Problem:** A scanner that only checks which characters are allowed, and in what order, accepts strings that contain no digits at all, such as `"."`, `"+."`, or `".e5"`. They must be rejected because a number needs at least one digit in its mantissa.

**How the main solution handles it:** It never relies on a digit count. The sign check rejects a lone `"+"`, the special-case check `s[i] == '.' and (i + 1 == n or s[i + 1] in 'eE')` rejects a dot with no digit after it when it is the first character of the mantissa, and the `j == i` check rejects an exponent with nothing before it. These three checks together cover every digit-less mantissa, which is easy to break when modifying the code.

**Example of Failure (without those checks):**
- Input: `"."` - Just a decimal point with no digits
- A scanner that only rejects a second dot or a dot after `e` accepts it
- Expected: `false`

**Alternative Solution:** Track digits explicitly with a flag, which makes the rule visible in one place:

```python
def isNumber(self, s: str) -> bool:
    n = len(s)
    index = 0
  
    # Check for optional sign at the beginning
    if s[index] in '+-':
        index += 1
  
    if index == n:
        return False
  
    # Track if we've seen required components
    has_decimal = False
    has_exponent = False
    has_digit = False  # NEW: Track if we've seen any digit
  
    current_index = index
  
    while current_index < n:
        if s[current_index].isnumeric():
            has_digit = True  # Mark that we've seen a digit
          
        elif s[current_index] == '.':
            if has_exponent or has_decimal:
                return False
            has_decimal = True
          
        elif s[current_index] in 'eE':
            # Must have seen at least one digit before exponent
            if not has_digit or has_exponent or current_index == n - 1:
                return False
            has_exponent = True
          
            # Reset digit flag for the exponent part
            has_digit = False
          
            # Handle optional sign after exponent
            if current_index + 1 < n and s[current_index + 1] in '+-':
                current_index += 1
                if current_index == n - 1:
                    return False
                  
        else:
            return False
          
        current_index += 1
  
    # Must have seen at least one digit (either in mantissa or after exponent)
    return has_digit
```

### Pitfall 2: Misjudging the Special-Case Dot Check

**The Problem:** The check `if s[index] == '.' and (index + 1 == n or s[index + 1] in 'eE')` can look too strict, as if it might reject `"4."` or `".5"`. It does not: for `"4."` the character at `index` is `4`, not a dot, and for `".5"` the dot is followed by a digit. The check fires only for a mantissa that starts with a dot and has no digit right after it (`"."`, `"-."`, `".e3"`), and those are all invalid.

The real risk is deleting the check because it looks redundant. Without it, `"."` and `".e3"` would be accepted.

**Alternative Solution:** If you want a version without special cases, track digits before and after the exponent separately:

```python
def isNumber(self, s: str) -> bool:
    n = len(s)
    index = 0
  
    if s[index] in '+-':
        index += 1
  
    if index == n:
        return False
  
    has_decimal = False
    has_exponent = False
    has_digit_before_e = False  # Digits before exponent
    has_digit_after_e = False   # Digits after exponent
  
    current_index = index
  
    while current_index < n:
        if s[current_index].isnumeric():
            if has_exponent:
                has_digit_after_e = True
            else:
                has_digit_before_e = True
              
        elif s[current_index] == '.':
            if has_exponent or has_decimal:
                return False
            has_decimal = True
          
        elif s[current_index] in 'eE':
            if has_exponent or not has_digit_before_e:
                return False
            has_exponent = True
          
            if current_index + 1 < n and s[current_index + 1] in '+-':
                current_index += 1
          
            if current_index == n - 1:
                return False
              
        else:
            return False
          
        current_index += 1
  
    # If we have exponent, must have digits after it
    # Otherwise, just need digits before it
    return has_digit_after_e if has_exponent else has_digit_before_e
```

This approach properly validates that:
1. There are digits in the mantissa (before any exponent)
2. If an exponent exists, there are digits after it
3. Decimal points can appear with digits on either or both sides

