# 44. Wildcard Matching

> Source: https://algo.monster/liteproblems/44

## Problem Statement

You need to implement a wildcard pattern matching algorithm that checks if a given string `s` matches a pattern `p`. The pattern can contain two special wildcard characters:

- `'?'` - Matches exactly one character (any single character)
- `'*'` - Matches any sequence of characters (can match zero, one, or multiple characters)

The matching must cover the **entire** input string, not just a partial match.

For example:
- Pattern `"a*b"` would match strings like `"ab"`, `"aab"`, `"aaab"`, `"axyzb"` etc.
- Pattern `"a?b"` would match strings like `"aab"`, `"acb"`, `"adb"` but not `"ab"` or `"aaab"`
- Pattern `"*"` would match any string including empty string
- Pattern `"a*b*c"` would match `"abc"`, `"axxxbxxxc"`, `"abbc"`, etc.

The solution uses a recursive approach with memoization. The function `dfs(i, j)` checks if the substring of `s` starting at index `i` matches the substring of pattern `p` starting at index `j`. 

The key logic handles four cases:
1. When we've processed all characters in `s` (base case)
2. When we've processed all characters in `p` (base case) 
3. When the current pattern character is `'*'` - we try three possibilities: skip a character in `s`, skip both current characters, or skip the `'*'` in pattern
4. When the current pattern character is `'?'` or a regular character - we check for a match and continue with the next characters in both strings

The `@cache` decorator is used to store previously computed results and avoid redundant calculations, improving the time complexity.

## Intuition

When we see a pattern matching problem with wildcards, the natural approach is to process both strings character by character and make decisions based on what we encounter. Think of it like walking through both strings simultaneously with two pointers.

The key insight is that at each position, we need to make a decision based on the current characters we're looking at. If we see a regular character or `'?'`, the decision is straightforward - they either match or they don't. But when we encounter `'*'`, things get interesting because it can match any number of characters.

For the `'*'` wildcard, we have multiple choices:
- We can match it with nothing (skip the `'*'` and move to the next pattern character)
- We can match it with one character from `s` (move forward in `s` but stay at `'*'` in case we need to match more)
- We can match it with one character and move past the `'*'` (move forward in both strings)

This branching nature suggests a recursive solution where we explore all possible matching paths. However, we might encounter the same subproblem multiple times. For instance, pattern `"a*b*c"` matching string `"aaabbbccc"` could reach the state `(i=3, j=3)` through different paths - this is where memoization becomes crucial.

The recursive structure emerges naturally:
- Base cases: What happens when we reach the end of either string?
- Recursive cases: How do we handle each type of pattern character?

By breaking down the problem into "does substring `s[i:]` match pattern `p[j:]`?", we transform it into smaller subproblems. The answer to our original question is simply whether `s[0:]` matches `p[0:]`, which is `dfs(0, 0)`.

The beauty of this approach is that it mirrors how we would manually check if a string matches a pattern - we'd go character by character, and when we see a `'*'`, we'd consider all possibilities of what it could match.

