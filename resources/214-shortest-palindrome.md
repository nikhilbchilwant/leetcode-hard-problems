# 214. Shortest Palindrome

> Source: https://algo.monster/liteproblems/214

## Problem Statement

You are given a string `s`. Your task is to find the shortest possible palindrome that can be created by adding characters only to the beginning (front) of the string.

A palindrome is a string that reads the same forwards and backwards. For example, "racecar" and "noon" are palindromes.

The goal is to determine what characters need to be prepended to `s` to make the entire resulting string a palindrome, and this resulting palindrome should be as short as possible.

For example:
- If `s = "aacecaaa"`, the shortest palindrome would be `"aaacecaaa"` (adding just one 'a' to the front)
- If `s = "abcd"`, the shortest palindrome would be `"dcbabcd"` (adding "dcb" to the front)

The solution uses a rolling hash technique to efficiently find the longest prefix of the string that is also a palindrome. This is done by:
1. Computing a forward hash (`prefix`) and a reverse hash (`suffix`) simultaneously as we traverse the string
2. When these hashes match at position `i`, the substring from index 0 to `i` is treated as a palindrome (equal hashes mean equal strings except in the rare case of a hash collision)
3. Tracking the largest such index (`idx`) where this occurs
4. The characters after this longest palindromic prefix need to be reversed and prepended to create the shortest palindrome

The final answer is constructed by taking the non-palindromic suffix `s[idx:]`, reversing it, and prepending it to the original string `s`.

## Intuition

To make a string into a palindrome by only adding characters to the front, we need to think about what part of the original string can already serve as the "middle" of our palindrome.

The key insight is that we want to find the longest prefix of the string that is already a palindrome. Why? Because this prefix won't need any corresponding characters added to the front - it's already symmetric.

For example, if `s = "aacecaaa"`, the prefix "aacecaa" is already a palindrome. So we only need to handle the remaining part "a" by adding its reverse to the front.

The challenge becomes: how do we efficiently check if a prefix is a palindrome? Checking every possible prefix naively would be expensive. This is where rolling hash comes in.

The rolling hash technique allows us to:
- Build a hash of the string reading forward (`prefix`)
- Build a hash of the string reading backward (`suffix`)
- When these two hashes are equal at position `i`, the substring from start to position `i` is almost certainly a palindrome (two different strings can collide on the same hash, but with a large prime modulus this is rare)

Think of it like comparing fingerprints - if the forward fingerprint matches the backward fingerprint, we have a palindrome. We use modular arithmetic with a base (131) to create unique "fingerprints" for each substring.

As we traverse the string, we keep updating both hashes and check if they match. The last position where they match gives us the longest palindromic prefix. Everything after this prefix needs to be "mirrored" to the front to complete the palindrome.

The final step is simple: take whatever remains after the longest palindromic prefix, reverse it, and prepend it to the original string. This gives us the shortest possible palindrome.

## Solution Approach

The solution implements a rolling hash algorithm to find the longest palindromic prefix efficiently in O(n) time.

**Step-by-step Implementation:**

1. **Initialize Rolling Hash Variables:**
   - `base = 131`: The base for our polynomial hash function
   - `mod = 10^9 + 7`: A large prime number to prevent overflow
   - `prefix = 0`: Hash value built from left to right
   - `suffix = 0`: Hash value built from right to left
   - `mul = 1`: Multiplier for the suffix hash
   - `idx = 0`: Tracks the end index of the longest palindromic prefix

2. **Build Hashes While Traversing:**
   For each character at position `i`:
   - **Update prefix hash:** `prefix = (prefix * base + (ord(c) - ord('a') + 1)) % mod`
     - This shifts existing hash left by multiplying with base
     - Adds the new character's value (1-26 for 'a'-'z')
 
   - **Update suffix hash:** `suffix = (suffix + (ord(c) - ord('a') + 1) * mul) % mod`
     - Adds new character with appropriate positional weight
     - The weight `mul` increases by factor of `base` each iteration
 
   - **Update multiplier:** `mul = (mul * base) % mod`

3. **Check for Palindrome:**
   - If `prefix == suffix`, the substring from index 0 to `i` is a palindrome
   - Update `idx = i + 1` to mark this position

4. **Construct the Result:**
   - If `idx == n`: The entire string is already a palindrome, return `s`
   - Otherwise: Take `s[idx:]` (the non-palindromic suffix), reverse it with `[::-1]`, and prepend it to `s`

**Example Walkthrough:**

For `s = "aacecaaa"`:
- At index 0 ('a'): prefix and suffix both equal hash('a'), idx = 1
- At index 1 ('a'): prefix = hash('aa'), suffix = hash('aa'), idx = 2
- Continue until index 6: prefix = hash('aacecaa'), suffix = hash('aacecaa'), idx = 7
- Final: `s[7:]` = "a", reversed = "a", result = "a" + "aacecaaa" = "aaacecaaa"

The algorithm cleverly uses the property that if two strings have the same hash value, they are likely the same string (with very high probability due to the large prime modulus).

