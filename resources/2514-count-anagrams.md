# 2514. Count Anagrams

> Source: https://algo.monster/liteproblems/2514

## Problem Statement

You have a string `s` that contains one or more words separated by single spaces.

A string `t` is considered an **anagram** of string `s` if each word in `t` is a rearrangement (permutation) of the corresponding word in `s` at the same position.

For example:
- `"acb dfe"` is an anagram of `"abc def"` because:
  - The first word `"acb"` is a permutation of `"abc"`
  - The second word `"dfe"` is a permutation of `"def"`
- `"def cab"` is NOT an anagram of `"abc def"` because the words are in different positions
- `"adc bef"` is NOT an anagram of `"abc def"` because `"adc"` is not a permutation of `"abc"`

Your task is to count how many **distinct anagrams** the string `s` can form. Since the answer can be very large, return it modulo `10^9 + 7`.

The solution calculates this by:
1. For each word in `s`, determining how many different ways its letters can be arranged (permutations)
2. Multiplying the permutation counts of all words together to get the total number of distinct anagrams

The mathematical approach uses the formula for permutations with repetitions: for a word of length `n` with repeated characters, the number of distinct permutations is `n! / (c1! × c2! × ... × ck!)` where `c1, c2, ..., ck` are the frequencies of each unique character.

## Intuition

To find the total number of distinct anagrams, we need to think about what makes anagrams distinct. Since each word must maintain its position (the first word stays first, second stays second, etc.), we can treat each word independently and multiply their possibilities together.

For a single word, the number of distinct arrangements depends on repeated characters. If a word has all unique characters like `"abc"`, we have `3! = 6` arrangements. But if a word is `"aab"`, we have fewer distinct arrangements because swapping the two `'a'`s doesn't create a new arrangement. The formula becomes `3! / 2! = 3` (the arrangements are `"aab"`, `"aba"`, `"baa"`).

This leads us to the general formula: for a word of length `n` with character frequencies `f1, f2, ..., fk`, the number of distinct permutations is:
`n! / (f1! × f2! × ... × fk!)`

The clever part of the solution is how it computes this efficiently. Instead of calculating factorials separately and then dividing, it builds the result incrementally:
- As we process each character at position `i`, we multiply our answer by `i` (building up the numerator `n!`)
- We track the frequency of each character and multiply by that frequency (building up the denominator)
- At the end, we use modular inverse to perform the division under modulo

The modular inverse is necessary because we're working with modulo `10^9 + 7`, and direct division doesn't work properly with modular arithmetic. Using Fermat's Little Theorem, `a^(-1) ≡ a^(p-2) (mod p)` when `p` is prime, which allows us to convert division into multiplication.

This approach processes all words simultaneously, accumulating the total count of anagrams by maintaining the running product of possibilities for each word position.

**Learn more about [Math](/problems/math-basics) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The implementation uses a mathematical approach to count permutations with repetitions, processing each word in the string sequentially.

**Key Variables:**
- `ans`: Accumulates the product of all factorial values (numerators)
- `mul`: Accumulates the product of all character frequency factorials (denominators)
- `mod`: The modulo value `10^9 + 7` to prevent overflow

**Step-by-step Implementation:**

1. **Initialize variables**: Set both `ans` and `mul` to 1 as we'll be building products.

2. **Process each word**: Split the string `s` by spaces and iterate through each word.

3. **For each word**, use a Counter (hash map) to track character frequencies:
   - Process each character at position `i` (1-indexed)
   - Increment the character's count in the Counter
   - Update `mul` by multiplying it with the new count: `mul = mul * cnt[c] % mod`
   - Update `ans` by multiplying it with the position index: `ans = ans * i % mod`

4. **Why this works:**
   - When we multiply `ans` by `i` for each position, we're building up the factorial `n!` for a word of length `n`
   - When we multiply `mul` by `cnt[c]`, we're building up the product of factorials of character frequencies
   - For example, if a character appears 3 times, we multiply by 1, then 2, then 3, giving us `3!`

5. **Final calculation**: 
   - Use modular inverse to divide: `ans * pow(mul, -1, mod) % mod`
   - Python's `pow(mul, -1, mod)` computes the modular inverse of `mul` using Fermat's Little Theorem
   - This effectively performs `ans / mul` under modulo arithmetic

