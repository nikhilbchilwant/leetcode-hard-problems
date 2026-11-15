# 726. Number of Atoms

> Source: https://algo.monster/liteproblems/726

## Problem Statement

This problem asks you to parse a chemical formula string and return the count of each atom in alphabetical order.

The parsing rules are:
- Each element starts with an uppercase letter, followed by zero or more lowercase letters (e.g., `H`, `Ca`, `Mg`)
- A number after an element indicates its count. If no number follows, the count is 1 (e.g., `H2` means 2 hydrogen atoms, `O` means 1 oxygen atom)
- Multiple elements can be concatenated (e.g., `H2O` contains 2 H atoms and 1 O atom)
- Parentheses can group elements together, and a number after the closing parenthesis multiplies all elements inside (e.g., `(OH)2` means 2 O atoms and 2 H atoms)
- Parentheses can be nested

The solution processes the formula from right to left, using:
- A `multiplier` variable to track the current multiplication factor from parentheses
- A [stack](/problems/stack_intro) to save multipliers when entering nested parentheses (encountering `)` from right to left)
- A frequency counter to build multi-digit numbers
- A HashMap to accumulate the total count for each element

The algorithm:
1. Traverse the formula string from right to left
2. When encountering lowercase letters, build the complete element name backwards
3. When encountering an uppercase letter (alone or after building lowercase letters), it marks the start of an element - add it to the map with count = `freq * multiplier`
4. When encountering digits, build the complete number (which applies to the element/group to its left)
5. When encountering `)`, push the current multiplier to [stack](/problems/stack_intro) and update multiplier by multiplying with the frequency
6. When encountering `(`, restore the previous multiplier from stack

Finally, sort the element names alphabetically and build the output string, appending counts only when greater than 1.

## Intuition

The key insight is that parsing from right to left simplifies handling parentheses and multipliers. When we parse left to right, we don't know what multiplier applies to an element until we've seen all the closing parentheses and numbers that follow it. But parsing right to left, we always know the current multiplier before we encounter any element.

Consider `Ca(OH)2` - if parsing left to right, when we see `O` and `H`, we don't yet know they'll be multiplied by 2. But parsing right to left, we see the `2` first, then the `)`, so we know everything inside the parentheses should be multiplied by 2 before we even reach `O` and `H`.

The [stack](/problems/stack_intro) becomes necessary for nested parentheses. Each time we encounter a `)` (moving right to left), we're entering a new group that has its own multiplier. We need to save the current multiplier so we can restore it when we exit this group (when we hit the matching `(`). Think of it like diving into nested layers - we need to remember what multiplication factor we had at each level.

The formula `Mg(OH)2` when parsed right to left becomes:
- See `2`: this is the frequency for the next group
- See `)`: entering a group, save current multiplier (1), new multiplier = 1 * 2 = 2
- See `H`: add H with count 1 * 2 = 2
- See `O`: add O with count 1 * 2 = 2  
- See `(`: exiting the group, restore multiplier to 1
- See `g`, then `M`: add Mg with count 1 * 1 = 1

This right-to-left approach with a multiplier [stack](/problems/stack_intro) elegantly handles all nesting levels without needing recursive parsing or complex lookahead logic.

