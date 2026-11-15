# 30. Substring with Concatenation of All Words

> Source: https://algo.monster/liteproblems/30

## Problem Statement

You need to find all starting positions in a string `s` where you can find a substring that is formed by concatenating all words from a given array `words` in any order.

Given:
- A string `s`
- An array of strings `words` where all strings have the same length

A **concatenated string** is formed by taking all the words from `words` and joining them together in any order, with each word used exactly once.

For example, if `words = ["ab","cd","ef"]`:
- Valid concatenated strings include: `"abcdef"`, `"abefcd"`, `"cdabef"`, `"cdefab"`, `"efabcd"`, `"efcdab"`
- Invalid example: `"acdbef"` (not a valid concatenation of the words)

Your task is to find all positions in `s` where a substring starts that matches any valid concatenation of all words in `words`. Return these starting indices in any order.

The solution uses a [sliding window](/problems/sliding_window_maximum) approach with hash tables. It maintains a counter `cnt` for the frequency of words in the original `words` array and another counter `cnt1` for the current window. The algorithm iterates through possible starting positions (0 to k-1, where k is the length of each word) and for each starting position, slides a window of size `n * k` (where n is the number of words) through the string. 

The window moves by word-length increments, checking if the current window contains exactly the same words as the `words` array. When a valid concatenation is found (window size equals `n * k` and all word frequencies match), the starting position is added to the result.

## Intuition

The key insight is recognizing that since all words have the same length `k`, we can treat the problem as finding windows in the string that contain exactly the right words with the right frequencies.

Instead of checking every possible substring of length `n * k` starting from every position (which would be inefficient), we can optimize by observing that valid concatenations must align with word boundaries. Since words are of fixed length `k`, any valid substring must start at positions that, when divided by `k`, give us consistent word boundaries.

This leads us to the realization that we only need to check `k` different "tracks" or starting offsets (0, 1, 2, ..., k-1). For each offset `i`, we can slide through the string jumping by word-length `k` at a time, essentially reading the string word by word from that offset.

The [sliding window](/problems/sliding_window_maximum) technique becomes natural here because:
1. We're looking for a fixed-size window (containing exactly `n` words)
2. As we slide, we can incrementally update our word counts rather than recounting from scratch
3. When we encounter an invalid word or too many occurrences of a valid word, we can shrink the window from the left

The use of hash tables is intuitive for tracking word frequencies - we need to quickly check if a word exists in our target list and compare frequencies between our target (`cnt`) and current window (`cnt1`).

The optimization of clearing the window when encountering an invalid word (not in `words`) comes from realizing that any window containing such a word cannot be valid, so we might as well start fresh from the position after that invalid word.