**Example walkthrough** with `s = "aa b"`:
- Word 1: `"aa"`
  - Character 1 (`'a'`): `ans = 1 * 1 = 1`, `mul = 1 * 1 = 1`
  - Character 2 (`'a'`): `ans = 1 * 2 = 2`, `mul = 1 * 2 = 2`
- Word 2: `"b"`
  - Character 1 (`'b'`): `ans = 2 * 1 = 2`, `mul = 2 * 1 = 2`
- Final: `2 * pow(2, -1, mod) % mod = 1`

The result is 1, which represents the single distinct anagram `"aa b"` (since both `'a'`s in the first word are identical).

## Example Walkthrough

Let's walk through the solution with `s = "abc aab"`:

**Step 1: Initialize variables**
- `ans = 1`, `mul = 1`, `mod = 10^9 + 7`

**Step 2: Process first word "abc"**
- Create a Counter for this word
- Process each character with 1-based indexing:

  Position 1, character 'a':
  - `cnt['a'] = 1`
  - `mul = mul * cnt['a'] = 1 * 1 = 1`
  - `ans = ans * 1 = 1 * 1 = 1`

  Position 2, character 'b':
  - `cnt['b'] = 1`
  - `mul = mul * cnt['b'] = 1 * 1 = 1`
  - `ans = ans * 2 = 1 * 2 = 2`

  Position 3, character 'c':
  - `cnt['c'] = 1`
  - `mul = mul * cnt['c'] = 1 * 1 = 1`
  - `ans = ans * 3 = 2 * 3 = 6`

After word 1: `ans = 6` (which is 3!), `mul = 1` (since 1! × 1! × 1! = 1)

**Step 3: Process second word "aab"**
- Create a new Counter for this word
- Process each character:

  Position 1, character 'a':
  - `cnt['a'] = 1`
  - `mul = mul * cnt['a'] = 1 * 1 = 1`
  - `ans = ans * 1 = 6 * 1 = 6`

  Position 2, character 'a':
  - `cnt['a'] = 2`
  - `mul = mul * cnt['a'] = 1 * 2 = 2`
  - `ans = ans * 2 = 6 * 2 = 12`

  Position 3, character 'b':
  - `cnt['b'] = 1`
  - `mul = mul * cnt['b'] = 2 * 1 = 2`
  - `ans = ans * 3 = 12 * 3 = 36`

After word 2: `ans = 36` (which is 3! × 3!), `mul = 2` (which is 2!)

**Step 4: Calculate final result**
- `result = ans * pow(mul, -1, mod) % mod`
- `result = 36 * pow(2, -1, mod) % mod`
- `result = 36 / 2 = 18`

**Verification:**
- First word "abc" has 3! = 6 distinct permutations: "abc", "acb", "bac", "bca", "cab", "cba"
- Second word "aab" has 3!/2! = 3 distinct permutations: "aab", "aba", "baa"
- Total distinct anagrams = 6 × 3 = 18

The 18 distinct anagrams include combinations like "abc aab", "acb aba", "bac baa", etc.

## Python Solution

