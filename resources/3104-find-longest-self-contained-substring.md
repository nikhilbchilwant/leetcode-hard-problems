# 3104. Find Longest Self-Contained Substring

> Source: https://algo.monster/liteproblems/3104

## Problem Statement

Given a string `s`, you need to find the length of the longest **self-contained** substring.

A substring `t` of string `s` is called **self-contained** if:
1. `t != s` (the substring cannot be the entire string)
2. For every character in `t`, that character does not appear anywhere else in `s` outside of `t`

In other words, a self-contained substring is a portion of the string where all occurrences of every character within that substring are completely contained within the substring itself - none of those characters appear in the rest of the string.

For example:
- If `s = "abcabc"`, the substring `"abc"` (from index 0 to 2) is self-contained because the characters 'a', 'b', and 'c' at positions 0, 1, 2 don't appear anywhere after index 2 in the remaining string.
- If `s = "abcab"`, there is no self-contained substring because every character appears both inside and outside any potential substring.

Return the length of the longest self-contained substring if one exists, otherwise return -1.

## Intuition

The key observation is that if a substring is self-contained, it must include **all occurrences** of every character that appears in it. This means if character 'a' appears in our substring, we must include every 'a' in the entire string for the substring to be self-contained.

This leads to an important insight: a self-contained substring can only start at a position where a character appears for the **first time** in the string. Why? Because if we start at the second or later occurrence of a character, we've already left out at least one occurrence of that character, violating the self-contained property.

Consider the string `"abcabc"`. If we want a substring containing 'a', we must start at index 0 (first 'a') and extend at least to index 3 (last 'a'). Starting at any other position would leave some 'a' outside our substring.

The approach then becomes:
1. For each character, record where it first appears and where it last appears
2. Try starting a substring at each character's first occurrence position
3. As we extend the substring, keep track of the rightmost position we must reach to include all occurrences of characters we've seen so far
4. If we encounter a character whose first occurrence is before our starting position, this substring cannot be self-contained
5. If we successfully reach a point where we've included all necessary occurrences and haven't used the entire string, we have a valid self-contained substring

This greedy expansion approach works because once we commit to including a character, we must include all its occurrences, which might force us to include more characters, creating a cascading effect until we either find a valid boundary or determine the substring cannot be self-contained.

