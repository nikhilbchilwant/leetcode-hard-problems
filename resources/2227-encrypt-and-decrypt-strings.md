# 2227. Encrypt and Decrypt Strings

> Source: https://algo.monster/liteproblems/2227

## Problem Statement

You need to implement a data structure that can encrypt and decrypt strings based on a given cipher system.

You're given three inputs:
- `keys`: An array of unique characters
- `values`: An array of strings (each of length 2) that correspond to the encrypted form of each character in `keys`
- `dictionary`: An array of valid original strings that can appear after decryption

**Encryption Process:**
To encrypt a string, you replace each character with its corresponding encrypted value:
1. For each character `c` in the input string, find its index `i` where `keys[i] == c`
2. Replace `c` with `values[i]` (which is a 2-character string)
3. If any character in the input string doesn't exist in `keys`, the encryption fails and returns an empty string `""`

For example, if `keys = ['a', 'b']` and `values = ["aa", "bb"]`, then encrypting `"ab"` would give `"aabb"`.

**Decryption Process:**
To decrypt a string, you reverse the process by replacing every 2-character substring (at even indices) with its original character:
1. Take each substring of length 2 at even positions (0, 2, 4, ...)
2. Find the index `i` where `values[i]` equals this substring
3. Replace the substring with `keys[i]`
4. Note: Multiple characters might encrypt to the same value, so one encrypted string could decrypt to multiple possible original strings

The key challenge is that decryption isn't unique - multiple original strings might encrypt to the same result.

**Required Methods:**
1. `Encrypter(keys, values, dictionary)`: Initialize the encrypter with the given parameters
2. `encrypt(word1)`: Encrypt `word1` and return the encrypted string
3. `decrypt(word2)`: Return the count of how many strings in `dictionary` would encrypt to `word2`

The clever part of the solution is pre-computing all encrypted forms of dictionary words during initialization, then simply looking up the count when `decrypt()` is called, rather than trying to decrypt and check each possibility.

## Intuition

The key insight comes from understanding the asymmetry between encryption and decryption. While encryption is straightforward (each character maps to exactly one encrypted value), decryption is complex because multiple characters might encrypt to the same value, creating ambiguity.

Initially, you might think to implement `decrypt()` by actually decrypting the string and checking all possible original strings against the dictionary. However, this becomes complicated because:
1. We'd need to handle multiple possible decryptions due to non-unique mappings
2. We'd have to generate all possible combinations when multiple characters map to the same encrypted value
3. Each decrypt call would require significant computation

The breakthrough comes from reversing our perspective: instead of decrypting `word2` to find matches in the dictionary, why not encrypt everything in the dictionary first and count how many encrypt to `word2`?

This approach is brilliant because:
- Encryption is deterministic - each dictionary word has exactly one encrypted form
- We can pre-compute all encrypted dictionary words once during initialization
- The `decrypt()` function becomes a simple lookup operation

Think of it like this: if you want to know how many English words become "xyz" when encoded, instead of trying to decode "xyz" in all possible ways and checking which are valid English words, you encode all English words first and count how many become "xyz".

By using a `Counter` to store the frequency of each encrypted dictionary word, we transform a complex decryption problem into a simple counting problem. The `decrypt()` function just returns `cnt[word2]` - the pre-computed count of how many dictionary words encrypt to `word2`.

This pre-computation strategy trades initialization time and space for extremely fast decrypt operations, which is often the right tradeoff when decrypt might be called many times.

**Learn more about [Trie](/problems/trie_intro) patterns.**

## Solution Approach

The solution uses two hash tables to efficiently handle encryption and decryption operations:

1. **Hash Table `mp`**: Maps each character to its encrypted value
2. **Hash Table `cnt`**: Stores the frequency of encrypted forms of all dictionary words

**Constructor Implementation:**

```python
def __init__(self, keys: List[str], values: List[str], dictionary: List[str]):
    self.mp = dict(zip(keys, values))
    self.cnt = Counter(self.encrypt(v) for v in dictionary)
```

- `dict(zip(keys, values))` creates a mapping where `mp[keys[i]] = values[i]`. This gives us O(1) lookup for encryption.
- We iterate through each word `v` in the dictionary, encrypt it using our `encrypt` method, and count the frequency of each encrypted result using Python's `Counter`.
- Time complexity: O(n + m × l) where n is the length of keys, m is the dictionary size, and l is the average word length.

**Encryption Implementation:**

```python
def encrypt(self, word1: str) -> str:
    res = []
    for c in word1:
        if c not in self.mp:
            return ''
        res.append(self.mp[c])
    return ''.join(res)
```

- For each character `c` in `word1`, we look up its encrypted value in `mp`.
- If any character doesn't exist in `mp`, encryption is impossible, so we return an empty string.
- Otherwise, we append the 2-character encrypted value to our result list.
- Finally, we join all encrypted values to form the complete encrypted string.
- Time complexity: O(k) where k is the length of `word1`.

