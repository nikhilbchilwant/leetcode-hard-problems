# 828. Count Unique Characters of All Substrings of a Given String

> Source: https://algo.monster/liteproblems/828

## Problem Statement

We need to find the sum of unique character counts across all possible substrings of a given string `s`.

First, let's understand what "unique characters" means. A character is considered unique in a string if it appears exactly once in that string. For example:
- In `"LEETCODE"`, the unique characters are `'L'`, `'T'`, `'C'`, `'O'`, `'D'` (they each appear once), while `'E'` appears three times so it's not unique. The count of unique characters is 5.
- In `"AAA"`, there are no unique characters since `'A'` appears three times. The count is 0.
- In `"ABC"`, all characters are unique. The count is 3.

The problem asks us to:
1. Generate all possible substrings of the input string `s`
2. For each substring, count how many characters appear exactly once in that substring
3. Sum up all these counts

For example, if `s = "ABC"`:
- Substrings of length 1: `"A"` (1 unique), `"B"` (1 unique), `"C"` (1 unique)
- Substrings of length 2: `"AB"` (2 unique), `"BC"` (2 unique)  
- Substrings of length 3: `"ABC"` (3 unique)
- Total sum = 1 + 1 + 1 + 2 + 2 + 3 = 10

Note that if a substring appears multiple times, we count each occurrence separately. For instance, if `s = "ABA"`:
- The substring `"A"` appears at positions 0 and 2, so we count it twice
- Each occurrence contributes to the final sum

The challenge is to efficiently calculate this sum without explicitly generating and checking every substring, as that would be computationally expensive for long strings.

## Intuition

Instead of looking at every substring and counting unique characters in each one, let's flip our perspective: for each character in the string, we can ask "in how many substrings does this character appear exactly once?"

Consider a character at position `i` in the string. For this character to be unique in a substring, that substring must include position `i` but cannot include any other position where the same character appears.

Let's say character `'A'` appears at positions `[2, 5, 8]` in our string. For the `'A'` at position 5 to be unique in a substring:
- The substring cannot extend left beyond position 2 (otherwise it would include another `'A'`)
- The substring cannot extend right beyond position 8 (otherwise it would include another `'A'`)
- The substring must include position 5

So the valid substrings are those that:
- Start anywhere from position `3` to position `5` (positions after the previous `'A'` up to and including position 5)
- End anywhere from position `5` to position `7` (from position 5 up to but not including the next `'A'`)

The number of such substrings is `(5 - 2) × (8 - 5) = 3 × 3 = 9`.

This gives us the key insight: for each character at position `p`, if the previous occurrence of the same character is at position `l` and the next occurrence is at position `r`, then the character at `p` contributes `(p - l) × (r - p)` to the final answer.

By calculating this contribution for every character position in the string and summing them up, we get the total count of unique characters across all substrings. This approach is much more efficient than examining every substring individually.

To handle edge cases, we can treat the boundaries before the first occurrence as position `-1` and after the last occurrence as position `len(s)`, ensuring our formula works consistently for all characters.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement the contribution-based approach using a hash table to track character positions:

1. **Build Position Map**: First, we create a dictionary `d` (named `char_positions` in the Python solution, with `v` corresponding to `positions_with_bounds`) where each key is a character and each value is a list of indices where that character appears in the string. We iterate through the string once, recording each character's position:
   ```python
   d = defaultdict(list)
   for i, c in enumerate(s):
       d[c].append(i)
   ```
   For example, if `s = "LEETCODE"`, then `d['E'] = [1, 2, 7]`.

2. **Calculate Contributions**: For each character that appears in the string, we process its list of positions:
   ```python
   for v in d.values():
       v = [-1] + v + [len(s)]
   ```
   We pad the position list with `-1` at the beginning and `len(s)` at the end. This represents the boundaries - there's no occurrence of the character before index 0 or after the last index.

3. **Apply the Formula**: For each position `i` in the padded list (excluding the boundary values), we calculate how many substrings have the character at position `v[i]` as unique:
   ```python
   for i in range(1, len(v) - 1):
       ans += (v[i] - v[i - 1]) * (v[i + 1] - v[i])
   ```
   - `v[i] - v[i - 1]` gives the number of possible starting positions for substrings (from right after the previous occurrence up to the current position)
   - `v[i + 1] - v[i]` gives the number of possible ending positions for substrings (from the current position up to right before the next occurrence)
   - Their product gives the total number of substrings where this character appears exactly once

4. **Sum All Contributions**: We accumulate the contributions from all character positions to get the final answer.

