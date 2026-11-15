# 471. Encode String with Shortest Length

> Source: https://algo.monster/liteproblems/471

## Problem Statement

You need to find the shortest possible encoding of a given string `s` using a specific compression rule.

The compression rule allows you to represent repeated substrings in the format `k[encoded_string]`, where:
- `encoded_string` is the substring that repeats
- `k` is the number of times it repeats (must be a positive integer)
- The brackets `[]` are literal characters in the output

For example:
- `"aaa"` can be encoded as `"3[a]"` (shorter than original)
- `"aaaaa"` can be encoded as `"5[a]"`
- `"aaaaaaaaaa"` (10 a's) can be encoded as `"10[a]"`
- `"abcabcabcabc"` can be encoded as `"4[abc]"`
- `"abbbabbbabbb"` can be encoded as `"3[abbb]"` or even `"3[a3[b]]"` if that's shorter

The encoding can be applied recursively - you can encode parts within the brackets as well.

Important constraints:
- Only encode if it makes the string shorter. If encoding doesn't reduce the length, keep the original string
- If multiple valid encodings exist with the same shortest length, you can return any one of them
- The goal is to find the absolute shortest possible encoded representation

The challenge involves finding optimal ways to:
1. Identify repeating patterns in the string
2. Decide whether to encode a substring or leave it as is
3. Consider all possible ways to split and encode the string to achieve minimum length

## Intuition

The key insight is that finding the shortest encoding requires us to explore all possible ways to encode each substring and choose the best one. This naturally leads to a dynamic programming approach.

Think about any substring `s[i:j+1]`. There are essentially two ways we can encode it:

1. **Direct encoding**: Check if this substring contains a repeating pattern. If `"abcabcabc"` repeats `"abc"` three times, we can encode it as `"3[abc]"`. To detect this pattern, we can use a clever trick: concatenate the string with itself and search for the original string starting from index 1. If we find it before reaching the length of the original string, we've found a repeating pattern.

2. **Split encoding**: Break the substring into two parts at some position `k` and encode each part separately. For example, `"aaaaabbbbb"` might be better encoded as `"5[a]5[b]"` rather than trying to encode the whole thing as one pattern.

The challenge is we don't know which approach will give us the shortest result without trying both. For the direct encoding, we also need to recursively find the best encoding for the repeating unit itself (like encoding `"abc"` within `"3[abc]"`).

This recursive structure with overlapping subproblems suggests dynamic programming. We need to:
- Process substrings from shortest to longest (or use memoization)
- For each substring, try both direct encoding and all possible split points
- Keep track of the shortest encoding found for each substring

The reason we start from the end of the string and work backwards (`i` from `n-1` to `0`) while expanding `j` from `i` to `n-1` is to ensure that when we need the optimal encoding of a smaller substring, we've already computed it. This bottom-up approach guarantees that all necessary subproblems are solved before we need them.

The condition to check if encoding is worth it (length < 5) comes from the fact that the shortest possible encoding, such as `"4[a]"`, already has length 4 (a digit, two brackets, and at least one character), so a string shorter than 5 characters can never be shortened by encoding.

## Solution Approach

The solution uses dynamic programming with a 2D table `f[i][j]` where `f[i][j]` stores the shortest encoding of substring `s[i:j+1]`.

**Helper Function `g(i, j)`:**
This function attempts to directly encode the substring `s[i:j+1]` if it contains a repeating pattern.

1. Extract the substring `t = s[i:j+1]`
2. If length is less than 5, return the original string (encoding won't make it shorter)
3. Find the repeating pattern using the trick: `k = (t + t).index(t, 1)`
   - Concatenate `t` with itself to get `t + t`
   - Search for `t` starting from index 1
   - If `k < len(t)`, we found a repeating pattern with period `k`
4. If a pattern exists:
   - Calculate repetition count: `cnt = len(t) // k`
   - Return the encoded format: `"cnt[f[i][i+k-1]]"` where `f[i][i+k-1]` is the optimal encoding of one repeating unit
5. Otherwise, return the original substring

**Main Dynamic Programming Logic:**

1. Initialize a 2D array `f` of size `n×n` where `n = len(s)`

2. Process substrings in order of increasing length:
   - Outer loop: `i` from `n-1` down to `0` (starting position)
   - Inner loop: `j` from `i` to `n-1` (ending position)

3. For each substring `s[i:j+1]`:
   - First, try direct encoding: `f[i][j] = g(i, j)`
   - If substring length > 4, try all possible split points:
     ```
     for k in range(i, j):
         t = f[i][k] + f[k+1][j]
         if len(f[i][j]) > len(t):
             f[i][j] = t
     ```
   - This combines the optimal encodings of left part `f[i][k]` and right part `f[k+1][j]`
   - Keep the shorter encoding between direct encoding and split encoding

4. Return `f[0][-1]` which contains the optimal encoding of the entire string

**Time Complexity:** `O(n³)` - For each of `O(n²)` substrings, we check `O(n)` split points.

**Space Complexity:** `O(n²)` - For the 2D DP table storing optimal encodings of all substrings.

The algorithm ensures optimal substructure by processing smaller substrings first, guaranteeing that when we need `f[i][i+k-1]` in the helper function or `f[i][k]` and `f[k+1][j]` in split encoding, these values are already computed.

## Example Walkthrough

Let's walk through the algorithm with the string `s = "aabcabc"`.

**Step 1: Initialize DP table**
We create a 7×7 table `f` (since length = 7). We'll process substrings from length 1 to 7.

**Step 2: Process substrings (i from 6 to 0, j from i to 6)**

Starting with single characters (base cases):
- `f[0][0] = "a"` (substring "a")
- `f[1][1] = "a"`
- `f[2][2] = "b"`
- `f[3][3] = "c"`
- `f[4][4] = "a"`
- `f[5][5] = "b"`
- `f[6][6] = "c"`

Length 2 substrings:
- `f[0][1] = "aa"` → Direct encoding: g(0,1) returns `"aa"` immediately because its length is less than 5, so `f[0][1] = "aa"`
- `f[1][2] = "ab"` → No pattern, remains `"ab"`
- `f[2][3] = "bc"` → No pattern, remains `"bc"`
- Similarly for others...

Length 4 substring `"bcab"` at `f[2][5]`:
- Direct encoding: No repeating pattern → `"bcab"`
- Try splits:
  - Split at 2: `f[2][2] + f[3][5]` = `"b" + "cab"` = `"bcab"` (length 4)
  - Split at 3: `f[2][3] + f[4][5]` = `"bc" + "ab"` = `"bcab"` (length 4)
  - Split at 4: `f[2][4] + f[5][5]` = `"bca" + "b"` = `"bcab"` (length 4)
- Result: `f[2][5] = "bcab"`

Length 6 substring `"abcabc"` at `f[1][6]`:
- Direct encoding: g(1,6) finds "abcabc" = 2 × "abc"
  - `(abcabc + abcabc).index(abcabc, 1)` returns 3
  - Pattern length = 3, repetitions = 6/3 = 2
  - Encoding: `"2[" + f[1][3] + "]"` = `"2[abc]"` (length 6)
- Try splits:
  - Split at 1: `"a" + "bcabc"` = `"abcabc"` (length 6)
  - Split at 2: `"ab" + "cabc"` = `"abcabc"` (length 6)
  - Split at 3: `"abc" + "abc"` = `"abcabc"` (length 6)
  - Split at 4: `"abca" + "bc"` = `"abcabc"` (length 6)
  - Split at 5: `"abcab" + "c"` = `"abcabc"` (length 6)
- Result: `f[1][6] = "2[abc]"` (same length as original, but valid encoding)

**Final substring** `"aabcabc"` at `f[0][6]`:
- Direct encoding: No repeating pattern → `"aabcabc"`
- Try splits:
  - Split at 0: `f[0][0] + f[1][6]` = `"a" + "2[abc]"` = `"a2[abc]"` (length 7)
  - Split at 1: `f[0][1] + f[2][6]` = `"aa" + "bcabc"` = `"aabcabc"` (length 7)
  - Other splits also give length 7 or more
- No split is strictly shorter than 7, and the code replaces the current answer only when a split is strictly shorter, so the direct result stays.
- Result: `f[0][6] = "aabcabc"`

**Answer: `"aabcabc"`**. The encoding `"a2[abc]"` has the same length, 7, so neither one is shorter and either is an accepted answer. This solution returns the unencoded string because it keeps the first shortest candidate it finds.

## Python Solution

```python
class Solution:
    def encode(self, s: str) -> str:
        """
        Encode a string by replacing repeated substrings with k[pattern] format.
        Uses dynamic programming to find the optimal encoding.
        """

        def find_best_encoding(start: int, end: int) -> str:
            """
            Find the best encoding for substring s[start:end+1].
            If a repeating pattern exists, encode as k[pattern].
            """
            substring = s[start : end + 1]

            # Don't encode strings shorter than 5 characters
            if len(substring) < 5:
                return substring

            # Find if substring is made of repeating pattern
            # Trick: search for substring in doubled version starting from index 1
            # If found before len(substring), it means there's a repeating pattern
            pattern_end_index = (substring + substring).index(substring, 1)

            if pattern_end_index < len(substring):
                # Found a repeating pattern
                pattern_length = pattern_end_index
                repetition_count = len(substring) // pattern_length
                # Recursively encode the pattern itself
                encoded_pattern = dp[start][start + pattern_length - 1]
                return f"{repetition_count}[{encoded_pattern}]"

            return substring

        n = len(s)
        # dp[i][j] stores the best encoding for substring s[i:j+1]
        dp = [[None] * n for _ in range(n)]

        # Fill dp table from smaller substrings to larger ones
        # Process from right to left for start index
        for start in range(n - 1, -1, -1):
            # Process from left to right for end index
            for end in range(start, n):
                # First, try encoding the whole substring as a pattern
                dp[start][end] = find_best_encoding(start, end)

                # For substrings longer than 4, try splitting into two parts
                if end - start + 1 > 4:
                    for split_point in range(start, end):
                        # Combine encodings of left and right parts
                        left_part = dp[start][split_point]
                        right_part = dp[split_point + 1][end]
                        combined_encoding = left_part + right_part

                        # Keep the shorter encoding
                        if len(dp[start][end]) > len(combined_encoding):
                            dp[start][end] = combined_encoding

        # Return the best encoding for the entire string
        return dp[0][-1]
```

## Time and Space Complexity

**Time Complexity:** `O(n^3)`

The algorithm uses dynamic programming with a 2D table where `f[i][j]` represents the shortest encoding for substring `s[i:j+1]`.

- The outer two loops iterate through all possible substrings, contributing `O(n^2)` iterations.
- For each substring `(i, j)`:
  - The function `g(i, j)` is called, which:
    - Creates substring `t` in `O(n)` time
    - Performs pattern matching using `(t + t).index(t, 1)` in `O(n)` time
    - Overall `g()` takes `O(n)` time
  - When `j - i + 1 > 4`, we try all possible split points `k` between `i` and `j`, which adds another `O(n)` loop
  - String concatenation `f[i][k] + f[k + 1][j]` takes `O(n)` in worst case

Therefore, the total time complexity is `O(n^2) * O(n) = O(n^3)`.

**Space Complexity:** `O(n^2)`

- The 2D DP table `f` has dimensions `n × n`, storing strings that could be up to length `n` each
- In the worst case, each cell stores a string of length `O(n)`
- Therefore, the total space complexity is `O(n^2 * n) = O(n^3)` for storing all strings

However, if we only consider the space for the DP table structure itself (number of cells), it's `O(n^2)`. The actual space usage is `O(n^3)` when accounting for string storage.

## Common Pitfalls

### 1. **Incorrect Pattern Detection Logic**
The most common pitfall is misunderstanding how the pattern detection works with `(substring + substring).index(substring, 1)`. Many developers incorrectly assume this finds any repeating pattern, but it specifically finds the smallest period of repetition.

**Problem Example:**
```python
# Incorrect understanding might lead to:
substring = "abcabc"
# Some might think this detects "abc" repeating twice
# But actually: (substring + substring).index(substring, 1) returns 3
# This correctly identifies that "abc" is the repeating unit
```

**Solution:** Understand that this trick works because if a string has period `k`, then shifting it by `k` positions produces the same string. The `.index()` method finds the first occurrence of the full substring in the doubled version, which gives us the smallest period.

### 2. **Forgetting to Check Encoding Length**
A critical mistake is always applying the encoding without checking if it actually shortens the string.

**Problem Example:**
```python
# For substring "ab" repeated twice: "abab"
# Encoding would be "2[ab]" which has length 6
# Original "abab" has length 4 - shorter!
# Must keep original, not encode
```

**Solution:** Always compare the length of the encoded string with the original before deciding to use the encoding. The condition `len(substring) < 5` in the code handles this for the base case.

### 3. **Not Handling Edge Cases in Recursion**
When building the encoded string like `f"{repetition_count}[{encoded_pattern}]"`, forgetting that the pattern itself might need encoding can lead to suboptimal results.

**Problem Example:**
```python
# For "aaaaaaaaaa" (10 a's)
# Direct encoding: "10[a]" - length 6
# But if we had "aaaaaaaaaaaaaaaaaaaaa" (20 a's)
# We might get "20[a]" - length 6
# Or we could get "2[10[a]]" - length 9 (worse)
# The DP ensures we pick the optimal one
```

**Solution:** The recursive call `dp[start][start + pattern_length - 1]` ensures the pattern itself is optimally encoded before being used in the larger encoding.

### 4. **Incorrect DP Table Traversal Order**
Processing the DP table in the wrong order can lead to using uninitialized values.

**Problem Example:**
```python
# Wrong order:
for start in range(n):  # Going forward
    for end in range(start, n):
        # When we need dp[start][start + pattern_length - 1]
        # It might not be computed yet!
```

**Solution:** Process from right to left for the start index and ensure smaller substrings are computed before larger ones. The code correctly uses `range(n - 1, -1, -1)` for the outer loop.

### 5. **Off-by-One Errors in Index Management**
Mixing inclusive and exclusive indices can cause bugs, especially when dealing with substring slicing.

**Problem Example:**
```python
# If dp[i][j] represents s[i:j] (exclusive end)
# vs dp[i][j] represents s[i:j+1] (inclusive end)
# Mixing these conventions leads to errors
```

**Solution:** Be consistent with index conventions. The provided code uses `dp[i][j]` to represent `s[i:j+1]`, meaning `j` is inclusive. All substring operations must follow this convention consistently.

