# 10. Regular Expression Matching

> Source: https://algo.monster/liteproblems/10

## Problem Statement

You need to implement a regular expression matcher that supports two special characters:
- `'.'` - Matches any single character
- `'*'` - Matches zero or more of the preceding element

Given a string `s` and a pattern `p`, determine if the pattern matches the **entire** string (not just a substring).

The key rules are:
1. A dot `'.'` can match any single character in the string
2. A star `'*'` applies to the character immediately before it, allowing that character to appear 0 or more times
   - For example, `"a*"` can match `""` (empty), `"a"`, `"aa"`, `"aaa"`, etc.
   - `".*"` can match any sequence of characters including empty string

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization. The function `dfs(i, j)` checks if substring `s[i:]` matches pattern `p[j:]`. 

The algorithm handles three cases:
1. When pattern `p` is exhausted (`j >= n`), the match succeeds only if string `s` is also exhausted
2. When the next pattern character is `'*'` (at position `j+1`), we have two choices:
   - Skip the current pattern character and `'*'` (match 0 occurrences): `dfs(i, j+2)`
   - If current characters match, use the pattern again (match 1+ occurrences): `dfs(i+1, j)`
3. When there's no `'*'` following, characters must match directly and we advance both pointers: `dfs(i+1, j+1)`

The `@cache` decorator prevents recalculating the same subproblems, improving efficiency from exponential to polynomial time complexity.

## Intuition

The challenge with regular expression matching is handling the `'*'` character, which creates variable-length matching possibilities. When we see `"a*"` in the pattern, it could match 0, 1, 2, or any number of `'a'` characters in the string. This creates a branching decision tree.

The key insight is to think recursively: at each position, we make a decision based on what we see in the pattern. If we encounter a character followed by `'*'`, we face a choice - either use this pattern to match the current character in the string (and potentially use it again), or skip this pattern entirely (matching 0 occurrences).

Why does this naturally lead to [dynamic programming](/problems/dynamic_programming_intro)? Consider matching `"aaa"` with pattern `"a*a*"`. As we explore different ways to match, we'll repeatedly ask questions like "can the substring starting at position 2 match the pattern starting at position 3?" These subproblems overlap extensively.

The recursive structure emerges from breaking down the problem:
- Base case: If we've consumed the entire pattern, we succeed only if we've also consumed the entire string
- For `'*'` patterns: We can either skip it (match 0 times) or, if the current character matches, consume one character and keep the pattern available for reuse
- For regular characters or `'.'`: We must match exactly once and move forward in both string and pattern

The beauty of this approach is that it handles all the complex cases naturally. When we have `".*"`, the [recursion](/problems/recursion_intro) automatically explores matching different lengths. When we have consecutive `'*'` patterns like `"a*a*"`, the branching explores all valid distributions of characters between them.

Memoization becomes essential because without it, we'd recalculate the same `(i, j)` pairs multiple times through different recursive paths, leading to exponential time complexity.

