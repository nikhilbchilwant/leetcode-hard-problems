# 1216. Valid Palindrome III

> Source: https://algo.monster/liteproblems/1216

## Problem Statement

You are given a string `s` and an integer `k`. Your task is to determine if the string is a **k-palindrome**.

A string is considered a **k-palindrome** if you can transform it into a palindrome by removing at most `k` characters from it.

For example:
- If `s = "abcdeca"` and `k = 2`, you can remove characters 'b' and 'e' to get "acdca", which is a palindrome. So this would return `true`.
- If a string is already a palindrome, it's a k-palindrome for any `k ≥ 0`.
- If you need to remove more than `k` characters to make it a palindrome, then it's not a k-palindrome.

The function should return `true` if the string can be made into a palindrome by removing at most `k` characters, and `false` otherwise.

## Intuition

The key insight is to reframe the problem: instead of thinking about which characters to remove, we can think about which characters to keep. If we need to remove at most `k` characters to make a palindrome, that means we need to keep at least `n - k` characters that form a palindrome.

This transforms our problem into finding the longest palindromic subsequence (LPS) in the string. A subsequence doesn't need to be contiguous - we can pick any characters from the string while maintaining their relative order.

Why does this work? Consider that:
- If the LPS has length `L`, then we need to remove `n - L` characters to get that palindrome
- If `n - L ≤ k`, then we can make a palindrome by removing at most `k` characters
- This is equivalent to checking if `L + k ≥ n`

For finding the LPS, we can use [dynamic programming](/problems/dynamic_programming_intro). The idea is to build up solutions for smaller substrings and combine them:
- For a single character, the LPS length is 1
- For a substring `s[i..j]`, if the characters at both ends match (`s[i] == s[j]`), we can include both in our palindrome along with whatever palindrome we can form from the middle part `s[i+1..j-1]`
- If they don't match, we have to choose: either exclude `s[i]` and find the best palindrome in `s[i+1..j]`, or exclude `s[j]` and find the best palindrome in `s[i..j-1]`

By systematically building up these solutions from smaller to larger substrings, we can determine if any substring's LPS is long enough that removing the remaining characters would require at most `k` deletions.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement the solution using [dynamic programming](/problems/dynamic_programming_intro) with a 2D table `f[i][j]` where `f[i][j]` represents the length of the longest palindromic subsequence in the substring `s[i..j]`.

**Initialization:**
- Create an `n x n` matrix `f` where `n = len(s)`
- Initialize the diagonal: `f[i][i] = 1` for all `i`, since every single character is a palindrome of length 1

**Building the DP table:**
We fill the table by iterating through substrings of increasing length:
- Start from `i = n-2` and go backwards to `0` (outer loop)
- For each `i`, iterate `j` from `i+1` to `n-1` (inner loop)
- This ensures we always have the values for smaller substrings before computing larger ones

**Transition formula:**
For each substring `s[i..j]`:
- If `s[i] == s[j]`: The characters at both ends match, so we can include both in our palindrome:
  ```
  f[i][j] = f[i+1][j-1] + 2
  ```
  We add 2 to the LPS of the inner substring `s[i+1..j-1]`

- If `s[i] != s[j]`: The characters don't match, so we take the maximum of:
  ```
  f[i][j] = max(f[i+1][j], f[i][j-1])
  ```
  Either excluding `s[i]` or excluding `s[j]`

**Early termination optimization:**
After computing `f[i][j]`, we immediately check if `f[i][j] + k >= n`:
- If true, we've found a palindromic subsequence that's long enough
- This means we can remove at most `n - f[i][j]` characters, which is `≤ k`
- Return `True` immediately

**Final check:**
If we complete all iterations without finding a valid palindrome, return `False`.

The time complexity is `O(n²)` for filling the DP table, and space complexity is also `O(n²)` for storing the table.

## Example Walkthrough

Let's walk through the solution with `s = "abcdeca"` and `k = 2`.

