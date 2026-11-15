# 2156. Find Substring With Given Hash Value

> Source: https://algo.monster/liteproblems/2156

## Problem Statement

You are given a string `s` and need to find a specific substring based on a hash function. The hash function for a string of length `k` is calculated as:

`hash(s, p, m) = (val(s[0]) * p^0 + val(s[1]) * p^1 + ... + val(s[k-1]) * p^(k-1)) mod m`

Where `val(s[i])` represents the position of character `s[i]` in the alphabet: `val('a') = 1`, `val('b') = 2`, ..., `val('z') = 26`.

Given:
- A string `s` (0-indexed)
- Integer `power` (the base `p` in the hash formula)
- Integer `modulo` (the modulus `m` in the hash formula)  
- Integer `k` (the length of the substring to find)
- Integer `hashValue` (the target hash value)

Your task is to find and return the **first substring** of `s` with length `k` whose hash value equals `hashValue`.

For example, if we have a substring "abc" with `k=3`, `power=2`, and `modulo=100`, the hash would be:
`hash("abc", 2, 100) = (1 * 2^0 + 2 * 2^1 + 3 * 2^2) = (1 + 4 + 12) = 17`

The problem guarantees that at least one valid substring exists. A substring is defined as a contiguous sequence of characters within the string.

## Intuition

The naive approach would be to check every possible substring of length `k` by calculating its hash value and comparing it with the target `hashValue`. However, recalculating the hash from scratch for each substring would be inefficient.

The key insight is to use a **[sliding window](/problems/sliding_window_maximum)** technique where we can update the hash value incrementally as we move the window. But there's a challenge: when sliding the window forward (left to right), removing the leftmost character from the hash requires division by a power of `p`, which is problematic with modular arithmetic.

Consider how the hash changes when sliding forward:
- Current window: `s[i], s[i+1], ..., s[i+k-1]`
- Next window: `s[i+1], s[i+2], ..., s[i+k]`

To get the new hash, we need to:
1. Remove contribution of `s[i]`: subtract `val(s[i]) * p^0`
2. Shift all remaining terms down by one power (divide by `p`)
3. Add the new character: add `val(s[i+k]) * p^(k-1)`

The division operation in step 2 is complicated with modular arithmetic.

The clever solution is to **traverse the string in reverse**. When moving the window backward:
- Current window: `s[i], s[i+1], ..., s[i+k-1]`  
- Previous window: `s[i-1], s[i], ..., s[i+k-2]`

Now the transformation becomes:
1. Remove the rightmost character: subtract `val(s[i+k-1]) * p^(k-1)`
2. Multiply everything by `p` (shift up by one power)
3. Add the new leftmost character: add `val(s[i-1]) * p^0`

This only involves multiplication and addition, which work nicely with modular arithmetic! We start from the last `k` characters, calculate their hash, then slide the window backward, updating the hash incrementally until we find a match.

