# 1830. Minimum Number of Operations to Make String Sorted

> Source: https://algo.monster/liteproblems/1830

## Problem Statement

You are given a string `s` (0-indexed). You need to perform a specific set of operations on the string repeatedly until it becomes sorted. Each operation consists of four steps:

1. Find the **largest index** `i` such that `1 <= i < s.length` and `s[i] < s[i - 1]`. This means finding the rightmost position where a character is smaller than the character before it.

2. Find the **largest index** `j` such that `i <= j < s.length` and `s[k] < s[i - 1]` for all possible values of `k` in the range `[i, j]` inclusive. This means finding the rightmost position starting from `i` where all characters from position `i` to `j` are smaller than `s[i - 1]`.

3. Swap the two characters at indices `i - 1` and `j`.

4. Reverse the suffix starting at index `i`.

The problem asks you to return the **number of operations** needed to make the string sorted in ascending order. Since the answer can be very large, return it modulo `10^9 + 7`.

The operation described is essentially finding the previous permutation of the current string in lexicographical order. The solution uses the fact that counting the number of operations is equivalent to counting how many permutations of the string are lexicographically smaller than the given string. This is because each operation transforms the string into its immediate lexicographical predecessor, so the number of operations equals the number of permutations between the sorted string and the original string.

## Intuition

The key insight is recognizing that the described operation is actually finding the **previous permutation** in lexicographical order. Each operation transforms the string into the lexicographically previous permutation, moving step by step toward the sorted (smallest) permutation.

Think of it this way: if we start with "dcba" and keep applying the operation, we get "dcab" → "dbca" → "dbac" → ... → "abcd". Each step moves to the previous permutation until we reach the sorted string "abcd", which is the lexicographically smallest.

Since each operation moves exactly one step backward in the lexicographical ordering, the number of operations equals the number of permutations that come before our starting string in lexicographical order. This transforms the problem from simulation to **counting**.

To count permutations smaller than our string, we can think position by position from left to right. At each position `i`, we ask: "How many valid permutations would we get if we placed a smaller character here instead?"

For example, with string "dca":
- At position 0, we could place 'a', 'b', or 'c' (all smaller than 'd'). Each choice would lead to permutations smaller than "dca"
- For each such choice, we need to count how many ways we can arrange the remaining characters

The challenge with duplicate characters is handled using the multinomial coefficient formula. If we have `n` total characters with frequencies `n₁, n₂, ..., nₖ` for each distinct character, the number of unique permutations is `n! / (n₁! × n₂! × ... × nₖ!)`.

This approach avoids simulating each operation (which would be too slow) and instead directly computes the answer using combinatorial mathematics. We precompute factorials and their modular inverses to efficiently calculate these values as we traverse the string.

**Learn more about [Math](/problems/math-basics) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The solution uses **counting**, **permutation and combination**, and **preprocessing** to efficiently calculate the answer.

### Preprocessing Factorials and Modular Inverses

First, we precompute:
- Array `f` where `f[i] = i!` (factorial of i)
- Array `g` where `g[i]` is the modular inverse of `f[i]`

The modular inverse is calculated using Fermat's Little Theorem: for prime modulus `p`, the inverse of `a` is `a^(p-2) mod p`. This allows us to perform division under modulo by converting it to multiplication with the inverse.

```python
n = 3010
mod = 10**9 + 7
f = [1] + [0] * n  # f[i] will store i!
g = [1] + [0] * n  # g[i] will store inverse of i!

for i in range(1, n):
    f[i] = f[i - 1] * i % mod
    g[i] = pow(f[i], mod - 2, mod)  # Fermat's Little Theorem
```

### Main Algorithm

1. **Initialize a frequency counter** for all characters in the string using `Counter(s)`.

2. **Traverse the string from left to right**. For each position `i` with character `c`:
 
   a. **Count smaller characters**: Find how many characters in the remaining frequency map are smaller than `c`. This gives us `m` - the number of ways we could place a smaller character at this position.
 
   b. **Calculate permutations**: For each way of placing a smaller character, we need to count the permutations of the remaining `n - i - 1` characters. Using the multinomial coefficient:
      - Start with `t = f[n - i - 1] * m` (total remaining positions × number of smaller characters)
      - Divide by the factorial of each character's frequency: `t = t * g[v]` for each frequency `v`
      - This gives us `m × (n-i-1)! / (n₁! × n₂! × ... × nₖ!)`
 
   c. **Add to answer**: Add `t` to our running total (with modulo).
 
   d. **Update frequency**: Decrease the frequency of character `c` by 1, as we've "used" it at position `i`. Remove it from the counter if frequency becomes 0.

