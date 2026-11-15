# 1255. Maximum Score Words Formed by Letters

> Source: https://algo.monster/liteproblems/1255

## Problem Statement

You are given three inputs:
- A list of `words`
- A list of single `letters` (which may contain duplicates)
- A `score` array where `score[i]` represents the score for the i-th letter of the alphabet ('a' has score `score[0]`, 'b' has score `score[1]`, and so on)

Your task is to form a valid set of words from the given word list that maximizes the total score. A valid set must follow these rules:

1. Each word from the list can be used at most once (you cannot use the same word twice)
2. The total letters used across all selected words cannot exceed the available letters in the `letters` list
3. Each letter in `letters` can only be used once

The score of a valid set is calculated by summing up the scores of all individual letters used in the selected words.

For example, if you have:
- `words = ["dog", "cat"]`
- `letters = ["d", "o", "g", "c", "a", "t"]`
- `score = [1, 2, 3, ...]` (where 'a'=1, 'b'=2, 'c'=3, 'd'=4, etc.)

You could select both "dog" and "cat" since you have enough letters. The score would be the sum of scores for 'd', 'o', 'g', 'c', 'a', 't'.

You need to return the maximum possible score that can be achieved by selecting any valid subset of words (including the empty set, which has a score of 0). You don't have to use all the available letters.

## Intuition

The key insight is recognizing that we need to find the best subset of words from our word list. Since each word can either be included or excluded from our selection, we're essentially dealing with a subset selection problem.

Why can we use brute force here? Look at the constraints - with a small number of words (typically 14-15), we have at most `2^15 = 32,768` possible combinations to check. This is computationally manageable.

The challenge breaks down into three parts:
1. **Generate all possible word combinations** - We need a way to systematically explore every possible subset of words
2. **Validate each combination** - Check if we have enough letters to form all the words in our current subset
3. **Calculate and track the maximum score** - For valid combinations, compute the total score and keep track of the best one

Binary enumeration is a clever technique for generating all subsets. We can represent each subset as a binary number where each bit position corresponds to a word. For example, with 3 words `["dog", "cat", "dad"]`:
- `001` (binary) means select only "dad"
- `101` (binary) means select "dog" and "dad"
- `111` (binary) means select all three words

For validation, we count how many of each letter we need for our selected words and compare against what's available. If we need 2 'a's but only have 1 'a' available, that combination is invalid.

The scoring is straightforward - for each letter used in valid combinations, we add up their individual scores based on the given score array.

This approach guarantees we find the optimal solution because we literally check every possible way to select words, making it impossible to miss the best combination.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro), [Backtracking](/problems/backtracking) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The implementation uses binary enumeration to explore all possible word combinations systematically. Let's walk through the solution step by step:

**Step 1: Count Available Letters**
```python
cnt = Counter(letters)
```
We first create a frequency map of available letters. This tells us exactly how many of each letter we can use. For example, if `letters = ["a", "a", "b", "c"]`, then `cnt = {'a': 2, 'b': 1, 'c': 1}`.

**Step 2: Binary Enumeration Setup**
```python
for i in range(1 << n):
```
We iterate through all numbers from `0` to `2^n - 1`, where `n` is the number of words. The expression `1 << n` is equivalent to `2^n`. Each number `i` represents a unique subset of words through its binary representation.

**Step 3: Extract Selected Words**
```python
cur = Counter(''.join([words[j] for j in range(n) if i >> j & 1]))
```
For each subset `i`, we extract the selected words. The condition `i >> j & 1` checks if the `j`-th bit of `i` is set:
- `i >> j` shifts `i` right by `j` positions
- `& 1` checks if the least significant bit is 1

If the bit is set, we include `words[j]`. We then join all selected words into a single string and count the frequency of each letter needed.

**Step 4: Validate the Combination**
```python
if all(v <= cnt[c] for c, v in cur.items()):
```
We check if we have enough letters for the current word combination. For each letter `c` needed with count `v`, we verify that `v <= cnt[c]` (we need at most as many as we have available).

