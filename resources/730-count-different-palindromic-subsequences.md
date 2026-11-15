# 730. Count Different Palindromic Subsequences

> Source: https://algo.monster/liteproblems/730

## Problem Statement

You are given a string `s` consisting of characters. Your task is to count the total number of distinct non-empty palindromic subsequences that can be formed from this string.

A **subsequence** is a sequence that can be derived from the original string by deleting some (possibly zero) characters without changing the order of the remaining characters. For example, "ace" is a subsequence of "abcde".

A **palindromic sequence** reads the same forwards and backwards. For example, "aba", "aa", and "a" are palindromes.

Two subsequences are considered **different** if they differ as strings, that is, if they differ in at least one position of the resulting sequence. Picking the same palindrome string from different character positions of the original string counts only once.

Since the count can be very large, you need to return the result modulo `10^9 + 7`.

**Key Points:**
- You need to find all possible palindromic subsequences (not substrings)
- Each distinct palindromic subsequence should be counted once
- The subsequences must be non-empty
- Return the count modulo `10^9 + 7`

**Example:**
If `s = "bccb"`, the palindromic subsequences are: "b", "c", "bb", "cc", "bcb", "bccb" - giving us a total count of 6.

## Intuition

When thinking about counting palindromic subsequences, we need to consider how palindromes are structured - they read the same forwards and backwards. This means that for any palindrome, the first and last characters must be the same.

The key insight is to use [dynamic programming](/problems/dynamic_programming_intro) with a focus on the **boundary characters**. For any substring `s[i...j]`, we can count palindromic subsequences based on what characters appear at the boundaries.

