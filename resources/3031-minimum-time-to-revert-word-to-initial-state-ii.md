# 3031. Minimum Time to Revert Word to Initial State II

> Source: https://algo.monster/liteproblems/3031

## Problem Statement

You start with a string `word` and an integer `k`. Every second, you must perform exactly two operations:
1. Remove the first `k` characters from `word`
2. Add any `k` characters to the end of `word`

The goal is to find the minimum number of seconds (greater than 0) needed to transform `word` back to its original state.

For example, if `word = "abcdef"` and `k = 2`:
- After 1 second: Remove "ab", leaving "cdef". You could add "ab" to get "cdefab"
- After 2 seconds: Remove "cd", leaving "efab". You could add "cd" to get "efabcd"  
- After 3 seconds: Remove "ef", leaving "abcd". You could add "ef" to get "abcdef" (original string)

The solution uses rolling hash to efficiently check if a suffix of the original string matches a prefix. At each step `i` (where `i` is a multiple of `k`), it checks if `word[i:]` equals `word[:n-i]`. If they match, we know we can restore the original string in `i/k` seconds by:
- Removing the first `i` characters over `i/k` operations
- Adding back the appropriate characters to restore the original string

## Intuition

The key insight is that after removing the first `k` characters and adding `k` characters to the end, we're essentially performing a circular shift of the string. After `t` seconds, we will have removed the first `t*k` characters from the original string.

For the string to return to its initial state, the remaining portion of the original string (after removing `t*k` characters) must match the beginning of the original string. In other words, if we've removed `i = t*k` characters, then `word[i:]` must equal `word[:n-i]` for us to be able to reconstruct the original string by adding the appropriate characters.

Why does this work? Consider what happens after `t` seconds:
- We've removed characters at positions `[0, t*k)` from the original string
- The current string starts with characters from position `t*k` of the original
- To get back to the original, we need the suffix `word[t*k:]` to match the prefix `word[:n-t*k]`

This is because if they match, we can add the removed characters `word[:t*k]` to the end, and the string becomes `word[t*k:] + word[:t*k]`, which equals the original `word` when the suffix-prefix condition holds.

The rolling hash technique is used to efficiently compare these substrings in O(1) time rather than O(n) for direct string comparison. We check every possible position that's a multiple of `k` (since we can only remove `k` characters at a time), starting from `k` and going up to `n`. The first position where the suffix-prefix condition holds gives us our minimum time.

If no such position exists before reaching the end, it means we need to cycle through the entire string, which takes `ceiling(n/k)` seconds.

## Solution Approach

The solution implements a rolling hash algorithm to efficiently check substring equality. Here's how the implementation works:

**Rolling Hash Class (`Hashing`)**

The `Hashing` class precomputes hash values for all prefixes of the string:
- `h[i]` stores the hash value of the prefix `s[0:i]`
- `p[i]` stores `base^i mod mod` for efficient hash computation
- The hash formula used is: `hash(s) = (s[0]*base^(n-1) + s[1]*base^(n-2) + ... + s[n-1]) mod mod`

The `query(l, r)` method returns the hash of substring `s[l-1:r]` in O(1) time using the formula:
```
hash(s[l-1:r]) = (h[r] - h[l-1] * p[r-l+1]) mod mod
```

**Main Algorithm (`minimumTimeToInitialState`)**

1. Initialize the hashing object with base 13331 and modulo 998244353 (large prime to minimize collisions)

2. Iterate through all positions that are multiples of `k`:
   ```python
   for i in range(k, n, k):
   ```
   At each position `i`, we check if removing the first `i` characters would allow us to restore the original string.

3. For each position `i`, compare two substrings using hash values:
   - `hashing.query(1, n - i)`: Hash of `word[0:n-i]` (the prefix we need to match)
   - `hashing.query(i + 1, n)`: Hash of `word[i:n]` (the suffix that remains after removal)
 
   Note: The query uses 1-based indexing, so we add 1 to convert from 0-based.

4. If the hashes match, it means `word[0:n-i] == word[i:n]`. This indicates we can restore the original string in `i // k` seconds by:
   - Removing `k` characters `i/k` times
   - Adding back the appropriate characters to complete the original string

