# 1771. Maximize Palindrome Length From Subsequences

> Source: https://algo.monster/liteproblems/1771

## Problem Statement

You are given two strings `word1` and `word2`. Your task is to construct a palindrome by:

1. Selecting a **non-empty** subsequence from `word1` (let's call it `subsequence1`)
2. Selecting a **non-empty** subsequence from `word2` (let's call it `subsequence2`) 
3. Concatenating them as `subsequence1 + subsequence2`

The goal is to find the **length** of the longest palindrome that can be formed this way. If no palindrome can be constructed, return `0`.

Key definitions:
- A **subsequence** is formed by deleting some (possibly zero) characters from the original string without changing the order of remaining characters
- A **palindrome** reads the same forwards and backwards

For example, if `word1 = "cacb"` and `word2 = "cbba"`:
- You could take subsequence "ab" from `word1` and subsequence "ba" from `word2`
- Concatenating gives "abba", which is a palindrome of length 4

The constraint that makes this problem interesting is that you must use at least one character from each word - you cannot form the palindrome using only characters from a single word.

## Intuition

The key insight is to recognize that when we concatenate `subsequence1` from `word1` and `subsequence2` from `word2`, we're essentially looking for a palindrome within the combined string `word1 + word2`. 

Think about what makes a valid palindrome from two subsequences: characters from `word1` must match with characters from `word2` (or within the same word) in a palindromic pattern. For the palindrome to be valid according to our constraints, at least one character from `word1` must be paired with at least one character from `word2`.

This naturally leads us to consider the classic "Longest Palindromic Subsequence" problem on the concatenated string `s = word1 + word2`. The standard [dynamic programming](/problems/dynamic_programming_intro) solution for finding the longest palindromic subsequence can be adapted here.

The crucial modification is ensuring our palindrome spans both words. When we find matching characters `s[i] == s[j]` that contribute to our palindrome, we need to check if one character comes from `word1` (index `i < len(word1)`) and the other comes from `word2` (index `j >= len(word1)`). Only such palindromes are valid answers to our problem.

Why does this work? Because in a palindrome, characters are mirrored around the center. If we have characters from both words participating in this mirroring (one from the left part coming from `word1`, one from the right part coming from `word2`), we guarantee that both subsequences are non-empty.

The DP table `f[i][j]` tracks the longest palindromic subsequence in the range `[i, j]` of the concatenated string, and we only update our answer when we find a palindrome that satisfies our cross-word constraint.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement the solution using [dynamic programming](/problems/dynamic_programming_intro) on the concatenated string `s = word1 + word2`.

**Step 1: Setup**
- Concatenate both strings: `s = word1 + word2`
- Create a 2D DP table `f[i][j]` where `f[i][j]` represents the length of the longest palindromic subsequence in the substring `s[i...j]`
- Initialize the diagonal: `f[i][i] = 1` for all `i`, since each single character is a palindrome of length 1

**Step 2: Fill the DP Table**
We iterate through the string in a specific order - starting from the bottom-right and moving towards the top-left. For each pair of indices `(i, j)` where `i < j`:

- **Case 1: Characters match** (`s[i] == s[j]`)
  - These characters can form the outer boundaries of our palindrome
  - Update: `f[i][j] = f[i + 1][j - 1] + 2`
  - Check if this palindrome spans both words: if `i < len(word1) <= j`
    - This means character at index `i` is from `word1` and character at index `j` is from `word2`
    - Update answer: `ans = max(ans, f[i][j])`

- **Case 2: Characters don't match** (`s[i] != s[j]`)
  - We can't include both characters in our palindrome
  - Take the maximum of excluding either character:
  - Update: `f[i][j] = max(f[i + 1][j], f[i][j - 1])`

**Step 3: Return Result**
The variable `ans` tracks the maximum length of a valid palindrome that uses characters from both words.

**Why the iteration order matters:**
We iterate `i` from `n-2` down to `0` and `j` from `i+1` to `n-1`. This ensures that when we compute `f[i][j]`, the subproblems `f[i+1][j-1]`, `f[i+1][j]`, and `f[i][j-1]` have already been solved.

**Time Complexity:** `O(n²)` where `n = len(word1) + len(word2)`  
**Space Complexity:** `O(n²)` for the DP table

## Example Walkthrough

Let's walk through a small example with `word1 = "ac"` and `word2 = "ca"`.

**Step 1: Setup**
- Concatenate: `s = "ac" + "ca" = "acca"`
- Length: `n = 4`
- Initialize DP table `f[i][j]` (4×4 matrix)
- Set diagonal: `f[0][0] = f[1][1] = f[2][2] = f[3][3] = 1`
- Track `ans = 0` for our result

**Step 2: Fill DP Table**

We iterate with `i` from 2 down to 0, and for each `i`, iterate `j` from `i+1` to 3:

**i = 2, j = 3:**
- `s[2] = 'c'`, `s[3] = 'a'` → characters don't match
- `f[2][3] = max(f[3][3], f[2][2]) = max(1, 1) = 1`

**i = 1:**
- **j = 2:** `s[1] = 'c'`, `s[2] = 'c'` → characters match!
  - `f[1][2] = f[2][1] + 2 = 0 + 2 = 2` (base case: empty range)
  - Check spanning: `1 < 2 ≤ 2` ✓ (index 1 is from word1, index 2 is from word2)
  - Update `ans = 2`

- **j = 3:** `s[1] = 'c'`, `s[3] = 'a'` → characters don't match
  - `f[1][3] = max(f[2][3], f[1][2]) = max(1, 2) = 2`

**i = 0:**
- **j = 1:** `s[0] = 'a'`, `s[1] = 'c'` → characters don't match
  - `f[0][1] = max(f[1][1], f[0][0]) = max(1, 1) = 1`

- **j = 2:** `s[0] = 'a'`, `s[2] = 'c'` → characters don't match
  - `f[0][2] = max(f[1][2], f[0][1]) = max(2, 1) = 2`

- **j = 3:** `s[0] = 'a'`, `s[3] = 'a'` → characters match!
  - `f[0][3] = f[1][2] + 2 = 2 + 2 = 4`
  - Check spanning: `0 < 2 ≤ 3` ✓ (index 0 is from word1, index 3 is from word2)
  - Update `ans = 4`

**Step 3: Result**
The answer is 4, representing the palindrome "acca" formed by:
- Taking subsequence "ac" from word1 (the entire string)
- Taking subsequence "ca" from word2 (the entire string)
- Concatenating to get "acca", which is a palindrome

The DP table helped us identify that characters at positions 0 and 3 form a matching pair (both 'a'), and characters at positions 1 and 2 form another matching pair (both 'c'), creating our palindrome that spans both original words.

## Python Solution

```python
class Solution:
    def longestPalindrome(self, word1: str, word2: str) -> int:
        # Concatenate both words to form a single string
        combined_string = word1 + word2
        n = len(combined_string)
      
        # dp[i][j] represents the length of longest palindromic subsequence
        # in combined_string[i:j+1]
        dp = [[0] * n for _ in range(n)]
      
        # Base case: single character is a palindrome of length 1
        for i in range(n):
            dp[i][i] = 1
      
        # Variable to store the maximum palindrome length that satisfies the condition
        max_palindrome_length = 0
      
        # Fill the dp table bottom-up (from shorter to longer substrings)
        for i in range(n - 2, -1, -1):  # Start from second last character
            for j in range(i + 1, n):    # End position after start
                if combined_string[i] == combined_string[j]:
                    # If characters match, add 2 to the inner subsequence length
                    dp[i][j] = dp[i + 1][j - 1] + 2
                  
                    # Check if this palindrome uses characters from both words
                    # i must be in word1 (i < len(word1)) and j must be in word2 (j >= len(word1))
                    if i < len(word1) <= j:
                        max_palindrome_length = max(max_palindrome_length, dp[i][j])
                else:
                    # If characters don't match, take maximum of excluding either character
                    dp[i][j] = max(dp[i + 1][j], dp[i][j - 1])
      
        return max_palindrome_length
```

## Time and Space Complexity

The time complexity is `O(n^2)`, where `n = len(word1) + len(word2)` is the total length of the concatenated string `s`. This is because we have two nested loops: the outer loop iterates from `n-2` to `0` (which is `O(n)` iterations), and for each iteration, the inner loop iterates from `i+1` to `n-1` (which can be up to `O(n)` iterations). Therefore, the total number of operations is proportional to `n^2`.

The space complexity is `O(n^2)`, where `n = len(word1) + len(word2)`. This is due to the 2D dynamic programming table `f` which has dimensions `n × n`, requiring `n^2` space to store all the intermediate results for the longest palindromic subsequence calculations.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrect Boundary Check for Valid Palindromes

**The Problem:**
A common mistake is incorrectly checking whether a palindrome spans both words. Developers might write:
```python
if i < len(word1) and j >= len(word1):  # Wrong!
```

This condition doesn't guarantee that characters at positions `i` and `j` are actually from different words. The character at position `i` could be from `word1`, but the character at position `j` might not be the matching character that forms the palindrome boundary.

**Why It Happens:**
The confusion arises because when `s[i] == s[j]`, we know these characters form the outer boundary of our palindrome. However, the palindrome might include many characters between them, and we need to ensure the actual matching pair crosses the word boundary.

**The Correct Approach:**
```python
if i < len(word1) <= j:  # Correct!
```
This simpler condition works because:
- When `s[i] == s[j]` and we're adding them as palindrome boundaries
- If `i < len(word1)`, then `s[i]` is from `word1`
- If `j >= len(word1)`, then `s[j]` is from `word2`
- Together, this ensures our palindrome uses at least one character from each word

### Pitfall 2: Forgetting to Initialize the DP Table Diagonal

**The Problem:**
Omitting the initialization of `dp[i][i] = 1` leads to incorrect results:
```python
# Missing initialization - Wrong!
dp = [[0] * n for _ in range(n)]
# Directly starting the nested loops...
```

**Why It Matters:**
- Single characters are palindromes of length 1
- The recurrence relation `dp[i][j] = dp[i+1][j-1] + 2` depends on these base cases
- Without proper initialization, when `i+1 == j` (adjacent characters that match), we'd get `dp[i][j] = 0 + 2 = 2`, which is correct by coincidence
- But for longer sequences, the calculations cascade incorrectly

**The Fix:**
Always initialize the diagonal before filling the rest of the table:
```python
for i in range(n):
    dp[i][i] = 1
```

### Pitfall 3: Wrong Iteration Order

**The Problem:**
Iterating in the wrong order causes accessing uncomputed values:
```python
# Wrong iteration order!
for i in range(n):
    for j in range(i + 1, n):
        # dp[i+1][j-1] might not be computed yet!
        if s[i] == s[j]:
            dp[i][j] = dp[i + 1][j - 1] + 2
```

**Why It Fails:**
The recurrence depends on:
- `dp[i+1][j-1]` (smaller substring, one character removed from each end)
- `dp[i+1][j]` (substring without first character)
- `dp[i][j-1]` (substring without last character)

These subproblems must be solved before computing `dp[i][j]`.

**The Correct Order:**
```python
for i in range(n - 2, -1, -1):  # Iterate i backwards
    for j in range(i + 1, n):    # Iterate j forwards
        # Now all required subproblems are already computed
```
This ensures we compute smaller substrings before larger ones.