**Learn more about [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

Following the reverse traversal strategy, here's how we implement the solution:

**Step 1: Initialize and calculate the hash for the last k characters**

We start by calculating the hash value for the last `k` characters of the string (positions `n-k` to `n-1`):

```python
h = 0
p = 1
for i in range(n - 1, n - 1 - k, -1):
    val = ord(s[i]) - ord("a") + 1
    h = ((h * power) + val) % modulo
    if i != n - k:
        p = p * power % modulo
```

- We traverse from right to left within this window
- For each character, we multiply the current hash by `power` and add the character's value
- We also calculate `p = power^(k-1) % modulo` which represents the highest power coefficient we'll need for removing characters

**Step 2: Slide the window backward through the string**

Now we slide the window from right to left through the entire string:

```python
j = n - k  # Track the starting position of valid substring
for i in range(n - 1 - k, -1, -1):
    pre = ord(s[i + k]) - ord("a") + 1  # Character being removed
    cur = ord(s[i]) - ord("a") + 1      # Character being added
    h = ((h - pre * p) * power + cur) % modulo
```

The hash update formula works as follows:
1. Remove the rightmost character's contribution: `h - pre * p`
2. Shift all terms up by multiplying by `power`: `(h - pre * p) * power`
3. Add the new leftmost character with coefficient `p^0 = 1`: `+ cur`

**Step 3: Check for matches and track the answer**

```python
if h == hashValue:
    j = i
```

Whenever the current window's hash equals the target `hashValue`, we update `j` to store this position. Since we're traversing backward, the last match we find will actually be the first occurrence in the string.

**Step 4: Return the result**

```python
return s[j : j + k]
```

Return the substring starting at position `j` with length `k`.

**Time Complexity:** `O(n)` where `n` is the length of the string, as we process each character at most twice.

**Space Complexity:** `O(1)` as we only use a constant amount of extra space.

## Example Walkthrough

Let's walk through a concrete example to understand how the solution works.

**Given:**
- `s = "fbxzaad"`
- `power = 31`
- `modulo = 100`
- `k = 3`
- `hashValue = 32`

We need to find a substring of length 3 whose hash equals 32.

**Step 1: Calculate hash for the last k characters ("aad")**

Starting with the last 3 characters (indices 4, 5, 6):
- Initialize `h = 0`, `p = 1`
- Process 'd' (index 6): 
  - val('d') = 4
  - `h = (0 * 31 + 4) % 100 = 4`
- Process 'a' (index 5):
  - val('a') = 1
  - `h = (4 * 31 + 1) % 100 = 125 % 100 = 25`
  - `p = 31` (power for position 0)
- Process 'a' (index 4):
  - val('a') = 1
  - `h = (25 * 31 + 1) % 100 = 776 % 100 = 76`
  - `p = 31 * 31 % 100 = 961 % 100 = 61` (power^(k-1))

Hash of "aad" = 76, which ≠ 32, so we continue.

**Step 2: Slide window backward**

**Window at indices [3,4,5] - "zaa":**
- Remove 'd' (index 6): val('d') = 4
- Add 'z' (index 3): val('z') = 26
- `h = ((76 - 4 * 61) * 31 + 26) % 100`
- `h = ((76 - 244) * 31 + 26) % 100`
- `h = ((-168) * 31 + 26) % 100`
- `h = (-5208 + 26) % 100 = -5182 % 100 = 18`

Hash of "zaa" = 18, which ≠ 32, continue.

**Window at indices [2,3,4] - "xza":**
- Remove 'a' (index 5): val('a') = 1
- Add 'x' (index 2): val('x') = 24
- `h = ((18 - 1 * 61) * 31 + 24) % 100`
- `h = ((18 - 61) * 31 + 24) % 100`
- `h = ((-43) * 31 + 24) % 100`
- `h = (-1333 + 24) % 100 = -1309 % 100 = 91`

Hash of "xza" = 91, which ≠ 32, continue.

**Window at indices [1,2,3] - "bxz":**
- Remove 'a' (index 4): val('a') = 1
- Add 'b' (index 1): val('b') = 2
- `h = ((91 - 1 * 61) * 31 + 2) % 100`
- `h = ((91 - 61) * 31 + 2) % 100`
- `h = (30 * 31 + 2) % 100`
- `h = (930 + 2) % 100 = 932 % 100 = 32`

Hash of "bxz" = 32, which equals our target! Update `j = 1`.

**Window at indices [0,1,2] - "fbx":**
- Remove 'z' (index 3): val('z') = 26
- Add 'f' (index 0): val('f') = 6
- `h = ((32 - 26 * 61) * 31 + 6) % 100`
- `h = ((32 - 1586) * 31 + 6) % 100`
- `h = ((-1554) * 31 + 6) % 100`
- `h = (-48174 + 6) % 100 = -48168 % 100 = 32`

Hash of "fbx" = 32, which also equals our target! Update `j = 0`.

**Step 3: Return the result**

Since we traversed backward and the last matching position was `j = 0`, we return `s[0:3] = "fbx"`.

This demonstrates why reverse traversal is clever: it naturally gives us the **first** occurrence of a matching substring, and the rolling hash update only requires multiplication and addition (no division needed).

## Python Solution

```python
class Solution:
    def subStrHash(
        self, s: str, power: int, modulo: int, k: int, hashValue: int
    ) -> str:
        """
        Find the substring of length k whose hash equals hashValue.
        Hash formula: (val(s[0]) * power^0 + val(s[1]) * power^1 + ... + val(s[k-1]) * power^(k-1)) % modulo
        where val(c) = ord(c) - ord('a') + 1
        """
        # Initialize variables
        current_hash = 0
        string_length = len(s)
        power_k_minus_1 = 1  # Will store power^(k-1) % modulo
      
        # Calculate hash for the last k characters (rightmost substring)
        # Building hash from right to left within the window
        for i in range(string_length - 1, string_length - 1 - k, -1):
            char_value = ord(s[i]) - ord("a") + 1
            current_hash = (current_hash * power + char_value) % modulo
          
            # Calculate power^(k-1) % modulo for later use in rolling hash
            if i != string_length - k:
                power_k_minus_1 = (power_k_minus_1 * power) % modulo
      
        # Initialize result position with the last possible substring
        result_position = string_length - k
      
        # Roll the hash window from right to left through the string
        for i in range(string_length - 1 - k, -1, -1):
            # Remove the rightmost character from current window
            removed_char_value = ord(s[i + k]) - ord("a") + 1
            # Add the new leftmost character to current window
            new_char_value = ord(s[i]) - ord("a") + 1
          
            # Update rolling hash: remove old character contribution, shift left, add new character
            current_hash = ((current_hash - removed_char_value * power_k_minus_1) * power + new_char_value) % modulo
          
            # Check if current hash matches target
            if current_hash == hashValue:
                result_position = i
      
        # Return the substring starting at result_position with length k
        return s[result_position : result_position + k]
```

## Time and Space Complexity

**Time Complexity:** `O(n)`, where `n` is the length of the string `s`.

The algorithm consists of two main loops:
1. The first loop runs from `n-1` to `n-k`, iterating exactly `k` times to compute the initial hash value of the last substring of length `k`. This takes `O(k)` time.
2. The second loop runs from `n-1-k` to `0`, iterating `n-k` times. In each iteration, it performs constant time operations:
   - Computing the hash of the new substring using the rolling hash technique by removing the contribution of the character going out of the window (`s[i+k]`) and adding the contribution of the new character (`s[i]`)
   - Comparing the hash value with the target `hashValue`

Since `k ≤ n`, the total time complexity is `O(k) + O(n-k) = O(n)`.

**Space Complexity:** `O(1)`

The algorithm uses only a constant amount of extra space:
- Variables `h`, `n`, `p`, `j` to store the hash value, string length, power value, and result index
- Variables `pre`, `cur` to store character values during the rolling hash computation

The space used does not depend on the input size, making the space complexity `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Negative Modulo Values**

The most critical pitfall in this solution is handling negative values during the modulo operation. When we compute `(current_hash - removed_char_value * power_k_minus_1) * power % modulo`, the subtraction can result in a negative number.

**Why this happens:**
- In Python, the modulo operation with negative numbers can produce unexpected results
- For example, if `current_hash = 5`, `removed_char_value * power_k_minus_1 = 10`, and `modulo = 7`, then `(5 - 10) % 7` gives `-5` in intermediate calculation

**Solution:**
Add `modulo` before taking the final modulo to ensure a positive result:
```python
current_hash = ((current_hash - removed_char_value * power_k_minus_1 % modulo + modulo) * power + new_char_value) % modulo
```

### 2. **Integer Overflow in Power Calculation**

When calculating `power_k_minus_1`, repeated multiplication can cause integer overflow for large values of `k` and `power`.

**Solution:**
Apply modulo operation at each multiplication step:
```python
if i != string_length - k:
    power_k_minus_1 = (power_k_minus_1 * power) % modulo  # Already correct in the code
```

### 3. **Incorrect Hash Calculation Direction**

A common mistake is calculating the initial hash from left to right instead of right to left within the window, which would give incorrect power coefficients.

**Why this matters:**
- The hash formula requires `s[0] * power^0 + s[1] * power^1 + ...`
- Building from right to left naturally gives us the correct coefficients when we multiply by `power` at each step

**Solution:**
Ensure the initial hash calculation traverses from right to left:
```python
for i in range(string_length - 1, string_length - 1 - k, -1):
    current_hash = (current_hash * power + char_value) % modulo
```

### 4. **Off-by-One Errors in Window Boundaries**

It's easy to make mistakes with index boundaries when sliding the window, especially when working backward.

**Common mistakes:**
- Starting the sliding window at wrong position
- Incorrectly calculating which character to remove/add

**Solution:**
Carefully track indices:
- Window starts at position `i` and ends at position `i + k - 1`
- Character being removed is at position `i + k`
- Character being added is at position `i`

### 5. **Forgetting to Apply Modulo Consistently**

Missing modulo operations at any step can lead to incorrect hash values or overflow.

**Solution:**
Apply modulo after every arithmetic operation that could exceed the modulo value:
```python
current_hash = ((current_hash - removed_char_value * power_k_minus_1 % modulo + modulo) % modulo * power % modulo + new_char_value) % modulo
```

### Corrected Critical Section:
The most important fix to apply to the original code:
```python
# Original (potentially buggy)
current_hash = ((current_hash - removed_char_value * power_k_minus_1) * power + new_char_value) % modulo

# Corrected (handles negative values properly)
current_hash = ((current_hash - removed_char_value * power_k_minus_1 % modulo + modulo) % modulo * power + new_char_value) % modulo
```

