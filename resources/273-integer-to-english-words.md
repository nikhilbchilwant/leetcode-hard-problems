# 273. Integer to English Words

> Source: https://algo.monster/liteproblems/273

## Problem Statement

This problem asks you to convert a non-negative integer into its English words representation.

Given a non-negative integer `num`, you need to return a string that represents the number spelled out in English words.

For example:
- Input: `123` → Output: `"One Hundred Twenty Three"`
- Input: `12345` → Output: `"Twelve Thousand Three Hundred Forty Five"`
- Input: `1234567` → Output: `"One Million Two Hundred Thirty Four Thousand Five Hundred Sixty Seven"`
- Input: `0` → Output: `"Zero"`

The solution breaks down the problem by:

1. **Handling special cases**: Numbers from 0-19 have unique names (`"Zero"`, `"One"`, `"Two"`, ..., `"Nineteen"`)

2. **Handling tens**: Numbers like 20, 30, 40, etc. have their own names (`"Twenty"`, `"Thirty"`, `"Forty"`, ...)

3. **Processing groups of three digits**: The algorithm processes the number in groups of three digits from left to right, adding appropriate scale words (`"Billion"`, `"Million"`, `"Thousand"`) after each group.

4. **Using a helper function `transfer()`**: This function converts numbers less than 1000 into words:
   - If the number is less than 20, it uses the `lt20` array directly
   - If the number is less than 100, it combines the tens place with the ones place
   - If the number is less than 1000, it processes the hundreds place first, then recursively handles the remainder

5. **Main logic**: The algorithm starts with the billions place (1,000,000,000) and works down, extracting each group of three digits, converting them to words using the `transfer()` function, and appending the appropriate scale word.

The final result strips any trailing spaces and returns the complete English representation of the number.

## Intuition

When we think about how we naturally read numbers in English, we notice a pattern. We don't read a large number like 1,234,567 digit by digit. Instead, we group digits and apply scale words based on their position.

The key insight is that English number representation follows a hierarchical structure based on groups of three digits. We say "One Million, Two Hundred Thirty Four Thousand, Five Hundred Sixty Seven" - notice how we're essentially breaking the number into chunks of three digits (1, 234, 567) and applying scale words (Million, Thousand) to each chunk.

This leads us to think about the problem in terms of:

1. **Recurring patterns**: Numbers within each group of three digits (0-999) follow the same naming pattern regardless of their position in the larger number. Whether it's "234" in the thousands place or "567" in the ones place, the way we say "Two Hundred Thirty Four" or "Five Hundred Sixty Seven" remains the same.

2. **Scale separation**: The scales in English (Billion, Million, Thousand) each represent 1000x the previous scale. This means we can process the number by repeatedly dividing by 1000.

3. **Base cases for naming**: We observe that:
   - Numbers 0-19 have unique, non-pattern-based names
   - Numbers 20-99 follow a pattern of `[tens_name] + [ones_name]`
   - Numbers 100-999 follow a pattern of `[hundreds_digit] + "Hundred" + [remainder]`

This recursive structure naturally suggests a solution where we:
- Process the number from the largest scale (billions) down to the smallest
- For each scale, extract the relevant three-digit group
- Convert that three-digit group to words using the same logic regardless of scale
- Append the appropriate scale word
- Move to the next smaller scale

The beauty of this approach is that once we solve how to convert any number less than 1000 to words, we can reuse that logic for every three-digit group in the larger number, just adding the appropriate scale word after each group.

## Solution Approach

The implementation uses a divide-and-conquer strategy with careful handling of English number naming conventions.

**Data Structures Setup:**

First, we define three arrays to store the English words:
- `lt20`: Array containing words for numbers 0-19, since these have unique names that don't follow a pattern
- `tens`: Array for multiples of 10 (20, 30, 40, ..., 90)
- `thousands`: Array containing scale words in descending order: `['Billion', 'Million', 'Thousand', '']`

**Helper Function - `transfer(num)`:**

This recursive function converts any number less than 1000 into its English representation:

1. **Base case**: If `num == 0`, return empty string (to avoid adding extra words for trailing zeros)

2. **Numbers less than 20**: Directly look up in `lt20` array and add a space: `lt20[num] + ' '`

