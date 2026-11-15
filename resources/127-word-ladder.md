# 127. Word Ladder

> Source: https://algo.monster/liteproblems/127

## Problem Statement

This problem asks you to find the shortest transformation sequence between two words by changing one letter at a time, where each intermediate word must exist in a given dictionary.

Given:
- `beginWord`: The starting word
- `endWord`: The target word we want to reach
- `wordList`: A list of valid words that can be used as intermediate steps

Rules for transformation:
1. You can only change **one letter** at a time to form the next word
2. Each intermediate word must be present in `wordList` (except `beginWord` which doesn't need to be in the list)
3. The final word in the sequence must be `endWord`

The task is to return the **number of words** in the shortest transformation sequence from `beginWord` to `endWord`. If no valid transformation sequence exists, return `0`.

For example, if `beginWord = "hit"`, `endWord = "cog"`, and `wordList = ["hot","dot","dog","lot","log","cog"]`, one possible shortest sequence would be:
`"hit" -> "hot" -> "dot" -> "dog" -> "cog"`

This sequence has 5 words total, so the answer would be 5.

The solution uses **Breadth-First Search (BFS)** to find the shortest path. It explores all possible one-letter transformations level by level, ensuring that when the target word is found, it's reached via the shortest possible path. The algorithm:
1. Starts from `beginWord` and explores all valid one-letter transformations
2. For each word, tries changing each character to all 26 possible letters
3. Checks if the new word exists in the word list
4. Continues until `endWord` is found or all possibilities are exhausted
5. Removes visited words from the set to avoid revisiting them

## Intuition

The key insight is recognizing that this is a **shortest path problem in an unweighted graph**. Each word represents a node, and two words are connected if they differ by exactly one letter. Since we want the minimum number of transformations, we need to find the shortest path from `beginWord` to `endWord`.

Why BFS? When all edges have equal weight (each transformation counts as 1 step), BFS naturally finds the shortest path. It explores all words reachable in 1 step, then all words reachable in 2 steps, and so on. This level-by-level exploration guarantees that the first time we reach `endWord`, we've found the shortest path.

The clever part of the implementation is **how we find neighbors**. Instead of comparing every word with every other word to build the graph (which would be expensive), we generate all possible one-letter transformations on the fly. For each position in the current word, we try all 26 letters and check if the resulting word exists in our word list.

To avoid revisiting words and getting stuck in cycles, we **remove words from the set once visited**. This serves as our "visited" marker and prevents infinite loops. Think of it like marking rooms as explored in a maze - once you've been to a room at distance `d`, there's no point visiting it again at distance `d+1` or greater.

The algorithm maintains a counter for the number of words in the path. We start with 1 (for `beginWord`) and increment it each time we move to the next level of BFS. When we find `endWord`, we return this counter. If the queue becomes empty without finding `endWord`, it means no valid transformation sequence exists, so we return 0.

## Solution Approach

The solution implements a standard BFS approach with some optimizations for finding neighbors efficiently.

**Data Structures Used:**
- `words`: A set containing all valid words from `wordList` for O(1) lookup
- `q`: A deque (double-ended queue) for BFS traversal
- `ans`: Counter to track the number of words in the transformation sequence

**Algorithm Steps:**

1. **Initialize the BFS:**
   - Convert `wordList` to a set for fast membership checking
   - Add `beginWord` to the queue
   - Set `ans = 1` (counting `beginWord`)

2. **Level-by-level BFS traversal:**
   - For each level, increment `ans` at the start (representing the next step)
   - Process all words at the current level using `for _ in range(len(q))`

3. **Generate neighbors for each word:**
   - Pop a word from the queue and convert it to a list of characters
   - For each position `i` in the word:
     - Save the original character
     - Try all 26 possible letters (`'a'` to `'z'`)
     - Form the new word by joining the modified character list
     - Check if this new word exists in our word set

4. **Process valid neighbors:**
   - If the new word equals `endWord`, we've found the shortest path - return `ans`
   - Otherwise, add the new word to the queue for the next level
   - Remove the word from the set to mark it as visited

5. **Handle no solution:**
   - If the queue becomes empty without finding `endWord`, return 0

**Key Optimization:**
A common further optimization is **Bidirectional BFS**. Instead of searching only from start to end, we search from both directions simultaneously:
- Maintain two queues: `q1` (start → end) and `q2` (end → start)
- Always expand the smaller queue to reduce search space
- When a word is found in both directions, we've found the shortest path
- The total distance is `step1 + step2 + 1`

This bidirectional approach significantly reduces the search space, especially when the shortest path is long, as we explore from both ends rather than expanding exponentially from just one direction.

## Example Walkthrough

Let's walk through a small example to illustrate the BFS solution approach.

**Input:**
- `beginWord = "hit"`
- `endWord = "cog"`
- `wordList = ["hot", "dot", "dog", "cog"]`

**Step-by-step execution:**

**Initialization:**
- Convert wordList to set: `words = {"hot", "dot", "dog", "cog"}`
- Queue: `q = ["hit"]`
- Answer counter: `ans = 1` (counting "hit")

**Level 1 (ans = 2):**
- Process "hit" (only word in queue)
- Generate all one-letter transformations:
  - Position 0: "ait", "bit", ..., "zit" → only "hot" is in words set
  - Position 1: "hat", "hbt", ..., "hzt" → none in words set  
  - Position 2: "hia", "hib", ..., "hiz" → none in words set
- Found neighbor: "hot"
- Add "hot" to queue, remove from set
- Queue: `q = ["hot"]`, words: `{"dot", "dog", "cog"}`

**Level 2 (ans = 3):**
- Process "hot"
- Generate transformations:
  - Position 0: "aot", "bot", ..., "dot", ..., "zot" → "dot" is in words set
  - Position 1: "hat", "hbt", ..., "hzt" → none in words set
  - Position 2: "hoa", "hob", ..., "hoz" → none in words set
- Found neighbor: "dot"
- Queue: `q = ["dot"]`, words: `{"dog", "cog"}`

**Level 3 (ans = 4):**
- Process "dot"
- Generate transformations:
  - Position 0: "aot", "bot", ..., "zot" → none in remaining words
  - Position 1: "dat", "dbt", ..., "dzt" → none in remaining words
  - Position 2: "doa", "dob", ..., "dog", ..., "doz" → "dog" is in words set
- Found neighbor: "dog"
- Queue: `q = ["dog"]`, words: `{"cog"}`

**Level 4 (ans = 5):**
- Process "dog"
- Generate transformations:
  - Position 0: "aog", "bog", "cog", ..., "zog" → "cog" is in words set!
- Found "cog" which equals endWord
- **Return ans = 5**

The transformation sequence is: "hit" → "hot" → "dot" → "dog" → "cog" (5 words total)

**Key observations:**
1. BFS explores level by level, ensuring the first path found is the shortest
2. Each word is visited only once (removed from set after visiting)
3. We generate neighbors dynamically by trying all 26 letters at each position
4. The answer increments at each level, counting the number of words in the path

## Python Solution

```python
from collections import deque
from typing import List

class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        # Convert word list to set for O(1) lookup
        available_words = set(wordList)
      
        # Early termination if endWord is not in wordList
        if endWord not in available_words:
            return 0
      
        # Initialize BFS queue with the starting word
        queue = deque([beginWord])
      
        # Track the transformation sequence length (starting at 1 for beginWord)
        transformation_length = 1
      
        # BFS to find shortest transformation sequence
        while queue:
            transformation_length += 1
          
            # Process all words at current level
            current_level_size = len(queue)
            for _ in range(current_level_size):
                current_word = queue.popleft()
              
                # Convert string to list for character manipulation
                word_chars = list(current_word)
              
                # Try changing each character position
                for position in range(len(word_chars)):
                    # Store original character to restore later
                    original_char = word_chars[position]
                  
                    # Try all 26 lowercase letters
                    for letter_index in range(26):
                        word_chars[position] = chr(ord('a') + letter_index)
                      
                        # Form the new word
                        new_word = ''.join(word_chars)
                      
                        # Skip if the new word is not in the available words
                        if new_word not in available_words:
                            continue
                      
                        # Check if we've reached the target word
                        if new_word == endWord:
                            return transformation_length
                      
                        # Add valid transformation to queue for next level
                        queue.append(new_word)
                      
                        # Remove word from available set to avoid revisiting
                        available_words.remove(new_word)
                  
                    # Restore the original character for next position
                    word_chars[position] = original_char
      
        # No transformation sequence found
        return 0
```

## Time and Space Complexity

**Time Complexity:** `O(M² × N)`, where `M` is the length of each word and `N` is the total number of words in the word list.

- The algorithm uses BFS to traverse through possible word transformations
- In the worst case, we visit every word in the wordList: `O(N)`
- For each word visited, we iterate through all character positions: `O(M)`
- For each position, we try all 26 lowercase letters: `O(26)` = `O(1)`
- For each generated word, we perform string operations:
  - Converting list to string with `''.join(s)`: `O(M)`
  - Checking membership in the set `t not in words`: `O(M)` for string hashing
  - Comparing strings `t == endWord`: `O(M)`
- Total: `O(N × M × 26 × M)` = `O(M² × N)`

**Space Complexity:** `O(M × N)`

- The `words` set stores all words from wordList: `O(M × N)` where each word has length `M`
- The BFS queue can contain at most `N` words in the worst case: `O(M × N)`
- The temporary list `s` for character manipulation: `O(M)`
- Overall space complexity: `O(M × N)`

## Common Pitfalls

### 1. **Forgetting to Check if `endWord` Exists in `wordList`**
One of the most common mistakes is not verifying whether `endWord` is present in the word list before starting the BFS. If `endWord` isn't in the list, it's impossible to reach it, yet the algorithm might waste time exploring all possibilities.

**Problem Code:**
```python
# Missing this crucial check
available_words = set(wordList)
queue = deque([beginWord])
# Directly starts BFS without checking endWord
```

**Solution:**
```python
available_words = set(wordList)
# Early termination check
if endWord not in available_words:
    return 0
queue = deque([beginWord])
```

### 2. **Not Processing Words Level by Level**
A critical mistake is processing words continuously without maintaining level boundaries. This breaks the BFS property of finding the shortest path and makes it impossible to track the correct transformation length.

**Problem Code:**
```python
while queue:
    current_word = queue.popleft()
    # Incorrectly increments for each word, not each level
    transformation_length += 1
    # ... generate neighbors
```

**Solution:**
```python
while queue:
    transformation_length += 1
    # Process all words at the current level together
    current_level_size = len(queue)
    for _ in range(current_level_size):
        current_word = queue.popleft()
        # ... generate neighbors
```

### 3. **Trying to Modify a String in Place**
Python strings are immutable, so assigning to one character raises an error.

**Problem Code:**
```python
for position in range(len(current_word)):
    for letter in 'abcdefghijklmnopqrstuvwxyz':
        current_word[position] = letter  # TypeError: 'str' object does not support item assignment
```

**Solution:** Build each candidate as a new string. Slicing works (`current_word[:position] + letter + current_word[position+1:]`), and so does editing a character list and joining it. Both create an `O(M)` string per candidate, so neither is asymptotically faster; the list version just avoids re-slicing:
```python
# Convert to list for character manipulation
word_chars = list(current_word)
for position in range(len(word_chars)):
    original_char = word_chars[position]
    for letter_index in range(26):
        word_chars[position] = chr(ord('a') + letter_index)
        new_word = ''.join(word_chars)
    word_chars[position] = original_char  # Restore for next position
```

### 4. **Not Removing Visited Words from the Set**
Failing to mark words as visited leads to infinite loops or redundant processing. Words might be added to the queue multiple times, causing incorrect distance calculations and performance issues.

**Problem Code:**
```python
if new_word in available_words:
    if new_word == endWord:
        return transformation_length
    queue.append(new_word)
    # Forgot to remove the word from available_words
```

**Solution:**
```python
if new_word in available_words:
    if new_word == endWord:
        return transformation_length
    queue.append(new_word)
    # Mark as visited by removing from set
    available_words.remove(new_word)
```

### 5. **Including `beginWord` in the Available Words Set**
If `beginWord` happens to be in `wordList`, it stays in the set and gets added to the queue again at level 2. The answer is still correct, because BFS never shortens a path by returning to the start, but that whole subtree is explored a second time.

**Problem Code:**
```python
available_words = set(wordList)
# If beginWord is in wordList, it might be revisited
queue = deque([beginWord])
```

**Solution:**
```python
available_words = set(wordList)
# Remove beginWord if it exists to prevent revisiting
available_words.discard(beginWord)  # discard() won't raise error if not present
queue = deque([beginWord])
```

### 6. **Incorrect Initial Counter Value**
Starting with `transformation_length = 0` or incrementing at the wrong time leads to off-by-one errors in the final count.

**Problem Code:**
```python
transformation_length = 0  # Wrong: should start at 1 for beginWord
while queue:
    # Or incrementing after processing instead of before
    for _ in range(len(queue)):
        # process words
    transformation_length += 1  # Wrong timing
```

**Solution:**
```python
# Start at 1 to count beginWord
transformation_length = 1
while queue:
    # Increment at the start of each new level
    transformation_length += 1
    current_level_size = len(queue)
    for _ in range(current_level_size):
        # process words
```

