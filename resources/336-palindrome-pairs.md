# 336. Palindrome Pairs

> Source: https://algo.monster/liteproblems/336

## Problem Statement

You are given an array of unique strings called `words`. Your task is to find all pairs of indices `(i, j)` where concatenating `words[i]` and `words[j]` forms a palindrome.

A valid palindrome pair must satisfy:
- Both indices `i` and `j` are within the array bounds (0 to length-1)
- The indices must be different (`i != j`)
- The concatenation `words[i] + words[j]` reads the same forwards and backwards

For example, if `words = ["abc", "cba"]`, then the pair `(0, 1)` would be valid because `"abc" + "cba" = "abccba"` is a palindrome.

The algorithm examines each word and splits it at every possible position. For each split:
- It checks if the reverse of the left part exists in the word list and if the right part is already a palindrome
- It checks if the reverse of the right part exists in the word list and if the left part is already a palindrome

The solution uses a hash map to store word-to-index mappings for O(1) lookups. For each word, it iterates through all possible split positions (0 to word length). At each split position `j`:
- `a` = first `j` characters, `b` = remaining characters
- `ra` = reverse of `a`, `rb` = reverse of `b`
- If `ra` exists in the word list and `b` is a palindrome, then `[current_index, index_of_ra]` forms a valid pair
- If `rb` exists in the word list and `a` is a palindrome, then `[index_of_rb, current_index]` forms a valid pair

The condition `j > 0` in the second check prevents duplicate pairs when dealing with empty string splits.

The running time is O(n × k²), where n is the number of words and k is the maximum word length: each word has k + 1 split points, and each split does O(k) work for slicing, reversing, hashing, and palindrome checks.

## Intuition

The key insight is that if two words concatenate to form a palindrome, there must be a special relationship between their characters. Let's think about what happens when `word1 + word2` forms a palindrome.

Consider the concatenated string as having a "pivot point" somewhere. For the result to be a palindrome, the characters before the pivot must mirror the characters after it. This pivot could be:
1. Inside `word1` 
2. Inside `word2`
3. Exactly at the boundary between the two words

Let's explore case 1 where the pivot is inside `word1`. If we split `word1` into two parts `[prefix][suffix]`, then for `[prefix][suffix][word2]` to be a palindrome:
- The `suffix` must be a palindrome by itself (it sits at the center and mirrors around the pivot)
- The `prefix` and `word2` must mirror each other around that center
- This means `word2` must equal the reverse of `prefix`

Similarly, if the pivot is in `word2`, we split it as `[prefix][suffix]` and for `[word1][prefix][suffix]` to be a palindrome:
- The `prefix` must be a palindrome by itself
- `word1` must equal the reverse of `suffix`

This naturally leads us to the approach: for each word, try all possible split positions. At each split:
- Check if one part is a palindrome
- Check if the reverse of the other part exists in our word list
- If both conditions are met, we've found a valid palindrome pair

The beauty of this approach is that it systematically covers all possible pivot positions. By checking both directions (word as first element and word as second element), we ensure we find all valid pairs.

Using a hash map to store word-to-index mappings allows us to quickly check if a reversed substring exists in our word list, making the algorithm efficient enough to meet the time complexity requirement.

## Solution Approach

The implementation uses a hash map and string manipulation to efficiently find all palindrome pairs:

**1. Build Hash Map for O(1) Lookups**
```python
d = {w: i for i, w in enumerate(words)}
```
Create a dictionary mapping each word to its index. This allows constant-time lookups when checking if a reversed substring exists in the word list.

**2. Iterate Through Each Word**
```python
for i, w in enumerate(words):
```
Process each word in the array along with its index.

**3. Try All Possible Split Positions**
```python
for j in range(len(w) + 1):
    a, b = w[:j], w[j:]
```
For each word, split it at every position from 0 to its length. This creates two parts:
- `a`: prefix (first `j` characters)
- `b`: suffix (remaining characters)

The `+1` in the range ensures we also consider the case where one part is empty.

**4. Calculate Reverses**
```python
ra, rb = a[::-1], b[::-1]
```
Compute the reverse of both parts for checking palindrome conditions and dictionary lookups.

**5. Check First Configuration: Current Word as First Element**
```python
if ra in d and d[ra] != i and b == rb:
    ans.append([i, d[ra]])
```
This checks if `words[i] + words[d[ra]]` forms a palindrome:
- `ra in d`: The reverse of prefix `a` exists in our word list
- `d[ra] != i`: Ensure we're not pairing a word with itself
- `b == rb`: The suffix `b` is a palindrome

If all conditions are met, `[prefix_a][suffix_b] + [reverse_of_a]` forms a palindrome.