5. If no match is found in the loop, return `(n + k - 1) // k`, which is the ceiling division representing the time needed to completely cycle through the entire string.

The time complexity is O(n) for preprocessing the hash values and O(n/k) for checking positions, resulting in O(n) overall. The space complexity is O(n) for storing the hash arrays.

## Example Walkthrough

Let's trace through the algorithm with `word = "abacaba"` and `k = 3`.

**Step 1: Initialize Rolling Hash**
- Create hash arrays for the string "abacaba" (length n = 7)
- The hash will help us compare substrings in O(1) time

**Step 2: Check positions that are multiples of k**

*Position i = 3 (after 1 second):*
- We check if removing the first 3 characters allows restoration
- Need to compare: `word[0:4]` ("abac") with `word[3:7]` ("caba")
- Using hash: `query(1, 4)` vs `query(4, 7)`
- "abac" ≠ "caba", so we can't restore in 1 second

*Position i = 6 (after 2 seconds):*
- We check if removing the first 6 characters allows restoration  
- Need to compare: `word[0:1]` ("a") with `word[6:7]` ("a")
- Using hash: `query(1, 1)` vs `query(7, 7)`
- "a" = "a", so we CAN restore in 2 seconds!

**Why does this work?**
After 2 seconds of operations:
- Second 1: Remove "aba" → "caba", add any 3 chars (say "xyz") → "cabaxyz"
- Second 2: Remove "cab" → "axyz", add "aba" → "axyzaba"
- But wait, since word[0:1] = word[6:7] = "a", we can be smarter:
- Second 1: Remove "aba" → "caba", add "aba" → "cabaaba"  
- Second 2: Remove "cab" → "aaba", add "cab" → "aabacab"
- Actually, let me reconsider...
- Second 1: Remove "aba" → "caba", we need to add 3 chars that will help us get back
- Second 2: Remove "cab" → "a" + (our 3 added chars)
- Since word[6:7] = "a" matches word[0:1] = "a", we can add "bacaba" over the 2 operations to restore "abacaba"

The answer is **2 seconds**.

If no match was found at any position, we would return `⌈7/3⌉ = 3` seconds.

## Python Solution