**Learn more about [Binary Search](/problems/binary_search_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

First, we create two hash tables (or dictionaries) `first` and `last` to record the first and last occurrence positions of each character in the string:

```python
first, last = {}, {}
for i, c in enumerate(s):
    if c not in first:
        first[c] = i
    last[c] = i
```

Next, we enumerate each character `c` as a potential starting point for our self-contained substring. We only consider positions where characters appear for the first time (stored in `first`):

```python
for c, i in first.items():
    mx = last[c]  # Initialize mx with the last occurrence of character c
```

Here, `i` is the starting position of our potential substring, and `mx` tracks the rightmost position we must reach to include all occurrences of characters we've encountered.

For each starting position `i`, we traverse from position `i` to the end of the string:

```python
for j in range(i, n):
    a, b = first[s[j]], last[s[j]]
```

At each position `j`, we get:
- `a`: the first occurrence position of character `s[j]`
- `b`: the last occurrence position of character `s[j]`

If `a < i`, it means character `s[j]` has already appeared before our starting position `i`. This violates the self-contained property, so we break out of the inner loop:

```python
if a < i:
    break
```

Otherwise, we update `mx` to ensure we include all occurrences of `s[j]`:

```python
mx = max(mx, b)
```

If `mx == j`, it means we've reached a position where we've included all necessary occurrences of all characters seen so far. If additionally `j - i + 1 < n` (ensuring the substring is not the entire string), we have found a valid self-contained substring:

```python
if mx == j and j - i + 1 < n:
    ans = max(ans, j - i + 1)
```

The algorithm continues this process for all possible starting positions and returns the maximum length found, or -1 if no valid self-contained substring exists.

The time complexity is `O(n²)` where `n` is the length of the string, as we potentially examine each substring starting from each character's first occurrence. The space complexity is `O(k)` where `k` is the number of unique characters in the string.

## Example Walkthrough

Let's walk through the solution with the string `s = "abcabc"`.

**Step 1: Build first and last occurrence maps**
- Iterate through the string to record positions:
  - 'a': first = 0, last = 3
  - 'b': first = 1, last = 4
  - 'c': first = 2, last = 5

**Step 2: Try each character's first occurrence as a starting point**

*Starting from 'a' (position 0):*
- Initialize: `i = 0`, `mx = 3` (last position of 'a')
- Position 0: character 'a'
  - First occurrence of 'a' = 0 (not < 0, so continue)
  - Update `mx = max(3, 3) = 3`
  - Check: `mx (3) ≠ j (0)`, continue
- Position 1: character 'b'
  - First occurrence of 'b' = 1 (not < 0, so continue)
  - Update `mx = max(3, 4) = 4`
  - Check: `mx (4) ≠ j (1)`, continue
- Position 2: character 'c'
  - First occurrence of 'c' = 2 (not < 0, so continue)
  - Update `mx = max(4, 5) = 5`
  - Check: `mx (5) ≠ j (2)`, continue
- Position 3: character 'a'
  - First occurrence of 'a' = 0 (not < 0, so continue)
  - Update `mx = max(5, 3) = 5`
  - Check: `mx (5) ≠ j (3)`, continue
- Position 4: character 'b'
  - First occurrence of 'b' = 1 (not < 0, so continue)
  - Update `mx = max(5, 4) = 5`
  - Check: `mx (5) ≠ j (4)`, continue
- Position 5: character 'c'
  - First occurrence of 'c' = 2 (not < 0, so continue)
  - Update `mx = max(5, 5) = 5`
  - Check: `mx (5) = j (5)` ✓ and `5 - 0 + 1 = 6 = n`, not valid (equals entire string)

*Starting from 'b' (position 1):*
- Initialize: `i = 1`, `mx = 4` (last position of 'b')
- Position 1: character 'b'
  - First occurrence of 'b' = 1 (not < 1, so continue)
  - Update `mx = max(4, 4) = 4`
  - Check: `mx (4) ≠ j (1)`, continue
- Position 2: character 'c'
  - First occurrence of 'c' = 2 (not < 1, so continue)
  - Update `mx = max(4, 5) = 5`
  - Check: `mx (5) ≠ j (2)`, continue
- Position 3: character 'a'
  - First occurrence of 'a' = 0 (0 < 1) ✗ Break!
  - Character 'a' appears before our start position, invalid

*Starting from 'c' (position 2):*
- Initialize: `i = 2`, `mx = 5` (last position of 'c')
- Position 2: character 'c'
  - First occurrence of 'c' = 2 (not < 2, so continue)
  - Update `mx = max(5, 5) = 5`
  - Check: `mx (5) ≠ j (2)`, continue
- Position 3: character 'a'
  - First occurrence of 'a' = 0 (0 < 2) ✗ Break!
  - Character 'a' appears before our start position, invalid

**Step 3: Return result**
No valid self-contained substring was found, so return -1.

---

Let's try another example where a valid substring exists: `s = "abcdefabc"`

**Step 1: Build maps**
- 'a': first = 0, last = 6
- 'b': first = 1, last = 7
- 'c': first = 2, last = 8
- 'd': first = 3, last = 3
- 'e': first = 4, last = 4
- 'f': first = 5, last = 5

**Step 2: Try starting from 'd' (position 3):**
- Initialize: `i = 3`, `mx = 3`
- Position 3: character 'd'
  - First occurrence = 3 (not < 3, continue)
  - Update `mx = max(3, 3) = 3`
  - Check: `mx (3) = j (3)` ✓ and `3 - 3 + 1 = 1 < 9` ✓
  - Found valid substring "d" with length 1

**Continuing to check 'e' (position 4):**
- Similar process yields "e" with length 1

**Continuing to check 'f' (position 5):**
- Similar process yields "f" with length 1

The longest self-contained substring has length 1 (any of "d", "e", or "f").

## Python Solution

```python
class Solution:
    def maxSubstringLength(self, s: str) -> int:
        # Build dictionaries to track first and last occurrence of each character
        first_occurrence = {}
        last_occurrence = {}

        # Populate the occurrence dictionaries
        for index, char in enumerate(s):
            if char not in first_occurrence:
                first_occurrence[char] = index
            last_occurrence[char] = index

        # Initialize result and get string length
        max_length = -1
        string_length = len(s)

        # Try each character as a potential starting point for a valid substring
        for char, start_index in first_occurrence.items():
            # Track the maximum index we need to include to satisfy all character occurrences
            required_end = last_occurrence[char]

            # Expand from start_index and check if we can form a valid substring
            for current_index in range(start_index, string_length):
                current_char = s[current_index]
                char_first = first_occurrence[current_char]
                char_last = last_occurrence[current_char]

                # If current character's first occurrence is before our start, invalid substring
                if char_first < start_index:
                    break

                # Update the required end to include all occurrences of current character
                required_end = max(required_end, char_last)

                # Check if we've reached a valid substring endpoint
                # Valid if: we've covered all required occurrences AND it's not the entire string
                if required_end == current_index and current_index - start_index + 1 < string_length:
                    max_length = max(max_length, current_index - start_index + 1)

        return max_length
```

## Time and Space Complexity

**Time Complexity:** `O(n × |Σ|)`

The algorithm consists of two main parts:
1. The first loop iterates through the string once to build the `first` and `last` dictionaries, taking `O(n)` time.
2. The second nested loop structure:
   - The outer loop iterates through each unique character in the `first` dictionary, which can be at most `|Σ|` characters
   - For each character, the inner loop can iterate up to `n` positions in the worst case
   - Therefore, this part takes `O(|Σ| × n)` time

The overall time complexity is `O(n) + O(|Σ| × n) = O(n × |Σ|)`, where `n` is the length of string `s` and `|Σ|` represents the size of the character set (26 for lowercase letters).

**Space Complexity:** `O(|Σ|)`

The algorithm uses:
- Two dictionaries `first` and `last` to store the first and last occurrence indices of each character
- Each dictionary can store at most `|Σ|` key-value pairs (one for each unique character in the string)
- A few additional variables (`ans`, `n`, `mx`, etc.) that use constant space

Therefore, the space complexity is `O(|Σ|)`, where `|Σ| = 26` for lowercase English letters.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Breaking Early When Character Appears Before Start Position**

A critical pitfall is forgetting to break the inner loop when encountering a character that has already appeared before the current starting position. This would lead to incorrect results by considering invalid substrings.

**Incorrect Implementation:**
```python
for current_index in range(start_index, string_length):
    current_char = s[current_index]
    char_first = first_occurrence[current_char]
    char_last = last_occurrence[current_char]

    # MISSING: Should break if char_first < start_index

    required_end = max(required_end, char_last)
    # ... rest of code
```

**Why This Fails:**
Without the break condition, the algorithm would continue processing even when it encounters characters that violate the self-contained property. For example, with `s = "abcabc"`, when starting from index 3 (second 'a'), it would incorrectly consider the substring "abc" at indices 3-5 as valid, even though 'a', 'b', 'c' also appear at indices 0-2.

**Solution:**
Always include the break condition:
```python
if char_first < start_index:
    break
```

### 2. **Incorrect Validation of Substring Length**

Another common mistake is using `<=` instead of `<` when checking if the substring length is less than the total string length.

**Incorrect Implementation:**
```python
if required_end == current_index and current_index - start_index + 1 <= string_length:
    max_length = max(max_length, current_index - start_index + 1)
```

**Why This Fails:**
Using `<=` would allow the entire string to be considered as a valid self-contained substring, which violates the constraint that `t != s`. This would return the length of the entire string instead of -1 for strings like `s = "aaa"` where no proper self-contained substring exists.

**Solution:**
Use strict inequality:
```python
if required_end == current_index and current_index - start_index + 1 < string_length:
    max_length = max(max_length, current_index - start_index + 1)
```

### 3. **Not Initializing `required_end` Correctly**

Forgetting to initialize `required_end` with the last occurrence of the starting character can cause the algorithm to miss valid substrings.

**Incorrect Implementation:**
```python
for char, start_index in first_occurrence.items():
    required_end = start_index  # Wrong: should be last_occurrence[char]
    # ... rest of code
```

**Why This Fails:**
If a character appears multiple times, we must ensure all its occurrences are included in the substring. Starting with `required_end = start_index` would fail to account for later occurrences of the starting character itself.

**Solution:**
Initialize with the last occurrence of the starting character:
```python
required_end = last_occurrence[char]
```