**Decryption Implementation:**

```python
def decrypt(self, word2: str) -> int:
    return self.cnt[word2]
```

- Since we pre-computed how many dictionary words encrypt to each possible string, decryption is just a lookup.
- We return `self.cnt[word2]`, which gives the count of dictionary words that encrypt to `word2`.
- If `word2` isn't in `cnt`, Python's Counter returns 0 by default.
- Time complexity: O(1) for the lookup operation.

**Why This Approach Works:**

The clever trick is recognizing that we don't need to actually decrypt strings. Instead, we pre-encrypt all valid dictionary words and count their encrypted forms. This transforms the complex problem of "which dictionary words could this decrypt to?" into the simple problem of "how many dictionary words encrypt to this?"

The space complexity is O(n + m × l) for storing the mappings and encrypted dictionary counts, where n is the number of keys, m is the dictionary size, and l is the average length of encrypted words.

## Example Walkthrough

Let's walk through a concrete example to understand how the solution works.

**Given:**
- `keys = ['a', 'b', 'c']`
- `values = ["ei", "zf", "ei"]`  (note: 'a' and 'c' both encrypt to "ei")
- `dictionary = ["abcd", "acbd", "adbc", "badc"]`

**Step 1: Initialization**

First, we create the character-to-encryption mapping:
- `mp = {'a': "ei", 'b': "zf", 'c': "ei"}`

Next, we encrypt each dictionary word and count frequencies:
- `"abcd"` → `"ei" + "zf" + "ei" + ?` → Can't encrypt 'd' (not in keys) → returns `""`
- `"acbd"` → `"ei" + "ei" + "zf" + ?` → Can't encrypt 'd' → returns `""`
- `"adbc"` → Can't encrypt 'd' → returns `""`
- `"badc"` → `"zf" + "ei" + ?` → Can't encrypt 'd' → returns `""`

Since 'd' isn't in our keys, none of these words can be encrypted. Let's modify the dictionary to `["abc", "bac", "cab", "cba"]`:

- `"abc"` → `"ei" + "zf" + "ei"` = `"eizfei"`
- `"bac"` → `"zf" + "ei" + "ei"` = `"zfeiei"`
- `"cab"` → `"ei" + "ei" + "zf"` = `"eieizf"`
- `"cba"` → `"ei" + "zf" + "ei"` = `"eizfei"`

Our counter becomes:
- `cnt = {"eizfei": 2, "zfeiei": 1, "eieizf": 1}`

**Step 2: Encrypt Operation**

If we call `encrypt("cab")`:
1. 'c' → look up in `mp` → `"ei"`
2. 'a' → look up in `mp` → `"ei"`
3. 'b' → look up in `mp` → `"zf"`
4. Join them: `"eieizf"`

**Step 3: Decrypt Operation**

If we call `decrypt("eizfei")`:
- Simply look up in `cnt["eizfei"]` → returns `2`

This tells us that 2 dictionary words (`"abc"` and `"cba"`) encrypt to `"eizfei"`.

**Why this is clever:**
Notice that `"eizfei"` could theoretically decrypt to multiple strings because both 'a' and 'c' map to `"ei"`. The encrypted string `"eizfei"` could represent:
- `"abc"` (ei=a, zf=b, ei=c)
- `"cba"` (ei=c, zf=b, ei=a)
- `"aba"` (ei=a, zf=b, ei=a)
- `"cbc"` (ei=c, zf=b, ei=c)

But we only care about which ones are in our dictionary! By pre-encrypting the dictionary words, we avoid the complex task of generating all possible decryptions and checking each one. Instead, we just count how many dictionary words produce this encrypted result.

## Python Solution

```python
from typing import List
from collections import Counter


class Encrypter:
    def __init__(self, keys: List[str], values: List[str], dictionary: List[str]):
        """
        Initialize the Encrypter with encryption mappings and a dictionary.
      
        Args:
            keys: List of characters to be encrypted
            values: List of corresponding encrypted strings for each key
            dictionary: List of valid words to be used for decryption counting
        """
        # Create a mapping from each key character to its encrypted value
        self.encryption_map = dict(zip(keys, values))
      
        # Pre-compute encrypted versions of all dictionary words
        # and count how many dictionary words encrypt to each result
        self.encrypted_word_count = Counter(
            self.encrypt(word) for word in dictionary
        )

    def encrypt(self, word1: str) -> str:
        """
        Encrypt a word by replacing each character with its mapped value.
      
        Args:
            word1: The word to encrypt
          
        Returns:
            The encrypted string, or empty string if any character cannot be encrypted
        """
        encrypted_parts = []
      
        # Process each character in the word
        for char in word1:
            # If character has no encryption mapping, return empty string
            if char not in self.encryption_map:
                return ''
            # Add the encrypted value for this character
            encrypted_parts.append(self.encryption_map[char])
      
        # Join all encrypted parts into final encrypted word
        return ''.join(encrypted_parts)

    def decrypt(self, word2: str) -> int:
        """
        Return the count of dictionary words that could encrypt to the given string.
      
        Args:
            word2: The encrypted string to decrypt
          
        Returns:
            Number of possible original words from the dictionary
        """
        # Return pre-computed count of dictionary words that encrypt to word2
        return self.encrypted_word_count[word2]


# Your Encrypter object will be instantiated and called as such:
# obj = Encrypter(keys, values, dictionary)
# param_1 = obj.encrypt(word1)
# param_2 = obj.decrypt(word2)
```

