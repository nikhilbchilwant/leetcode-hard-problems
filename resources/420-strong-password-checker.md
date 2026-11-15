# 420. Strong Password Checker

> Source: https://algo.monster/liteproblems/420

## Problem Statement

This problem asks you to determine the minimum number of operations needed to make a password strong. A strong password must satisfy three conditions:

1. **Length requirement**: The password must have between 6 and 20 characters (inclusive)
2. **Character type requirement**: The password must contain at least one lowercase letter, one uppercase letter, and one digit
3. **No triple repetition**: The password cannot have three identical characters in a row (e.g., `"aaa"` is not allowed)

You can perform three types of operations, each counting as one step:
- **Insert**: Add one character at any position
- **Delete**: Remove one character from any position  
- **Replace**: Change one character to a different character

The challenge is to find the minimum number of operations needed based on the password's current state:

- **If the password is too short** (less than 6 characters): You'll need to insert characters. These insertions can also help satisfy missing character types.

- **If the password has acceptable length** (6-20 characters): Focus on fixing any triple repetitions and ensuring all three character types are present. Replacements are often the most efficient operation here.

- **If the password is too long** (more than 20 characters): You must delete excess characters. The strategy becomes complex because deletions can also help break up triple repetitions efficiently. The solution needs to carefully balance deletions with replacements to minimize total operations.

## Intuition

The key insight is to handle the problem differently based on the password length, as each scenario has distinct optimal strategies.

**For passwords shorter than 6 characters:** We must add characters to reach the minimum length. Since we're already adding characters, we can strategically choose what to add to satisfy the character type requirements. The answer is simply `max(6 - n, 3 - types)` because we need at least `6 - n` insertions for length, and at least `3 - types` operations to get all required character types. These goals can overlap when we insert missing character types.

**For passwords with length 6-20:** Length is acceptable, so we focus on fixing repetitions and character types. When we encounter sequences of 3+ identical characters, we need to break them up. A sequence of length `k` requires `k // 3` replacements to fix (e.g., `"aaaaa"` needs 1 replacement to become `"aaXaa"`, and `"aaaaaa"` needs 2 replacements). Replacements are perfect here because one replacement can both break a repetition AND add a missing character type. The answer is `max(replace, 3 - types)` since replacements can serve double duty.

**For passwords longer than 20 characters:** This is the most complex case. We must delete characters, but deletions interact cleverly with repetitions. Consider `"aaa"` - normally this needs 1 replacement. But if we delete one character to get `"aa"`, no replacement is needed! This creates an optimization opportunity.

The crucial observation is that sequences of repeated characters have different deletion efficiencies based on their length modulo 3:
- Length `3k` (e.g., `"aaa"`, `"aaaaaa"`): Deleting 1 character saves 1 replacement
- Length `3k + 1` (e.g., `"aaaa"`): Deleting 2 characters saves 1 replacement  
- Length `3k + 2` (e.g., `"aaaaa"`): Deleting 3 characters saves 1 replacement

The algorithm prioritizes deletions in order of efficiency: first handle `3k` sequences (1 deletion per saved replacement), then `3k + 1` sequences (2 deletions per saved replacement), and finally `3k + 2` sequences (3 deletions per saved replacement). After using deletions optimally to reduce required replacements, we still need to ensure we have all character types, leading to the formula `n - 20 + max(replace, 3 - types)`.

## Solution Approach

The implementation divides the problem into three cases based on password length, with a helper function to count character types.

**Helper Function - Counting Character Types:**
```python
def countTypes(s):
    a = b = c = 0
    for ch in s:
        if ch.islower(): a = 1
        elif ch.isupper(): b = 1
        elif ch.isdigit(): c = 1
    return a + b + c
```
This function returns how many of the three required character types are present (0-3).

**Case 1: Length < 6**
```python
if n < 6:
    return max(6 - n, 3 - types)
```
We need `6 - n` insertions to reach minimum length and `3 - types` operations to get all character types. Since insertions can add missing types, we take the maximum.

