# 1178. Number of Valid Words for Each Puzzle

> Source: https://algo.monster/liteproblems/1178

## Problem Statement

You are given a list of `words` and a list of `puzzles`. For each puzzle, you need to count how many words from the word list are "valid" with respect to that puzzle.

A word is considered **valid** for a puzzle if it satisfies both of these conditions:
1. The word must contain the first letter of the puzzle
2. Every letter in the word must exist somewhere in the puzzle

For example, if the puzzle is `"abcdefg"`:
- `"faced"` is valid (contains 'a' which is the first letter, and all letters f,a,c,e,d are in the puzzle)
- `"cabbage"` is valid (contains 'a' and all its letters are in the puzzle)  
- `"baggage"` is valid (contains 'a' and all its letters are in the puzzle)
- `"beefed"` is invalid (doesn't contain 'a', the first letter of the puzzle)
- `"based"` is invalid (contains 's' which is not in the puzzle)

Your task is to return an array `answer` where `answer[i]` represents the number of valid words for `puzzles[i]`.

## Intuition

The key insight is recognizing that we only care about which unique letters are present in each word, not their frequency or order. This immediately suggests using a set-like representation for each word.

Since we're dealing with lowercase English letters (only 26 possibilities), we can use bit manipulation where each bit position represents whether a specific letter exists. For example, if bit 0 is set, it means 'a' is present; if bit 1 is set, 'b' is present, and so on. This gives us a compact integer representation for any combination of letters.

The naive approach would be to check every word against every puzzle, but this would be inefficient. Instead, we can preprocess all words into their bitmask representations and count how many words share the same bitmask using a hash table. This is beneficial because multiple words might have the same set of unique letters (like "abc", "cab", "bca" all have the same bitmask).

For each puzzle, we need to find all words whose letters are a subset of the puzzle's letters AND contain the puzzle's first letter. Here's where the crucial observation comes in: since puzzles have exactly 7 letters, there are at most `2^7 = 128` possible subsets for each puzzle. This is a manageable number to enumerate.

We can iterate through all subsets of a puzzle's bitmask that include the first letter. For each such subset, we check our hash table to see how many words have exactly that bitmask. The sum of all these counts gives us the answer for that puzzle.

The subset enumeration technique `j = (j - 1) & mask` is a clever bit manipulation trick that efficiently generates all subsets of a given bitmask in descending order, allowing us to check each valid combination exactly once.

**Learn more about [Trie](/problems/trie_intro) patterns.**

## Solution Approach

The solution uses state compression with bit manipulation, a hash table for counting, and subset enumeration. Let's walk through the implementation step by step:

**Step 1: Preprocessing Words with State Compression**

First, we convert each word into a bitmask and count occurrences:
```python
cnt = Counter()
for w in words:
    mask = 0
    for c in w:
        mask |= 1 << (ord(c) - ord("a"))
    cnt[mask] += 1
```

For each word, we create a bitmask where bit `i` is set if letter `i` appears in the word. The expression `1 << (ord(c) - ord("a"))` creates a bitmask with only the bit corresponding to character `c` set. We use the OR operation `|=` to combine all character bits. The Counter stores how many words map to each unique bitmask.

**Step 2: Processing Each Puzzle**

For each puzzle, we need to find all valid words:
```python
ans = []
for p in puzzles:
    mask = 0
    for c in p:
        mask |= 1 << (ord(c) - ord("a"))
```

Similar to words, we first create a bitmask for the puzzle representing all its letters.

**Step 3: Subset Enumeration with First Letter Constraint**

```python
x, i, j = 0, ord(p[0]) - ord("a"), mask
while j:
    if j >> i & 1:
        x += cnt[j]
    j = (j - 1) & mask
ans.append(x)
```

Here's the clever part:
- `x` accumulates the count of valid words for this puzzle
- `i` stores the bit position of the puzzle's first letter
- `j` iterates through all subsets of the puzzle's bitmask

The subset enumeration uses the technique `j = (j - 1) & mask`. This generates all subsets of `mask` in descending order. Here's how it works:
- Subtracting 1 from `j` flips all trailing 1s to 0s and the rightmost 0 to 1
- ANDing with `mask` ensures we only keep bits that were originally in the puzzle
- This efficiently generates each subset exactly once

The condition `if j >> i & 1` checks if the current subset `j` contains the puzzle's first letter (bit at position `i`). Only if this condition is true do we add the count of words with this exact bitmask to our answer.

**Time Complexity**: O(N × W + M × 2^7) where N is the number of words, W is the average word length, and M is the number of puzzles. The 2^7 factor comes from enumerating at most 128 subsets per puzzle.

**Space Complexity**: O(N) for storing the hash table of word bitmasks.

## Example Walkthrough

Let's walk through a small example with `words = ["ace", "aec", "face"]` and `puzzles = ["cafe"]`.

**Step 1: Preprocessing Words**

Convert each word to a bitmask:
- `"ace"`: 
  - 'a' → bit 0: `mask = 0000...001`
  - 'c' → bit 2: `mask = 0000...101`
  - 'e' → bit 4: `mask = 0001...0101` (binary: 10101, decimal: 21)

- `"aec"`: Same letters as "ace" → bitmask = 21

- `"face"`:
  - 'f' → bit 5: starts with `0010...0000`
  - 'a' → bit 0: becomes `0010...0001`
  - 'c' → bit 2: becomes `0010...0101`
  - 'e' → bit 4: becomes `0011...0101` (binary: 110101, decimal: 53)

Our hash table `cnt`:
- `cnt[21] = 2` (both "ace" and "aec" map to this)
- `cnt[53] = 1` ("face" maps to this)

**Step 2: Processing Puzzle "cafe"**

Convert puzzle to bitmask:
- 'c' → bit 2, 'a' → bit 0, 'f' → bit 5, 'e' → bit 4
- Puzzle bitmask = `0011...0101` (binary: 110101, decimal: 53)

**Step 3: Enumerate Subsets**

First letter is 'c' (bit position 2). We enumerate all subsets of 53 and check if they contain bit 2:

Starting with `j = 53` (binary: 110101):
- `j = 53` (110101): Contains bit 2? YES → Check `cnt[53] = 1` → Add 1 to count
- `j = 52` (110100): Contains bit 2? YES → Check `cnt[52] = 0` → Add 0
- `j = 37` (100101): Contains bit 2? YES → Check `cnt[37] = 0` → Add 0
- `j = 36` (100100): Contains bit 2? YES → Check `cnt[36] = 0` → Add 0
- `j = 21` (010101): Contains bit 2? YES → Check `cnt[21] = 2` → Add 2 to count
- `j = 20` (010100): Contains bit 2? YES → Check `cnt[20] = 0` → Add 0
- `j = 5` (000101): Contains bit 2? YES → Check `cnt[5] = 0` → Add 0
- `j = 4` (000100): Contains bit 2? YES → Check `cnt[4] = 0` → Add 0
- Continue until `j = 0`...

Total valid words for puzzle "cafe" = 1 + 2 = 3

All three words are valid because:
- "ace" and "aec" contain 'c' (first letter) and all their letters {a,c,e} are in {c,a,f,e}
- "face" contains 'c' (first letter) and all its letters {f,a,c,e} are in {c,a,f,e}

## Python Solution

```python
class Solution:
    def findNumOfValidWords(self, words: List[str], puzzles: List[str]) -> List[int]:
        # Count frequency of each word's letter bitmask
        # Each bit position represents a letter (a=0, b=1, ..., z=25)
        word_bitmask_count = Counter()
      
        for word in words:
            bitmask = 0
            # Convert word to bitmask representation
            for char in word:
                bit_position = ord(char) - ord('a')
                bitmask |= 1 << bit_position
            word_bitmask_count[bitmask] += 1
      
        result = []
      
        for puzzle in puzzles:
            # Create bitmask for puzzle letters
            puzzle_bitmask = 0
            for char in puzzle:
                bit_position = ord(char) - ord('a')
                puzzle_bitmask |= 1 << bit_position
          
            # Count valid words for this puzzle
            valid_word_count = 0
            first_letter_bit = ord(puzzle[0]) - ord('a')
          
            # Iterate through all subsets of puzzle_bitmask that contain the first letter
            # Using technique: iterate through subsets by decrementing and ANDing with original mask
            subset = puzzle_bitmask
            while subset:
                # Check if subset contains the first letter of puzzle
                if (subset >> first_letter_bit) & 1:
                    valid_word_count += word_bitmask_count[subset]
                # Move to next subset of puzzle_bitmask
                subset = (subset - 1) & puzzle_bitmask
          
            result.append(valid_word_count)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(m × |w| + n × 2^|p|)`

The time complexity consists of two main parts:

1. **Processing words:** `O(m × |w|)`
   - We iterate through `m` words
   - For each word, we iterate through at most `|w|` characters to create the bitmask
   - Each character operation (bitwise OR) takes `O(1)` time

2. **Processing puzzles:** `O(n × 2^|p|)`
   - We iterate through `n` puzzles
   - For each puzzle, we first create a bitmask in `O(|p|)` time
   - Then we enumerate all submasks using the technique `j = (j - 1) & mask`
   - This submask enumeration iterates through all submasks of `mask`, which can be at most `2^|p|` submasks (where `|p|` is the number of set bits in the mask, bounded by the puzzle length)
   - The constraint check `j >> i & 1` ensures we only count valid submasks containing the first letter

**Space Complexity:** `O(m)`

The space complexity is determined by:
- The `Counter` dictionary `cnt` which stores at most `m` unique bitmasks (one for each word in the worst case)
- The `ans` list which stores `n` results, but this is considered output space and typically not counted in auxiliary space
- Other variables use `O(1)` space

Where:
- `m` = length of the `words` array
- `n` = length of the `puzzles` array  
- `|w|` = maximum length of words in the `words` array
- `|p|` = length of puzzles (typically 7 in this problem)

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Missing the Zero Subset
The subset enumeration loop `while subset:` terminates when `subset` becomes 0, but 0 is actually a valid subset (the empty subset). This means we miss counting words that consist only of the puzzle's first letter.

**Example:** If puzzle is "abcdefg" and we have a word "aaa", its bitmask would be just the bit for 'a'. When enumerating subsets, we never check subset = 0b0000001 (only 'a' set) if we stop at 0.

**Solution:** Add a special check after the loop for the subset containing only the first letter:
```python
# After the while loop
subset = 1 << first_letter_bit  # Subset with only first letter
valid_word_count += word_bitmask_count[subset]
```

### 2. Not Filtering Words with Too Many Unique Letters
Words can have more than 7 unique letters, but puzzles always have exactly 7 letters. A word with 8+ unique letters can never be valid for any puzzle, yet we still store them in our hash table, wasting memory.

**Example:** The word "acknowledgment" has 12 unique letters and will never match any 7-letter puzzle.

**Solution:** Skip words with more than 7 unique letters during preprocessing:
```python
for word in words:
    bitmask = 0
    for char in word:
        bitmask |= 1 << (ord(char) - ord('a'))
  
    # Count number of set bits (unique letters)
    if bin(bitmask).count('1') <= 7:
        word_bitmask_count[bitmask] += 1
```

### 3. Incorrect Subset Enumeration Starting Point
Some implementations might accidentally start subset enumeration from `puzzle_bitmask - 1` instead of `puzzle_bitmask`, missing the case where all puzzle letters are used.

**Example:** If puzzle is "abc" with bitmask 0b111, starting from 0b110 would miss checking if any words have exactly the letters "abc".

**Solution:** Always initialize `subset = puzzle_bitmask` before the enumeration loop to ensure we check the full set of puzzle letters first.

### 4. Confusing Bit Position with Bit Value
A common mistake is using `1 << first_letter_bit` when we should use just `first_letter_bit` as the position, or vice versa.

**Incorrect:**
```python
if subset & (1 << first_letter_bit):  # Wrong - double shifting
```

**Correct:**
```python
if (subset >> first_letter_bit) & 1:  # Right - check bit at position
```