Since the problem mentions the string contains only lowercase letters (and from the solution we can see it's specifically 'a', 'b', 'c', 'd'), we can track palindromes ending with each specific character. This leads us to define our DP state as `dp[i][j][k]` where:
- `i` and `j` represent the substring boundaries
- `k` represents which character (0 for 'a', 1 for 'b', 2 for 'c', 3 for 'd') the palindrome starts and ends with

The critical observation is how to build larger palindromes from smaller ones:

1. **When `s[i] == s[j] == c`** (both boundaries match character `c`): We can form new palindromes by:
   - Taking just the character `c` itself (count = 1)
   - Taking `c + c` (count = 1)  
   - Taking any palindrome from the inner substring `s[i+1...j-1]` and wrapping it with `c` on both sides
 
   This gives us: `dp[i][j][k] = 2 + sum(all palindromes in dp[i+1][j-1])`

2. **When only `s[i] == c`**: The palindromes ending with `c` must start at position `i`, so we look at `s[i...j-1]`

3. **When only `s[j] == c`**: The palindromes ending with `c` must end at position `j`, so we look at `s[i+1...j]`

4. **When neither boundary is `c`**: Any palindrome with character `c` must be completely contained in the inner substring `s[i+1...j-1]`

By building up from single characters (base case) to larger substrings, we can count all distinct palindromic subsequences. The final answer is the sum of all palindromes in the entire string `s[0...n-1]` for all possible ending characters.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a 3D [dynamic programming](/problems/dynamic_programming_intro) approach to count palindromic subsequences. Let's walk through the implementation step by step:

**1. Initialization:**
```python
dp = [[[0] * 4 for _ in range(n)] for _ in range(n)]
```
We create a 3D DP table where `dp[i][j][k]` stores the count of distinct palindromic subsequences in substring `s[i...j]` that start and end with character `k` (where `k` maps to 'a', 'b', 'c', 'd' as 0, 1, 2, 3).

**2. Base Case:**
```python
for i, char in enumerate(s):
    char_index = ord(char) - ord('a')
    dp[i][i][char_index] = 1
```
Single characters are palindromes themselves. For each position `i`, we mark that there's exactly 1 palindrome of length 1 ending with that character.

**3. Building Up Solutions:**
```python
for length in range(2, n + 1):  # substring length
    for start in range(n - length + 1):  # starting position
        end = start + length - 1  # ending position
```
We iterate through all possible substring lengths from 2 to `n`, and for each length, we consider all possible starting positions.

**4. Core DP Transition:**
For each substring `s[i...j]` and each possible character `c` in 'abcd':

- **Case 1: `s[i] == s[j] == c`**
  ```python
  dp[start][end][char_index] = (2 + sum(dp[start + 1][end - 1])) % MOD
  ```
  When both boundaries match character `c`, we can form:
  - The single character `c` (count = 1)
  - The double character `cc` (count = 1)
  - Any palindrome from inner substring wrapped with `c` on both sides
  
  The result is reduced modulo `10^9 + 7` right away so the stored numbers stay small.

- **Case 2: `s[i] == c` but `s[j] != c`**
  ```python
  dp[start][end][char_index] = dp[start][end - 1][char_index]
  ```
  Palindromes with character `c` must start at position `i`, so we exclude the last character.

- **Case 3: `s[i] != c` but `s[j] == c`**
  ```python
  dp[start][end][char_index] = dp[start + 1][end][char_index]
  ```
  Palindromes with character `c` must end at position `j`, so we exclude the first character.

- **Case 4: Neither boundary is `c`**
  ```python
  dp[start][end][char_index] = dp[start + 1][end - 1][char_index]
  ```
  Any palindrome with character `c` must be completely within the inner substring.

**5. Final Result:**
```python
return sum(dp[0][n - 1]) % MOD
```
The total count is the sum of all palindromic subsequences in the entire string `s[0...n-1]` for all possible characters, taken modulo `10^9 + 7`.

**Time Complexity:** `O(n² × 4)` = `O(n²)` where n is the length of the string
**Space Complexity:** `O(n² × 4)` = `O(n²)` for the 3D DP table

## Example Walkthrough

Let's walk through a small example with `s = "bccb"` to illustrate the solution approach.

**Step 1: Initialize the DP table**
We create a 3D DP table `dp[i][j][k]` where indices represent:
- `i`, `j`: substring boundaries (0 to 3 for our string of length 4)
- `k`: character type (0='a', 1='b', 2='c', 3='d')

**Step 2: Base case - single characters**
```
s = "b c c b"
     0 1 2 3 (indices)
```
For each single character:
- Position 0: 'b' → `dp[0][0][1] = 1` (one palindrome "b")
- Position 1: 'c' → `dp[1][1][2] = 1` (one palindrome "c")
- Position 2: 'c' → `dp[2][2][2] = 1` (one palindrome "c")
- Position 3: 'b' → `dp[3][3][1] = 1` (one palindrome "b")

**Step 3: Build substrings of length 2**

For substring "bc" (i=0, j=1):
- For character 'b': s[0]='b', s[1]='c' → only left boundary matches
  - `dp[0][1][1] = dp[0][0][1] = 1` (palindrome "b")
- For character 'c': s[0]='b', s[1]='c' → only right boundary matches
  - `dp[0][1][2] = dp[1][1][2] = 1` (palindrome "c")

For substring "cc" (i=1, j=2):
- For character 'c': s[1]='c', s[2]='c' → both boundaries match!
  - `dp[1][2][2] = 2 + sum(dp[2][1])` 
  - Since i+1 > j-1, inner substring is empty, sum = 0
  - `dp[1][2][2] = 2` (palindromes "c" and "cc")

For substring "cb" (i=2, j=3):
- For character 'c': s[2]='c', s[3]='b' → only left boundary matches
  - `dp[2][3][2] = dp[2][2][2] = 1` (palindrome "c")
- For character 'b': s[2]='c', s[3]='b' → only right boundary matches
  - `dp[2][3][1] = dp[3][3][1] = 1` (palindrome "b")

**Step 4: Build substrings of length 3**

For substring "bcc" (i=0, j=2):
- For character 'b': s[0]='b', s[2]='c' → only left boundary matches
  - `dp[0][2][1] = dp[0][1][1] = 1` (palindrome "b")
- For character 'c': s[0]='b', s[2]='c' → only right boundary matches
  - `dp[0][2][2] = dp[1][2][2] = 2` (palindromes "c", "cc")

For substring "ccb" (i=1, j=3):
- For character 'c': s[1]='c', s[3]='b' → only left boundary matches
  - `dp[1][3][2] = dp[1][2][2] = 2` (palindromes "c", "cc")
- For character 'b': s[1]='c', s[3]='b' → only right boundary matches
  - `dp[1][3][1] = dp[2][3][1] = 1` (palindrome "b")

**Step 5: Build substring of length 4 (entire string)**

For substring "bccb" (i=0, j=3):
- For character 'b': s[0]='b', s[3]='b' → both boundaries match!
  - Inner substring is "cc" (i+1=1, j-1=2)
  - Sum of inner palindromes = `dp[1][2][2] = 2`
  - `dp[0][3][1] = 2 + 2 = 4` (palindromes "b", "bb", "bcb", "bccb")
- For character 'c': s[0]='b', s[3]='b' → neither boundary matches
  - `dp[0][3][2] = dp[1][2][2] = 2` (palindromes "c", "cc")

**Step 6: Calculate final result**
Total palindromic subsequences = sum of all characters for entire string:
- `dp[0][3][0]` (character 'a') = 0
- `dp[0][3][1]` (character 'b') = 4
- `dp[0][3][2]` (character 'c') = 2  
- `dp[0][3][3]` (character 'd') = 0

**Total = 0 + 4 + 2 + 0 = 6**

The 6 distinct palindromic subsequences are: "b", "c", "bb", "cc", "bcb", "bccb"

## Python Solution

```python
class Solution:
    def countPalindromicSubsequences(self, s: str) -> int:
        MOD = 10**9 + 7
        n = len(s)
      
        # dp[i][j][char_idx] = number of distinct palindromic subsequences 
        # in s[i:j+1] that start and end with character at index char_idx
        # where char_idx: 0='a', 1='b', 2='c', 3='d'
        dp = [[[0] * 4 for _ in range(n)] for _ in range(n)]
      
        # Base case: single characters are palindromes
        for i, char in enumerate(s):
            char_index = ord(char) - ord('a')
            dp[i][i][char_index] = 1
      
        # Process substrings of increasing length
        for length in range(2, n + 1):
            for start in range(n - length + 1):
                end = start + length - 1
              
                # Try each possible character as start/end of palindrome
                for char in 'abcd':
                    char_index = ord(char) - ord('a')
                  
                    if s[start] == s[end] == char:
                        # Both ends match the current character
                        # Count: "c" + "cc" + every inner palindrome wrapped as c...c
                        dp[start][end][char_index] = (2 + sum(dp[start + 1][end - 1])) % MOD
                    elif s[start] == char:
                        # Only start matches - exclude the end character
                        dp[start][end][char_index] = dp[start][end - 1][char_index]
                    elif s[end] == char:
                        # Only end matches - exclude the start character
                        dp[start][end][char_index] = dp[start + 1][end][char_index]
                    else:
                        # Neither end matches - look at inner substring
                        dp[start][end][char_index] = dp[start + 1][end - 1][char_index]
      
        # Return total count of distinct palindromic subsequences
        return sum(dp[0][n - 1]) % MOD
```

## Time and Space Complexity

**Time Complexity:** `O(n² × 4)` = `O(n²)`

The algorithm uses dynamic programming with three nested loops:
- The outer loop iterates through all possible substring lengths from 2 to n: `O(n)` iterations
- The middle loop iterates through all possible starting positions for substrings of length l: `O(n)` iterations 
- The inner loop iterates through all 4 possible characters ('a', 'b', 'c', 'd'): `O(4)` = `O(1)` iterations

Within the innermost loop, the operations are mostly `O(1)`, except for one case where `sum(dp[i + 1][j - 1])` is called, which takes `O(4)` = `O(1)` time since it sums over 4 elements.

Therefore, the total time complexity is `O(n × n × 4)` = `O(n²)`.

**Space Complexity:** `O(n² × 4)` = `O(n²)`

The algorithm uses a 3-dimensional DP table:
- First dimension represents the starting index: `n` positions
- Second dimension represents the ending index: `n` positions  
- Third dimension represents the character type (a, b, c, or d): `4` values

The total space required is `n × n × 4` = `O(n²)` since 4 is a constant.

No additional significant auxiliary space is used beyond the DP table and a few variables, so the overall space complexity remains `O(n²)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Modulo Application**
A common pitfall is applying the modulo operation only at the final return statement. The true counts grow exponentially with `n`, so intermediate values overflow in languages with fixed-size integers (Java, C++, TypeScript). In Python the result stays correct, but the numbers grow to hundreds of bits and every addition becomes slower.

**Problem Code:**
```python
dp[start][end][char_index] = 2 + sum(dp[start + 1][end - 1])
# ... other assignments without modulo
return sum(dp[0][n - 1]) % MOD
```

**Solution:**
Apply the modulo operation in the one case that adds values, as the solution does. The other three cases copy a value that is already reduced, so they need no extra modulo:
```python
if s[start] == s[end] == char:
    dp[start][end][char_index] = (2 + sum(dp[start + 1][end - 1])) % MOD
```

### 2. **Reading the Empty Inner Substring for Length 2**
When computing `dp[start + 1][end - 1]` for substrings of length 2, we get `start + 1 > end - 1`, which describes an empty inner substring.

**What Happens in the Solution:**
When `length = 2`, we have `end = start + 1`, so `dp[start + 1][end - 1]` is `dp[end][start]`. The solution never writes to cells with first index greater than second index, so this cell keeps its initial value of all zeros. That is exactly the count for an empty substring, so the solution handles this case correctly without a special branch. The pitfall appears only in variants that fill the table differently (for example, reusing memory or using a memoized recursion that does not return 0 for `i > j`).

**Explicit Alternative:**
Some implementations make the empty case explicit for readability:
```python
if s[start] == s[end] == char:
    if length == 2:
        # For length 2, we have two palindromes: single char and double char
        dp[start][end][char_index] = 2
    else:
        # For longer strings, include inner palindromes
        inner_sum = sum(dp[start + 1][end - 1]) % MOD
        dp[start][end][char_index] = (2 + inner_sum) % MOD
```

### 3. **Assumption About Character Set**
The code assumes the input string contains only characters 'a', 'b', 'c', 'd'. If the string contains other characters, the solution will fail or produce incorrect results.

**Problem:**
```python
for char in 'abcd':  # Hard-coded character set
    char_index = ord(char) - ord('a')
```

**Solution:**
Either validate the input or dynamically determine the character set:
```python
# Option 1: Validate input
unique_chars = set(s)
if not unique_chars.issubset({'a', 'b', 'c', 'd'}):
    raise ValueError("String must contain only characters a, b, c, d")

# Option 2: Dynamic character set
unique_chars = sorted(set(s))
char_to_index = {char: i for i, char in enumerate(unique_chars)}
dp = [[[0] * len(unique_chars) for _ in range(n)] for _ in range(n)]
```

### Solution with Explicit Edge Cases:
```python
class Solution:
    def countPalindromicSubsequences(self, s: str) -> int:
        MOD = 10**9 + 7
        n = len(s)
      
        dp = [[[0] * 4 for _ in range(n)] for _ in range(n)]
      
        # Base case
        for i, char in enumerate(s):
            char_index = ord(char) - ord('a')
            dp[i][i][char_index] = 1
      
        # Process substrings
        for length in range(2, n + 1):
            for start in range(n - length + 1):
                end = start + length - 1
              
                for char in 'abcd':
                    char_index = ord(char) - ord('a')
                  
                    if s[start] == s[end] == char:
                        if length == 2:
                            dp[start][end][char_index] = 2
                        else:
                            inner_sum = sum(dp[start + 1][end - 1]) % MOD
                            dp[start][end][char_index] = (2 + inner_sum) % MOD
                    elif s[start] == char:
                        dp[start][end][char_index] = dp[start][end - 1][char_index]
                    elif s[end] == char:
                        dp[start][end][char_index] = dp[start + 1][end][char_index]
                    else:
                        if length > 2:
                            dp[start][end][char_index] = dp[start + 1][end - 1][char_index]
                        # else: remains 0 for length 2 when neither end matches
      
        return sum(dp[0][n - 1]) % MOD
```