**Case 2: Length 6-20**
```python
if n <= 20:
    replace = cnt = 0
    prev = '~'
    for curr in password:
        if curr == prev:
            cnt += 1
        else:
            replace += cnt // 3
            cnt = 1
            prev = curr
    replace += cnt // 3
    return max(replace, 3 - types)
```
We scan through the password tracking consecutive identical characters. When the character changes or we reach the end, we calculate replacements needed: `cnt // 3`. For example, `"aaaaa"` (cnt=5) needs `5//3 = 1` replacement. The final answer is the maximum of total replacements needed and missing character types.

**Case 3: Length > 20**
```python
replace = cnt = 0
remove, remove2 = n - 20, 0
prev = '~'
```
We need to remove `n - 20` characters. The algorithm tracks:
- `remove`: remaining deletions available
- `remove2`: count of sequences with length `3k + 1`
- `replace`: replacements still needed after optimal deletions

First pass - handle sequences and optimize `3k` length sequences:
```python
for curr in password:
    if curr == prev:
        cnt += 1
    else:
        if remove > 0 and cnt >= 3:
            if cnt % 3 == 0:  # Length 3k - most efficient
                remove -= 1
                replace -= 1
            elif cnt % 3 == 1:  # Length 3k+1 - track for later
                remove2 += 1
        replace += cnt // 3
        cnt = 1
        prev = curr
```

Second pass - use remaining deletions optimally:
```python
use2 = min(replace, remove2, remove // 2)  # Handle 3k+1 sequences
replace -= use2
remove -= use2 * 2

use3 = min(replace, remove // 3)  # Handle 3k+2 sequences
replace -= use3
remove -= use3 * 3
```

The algorithm prioritizes:
1. Delete 1 character from `3k` sequences (1 deletion saves 1 replacement)
2. Delete 2 characters from `3k+1` sequences (2 deletions save 1 replacement)
3. Delete 3 characters from `3k+2` sequences (3 deletions save 1 replacement)

Finally, return `n - 20 + max(replace, 3 - types)`, which represents:
- `n - 20`: mandatory deletions
- `max(replace, 3 - types)`: additional operations needed for repetitions and character types

This greedy approach ensures we use deletions as efficiently as possible to minimize total operations.

## Example Walkthrough

Let's walk through a concrete example: password = `"aaaaAAA1234567890123"`

**Step 1: Initial Analysis**
- Length: 20 characters (valid range, no insertions or deletions needed)
- Character types: lowercase 'a', uppercase 'A', digit '1' - all 3 types present ✓
- Repetitions: "aaaa" (4 consecutive) and "AAA" (3 consecutive)

**Step 2: Count Required Replacements**
We scan through the password tracking consecutive identical characters:
- "aaaa" has length 4, needs `4 // 3 = 1` replacement
- "AAA" has length 3, needs `3 // 3 = 1` replacement
- Total replacements needed: 2

**Step 3: Calculate Final Answer**
Since we're in the 6-20 length range:
- Replacements needed: 2
- Missing character types: 0 (we have all 3)
- Answer: `max(2, 0) = 2`

**The Fix:** We could change the password to `"aaXaAAX1234567890123"` with 2 replacements.

---

Now let's see a more complex example: password = `"aaaaaaaa"` (8 'a's)

**Step 1: Initial Analysis**
- Length: 8 characters (valid range)
- Character types: only lowercase - missing uppercase and digit (need 2 more types)
- Repetitions: "aaaaaaaa" (8 consecutive)

**Step 2: Count Required Replacements**
- "aaaaaaaa" has length 8, needs `8 // 3 = 2` replacements

**Step 3: Calculate Final Answer**
- Replacements needed: 2
- Missing character types: 2 (need uppercase and digit)
- Answer: `max(2, 2) = 2`

**The Fix:** We could change to `"aaAaa1aa"` - the 2 replacements both break repetitions AND add missing character types, achieving both goals simultaneously!

---

Finally, a deletion example: password = `"aaaaaaaaaaaaaaaaaaaaa"` (21 'a's)

**Step 1: Initial Analysis**
- Length: 21 characters (too long, need to delete 1)
- Character types: only lowercase - missing 2 types
- Repetitions: 21 consecutive 'a's

**Step 2: Apply Deletion Strategy**
- Without any optimization: 21 'a's would need `21 // 3 = 7` replacements
- We must delete 1 character (21 - 20 = 1)
- Length 21 = 3×7, so this is a `3k` length sequence
- Deleting 1 character gives us 20 'a's, which needs `20 // 3 = 6` replacements
- We saved 1 replacement with 1 deletion!