**Learn more about [Stack](/problems/stack_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation uses a dictionary to store element counts, a [stack](/problems/stack_intro) for managing multipliers, and processes the formula string from right to left.

**Data Structures:**
- `counts`: a `defaultdict(int)` that stores the total count of each element
- `stack`: a list that saves the outer multiplier each time the scan enters a parenthesized group
- `multiplier`: the product of all group multipliers that apply at the current position (initially 1)
- `freq`: the number most recently read; `0` means no number was written

**Algorithm Steps:**

1. **Scan from right to left** with an index `i` that starts at `len(formula) - 1`.

2. **Handle digits** (`char.isdigit()`):
   - Read the whole number from right to left: `freq += int(formula[i]) * place_value`, with `place_value` multiplied by 10 after each digit
   - This builds `123` as `3 + 10*2 + 100*1`
   - The number applies to the element or group immediately to its left, which the scan reaches next

3. **Handle letters** (`char.isalpha()`):
   - Move `i` left over any lowercase letters until it reaches the uppercase letter that starts the name
   - Take the name as `formula[i:end + 1]` (for example `Mg` or `H`)
   - Add `max(freq, 1) * multiplier` to `counts[element]`
   - Reset `freq = 0`

4. **Handle closing parenthesis** `')'` (entering a group, since we scan from the right):
   - Save the current multiplier: `stack.append(multiplier)`
   - Update the multiplier for the group: `multiplier *= max(freq, 1)`
   - Reset `freq = 0`

5. **Handle opening parenthesis** `'('` (leaving the group):
   - Restore the outer multiplier: `multiplier = stack.pop()`

6. **Generate output**:
   - Visit the element names in sorted order
   - Append each name, and append its count only if the count is greater than 1

The `max(freq, 1)` expression treats a missing number (`freq = 0`) as 1. This handles cases like `H2O`, where `O` has no explicit count, and `(OH)` with no number after the group.

## Example Walkthrough

Let's walk through parsing the formula `K4(ON(SO3)2)2` step by step, processing from right to left.

**Initial state:**
- `multiplier = 1`
- `freq = 0` 
- `stack = []` (empty)
- `counts = {}` (empty)

**Processing right to left:**

1. **Index 12: '2'** - It's a digit
   - `freq = 2`

2. **Index 11: ')'** - Closing parenthesis
   - Push current multiplier to stack: `stack = [1]`
   - Update multiplier: `multiplier = 1 * 2 = 2`
   - Reset `freq = 0`

3. **Index 10: '2'** - It's a digit
   - `freq = 2`

4. **Index 9: ')'** - Closing parenthesis
   - Push current multiplier to stack: `stack = [1, 2]`
   - Update multiplier: `multiplier = 2 * 2 = 4`
   - Reset `freq = 0`

5. **Index 8: '3'** - It's a digit
   - `freq = 3`

6. **Index 7: 'O'** - Uppercase letter
   - Add to counts: `O` with count `3 * 4 = 12`
   - Reset `freq = 0`
   - `counts = {O: 12}`

7. **Index 6: 'S'** - Uppercase letter
   - Add to counts: `S` with count `1 * 4 = 4` (freq was 0, so use 1)
   - Reset `freq = 0`
   - `counts = {O: 12, S: 4}`

8. **Index 5: '('** - Opening parenthesis
   - Restore multiplier from stack: `multiplier = 2`
   - `stack = [1]`

9. **Index 4: 'N'** - Uppercase letter
   - Add to counts: `N` with count `1 * 2 = 2`
   - Reset `freq = 0`
   - `counts = {O: 12, S: 4, N: 2}`

10. **Index 3: 'O'** - Uppercase letter
    - Add to existing O count: `O` gets `1 * 2 = 2` more
    - `counts = {O: 14, S: 4, N: 2}`

11. **Index 2: '('** - Opening parenthesis
    - Restore multiplier from stack: `multiplier = 1`
    - `stack = []`

12. **Index 1: '4'** - It's a digit
    - `freq = 4`

13. **Index 0: 'K'** - Uppercase letter
    - Add to counts: `K` with count `4 * 1 = 4`
    - Reset `freq = 0`
    - `counts = {O: 14, S: 4, N: 2, K: 4}`

**Final step - Generate output:**
- Sort keys alphabetically: `[K, N, O, S]`
- Build result string:
  - K has count 4, append "K4"
  - N has count 2, append "N2"
  - O has count 14, append "O14"
  - S has count 4, append "S4"
- Result: `"K4N2O14S4"`

The key insight is how the multiplier changes as we enter (`)`) and exit (`(`) parentheses groups, always maintaining the correct multiplication factor for each element we encounter.

## Python Solution

```python
from collections import defaultdict


class Solution:
    def countOfAtoms(self, formula: str) -> str:
        """
        Counts atoms in a chemical formula and returns them in alphabetical order.
        The formula is scanned from right to left, so the multiplier for a group
        is always known before the elements inside the group are reached.

        Args:
            formula: Chemical formula string (e.g., "H2O", "Mg(OH)2")

        Returns:
            Formatted string with atom counts (e.g., "H2O", "H2MgO2")
        """
        # Total count for each element name
        counts = defaultdict(int)
        # Multipliers saved when entering a group (seeing ')' from the right)
        stack = []
        # Product of all group multipliers that apply at the current position
        multiplier = 1
        # Number most recently read (0 means no number was written)
        freq = 0

        i = len(formula) - 1
        while i >= 0:
            char = formula[i]

            if char.isdigit():
                # Read the whole number from right to left
                freq = 0
                place_value = 1
                while i >= 0 and formula[i].isdigit():
                    freq += int(formula[i]) * place_value
                    place_value *= 10
                    i -= 1
                continue

            if char.isalpha():
                # Walk left over lowercase letters to the uppercase letter
                end = i
                while formula[i].islower():
                    i -= 1
                element = formula[i:end + 1]
                # A missing count means 1
                counts[element] += max(freq, 1) * multiplier
                freq = 0
            elif char == ')':
                # Entering a group: save the outer multiplier, apply the group's count
                stack.append(multiplier)
                multiplier *= max(freq, 1)
                freq = 0
            else:
                # char == '(': leaving the group, restore the outer multiplier
                multiplier = stack.pop()

            i -= 1

        # Sort element names and append counts greater than 1
        result = []
        for element in sorted(counts):
            result.append(element)
            if counts[element] > 1:
                result.append(str(counts[element]))

        return ''.join(result)
```

## Time and Space Complexity

**Time Complexity:** `O(n + k log k)` where `n` is the length of the input formula string and `k` is the number of unique atoms.

- The main loop traverses the formula string once from right to left: `O(n)`
- Within the loop:
  - Each letter and digit is visited once while reading element names and numbers
  - Slicing an element name and hashing it costs time proportional to the name length, which sums to `O(n)` over the whole formula
  - Dictionary updates are `O(1)` on average, and stack pushes and pops are `O(1)`
- After parsing, sorting the keys: `O(k log k)` comparisons, where `k` is the number of unique atoms
- Building the final string: proportional to the output length

Overall: `O(n)` for parsing + `O(k log k)` for sorting = `O(n + k log k)`

**Space Complexity:** `O(n + k)`

- Dictionary `counts` storing atoms and their counts: `O(k)` entries, whose names have total length at most `n`
- Multiplier `stack`: `O(n)` in the worst case, one entry per level of nested parentheses
- Sorted list of keys and the `result` list for the output: `O(k)` entries plus the output characters

Overall: `O(n + k)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Multi-Digit Numbers**
When parsing numbers that follow elements or closing parentheses, a common mistake is only reading a single digit instead of the complete multi-digit number. For example, in `Ca(OH)12`, the multiplier is 12, not just 1.

**Pitfall Example:**
```python
# Wrong: Only reads one digit
if formula[i].isdigit():
    count = int(formula[i])
```

**Solution:**
```python
# Correct: Reads all consecutive digits
count_str = ''
while i < len(formula) and formula[i].isdigit():
    count_str += formula[i]
    i += 1
count = int(count_str) if count_str else 1
```

### 2. **Mishandling Nested Parentheses**
A frequent error is not properly tracking multiplication factors when parentheses are nested. Each level of nesting requires maintaining its own multiplier, and these multipliers must be combined correctly.

**Pitfall Example:**
```python
# Wrong: Doesn't properly handle nested multipliers
if char == '(':
    # Process without maintaining multiplier stack
    multiplier = get_number_after_closing()
```

**Solution:**
```python
# Correct: Use a stack to maintain multipliers at each nesting level
if char == ')':  # Processing right-to-left
    stack.append(current_multiplier)
    current_multiplier *= get_following_number()
elif char == '(':  # Processing right-to-left
    current_multiplier = stack.pop()
```

### 3. **Element Name Parsing Errors**
Elements can have lowercase letters (like Ca, Mg), and failing to capture the complete element name is a common mistake. Reading only the uppercase letter would incorrectly parse "Ca" as "C" with "a" being treated separately.

**Pitfall Example:**
```python
# Wrong: Only captures uppercase letter
if char.isupper():
    element = char
    add_to_map(element, count)
```

**Solution:**
```python
# Correct: Captures uppercase followed by all lowercase letters
if char.isupper():
    element = char
    j = i + 1
    while j < len(formula) and formula[j].islower():
        element += formula[j]
        j += 1
    add_to_map(element, count)
```

### 4. **Incorrect Order of Operations When Processing Right-to-Left**
When processing the formula from right to left, the order of building numbers and element names must be reversed. Numbers that appear before elements (when reading left-to-right) must be accumulated correctly.

**Pitfall Example:**
```python
# Wrong: Building number incorrectly when going right-to-left
freq = 0
for digit in reversed_digits:
    freq = freq * 10 + int(digit)  # Wrong order
```

**Solution:**
```python
# Correct: Building number with proper positional values
freq = 0
power = 1
for digit in digits_right_to_left:
    freq += int(digit) * power
    power *= 10
```

### 5. **Forgetting Default Count of 1**
When an element or group has no explicit number, its count should default to 1. Missing this leads to incorrect atom counts.

**Pitfall Example:**
```python
# Wrong: Doesn't handle missing count
count = freq  # freq could be 0 if no number was found
```

**Solution:**
```python
# Correct: Uses 1 as default when no number is specified
count = freq if freq > 0 else 1
# Or more concisely:
count = max(freq, 1)
```

