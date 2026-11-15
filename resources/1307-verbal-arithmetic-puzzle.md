# 1307. Verbal Arithmetic Puzzle

> Source: https://algo.monster/liteproblems/1307

## Problem Statement

This problem asks you to determine if a cryptarithmetic equation can be solved. You're given a list of words that represent the left side of an addition equation, and a result word that represents the right side. Each letter in these words must be mapped to a unique digit (0-9) such that when the words are treated as numbers and added together, they equal the result.

The key constraints are:

1. **Each letter maps to exactly one digit (0-9)**: Once you assign a digit to a letter, that letter always represents that same digit throughout the equation.

2. **No two letters can map to the same digit**: Each letter must have a unique digit assignment.

3. **No leading zeros**: Numbers cannot have leading zeros, except for single-digit numbers. For example, if a word has multiple characters, its first character cannot map to 0.

4. **The equation must be valid**: When you replace all letters with their assigned digits and perform the addition, the sum of the words on the left must equal the result on the right.

For example:
- If `words = ["SEND", "MORE"]` and `result = "MONEY"`
- One valid mapping could be: S=9, E=5, N=6, D=7, M=1, O=0, R=8, Y=2
- This gives us: 9567 + 1085 = 10652, which is correct

The function should return `true` if at least one valid digit assignment exists that makes the equation work, and `false` otherwise.

## Intuition

Think about how you would solve a cryptarithmetic puzzle by hand. You'd try different digit assignments for letters, checking if they satisfy the equation. When you hit a contradiction, you'd erase your work and try different assignments. This is exactly [backtracking](/problems/backtracking).

The key insight is to process the addition column by column, just like manual addition works - from right to left with carries. Why? Because this allows us to fail fast. If the rightmost column doesn't add up correctly with our current letter-to-digit mappings, we know immediately that this path won't work and can backtrack early.

Consider the equation `SEND + MORE = MONEY`. When adding manually:
```
    S E N D
  + M O R E
  ---------
  M O N E Y
```