**Step 3: Calculate Final Answer**
- Mandatory deletions: 1
- Replacements still needed after deletion: 6
- Missing character types: 2
- Answer: `1 + max(6, 2) = 1 + 6 = 7`

**The Fix:** Delete one 'a' to get 20 characters, then make 6 replacements to break repetitions and add missing types.

## Python Solution

```python
class Solution:
    def strongPasswordChecker(self, password: str) -> int:
        def count_character_types(s: str) -> int:
            """
            Count how many character types (lowercase, uppercase, digit) are present.
            Returns a number between 0 and 3.
            """
            has_lower = has_upper = has_digit = 0
          
            for char in s:
                if char.islower():
                    has_lower = 1
                elif char.isupper():
                    has_upper = 1
                elif char.isdigit():
                    has_digit = 1
          
            return has_lower + has_upper + has_digit
      
        # Count existing character types and get password length
        existing_types = count_character_types(password)
        password_length = len(password)
      
        # Case 1: Password is too short (less than 6 characters)
        if password_length < 6:
            # Need to add characters to reach 6, and ensure we have all 3 types
            chars_to_add = 6 - password_length
            types_to_add = 3 - existing_types
            return max(chars_to_add, types_to_add)
      
        # Case 2: Password length is acceptable (6-20 characters)
        if password_length <= 20:
            # Count replacements needed to break sequences of 3+ repeating characters
            replacements_needed = 0
            consecutive_count = 1
            previous_char = '~'  # Use a non-password character as initial value
          
            for current_char in password:
                if current_char == previous_char:
                    consecutive_count += 1
                else:
                    # For every 3 consecutive chars, we need 1 replacement
                    replacements_needed += consecutive_count // 3
                    consecutive_count = 1
                    previous_char = current_char
          
            # Don't forget the last sequence
            replacements_needed += consecutive_count // 3
          
            # Need to fix both repeating sequences and missing character types
            types_to_add = 3 - existing_types
            return max(replacements_needed, types_to_add)
      
        # Case 3: Password is too long (more than 20 characters)
        # Strategy: Remove excess characters optimally to minimize replacements
      
        replacements_needed = 0
        consecutive_count = 1
        chars_to_remove = password_length - 20
      
        # Track sequences with remainder 1 when divided by 3
        # These are priority for using 2-char removals
        sequences_with_remainder_1 = 0
      
        previous_char = '~'
      
        for current_char in password:
            if current_char == previous_char:
                consecutive_count += 1
            else:
                if chars_to_remove > 0 and consecutive_count >= 3:
                    # Sequences divisible by 3: removing 1 char saves 1 replacement
                    if consecutive_count % 3 == 0:
                        chars_to_remove -= 1
                        replacements_needed -= 1
                    # Sequences with remainder 1: good candidates for 2-char removal
                    elif consecutive_count % 3 == 1:
                        sequences_with_remainder_1 += 1
              
                replacements_needed += consecutive_count // 3
                consecutive_count = 1
                previous_char = current_char
      
        # Process the last sequence
        if chars_to_remove > 0 and consecutive_count >= 3:
            if consecutive_count % 3 == 0:
                chars_to_remove -= 1
                replacements_needed -= 1
            elif consecutive_count % 3 == 1:
                sequences_with_remainder_1 += 1
      
        replacements_needed += consecutive_count // 3
      
        # Use remaining removals optimally
        # Priority 1: Remove 2 chars from sequences with remainder 1
        two_char_removals = min(replacements_needed, sequences_with_remainder_1, chars_to_remove // 2)
        replacements_needed -= two_char_removals
        chars_to_remove -= two_char_removals * 2
      
        # Priority 2: Remove 3 chars to eliminate entire replacement needs
        three_char_removals = min(replacements_needed, chars_to_remove // 3)
        replacements_needed -= three_char_removals
        chars_to_remove -= three_char_removals * 3
      
        # Total operations = deletions + max(remaining replacements, missing types)
        types_to_add = 3 - existing_types
        return (password_length - 20) + max(replacements_needed, types_to_add)
```

## Time and Space Complexity

**Time Complexity:** `O(n)` where `n` is the length of the password string.

