# 2573. Find the String with LCP

> Source: https://algo.monster/liteproblems/2573

## Problem Statement

Given an `n x n` matrix called `lcp`, you need to find the alphabetically smallest string `word` of length `n` that would produce this exact `lcp` matrix. If no such string exists, return an empty string.

The `lcp` matrix is defined as follows:
- `lcp[i][j]` represents the length of the **longest common prefix** between two substrings of `word`:
  - The first substring is `word[i,n-1]` (from index `i` to the end)
  - The second substring is `word[j,n-1]` (from index `j` to the end)

For example, if `word = "abab"`:
- `lcp[0][2] = 2` because substrings `"abab"` and `"ab"` have a longest common prefix of `"ab"` (length 2)
- `lcp[1][3] = 1` because substrings `"bab"` and `"b"` have a longest common prefix of `"b"` (length 1)

The solution requires you to:
1. Construct a string that would generate the given `lcp` matrix
2. Ensure the string uses only lowercase English letters
3. Return the lexicographically smallest valid string (earliest in dictionary order)
4. Return an empty string if no valid string can produce the given `lcp` matrix

## Intuition

The key insight is understanding what the `lcp` matrix tells us about the structure of the string. If `lcp[i][j] > 0`, it means the substrings starting at positions `i` and `j` share at least one common character at their beginning. More specifically, if `lcp[i][j] = k`, then `word[i] = word[j]`, `word[i+1] = word[j+1]`, ..., up to `word[i+k-1] = word[j+k-1]`.

Since we want the lexicographically smallest string, we should use characters in alphabetical order starting from `'a'`. The [greedy](/problems/greedy_intro) approach works here: whenever we encounter a position that hasn't been assigned a character yet, we assign it the smallest available character.

The construction process follows this logic:
1. Start with the first unfilled position and assign it `'a'`
2. Look at all positions `j` where `lcp[i][j] > 0` - these positions must also start with `'a'` because they share a common prefix with position `i`
3. Move to the next unfilled position and assign it `'b'`, repeating the process
4. Continue until all positions are filled or we run out of letters