We start from the rightmost column: `D + E = Y` (or `Y + 10` if there's a carry). If our current digit assignments for D and E don't produce the correct digit for Y (considering any carry), we can immediately reject this path without checking the entire equation.

The solution cleverly traverses the problem in a 2D manner:
- **Columns** represent digit positions (ones, tens, hundreds, etc.)
- **Rows** represent different words in the equation

For each position, we:
1. Try to assign digits to unmapped letters
2. Check if the column sum is valid (considering carries)
3. If valid, move to the next position; if not, backtrack

The balance (`bal`) variable tracks the running sum as we process each column. When we finish a column, we check if `bal % 10` equals 0 (meaning the column adds up correctly), and carry forward `bal // 10` to the next column.

The leading zero constraint is handled by checking the position - if we're at the leftmost digit of a multi-character word, we can't assign 0 to it.

This approach is efficient because it prunes the search space early. Instead of generating all possible complete mappings and then checking the equation (which would be factorial in complexity), we validate incrementally and abandon invalid paths as soon as possible.

**Learn more about [Math](/problems/math-basics) and [Backtracking](/problems/backtracking) patterns.**

## Solution Approach

The implementation uses a recursive [backtracking](/problems/backtracking) function `isAnyMapping` that processes the equation systematically. Here's how the solution works:

**Data Structures:**
- `letToDig`: HashMap storing letter → digit mappings
- `digToLet`: Array storing digit → letter mappings (using "-" for unmapped digits)
- `bal`: Running balance to track the column sum with carries

**Main Algorithm Flow:**

1. **Setup Phase:**
   - Append the `result` word to the `words` list (treating it as the last word with negative sign)
   - Find `totalCols` (length of longest word) and `totalRows` (number of words including result)
   - Initialize empty mappings

2. **Recursive Processing (`isAnyMapping`):**
 
   The function processes position by position using `(row, col)` coordinates:
 
   **Base Cases:**
   - If `col == totalCols`: All columns processed, return `bal == 0` (no remaining carry)
   - If `row == totalRows`: Column complete, check if `bal % 10 == 0` and recurse to next column with carry `bal // 10`

3. **Processing Each Position:**
 
   For current word at position `(row, col)`:
   - Skip if word is shorter than current column: `col >= len(w)`
   - Extract letter from right to left: `letter = w[len(w) - 1 - col]`
   - Determine sign: positive for addends, negative for result (last word)

4. **Letter-to-Digit Assignment:**
 
   **Case 1: Letter already mapped**
   - Validate no leading zeros: Check if digit is 0 and position is leftmost in multi-character word
   - If valid, continue with existing mapping and update balance

   **Case 2: Letter needs mapping**
   - Try each digit 0-9:
     ```python
     for i in range(10):
         if digToLet[i] == "-":  # Digit not yet assigned
     ```
   - Check leading zero constraint: `i != 0` or single character word or not leftmost position
   - Assign mapping: `digToLet[i] = letter`, `letToDig[letter] = i`
   - Recursively check if this leads to solution
   - Backtrack if unsuccessful: Remove mappings and try next digit

5. **Balance Tracking:**
 
   The `bal` variable accumulates the weighted sum for each column:
   - Add digit value for addend words: `bal + sign * letToDig[letter]`
   - Subtract digit value for result word: `bal - letToDig[letter]`
   - When column completes, `bal % 10` should be 0, and `bal // 10` becomes carry

6. **Pruning Strategy:**
 
   The column-by-column approach enables early termination:
   - If a column doesn't balance with current mappings, immediately backtrack
   - No need to complete entire mapping before validation
   - Significantly reduces search space from O(10!) to practical time

The algorithm elegantly combines the manual addition process with [backtracking](/problems/backtracking) search, validating constraints incrementally rather than generating complete mappings upfront.

## Example Walkthrough

Let's trace through a simple example: `words = ["AB", "CD"]` and `result = "EF"`

We need to find digit mappings such that `AB + CD = EF`.

**Setup:**
- Append result to words: `["AB", "CD", "EF"]`
- `totalCols = 2` (max word length)
- `totalRows = 3` (number of words)
- Initialize empty mappings

**Processing (row, col) positions:**

Start at `(0, 0)` - rightmost column, first word:

```
    A B
  + C D
  -----
    E F
    ↑
```

**Column 0 (ones place):**

1. `(0, 0)`: Word "AB", position 0 → letter 'B'
   - B unmapped, try digits 0-9
   - Let's try B=5: `bal = 0 + 5 = 5`

2. `(1, 0)`: Word "CD", position 0 → letter 'D'
   - D unmapped, try digits (except 5)
   - Let's try D=7: `bal = 5 + 7 = 12`

3. `(2, 0)`: Word "EF", position 0 → letter 'F'
   - F unmapped, need F such that `(12 - F) % 10 = 0`
   - F must be 2: `bal = 12 - 2 = 10`
 
4. `(3, 0)`: Row complete, check column
   - `bal % 10 = 0` ✓ (column balances)
   - Move to column 1 with carry: `bal = 10 // 10 = 1`

**Column 1 (tens place):**

```
    A B
  + C D
  -----
    E F
  ↑
```

5. `(0, 1)`: Word "AB", position 1 → letter 'A'
   - A unmapped, can't be 0 (leading digit)
   - Try A=3: `bal = 1 + 3 = 4`

6. `(1, 1)`: Word "CD", position 1 → letter 'C'
   - C unmapped, can't be 0 (leading digit)
   - Try C=4: `bal = 4 + 4 = 8`

7. `(2, 1)`: Word "EF", position 1 → letter 'E'
   - E unmapped, can't be 0 (leading digit)
   - Need E such that `(8 - E) % 10 = 0`
   - E must be 8: `bal = 8 - 8 = 0`

8. `(3, 1)`: Row complete
   - `bal % 10 = 0` ✓
   - Move to column 2 with carry: `bal = 0`

9. `(0, 2)`: `col == totalCols`
   - All columns processed
   - `bal == 0` ✓ (no remaining carry)
   - **Success!**

**Valid mapping found:**
- A=3, B=5, C=4, D=7, E=8, F=2
- Verification: 35 + 47 = 82 ✓

If at any point a column didn't balance (e.g., if we had tried A=1 instead, giving us 15 + 47 = 62, but E can't be both 6 and 8), we would backtrack and try different digit assignments. The algorithm continues until finding a valid mapping or exhausting all possibilities.

## Python Solution