## Example Walkthrough

Let's walk through the solution with `s = "abcd"`:

**Initial Setup:**
- `base = 131`, `mod = 10^9 + 7`
- `prefix = 0`, `suffix = 0`, `mul = 1`, `idx = 0`

**Iteration 1 (i=0, c='a'):**
- Calculate character value: `ord('a') - ord('a') + 1 = 1`
- Update prefix: `prefix = (0 * 131 + 1) % mod = 1`
- Update suffix: `suffix = (0 + 1 * 1) % mod = 1`
- Check: `prefix == suffix` ✓ (both equal 1)
- Update: `idx = 1` (substring "a" is palindrome)
- Update: `mul = (1 * 131) % mod = 131`

**Iteration 2 (i=1, c='b'):**
- Character value: `ord('b') - ord('a') + 1 = 2`
- Update prefix: `prefix = (1 * 131 + 2) % mod = 133`
- Update suffix: `suffix = (1 + 2 * 131) % mod = 263`
- Check: `prefix != suffix` ✗ (133 ≠ 263)
- No update to `idx` (substring "ab" is not palindrome)
- Update: `mul = (131 * 131) % mod = 17161`

**Iteration 3 (i=2, c='c'):**
- Character value: `ord('c') - ord('a') + 1 = 3`
- Update prefix: `prefix = (133 * 131 + 3) % mod = 17426`
- Update suffix: `suffix = (263 + 3 * 17161) % mod = 51746`
- Check: `prefix != suffix` ✗
- No update to `idx`
- Update: `mul = (17161 * 131) % mod`

**Iteration 4 (i=3, c='d'):**
- Character value: `ord('d') - ord('a') + 1 = 4`
- Update prefix and suffix with same process
- Check: `prefix != suffix` ✗
- No update to `idx`

**Final Construction:**
- Longest palindromic prefix ends at `idx = 1` (just "a")
- Non-palindromic suffix: `s[1:]` = "bcd"
- Reverse the suffix: "bcd" → "dcb"
- Final result: "dcb" + "abcd" = **"dcbabcd"**

We can verify "dcbabcd" is indeed a palindrome: reading forward gives d-c-b-a-b-c-d, reading backward gives d-c-b-a-b-c-d ✓

## Python Solution

```python
class Solution:
    def shortestPalindrome(self, s: str) -> str:
        """
        Find the shortest palindrome by adding characters in front of string s.
        Uses rolling hash to find the longest palindrome prefix efficiently.
      
        Args:
            s: Input string to convert to palindrome
      
        Returns:
            Shortest palindrome formed by adding characters to the beginning
        """
        # Rolling hash parameters
        BASE = 131  # Prime base for polynomial rolling hash
        MOD = 10**9 + 7  # Large prime modulus to prevent overflow
      
        n = len(s)
      
        # Hash values for prefix and suffix
        prefix_hash = 0  # Hash of s[0:i+1]
        suffix_hash = 0  # Hash of reverse of s[0:i+1]
      
        # Multiplier for suffix hash calculation
        base_power = 1
      
        # Track the longest palindrome prefix ending position
        longest_palindrome_end = 0
      
        # Iterate through each character to find longest palindrome prefix
        for i, char in enumerate(s):
            # Calculate character value (1-indexed)
            char_value = ord(char) - ord('a') + 1
          
            # Update prefix hash: hash = hash * base + char_value
            prefix_hash = (prefix_hash * BASE + char_value) % MOD
          
            # Update suffix hash: builds hash of reversed string
            # For reversed string, we add new character at the beginning
            suffix_hash = (suffix_hash + char_value * base_power) % MOD
          
            # Update base power for next iteration
            base_power = (base_power * BASE) % MOD
          
            # If hashes match, substring s[0:i+1] is a palindrome
            if prefix_hash == suffix_hash:
                longest_palindrome_end = i + 1
      
        # If entire string is palindrome, return as is
        if longest_palindrome_end == n:
            return s
      
        # Otherwise, prepend the reverse of non-palindrome suffix
        # s[longest_palindrome_end:] is the part that's not palindrome
        non_palindrome_suffix = s[longest_palindrome_end:]
        return non_palindrome_suffix[::-1] + s
```

## Time and Space Complexity

**Time Complexity:** `O(n)` where `n` is the length of the input string `s`.

The algorithm iterates through the string exactly once with a single for loop. Within each iteration, all operations are constant time:
- Computing `prefix` involves multiplication and modulo operations: `O(1)`
- Computing `suffix` involves addition, multiplication and modulo operations: `O(1)`
- Updating `mul` involves multiplication and modulo: `O(1)`
- Comparison `prefix == suffix`: `O(1)`

The string slicing and reversal operation `s[idx:][::-1]` at the end takes `O(n-idx)` in the worst case, which is still `O(n)`. The string concatenation also takes `O(n)`.

Therefore, the overall time complexity is `O(n)`.

**Space Complexity:** `O(n)` 