**Learn more about [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

The solution uses a **[sliding window](/problems/sliding_window_maximum) technique with hash tables** to efficiently find all valid starting positions.

**Data Structures:**
- `cnt`: A Counter (hash table) storing the frequency of each word in `words`
- `cnt1`: A Counter tracking the frequency of words in the current [sliding window](/problems/sliding_window_maximum)
- `ans`: List to store valid starting indices

**Algorithm Implementation:**

1. **Initialize variables:**
   - `m = len(s)`: length of the string
   - `n = len(words)`: number of words
   - `k = len(words[0])`: length of each word

2. **Iterate through k different starting offsets** (`i` from 0 to k-1):
   For each offset, we create a separate [sliding window](/problems/sliding_window_maximum) track.

3. **For each offset i, maintain a [sliding window](/problems/sliding_window_maximum):**
   - `l = r = i`: Initialize left and right pointers at offset `i`
   - `cnt1 = Counter()`: Reset word frequency counter for this track

4. **Slide the window by word-length increments:**
   ```
   while r + k <= m:
       t = s[r : r + k]  # Extract word at right boundary
       r += k            # Move right pointer by word length
   ```

5. **Handle three cases for each extracted word `t`:**

   **Case 1: Invalid word** (`cnt[t] == 0`):
   - The word doesn't exist in `words`
   - Reset window: `l = r`, clear `cnt1`
   - Continue to next iteration

   **Case 2: Valid word:**
   - Add to window: `cnt1[t] += 1`
   - If frequency exceeds allowed (`cnt1[t] > cnt[t]`):
     - Shrink window from left until frequency is valid:
     ```
     while cnt1[t] > cnt[t]:
         rem = s[l : l + k]
         l += k
         cnt1[rem] -= 1
     ```

   **Case 3: Valid window found:**
   - When `r - l == n * k` (window contains exactly `n` words)
   - Add starting position `l` to answer list

6. **Return the collected starting indices** after checking all `k` offsets.

The algorithm efficiently processes the string in `O(m * k)` time, where instead of checking every position, we only check `k` different tracks, and for each track, we scan through the string once using the [sliding window](/problems/sliding_window_maximum) technique.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Input:**
- `s = "barfoothefoobarman"`
- `words = ["foo", "bar"]`

**Setup:**
- `m = 18` (length of s)
- `n = 2` (number of words)
- `k = 3` (length of each word)
- `cnt = {"foo": 1, "bar": 1}` (frequency map of words)

We need to check `k = 3` different starting offsets (0, 1, 2).

**Offset 0 (i = 0):**
- Initialize: `l = 0, r = 0, cnt1 = {}`
- Extract "bar" (r=0 to 3): Valid word, `cnt1 = {"bar": 1}`, move `r = 3`
- Extract "foo" (r=3 to 6): Valid word, `cnt1 = {"bar": 1, "foo": 1}`, move `r = 6`
- Window size: `r - l = 6 - 0 = 6 = n * k` ✓ **Found valid position at index 0**
- Extract "the" (r=6 to 9): Invalid word (not in words), reset `l = 9, r = 9, cnt1 = {}`
- Extract "foo" (r=9 to 12): Valid word, `cnt1 = {"foo": 1}`, move `r = 12`
- Extract "bar" (r=12 to 15): Valid word, `cnt1 = {"foo": 1, "bar": 1}`, move `r = 15`
- Window size: `r - l = 15 - 9 = 6 = n * k` ✓ **Found valid position at index 9**
- Extract "man" (r=15 to 18): Invalid word, reset (end of string)

**Offset 1 (i = 1):**
- Initialize: `l = 1, r = 1, cnt1 = {}`
- Extract "arf" (r=1 to 4): Invalid word, reset `l = 4, r = 4, cnt1 = {}`
- Extract "oot" (r=4 to 7): Invalid word, reset `l = 7, r = 7, cnt1 = {}`
- Extract "hef" (r=7 to 10): Invalid word, reset `l = 10, r = 10, cnt1 = {}`
- Extract "oob" (r=10 to 13): Invalid word, reset `l = 13, r = 13, cnt1 = {}`
- Extract "arm" (r=13 to 16): Invalid word, reset `l = 16, r = 16, cnt1 = {}`
- Extract "an" (r=16 to 18): Only 2 characters left, stop

**Offset 2 (i = 2):**
- Initialize: `l = 2, r = 2, cnt1 = {}`
- Extract "rfo" (r=2 to 5): Invalid word, reset `l = 5, r = 5, cnt1 = {}`
- Extract "oth" (r=5 to 8): Invalid word, reset `l = 8, r = 8, cnt1 = {}`
- Extract "efo" (r=8 to 11): Invalid word, reset `l = 11, r = 11, cnt1 = {}`
- Extract "oba" (r=11 to 14): Invalid word, reset `l = 14, r = 14, cnt1 = {}`
- Extract "rma" (r=14 to 17): Invalid word, reset `l = 17, r = 17, cnt1 = {}`
- Extract "n" (r=17 to 18): Only 1 character left, stop

**Result:** `[0, 9]`

The algorithm found two valid starting positions where concatenations of all words exist:
- Position 0: "barfoo" = "bar" + "foo"
- Position 9: "foobar" = "foo" + "bar"

## Python Solution

```python
from typing import List
from collections import Counter

class Solution:
    def findSubstring(self, s: str, words: List[str]) -> List[int]:
        # Count frequency of each word in the words list
        word_count = Counter(words)
      
        # Get lengths: string length, number of words, and length of each word
        string_length = len(s)
        num_words = len(words)
        word_length = len(words[0])  # All words have the same length
      
        result = []
      
        # Try all possible starting positions within a word length
        # This ensures we check all possible alignments
        for start_pos in range(word_length):
            left = right = start_pos
            current_word_count = Counter()
          
            # Slide the window through the string
            while right + word_length <= string_length:
                # Extract the next word from the string
                current_word = s[right:right + word_length]
                right += word_length
              
                # If the word is not in our target words, reset the window
                if word_count[current_word] == 0:
                    left = right
                    current_word_count.clear()
                    continue
              
                # Add the current word to our window
                current_word_count[current_word] += 1
              
                # If we have too many of the current word, shrink window from left
                while current_word_count[current_word] > word_count[current_word]:
                    removed_word = s[left:left + word_length]
                    left += word_length
                    current_word_count[removed_word] -= 1
              
                # Check if we have found a valid concatenation
                # Window size should equal total length of all words
                if right - left == num_words * word_length:
                    result.append(left)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(m × k)`

The algorithm uses a sliding window approach with `k` different starting positions (from `0` to `k-1`). For each starting position, it processes the string `s` by moving the window in steps of size `k`. In each iteration:
- The right pointer `r` moves through the string in increments of `k`, covering at most `m/k` positions
- For each position, we extract a substring of length `k` (takes `O(k)` time)
- The inner while loop for adjusting the window also processes each character at most once

Since we have `k` starting positions and each processes the string with `O(m)` operations involving strings of length `k`, the total time complexity is `O(k × m/k × k) = O(m × k)`.

**Space Complexity:** `O(n × k)`

The space is used for:
- `cnt`: A Counter storing all words from the `words` array, which contains `n` words each of length `k`, requiring `O(n × k)` space
- `cnt1`: Another Counter that at most stores `n` words (when a valid window is found), also requiring `O(n × k)` space in the worst case
- `ans`: The result list stores indices, which is `O(number of results)` and doesn't dominate the complexity
- Temporary string slices like `t` and `rem` use `O(k)` space

The dominant space complexity is `O(n × k)` from the Counter data structures storing the word frequencies.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Reset the Window Counter for Each Starting Offset**

One of the most common mistakes is not properly resetting the `current_word_count` counter when starting a new offset track. This can lead to incorrect word frequencies being carried over between different starting positions.

**Incorrect approach:**
```python
# WRONG: Counter declared outside the loop
current_word_count = Counter()
for start_pos in range(word_length):
    left = right = start_pos
    # Counter not reset - carries over frequencies from previous offset!
```

**Correct approach:**
```python
for start_pos in range(word_length):
    left = right = start_pos
    current_word_count = Counter()  # Reset for each new offset
```

### 2. **Not Handling the Case When Window Needs to Shrink from the Left**

Another critical pitfall is failing to properly maintain the sliding window when we encounter a valid word but have too many occurrences of it. Simply moving the entire window or resetting it would miss valid concatenations.

**Incorrect approach:**
```python
if current_word_count[current_word] > word_count[current_word]:
    # WRONG: Completely resetting loses valid partial matches
    left = right
    current_word_count.clear()
```

**Correct approach:**
```python
# Shrink from left until the frequency is valid
while current_word_count[current_word] > word_count[current_word]:
    removed_word = s[left:left + word_length]
    left += word_length
    current_word_count[removed_word] -= 1
```

### 3. **Incorrect Window Size Validation**

A subtle error is checking the window size incorrectly or at the wrong time. The window should be checked after ensuring all word frequencies are valid, not before.

**Incorrect approach:**
```python
# WRONG: Checking size before frequency validation
if right - left == num_words * word_length:
    result.append(left)
current_word_count[current_word] += 1
# Then handling excess frequencies...
```

**Correct approach:**
```python
# First add the word and handle frequencies
current_word_count[current_word] += 1

# Handle excess frequencies if needed
while current_word_count[current_word] > word_count[current_word]:
    # ... shrink window

# THEN check if we have a valid window size
if right - left == num_words * word_length:
    result.append(left)
```

### 4. **Off-by-One Errors in Boundary Checks**

Be careful with the loop boundary condition. The check should be `right + word_length <= string_length` to ensure we can extract a complete word.

**Incorrect approach:**
```python
# WRONG: Reads a partial word at the end of the string
while right < string_length:
    current_word = s[right:right + word_length]  # May be shorter than word_length
```

**Correct approach:**
```python
while right + word_length <= string_length:
    current_word = s[right:right + word_length]  # Always gets complete word
```

In Python the short slice happens to fail the dictionary lookup and only wastes an iteration, but in Java `s.substring(right, right + wordLength)` throws `StringIndexOutOfBoundsException`.

These pitfalls can cause the algorithm to miss valid concatenations, return incorrect indices, or even crash with index errors. Always ensure proper window management and boundary checks when implementing sliding window algorithms.