The algorithm makes a single pass through the password string to count character types in the `countTypes` function, which takes `O(n)` time. Then, depending on the password length:
- If `n < 6`: Direct calculation in `O(1)`
- If `6 <= n <= 20`: One pass through the password to count consecutive characters, taking `O(n)`
- If `n > 20`: One pass through the password to count consecutive characters and calculate removals, taking `O(n)`

All operations within the loops (comparisons, arithmetic operations, modulo calculations) are `O(1)`. Therefore, the overall time complexity is `O(n)`.

**Space Complexity:** `O(1)`

The algorithm uses only a constant amount of extra space:
- Variables for counting types (`a`, `b`, `c`, `types`): `O(1)`
- Variables for tracking consecutive characters and operations (`replace`, `cnt`, `remove`, `remove2`, `use2`, `use3`): `O(1)`
- Variables for character comparison (`prev`, `curr`): `O(1)`

No additional data structures that scale with input size are used, making the space complexity `O(1)`.

## Common Pitfalls

### 1. **Forgetting to Process the Last Sequence of Repeating Characters**

One of the most common mistakes is not handling the final sequence of consecutive characters after the main loop completes. The loop only processes a sequence when it encounters a different character, but if the password ends with repeating characters, these won't be counted.

**Incorrect Implementation:**
```python
for current_char in password:
    if current_char == previous_char:
        consecutive_count += 1
    else:
        replacements_needed += consecutive_count // 3
        consecutive_count = 1
        previous_char = current_char
# Missing: Final sequence is never processed!
```

**Correct Implementation:**
```python
for current_char in password:
    if current_char == previous_char:
        consecutive_count += 1
    else:
        replacements_needed += consecutive_count // 3
        consecutive_count = 1
        previous_char = current_char

# Process the last sequence after loop ends
replacements_needed += consecutive_count // 3
```

### 2. **Incorrect Initialization of Previous Character**

Using a character that could appear in the password as the initial value for `previous_char` can cause the first character to be counted as part of a sequence. With `consecutive_count = 1` and `previous_char = 'a'`, the password `"aab..."` starts with a count of 3 for the two leading `'a'` characters and adds a replacement that is not needed. Any value that never equals a password character works, such as `'~'` or the empty string `''`.

**Incorrect Implementation:**
```python
previous_char = 'a'  # If password starts with 'a', it's counted incorrectly
```

**Correct Implementation:**
```python
previous_char = '~'  # Use a character that won't appear in passwords
```

### 3. **Misunderstanding the Deletion Priority in Case 3**

When the password is too long, not understanding why we prioritize sequences with different remainders can lead to suboptimal solutions. The algorithm must recognize that:
- Deleting 1 char from a sequence of length `3k` (remainder 0) saves 1 replacement
- Deleting 2 chars from a sequence of length `3k+1` (remainder 1) saves 1 replacement
- Deleting 3 chars from a sequence of length `3k+2` (remainder 2) saves 1 replacement

**Incorrect Approach:**
```python
# Simply deleting characters without considering optimization
chars_to_remove = password_length - 20
return chars_to_remove + max(replacements_needed, 3 - existing_types)
```

**Correct Approach:**
```python
# First handle remainder 0 sequences (most efficient)
if consecutive_count % 3 == 0:
    chars_to_remove -= 1
    replacements_needed -= 1

# Then handle remainder 1 sequences with 2-char removals
two_char_removals = min(replacements_needed, sequences_with_remainder_1, chars_to_remove // 2)

# Finally handle remainder 2 sequences with 3-char removals
three_char_removals = min(replacements_needed, chars_to_remove // 3)
```

### 4. **Not Handling Edge Cases in the Counting Logic**

Failing to properly reset the consecutive count or handle single characters can lead to incorrect calculations.

**Incorrect Implementation:**
```python
for current_char in password:
    if current_char == previous_char:
        consecutive_count += 1
    else:
        replacements_needed += consecutive_count // 3
        # Forgot to reset consecutive_count!
        previous_char = current_char
```

**Correct Implementation:**
```python
for current_char in password:
    if current_char == previous_char:
        consecutive_count += 1
    else:
        replacements_needed += consecutive_count // 3
        consecutive_count = 1  # Reset count for new sequence
        previous_char = current_char
```

