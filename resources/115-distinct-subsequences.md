# 115. Distinct Subsequences

> Source: https://algo.monster/liteproblems/115

## Problem Statement

Given two strings `s` and `t`, you need to find the number of distinct subsequences of string `s` that equal string `t`.

A subsequence is a sequence that can be derived from the original string by deleting some (or no) characters without changing the order of the remaining characters. For example, "ace" is a subsequence of "aec**ace**".

The task is to count how many different ways you can form string `t` by selecting characters from string `s` in their original order.

For example:
- If `s = "rabbbit"` and `t = "rabbit"`, there are 3 distinct ways to form "rabbit" from "rabbbit" by choosing different combinations of the letter 'b'.
- If `s = "babgbag"` and `t = "bag"`, there are 5 distinct ways to form "bag" from "babgbag".

The problem uses dynamic programming where `f[i][j]` represents the number of ways to form the first `j` characters of string `t` using the first `i` characters of string `s`.

The base case is `f[i][0] = 1` for all `i`, meaning there's exactly one way to form an empty string (by selecting nothing).

For each position, if the characters match (`s[i-1] == t[j-1]`), we have two choices:
- Use the current character from `s`: add the count from `f[i-1][j-1]`
- Skip the current character from `s`: take the count from `f[i-1][j]`

If the characters don't match, we can only skip the current character from `s`, so `f[i][j] = f[i-1][j]`.

The final answer is `f[m][n]`, where `m` and `n` are the lengths of strings `s` and `t` respectively.

## Intuition

The key insight is to think about this problem as making choices at each character position. When we're trying to match string `t` using characters from string `s`, we're essentially asking: "How many ways can I select characters from `s` to form `t`?"

Let's think step by step. Imagine we're building `t` character by character from `s`. At any point, we've processed some portion of both strings. The critical observation is that when we encounter a character in `s`, we face a decision:

1. If the current character in `s` matches the current character we need in `t`, we have two options:
   - Use this character as part of our subsequence
   - Skip it and look for another matching character later

2. If the characters don't match, we have no choice but to skip the current character in `s`.

This naturally leads to a dynamic programming approach. We can build up our solution by considering smaller subproblems first. The question "How many ways can we form the first `j` characters of `t` using the first `i` characters of `s`?" can be answered by looking at smaller versions of the same problem.

The beauty of this approach is that it captures the overlapping nature of the problem. When we have multiple occurrences of the same character in `s` (like multiple 'b's in "rabbbit"), each occurrence creates a different path to form `t`. By systematically considering each character position and accumulating the counts, we avoid counting the same subsequence twice while ensuring we count all possible distinct subsequences.

The base case makes intuitive sense too: there's exactly one way to form an empty string from any string - by selecting nothing. This gives us our starting point to build up the solution for longer substrings.

## Solution Approach

We implement the solution using a 2D dynamic programming table `f[i][j]` where `i` ranges from `0` to `m` (length of string `s`) and `j` ranges from `0` to `n` (length of string `t`). This table stores the number of ways to form the first `j` characters of `t` using the first `i` characters of `s`.

**Initialization:**
- Create a 2D array `f` with dimensions `(m+1) × (n+1)`, initialized with zeros
- Set `f[i][0] = 1` for all `i` from `0` to `m`, representing that there's exactly one way to form an empty string

**Filling the DP Table:**
We iterate through each character of `s` (index `i` from `1` to `m`) and for each position, we iterate through each character of `t` (index `j` from `1` to `n`).

For each cell `f[i][j]`, we apply the state transition formula:

$$
f[i][j]=\left\{
\begin{aligned}
&f[i-1][j], &s[i-1] \ne t[j-1] \\
&f[i-1][j-1]+f[i-1][j], &s[i-1]=t[j-1]
\end{aligned}
\right.
$$

Breaking this down:
- `f[i-1][j]` represents not using the current character `s[i-1]`, which is always an option
- When `s[i-1] == t[j-1]`, we additionally add `f[i-1][j-1]`, which represents using the current character to match

**Implementation Details:**
```python
# Initialize base cases
for i in range(m + 1):
    f[i][0] = 1

# Fill the DP table
for i, a in enumerate(s, 1):  # Start from index 1
    for j, b in enumerate(t, 1):  # Start from index 1
        f[i][j] = f[i - 1][j]  # Don't use s[i-1]
        if a == b:  # Characters match
            f[i][j] += f[i - 1][j - 1]  # Also add the option of using s[i-1]
```

The enumerate function with starting value 1 elegantly handles the 1-based indexing for the DP table while iterating through 0-based string indices.

**Result:**
The final answer is stored in `f[m][n]`, representing the total number of distinct subsequences of the entire string `s` that equal the entire string `t`.

**Time and Space Complexity:**
- Time Complexity: `O(m × n)` as we fill each cell of the DP table exactly once
- Space Complexity: `O(m × n)` for storing the DP table

## Example Walkthrough

Let's walk through a small example with `s = "babgbag"` and `t = "bag"` to see how the dynamic programming solution works.

We'll build a DP table where `f[i][j]` represents the number of ways to form the first `j` characters of `t` using the first `i` characters of `s`.

**Step 1: Initialize the table**
- Create a table with dimensions (8 × 4) since len(s) = 7 and len(t) = 3
- Set `f[i][0] = 1` for all rows (one way to form empty string)

Initial table:
```
    ""  b  a  g
""   1  0  0  0
b    1  0  0  0
a    1  0  0  0
b    1  0  0  0
g    1  0  0  0
b    1  0  0  0
a    1  0  0  0
g    1  0  0  0
```

**Step 2: Fill the table row by row**

For `i=1, s[0]='b'`:
- `j=1, t[0]='b'`: Characters match! `f[1][1] = f[0][1] + f[0][0] = 0 + 1 = 1`
- `j=2, t[1]='a'`: No match. `f[1][2] = f[0][2] = 0`
- `j=3, t[2]='g'`: No match. `f[1][3] = f[0][3] = 0`

For `i=2, s[1]='a'`:
- `j=1, t[0]='b'`: No match. `f[2][1] = f[1][1] = 1`
- `j=2, t[1]='a'`: Match! `f[2][2] = f[1][2] + f[1][1] = 0 + 1 = 1`
- `j=3, t[2]='g'`: No match. `f[2][3] = f[1][3] = 0`

For `i=3, s[2]='b'`:
- `j=1, t[0]='b'`: Match! `f[3][1] = f[2][1] + f[2][0] = 1 + 1 = 2`
- `j=2, t[1]='a'`: No match. `f[3][2] = f[2][2] = 1`
- `j=3, t[2]='g'`: No match. `f[3][3] = f[2][3] = 0`

Continuing this process for all positions...

**Final table:**
```
    ""  b  a  g
""   1  0  0  0
b    1  1  0  0
a    1  1  1  0
b    1  2  1  0
g    1  2  1  1
b    1  3  1  1
a    1  3  4  1
g    1  3  4  5
```

**Step 3: Read the answer**
The value at `f[7][3] = 5` tells us there are 5 distinct ways to form "bag" from "babgbag".

These 5 ways correspond to the following character positions in `s` (1-indexed):
1. **ba**b**g**bag (positions 1, 2, 4)
2. **ba**bgba**g** (positions 1, 2, 7)
3. **b**abgb**ag** (positions 1, 6, 7)
4. ba**b**gb**ag** (positions 3, 6, 7)
5. babg**bag** (positions 5, 6, 7)

The key insight: at each position where characters match, we accumulate counts from two sources:
- Not using the current character (inherit from `f[i-1][j]`)
- Using the current character (add `f[i-1][j-1]`)

This ensures we count all distinct subsequences exactly once.

## Python Solution

```python
class Solution:
    def numDistinct(self, s: str, t: str) -> int:
        """
        Count the number of distinct subsequences of s that equal t.
        Dynamic Programming approach.
      
        Args:
            s: The source string
            t: The target string to match as subsequence
          
        Returns:
            Number of distinct subsequences of s that equal t
        """
        source_len, target_len = len(s), len(t)
      
        # Create DP table where dp[i][j] represents the number of ways
        # to form t[0:j] using s[0:i]
        # dp[i][j] = number of distinct subsequences of s[0:i] that equals t[0:j]
        dp = [[0] * (target_len + 1) for _ in range(source_len + 1)]
      
        # Base case: empty target string can be formed in exactly one way
        # (by selecting nothing from source string)
        for i in range(source_len + 1):
            dp[i][0] = 1
      
        # Fill the DP table
        for i, source_char in enumerate(s, 1):
            for j, target_char in enumerate(t, 1):
                # Case 1: Don't use current character from source string
                # Inherit the count from previous source position
                dp[i][j] = dp[i - 1][j]
              
                # Case 2: If characters match, we can also use current character
                # Add the count from diagonal (both strings reduced by 1)
                if source_char == target_char:
                    dp[i][j] += dp[i - 1][j - 1]
      
        # Return the final result: ways to form entire t using entire s
        return dp[source_len][target_len]
```

## Time and Space Complexity

**Time Complexity:** `O(m * n)`, where `m` is the length of string `s` and `n` is the length of string `t`. The algorithm uses two nested loops that iterate through all characters of both strings, resulting in `m * n` iterations. Each iteration performs constant time operations (array access and addition).

**Space Complexity:** `O(m * n)` for the current implementation, which uses a 2D array `f` of size `(m + 1) × (n + 1)` to store the dynamic programming states.

**Space Optimization:** Since the calculation of `f[i][j]` only depends on values from the previous row (`f[i-1][j]` and `f[i-1][j-1]`), the space complexity can be optimized to `O(n)` by using a 1D array instead of a 2D array. This would involve maintaining only the current and previous rows' values, or using a single row with careful updating from right to left or using temporary variables.

## Common Pitfalls

### 1. Integer Overflow in Languages with Fixed Integer Size

**The Pitfall:**
When dealing with large strings, the number of distinct subsequences can grow exponentially. For instance, if `s` contains many repeated characters that match characters in `t`, the count can become extremely large. In languages like Java or C++ with fixed integer sizes (int: 32-bit, long: 64-bit), this can lead to integer overflow, producing incorrect results.

**Example scenario:**
```python
s = "a" * 1000  # 1000 'a's
t = "a" * 100   # 100 'a's
# The result would be C(1000, 100) which is astronomically large
```

**Solution:**
- In Python, integers have arbitrary precision, so overflow isn't an issue
- LeetCode guarantees that the final answer fits in a 32-bit signed integer, but intermediate cells such as `dp[i][j]` for a short prefix of `t` can be far larger, larger even than 64 bits, so switching to `long` alone is not enough in general
- Because only additions are involved, wrap-around arithmetic still gives the right final value: an `unsigned long long` table in C++ (as in the C++ solution above) or a plain `int` table in Java computes every cell modulo `2^64` or `2^32`, and the final answer, which is below `2^31`, comes out exact
- Some variants ask for the result modulo a prime number (e.g., 10^9 + 7). In that case, apply the modulo at each step:
```python
MOD = 10**9 + 7
if source_char == target_char:
    dp[i][j] = (dp[i][j] + dp[i - 1][j - 1]) % MOD
```

### 2. Incorrect Base Case Initialization

**The Pitfall:**
A common mistake is forgetting to initialize `dp[0][0] = 1` or only initializing `dp[i][0] = 1` for `i > 0`, missing the case when both strings are empty. This leads to incorrect calculations throughout the entire DP table.

**Incorrect initialization:**
```python
# Wrong: Starting from 1 instead of 0
for i in range(1, source_len + 1):
    dp[i][0] = 1  # Misses dp[0][0] = 1
```

**Solution:**
Always initialize from index 0:
```python
# Correct: Include all rows from 0 to source_len
for i in range(source_len + 1):
    dp[i][0] = 1
```

### 3. Off-by-One Errors in Index Mapping

**The Pitfall:**
Confusion between DP table indices (1-based) and string indices (0-based) often leads to accessing wrong characters. Using `s[i]` instead of `s[i-1]` when `i` represents the DP table index is a frequent error.

**Incorrect character access:**
```python
# Wrong: Using i and j directly as string indices
if s[i] == t[j]:  # Should be s[i-1] == t[j-1]
    dp[i][j] += dp[i - 1][j - 1]
```

**Solution:**
Use enumerate with start=1 for cleaner code:
```python
# Clear and correct: enumerate handles the offset
for i, source_char in enumerate(s, 1):
    for j, target_char in enumerate(t, 1):
        dp[i][j] = dp[i - 1][j]
        if source_char == target_char:
            dp[i][j] += dp[i - 1][j - 1]
```

### 4. Space Optimization Pitfall

**The Pitfall:**
When optimizing space from O(m×n) to O(n) using a 1D array, updating values in the wrong order can overwrite values needed for subsequent calculations.

**Incorrect space optimization:**
```python
# Wrong: Left-to-right iteration overwrites needed values
dp = [1] + [0] * target_len
for source_char in s:
    for j in range(1, target_len + 1):  # Wrong direction!
        if source_char == t[j - 1]:
            dp[j] += dp[j - 1]  # dp[j-1] might be already updated
```

**Solution:**
Iterate from right to left when using 1D array:
```python
# Correct: Right-to-left preserves needed values
dp = [1] + [0] * target_len
for source_char in s:
    for j in range(target_len, 0, -1):  # Reverse iteration
        if source_char == t[j - 1]:
            dp[j] += dp[j - 1]  # dp[j-1] still has value from previous row
```