```python
from collections import Counter

class Solution:
    def countAnagrams(self, s: str) -> int:
        MOD = 10**9 + 7
      
        # total_anagrams tracks the product of anagrams for all words
        total_anagrams = 1
      
        # denominator tracks the product of factorials of character frequencies
        denominator = 1
      
        # Process each word separately
        for word in s.split():
            char_frequency = Counter()
          
            # For each character in the word
            for position, char in enumerate(word, 1):
                # Update character frequency
                char_frequency[char] += 1
              
                # Update denominator: multiply by frequency (building factorial of frequencies)
                denominator = denominator * char_frequency[char] % MOD
              
                # Update numerator: multiply by position (building n!)
                total_anagrams = total_anagrams * position % MOD
      
        # Final result: (n1! * n2! * ... * nk!) / (freq1! * freq2! * ... * freqm!)
        # Using modular inverse for division
        return total_anagrams * pow(denominator, -1, MOD) % MOD
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm iterates through each character in the string exactly once. Let `n` be the total length of the input string `s`.
- Splitting the string by spaces takes `O(n)` time
- For each word, we iterate through its characters once, updating the counter and performing modular arithmetic operations
- The modular arithmetic operations (multiplication and modular inverse) are `O(1)` for each character
- The final modular inverse calculation using `pow(mul, -1, mod)` takes `O(log mod)` time, which is effectively `O(1)` since `mod` is a constant

Therefore, the overall time complexity is `O(n)`.

**Space Complexity:** `O(k)`

Where `k` is the number of unique characters across all words.
- The `Counter` object `cnt` stores at most 26 entries (for lowercase English letters) per word, but it's reused for each word
- The split operation creates a list of words, which in worst case could be `O(n)` if every character is a separate word, but typically would be `O(w)` where `w` is the number of words
- Other variables (`ans`, `mul`, `mod`) use constant space

The dominant space usage is from the counter and the split operation, giving us `O(k)` space complexity where `k` represents the unique characters being tracked at any given time (bounded by 26 for English letters).

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow Without Proper Modulo Operations**

**Pitfall:** Forgetting to apply modulo at each multiplication step, leading to integer overflow even in languages like Python that handle big integers (causing performance issues).

```python
# WRONG: May cause performance issues with very large numbers
total_anagrams = total_anagrams * position  # Missing % MOD
denominator = denominator * char_frequency[char]  # Missing % MOD

# Later applying modulo doesn't help with intermediate overflow
return (total_anagrams * pow(denominator, -1, MOD)) % MOD
```

**Solution:** Apply modulo after every multiplication operation to keep numbers bounded:

```python
# CORRECT: Keep numbers bounded throughout computation
total_anagrams = total_anagrams * position % MOD
denominator = denominator * char_frequency[char] % MOD
```

### 2. **Incorrect Modular Inverse Calculation**

**Pitfall:** Using regular division instead of modular inverse, which doesn't work correctly under modulo arithmetic:

```python
# WRONG: Regular division doesn't work with modulo
return (total_anagrams // denominator) % MOD

# WRONG: Attempting to use modular arithmetic incorrectly
return (total_anagrams % MOD) // (denominator % MOD)
```

**Solution:** Use Python's built-in `pow(base, -1, mod)` for modular inverse (available in Python 3.8+) or implement using Fermat's Little Theorem:

```python
# CORRECT: Using Python 3.8+ modular inverse
return total_anagrams * pow(denominator, -1, MOD) % MOD

# Alternative for older Python versions (Fermat's Little Theorem)
return total_anagrams * pow(denominator, MOD - 2, MOD) % MOD
```

### 3. **Resetting Counter Between Words**

**Pitfall:** Using a single Counter object across all words, which incorrectly accumulates character frequencies:

```python
# WRONG: Single counter for all words
char_frequency = Counter()
for word in s.split():
    for position, char in enumerate(word, 1):
        char_frequency[char] += 1  # Accumulates across words!
        # ...
```

**Solution:** Create a new Counter for each word:

```python
# CORRECT: Fresh counter for each word
for word in s.split():
    char_frequency = Counter()  # New counter for this word
    for position, char in enumerate(word, 1):
        char_frequency[char] += 1
        # ...
```

### 4. **Off-by-One Error in Position Enumeration**

**Pitfall:** Using 0-based indexing when building the factorial:

```python
# WRONG: Starting from 0 breaks factorial calculation
for position, char in enumerate(word):  # position starts at 0
    total_anagrams = total_anagrams * position % MOD  # First multiply by 0!
```

**Solution:** Use 1-based indexing with `enumerate(word, 1)`:

```python
# CORRECT: 1-based indexing for factorial
for position, char in enumerate(word, 1):  # position starts at 1
    total_anagrams = total_anagrams * position % MOD
```

### 5. **Edge Case: Empty String or Single Space**

**Pitfall:** Not handling edge cases where the input might be empty or contain only spaces:

```python
# Potential issue with edge cases
s = ""  # split() returns []
s = " "  # split() returns []
```

**Solution:** The current implementation handles these naturally (returns 1 for empty input), but it's good to be aware and potentially add explicit validation if needed:

```python
def countAnagrams(self, s: str) -> int:
    if not s or not s.strip():
        return 1  # Empty string has 1 anagram (itself)
  
    # Continue with normal logic...
```