**6. Check Second Configuration: Current Word as Second Element**
```python
if j and rb in d and d[rb] != i and a == ra:
    ans.append([d[rb], i])
```
This checks if `words[d[rb]] + words[i]` forms a palindrome:
- `j > 0`: Avoid duplicates when `j=0` (empty prefix case already handled in first check)
- `rb in d`: The reverse of suffix `b` exists in our word list
- `d[rb] != i`: Ensure different indices
- `a == ra`: The prefix `a` is a palindrome

If all conditions are met, `[reverse_of_b] + [prefix_a][suffix_b]` forms a palindrome.

**Time Complexity Analysis:**
- For each word of length `m`, we perform `m+1` splits
- Each split involves creating substrings and checking palindromes: O(m) operations
- Total: O(n × m × m) where n is the number of words and m is average word length

The algorithm elegantly handles all cases including empty strings and ensures no duplicate pairs are added to the result.

## Example Walkthrough

Let's walk through the algorithm with `words = ["bat", "tab", "cat"]`.

**Step 1: Build Hash Map**
```
d = {"bat": 0, "tab": 1, "cat": 2}
```

**Step 2: Process word "bat" (index 0)**

Split at position 0:
- `a = ""`, `b = "bat"`
- `ra = ""`, `rb = "tab"`
- Check 1: Is `ra` ("") in dictionary? No, skip.
- Check 2: Skip (j = 0)

Split at position 1:
- `a = "b"`, `b = "at"`
- `ra = "b"`, `rb = "ta"`
- Check 1: Is `ra` ("b") in dictionary? No, skip.
- Check 2: Is `rb` ("ta") in dictionary? No, skip.

Split at position 2:
- `a = "ba"`, `b = "t"`
- `ra = "ab"`, `rb = "t"`
- Check 1: Is `ra` ("ab") in dictionary? No, skip.
- Check 2: Is `rb` ("t") in dictionary? No, skip.

Split at position 3:
- `a = "bat"`, `b = ""`
- `ra = "tab"`, `rb = ""`
- Check 1: Is `ra` ("tab") in dictionary? Yes! Index 1.
  - Is `d["tab"] != 0`? Yes (1 ≠ 0).
  - Is `b == rb` ("" == "")? Yes, palindrome!
  - **Found pair: [0, 1]** → "bat" + "tab" = "battab" ✓
- Check 2: Is `rb` ("") in dictionary? No, skip.

**Step 3: Process word "tab" (index 1)**

Split at position 0:
- `a = ""`, `b = "tab"`
- `ra = ""`, `rb = "bat"`
- Check 1: Is `ra` ("") in dictionary? No, skip.
- Check 2: Skip (j = 0)

Split at position 1:
- `a = "t"`, `b = "ab"`
- `ra = "t"`, `rb = "ba"`
- Check 1: Is `ra` ("t") in dictionary? No, skip.
- Check 2: Is `rb` ("ba") in dictionary? No, skip.

Split at position 2:
- `a = "ta"`, `b = "b"`
- `ra = "at"`, `rb = "b"`
- Check 1: Is `ra` ("at") in dictionary? No, skip.
- Check 2: Is `rb` ("b") in dictionary? No, skip.

Split at position 3:
- `a = "tab"`, `b = ""`
- `ra = "bat"`, `rb = ""`
- Check 1: Is `ra` ("bat") in dictionary? Yes! Index 0.
  - Is `d["bat"] != 1`? Yes (0 ≠ 1).
  - Is `b == rb` ("" == "")? Yes, palindrome!
  - **Found pair: [1, 0]** → "tab" + "bat" = "tabbat" ✓
- Check 2: Is `rb` ("") in dictionary? No, skip.

**Step 4: Process word "cat" (index 2)**

After checking all splits for "cat", no valid pairs are found because:
- "tac" (reverse of "cat") is not in the dictionary
- No partial reverses match with palindromic remainders

**Final Result: `[[0, 1], [1, 0]]`**

Both pairs form palindromes:
- "bat" + "tab" = "battab" (palindrome ✓)
- "tab" + "bat" = "tabbat" (palindrome ✓)

## Python Solution