**Step 1: Initialize the DP table**
We create a 7×7 matrix (since n=7) and initialize the diagonal with 1s:

```
    a b c d e c a
    0 1 2 3 4 5 6
a 0 [1 ? ? ? ? ? ?]
b 1 [  1 ? ? ? ? ?]
c 2 [    1 ? ? ? ?]
d 3 [      1 ? ? ?]
e 4 [        1 ? ?]
c 5 [          1 ?]
a 6 [            1]
```

**Step 2: Fill the table bottom-up**
We start from i=5 (second-to-last row) and work backwards:

*When i=5, j=6:*
- Compare s[5]='c' with s[6]='a' → they don't match
- f[5][6] = max(f[6][6], f[5][5]) = max(1, 1) = 1
- Check: 1 + 2 >= 7? No, continue

*When i=4:*
- j=5: s[4]='e' vs s[5]='c' → don't match → f[4][5] = 1
- j=6: s[4]='e' vs s[6]='a' → don't match → f[4][6] = max(f[5][6], f[4][5]) = 1

*When i=3:*
- j=4: s[3]='d' vs s[4]='e' → don't match → f[3][4] = 1
- j=5: s[3]='d' vs s[5]='c' → don't match → f[3][5] = 1
- j=6: s[3]='d' vs s[6]='a' → don't match → f[3][6] = 1

*When i=2:*
- j=3: s[2]='c' vs s[3]='d' → don't match → f[2][3] = 1
- j=4: s[2]='c' vs s[4]='e' → don't match → f[2][4] = 1
- j=5: s[2]='c' vs s[5]='c' → **match!** → f[2][5] = f[3][4] + 2 = 1 + 2 = 3
- Check: 3 + 2 >= 7? No, continue
- j=6: s[2]='c' vs s[6]='a' → don't match → f[2][6] = max(f[3][6], f[2][5]) = 3

*When i=1:*
- j=2: s[1]='b' vs s[2]='c' → don't match → f[1][2] = 1
- j=3: s[1]='b' vs s[3]='d' → don't match → f[1][3] = 1
- j=4: s[1]='b' vs s[4]='e' → don't match → f[1][4] = 1
- j=5: s[1]='b' vs s[5]='c' → don't match → f[1][5] = max(f[2][5], f[1][4]) = 3
- j=6: s[1]='b' vs s[6]='a' → don't match → f[1][6] = max(f[2][6], f[1][5]) = 3

*When i=0:*
- j=1: s[0]='a' vs s[1]='b' → don't match → f[0][1] = 1
- j=2: s[0]='a' vs s[2]='c' → don't match → f[0][2] = 1
- j=3: s[0]='a' vs s[3]='d' → don't match → f[0][3] = 1
- j=4: s[0]='a' vs s[4]='e' → don't match → f[0][4] = 1
- j=5: s[0]='a' vs s[5]='c' → don't match → f[0][5] = max(f[1][5], f[0][4]) = 3
- j=6: s[0]='a' vs s[6]='a' → **match!** → f[0][6] = f[1][5] + 2 = 3 + 2 = 5
- Check: 5 + 2 >= 7? **Yes!**

**Step 3: Return True**
We found that the longest palindromic subsequence from index 0 to 6 has length 5. This means we only need to remove 7 - 5 = 2 characters, which equals our k value. The function returns `True`.

The palindromic subsequence is "acdca" (keeping characters at positions 0, 2, 3, 5, 6), achieved by removing 'b' and 'e'.

## Python Solution

