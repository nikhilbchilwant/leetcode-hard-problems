# 1312. Minimum Insertion Steps to Make a String Palindrome

> Source: https://algo.monster/liteproblems/1312

## Problem Statement

You are given a string `s`. You can insert any character at any position in the string (including at the beginning or end). Your goal is to find the minimum number of character insertions needed to make the string a palindrome.

A palindrome is a string that reads the same forwards and backwards. For example, "racecar" and "noon" are palindromes.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with memoization. The approach works by comparing characters from both ends of the string:

- The function `dfs(i, j)` finds the minimum insertions needed to make the substring from index `i` to index `j` a palindrome
- If `i >= j`, we've processed the entire substring (base case), so return 0
- If the characters at positions `i` and `j` match (`s[i] == s[j]`), we can move both pointers inward without any insertions
- If they don't match, we have two choices:
  - Insert a character matching `s[j]` before position `i`, then solve for `dfs(i, j-1)`
  - Insert a character matching `s[i]` after position `j`, then solve for `dfs(i+1, j)`
  - Take the minimum of these two options and add 1 for the insertion

The `@cache` decorator memoizes results to avoid recalculating the same subproblems.

For example, with string "ab":
- Characters don't match, so we need at least 1 insertion
- We can either insert 'b' at the beginning to get "bab" or insert 'a' at the end to get "aba"
- Both result in 1 insertion, which is the minimum

## Intuition

The key insight is recognizing that this problem is closely related to finding the Longest Palindromic Subsequence (LPS). When we think about making a string a palindrome through insertions, we're essentially trying to match characters that don't already form a palindromic pattern.

Consider what happens when we examine a string from both ends. If the characters at the two ends match, they already contribute to the palindromic structure - we don't need to insert anything for them. The problem then reduces to making the middle portion a palindrome.

When the characters don't match, we face a decision: which character should we "duplicate" through insertion? If we have `s[i]` on the left and `s[j]` on the right that don't match, we could either:
1. Insert `s[j]` somewhere on the left side to match with the existing `s[j]` on the right
2. Insert `s[i]` somewhere on the right side to match with the existing `s[i]` on the left

This naturally leads to a recursive structure where we try both options and pick the one requiring fewer insertions. The beauty of this approach is that it breaks down the problem into smaller subproblems - each recursive call handles a smaller substring.

The relationship to LPS becomes clearer when we realize that the minimum insertions needed equals `n - LPS_length`, where `n` is the string length. The characters already in the LPS don't need matching insertions; we only need to insert characters for those not part of the LPS. However, directly computing using the two-pointer comparison approach as shown in the solution is more intuitive and equally efficient.

The overlapping subproblems make this perfect for [dynamic programming](/problems/dynamic_programming_intro) - the same substring ranges will be evaluated multiple times in different recursive paths, so memoization dramatically improves efficiency from exponential to `O(n²)`.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a top-down [dynamic programming](/problems/dynamic_programming_intro) approach using recursion with memoization. Let's walk through the implementation step by step:

**Function Structure:**
The main function `minInsertions` defines a nested helper function `dfs(i, j)` that computes the minimum insertions needed for the substring `s[i:j+1]`.

**Memoization:**
The `@cache` decorator is used to automatically memoize the results of `dfs`. This prevents redundant calculations when the same substring range `(i, j)` is encountered multiple times during recursion.

**Base Case:**
```python
if i >= j:
    return 0
```
When `i >= j`, it means we're dealing with either:
- A single character (`i == j`), which is already a palindrome
- An empty substring (`i > j`), which is also considered a palindrome
Both cases require 0 insertions.

**Matching Characters:**
```python
if s[i] == s[j]:
    return dfs(i + 1, j - 1)
```
When characters at both ends match, they already form part of the palindrome structure. We move both pointers inward without adding any insertions, recursively solving for the inner substring.

**Non-matching Characters:**
```python
return 1 + min(dfs(i + 1, j), dfs(i, j - 1))
```
When characters don't match, we have two choices:
- `dfs(i + 1, j)`: Skip character at position `i`, meaning we'll insert a matching character for `s[i]` on the right side later
- `dfs(i, j - 1)`: Skip character at position `j`, meaning we'll insert a matching character for `s[j]` on the left side later

We take the minimum of both options and add 1 for the required insertion.

**Initial Call:**
```python
return dfs(0, len(s) - 1)
```
The recursion starts with the entire string, from index 0 to `len(s) - 1`.

**Time Complexity:** `O(n²)` where n is the length of the string, as there are `n²` possible unique `(i, j)` pairs, and each is computed once due to memoization.

**Space Complexity:** `O(n²)` for the memoization cache storing results for all possible substring ranges.

## Example Walkthrough

Let's trace through the algorithm with the string `s = "abc"` to find the minimum insertions needed to make it a palindrome.

**Initial Call:** `dfs(0, 2)` - examining the full string "abc"
- Compare `s[0]='a'` with `s[2]='c'`
- They don't match, so we need to choose between two options:
  - Option 1: `dfs(1, 2)` - skip 'a', will insert 'a' on the right later
  - Option 2: `dfs(0, 1)` - skip 'c', will insert 'c' on the left later

**Exploring Option 1:** `dfs(1, 2)` - examining substring "bc"
- Compare `s[1]='b'` with `s[2]='c'`
- They don't match, so we evaluate:
  - `dfs(2, 2)` - single character 'c', returns 0 (base case)
  - `dfs(1, 1)` - single character 'b', returns 0 (base case)
