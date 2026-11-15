# 940. Distinct Subsequences II

> Source: https://algo.monster/liteproblems/940

## Problem Statement

You are given a string `s` and need to find the total number of **distinct non-empty subsequences** that can be formed from it.

A subsequence is created by removing some (possibly zero) characters from the original string while keeping the remaining characters in their original order. For example, from the string `"abcde"`, we can form `"ace"` by removing `'b'` and `'d'`, but `"aec"` is not a valid subsequence because it changes the relative order of characters.

The key requirements are:
- Count only **distinct** subsequences (no duplicates)
- The subsequences must be **non-empty** (at least one character)
- Return the answer **modulo** `10^9 + 7` since the count can be very large

For example, if `s = "abc"`, the distinct non-empty subsequences are: `"a"`, `"b"`, `"c"`, `"ab"`, `"ac"`, `"bc"`, and `"abc"`, giving us a total of 7.

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with a 2D array `dp[i][j]` where:
- `i` represents the position in the string (from 0 to n)
- `j` represents each of the 26 lowercase letters
- `dp[i][j]` stores the count of distinct subsequences ending with character `j` using the first `i` characters of the string

For each character in the string, the algorithm updates the count of subsequences ending with that character by summing all previous subsequences and adding 1 (for the single character subsequence). The final answer is the sum of all distinct subsequences ending with any character.

## Intuition

The key insight is to track subsequences by their ending character to avoid counting duplicates. When we encounter a character, we need to decide how it affects our count of distinct subsequences.

Consider what happens when we process each character in the string. For any character `c` at position `i`, we can:
1. Start a new subsequence with just this character
2. Append this character to all existing subsequences

The challenge is handling duplicates. If we've seen the character `c` before, simply adding it to all existing subsequences might create duplicates. For example, in `"aba"`, when we reach the second `'a'`, we could form `"aa"` in two ways: by adding the second `'a'` to the first `'a'`, or by taking both `'a'`s directly.

To solve this, we track subsequences by their ending character. The brilliant idea is that if we know how many distinct subsequences end with each letter after processing `i-1` characters, we can compute the count after processing the `i`-th character.

When we see character `c` at position `i`:
- All previous subsequences (regardless of their ending character) can be extended by appending `c`, creating new subsequences ending with `c`
- We also create one new subsequence consisting of just `c` itself
- The total number of subsequences ending with `c` becomes: `sum of all previous subsequences + 1`

This approach naturally handles duplicates because:
- If `c` appeared before, we're essentially "updating" the count of subsequences ending with `c`
- The old subsequences ending with `c` are replaced by the new count
- This prevents us from counting the same subsequence multiple times

For other characters that aren't `c`, their counts remain unchanged at position `i` since we're not adding those characters.

The final answer is the sum of subsequences ending with any character, giving us all distinct non-empty subsequences.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with a 2D array to track distinct subsequences by their ending character.

**Data Structure:**
- `dp[i][j]`: A 2D array where `dp[i][j]` represents the count of distinct subsequences ending with character `j` (where `j` is 0-25 for 'a'-'z') using the first `i` characters of string `s`
- Initialize with dimensions `(n+1) × 26` where `n` is the length of the string

**Algorithm Steps:**

1. **Initialization:** Create a 2D DP table with `n+1` rows and 26 columns (one for each lowercase letter). All values start at 0.

2. **Iterate through the string:** For each character at position `i` (1-indexed):
   - Convert the character to its alphabet index: `k = ord(c) - ord('a')`
 
3. **Update DP values:** For each of the 26 possible ending characters `j`:
   - **If `j == k` (current character matches):**
     - Set `dp[i][j] = sum(dp[i-1]) % mod + 1`
     - This means: take all subsequences from the previous position (regardless of their ending character) and append the current character to them, plus create a new single-character subsequence
 
   - **If `j != k` (different character):**
     - Set `dp[i][j] = dp[i-1][j]`
     - This means: carry forward the count unchanged since we're not adding character `j` at this position