```python
class Solution:
    def isAnyMapping(
        self, words, row, col, balance, letter_to_digit, digit_to_letter, total_rows, total_cols
    ):
        """
        Recursively check if there exists a valid digit mapping for the cryptarithmetic equation.
      
        Args:
            words: List of words including the result word at the end
            row: Current row (word) being processed
            col: Current column (digit position from right) being processed
            balance: Running sum/balance for the current column
            letter_to_digit: Dictionary mapping letters to their assigned digits
            digit_to_letter: List mapping digits to their assigned letters
            total_rows: Total number of words
            total_cols: Maximum length among all words
      
        Returns:
            True if a valid mapping exists, False otherwise
        """
        # Base case: Processed all columns successfully
        if col == total_cols:
            return balance == 0

        # Finished processing current column, move to next column
        if row == total_rows:
            # Check if current column sums to 0 (mod 10) and propagate carry
            return balance % 10 == 0 and self.isAnyMapping(
                words, 0, col + 1, balance // 10, letter_to_digit, digit_to_letter, total_rows, total_cols
            )

        current_word = words[row]

        # Skip if current word doesn't have a character at this column position
        if col >= len(current_word):
            return self.isAnyMapping(
                words, row + 1, col, balance, letter_to_digit, digit_to_letter, total_rows, total_cols
            )

        # Get the letter at current position (counting from right)
        letter = current_word[len(current_word) - 1 - col]

        # Determine sign: positive for addends, negative for result (last word)
        sign = 1 if row < total_rows - 1 else -1

        # Case 1: Letter already has a digit mapping
        if letter in letter_to_digit:
            digit = letter_to_digit[letter]
          
            # Check for leading zero constraint
            is_leading_position = (col == len(current_word) - 1)
            is_single_char_word = (len(current_word) == 1)
          
            # Leading zeros are only allowed for single-character words
            if digit != 0 or is_single_char_word or not is_leading_position:
                return self.isAnyMapping(
                    words,
                    row + 1,
                    col,
                    balance + sign * digit,
                    letter_to_digit,
                    digit_to_letter,
                    total_rows,
                    total_cols,
                )
            else:
                return False

        # Case 2: Letter needs a new digit mapping
        else:
            # Try each available digit
            for digit in range(10):
                # Check if digit is available
                if digit_to_letter[digit] == "-":
                    # Check leading zero constraint
                    is_leading_position = (col == len(current_word) - 1)
                    is_single_char_word = (len(current_word) == 1)
                  
                    # Skip if this would create an invalid leading zero
                    if digit == 0 and is_leading_position and not is_single_char_word:
                        continue
                  
                    # Make the assignment
                    digit_to_letter[digit] = letter
                    letter_to_digit[letter] = digit

                    # Recursively check if this assignment works
                    if self.isAnyMapping(
                        words,
                        row + 1,
                        col,
                        balance + sign * digit,
                        letter_to_digit,
                        digit_to_letter,
                        total_rows,
                        total_cols,
                    ):
                        return True

                    # Backtrack: undo the assignment
                    digit_to_letter[digit] = "-"
                    if letter in letter_to_digit:
                        del letter_to_digit[letter]

        # No valid mapping found
        return False

    def isSolvable(self, words, result):
        """
        Check if the cryptarithmetic equation has a valid solution.
      
        Args:
            words: List of addend words
            result: The result word
      
        Returns:
            True if equation is solvable, False otherwise
        """
        # Combine all words into a single list (result goes last)
        all_words = words + [result]

        # Count total words and find maximum word length
        total_rows = len(all_words)
        total_cols = max(len(word) for word in all_words)

        # Initialize mapping structures
        letter_to_digit = {}  # Maps each letter to its assigned digit
        digit_to_letter = ["-"] * 10  # Maps each digit to its assigned letter ("-" means unassigned)

        # Start the recursive search
        return self.isAnyMapping(
            all_words, 0, 0, 0, letter_to_digit, digit_to_letter, total_rows, total_cols
        )
```

## Time and Space Complexity

**Time Complexity:** `O(n * 10^m)` where `n` is the total number of characters across all words and `m` is the number of unique letters.

The algorithm uses backtracking to try all possible digit mappings for each unique letter. In the worst case:
- We have up to `m` unique letters that need to be assigned digits
- For each letter, we try up to 10 different digits (0-9)
- This gives us `10^m` possible combinations to explore
- For each combination, we traverse through all positions in the grid, which is bounded by `O(n)` where `n` is the total number of characters
- The recursion depth is at most `totalRows * totalCols`, which is also `O(n)`

