# 1092. Shortest Common Supersequence

> Source: https://algo.monster/liteproblems/1092

## Problem Statement

You are given two strings `str1` and `str2`. Your task is to find the shortest string that contains both `str1` and `str2` as subsequences.

A subsequence is formed by deleting some (possibly zero) characters from a string without changing the order of the remaining characters. For example, `"ace"` is a subsequence of `"abcde"` (delete `'b'` and `'d'`).

The goal is to construct the shortest possible string where:
- If you delete some characters from this string, you can get `str1`
- If you delete some (other) characters from this string, you can get `str2`

If multiple valid shortest strings exist, you can return any one of them.

**Example:**
- If `str1 = "abac"` and `str2 = "cab"`, one possible shortest common supersequence is `"cabac"`.
  - Deleting the second `'c'` gives `"abac"` (which is `str1`)
  - Deleting the first `'a'` and last `'a'` and `'c'` gives `"cab"` (which is `str2`)

## Intuition

The key insight is recognizing that the shortest common supersequence is closely related to the Longest Common Subsequence (LCS) of the two strings.

Think about it this way: if two strings share some common characters in the same order, we only need to include those shared characters once in our result. The more characters they share (i.e., the longer their LCS), the shorter our final supersequence will be.

For example, if `str1 = "abac"` and `str2 = "cab"`, their LCS is `"ab"`. To build the shortest supersequence:
- We must include all characters from both strings
- But we can "merge" the common subsequence characters instead of duplicating them

The relationship can be expressed as:
`Length of shortest supersequence = len(str1) + len(str2) - len(LCS)`

This is because:
- We need all characters from `str1` (length `m`)
- We need all characters from `str2` (length `n`) 
- But the LCS characters are counted twice, so we subtract them once

Once we understand this relationship, the solution strategy becomes clear:
1. First, find the LCS using [dynamic programming](/problems/dynamic_programming_intro)
2. Then, reconstruct the actual supersequence by backtracking through the DP table

During reconstruction, we traverse backwards through the DP table:
- When characters match (part of LCS), we include that character once
- When they don't match, we need to include the character from whichever string would maintain the subsequence property
- At boundaries (when one string is exhausted), we include all remaining characters from the other string

This greedy reconstruction ensures we build the shortest possible string that contains both original strings as subsequences.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation consists of two main phases: building the LCS table and reconstructing the shortest supersequence.

**Phase 1: Building the LCS [Dynamic Programming](/problems/dynamic_programming_intro) Table**

We create a 2D table `f[i][j]` where `f[i][j]` represents the length of the LCS between the first `i` characters of `str1` and the first `j` characters of `str2`.

```python
f = [[0] * (n + 1) for _ in range(m + 1)]
```

The table is filled using the recurrence relation:
- If `str1[i-1] == str2[j-1]`: `f[i][j] = f[i-1][j-1] + 1` (characters match, extend LCS)
- Otherwise: `f[i][j] = max(f[i-1][j], f[i][j-1])` (take the better LCS without current character)

**Phase 2: Backtracking to Build the Result**

Starting from `f[m][n]` (bottom-right corner), we trace back to reconstruct the shortest supersequence:

```python
i, j = m, n
ans = []
```

The backtracking logic follows these rules:

1. **Boundary cases**: 
   - If `i == 0`: We've exhausted `str1`, so add remaining characters from `str2[j]`
   - If `j == 0`: We've exhausted `str2`, so add remaining characters from `str1[i]`

2. **Non-boundary cases**:
   - If `f[i][j] == f[i-1][j]`: The LCS value came from excluding `str1[i-1]`, so we add `str1[i-1]` to result and move to `(i-1, j)`
   - Else if `f[i][j] == f[i][j-1]`: The LCS value came from excluding `str2[j-1]`, so we add `str2[j-1]` to result and move to `(i, j-1)`
   - Else: Characters matched (part of LCS), add the character once and move diagonally to `(i-1, j-1)`

Since we build the result by traversing backwards, we reverse the final array:

```python
return ''.join(ans[::-1])
```

**Time Complexity**: `O(m * n)` for building the DP table and `O(m + n)` for reconstruction
**Space Complexity**: `O(m * n)` for the DP table

The algorithm guarantees the shortest supersequence because it includes each LCS character exactly once while preserving all non-LCS characters from both strings in their required order.