The validation step is crucial because not all `lcp` matrices correspond to valid strings. The relationship between `lcp` values must be consistent:
- If `s[i] = s[j]`, then `lcp[i][j]` should equal `1 + lcp[i+1][j+1]` (except at the string's end where it should be `1`)
- If `s[i] ≠ s[j]`, then `lcp[i][j]` must be `0`

This recursive relationship makes sense: if two substrings start with the same character, their longest common prefix is 1 plus the longest common prefix of the substrings starting from their next positions. By checking these conditions backwards (from the end of the string), we can verify if our constructed string produces the exact `lcp` matrix given.

**Learn more about [Greedy](/problems/greedy_intro), [Union Find](/problems/dsu_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation consists of two main phases: **construction** and **validation**.

**Phase 1: Construction**

We initialize an empty string array `s` of length `n` and attempt to fill it with characters:

```python
s = [""] * n
i = 0
for c in ascii_lowercase:
    while i < n and s[i]:
        i += 1
    if i == n:
        break
    for j in range(i, n):
        if lcp[i][j]:
            s[j] = c
```

- We iterate through lowercase letters starting from `'a'`
- For each letter, we find the first unfilled position `i`
- If `lcp[i][j] > 0` for any position `j ≥ i`, we know positions `i` and `j` must share the same starting character, so we assign the current letter to position `j`
- This [greedy](/problems/greedy_intro) assignment ensures we use the smallest possible letters in lexicographical order

After construction, if any position remains unfilled (`"" in s`), it means the `lcp` matrix is invalid, and we return an empty string.

**Phase 2: Validation**

We verify that our constructed string actually produces the given `lcp` matrix:

```python
for i in range(n - 1, -1, -1):
    for j in range(n - 1, -1, -1):
        if s[i] == s[j]:
            if i == n - 1 or j == n - 1:
                if lcp[i][j] != 1:
                    return ""
            elif lcp[i][j] != lcp[i + 1][j + 1] + 1:
                return ""
        elif lcp[i][j]:
            return ""
```

We iterate backwards from the end of the string to check:

1. **When `s[i] = s[j]`:**
   - If either `i` or `j` is at the last position (`n-1`), then `lcp[i][j]` should equal `1` (only one character can match)
   - Otherwise, `lcp[i][j]` should equal `lcp[i+1][j+1] + 1` (the recursive relationship: matching character plus the LCP of remaining substrings)

2. **When `s[i] ≠ s[j]`:**
   - `lcp[i][j]` must be `0` (no common prefix possible)

The backward iteration is crucial because it allows us to build up the validation from known base cases (at the string's end) to earlier positions.

If all checks pass, we join the character array into a string and return it: `return "".join(s)`.

## Example Walkthrough

Let's walk through a small example with `lcp = [[3,2,1],[2,2,1],[1,1,1]]`.

**Step 1: Construction Phase**

We start with an empty array `s = ["", "", ""]` and iterate through lowercase letters.

*Using letter 'a' (i=0):*
- First unfilled position is `i=0`
- Check `lcp[0][j]` for all `j`:
  - `lcp[0][0] = 3` → assign `s[0] = 'a'`
  - `lcp[0][1] = 2` → assign `s[1] = 'a'`
  - `lcp[0][2] = 1` → assign `s[2] = 'a'`
- Result: `s = ['a', 'a', 'a']`

*Using letter 'b' (i=1):*
- First unfilled position is `i=1`, but `s[1]` is already filled
- Move to `i=2`, but `s[2]` is already filled
- Move to `i=3`, which exceeds array bounds
- All positions are filled, so we exit

Our constructed string is `"aaa"`.

**Step 2: Validation Phase**

Now we validate backwards that `"aaa"` produces the given `lcp` matrix.

*Check position (2,2):*
- `s[2] = s[2]` ('a' = 'a')
- Both indices are at position `n-1 = 2`
- Expected: `lcp[2][2] = 1` ✓

*Check position (2,1):*
- `s[2] = s[1]` ('a' = 'a')
- Index 2 is at position `n-1`
- Expected: `lcp[2][1] = 1` ✓

*Check position (2,0):*
- `s[2] = s[0]` ('a' = 'a')
- Index 2 is at position `n-1`
- Expected: `lcp[2][0] = 1` ✓

*Check position (1,2):*
- `s[1] = s[2]` ('a' = 'a')
- Index 2 is at position `n-1`
- Expected: `lcp[1][2] = 1` ✓

*Check position (1,1):*
- `s[1] = s[1]` ('a' = 'a')
- Neither index at `n-1`, so check: `lcp[1][1] = lcp[2][2] + 1 = 1 + 1 = 2` ✓

*Check position (1,0):*
- `s[1] = s[0]` ('a' = 'a')
- Neither index at `n-1`, so check: `lcp[1][0] = lcp[2][1] + 1 = 1 + 1 = 2` ✓

*Check position (0,2):*
- `s[0] = s[2]` ('a' = 'a')
- Index 2 is at position `n-1`
- Expected: `lcp[0][2] = 1` ✓

*Check position (0,1):*
- `s[0] = s[1]` ('a' = 'a')
- Neither index at `n-1`, so check: `lcp[0][1] = lcp[1][2] + 1 = 1 + 1 = 2` ✓

*Check position (0,0):*
- `s[0] = s[0]` ('a' = 'a')
- Neither index at `n-1`, so check: `lcp[0][0] = lcp[1][1] + 1 = 2 + 1 = 3` ✓

All validations pass! The final answer is `"aaa"`.

To verify our understanding, let's confirm the LCP values:
- `lcp[0][0] = 3`: substrings "aaa" and "aaa" have LCP "aaa" (length 3) ✓
- `lcp[0][1] = 2`: substrings "aaa" and "aa" have LCP "aa" (length 2) ✓
- `lcp[0][2] = 1`: substrings "aaa" and "a" have LCP "a" (length 1) ✓
- `lcp[1][1] = 2`: substrings "aa" and "aa" have LCP "aa" (length 2) ✓
- `lcp[1][2] = 1`: substrings "aa" and "a" have LCP "a" (length 1) ✓
- `lcp[2][2] = 1`: substrings "a" and "a" have LCP "a" (length 1) ✓

## Python Solution

```python
from typing import List
from string import ascii_lowercase

class Solution:
    def findTheString(self, lcp: List[List[int]]) -> str:
        """
        Constructs a string from its LCP (Longest Common Prefix) matrix.
      
        Args:
            lcp: A 2D matrix where lcp[i][j] represents the longest common prefix
                 length starting from positions i and j in the target string.
      
        Returns:
            The lexicographically smallest string that matches the LCP matrix,
            or empty string if no valid string exists.
        """
        n = len(lcp)
      
        # Initialize result string array with empty strings
        result_string = [""] * n
      
        # Greedy assignment: assign smallest available character to unassigned positions
        current_position = 0
        for character in ascii_lowercase:
            # Find next unassigned position
            while current_position < n and result_string[current_position]:
                current_position += 1
          
            # All positions have been assigned
            if current_position == n:
                break
          
            # Assign current character to all positions that must have same character
            # based on LCP matrix (if lcp[i][j] > 0, positions i and j must share prefix)
            for j in range(current_position, n):
                if lcp[current_position][j] > 0:
                    result_string[j] = character
      
        # Check if all positions have been assigned
        if "" in result_string:
            return ""
      
        # Validate the constructed string against the LCP matrix
        for i in range(n - 1, -1, -1):
            for j in range(n - 1, -1, -1):
                if result_string[i] == result_string[j]:
                    # Characters match, so LCP should continue
                    if i == n - 1 or j == n - 1:
                        # At boundary: LCP should be exactly 1
                        if lcp[i][j] != 1:
                            return ""
                    else:
                        # Not at boundary: LCP should be 1 + LCP of next positions
                        if lcp[i][j] != lcp[i + 1][j + 1] + 1:
                            return ""
                else:
                    # Characters don't match, so LCP should be 0
                    if lcp[i][j] != 0:
                        return ""
      
        # Join the character array into final string
        return "".join(result_string)
```

## Time and Space Complexity

**Time Complexity: `O(n²)`**

The algorithm consists of three main parts:

1. **String construction phase**: The outer loop iterates through at most 26 characters (lowercase letters). For each character, we scan through the array to find an empty position (`O(n)`), and then iterate through positions to assign the character based on `lcp[i][j]` values (`O(n)`). Overall, this phase is `O(26 × n) = O(n)`.

2. **Validation phase**: This uses two nested loops that iterate from `n-1` to `0`, checking each pair of positions `(i, j)`. For each pair, we perform constant-time operations to validate the LCP constraints. This results in `O(n²)` operations.

The dominant term is `O(n²)` from the validation phase, giving us an overall time complexity of `O(n²)`.

**Space Complexity: `O(n)`**

The algorithm uses:
- Array `s` of size `n` to store the constructed string: `O(n)`
- A few scalar variables (`i`, `c`): `O(1)`
- The final string created by `"".join(s)`: `O(n)`

The total space complexity is `O(n)`, where `n` is the length of the string to be constructed.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Validation Order - Forward vs Backward Iteration**

One of the most common mistakes is validating the LCP matrix in forward order (from index 0 to n-1) instead of backward order. This leads to referencing `lcp[i+1][j+1]` values that haven't been validated yet.

**Incorrect approach:**
```python
# Wrong: Forward iteration
for i in range(n):
    for j in range(n):
        if s[i] == s[j]:
            if i == n - 1 or j == n - 1:
                if lcp[i][j] != 1:
                    return ""
            elif lcp[i][j] != lcp[i + 1][j + 1] + 1:
                # Problem: lcp[i+1][j+1] hasn't been validated yet!
                return ""
```

**Correct approach:**
```python
# Correct: Backward iteration
for i in range(n - 1, -1, -1):
    for j in range(n - 1, -1, -1):
        if s[i] == s[j]:
            if i == n - 1 or j == n - 1:
                if lcp[i][j] != 1:
                    return ""
            elif lcp[i][j] != lcp[i + 1][j + 1] + 1:
                # Now lcp[i+1][j+1] has already been validated
                return ""
```

### 2. **Not Checking Matrix Symmetry and Diagonal Values**

The LCP matrix must be symmetric (`lcp[i][j] == lcp[j][i]`) and diagonal values must equal `n - i` (since `lcp[i][i]` represents the full suffix starting at position i). Failing to validate these properties can lead to accepting invalid matrices.

**Solution - Add preliminary validation:**
```python
# Check matrix properties before construction
for i in range(n):
    for j in range(n):
        # Check symmetry
        if lcp[i][j] != lcp[j][i]:
            return ""
        # Check diagonal values
        if i == j and lcp[i][j] != n - i:
            return ""
        # Check bounds
        if lcp[i][j] > n - max(i, j):
            return ""
```

### 3. **Forgetting to Handle Empty Positions After Character Assignment**

After the greedy character assignment phase, some positions might remain unassigned if we run out of lowercase letters (26 characters). Not checking for this case can cause errors when joining the string.

**Incomplete check:**
```python
# Missing validation after assignment
result_string = [""] * n
# ... character assignment logic ...
# Directly joining without checking
return "".join(result_string)  # Error if any position is still ""
```

**Complete solution:**
```python
# Proper validation
if "" in result_string:
    return ""  # Invalid LCP matrix - can't construct valid string
```

### 4. **Misunderstanding the LCP Relationship**

A subtle but critical error is misunderstanding how LCP values relate to character positions. When `lcp[i][j] > 0`, it means positions `i` and `j` must have the same character, but this doesn't mean only these two positions share the character - all positions with non-zero LCP values with position `i` must share the same character.

**Incorrect assignment:**
```python
# Wrong: Only assigning to position j when lcp[i][j] > 0
if lcp[i][j] > 0:
    result_string[j] = character
    break  # Wrong! Should check all positions
```

**Correct assignment:**
```python
# Correct: Check all positions from current_position to n
for j in range(current_position, n):
    if lcp[current_position][j] > 0:
        result_string[j] = character
```

### 5. **Off-by-One Errors in Boundary Conditions**

When checking if we're at the string's boundary, ensure you're comparing with `n - 1` (the last valid index) rather than `n`.

**Wrong boundary check:**
```python
if i == n or j == n:  # Wrong: n is out of bounds
    if lcp[i][j] != 1:
        return ""
```

**Correct boundary check:**
```python
if i == n - 1 or j == n - 1:  # Correct: n-1 is the last position
    if lcp[i][j] != 1:
        return ""
```