```python
class Solution:
    def makeStringSorted(self, s: str) -> int:
        cnt = Counter(s)
        ans, n = 0, len(s)
      
        for i, c in enumerate(s):
            # Count characters smaller than c
            m = sum(v for a, v in cnt.items() if a < c)
          
            # Calculate permutations with smaller character at position i
            t = f[n - i - 1] * m
            for v in cnt.values():
                t = t * g[v] % mod
          
            # Add to answer
            ans = (ans + t) % mod
          
            # Update frequency counter
            cnt[c] -= 1
            if cnt[c] == 0:
                cnt.pop(c)
              
        return ans
```

The algorithm efficiently counts all lexicographically smaller permutations without generating them, achieving a time complexity of `O(n × k)` where `n` is the string length and `k` is the number of distinct characters (at most 26 for lowercase letters).

## Example Walkthrough

Let's walk through the solution with the string `s = "bca"`.

**Initial Setup:**
- String: "bca"
- Frequency counter: {'b': 1, 'c': 1, 'a': 1}
- We need to count how many permutations are lexicographically smaller than "bca"

**Position 0 (character 'b'):**
- Current character: 'b'
- Characters smaller than 'b' in our frequency map: only 'a' (count = 1)
- So we have `m = 1` way to place a smaller character here
- If we place 'a' at position 0, we need to arrange the remaining 2 characters ('b' and 'c')
- Number of permutations: `1 × 2! / (1! × 1!) = 1 × 2 / 1 = 2`
- These permutations are: "abc" and "acb" (both start with 'a' which is smaller than 'b')
- Add 2 to our answer: `ans = 2`
- Update frequency: {'b': 0, 'c': 1, 'a': 1} → {'c': 1, 'a': 1}

**Position 1 (character 'c'):**
- Current character: 'c'
- Characters smaller than 'c' in remaining frequency map: only 'a' (count = 1)
- So we have `m = 1` way to place a smaller character here
- If we place 'a' at position 1 (keeping 'b' at position 0), we need to arrange the remaining 1 character ('c')
- Number of permutations: `1 × 1! / 1! = 1`
- This permutation is: "bac" (which has 'ba' as prefix and is smaller than "bca")
- Add 1 to our answer: `ans = 2 + 1 = 3`
- Update frequency: {'c': 0, 'a': 1} → {'a': 1}

**Position 2 (character 'a'):**
- Current character: 'a'
- Characters smaller than 'a' in remaining frequency map: none
- So `m = 0`
- No permutations to add: `ans = 3`

**Final Answer: 3**

This means there are 3 permutations lexicographically smaller than "bca":
1. "abc" (smallest)
2. "acb" 
3. "bac"

And indeed, if we started with "bca" and applied the operation 3 times:
- "bca" → "bac" (1st operation)
- "bac" → "acb" (2nd operation)  
- "acb" → "abc" (3rd operation, now sorted)

The solution correctly counts these without simulating each operation!

## Python Solution

```python
from collections import Counter
from typing import List

# Precompute factorials and their modular inverses
MAX_N = 3010
MOD = 10**9 + 7

# factorial[i] = i! mod MOD
factorial = [1] + [0] * MAX_N
# factorial_inverse[i] = (i!)^(-1) mod MOD
factorial_inverse = [1] + [0] * MAX_N

for i in range(1, MAX_N):
    factorial[i] = factorial[i - 1] * i % MOD
    factorial_inverse[i] = pow(factorial[i], MOD - 2, MOD)


class Solution:
    def makeStringSorted(self, s: str) -> int:
        """
        Calculate the number of permutations of string s that come before s
        in lexicographical order.
      
        The algorithm works by counting how many permutations would start with
        a character smaller than the current character at each position.
        """
        # Count frequency of each character in the string
        char_count = Counter(s)
        result = 0
        string_length = len(s)
      
        for position, current_char in enumerate(s):
            # Count characters smaller than current character
            smaller_chars_count = sum(
                freq for char, freq in char_count.items() 
                if char < current_char
            )
          
            # Calculate number of permutations starting with a smaller character
            # This is: (remaining_positions)! * smaller_chars_count / product(freq!)
            remaining_positions = string_length - position - 1
            permutations = factorial[remaining_positions] * smaller_chars_count
          
            # Divide by factorial of each character frequency (for duplicate handling)
            for frequency in char_count.values():
                permutations = permutations * factorial_inverse[frequency] % MOD
          
            # Add to result
            result = (result + permutations) % MOD
          
            # Update character count for next iteration
            char_count[current_char] -= 1
            if char_count[current_char] == 0:
                char_count.pop(current_char)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n × k)`

