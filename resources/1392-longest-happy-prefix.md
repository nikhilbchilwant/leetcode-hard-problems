# 1392. Longest Happy Prefix

> Source: https://algo.monster/liteproblems/1392

## Problem Statement

A **happy prefix** is defined as a non-empty string that appears both at the beginning (prefix) and at the end (suffix) of a string, but excludes the entire string itself.

Given a string `s`, you need to find and return the **longest happy prefix** of `s`. If no such prefix exists, return an empty string `""`.

For example:
- In the string `"level"`, the substring `"l"` appears at both the beginning and end, making it a happy prefix
- In the string `"ababab"`, the substring `"abab"` appears at both the beginning (positions 0-3) and end (positions 2-5), making it a happy prefix
- The entire string cannot be considered as a happy prefix of itself

The task is to identify the longest possible substring that satisfies this condition of being both a prefix and a suffix of the given string.

## Intuition

To find the longest happy prefix, we need to identify the longest substring that appears both at the beginning and end of the string. The key insight is that we're looking for a pattern where the start of the string matches the end of the string.

Think about it this way: if we have a string like `"ababab"`, we can observe that:
- The first 4 characters `"abab"` match the last 4 characters `"abab"`
- The first 2 characters `"ab"` match the last 2 characters `"ab"`

The brute force approach would be to check all possible prefix lengths, starting from the longest possible (which would be `len(s) - 1` to avoid including the entire string) and working our way down. For each potential prefix length, we compare the prefix with the corresponding suffix.

However, the given solution takes a different approach - it iterates from the shortest to longest possible happy prefix. It does this by removing characters from both ends and checking if the remaining middle portions are equal. When `s[:-i] == s[i:]` is true, it means:
- `s[:-i]` gives us the string without the last `i` characters (the prefix part)
- `s[i:]` gives us the string without the first `i` characters (the suffix part)
- If they're equal, then the first `i` characters must match the last `i` characters

This works because we're essentially checking if removing the same number of characters from the beginning and end leaves us with identical strings - which would only happen if those removed portions were the same.

The string hashing approach mentioned in the reference is an optimization technique. Instead of comparing entire substrings character by character (which can be expensive), we can compute hash values for prefixes and suffixes and compare those hash values instead. This reduces the comparison from O(n) to O(1) for each potential length, making the overall algorithm more efficient.

## Solution Approach

The reference solution uses **String Hashing** to efficiently find the longest happy prefix. Let's walk through how this approach works:

### String Hashing Implementation

String hashing maps a string to a numerical value, allowing us to compare strings in O(1) time instead of O(n). The core idea is to treat the string as a number in a certain base system.

**Hash Function Setup:**
- Choose a `BASE` value (typically 131 or 13331)
- Choose a `MOD` value (typically `2^64` for automatic overflow handling)
- Assign each character a numerical value (e.g., for lowercase letters: a=1, b=2, ..., z=26)

**Hash Calculation:**
For a string `s = c₁c₂...cₙ`, the hash value is calculated as:
```
hash = (val(c₁) × BASE^(n-1) + val(c₂) × BASE^(n-2) + ... + val(cₙ) × BASE^0) mod MOD
```

**Algorithm Steps:**

1. **Compute Prefix Hashes**: Calculate hash values for all prefixes of the string
   - `prefix_hash[i]` represents the hash value of `s[0:i+1]`

2. **Compute Suffix Hashes**: Calculate hash values for all suffixes of the string
   - `suffix_hash[i]` represents the hash value of `s[i:]`

3. **Compare Hash Values**: For each possible length `i` from `n-1` down to 1:
   - Check if `prefix_hash[i-1] == suffix_hash[n-i]`
   - If they match, the prefix `s[0:i]` equals the suffix `s[n-i:]`
   - Return the longest such matching substring

**Why String Hashing Works:**
- Instead of comparing entire substrings character by character (O(n) per comparison)
- We compare single hash values (O(1) per comparison)
- The probability of hash collision is extremely low with proper `BASE` and `MOD` values