**Learn more about [Greedy](/problems/greedy_intro), [Recursion](/problems/recursion_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a recursive approach with memoization using a depth-first search function `dfs(i, j)`. This function determines whether the substring `s[i:]` matches the pattern `p[j:]`.

Let's walk through the implementation step by step:

**1. Base Cases:**

When `i >= len(s)` (we've processed all characters in `s`):
- Return `True` if `j >= len(p)` (both strings are exhausted)
- Or if `p[j] == '*'` and `dfs(i, j + 1)` is `True` (remaining pattern can be all `'*'` which matches empty string)

When `j >= len(p)` (pattern is exhausted but string is not):
- Return `False` since we have unmatched characters in `s`

**2. Handling the `'*'` Wildcard:**

When `p[j] == '*'`, we have three possible transitions:
- `dfs(i + 1, j)`: Match `'*'` with one character from `s` and keep `'*'` for potential future matches
- `dfs(i + 1, j + 1)`: Match `'*'` with one character from `s` and move past `'*'`
- `dfs(i, j + 1)`: Match `'*'` with zero characters (skip the `'*'`)

The function returns `True` if any of these paths lead to a successful match.

**3. Handling Regular Characters and `'?'`:**

For non-`'*'` characters:
- Check if `p[j] == '?'` (matches any single character) or `s[i] == p[j]` (exact character match)
- If there's a match, recursively check `dfs(i + 1, j + 1)` for the remaining substrings
- Return `True` only if both conditions are satisfied

**4. Memoization with `@cache`:**

The `@cache` decorator automatically stores the results of `dfs(i, j)` calls. This prevents redundant computations when the same `(i, j)` pair is encountered through different recursive paths, reducing time complexity from exponential to polynomial.

**Time Complexity:** `O(m * n)` where `m` is the length of string `s` and `n` is the length of pattern `p`, since each `(i, j)` pair is computed at most once.

**Space Complexity:** `O(m * n)` for the memoization cache plus `O(m + n)` for the [recursion](/problems/recursion_intro) stack.

The algorithm systematically explores all valid matching possibilities while efficiently pruning redundant paths through memoization, making it an optimal solution for wildcard pattern matching.

## Example Walkthrough

Let's trace through matching string `s = "abc"` with pattern `p = "a*c"`.

We start with `dfs(0, 0)`:
- `i = 0, j = 0`: Looking at `s[0] = 'a'` and `p[0] = 'a'`
- Neither base case applies (both strings have remaining characters)
- `p[0]` is not `'*'`, so we check if characters match
- `s[0] == p[0]` ('a' == 'a'), so we recursively call `dfs(1, 1)`

Next, `dfs(1, 1)`:
- `i = 1, j = 1`: Looking at `s[1] = 'b'` and `p[1] = '*'`
- `p[1] == '*'`, so we have three options to explore:
  - Option 1: `dfs(2, 1)` - match '*' with 'b', keep '*' for more matches
  - Option 2: `dfs(2, 2)` - match '*' with 'b', move past '*'
  - Option 3: `dfs(1, 2)` - skip '*' without matching anything

Let's explore Option 1: `dfs(2, 1)`:
- `i = 2, j = 1`: Looking at `s[2] = 'c'` and `p[1] = '*'`
- `p[1] == '*'`, so again three options:
  - `dfs(3, 1)` - match '*' with 'c', keep '*'
  - `dfs(3, 2)` - match '*' with 'c', move past '*'
  - `dfs(2, 2)` - skip '*'

The code tries `dfs(3, 1)` first:
- `i = 3 >= len(s)`, we've exhausted string `s`
- `p[1] = '*'`, so it can match the empty rest of `s`; the result depends on `dfs(3, 2)`

Following `dfs(3, 2)`:
- `i = 3 >= len(s)`, we've exhausted string `s`
- `j = 2`: Looking at `p[2] = 'c'`
- Since `p[2]` is not '*' and we have no characters left in `s`, return `False`
- So `dfs(3, 1)` is `False` as well, and the second option `dfs(3, 2)` is already cached as `False`

Let's backtrack and try `dfs(2, 2)` from the previous level:
- `i = 2, j = 2`: Looking at `s[2] = 'c'` and `p[2] = 'c'`
- Characters match ('c' == 'c'), so call `dfs(3, 3)`

Finally, `dfs(3, 3)`:
- `i = 3 >= len(s)` and `j = 3 >= len(p)`
- Both strings are exhausted, return `True`

The `True` value propagates back up through the recursive calls, ultimately returning `True` for `dfs(0, 0)`.

The memoization cache would store results like:
- `(3, 3) → True`
- `(2, 2) → True`
- `(3, 2) → False`
- `(3, 1) → False`
- `(2, 1) → True`
- `(1, 1) → True`
- `(0, 0) → True`

This prevents recalculating these subproblems if encountered again through different paths.

## Python Solution

```python
class Solution:
    def isMatch(self, s: str, p: str) -> bool:
        """
        Wildcard pattern matching with '?' and '*'.
        '?' matches any single character.
        '*' matches any sequence of characters (including empty sequence).
      
        Args:
            s: The input string to match.
            p: The pattern string containing wildcards.
      
        Returns:
            True if the entire string matches the pattern, False otherwise.
        """
        from functools import cache
      
        @cache
        def dfs(string_idx: int, pattern_idx: int) -> bool:
            """
            Recursively check if substring s[string_idx:] matches pattern p[pattern_idx:].
          
            Args:
                string_idx: Current index in the string s.
                pattern_idx: Current index in the pattern p.
          
            Returns:
                True if the remaining substring matches the remaining pattern.
            """
            # Base case: reached end of string
            if string_idx >= len(s):
                # Check if pattern is also exhausted or only has '*' remaining
                return pattern_idx >= len(p) or (p[pattern_idx] == "*" and dfs(string_idx, pattern_idx + 1))
          
            # Base case: pattern exhausted but string still has characters
            if pattern_idx >= len(p):
                return False
          
            # Case 1: current pattern character is '*'
            if p[pattern_idx] == "*":
                # Three options:
                # 1. Match one character from string and stay at '*' (match more)
                # 2. Match one character from string and move past '*' (match exactly one)
                # 3. Match zero characters and move past '*' (match empty)
                return (dfs(string_idx + 1, pattern_idx) or 
                        dfs(string_idx + 1, pattern_idx + 1) or 
                        dfs(string_idx, pattern_idx + 1))
          
            # Case 2: current pattern is '?' or exact character match
            # Both string and pattern must advance together
            return ((p[pattern_idx] == "?" or s[string_idx] == p[pattern_idx]) and 
                    dfs(string_idx + 1, pattern_idx + 1))
      
        # Start matching from the beginning of both strings
        return dfs(0, 0)
```

## Time and Space Complexity

The time complexity is `O(m × n)`, where `m` is the length of string `s` and `n` is the length of pattern `p`. This is because the memoized recursive function `dfs(i, j)` can have at most `(m + 1) × (n + 1)` unique states, where `i` ranges from `0` to `m` and `j` ranges from `0` to `n`. Due to the `@cache` decorator, each state is computed only once, and each state computation involves constant time operations (excluding recursive calls).

The space complexity is `O(m × n)`. This consists of two components:
1. The cache storage for memoization requires `O(m × n)` space to store the results of all possible `(i, j)` pairs.
2. The recursion call stack can go up to `O(m + n)` depth in the worst case, but this is dominated by the cache storage requirement.

Therefore, the overall space complexity is `O(m × n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Inefficient Handling of Consecutive Asterisks**

One common pitfall is not optimizing for patterns with multiple consecutive `'*'` characters (e.g., `"a***b"`). While the current solution works correctly, consecutive asterisks are functionally equivalent to a single asterisk, and processing them separately creates unnecessary recursive branches.

**Problem Example:**
```python
# Pattern: "a***b" matching string: "axxxxb"
# This creates many redundant recursive paths through the three asterisks
```

**Solution:**
Preprocess the pattern to collapse consecutive asterisks into a single one:

```python
def isMatch(self, s: str, p: str) -> bool:
    # Preprocess pattern to remove consecutive asterisks
    cleaned_pattern = []
    for char in p:
        if char != '*' or (not cleaned_pattern or cleaned_pattern[-1] != '*'):
            cleaned_pattern.append(char)
    p = ''.join(cleaned_pattern)
  
    # Continue with the original solution...
    from functools import cache
  
    @cache
    def dfs(string_idx: int, pattern_idx: int) -> bool:
        # ... rest of the implementation
```

### 2. **Stack Overflow with Deep Recursion**

For very long strings or patterns, the recursive solution might hit Python's default recursion limit (typically 1000 levels deep), causing a `RecursionError`.

**Problem Example:**
```python
# Very long string with many asterisks
s = "a" * 5000
p = "*" * 100 + "a" * 5000
# This could exceed recursion depth
```

**Solution:**
Either increase the recursion limit or convert to an iterative dynamic programming approach:

```python
import sys
sys.setrecursionlimit(10000)  # Increase limit if needed

# Or use iterative DP approach:
def isMatch(self, s: str, p: str) -> bool:
    m, n = len(s), len(p)
    dp = [[False] * (n + 1) for _ in range(m + 1)]
    dp[0][0] = True
  
    # Handle patterns starting with '*'
    for j in range(1, n + 1):
        if p[j-1] == '*':
            dp[0][j] = dp[0][j-1]
  
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if p[j-1] == '*':
                dp[i][j] = dp[i-1][j] or dp[i][j-1]
            elif p[j-1] == '?' or s[i-1] == p[j-1]:
                dp[i][j] = dp[i-1][j-1]
  
    return dp[m][n]
```

### 3. **Memory Issues with Large Input Combinations**

The `@cache` decorator stores all unique `(string_idx, pattern_idx)` combinations. For very large strings and patterns, this could consume significant memory.

**Problem Example:**
```python
# Large inputs
s = "x" * 10000
p = "*x*x*x*" * 1000
# Cache could store up to O(m*n) entries
```

**Solution:**
Do not cap the cache with `lru_cache(maxsize=...)`: evicted states get recomputed, and the running time can become exponential again. To reduce memory, switch to the bottom-up DP from Pitfall 2 and keep only two rows, because row `i` depends only on row `i - 1`:

```python
def isMatch(self, s: str, p: str) -> bool:
    m, n = len(s), len(p)
    prev = [False] * (n + 1)
    prev[0] = True
    for j in range(1, n + 1):
        prev[j] = prev[j - 1] and p[j - 1] == '*'

    for i in range(1, m + 1):
        cur = [False] * (n + 1)
        for j in range(1, n + 1):
            if p[j - 1] == '*':
                cur[j] = prev[j] or cur[j - 1]
            elif p[j - 1] == '?' or s[i - 1] == p[j - 1]:
                cur[j] = prev[j - 1]
        prev = cur

    return prev[n]
```

This keeps the `O(m × n)` time but uses only `O(n)` extra space.

