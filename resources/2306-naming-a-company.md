# 2306. Naming a Company

> Source: https://algo.monster/liteproblems/2306

## Problem Statement

You have an array of strings called `ideas` that contains potential names for a company. To create a company name, you need to follow these steps:

1. Pick two different names from the `ideas` array - let's call them `ideaA` and `ideaB`.
2. Swap the first letters of these two names with each other. For example, if `ideaA = "coffee"` and `ideaB = "donuts"`, after swapping first letters you get `"doffee"` and `"conuts"`.
3. Check if both newly formed words are NOT in the original `ideas` array. If both new words don't exist in `ideas`, then you can create a valid company name by concatenating them with a space: `"doffee conuts"`.
4. If either of the new words already exists in `ideas`, this combination doesn't form a valid company name.

Your task is to count how many distinct valid company names can be formed using this process.

For example, if `ideas = ["coffee", "donuts", "time", "toffee"]`:
- Swapping first letters of `"coffee"` and `"donuts"` gives `"doffee"` and `"conuts"`. Since neither exists in `ideas`, `"doffee conuts"` is valid.
- Swapping first letters of `"coffee"` and `"toffee"` gives `"toffee"` and `"coffee"`. Both exist in `ideas`, so this is NOT valid.
- Swapping first letters of `"donuts"` and `"coffee"` gives `"conuts"` and `"doffee"`. Since neither exists in `ideas`, `"conuts doffee"` is valid (this is different from `"doffee conuts"`).

The function should return the total count of all possible distinct valid company names.

## Intuition

The brute force approach would be to try all pairs of ideas and check if swapping their first letters creates two new valid words. However, with potentially thousands of ideas, checking all pairs would be inefficient.

Let's think about what makes a valid pair. For two words `ideaA` and `ideaB` to form a valid company name after swapping first letters:
- When we replace `ideaA`'s first letter with `ideaB`'s first letter, the resulting word shouldn't exist in `ideas`
- When we replace `ideaB`'s first letter with `ideaA`'s first letter, the resulting word shouldn't exist in `ideas`

The key insight is that we can precompute this information. Since there are only 26 possible first letters (a-z), we can create a 2D array `f[i][j]` where:
- `i` represents the original first letter (0-25 for a-z)
- `j` represents the replacement first letter (0-25 for a-z)
- `f[i][j]` counts how many words starting with letter `i` can have their first letter replaced with letter `j` without creating a word that exists in `ideas`

Once we have this precomputed table, we can efficiently count valid pairs. For each word in `ideas`, we check which letters it can swap with (creating a non-existing word). For each such valid replacement letter `j`, we add `f[j][i]` to our answer. This counts all the words that start with letter `j` and can swap with our current word's first letter `i` to form valid pairs.

This approach avoids checking every pair explicitly. Instead, we use the fact that the validity of a swap depends only on the first letters and whether the resulting words exist in our set, allowing us to group and count efficiently based on first letters.

## Solution Approach

We implement the solution using a counting approach with preprocessing:

**Step 1: Initialize Data Structures**
- Create a hash set `s` containing all ideas for O(1) lookup
- Create a 2D array `f[26][26]` initialized to zeros, where `f[i][j]` will store the count of words starting with letter `i` that can be replaced with letter `j` without creating an existing word

**Step 2: Build the Frequency Table**
For each word `v` in `ideas`:
- Get the index `i` of its first letter: `i = ord(v[0]) - ord('a')`
- Convert the word to a list for easy character manipulation
- Try replacing the first letter with each of the 26 possible letters `j`:
  - Create a new word by setting `t[0] = chr(ord('a') + j)`
  - If this new word is NOT in the set `s`, increment `f[i][j] += 1`

This preprocessing tells us: for words starting with letter `i`, how many can have their first letter replaced with letter `j` without creating an existing word.

**Step 3: Count Valid Pairs**
Initialize `ans = 0` to store the final count. For each word `v` in `ideas`:
- Get the index `i` of its first letter
- For each possible replacement letter `j`:
  - Create a new word by replacing `v`'s first letter with letter `j`
  - If this new word is NOT in set `s`:
    - Add `f[j][i]` to the answer
    - This counts all words that start with letter `j` and can swap with our current word (which starts with letter `i`) to form valid pairs

**Why This Works:**
When we find that word `v` (starting with letter `i`) can have its first letter replaced with `j` without creating an existing word, we look up `f[j][i]`. This value tells us how many words starting with letter `j` can have their first letter replaced with `i` without creating an existing word. Each such word forms a valid pair with `v` because:
- Swapping `v`'s first letter `i` with `j` creates a non-existing word
- Swapping that other word's first letter `j` with `i` creates a non-existing word