4. **Calculate the result:** Sum all values in the last row `dp[-1]` and return modulo `10^9 + 7`

**Why this works:**
- By tracking subsequences by their ending character, we automatically handle duplicates
- When we encounter a character that appeared before, we're essentially updating (not adding to) its count, which prevents double-counting
- The modulo operation is applied at each step to prevent integer overflow

**Time Complexity:** `O(26n) = O(n)` where `n` is the length of the string
**Space Complexity:** `O(26n) = O(n)` for the DP table

## Example Walkthrough

Let's walk through the solution with `s = "aba"`.

**Setup:**
- String: "aba" (length = 3)
- Create `dp[4][26]` table (4 rows for positions 0-3, 26 columns for letters a-z)
- We'll focus on columns for 'a' (index 0) and 'b' (index 1) since those are the only letters present

**Initial State (i=0):**
- All `dp[0][j] = 0` (no characters processed yet)

**Processing character 'a' at position 1:**
- Current character index: k = 0 (for 'a')
- For j = 0 (letter 'a'): Since j == k:
  - `dp[1][0] = sum(dp[0]) + 1 = 0 + 1 = 1`
  - This represents the subsequence "a"
- For j = 1 (letter 'b'): Since j != k:
  - `dp[1][1] = dp[0][1] = 0`
- For all other j: `dp[1][j] = 0`

State after position 1: We have 1 distinct subsequence: "a"

**Processing character 'b' at position 2:**
- Current character index: k = 1 (for 'b')
- For j = 0 (letter 'a'): Since j != k:
  - `dp[2][0] = dp[1][0] = 1`
  - Still have subsequence "a"
- For j = 1 (letter 'b'): Since j == k:
  - `dp[2][1] = sum(dp[1]) + 1 = 1 + 1 = 2`
  - This represents subsequences "b" and "ab"

State after position 2: We have 3 distinct subsequences: "a", "b", "ab"

**Processing character 'a' at position 3:**
- Current character index: k = 0 (for 'a')
- For j = 0 (letter 'a'): Since j == k:
  - `dp[3][0] = sum(dp[2]) + 1 = (1 + 2) + 1 = 4`
  - This represents: "a" (single char), "aa" (first 'a' + last 'a'), "ba" ('b' + last 'a'), "aba" (full string)
- For j = 1 (letter 'b'): Since j != k:
  - `dp[3][1] = dp[2][1] = 2`
  - Still have "b" and "ab"

**Final Result:**
- Sum of `dp[3]`: 4 + 2 = 6
- The 6 distinct subsequences are: "a", "b", "aa", "ab", "ba", "aba"

Notice how when we encountered the second 'a', we didn't add to the existing count for 'a'-ending subsequences but replaced it entirely. This prevents counting duplicates - for instance, we only count "a" once even though it could be formed from either the first or second 'a'.

## Python Solution

```python
class Solution:
    def distinctSubseqII(self, s: str) -> int:
        """
        Count the number of distinct non-empty subsequences of string s.
      
        Args:
            s: Input string consisting of lowercase letters
          
        Returns:
            Number of distinct subsequences modulo 10^9 + 7
        """
        MOD = 10**9 + 7
        n = len(s)
      
        # dp[i][j] represents the count of distinct subsequences ending with character 'a'+j
        # up to position i in the string
        dp = [[0] * 26 for _ in range(n + 1)]
      
        # Process each character in the string
        for i, char in enumerate(s, 1):
            # Convert character to index (0-25 for 'a'-'z')
            char_index = ord(char) - ord('a')
          
            # Update dp values for all 26 possible ending characters
            for j in range(26):
                if j == char_index:
                    # If current character matches, we can:
                    # 1. Append it to all previous subsequences (sum of dp[i-1])
                    # 2. Create a new single-character subsequence (+1)
                    dp[i][j] = (sum(dp[i - 1]) % MOD + 1) % MOD
                else:
                    # If character doesn't match, carry forward the previous count
                    dp[i][j] = dp[i - 1][j]
      
        # Return total count of distinct subsequences (sum of all ending possibilities)
        return sum(dp[-1]) % MOD
```