**Example Walkthrough**: 
If `s = "ABA"`:
- `d['A'] = [0, 2]`, `d['B'] = [1]`
- For `'A'`: padded list = `[-1, 0, 2, 3]`
  - Position 0: `(0 - (-1)) × (2 - 0) = 1 × 2 = 2`
  - Position 2: `(2 - 0) × (3 - 2) = 2 × 1 = 2`
- For `'B'`: padded list = `[-1, 1, 3]`
  - Position 1: `(1 - (-1)) × (3 - 1) = 2 × 2 = 4`
- Total = `2 + 2 + 4 = 8`

The time complexity is `O(n)` where `n` is the length of the string, and space complexity is `O(n)` for storing the position lists.

## Example Walkthrough

Let's walk through the solution with `s = "ABCB"`:

**Step 1: Build Position Map**
We track where each character appears:
- `'A'` appears at position `[0]`
- `'B'` appears at positions `[1, 3]`
- `'C'` appears at position `[2]`

So our dictionary `d` becomes:
- `d['A'] = [0]`
- `d['B'] = [1, 3]`
- `d['C'] = [2]`

**Step 2: Pad Position Lists with Boundaries**
We add `-1` before and `len(s) = 4` after each position list:
- `'A'`: `[-1, 0, 4]`
- `'B'`: `[-1, 1, 3, 4]`
- `'C'`: `[-1, 2, 4]`

**Step 3: Calculate Contributions for Each Character Position**

For `'A'` at position 0:
- Previous occurrence boundary: `-1`
- Current position: `0`
- Next occurrence boundary: `4`
- Contribution: `(0 - (-1)) × (4 - 0) = 1 × 4 = 4`
- This means the `'A'` at position 0 appears as unique in 4 substrings: `"A"`, `"AB"`, `"ABC"`, `"ABCB"`

For `'B'` at position 1:
- Previous occurrence boundary: `-1`
- Current position: `1`
- Next occurrence: `3`
- Contribution: `(1 - (-1)) × (3 - 1) = 2 × 2 = 4`
- This means the `'B'` at position 1 appears as unique in 4 substrings: `"B"`, `"BC"`, `"AB"`, `"ABC"`

For `'B'` at position 3:
- Previous occurrence: `1`
- Current position: `3`
- Next occurrence boundary: `4`
- Contribution: `(3 - 1) × (4 - 3) = 2 × 1 = 2`
- This means the `'B'` at position 3 appears as unique in 2 substrings: `"CB"`, `"B"`

For `'C'` at position 2:
- Previous occurrence boundary: `-1`
- Current position: `2`
- Next occurrence boundary: `4`
- Contribution: `(2 - (-1)) × (4 - 2) = 3 × 2 = 6`
- This means the `'C'` at position 2 appears as unique in 6 substrings: `"C"`, `"BC"`, `"CB"`, `"ABC"`, `"BCB"`, `"ABCB"`

**Step 4: Sum All Contributions**
Total = `4 + 4 + 2 + 6 = 16`

We can verify this by manually checking all substrings:
- Length 1: `"A"` (1), `"B"` (1), `"C"` (1), `"B"` (1) = 4 total
- Length 2: `"AB"` (2), `"BC"` (2), `"CB"` (2) = 6 total  
- Length 3: `"ABC"` (3), `"BCB"` (1) = 4 total
- Length 4: `"ABCB"` (2) = 2 total
- Sum = 4 + 6 + 4 + 2 = 16 ✓

## Python Solution

```python
from collections import defaultdict
from typing import List

class Solution:
    def uniqueLetterString(self, s: str) -> int:
        # Dictionary to store all positions where each character appears
        char_positions = defaultdict(list)
      
        # Record the index position for each character in the string
        for index, char in enumerate(s):
            char_positions[char].append(index)
      
        total_count = 0
      
        # For each unique character, calculate its contribution to all substrings
        for positions in char_positions.values():
            # Add boundary markers: -1 at start and len(s) at end
            # This helps calculate the range where each occurrence is unique
            positions_with_bounds = [-1] + positions + [len(s)]
          
            # For each occurrence of the character, calculate how many substrings
            # contain this occurrence as the unique instance of that character
            for i in range(1, len(positions_with_bounds) - 1):
                # Left distance: from previous occurrence (or start boundary)
                left_distance = positions_with_bounds[i] - positions_with_bounds[i - 1]
                # Right distance: to next occurrence (or end boundary)
                right_distance = positions_with_bounds[i + 1] - positions_with_bounds[i]
                # Total substrings where this character occurrence is unique
                total_count += left_distance * right_distance
      
        return total_count
```