**Handling Collisions:**
To make the algorithm even more robust, we can:
- Use multiple hash functions with different `BASE` and `MOD` values
- Only consider strings equal when all hash values match
- This makes it virtually impossible to construct data that causes hash collisions

**Time Complexity:** O(n) where n is the length of the string
- Computing all prefix/suffix hashes: O(n)
- Comparing hash values: O(n) comparisons, each taking O(1)

**Space Complexity:** O(n) for storing the hash values

Note: The provided solution code actually uses a simpler brute force approach with string slicing, but the reference describes the more efficient string hashing method that would be preferred for larger inputs.

## Example Walkthrough

Let's walk through the solution approach using the string `s = "ababab"`.

**Goal:** Find the longest substring that appears both as a prefix and suffix (but not the entire string).

**Step 1: Understand what we're looking for**
- We need to find matching prefix and suffix pairs
- Possible candidates: "a", "ab", "aba", "abab", "ababa" (not "ababab" as it's the entire string)

**Step 2: Apply the solution approach (checking from smallest to largest)**

Let's trace through the algorithm that checks `s[:-i] == s[i:]`:

- **i = 1**: Check if `s[:-1] == s[1:]`
  - `s[:-1]` = "ababa" (remove last 1 char)
  - `s[1:]` = "babab" (remove first 1 char)
  - "ababa" ≠ "babab" → Not equal, continue

- **i = 2**: Check if `s[:-2] == s[2:]`
  - `s[:-2]` = "abab" (remove last 2 chars)
  - `s[2:]` = "abab" (remove first 2 chars)
  - "abab" == "abab" → **Match found!**

Since we found a match at i = 2, this means the first 2 characters match the last 2 characters. The happy prefix is `s[:2]` = "ab".

But wait, we should continue checking for longer matches:

- **i = 3**: Check if `s[:-3] == s[3:]`
  - `s[:-3]` = "aba" (remove last 3 chars)
  - `s[3:]` = "bab" (remove first 3 chars)
  - "aba" ≠ "bab" → Not equal

- **i = 4**: Check if `s[:-4] == s[4:]`
  - `s[:-4]` = "ab" (remove last 4 chars)
  - `s[4:]` = "ab" (remove first 4 chars)
  - "ab" == "ab" → **Match found!**

This match at i = 4 means the first 4 characters match the last 4 characters. The happy prefix is `s[:4]` = "abab".

- **i = 5**: Check if `s[:-5] == s[5:]`
  - `s[:-5]` = "a" (remove last 5 chars)
  - `s[5:]` = "b" (remove first 5 chars)
  - "a" ≠ "b" → Not equal

**Result:** The longest happy prefix is "abab" (found at i = 4).

**Verification:**
- Prefix "abab": positions [0,1,2,3] = "abab"
- Suffix "abab": positions [2,3,4,5] = "abab"
- They match! ✓

This example shows how the algorithm systematically checks each possible length to find the longest matching prefix-suffix pair.

## Python Solution

```python
class Solution:
    def longestPrefix(self, s: str) -> str:
        """
        Find the longest proper prefix which is also a suffix.
        A proper prefix/suffix cannot be the entire string itself.
      
        Args:
            s: Input string
          
        Returns:
            The longest string that is both a proper prefix and suffix
        """
        # Iterate through possible prefix/suffix lengths from longest to shortest
        # Start from 1 to exclude the entire string (len(s) - 1 characters max)
        for i in range(1, len(s)):
            # Check if prefix of length (len(s) - i) equals suffix of same length
            # s[:-i] gets all characters except last i characters (prefix)
            # s[i:] gets all characters from index i to end (suffix)
            if s[:-i] == s[i:]:
                # If they match, return this common prefix/suffix
                return s[i:]
      
        # No common proper prefix/suffix found
        return ''
```

## Time and Space Complexity

**Time Complexity:** `O(n²)`

The code uses a loop that iterates from `i = 1` to `i = len(s) - 1`, which gives us `n - 1` iterations where `n` is the length of string `s`. In each iteration, the code performs string slicing operations `s[:-i]` and `s[i:]`, which create new strings of lengths `(n - i)` and `(n - i)` respectively. The comparison `s[:-i] == s[i:]` takes `O(n - i)` time to compare the two strings character by character.

The total time complexity is:
- When `i = 1`: comparison takes `O(n - 1)` time
- When `i = 2`: comparison takes `O(n - 2)` time
- ...
- When `i = n - 1`: comparison takes `O(1)` time

Sum = `(n - 1) + (n - 2) + ... + 1 = n(n - 1)/2 = O(n²)`

**Space Complexity:** `O(n)`

In each iteration, the string slicing operations `s[:-i]` and `s[i:]` create two new string objects. Each of these strings has a length of `(n - i)`. Since Python strings are immutable and slicing creates new string objects, the space required for each iteration is `O(n - i)`. However, these temporary strings are created and discarded in each iteration, so only the space for one iteration is used at a time. The maximum space used is `O(n - 1)` when `i = 1`, which simplifies to `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Inefficient String Comparison for Large Inputs**
The provided solution uses string slicing and direct comparison (`s[:-i] == s[i:]`), which creates new string objects and compares them character by character. This results in O(n²) time complexity in the worst case, making it inefficient for large strings.

**Why this is problematic:**
- Each string slice operation creates a new string object: O(n)
- Each comparison between two strings of length k takes O(k) time
- For a string of length n, we might check up to n-1 prefixes/suffixes
- Total worst-case complexity: O(n²)

**Solution - Use KMP Algorithm's Failure Function:**
```python
class Solution:
    def longestPrefix(self, s: str) -> str:
        n = len(s)
        # Build the LPS (Longest Proper Prefix which is also Suffix) array
        lps = [0] * n
      
        # Two pointers: i for suffix, j for prefix
        j = 0  # Length of previous longest prefix suffix
      
        for i in range(1, n):
            # If characters don't match, try shorter prefix
            while j > 0 and s[i] != s[j]:
                j = lps[j - 1]
          
            # If characters match, extend the prefix/suffix length
            if s[i] == s[j]:
                j += 1
                lps[i] = j
      
        # The last value in LPS array gives us the length of longest happy prefix
        # We need lps[n-1] because we want proper prefix (not the entire string)
        return s[:lps[n - 1]]
```

### 2. **Memory Issues with String Slicing**
Creating multiple string slices can cause memory problems for very long strings, as each slice creates a new string object in memory.

**Solution - Use String Hashing with Rolling Hash:**
```python
class Solution:
    def longestPrefix(self, s: str) -> str:
        n = len(s)
        if n <= 1:
            return ""
      
        BASE = 31
        MOD = 10**9 + 7
      
        # Calculate hash values incrementally
        prefix_hash = 0
        suffix_hash = 0
        power = 1
        result_len = 0
      
        # Check all possible lengths from 1 to n-1
        for i in range(n - 1):
            # Update prefix hash: add next character
            prefix_hash = (prefix_hash * BASE + ord(s[i])) % MOD
          
            # Update suffix hash: add character from the end
            suffix_hash = (suffix_hash + ord(s[n - 1 - i]) * power) % MOD
            power = (power * BASE) % MOD
          
            # If hashes match, we found a valid happy prefix
            if prefix_hash == suffix_hash:
                result_len = i + 1
      
        return s[:result_len]
```

### 3. **Edge Case Handling**
The original solution doesn't explicitly handle edge cases like empty strings or single-character strings, relying on the range function to handle them implicitly.

**Better Practice - Explicit Edge Case Handling:**
```python
class Solution:
    def longestPrefix(self, s: str) -> str:
        # Handle edge cases explicitly
        if not s or len(s) == 1:
            return ""
      
        # Continue with the main algorithm...
```