## Time and Space Complexity

**Time Complexity:**
- `__init__`: `O(n * k + m)` where `n` is the length of the dictionary, `k` is the average length of words in the dictionary, and `m` is the length of the keys array. Creating the mapping takes `O(m)` time, and encrypting all dictionary words takes `O(n * k)` time since each word needs to be processed character by character.

- `encrypt`: `O(k)` where `k` is the length of the input word. Each character lookup in the mapping is `O(1)`, and we process each character once.

- `decrypt`: `O(1)` since it's just a dictionary lookup in the Counter object.

**Space Complexity:** `O(n * k + m)`
- The mapping `self.mp` stores `m` key-value pairs: `O(m)`
- The Counter `self.cnt` stores up to `n` encrypted strings, where each encrypted string can be up to `2k` characters long (since each character maps to a 2-character string based on the problem context): `O(n * k)`
- Overall space complexity is `O(n * k + m)`, which can be simplified to `O(n + m)` when considering that `k` is typically bounded by a constant in most practical scenarios.

The reference answer `O(n + m)` assumes that the average word length `k` is constant or bounded, which is a reasonable assumption for most dictionary problems.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Attempting to Actually Decrypt the String**

**The Pitfall:** 
A natural instinct is to implement the `decrypt` method by actually reversing the encryption process - parsing the encrypted string into 2-character chunks, finding matching values, and building all possible original strings. This approach has several problems:
- One encrypted string can map to multiple original strings (when different characters encrypt to the same value)
- The decryption tree can grow exponentially with the string length
- You'd need to check each decrypted possibility against the dictionary

**Why It's Wrong:**
```python
# INCORRECT approach - trying to actually decrypt
def decrypt(self, word2: str) -> int:
    # This would require complex backtracking to find all possibilities
    possible_words = []
    # Parse word2 into 2-char chunks and try to reverse map...
    # This becomes exponentially complex!
```

**The Solution:**
Pre-compute all encrypted forms of dictionary words during initialization, then simply look up the count. This transforms an exponential problem into O(1) lookup.

### 2. **Not Handling Empty Strings in Counter**

**The Pitfall:**
When encrypting dictionary words in the constructor, some words might fail to encrypt (return empty string). If you don't handle this, the Counter might include empty strings.

```python
# PROBLEMATIC: Empty strings get counted
self.encrypted_word_count = Counter(
    self.encrypt(word) for word in dictionary
)
# If encrypt returns "", Counter will count empty strings
```

**The Solution:**
Filter out empty strings before counting:
```python
self.encrypted_word_count = Counter(
    encrypted for word in dictionary 
    if (encrypted := self.encrypt(word))
)
```

### 3. **Assuming One-to-One Mapping**

**The Pitfall:**
Incorrectly assuming that each character maps to a unique encrypted value, or that each encrypted value maps back to a unique character. The problem explicitly allows multiple characters to encrypt to the same value.

**Why It Matters:**
If `keys = ['a', 'b']` and `values = ['xx', 'xx']`, both 'a' and 'b' encrypt to 'xx'. So 'xx' could decrypt to either 'a' or 'b', and 'xxxx' could decrypt to 'aa', 'ab', 'ba', or 'bb'.

### 4. **Incorrect String Parsing in Decryption Logic**

**The Pitfall:**
If you were to implement actual decryption, you might incorrectly parse the encrypted string:
```python
# WRONG: Assumes we need to decrypt at odd positions or variable lengths
for i in range(0, len(word2), 2):
    chunk = word2[i:i+2]  # What if len(word2) is odd?
```

**The Solution:**
The problem guarantees that encrypted strings have even length (each character becomes 2 characters), but our approach avoids this parsing entirely by pre-computing encrypted forms.

### 5. **Not Using Efficient Data Structures**

**The Pitfall:**
Using lists and linear search instead of hash tables:
```python
# INEFFICIENT: O(n) lookup for each character
def encrypt(self, word1: str) -> str:
    result = []
    for c in word1:
        for i, key in enumerate(self.keys):  # O(n) search
            if key == c:
                result.append(self.values[i])
                break
        else:
            return ''
```

**The Solution:**
Use a dictionary for O(1) lookup: `self.encryption_map = dict(zip(keys, values))`