```python
class Hashing:
    """Rolling hash implementation for efficient substring comparison."""
  
    __slots__ = ["mod", "h", "p"]

    def __init__(self, s: str, base: int, mod: int) -> None:
        """
        Initialize the rolling hash for string s.
      
        Args:
            s: Input string to be hashed
            base: Base value for polynomial rolling hash
            mod: Modulo value to prevent overflow
        """
        self.mod = mod
        # h[i] stores the hash value of prefix s[0:i]
        self.h = [0] * (len(s) + 1)
        # p[i] stores base^i mod mod
        self.p = [1] * (len(s) + 1)
      
        # Build prefix hash and power arrays
        for i in range(1, len(s) + 1):
            # Calculate hash of prefix ending at position i
            self.h[i] = (self.h[i - 1] * base + ord(s[i - 1])) % mod
            # Calculate base^i for later use
            self.p[i] = (self.p[i - 1] * base) % mod

    def query(self, l: int, r: int) -> int:
        """
        Get the hash value of substring s[l-1:r].
      
        Args:
            l: Starting position (1-indexed)
            r: Ending position (1-indexed, inclusive)
          
        Returns:
            Hash value of the substring
        """
        # Extract hash of substring using prefix hash difference
        return (self.h[r] - self.h[l - 1] * self.p[r - l + 1]) % self.mod


class Solution:
    def minimumTimeToInitialState(self, word: str, k: int) -> int:
        """
        Find minimum time to return string to initial state by removing k characters
        from the beginning and appending them to the end.
      
        Args:
            word: Input string
            k: Number of characters to remove/append in each operation
          
        Returns:
            Minimum number of operations needed
        """
        # Initialize rolling hash with chosen base and prime modulo
        hashing = Hashing(word, base=13331, mod=998244353)
        n = len(word)
      
        # Try each possible number of k-sized chunks removed
        for i in range(k, n, k):
            # Check if remaining suffix matches the prefix of same length
            # Compare word[0:n-i] with word[i:n]
            if hashing.query(1, n - i) == hashing.query(i + 1, n):
                return i // k
      
        # If no match found, need to process entire string
        # Return ceiling division of n by k
        return (n + k - 1) // k
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The analysis breaks down as follows:
- The `Hashing.__init__` method performs a single loop from 1 to `len(s) + 1`, which takes `O(n)` time where `n` is the length of the string.
- The `Hashing.query` method performs constant time operations (arithmetic and modulo operations), so it takes `O(1)` time.
- In the `minimumTimeToInitialState` method:
  - Creating the `Hashing` object takes `O(n)` time.
  - The for loop iterates from `k` to `n` with step `k`, resulting in at most `n/k` iterations.
  - Each iteration calls `hashing.query` twice, each taking `O(1)` time.
  - Therefore, the loop takes `O(n/k)` time.
- Since `k ≥ 1`, the loop complexity `O(n/k)` is bounded by `O(n)`.
- The overall time complexity is `O(n) + O(n/k) = O(n)`.

**Space Complexity:** `O(n)`

The space analysis:
- The `Hashing` class stores two arrays `h` and `p`, each of size `len(s) + 1 = n + 1`.
- These arrays take `O(n)` space.
- The `Solution` class uses only a constant amount of additional space for variables like `i` and `n`.
- Therefore, the total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Hash Collision Leading to False Positives**

The most critical pitfall in this rolling hash solution is assuming that equal hash values always mean equal strings. Hash collisions can occur even with a good hash function, leading to incorrect results.

**Problem Example:**
```python
# Even with base=13331 and mod=998244353, collisions are possible
# The algorithm might incorrectly identify a match when hashes are equal
# but the actual substrings are different
```

**Solution:**
Add a verification step when hash values match to ensure the substrings are actually equal:

```python
class Solution:
    def minimumTimeToInitialState(self, word: str, k: int) -> int:
        hashing = Hashing(word, base=13331, mod=998244353)
        n = len(word)
      
        for i in range(k, n, k):
            # First check hash equality
            if hashing.query(1, n - i) == hashing.query(i + 1, n):
                # Verify actual string equality to avoid hash collision
                if word[:n-i] == word[i:]:
                    return i // k
      
        return (n + k - 1) // k
```

### 2. **Integer Overflow in Hash Calculation**

Without proper modulo operations, the hash values can overflow, especially for long strings.

**Problem Example:**
```python
# Incorrect: Missing modulo in intermediate calculations
self.h[i] = self.h[i - 1] * base + ord(s[i - 1])  # Can overflow!
```

**Solution:**
Always apply modulo operations at each step:

```python
# Correct: Apply modulo at each operation
self.h[i] = (self.h[i - 1] * base + ord(s[i - 1])) % mod
self.p[i] = (self.p[i - 1] * base) % mod
```

### 3. **Off-by-One Errors in Index Conversion**

The `Hashing` class uses 1-based indexing while Python uses 0-based indexing, making it easy to make indexing errors.

**Problem Example:**
```python
# Incorrect indexing - forgetting to adjust for 1-based indexing
if hashing.query(0, n - i - 1) == hashing.query(i, n - 1):  # Wrong!
```

**Solution:**
Document the indexing convention clearly and be consistent:

```python
# Correct: Convert 0-based to 1-based indexing
# To check word[0:n-i] == word[i:n]
if hashing.query(1, n - i) == hashing.query(i + 1, n):
    return i // k
```

### 4. **Using Double Hashing for Collision Resistance**

For production code where correctness is critical, use double hashing with different bases and moduli:

```python
class Solution:
    def minimumTimeToInitialState(self, word: str, k: int) -> int:
        # Use two different hash functions
        hash1 = Hashing(word, base=13331, mod=998244353)
        hash2 = Hashing(word, base=31, mod=10**9 + 7)
        n = len(word)
      
        for i in range(k, n, k):
            # Check both hashes match
            if (hash1.query(1, n - i) == hash1.query(i + 1, n) and
                hash2.query(1, n - i) == hash2.query(i + 1, n)):
                return i // k
      
        return (n + k - 1) // k
```

This dramatically reduces the probability of collision from approximately 1/mod to 1/(mod1 * mod2).