## Example Walkthrough

Let's walk through the algorithm with `str1 = "abc"` and `str2 = "adc"`.

**Step 1: Build the LCS DP Table**

We create a table `f[i][j]` where `f[i][j]` represents the LCS length for the first `i` characters of `str1` and first `j` characters of `str2`.

```
    ""  a  d  c
""   0  0  0  0
a    0  1  1  1
b    0  1  1  1
c    0  1  1  2
```

Building process:
- `f[1][1]`: `str1[0]='a'` == `str2[0]='a'`, so `f[1][1] = f[0][0] + 1 = 1`
- `f[1][2]`: `str1[0]='a'` != `str2[1]='d'`, so `f[1][2] = max(f[0][2], f[1][1]) = 1`
- `f[2][1]`: `str1[1]='b'` != `str2[0]='a'`, so `f[2][1] = max(f[1][1], f[2][0]) = 1`
- `f[3][3]`: `str1[2]='c'` == `str2[2]='c'`, so `f[3][3] = f[2][2] + 1 = 2`

The LCS length is 2 (characters 'a' and 'c').

**Step 2: Backtrack to Build Result**

Starting from `(i=3, j=3)`, we trace backwards:

1. **At (3,3)**: `f[3][3]=2` != `f[2][3]=1` and != `f[3][2]=1`
   - This means `str1[2]='c'` and `str2[2]='c'` match
   - Add 'c' to result: `ans = ['c']`
   - Move diagonally to (2,2)

2. **At (2,2)**: `f[2][2]=1` == `f[2][1]=1`
   - The LCS value came from excluding `str2[1]='d'`
   - Add 'd' to result: `ans = ['c', 'd']`
   - Move left to (2,1)

3. **At (2,1)**: `f[2][1]=1` == `f[1][1]=1`
   - The LCS value came from excluding `str1[1]='b'`
   - Add 'b' to result: `ans = ['c', 'd', 'b']`
   - Move up to (1,1)

4. **At (1,1)**: `f[1][1]=1` != `f[0][1]=0` and != `f[1][0]=0`
   - This means `str1[0]='a'` and `str2[0]='a'` match
   - Add 'a' to result: `ans = ['c', 'd', 'b', 'a']`
   - Move diagonally to (0,0)

5. **At (0,0)**: Both strings exhausted, stop.

**Step 3: Reverse the Result**

Since we built backwards, reverse the array: `['c', 'd', 'b', 'a']` → `"adbc"`

**Verification:**
- `"adbc"` contains `"abc"` as subsequence (delete 'd')
- `"adbc"` contains `"adc"` as subsequence (delete 'b')
- Length is 4, which equals `len("abc") + len("adc") - LCS_length = 3 + 3 - 2 = 4` ✓

The shortest common supersequence is `"adbc"`.

## Python Solution

```python
class Solution:
    def shortestCommonSupersequence(self, str1: str, str2: str) -> str:
        # Get lengths of both strings
        len1, len2 = len(str1), len(str2)
      
        # Create DP table to store length of longest common subsequence (LCS)
        # dp[i][j] represents LCS length of str1[0:i] and str2[0:j]
        dp = [[0] * (len2 + 1) for _ in range(len1 + 1)]
      
        # Fill the DP table to find LCS lengths
        for i in range(1, len1 + 1):
            for j in range(1, len2 + 1):
                if str1[i - 1] == str2[j - 1]:
                    # Characters match, extend the LCS
                    dp[i][j] = dp[i - 1][j - 1] + 1
                else:
                    # Characters don't match, take maximum from previous states
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
      
        # Build the shortest common supersequence by backtracking through DP table
        result = []
        i, j = len1, len2
      
        while i > 0 or j > 0:
            if i == 0:
                # Reached end of str1, add remaining characters from str2
                j -= 1
                result.append(str2[j])
            elif j == 0:
                # Reached end of str2, add remaining characters from str1
                i -= 1
                result.append(str1[i])
            else:
                # Both strings have remaining characters
                if dp[i][j] == dp[i - 1][j]:
                    # Current character from str1 is not part of LCS
                    i -= 1
                    result.append(str1[i])
                elif dp[i][j] == dp[i][j - 1]:
                    # Current character from str2 is not part of LCS
                    j -= 1
                    result.append(str2[j])
                else:
                    # Characters match and are part of LCS, add once
                    i -= 1
                    j -= 1
                    result.append(str1[i])
      
        # Reverse the result since we built it backwards
        return ''.join(reversed(result))
```