The algorithm uses:
- Fixed number of integer variables (`base`, `mod`, `n`, `prefix`, `suffix`, `mul`, `idx`): `O(1)`
- The output string construction `s[idx:][::-1] + s` creates a new string of length at most `2n - 1`: `O(n)`

The dominant space usage comes from the string construction for the return value, giving us a total space complexity of `O(n)`.

## Common Pitfalls

### 1. **Hash Collision Leading to Incorrect Results**

The rolling hash algorithm relies on the assumption that if two hash values are equal, the strings are equal. However, hash collisions can occur where different strings produce the same hash value, leading to incorrectly identifying a non-palindrome as a palindrome.

**Example of the problem:**
```python
# Vulnerable implementation - single hash
def shortestPalindrome_vulnerable(s: str):
    BASE = 131
    MOD = 10**9 + 7
    prefix_hash = suffix_hash = 0
    base_power = 1
    longest_palindrome_end = 0
  
    for i, char in enumerate(s):
        char_value = ord(char) - ord('a') + 1
        prefix_hash = (prefix_hash * BASE + char_value) % MOD
        suffix_hash = (suffix_hash + char_value * base_power) % MOD
        base_power = (base_power * BASE) % MOD
      
        # Collision might cause false positive here
        if prefix_hash == suffix_hash:
            longest_palindrome_end = i + 1
  
    return s[longest_palindrome_end:][::-1] + s
```

**Solution - Use Double Hashing or Explicit Verification:**
```python
def shortestPalindrome_safe(s: str):
    # Use two different hash functions
    BASE1, MOD1 = 131, 10**9 + 7
    BASE2, MOD2 = 137, 10**9 + 9
  
    n = len(s)
    prefix_hash1 = suffix_hash1 = 0
    prefix_hash2 = suffix_hash2 = 0
    base_power1 = base_power2 = 1
    longest_palindrome_end = 0
  
    for i, char in enumerate(s):
        char_value = ord(char) - ord('a') + 1
      
        # First hash function
        prefix_hash1 = (prefix_hash1 * BASE1 + char_value) % MOD1
        suffix_hash1 = (suffix_hash1 + char_value * base_power1) % MOD1
        base_power1 = (base_power1 * BASE1) % MOD1
      
        # Second hash function
        prefix_hash2 = (prefix_hash2 * BASE2 + char_value) % MOD2
        suffix_hash2 = (suffix_hash2 + char_value * base_power2) % MOD2
        base_power2 = (base_power2 * BASE2) % MOD2
      
        # Both hashes must match for palindrome confirmation
        if prefix_hash1 == suffix_hash1 and prefix_hash2 == suffix_hash2:
            longest_palindrome_end = i + 1
  
    if longest_palindrome_end == n:
        return s
    return s[longest_palindrome_end:][::-1] + s
```

### 2. **Integer Overflow in Languages Without Automatic Big Integer Handling**

In C++ or Java, `prefix_hash` stays below `MOD = 10^9 + 7` after each `% MOD`, but the next step computes `prefix_hash * 131` before taking the modulo. That product can reach about `1.3 * 10^11`, which overflows a 32-bit `int` (maximum about `2.1 * 10^9`). The same applies to `char_value * base_power` and `base_power * BASE`.

Splitting the expression into more modulo steps does not help, because `(prefix_hash * BASE) % MOD` still forms the full product first. The fix is to do the arithmetic in a 64-bit type:

```java
long prefixHash = 0, suffixHash = 0, basePower = 1;  // 64-bit holds products up to about 9.2 * 10^18
for (char c : s.toCharArray()) {
    int charValue = c - 'a' + 1;
    prefixHash = (prefixHash * BASE + charValue) % MOD;
    suffixHash = (suffixHash + charValue * basePower) % MOD;
    basePower = (basePower * BASE) % MOD;
}
```

Python integers have no fixed size, so the Python solution cannot overflow; it applies `% MOD` only to keep the numbers small and fast.

### 3. **Incorrect Character Value Calculation for Non-Lowercase Strings**

The current implementation assumes all characters are lowercase letters ('a' to 'z'). If the input contains uppercase letters, digits, or special characters, the hash calculation will fail.

**Problem example:**
```python
# This will fail for s = "Aa" or s = "a1b"
char_value = ord(char) - ord('a') + 1  # Negative or incorrect values for non-lowercase
```

**Solution - Handle All ASCII Characters:**
```python
def shortestPalindrome_all_chars(s: str):
    BASE = 131
    MOD = 10**9 + 7
  
    n = len(s)
    prefix_hash = suffix_hash = 0
    base_power = 1
    longest_palindrome_end = 0
  
    for i, char in enumerate(s):
        # Use the actual ASCII value directly
        char_value = ord(char)  # Works for all characters
      
        prefix_hash = (prefix_hash * BASE + char_value) % MOD
        suffix_hash = (suffix_hash + char_value * base_power) % MOD
        base_power = (base_power * BASE) % MOD
      
        if prefix_hash == suffix_hash:
            longest_palindrome_end = i + 1
  
    if longest_palindrome_end == n:
        return s
    return s[longest_palindrome_end:][::-1] + s
```