3. **Numbers 20-99**: Combine the tens place with ones place:
   - Get tens word: `tens[num // 10]`
   - Recursively process remainder: `transfer(num % 10)`
   - Return: `tens[num // 10] + ' ' + transfer(num % 10)`

4. **Numbers 100-999**: Process hundreds first, then remainder:
   - Get hundreds digit word: `lt20[num // 100]`
   - Add "Hundred"
   - Recursively process remainder: `transfer(num % 100)`
   - Return: `lt20[num // 100] + ' Hundred ' + transfer(num % 100)`

**Main Algorithm:**

1. **Special case**: If `num == 0`, immediately return `'Zero'`

2. **Initialize variables**:
   - `i = 1000000000` (1 billion - starting with the largest scale)
   - `j = 0` (index for `thousands` array)
   - `res = []` (result array to build the string)

3. **Process each scale group**:
   ```python
   while i > 0:
       if num // i != 0:  # If there's a non-zero group at this scale
           res.append(transfer(num // i))  # Convert the group to words
           res.append(thousands[j])        # Add scale word
           res.append(' ')                  # Add space separator
           num %= i                         # Remove processed digits
       j += 1                              # Move to next scale word
       i //= 1000                          # Move to next smaller scale
   ```

4. **Final processing**:
   - Join all parts: `''.join(res)`
   - Strip trailing spaces: `.strip()`

**Example Walkthrough for `1234567`:**

- First iteration (`i=1000000000`): `1234567 // 1000000000 = 0`, skip
- Second iteration (`i=1000000`): 
  - `1234567 // 1000000 = 1`
  - Add: `transfer(1)` → "One "
  - Add: "Million"
  - Add: " "
  - Update: `num = 234567`
- Third iteration (`i=1000`):
  - `234567 // 1000 = 234`
  - Add: `transfer(234)` → "Two Hundred Thirty Four "
  - Add: "Thousand"
  - Add: " "
  - Update: `num = 567`
- Fourth iteration (`i=1`):
  - `567 // 1 = 567`
  - Add: `transfer(567)` → "Five Hundred Sixty Seven "
  - Add: "" (empty string for ones place)
  - Add: " "

Result after joining and stripping: `"One Million Two Hundred Thirty Four Thousand Five Hundred Sixty Seven"`

## Example Walkthrough

Let's walk through converting the number `12345` to its English representation.

**Initial Setup:**
- Input: `12345`
- We have our arrays ready:
  - `lt20 = ['', 'One', 'Two', ..., 'Nineteen']`
  - `tens = ['', '', 'Twenty', 'Thirty', ..., 'Ninety']`
  - `thousands = ['Billion', 'Million', 'Thousand', '']`

**Main Algorithm Process:**

Starting with `i = 1000000000` (1 billion), `j = 0`, `res = []`

**Iteration 1** (Billions):
- Check: `12345 // 1000000000 = 0`
- Since it's 0, skip this scale
- Update: `j = 1`, `i = 1000000`

**Iteration 2** (Millions):
- Check: `12345 // 1000000 = 0`
- Since it's 0, skip this scale
- Update: `j = 2`, `i = 1000`

**Iteration 3** (Thousands):
- Check: `12345 // 1000 = 12`
- Since it's non-zero, process this group:
  - Call `transfer(12)`:
    - 12 is less than 20
    - Return: `lt20[12] + ' '` = `"Twelve "`
  - Add to result: `res = ["Twelve "]`
  - Add scale word: `res = ["Twelve ", "Thousand"]`
  - Add space: `res = ["Twelve ", "Thousand", " "]`
  - Update number: `num = 12345 % 1000 = 345`
- Update: `j = 3`, `i = 1`

**Iteration 4** (Ones/Hundreds):
- Check: `345 // 1 = 345`
- Since it's non-zero, process this group:
  - Call `transfer(345)`:
    - 345 >= 100, so process hundreds
    - Hundreds digit: `345 // 100 = 3`
    - Build: `lt20[3] + ' Hundred ' = "Three Hundred "`
    - Process remainder: `transfer(345 % 100) = transfer(45)`
      - 45 >= 20 and < 100
      - Tens: `tens[45 // 10] = tens[4] = "Forty"`
      - Ones: `transfer(45 % 10) = transfer(5)`
        - 5 < 20, return `lt20[5] + ' ' = "Five "`
      - Return: `"Forty " + "Five " = "Forty Five "`
    - Complete: `"Three Hundred " + "Forty Five " = "Three Hundred Forty Five "`
  - Add to result: `res = ["Twelve ", "Thousand", " ", "Three Hundred Forty Five "]`
  - Add scale word: `thousands[3] = ""` (empty for ones place)
  - Add space: `res = ["Twelve ", "Thousand", " ", "Three Hundred Forty Five ", "", " "]`
