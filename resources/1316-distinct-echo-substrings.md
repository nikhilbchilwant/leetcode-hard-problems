# 1316. Distinct Echo Substrings

> Source: https://algo.monster/liteproblems/1316

## Problem Statement

The problem asks you to find the number of **distinct** substrings in a given string `text` that can be formed by concatenating a string with itself.

In other words, you need to count how many unique substrings have the pattern `a + a`, where `a` is some non-empty string. This means the substring can be split into two identical halves.

For example:
- In the string `"abcabc"`, the substring `"abcabc"` itself is an echo substring because it equals `"abc" + "abc"`
- In the string `"aaaa"`, substrings like `"aa"` (which is `"a" + "a"`), `"aaaa"` (which is `"aa" + "aa"`), are echo substrings

The key requirements are:
1. The substrings must be non-empty
2. Each substring must be able to be split into two identical consecutive parts
3. Count only distinct echo substrings (if the same echo substring appears multiple times at different positions, count it only once)

## Intuition

To find substrings that can be written as `a + a`, we need to check if any substring can be split into two identical halves. The straightforward approach would be to:

1. Generate all possible substrings
2. Check if each substring has even length (odd-length strings can't be split into two equal parts)
3. Split each even-length substring in half and compare if both halves are identical

However, directly comparing strings character by character for every possible substring would be inefficient, especially for longer strings. This is where the key insight comes in: we can use **rolling hash** to speed up string comparisons.

Rolling hash allows us to compute a hash value for any substring in `O(1)` time after preprocessing. The idea is:
- Precompute hash values for all prefixes of the string
- For any substring from position `i` to `j`, we can calculate its hash value using the prefix hashes
- If two substrings have the same hash value, they are likely identical (with very high probability when using a good hash function)

The algorithm flow becomes:
1. For each possible starting position `i` in the string
2. For each possible ending position `j` that makes the substring length even (by incrementing `j` by 2 each time)
3. Find the middle point `k` of the substring from `i` to `j`
4. Use hash values to quickly check if substring `[i, k]` equals substring `[k+1, j]`
5. If they match, we found an echo substring - add its hash to a set to track distinct ones

Using a set to store the hash values of valid echo substrings automatically handles the "distinct" requirement, as duplicate echo substrings will have the same hash and won't be counted twice.

**Learn more about [Trie](/problems/trie_intro) patterns.**

## Solution Approach

The solution implements a rolling hash algorithm to efficiently find all distinct echo substrings. Here's the step-by-step implementation:

**1. Rolling Hash Setup**
```python
base = 131
mod = int(1e9) + 7
h = [0] * (n + 10)  # Prefix hash array
p = [1] * (n + 10)  # Powers of base array
```
- We use `base = 131` as the base for our polynomial hash function
- `mod = 10^9 + 7` to prevent integer overflow
- `h[i]` stores the hash value of the prefix `text[0...i-1]`
- `p[i]` stores `base^i % mod` for quick computation

**2. Preprocessing - Computing Prefix Hashes**
```python
for i, c in enumerate(text):
    t = ord(c) - ord('a') + 1
    h[i + 1] = (h[i] * base) % mod + t
    p[i + 1] = (p[i] * base) % mod
```
- Convert each character to a number (1-26 for 'a'-'z')
- Build the prefix hash array using the formula: `h[i+1] = h[i] * base + text[i]`
- Precompute powers of base for later use

**3. Hash Extraction Function**
```python
def get(l, r):
    return (h[r] - h[l - 1] * p[r - l + 1]) % mod
```
- This function extracts the hash value of substring `text[l-1...r-1]` in O(1) time
- Formula: `hash(s[l...r]) = (h[r] - h[l-1] * base^(r-l+1)) % mod`
- This works by removing the contribution of the prefix before position `l`

**4. Finding Echo Substrings**
```python
vis = set()
for i in range(n - 1):
    for j in range(i + 1, n, 2):
        k = (i + j) >> 1
        a = get(i + 1, k + 1)
        b = get(k + 2, j + 1)
        if a == b:
            vis.add(a)
```
- `i` is the starting index (0-indexed in the loop, but 1-indexed for the `get` function)
- `j` is the ending index, incremented by 2 to ensure even-length substrings
- `k = (i + j) >> 1` finds the middle point (using bit shift for division by 2)
- Compare hash values of the first half `[i+1, k+1]` and second half `[k+2, j+1]`
- If hashes match, the substring is an echo substring - add its hash to the set
- Using a set automatically handles duplicates

**5. Return Result**
```python
return len(vis)
```
- The size of the set gives us the count of distinct echo substrings

The time complexity is `O(n²)` for checking all possible even-length substrings, with `O(1)` hash comparisons. The space complexity is `O(n)` for storing the hash arrays and the set of distinct echo substrings.

## Example Walkthrough

Let's walk through the solution with the string `text = "abcabc"`.

**Step 1: Initialize Rolling Hash**
- `n = 6` (length of "abcabc")
- Initialize arrays: `h = [0, 0, 0, 0, 0, 0, 0]` and `p = [1, 0, 0, 0, 0, 0, 0]`
- `base = 131`, `mod = 10^9 + 7`

**Step 2: Build Prefix Hash Array**
- For 'a' (index 0): `t = 1`, `h[1] = 0 * 131 + 1 = 1`, `p[1] = 131`
- For 'b' (index 1): `t = 2`, `h[2] = 1 * 131 + 2 = 133`, `p[2] = 131^2`
- For 'c' (index 2): `t = 3`, `h[3] = 133 * 131 + 3 = 17426`, `p[3] = 131^3`
- For 'a' (index 3): `t = 1`, `h[4] = 17426 * 131 + 1 = 2282807`, `p[4] = 131^4`
- For 'b' (index 4): `t = 2`, `h[5] = 2282807 * 131 + 2 = 299047719`, `p[5] = 131^5`
- For 'c' (index 5): `t = 3`, `h[6] = 299047719 * 131 + 3 = (computed with mod)`

**Step 3: Find Echo Substrings**

Let's trace through some iterations:

- **i = 0** (starting at 'a'):
  - **j = 1**: Substring "ab" (length 2)
    - Middle point: `k = (0 + 1) >> 1 = 0`
    - First half: `get(1, 1)` = hash of "a" = 1
    - Second half: `get(2, 2)` = hash of "b" = 2
    - 1 ≠ 2, not an echo substring

  - **j = 3**: Substring "abca" (length 4)
    - Middle point: `k = (0 + 3) >> 1 = 1`
    - First half: `get(1, 2)` = hash of "ab" = 133
    - Second half: `get(3, 4)` = hash of "ca" = different hash
    - Not an echo substring

  - **j = 5**: Substring "abcabc" (length 6)
    - Middle point: `k = (0 + 5) >> 1 = 2`
    - First half: `get(1, 3)` = hash of "abc" = 17426
    - Second half: `get(4, 6)` = hash of "abc" = 17426
    - **17426 = 17426, this IS an echo substring!**
    - Add 17426 to `vis` set

- **i = 2** (starting at 'c'):
  - **j = 3**: Substring "ca" (length 2)
    - First half: hash of "c", Second half: hash of "a"
    - Different hashes, not an echo substring

  - **j = 5**: Substring "cabc" (length 4)
    - First half: hash of "ca", Second half: hash of "bc"
    - Different hashes, not an echo substring

- **i = 3** (starting at 'a'):
  - **j = 4**: Substring "ab" (length 2)
    - This is the same "ab" we saw earlier
    - Not an echo substring (different halves)

**Step 4: Return Result**
- `vis = {17426}` contains only one distinct echo substring ("abcabc" = "abc" + "abc")
- Return `len(vis) = 1`

The algorithm correctly identifies that "abcabc" has exactly one distinct echo substring, which is the entire string itself formed by concatenating "abc" with itself.

## Python Solution

```python
class Solution:
    def distinctEchoSubstrings(self, text: str) -> int:
        """
        Find the number of distinct non-empty substrings that can be written 
        as the concatenation of some string with itself (echo substrings).
      
        Uses rolling hash technique for efficient substring comparison.
        """
        def get_hash(left: int, right: int) -> int:
            """
            Calculate the hash value of substring text[left-1:right].
            Uses 1-indexed positions for convenience with the hash array.
            """
            return (hash_values[right] - hash_values[left - 1] * powers[right - left + 1]) % MOD
      
        # Initialize constants and variables
        n = len(text)
        BASE = 131  # Prime base for rolling hash
        MOD = 10**9 + 7  # Large prime modulus to avoid collisions
      
        # Precompute hash values and powers of base
        # hash_values[i] stores hash of text[0:i]
        # powers[i] stores BASE^i mod MOD
        hash_values = [0] * (n + 10)
        powers = [1] * (n + 10)
      
        # Build hash array and power array
        for i, char in enumerate(text):
            # Convert character to number (a=1, b=2, ...)
            char_value = ord(char) - ord('a') + 1
            # Calculate cumulative hash
            hash_values[i + 1] = (hash_values[i] * BASE + char_value) % MOD
            # Calculate powers of base
            powers[i + 1] = (powers[i] * BASE) % MOD
      
        # Set to store unique echo substring hashes
        unique_echo_hashes = set()
      
        # Check all possible echo substrings
        # i is the starting position (0-indexed)
        for i in range(n - 1):
            # j is the ending position (0-indexed), step by 2 for even length
            for j in range(i + 1, n, 2):
                # Calculate midpoint of substring
                midpoint = (i + j) >> 1
              
                # Get hash of first half [i, midpoint]
                first_half_hash = get_hash(i + 1, midpoint + 1)
              
                # Get hash of second half [midpoint+1, j]
                second_half_hash = get_hash(midpoint + 2, j + 1)
              
                # If both halves are identical, it's an echo substring
                if first_half_hash == second_half_hash:
                    unique_echo_hashes.add(first_half_hash)
      
        return len(unique_echo_hashes)
```

## Time and Space Complexity

**Time Complexity:** `O(n²)`

The algorithm consists of two main parts:

1. **Preprocessing phase:** Computing the hash array `h` and power array `p` takes `O(n)` time, where `n` is the length of the input string.

2. **Main computation phase:** The nested loops iterate through all possible starting positions `i` (from `0` to `n-1`) and examine substrings of even length starting at position `i`. For each starting position `i`, the inner loop runs approximately `n/2` times in the worst case (when `j` goes from `i+1` to `n` with step size 2). This gives us `O(n) * O(n/2) = O(n²)` iterations. Inside the loops, the `get` function performs constant time operations using the precomputed hash values.

Therefore, the overall time complexity is `O(n) + O(n²) = O(n²)`.

**Space Complexity:** `O(n)`

The space usage breaks down as follows:

1. Hash array `h`: `O(n + 10) = O(n)` space
2. Power array `p`: `O(n + 10) = O(n)` space  
3. Set `vis` to store unique hash values: In the worst case, this could store `O(n²)` different substrings, but since we're only storing hash values of echo substrings (substrings that can be split into two identical halves), the maximum number of unique echo substrings is bounded by `O(n²)`. However, in practice, the set typically stores much fewer elements.

The dominant space usage comes from the auxiliary arrays and the set, giving us an overall space complexity of `O(n)` for the auxiliary arrays plus potentially `O(n²)` for the set in the worst case, resulting in `O(n²)` worst-case space complexity. However, if we consider only the additional space used beyond the input, the practical space complexity is often closer to `O(n)` due to the limited number of actual echo substrings.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Hash Collision Leading to False Positives**
The most critical pitfall in this rolling hash solution is that hash collisions can occur - two different strings might produce the same hash value, leading to incorrect counting of echo substrings.

**Problem Example:**
```python
# Even with a good hash function, collisions can happen
# String "ab" and "xy" might hash to the same value
# This would incorrectly identify "abxy" as an echo substring
```

**Solution:**
Use double hashing or verify matches with actual string comparison:
```python
def distinctEchoSubstrings(self, text: str) -> int:
    # ... (hash setup code remains the same)
  
    unique_echo_substrings = set()  # Store actual substrings, not hashes
  
    for i in range(n - 1):
        for j in range(i + 1, n, 2):
            midpoint = (i + j) >> 1
          
            first_half_hash = get_hash(i + 1, midpoint + 1)
            second_half_hash = get_hash(midpoint + 2, j + 1)
          
            if first_half_hash == second_half_hash:
                # Verify with actual string comparison to avoid false positives
                first_half = text[i:midpoint + 1]
                second_half = text[midpoint + 1:j + 1]
              
                if first_half == second_half:
                    unique_echo_substrings.add(text[i:j + 1])
  
    return len(unique_echo_substrings)
```

### 2. **Integer Overflow in Hash Calculation**
Even with modulo operations, intermediate calculations can overflow, especially `hash_values[left - 1] * powers[right - left + 1]`.

**Problem Example:**
```python
# This calculation can overflow before applying modulo
(hash_values[left - 1] * powers[right - left + 1]) % MOD
```

**Solution:**
Apply modulo at each step and handle negative values properly:
```python
def get_hash(left: int, right: int) -> int:
    # Apply modulo to each multiplication step
    hash_diff = hash_values[right] - (hash_values[left - 1] * powers[right - left + 1]) % MOD
    # Handle negative values correctly
    return (hash_diff % MOD + MOD) % MOD
```

### 3. **Off-by-One Errors in Index Conversion**
Mixing 0-indexed loop variables with 1-indexed hash array access is error-prone.

**Problem Example:**
```python
# Easy to confuse these indices:
for i in range(n - 1):  # i is 0-indexed
    get_hash(i + 1, k + 1)  # get_hash expects 1-indexed positions
```

**Solution:**
Use consistent indexing throughout or add clear documentation:
```python
def distinctEchoSubstrings(self, text: str) -> int:
    # Alternative: Use 0-indexed throughout
    def get_hash_0indexed(left: int, right: int) -> int:
        """Get hash of text[left:right+1] using 0-based indexing"""
        if left == 0:
            return hash_values[right + 1]
        return (hash_values[right + 1] - hash_values[left] * powers[right - left + 1]) % MOD
  
    # Now the loop is cleaner:
    for start in range(n - 1):
        for end in range(start + 1, n, 2):
            mid = (start + end) // 2
            first_half = get_hash_0indexed(start, mid)
            second_half = get_hash_0indexed(mid + 1, end)
            # ...
```

### 4. **Using Non-Prime Base or Small Modulus**
Using a composite number as base or a small modulus increases collision probability.

**Problem Example:**
```python
BASE = 256  # Power of 2 - higher collision probability
MOD = 10007  # Too small for large strings
```

**Solution:**
Always use large prime numbers:
```python
# Good choices for rolling hash
BASE = 131  # or 137, 139, 149 (prime numbers)
MOD = 10**9 + 7  # or 10**9 + 9, 2**61 - 1 (large primes)

# For extra security, use multiple hash functions
BASE1, MOD1 = 131, 10**9 + 7
BASE2, MOD2 = 137, 10**9 + 9
```