```python
class Solution:
    def palindromePairs(self, words: List[str]) -> List[List[int]]:
        # Create a dictionary mapping each word to its index for O(1) lookup
        word_to_index = {word: index for index, word in enumerate(words)}
        result = []
      
        # Check each word for potential palindrome pairs
        for current_index, current_word in enumerate(words):
            # Try splitting the current word at every possible position
            for split_pos in range(len(current_word) + 1):
                # Split current word into prefix and suffix
                prefix = current_word[:split_pos]
                suffix = current_word[split_pos:]
              
                # Get reversed versions of prefix and suffix
                reversed_prefix = prefix[::-1]
                reversed_suffix = suffix[::-1]
              
                # Case 1: If reversed_prefix exists in words and suffix is palindrome
                # Then current_word + words[reversed_prefix_index] forms a palindrome
                if reversed_prefix in word_to_index and word_to_index[reversed_prefix] != current_index and suffix == reversed_suffix:
                    result.append([current_index, word_to_index[reversed_prefix]])
              
                # Case 2: If reversed_suffix exists in words and prefix is palindrome
                # Then words[reversed_suffix_index] + current_word forms a palindrome
                # Note: split_pos > 0 prevents duplicate when both prefix and suffix are empty
                if split_pos > 0 and reversed_suffix in word_to_index and word_to_index[reversed_suffix] != current_index and prefix == reversed_prefix:
                    result.append([word_to_index[reversed_suffix], current_index])
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n * k^2)` where `n` is the number of words and `k` is the average length of each word.

- Building the dictionary takes `O(n * k)` time (iterating through `n` words, each hash operation takes `O(k)`)
- The outer loop iterates through `n` words
- For each word, the inner loop runs `k + 1` times (where `k` is the length of current word)
- Inside the inner loop:
  - Slicing operations `w[:j]` and `w[j:]` take `O(k)` time
  - Reversing strings `a[::-1]` and `b[::-1]` take `O(k)` time
  - Checking palindrome conditions `b == rb` and `a == ra` take `O(k)` time
  - Dictionary lookups take `O(k)` time (comparing string keys)
- Total: `O(n * k * k)` = `O(n * k^2)`

**Space Complexity:** `O(n * k)` 

- The dictionary `d` stores `n` words as keys, requiring `O(n * k)` space
- The `ans` list can store at most `O(n^2)` pairs, but each pair only contains two integers, so it's `O(n^2)` for the answer list
- Temporary variables `a`, `b`, `ra`, `rb` each take up to `O(k)` space
- Since typically `k << n` in this problem context, and the dictionary dominates the space usage, the overall space complexity is `O(n * k)` when considering string storage

## Common Pitfalls

### 1. **Duplicate Pairs from Empty String Splits**

The most critical pitfall is generating duplicate pairs when handling empty string splits. When `j=0`, the prefix is empty and when `j=len(word)`, the suffix is empty. Without proper handling, both cases can identify the same palindrome pair.

**Problem Example:**
```python
words = ["abc", "cba"]
# When processing "abc" at j=0:
#   prefix="" (palindrome), suffix="abc"
#   If reverse of suffix ("cba") exists → pair [1, 0]
# When processing "cba" at j=3:
#   prefix="cba", suffix="" (palindrome)
#   If reverse of prefix ("abc") exists → pair [1, 0] again!
```

**Solution:**
The condition `if split_pos > 0` in the second check prevents this duplication by ensuring empty prefix cases are only handled in the first check.

### 2. **Self-Pairing Prevention**

Another common mistake is forgetting to check `d[ra] != i` or `d[rb] != i`, which would allow a word to pair with itself.

**Problem Example:**
```python
words = ["aba"]  # "aba" is itself a palindrome
# Without the self-check, might incorrectly add [0, 0]
```

**Solution:**
Always include the condition `word_to_index[reversed_part] != current_index` to prevent self-pairing.

### 3. **Incorrect Palindrome Validation**

A subtle error occurs when checking if concatenations form palindromes. The algorithm must verify that the non-matching part (the part not being looked up) is itself a palindrome.

**Problem Example:**
```python
words = ["ab", "ba", "abc"]
# For "abc" split as "ab" + "c":
#   reverse("ab") = "ba" exists in words
#   But "c" is not a palindrome
#   "abc" + "ba" = "abcba" is NOT a palindrome
```

**Solution:**
Always verify `suffix == reversed_suffix` or `prefix == reversed_prefix` to ensure the remaining part is palindromic.

### 4. **Edge Case: Empty Strings in Input**

If the input contains empty strings, special care is needed as any palindrome concatenated with an empty string remains a palindrome.

**Problem Example:**
```python
words = ["", "aba", "xyz"]
# "" + "aba" = "aba" (palindrome)
# "aba" + "" = "aba" (palindrome)
```

**Solution:**
The algorithm naturally handles this case, but be aware that empty strings will match with all palindromic words in the list, potentially creating many valid pairs.

### 5. **Performance Degradation with Hash Collisions**

While using a hash map provides O(1) average lookup time, worst-case scenarios with many hash collisions could degrade performance.

**Solution:**
Use Python's built-in dictionary which handles hash collisions efficiently. For extremely large datasets, consider additional optimization strategies like prefix trees (tries) for substring matching.