## Time and Space Complexity

**Time Complexity:** `O(m * n + m + n)` which simplifies to `O(m * n)`

The algorithm consists of two main phases:

1. **Building the DP table:** The nested loops iterate through all cells of the `f` table, where each cell operation takes `O(1)` time. This results in `O(m * n)` time complexity.

2. **Reconstructing the shortest common supersequence:** The while loop backtracks through the DP table from position `(m, n)` to `(0, 0)`. In the worst case, we traverse at most `m + n` steps (moving either up or left in each iteration). Each operation within the loop takes `O(1)` time. This phase has `O(m + n)` time complexity.

The overall time complexity is `O(m * n) + O(m + n) = O(m * n)` since `m * n` dominates for non-trivial inputs.

**Space Complexity:** `O(m * n)`

The space usage breaks down as follows:

1. **DP table `f`:** A 2D array of size `(m + 1) × (n + 1)` storing the longest common subsequence lengths, requiring `O(m * n)` space.

2. **Result list `ans`:** In the worst case where the two strings have no common characters, the shortest common supersequence has length `m + n`, requiring `O(m + n)` space.

3. **Other variables:** `i`, `j`, `m`, `n` use `O(1)` space.

The total space complexity is `O(m * n) + O(m + n) + O(1) = O(m * n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrect Backtracking Order

**The Problem:**
A common mistake is incorrectly checking the DP table conditions during backtracking. Specifically, developers often check `dp[i][j] == dp[i][j-1]` before `dp[i][j] == dp[i-1][j]`, which can lead to an incorrect supersequence that doesn't maintain the proper character order.

**Why It Happens:**
The order matters because when both `dp[i-1][j]` and `dp[i][j-1]` equal `dp[i][j]`, we have a choice. The algorithm must consistently follow one path to ensure correctness.

**Incorrect Implementation:**
```python
if dp[i][j] == dp[i][j - 1]:  # Checking this first
    j -= 1
    result.append(str2[j])
elif dp[i][j] == dp[i - 1][j]:  # Checking this second
    i -= 1
    result.append(str1[i])
```

**Solution:**
Always check `dp[i-1][j]` first, then `dp[i][j-1]`. This ensures a consistent traversal pattern:
```python
if dp[i][j] == dp[i - 1][j]:
    i -= 1
    result.append(str1[i])
elif dp[i][j] == dp[i][j - 1]:
    j -= 1
    result.append(str2[j])
else:
    # Characters match (part of LCS)
    i -= 1
    j -= 1
    result.append(str1[i])
```

### Pitfall 2: Off-by-One Errors in Index Management

**The Problem:**
When backtracking, it's easy to confuse whether to use the character at index `i` or `i-1` after decrementing. Since the DP table uses 1-based indexing for convenience but strings use 0-based indexing, this mismatch can cause bugs.

**Incorrect Implementation:**
```python
# Wrong: Using index before decrementing
result.append(str1[i - 1])
i -= 1
```

**Solution:**
Always decrement the index first, then use it to access the string:
```python
# Correct: Decrement first, then use
i -= 1
result.append(str1[i])  # Now i is already decremented
```

### Pitfall 3: Forgetting to Handle Empty String Cases

**The Problem:**
If either `str1` or `str2` is empty, the algorithm should simply return the other string. Not handling this edge case explicitly can lead to unnecessary computation or errors.

**Solution:**
Add early return conditions:
```python
def shortestCommonSupersequence(self, str1: str, str2: str) -> str:
    if not str1:
        return str2
    if not str2:
        return str1
  
    # Continue with the main algorithm...
```

### Pitfall 4: Building Result in Wrong Direction

**The Problem:**
During backtracking, we traverse from `(m, n)` to `(0, 0)`, which means we're building the result string backwards. Forgetting to reverse the final result is a common oversight.

**Incorrect Implementation:**
```python
return ''.join(result)  # Forgot to reverse!
```

**Solution:**
Always reverse the result before returning:
```python
return ''.join(reversed(result))
# or
return ''.join(result[::-1])
```