- Update: `i = 0`, loop ends

**Final Processing:**
- Join result: `"Twelve Thousand Three Hundred Forty Five  "`
- Strip trailing spaces: `"Twelve Thousand Three Hundred Forty Five"`

**Output:** `"Twelve Thousand Three Hundred Forty Five"`

The algorithm successfully breaks down 12345 into two groups: "12" (thousands) and "345" (ones), converts each group independently using the `transfer()` function, and combines them with the appropriate scale word "Thousand".

## Python Solution

```python
class Solution:
    def numberToWords(self, num: int) -> str:
        """
        Convert an integer to its English words representation.
      
        Args:
            num: Integer between 0 and 2^31 - 1
          
        Returns:
            String representation of the number in English words
        """
        # Special case for zero
        if num == 0:
            return 'Zero'
      
        # Words for numbers less than 20
        less_than_20 = [
            '',         # 0
            'One',      # 1
            'Two',      # 2
            'Three',    # 3
            'Four',     # 4
            'Five',     # 5
            'Six',      # 6
            'Seven',    # 7
            'Eight',    # 8
            'Nine',     # 9
            'Ten',      # 10
            'Eleven',   # 11
            'Twelve',   # 12
            'Thirteen', # 13
            'Fourteen', # 14
            'Fifteen',  # 15
            'Sixteen',  # 16
            'Seventeen',# 17
            'Eighteen', # 18
            'Nineteen', # 19
        ]
      
        # Words for tens places (10, 20, 30, ..., 90)
        tens_words = [
            '',         # 0
            'Ten',      # 10
            'Twenty',   # 20
            'Thirty',   # 30
            'Forty',    # 40
            'Fifty',    # 50
            'Sixty',    # 60
            'Seventy',  # 70
            'Eighty',   # 80
            'Ninety',   # 90
        ]
      
        # Scale words for thousands, millions, and billions
        scale_words = ['Billion', 'Million', 'Thousand', '']
      
        def convert_hundreds(num: int) -> str:
            """
            Convert a number less than 1000 to words.
          
            Args:
                num: Integer between 0 and 999
              
            Returns:
                String representation with trailing space
            """
            if num == 0:
                return ''
          
            # Handle numbers less than 20
            if num < 20:
                return less_than_20[num] + ' '
          
            # Handle numbers less than 100
            if num < 100:
                return tens_words[num // 10] + ' ' + convert_hundreds(num % 10)
          
            # Handle numbers from 100 to 999
            return less_than_20[num // 100] + ' Hundred ' + convert_hundreds(num % 100)
      
        # Build result by processing billions, millions, thousands, and ones
        result = []
        scale_divisor = 1_000_000_000  # Start with billions
        scale_index = 0
      
        while scale_divisor > 0:
            # Extract the current group (billions, millions, thousands, or ones)
            current_group = num // scale_divisor
          
            if current_group != 0:
                # Convert the group to words
                result.append(convert_hundreds(current_group))
                # Add the scale word (Billion, Million, Thousand, or empty)
                result.append(scale_words[scale_index])
                result.append(' ')
                # Remove the processed digits
                num %= scale_divisor
          
            # Move to the next scale
            scale_index += 1
            scale_divisor //= 1000
      
        # Join all parts and remove trailing whitespace
        return ''.join(result).strip()
```

## Time and Space Complexity

**Time Complexity:** `O(1)`

The algorithm processes a fixed maximum of 4 groups (billions, millions, thousands, and ones), regardless of the input size. For each group:
- The `transfer` function is called with a number less than 1000
- Within `transfer`, there are at most 3 recursive calls (for hundreds, tens, and ones)
- Each operation (division, modulo, array access) takes constant time