## Time and Space Complexity

**Time Complexity:** `O(n)`, where `n` is the length of the string `s`.

The algorithm consists of two main parts:
1. First loop: Iterates through the string once to build the dictionary, which takes `O(n)` time.
2. Second loop: Iterates through each unique character's position list. Since each position in the string is processed exactly once across all characters (each index appears in exactly one character's list), the total work done is `O(n)`.

Even though there's a nested loop structure, each index of the string contributes to exactly one calculation, making the overall time complexity linear.

**Space Complexity:** `O(n)`, where `n` is the length of the string `s`.

The space is used for:
1. Dictionary `char_positions`: Stores position lists for each unique character. In the worst case (all characters are the same), we store `n` positions in one list. In the best case (all characters are different), we store `n` positions distributed across multiple lists. Either way, the total space for all positions is `O(n)`.
2. Temporary list `positions_with_bounds`: For each character, we create a new list with at most `n + 2` elements (original positions plus two sentinels). However, this doesn't add to the overall complexity as it's still bounded by `O(n)`.

Therefore, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Boundary Handling**
One of the most common mistakes is using the first and last indices of the string (`0` and `len(s) - 1`) as the boundaries instead of `-1` and `len(s)`. The boundaries stand for "no occurrence before the string starts" and "no occurrence after the string ends", so they must lie one step outside the string.

**Incorrect approach:**
```python
for positions in char_positions.values():
    # Wrong boundaries: they sit on real indices of the string
    positions_with_bounds = [0] + positions + [len(s) - 1]
    for i in range(1, len(positions_with_bounds) - 1):
        left_distance = positions_with_bounds[i] - positions_with_bounds[i - 1]
        right_distance = positions_with_bounds[i + 1] - positions_with_bounds[i]
        total_count += left_distance * right_distance
```

For `s = "ABC"` this returns 1 instead of 10: the `'A'` at index 0 gets a left distance of 0, and the `'C'` at index 2 gets a right distance of 0, so neither contributes anything.

**Solution:** Use `-1` and `len(s)` as the boundaries:
```python
positions_with_bounds = [-1] + positions + [len(s)]
```

### 2. **Off-by-One Errors in Distance Calculation**
Another pitfall is making off-by-one errors when calculating the left and right distances. Some might incorrectly add or subtract 1 from the distances.

**Incorrect approach:**
```python
# Wrong: Adding unnecessary +1 or -1
left_distance = positions_with_bounds[i] - positions_with_bounds[i - 1] - 1
right_distance = positions_with_bounds[i + 1] - positions_with_bounds[i] - 1
```

**Solution:** The distances should be calculated directly without any adjustments:
```python
left_distance = positions_with_bounds[i] - positions_with_bounds[i - 1]
right_distance = positions_with_bounds[i + 1] - positions_with_bounds[i]
```

### 3. **Using Regular Dictionary Instead of defaultdict**
Using a regular dictionary requires checking if a key exists before appending. The version below is correct, but it is longer than it needs to be.

**Verbose approach:**
```python
char_positions = {}
for index, char in enumerate(s):
    if char not in char_positions:
        char_positions[char] = []
    char_positions[char].append(index)
```

**Solution:** Use `defaultdict(list)` for cleaner code:
```python
char_positions = defaultdict(list)
for index, char in enumerate(s):
    char_positions[char].append(index)
```

### 4. **Modifying the Original Position List**
A subtle but dangerous pitfall is modifying the original position list when adding boundaries, which could cause issues if the list is used elsewhere.

**Incorrect approach:**
```python
for positions in char_positions.values():
    positions.insert(0, -1)  # Modifies original list
    positions.append(len(s))  # Modifies original list
    # Process positions...
```

**Solution:** Create a new list with boundaries:
```python
positions_with_bounds = [-1] + positions + [len(s)]
```

### 5. **Misunderstanding the Problem - Counting Characters Instead of Contributions**
Some might try to count unique characters in each substring explicitly, leading to an O(n³) solution that times out for large inputs.

**Incorrect approach:**
```python
total = 0
for i in range(len(s)):
    for j in range(i, len(s)):
        substring = s[i:j+1]
        char_count = {}
        for char in substring:
            char_count[char] = char_count.get(char, 0) + 1
        unique_count = sum(1 for count in char_count.values() if count == 1)
        total += unique_count
```

**Solution:** Use the contribution-based approach that calculates how many substrings contain each character position as unique, achieving O(n) time complexity.