The algorithm runs in O(n × 26) time where n is the number of ideas, as we iterate through all ideas twice, each time checking 26 possible letter replacements.

## Example Walkthrough

Let's walk through a small example with `ideas = ["coffee", "donuts", "time", "toffee"]`.

**Step 1: Initialize Data Structures**
- Create set `s = {"coffee", "donuts", "time", "toffee"}` for O(1) lookups
- Create 2D array `f[26][26]` initialized with zeros

**Step 2: Build the Frequency Table**

Process each word to fill `f[i][j]`:

For "coffee" (starts with 'c', index 2):
- Try replacing 'c' with each letter a-z
- "aoffee" ✓ not in s, so f[2][0]++ → f[2][0] = 1
- "boffee" ✓ not in s, so f[2][1]++ → f[2][1] = 1
- "coffee" ✗ exists in s, skip
- "doffee" ✓ not in s, so f[2][3]++ → f[2][3] = 1
- ... (other letters also don't exist, incrementing respective counters)

For "donuts" (starts with 'd', index 3):
- "aonuts" ✓ not in s, so f[3][0]++ → f[3][0] = 1
- "bonuts" ✓ not in s, so f[3][1]++ → f[3][1] = 1
- "conuts" ✓ not in s, so f[3][2]++ → f[3][2] = 1
- "donuts" ✗ exists in s, skip
- ... (continuing for all letters)

For "time" (starts with 't', index 19):
- Most replacements create non-existing words except "time" itself
- Updates f[19][j] for valid replacements

For "toffee" (starts with 't', index 19):
- "aoffee" ✓ not in s, so f[19][0]++ → f[19][0] = 2
- "boffee" ✓ not in s, so f[19][1]++ → f[19][1] = 2
- "coffee" ✗ exists in s, skip (f[19][2] stays at current value)
- "doffee" ✓ not in s, so f[19][3]++ → f[19][3] = 2
- ... (continuing for all letters)

**Step 3: Count Valid Pairs**

Now count valid pairs by checking each word:

For "coffee" (starts with 'c', index 2):
- Try replacing 'c' with each letter:
- 'c' → 'd': "doffee" ✓ not in s
  - Add f[3][2] = 1 (one word starting with 'd' can swap with 'c')
  - This counts the pair (coffee, donuts)
- 'c' → 't': "toffee" ✗ exists in s, skip
- Other valid replacements have f[j][2] = 0, so add nothing

For "donuts" (starts with 'd', index 3):
- 'd' → 'c': "conuts" ✓ not in s
  - Add f[2][3] = 1 (one word starting with 'c' can swap with 'd')
  - This counts the pair (donuts, coffee)
- Other valid replacements have f[j][3] = 0

For "time" (starts with 't', index 19):
- 't' → 'c': "cime" ✓ not in s
  - Add f[2][19] = 0 (no words starting with 'c' can swap with 't')
- 't' → 'd': "dime" ✓ not in s
  - Add f[3][19] = 0 (no words starting with 'd' can swap with 't')
- All valid replacements have f[j][19] = 0

For "toffee" (starts with 't', index 19):
- 't' → 'c': "coffee" ✗ exists in s, skip
- 't' → 'd': "doffee" ✓ not in s
  - Add f[3][19] = 0
- All valid replacements have f[j][19] = 0

**Final Answer: 2**
We found 2 valid company names:
1. "doffee conuts" (from swapping coffee and donuts)
2. "conuts doffee" (from swapping donuts and coffee)

## Python Solution

```python
class Solution:
    def distinctNames(self, ideas: List[str]) -> int:
        # Convert ideas list to set for O(1) lookup
        ideas_set = set(ideas)
      
        # Create a 26x26 matrix to store counts
        # count_matrix[i][j] represents how many valid swaps exist 
        # when changing from letter i to letter j
        count_matrix = [[0] * 26 for _ in range(26)]
      
        # First pass: For each idea, count valid letter substitutions
        for idea in ideas:
            # Get the index of the first letter (0-25 for a-z)
            original_letter_idx = ord(idea[0]) - ord('a')
          
            # Convert string to list for character manipulation
            char_list = list(idea)
          
            # Try replacing first letter with each letter a-z
            for new_letter_idx in range(26):
                char_list[0] = chr(ord('a') + new_letter_idx)
                new_word = ''.join(char_list)
              
                # If the new word doesn't exist in original ideas, it's valid
                if new_word not in ideas_set:
                    count_matrix[original_letter_idx][new_letter_idx] += 1
      
        # Second pass: Count total valid pairs
        total_pairs = 0
        for idea in ideas:
            # Get the index of the first letter
            original_letter_idx = ord(idea[0]) - ord('a')
          
            # Convert string to list for character manipulation
            char_list = list(idea)
          
            # For each valid substitution of current idea
            for new_letter_idx in range(26):
                char_list[0] = chr(ord('a') + new_letter_idx)
                new_word = ''.join(char_list)
              
                # If this substitution is valid (doesn't create existing idea)
                if new_word not in ideas_set:
                    # Add count of valid pairs where the other idea starts with
                    # new_letter_idx and can be swapped to original_letter_idx
                    total_pairs += count_matrix[new_letter_idx][original_letter_idx]
      
        return total_pairs
```

## Time and Space Complexity

The time complexity is `O(n × m × |Σ|)`, where:
- `n` is the number of strings in the `ideas` list
- `m` is the maximum length of the strings (used when creating new strings with `''.join(t)`)
- `|Σ|` is the size of the character set (26 letters in this problem)

The algorithm has two main loops that iterate through all `ideas`:
- First loop: For each idea (`n` iterations), it checks 26 possible first letter replacements (`|Σ|` iterations), and for each replacement, it creates a new string using `''.join(t)` which takes `O(m)` time. The set lookup `not in s` is `O(m)` in the worst case due to string hashing.
- Second loop: Similar structure with `n × |Σ| × m` operations.

Overall: `O(n × |Σ| × m) + O(n × |Σ| × m) = O(n × m × |Σ|)`

The space complexity is `O(|Σ|²)`, where:
- The 2D array `f` has dimensions `26 × 26`, which is `O(|Σ|²)`
- The set `s` stores `n` strings, each of length up to `m`, which is `O(n × m)`
- Temporary variables like list `t` use `O(m)` space

Since `|Σ|² = 26² = 676` is constant and typically smaller than `n × m` for large inputs, the dominant space factor depends on the input size. However, following the reference answer's notation, the auxiliary space complexity excluding the input storage is `O(|Σ|²)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Double Counting Valid Pairs**
A common mistake is thinking that if word A can swap with word B to form a valid pair, we might accidentally count this pair twice (once when processing A, once when processing B). However, the algorithm actually handles this correctly by design - it counts each ordered pair (A, B) exactly once, which is what we want since "doffee conuts" and "conuts doffee" are considered different company names.

### 2. **String Manipulation Inefficiency**
Creating a new list from string and joining it back for every letter check can be inefficient:
```python
# Inefficient approach
char_list = list(idea)
char_list[0] = new_letter
new_word = ''.join(char_list)
```

**Solution:** Use string slicing which is more efficient:
```python
# More efficient
new_word = chr(ord('a') + new_letter_idx) + idea[1:]
```

### 3. **Incorrect Understanding of Valid Pairs**
A critical pitfall is misunderstanding when a pair is valid. Both swapped words must NOT exist in the original set. A common error is checking only one direction:
```python
# WRONG: Only checking if one swap doesn't exist
if new_word not in ideas_set:
    total_pairs += 1  # This doesn't ensure the reverse swap is also valid
```

**Solution:** The algorithm correctly uses the pre-computed matrix to ensure both directions are valid. The count_matrix[j][i] value already represents words that start with 'j' and can validly swap to 'i'.

### 4. **Off-by-One Errors with Character Indexing**
Forgetting to handle the character-to-index conversion properly:
```python
# WRONG: Using character directly as index
count_matrix[idea[0]][new_letter]  # This will cause an error

# CORRECT: Convert to 0-based index
original_letter_idx = ord(idea[0]) - ord('a')
```

### 5. **Not Using a Set for Lookups**
Using the original list for membership checks instead of converting to a set:
```python
# WRONG: O(n) lookup time
if new_word not in ideas:  # If ideas is a list, this is slow

# CORRECT: O(1) lookup time
ideas_set = set(ideas)
if new_word not in ideas_set:
```

### Optimized Solution with Fixes:
```python
class Solution:
    def distinctNames(self, ideas: List[str]) -> int:
        ideas_set = set(ideas)
        count_matrix = [[0] * 26 for _ in range(26)]
      
        # Build frequency matrix
        for idea in ideas:
            first_letter_idx = ord(idea[0]) - ord('a')
            suffix = idea[1:]  # More efficient than list manipulation
          
            for new_letter_idx in range(26):
                new_word = chr(ord('a') + new_letter_idx) + suffix
                if new_word not in ideas_set:
                    count_matrix[first_letter_idx][new_letter_idx] += 1
      
        # Count valid pairs
        total_pairs = 0
        for idea in ideas:
            first_letter_idx = ord(idea[0]) - ord('a')
            suffix = idea[1:]
          
            for new_letter_idx in range(26):
                new_word = chr(ord('a') + new_letter_idx) + suffix
                if new_word not in ideas_set:
                    total_pairs += count_matrix[new_letter_idx][first_letter_idx]
      
        return total_pairs
```