- Result: `1 + min(0, 0) = 1`

**Exploring Option 2:** `dfs(0, 1)` - examining substring "ab"
- Compare `s[0]='a'` with `s[1]='b'`
- They don't match, so we evaluate:
  - `dfs(1, 1)` - single character 'b', returns 0 (base case)
  - `dfs(0, 0)` - single character 'a', returns 0 (base case)
- Result: `1 + min(0, 0) = 1`

**Final Calculation:** `dfs(0, 2) = 1 + min(1, 1) = 2`

The algorithm determines we need **2 insertions** to make "abc" a palindrome. We could either:
- Insert 'c' at the beginning and 'a' at the end: "c**ab**c**a**" → "cabca"
- Insert 'b' after 'a' and 'a' at the end: "a**b**c**ba**" → "abcba"

Both create valid palindromes with exactly 2 insertions, confirming our result.

## Python Solution

```python
class Solution:
    def minInsertions(self, s: str) -> int:
        """
        Find minimum number of insertions needed to make a string palindrome.
      
        Args:
            s: Input string
          
        Returns:
            Minimum number of character insertions required
        """
        from functools import cache
      
        @cache
        def dfs(left: int, right: int) -> int:
            """
            Recursively find minimum insertions for substring s[left:right+1].
          
            Args:
                left: Left pointer index
                right: Right pointer index
              
            Returns:
                Minimum insertions needed for current substring
            """
            # Base case: single character or empty substring is already palindrome
            if left >= right:
                return 0
          
            # If characters at both ends match, no insertion needed for these positions
            if s[left] == s[right]:
                return dfs(left + 1, right - 1)
          
            # Characters don't match, need to insert
            # Either insert s[right] at beginning or s[left] at end
            insert_at_left = dfs(left, right - 1)  # Insert s[right] before s[left]
            insert_at_right = dfs(left + 1, right)  # Insert s[left] after s[right]
          
            return 1 + min(insert_at_left, insert_at_right)
      
        # Start with entire string
        return dfs(0, len(s) - 1)
```

## Time and Space Complexity

**Time Complexity:** `O(n^2)` where `n` is the length of string `s`.

The recursive function `dfs(i, j)` explores all possible substrings of `s`. The memoization decorator `@cache` ensures each unique state `(i, j)` is computed only once. Since `i` ranges from `0` to `n-1` and `j` ranges from `0` to `n-1`, there are at most `n^2` unique states. Each state performs constant time operations (comparison and minimum calculation), resulting in `O(n^2)` time complexity.

**Space Complexity:** `O(n^2)`

The space complexity consists of two components:
1. **Memoization cache:** The `@cache` decorator stores up to `O(n^2)` unique states, each taking `O(1)` space to store the result.
2. **Recursion stack:** In the worst case, the recursion depth can be `O(n)` when we continuously move either `i` forward or `j` backward without finding matching characters.

The dominant factor is the memoization cache, giving us an overall space complexity of `O(n^2)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Base Case Handling**
A common mistake is using only `i == j` as the base case, forgetting that pointers can cross over (`i > j`) when processing even-length palindromic substrings.

**Incorrect:**
```python
if i == j:  # Wrong! Misses the i > j case
    return 0
```

**Correct:**
```python
if i >= j:  # Handles both single character and crossed pointers
    return 0
```

### 2. **Confusion About What We're Actually Inserting**
Many developers get confused about the insertion logic. When `s[i] != s[j]`, the recursive calls don't represent "inserting at position i or j" but rather:
- `dfs(i+1, j)`: We're planning to insert `s[i]` somewhere on the right side to match it
- `dfs(i, j-1)`: We're planning to insert `s[j]` somewhere on the left side to match it

**Mental Model Error:**
Thinking we need to track actual insertion positions or build the palindrome string.

**Solution:**
Focus on counting insertions only. The algorithm determines the minimum count without constructing the actual palindrome.

### 3. **Forgetting to Import or Apply Memoization**
Without memoization, the solution has exponential time complexity O(2^n), causing timeout on larger inputs.

**Incorrect:**
```python
def dfs(left, right):  # No memoization - will be too slow!
    if left >= right:
        return 0
    # ... rest of logic
```

**Correct:**
```python
from functools import cache

@cache  # Critical for performance!
def dfs(left, right):
    # ... implementation
```

### 4. **Using Wrong Indices in Recursive Calls**
A subtle but critical error is incrementing/decrementing the wrong pointer when characters don't match.

**Incorrect:**
```python
if s[left] != s[right]:
    # Wrong! Both moving the same direction
    return 1 + min(dfs(left + 1, right + 1), dfs(left - 1, right - 1))
```

**Correct:**
```python
if s[left] != s[right]:
    # One pointer moves inward while the other stays
    return 1 + min(dfs(left + 1, right), dfs(left, right - 1))
```

### 5. **Alternative But Incorrect DP Formulation**
Some might try to use a different DP state definition, like "minimum insertions to make s[0:i] a palindrome", which doesn't work because palindromes require matching from both ends.

**Why This Fails:**
Single-ended DP can't capture the two-pointer matching nature of palindrome checking.

**Solution:**
Always use two pointers (i, j) to represent the substring being processed, allowing comparison of characters from both ends.