Since the input is bounded by a 32-bit integer (maximum value ~2.1 billion), the number of iterations in the while loop is always exactly 4. The recursion depth in `transfer` is at most 3 levels. Therefore, the total number of operations is constant.

**Space Complexity:** `O(1)`

The space usage includes:
- Three fixed-size arrays (`lt20`, `tens`, `thousands`) with a total of 34 string elements
- The `res` list which stores at most 4 groups × 3 components (number word, scale word, space) = 12 elements
- The recursion stack for `transfer` function with maximum depth of 3
- A few integer variables (`i`, `j`, `num`)

All space requirements are bounded by constants that don't scale with the input value, resulting in constant space complexity.

## Common Pitfalls

### 1. **Incorrect Handling of Zero Groups in the Middle**

One of the most common pitfalls is not properly handling zero groups within a number. For example, the number `1000050` should be "One Million Fifty", not "One Million Thousand Fifty" or "One Million Zero Thousand Fifty".

**Problem Code Example:**
```python
# Incorrect approach that might add unnecessary words
if num // scale_divisor >= 0:  # Wrong condition
    result.append(convert_hundreds(num // scale_divisor))
    result.append(scale_words[scale_index])
```

**Solution:**
Always check if the current group is non-zero before adding its words and scale:
```python
if num // scale_divisor != 0:  # Correct: only process non-zero groups
    result.append(convert_hundreds(num // scale_divisor))
    result.append(scale_words[scale_index])
    result.append(' ')
```

### 2. **Improper Space Management**

Another frequent issue is ending up with multiple consecutive spaces or trailing spaces in the final result due to inconsistent space handling in the helper function.

**Problem Scenarios:**
- When `convert_hundreds(0)` returns an empty string but spaces are still added
- When recursively calling for numbers ending in zero (like 20, 100, etc.)

**Example of problematic output:**
- `120` might become "One Hundred  Twenty  " (extra spaces)
- `100` might become "One Hundred  " (trailing spaces from empty remainder)

**Solution:**
1. Make the helper function consistently return strings with trailing spaces for non-zero values
2. Always strip the final result to remove any trailing whitespace
3. Handle the base case of 0 by returning an empty string (no spaces)

```python
def convert_hundreds(num: int) -> str:
    if num == 0:
        return ''  # Return empty string, no spaces
    # ... rest of the logic ensures consistent spacing
  
# In main function:
return ''.join(result).strip()  # Strip removes all trailing spaces
```

### 3. **Off-by-One Errors in Array Indexing**

The `tens_words` array indexing can be tricky because the index needs to map correctly to the tens digit.

**Problem Example:**
```python
# Incorrect: This would give wrong tens words
tens_words = ['Twenty', 'Thirty', 'Forty', ...]  # Missing placeholders
return tens_words[num // 10 - 2] + ...  # Complex offset calculation
```

**Solution:**
Use a properly indexed array with empty strings as placeholders:
```python
tens_words = ['', 'Ten', 'Twenty', 'Thirty', 'Forty', 'Fifty', 'Sixty', 'Seventy', 'Eighty', 'Ninety']
# Now num // 10 directly gives the correct index
return tens_words[num // 10] + ' ' + convert_hundreds(num % 10)
```

### 4. **Forgetting the Special Case of Zero**

The number 0 is a special case that must be handled separately at the beginning of the main function. Without this check, the algorithm would return an empty string.

**Problem:**
```python
# Without special case handling
def numberToWords(self, num: int) -> str:
    # Directly start processing...
    # When num=0, all divisions result in 0, nothing gets added to result
    return ''.join(result).strip()  # Returns empty string!
```

**Solution:**
```python
def numberToWords(self, num: int) -> str:
    if num == 0:
        return 'Zero'  # Handle special case first
    # ... rest of the algorithm
```

### 5. **Integer Division vs Float Division**

Using float division (`/`) instead of integer division (`//`) can cause issues when using the result as an array index.

**Problem:**
```python
current_group = num / scale_divisor  # Returns float
tens_digit = num / 10  # Returns float
less_than_20[num / 100]  # TypeError: list indices must be integers
```

**Solution:**
Always use integer division (`//`) when working with array indices:
```python
current_group = num // scale_divisor
tens_digit = num // 10
less_than_20[num // 100]
```