**Space Complexity:** `O(m + n)` where `m` is the number of unique letters and `n` is the total number of characters.

The space complexity consists of:
- `letToDig` HashMap: `O(m)` for storing letter-to-digit mappings
- `digToLet` list: `O(10) = O(1)` for storing digit-to-letter mappings
- Recursion call stack: `O(totalRows * totalCols) = O(n)` in the worst case
- Input storage for words list: `O(n)` for storing all characters

Therefore, the total space complexity is `O(m + n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Leading Zero Validation**

**The Pitfall:** A common mistake is incorrectly checking for leading zeros. Developers often forget that:
- Single-character words CAN be 0 (e.g., "A" can map to 0)
- The leading zero check should only apply to the leftmost character of multi-character words
- The position check must account for processing from right to left

**Incorrect Implementation:**
```python
# Wrong: Doesn't allow single-character words to be 0
if digit == 0 and col == len(current_word) - 1:
    return False

# Wrong: Checks position incorrectly
if digit == 0 and col == 0:  # This checks rightmost, not leftmost!
    return False
```

**Correct Implementation:**
```python
is_leading_position = (col == len(current_word) - 1)
is_single_char_word = (len(current_word) == 1)

# Only reject if it's a leading zero in a multi-character word
if digit == 0 and is_leading_position and not is_single_char_word:
    continue  # or return False
```

### 2. **Forgetting to Properly Backtrack**

**The Pitfall:** When trying different digit assignments, it's crucial to completely undo the mapping if it doesn't lead to a solution. Forgetting to remove the letter from the dictionary can cause incorrect results.

**Incorrect Implementation:**
```python
# Wrong: Only undoing one side of the mapping
digit_to_letter[digit] = letter
letter_to_digit[letter] = digit

if self.isAnyMapping(...):
    return True

digit_to_letter[digit] = "-"  # Forgot to remove from letter_to_digit!
```

**Correct Implementation:**
```python
# Make the assignment
digit_to_letter[digit] = letter
letter_to_digit[letter] = digit

if self.isAnyMapping(...):
    return True

# Properly backtrack both mappings
digit_to_letter[digit] = "-"
if letter in letter_to_digit:
    del letter_to_digit[letter]
```

### 3. **Incorrect Balance/Carry Handling**

**The Pitfall:** The balance accumulation and carry propagation can be confusing. Common errors include:
- Not resetting balance when moving to a new column
- Incorrectly handling the sign for the result word
- Not properly checking if a column sums to 0 (mod 10)

**Incorrect Implementation:**
```python
# Wrong: Not checking modulo 10 for column validation
if row == total_rows:
    return balance == 0 and self.isAnyMapping(
        words, 0, col + 1, balance, ...)  # Should pass carry, not full balance!

# Wrong: Using same sign for all words
sign = 1  # Should be negative for result word
```

**Correct Implementation:**
```python
if row == total_rows:
    # Check column sums to 0 (mod 10) and pass carry to next column
    return balance % 10 == 0 and self.isAnyMapping(
        words, 0, col + 1, balance // 10, ...)

# Correct sign assignment
sign = 1 if row < total_rows - 1 else -1
```

### 4. **Processing Words in Wrong Direction**

**The Pitfall:** Since addition happens from right to left (units, tens, hundreds...), the column index represents positions from the right. Accessing characters incorrectly will lead to wrong results.

**Incorrect Implementation:**
```python
# Wrong: Accessing from left
letter = current_word[col]
```

**Correct Implementation:**
```python
# Access from right side (index from the end)
letter = current_word[len(current_word) - 1 - col]
```

### 5. **Not Handling Words of Different Lengths**

**The Pitfall:** Words can have different lengths, and shorter words don't have characters at higher column positions. Failing to skip these positions causes index errors or incorrect processing.

**Incorrect Implementation:**
```python
# Wrong: Always trying to access character at position
letter = current_word[len(current_word) - 1 - col]  # IndexError if col >= len(word)
```

**Correct Implementation:**
```python
# Skip if word doesn't have a character at this column
if col >= len(current_word):
    return self.isAnyMapping(
        words, row + 1, col, balance, ...)

# Now safe to access
letter = current_word[len(current_word) - 1 - col]
```