```python
class Solution:
    def isValidPalindrome(self, s: str, k: int) -> bool:
        """
        Check if string s can become a palindrome by removing at most k characters.

        Args:
            s: Input string to check
            k: Maximum number of characters that can be removed

        Returns:
            True if s can become a palindrome with at most k removals, False otherwise
        """
        n = len(s)

        # dp[i][j] represents the length of longest palindromic subsequence
        # in substring s[i:j+1]
        dp = [[0] * n for _ in range(n)]

        # Base case: single character is always a palindrome of length 1
        for i in range(n):
            dp[i][i] = 1

        # Fill the dp table from bottom-right to top-left
        # Process substrings of increasing length
        for i in range(n - 2, -1, -1):
            for j in range(i + 1, n):
                if s[i] == s[j]:
                    # If characters match, extend the palindrome from inner substring
                    # Add 2 for the matching characters at both ends
                    dp[i][j] = dp[i + 1][j - 1] + 2
                else:
                    # If characters don't match, take the maximum of:
                    # - Excluding character at position i
                    # - Excluding character at position j
                    dp[i][j] = max(dp[i + 1][j], dp[i][j - 1])

                # Early termination optimization:
                # If longest palindromic subsequence + k >= total length,
                # we can form a palindrome by removing at most k characters
                if dp[i][j] + k >= n:
                    return True

        # Check if the full string satisfies the condition
        return dp[0][n - 1] + k >= n
```

## Time and Space Complexity

The time complexity is `O(n^2)`, where `n` is the length of the string `s`. This is because the algorithm uses two nested loops - the outer loop iterates from `n-2` to `0` (approximately `n` iterations), and for each iteration, the inner loop runs from `i+1` to `n` (up to `n` iterations in the worst case). Each operation inside the nested loops takes constant time `O(1)`, resulting in an overall time complexity of `O(n^2)`.

The space complexity is `O(n^2)`, where `n` is the length of the string `s`. This is due to the 2D array `f` of size `n × n` that stores the dynamic programming states. Each cell in this array stores the length of the longest palindromic subsequence for the substring `s[i:j+1]`, requiring `O(n^2)` space in total.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Index Out of Bounds When Accessing `dp[i+1][j-1]`**

The most common pitfall in this solution is accessing `dp[i+1][j-1]` when `i+1 > j-1`, which happens when processing substrings of length 2 (where `j = i+1`). In this case, we're trying to access an "invalid" substring where the start index is greater than the end index.

**Why this happens:**
- When `j = i+1` (substring of length 2), computing `dp[i+1][j-1]` means accessing `dp[i+1][i]`
- This represents an empty substring between the two characters
- The value at this position might be uninitialized (0) or garbage, leading to incorrect results

**Solution:**
Add a special case to handle substrings of length 2:

```python
if s[i] == s[j]:
    if j == i + 1:  # Adjacent characters
        dp[i][j] = 2
    else:
        dp[i][j] = dp[i + 1][j - 1] + 2
else:
    dp[i][j] = max(dp[i + 1][j], dp[i][j - 1])
```

Or alternatively, initialize the base case for empty substrings:

```python
# After initializing diagonal elements
# Handle the conceptual "empty" substrings
for i in range(n):
    for j in range(i):
        dp[i][j] = 0  # Empty substring has LPS of 0
```

### 2. **Incorrect Early Termination Logic**

Another pitfall is misunderstanding when to return `True`. The condition `dp[i][j] + k >= n` checks if we can keep enough characters to form a palindrome, but placing this check inside the nested loops means we're checking partial substrings, not just the full string.

**Why this could be problematic:**
- Checking partial substrings `s[i..j]` where `j < n-1` might give false positives
- We might return `True` based on a substring's properties rather than the full string

**Solution:**
If you want to be more conservative and only check the full string:

```python
# Remove the early termination inside the loops
for i in range(n - 2, -1, -1):
    for j in range(i + 1, n):
        if s[i] == s[j]:
            dp[i][j] = dp[i + 1][j - 1] + 2
        else:
            dp[i][j] = max(dp[i + 1][j], dp[i][j - 1])

# Only check the full string at the end
return dp[0][n - 1] + k >= n
```

However, the original early termination is actually correct because if any substring `s[i..j]` containing all characters (when `j = n-1`) satisfies the condition, then the full string definitely can be made into a palindrome with at most k removals.