**Step 5: Calculate Score**
```python
t = sum(v * score[ord(c) - ord('a')] for c, v in cur.items())
ans = max(ans, t)
```
If the combination is valid, we calculate its total score. For each letter `c` used `v` times:
- `ord(c) - ord('a')` converts the letter to its index (0 for 'a', 1 for 'b', etc.)
- `score[ord(c) - ord('a')]` gets the score for that letter
- We multiply by `v` to account for multiple uses of the same letter

We keep track of the maximum score seen across all valid combinations.

**Time Complexity:** `O(2^n × m)` where `n` is the number of words and `m` is the average length of words. We check `2^n` subsets, and for each subset, we need `O(m)` time to count letters and calculate scores.

**Space Complexity:** `O(1)` extra space (not counting the input), as we only use counters with at most 26 entries for English letters.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach:

**Given:**
- `words = ["ace", "bed", "face"]`
- `letters = ["a", "c", "e", "f", "b"]`
- `score = [1, 9, 1, 2, 4, 2, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]`
  - This means: 'a'=1, 'b'=9, 'c'=1, 'd'=2, 'e'=4, 'f'=2, etc.

**Step 1: Count Available Letters**
```
cnt = {'a': 1, 'c': 1, 'e': 1, 'f': 1, 'b': 1}
```

**Step 2: Binary Enumeration (n=3 words, so we check 2^3 = 8 combinations)**

Let's trace through each subset:

| Binary | Decimal | Selected Words | Letters Needed | Valid? | Score Calculation |
|--------|---------|---------------|----------------|--------|-------------------|
| 000 | 0 | none | {} | ✓ | 0 |
| 001 | 1 | ["face"] | {f:1, a:1, c:1, e:1} | ✓ | 2+1+1+4 = 8 |
| 010 | 2 | ["bed"] | {b:1, e:1, d:1} | ✗ | - (no 'd' available) |
| 011 | 3 | ["bed", "face"] | {b:1, e:2, d:1, f:1, a:1, c:1} | ✗ | - (need 2 'e's, have 1) |
| 100 | 4 | ["ace"] | {a:1, c:1, e:1} | ✓ | 1+1+4 = 6 |
| 101 | 5 | ["ace", "face"] | {a:2, c:2, e:2, f:1} | ✗ | - (need 2 of each a,c,e) |
| 110 | 6 | ["ace", "bed"] | {a:1, c:1, e:2, b:1, d:1} | ✗ | - (need 2 'e's, no 'd') |
| 111 | 7 | all three | {a:2, c:2, e:3, b:1, d:1, f:1} | ✗ | - (multiple violations) |

**Step 3: Detailed Look at Valid Combinations**

For subset 001 (selecting "face"):
- Letters needed: f(1), a(1), c(1), e(1)
- All needed letters are available in our letter pool
- Score = score['f'] + score['a'] + score['c'] + score['e']
- Score = 2 + 1 + 1 + 4 = 8

For subset 100 (selecting "ace"):
- Letters needed: a(1), c(1), e(1)
- All needed letters are available
- Score = score['a'] + score['c'] + score['e']
- Score = 1 + 1 + 4 = 6

**Step 4: Find Maximum**
The maximum score among all valid combinations is 8 (from selecting just "face").

**Answer: 8**

This example demonstrates how the algorithm systematically checks every possible word combination, validates whether we have sufficient letters, and tracks the maximum achievable score.

## Python Solution

```python
class Solution:
    def maxScoreWords(
        self, words: List[str], letters: List[str], score: List[int]
    ) -> int:
        # Count available letters
        available_letters = Counter(letters)
        num_words = len(words)
        max_score = 0
      
        # Iterate through all possible subsets of words using bit manipulation
        # Total subsets = 2^n where n is the number of words
        for subset_mask in range(1 << num_words):
            # Build the current subset of words
            # Check each bit position to determine if word at that index is included
            selected_words = []
            for word_idx in range(num_words):
                if (subset_mask >> word_idx) & 1:
                    selected_words.append(words[word_idx])
          
            # Count letters needed for the current subset of words
            required_letters = Counter(''.join(selected_words))
          
            # Check if we have enough letters to form this subset
            is_valid = True
            for letter, count in required_letters.items():
                if count > available_letters[letter]:
                    is_valid = False
                    break
          
            # If valid subset, calculate its score
            if is_valid:
                current_score = 0
                for letter, count in required_letters.items():
                    # Get score for each letter (a=0, b=1, ..., z=25)
                    letter_score = score[ord(letter) - ord('a')]
                    current_score += count * letter_score
              
                # Update maximum score
                max_score = max(max_score, current_score)
      
        return max_score
```