The main loop iterates through each character in string `s` (length `n`). For each iteration:
- Computing `m = sum(v for a, v in cnt.items() if a < c)` takes `O(k)` time where `k` is the number of distinct characters
- The inner loop `for v in cnt.values()` also iterates through at most `k` values
- Other operations like modular arithmetic and dictionary updates are `O(1)`

Therefore, the overall time complexity is `O(n × k)`.

**Space Complexity:** `O(n)`

The space usage includes:
- Arrays `f` and `g` each of size `n + 1`, contributing `O(n)` space
- Counter `cnt` stores at most `k` distinct characters and their counts, which is `O(k)` and bounded by `O(n)`
- Other variables use constant space

Since `k ≤ n` (the number of distinct characters cannot exceed the string length), and the arrays `f` and `g` dominate the space usage, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow Without Proper Modulo Operations**

One of the most common pitfalls is forgetting to apply modulo operations at every multiplication step, which can lead to integer overflow even in languages like Python that handle big integers.

**Incorrect approach:**
```python
# Wrong - might overflow before taking modulo
permutations = factorial[remaining_positions] * smaller_chars_count
for frequency in char_count.values():
    permutations = permutations * factorial_inverse[frequency]
result = (result + permutations) % MOD  # Too late!
```

**Correct approach:**
```python
# Apply modulo after each multiplication
permutations = factorial[remaining_positions] * smaller_chars_count % MOD
for frequency in char_count.values():
    permutations = permutations * factorial_inverse[frequency] % MOD
result = (result + permutations) % MOD
```

### 2. **Incorrect Precomputation Bounds**

Another pitfall is setting the precomputation array size too small. If `MAX_N` is less than the actual string length, the code will crash with an index out of bounds error.

**Problem scenario:**
```python
MAX_N = 100  # Too small!
# If string length is 3000, factorial[2999] will cause IndexError
```

**Solution:**
Always set `MAX_N` to be at least as large as the maximum possible string length plus some buffer:
```python
MAX_N = 3010  # Safe for strings up to length 3000
```

### 3. **Not Handling Character Removal from Counter Properly**

Forgetting to remove characters with zero frequency from the counter can lead to incorrect calculations in subsequent iterations.

**Incorrect approach:**
```python
char_count[current_char] -= 1
# Forgot to remove when count becomes 0
# This leaves 0-frequency entries that affect the division calculation
```

**Correct approach:**
```python
char_count[current_char] -= 1
if char_count[current_char] == 0:
    char_count.pop(current_char)  # Remove to avoid division by 0!
```

### 4. **Misunderstanding the Multinomial Coefficient Formula**

A subtle pitfall is incorrectly implementing the multinomial coefficient. The formula should divide by the factorial of ALL character frequencies, not just the ones being considered for placement.

**Wrong interpretation:**
```python
# Incorrect - only dividing by frequency of smaller characters
for char, freq in char_count.items():
    if char < current_char:
        permutations = permutations * factorial_inverse[freq] % MOD
```

**Correct implementation:**
```python
# Correct - divide by factorial of ALL remaining character frequencies
for frequency in char_count.values():
    permutations = permutations * factorial_inverse[frequency] % MOD
```

### 5. **Off-by-One Error in Remaining Positions**

Calculating the wrong number of remaining positions is a common mistake that leads to incorrect permutation counts.

**Common mistake:**
```python
remaining_positions = string_length - position  # Wrong!
# This includes the current position
```

**Correct calculation:**
```python
remaining_positions = string_length - position - 1  # Correct
# Excludes the current position since we're fixing it
```