**Learn more about [Recursion](/problems/recursion_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses memoization search (top-down [dynamic programming](/problems/dynamic_programming_intro)) to efficiently explore all possible matching paths.

**Implementation Details:**

The core function `dfs(i, j)` determines if `s[i:]` matches `p[j:]`, where `i` and `j` are pointers to the current positions in string `s` and pattern `p` respectively.

**Base Case:**
```python
if j >= n:
    return i == m
```
When we've exhausted the pattern (`j >= n`), the match succeeds only if we've also exhausted the string (`i == m`).

**Case 1: Next character is `'*'`**
```python
if j + 1 < n and p[j + 1] == '*':
    return dfs(i, j + 2) or (
        i < m and (s[i] == p[j] or p[j] == '.') and dfs(i + 1, j)
    )
```
When we detect a `'*'` at position `j + 1`, we have two choices:
- **Match 0 occurrences**: Skip both the character and `'*'` by calling `dfs(i, j + 2)`
- **Match 1+ occurrences**: If current characters match (`s[i] == p[j]` or `p[j] == '.'`), consume one character from `s` while keeping the pattern available by calling `dfs(i + 1, j)`

The function returns `True` if either path succeeds (using logical OR).

**Case 2: Regular character or `'.'`**
```python
return i < m and (s[i] == p[j] or p[j] == '.') and dfs(i + 1, j + 1)
```
Without a `'*'` following, we must match exactly once. We check:
- String hasn't ended (`i < m`)
- Characters match (`s[i] == p[j]` or `p[j] == '.'`)
- The rest of the string matches the rest of the pattern (`dfs(i + 1, j + 1)`)

**Memoization:**
The `@cache` decorator automatically memoizes the function results. Each unique `(i, j)` pair is computed only once, reducing time complexity from exponential `O(2^(m+n))` to polynomial `O(m * n)`.

**Space Complexity:**
The solution uses `O(m * n)` space for the memoization cache, where `m` and `n` are the lengths of string `s` and pattern `p` respectively.

## Example Walkthrough

Let's trace through matching string `s = "aab"` with pattern `p = "c*a*b"`.

We start with `dfs(0, 0)` - matching from the beginning of both strings.

**Step 1:** `dfs(0, 0)` - Looking at `s[0]='a'` and `p[0]='c'`
- Next pattern character `p[1]='*'`, so we have a star case
- Two options:
  - Skip `"c*"`: Call `dfs(0, 2)` 
  - Match `'c'`: Would need `s[0]='a'` to equal `p[0]='c'` - this fails
- So we only explore `dfs(0, 2)`

**Step 2:** `dfs(0, 2)` - Looking at `s[0]='a'` and `p[2]='a'`
- Next pattern character `p[3]='*'`, so we have another star case
- Two options:
  - Skip `"a*"`: Call `dfs(0, 4)`
  - Match `'a'`: Since `s[0]='a'` equals `p[2]='a'`, call `dfs(1, 2)`
- We explore both paths

**Path A - Skip "a*":** `dfs(0, 4)` - Looking at `s[0]='a'` and `p[4]='b'`
- No star follows `'b'`
- Need `s[0]='a'` to equal `p[4]='b'` - this fails
- Return `False`

**Path B - Use "a*":** `dfs(1, 2)` - Looking at `s[1]='a'` and `p[2]='a'`
- Next pattern character `p[3]='*'`, star case again
- Two options:
  - Skip `"a*"`: Call `dfs(1, 4)`
  - Match another `'a'`: Since `s[1]='a'` equals `p[2]='a'`, call `dfs(2, 2)`

**Path B1:** `dfs(1, 4)` - Looking at `s[1]='a'` and `p[4]='b'`
- No star follows
- Need `s[1]='a'` to equal `p[4]='b'` - fails
- Return `False`

**Path B2:** `dfs(2, 2)` - Looking at `s[2]='b'` and `p[2]='a'`
- Next pattern character `p[3]='*'`, star case
- Two options:
  - Skip `"a*"`: Call `dfs(2, 4)`
  - Match: Need `s[2]='b'` to equal `p[2]='a'` - fails

**Final Path:** `dfs(2, 4)` - Looking at `s[2]='b'` and `p[4]='b'`
- No star follows
- Check: `s[2]='b'` equals `p[4]='b'` ✓
- Call `dfs(3, 5)`

**Base Case:** `dfs(3, 5)`
- `j=5 >= n=5` (pattern exhausted)
- Check if string exhausted: `i=3 == m=3` ✓
- Return `True`

The successful path: Skip `"c*"` → Match two `'a'`s with `"a*"` → Match `'b'` with `'b'`.

Each `(i, j)` pair is computed once and cached, preventing redundant calculations when multiple paths reach the same state.

## Python Solution

```python
class Solution:
    def isMatch(self, s: str, p: str) -> bool:
        """
        Regular Expression Matching with '.' and '*'
        '.' matches any single character
        '*' matches zero or more of the preceding element
      
        Args:
            s: The input string to match
            p: The pattern string with regular expression
          
        Returns:
            True if s matches p, False otherwise
        """
        from functools import cache
      
        # Get lengths of input string and pattern
        s_length = len(s)
        p_length = len(p)
      
        @cache
        def match_helper(s_index: int, p_index: int) -> bool:
            """
            Recursively check if substring s[s_index:] matches pattern p[p_index:]
          
            Args:
                s_index: Current index in string s
                p_index: Current index in pattern p
              
            Returns:
                True if the remaining portions match, False otherwise
            """
            # Base case: reached end of pattern
            if p_index >= p_length:
                # Pattern exhausted, check if string is also exhausted
                return s_index == s_length
          
            # Check if next character in pattern is '*' (Kleene star)
            if p_index + 1 < p_length and p[p_index + 1] == '*':
                # Two options with '*':
                # 1. Skip the pattern char and '*' (match 0 occurrences)
                skip_pattern = match_helper(s_index, p_index + 2)
              
                # 2. Match current char and stay at same pattern position (match 1+ occurrences)
                # Only if current position is valid and characters match
                match_current = (s_index < s_length and 
                                (s[s_index] == p[p_index] or p[p_index] == '.') and 
                                match_helper(s_index + 1, p_index))
              
                return skip_pattern or match_current
          
            # Regular character match (no '*' following)
            # Check if current characters match and continue with next positions
            return (s_index < s_length and 
                   (s[s_index] == p[p_index] or p[p_index] == '.') and 
                   match_helper(s_index + 1, p_index + 1))
      
        # Start matching from the beginning of both strings
        return match_helper(0, 0)
```

## Time and Space Complexity

The time complexity is `O(m × n)`, where `m` is the length of string `s` and `n` is the length of pattern `p`. This is because the recursive function `dfs(i, j)` explores different combinations of indices `i` (ranging from 0 to `m`) and `j` (ranging from 0 to `n`). Due to the `@cache` decorator, each unique state `(i, j)` is computed at most once, resulting in at most `(m + 1) × (n + 1)` states being evaluated.

The space complexity is `O(m × n)`. This consists of two main components:
1. The memoization cache from the `@cache` decorator stores up to `(m + 1) × (n + 1)` states
2. The recursion call stack depth in the worst case can be `O(m + n)`

Since the cache dominates the space usage with `O(m × n)`, the overall space complexity is `O(m × n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Consecutive Stars**
A common mistake is not properly handling patterns with consecutive star operators like `"a*b*c*"`. The algorithm might incorrectly process the star's scope.

**Issue Example:**
When matching string `"aaa"` with pattern `"a*a"`, developers might incorrectly think the `a*` should consume all `'a'`s, leaving nothing for the final `'a'` in the pattern.

**Solution:**
The recursive approach naturally handles this by exploring both possibilities at each step - either using the star (consuming a character) or skipping it. The memoization ensures we don't recalculate the same state.

### 2. **Off-by-One Errors When Checking for Star**
The most frequent bug occurs when checking if the next character is a star:

**Incorrect:**
```python
if p[j + 1] == '*':  # Might cause IndexError
```

**Correct:**
```python
if p_index + 1 < p_length and p[p_index + 1] == '*':  # Check bounds first
```

Always verify that `p_index + 1` is within bounds before accessing `p[p_index + 1]`.

### 3. **Forgetting to Handle Empty Pattern with Star**
Patterns like `"a*"` can match an empty string, but developers often forget this case.

**Issue:**
Not allowing the pattern to skip both the character and star together when the star means "zero occurrences".

**Solution:**
Always include the option `match_helper(s_index, p_index + 2)` when encountering a star, which skips both the preceding character and the star itself.

### 4. **Incorrect Base Case Logic**
A subtle but critical error is stopping when the string runs out instead of when the pattern runs out:

**Incorrect:**
```python
if s_index == s_length:
    return p_index == p_length  # Wrong! Rejects s = "", p = "a*"
```

**Correct:**
```python
if p_index >= p_length:
    return s_index == s_length  # Must be exactly at the end
```

An exhausted string can still match a pattern suffix such as `"a*b*"`, because each `x*` pair can match zero characters. Only an exhausted pattern decides the result immediately: the match succeeds exactly when the string is also fully consumed.

### 5. **Not Using Short-Circuit Evaluation Properly**
When checking conditions, the order matters for both correctness and efficiency:

**Problematic:**
```python
return (s[s_index] == p[p_index] or p[p_index] == '.') and s_index < s_length
# Accesses s[s_index] before checking if s_index is valid
```

**Correct:**
```python
return s_index < s_length and (s[s_index] == p[p_index] or p[p_index] == '.')
# Checks bounds first, preventing IndexError
```

### 6. **Misunderstanding Star's Greedy Nature**
Developers sometimes implement the star operator as greedy-only (consuming as many characters as possible) or lazy-only (consuming as few as possible).

**Solution:**
The algorithm must explore both paths - this is why we use:
```python
return skip_pattern or match_current
```
This tries both matching zero occurrences AND matching one-or-more occurrences, letting the recursion find the correct path.