## Time and Space Complexity

**Time Complexity:** `O(2^n × n × M)`

The algorithm iterates through all possible subsets of words using bit manipulation (`1 << n` generates `2^n` iterations). For each subset:
- It constructs a string by joining selected words, which takes `O(n × M)` time in the worst case where all `n` words are selected and each word has maximum length `M`
- Creating the Counter from the joined string takes `O(n × M)` time
- Checking if all characters in `cur` are available in `cnt` takes `O(C)` time where `C = 26` (alphabet size)
- Computing the score takes `O(C)` time

Since `C` is constant (26), the dominant factor is `O(2^n × n × M)`.

**Space Complexity:** `O(C)`

The space complexity is determined by:
- `cnt`: Counter for available letters, which stores at most `C = 26` distinct characters
- `cur`: Counter for current subset's characters, which also stores at most `C = 26` distinct characters
- The joined string created temporarily has size `O(n × M)`, but this is not persistent storage
- Other variables (`ans`, `t`, loop variables) use `O(1)` space

The dominant persistent space usage is `O(C)` where `C = 26`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrectly Handling Letter Reuse Across Words

**The Problem:**
A common mistake is checking letter availability for each word independently rather than considering the cumulative letter usage across all selected words. For example:

```python
# INCORRECT approach
for subset_mask in range(1 << num_words):
    is_valid = True
    for word_idx in range(num_words):
        if (subset_mask >> word_idx) & 1:
            # Checking each word against original letters independently
            if not can_form_word(words[word_idx], available_letters):
                is_valid = False
                break
```

This fails because if you select words ["ace", "bad"], and you have letters ["a", "b", "c", "d", "e"], the letter "a" would be incorrectly counted as available for both words.

**The Solution:**
Count all required letters across ALL selected words first, then check against available letters:

```python
# CORRECT approach
selected_words = []
for word_idx in range(num_words):
    if (subset_mask >> word_idx) & 1:
        selected_words.append(words[word_idx])

# Count total letters needed for ALL selected words
required_letters = Counter(''.join(selected_words))

# Now check if we have enough letters in total
is_valid = all(count <= available_letters[letter] 
              for letter, count in required_letters.items())
```

### Pitfall 2: Forgetting to Handle Missing Letters in Available Pool

**The Problem:**
When checking if required letters are available, forgetting that some letters might not exist in the available pool at all:

```python
# INCORRECT - This will raise KeyError if letter not in available_letters
for letter, count in required_letters.items():
    if count > available_letters[letter]:  # KeyError if letter not present
        is_valid = False
```

**The Solution:**
Use the Counter's default behavior or use `.get()` with a default value:

```python
# CORRECT - Counter returns 0 for missing keys by default
for letter, count in required_letters.items():
    if count > available_letters[letter]:  # Returns 0 if letter not present
        is_valid = False

# Alternative using .get()
for letter, count in required_letters.items():
    if count > available_letters.get(letter, 0):
        is_valid = False
```

### Pitfall 3: Off-by-One Error in Letter Score Indexing

**The Problem:**
Incorrectly calculating the index for the score array:

```python
# INCORRECT - Common mistakes
letter_score = score[ord(letter)]  # Wrong: uses ASCII value directly
letter_score = score[ord(letter) - ord('a') + 1]  # Wrong: off by one
```

**The Solution:**
Remember that 'a' should map to index 0, 'b' to index 1, etc.:

```python
# CORRECT
letter_score = score[ord(letter) - ord('a')]
# 'a': ord('a') - ord('a') = 0
# 'b': ord('b') - ord('a') = 1
# 'z': ord('z') - ord('a') = 25
```