## Time and Space Complexity

**Time Complexity:** `O(n × 26) = O(n)` where `n` is the length of the string `s`.

The algorithm uses a nested loop structure:
- The outer loop iterates through each character in the string, running `n` times
- The inner loop iterates through all 26 lowercase letters (constant time)
- Inside the inner loop, when `j == k`, we compute `sum(dp[i - 1])` which takes `O(26) = O(1)` time
- All other operations inside the loops are `O(1)`

Therefore, the total time complexity is `O(n × 26 × 1) = O(26n) = O(n)`.

**Space Complexity:** `O(n × 26) = O(n)` where `n` is the length of the string `s`.

The algorithm uses a 2D DP array `dp` with dimensions `(n + 1) × 26`:
- The array has `n + 1` rows (one for each position in the string plus the initial state)
- Each row has 26 columns (one for each lowercase letter)
- No other data structures use significant space

Therefore, the total space complexity is `O((n + 1) × 26) = O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Apply Modulo at Each Step**

**The Pitfall:**
A common mistake is to only apply the modulo operation at the final return statement, rather than during intermediate calculations. The subsequence count grows exponentially with the string length (up to 2000 characters here). In Java or C++ the `long` values overflow and the answer is wrong. Python integers never overflow, so the Python version still returns the right answer, but it slows down because it adds numbers with hundreds of digits.

```python
# RISKY - overflows in Java/C++, slow in Python
dp[i][j] = sum(dp[i - 1]) + 1  # No modulo here
# ...
return sum(dp[-1]) % MOD  # Only modulo at the end
```

**The Solution:**
Apply modulo operation consistently at every arithmetic operation to keep numbers within bounds:

```python
# CORRECT - Keeps every stored value below MOD
dp[i][j] = (sum(dp[i - 1]) % MOD + 1) % MOD
```

### 2. **Memory Optimization Opportunity Missed**

**The Pitfall:**
The current solution uses O(26n) space with a full 2D array, but since we only ever access the previous row (i-1) when computing row i, we're wasting memory by storing all previous rows.

**The Solution:**
Use two 1D arrays (current and previous) or even a single array with careful updating:

```python
def distinctSubseqII(self, s: str) -> int:
    MOD = 10**9 + 7
    # Only need to track counts for each ending character
    dp = [0] * 26
  
    for char in s:
        char_index = ord(char) - ord('a')
        # Calculate new count for this character
        # Add 1 for single character subsequence
        new_count = (sum(dp) % MOD + 1) % MOD
        dp[char_index] = new_count
  
    return sum(dp) % MOD
```

### 3. **Off-by-One Indexing Errors**

**The Pitfall:**
Mixing 0-based and 1-based indexing can cause confusion. The solution uses `enumerate(s, 1)` to start from index 1, which might lead to errors if you're not careful about array boundaries.

```python
# Potential confusion with indexing
for i, char in enumerate(s, 1):  # i starts from 1
    # Must remember dp has n+1 rows, indexed 0 to n
```

**The Solution:**
Be consistent with indexing approach. Either use 0-based throughout or clearly document the indexing strategy:

```python
# Alternative: Use 0-based indexing consistently
dp = [[0] * 26 for _ in range(len(s))]  # n rows here, not n + 1
for i in range(len(s)):
    char_index = ord(s[i]) - ord('a')
    for j in range(26):
        if j == char_index:
            if i == 0:
                dp[i][j] = 1  # First character
            else:
                dp[i][j] = (sum(dp[i - 1]) % MOD + 1) % MOD
        else:
            dp[i][j] = dp[i - 1][j] if i > 0 else 0
```

### 4. **Not Handling Edge Cases**

**The Pitfall:**
Not considering edge cases like empty strings or single-character strings, though the problem states subsequences must be non-empty.

**The Solution:**
Add validation if needed:

```python
if not s:
    return 0
if len(s) == 1:
    return 1
```

