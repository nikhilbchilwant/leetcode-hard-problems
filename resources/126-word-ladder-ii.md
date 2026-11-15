# 126. Word Ladder II

> Source: https://algo.monster/liteproblems/126

## Problem Statement

This problem asks you to find all the shortest transformation sequences between two words by changing one letter at a time, where each intermediate word must exist in a given word list.

Given:
- `beginWord`: The starting word
- `endWord`: The target word  
- `wordList`: A list of valid words that can be used as intermediate steps

You need to find all possible shortest paths from `beginWord` to `endWord` following these rules:

1. **Single Letter Change**: You can only change one letter at a time to transform from one word to the next. For example, "hit" → "hot" changes only the second letter.

2. **Valid Words Only**: Every intermediate word in the transformation must exist in `wordList`. The `beginWord` doesn't need to be in the list, but all other words (including `endWord`) must be present.

3. **Shortest Paths**: You must return only the transformation sequences with the minimum number of steps. If multiple paths exist with the same minimum length, return all of them.

The output should be a list of lists, where each inner list represents one complete transformation sequence from `beginWord` to `endWord`.

For example:
- If `beginWord = "hit"`, `endWord = "cog"`, and `wordList = ["hot","dot","dog","lot","log","cog"]`
- One possible shortest path could be: `["hit","hot","dot","dog","cog"]`
- Another could be: `["hit","hot","lot","log","cog"]`
- Both have the same length (5 words), so both should be returned

If no valid transformation sequence exists (like when `endWord` is not in `wordList`), return an empty list.

## Intuition

The key insight is that we need to find **all** shortest paths, not just one. This makes the problem more complex than a standard BFS shortest path problem.

Think of the word transformation as exploring a maze where each room (word) has doors to other rooms (words that differ by one letter). We want to find all the shortest routes from the entrance to the exit.

The challenge is that BFS typically stops when it first reaches the destination, giving us one shortest path. But we need all shortest paths of the same length. Here's how we can adapt BFS:

1. **Level-by-level exploration**: Instead of stopping immediately when we find `endWord`, we continue processing all nodes at the current level. This ensures we find all paths of the same minimum length.

2. **Track predecessors, not just visited**: In standard BFS, we mark nodes as visited to avoid revisiting them. However, since we need all paths, a word might be reached through multiple predecessors at the same level. For example, "dog" might be reached from both "dot" and "log" at the same distance from the start.

3. **Build paths backwards**: Rather than maintaining all partial paths during BFS (which would be memory-intensive), we can:
   - Use BFS to find the shortest distance and record which words can lead to each word at each level
   - Once we reach `endWord`, reconstruct all paths by backtracking through the predecessor relationships

4. **Optimize word generation**: For each word, we need to find all valid next words (those in `wordList` that differ by one letter). Instead of comparing with every word in the list, we can:
   - Try changing each character position to all 26 letters
   - Check if the resulting word exists in our word set
   - This is more efficient when the word list is large

The solution combines BFS for finding the shortest path length with a predecessor tracking mechanism that allows us to reconstruct all shortest paths through backtracking (DFS). The BFS phase builds a graph of predecessors, and the DFS phase extracts all valid paths from this graph.

## Solution Approach

The implementation uses a two-phase approach: BFS to find shortest paths and build predecessor relationships, followed by DFS to reconstruct all paths.

**Phase 1: BFS to Find Shortest Paths and Build Predecessor Graph**

1. **Initial Setup**:
   - Convert `wordList` to a set `words` for O(1) lookup
   - Check if `endWord` exists in the word list - if not, return empty result
   - Initialize `dist` dictionary to track the distance of each word from `beginWord`
   - Create `prev` as a defaultdict of sets to store predecessors for each word
   - Start BFS with `beginWord` in the queue

2. **Level-by-level BFS**:
   - Process all nodes at the current level before moving to the next (`step` tracks current distance)
   - For each word at current level:
     - Generate all possible one-letter transformations by:
       - Converting word to a list of characters
       - For each position, try all 26 letters: `chr(ord('a') + j)`
       - Form new word and check validity
 
3. **Predecessor Tracking**:
   - When we generate a transformed word `t` from word `p`:
     - If `dist.get(t, 0) == step`: Word `t` was already discovered at this level, add `p` as another predecessor
     - If `t` is in `words` (not yet visited): 
       - Add `p` as predecessor of `t`
       - Remove `t` from `words` to mark as visited
       - Add `t` to queue for next level
       - Record `dist[t] = step`
 
4. **Early Termination**:
   - Set `found = True` when we reach `endWord`
   - Continue processing the current level to find all paths of same length
   - Stop BFS after completing the level where `endWord` was found

**Phase 2: DFS to Reconstruct All Paths**

1. **Backtracking from End to Start**:
   - Start DFS from `endWord` with initial path `[endWord]`
   - Recursively explore all predecessors:
     - Base case: When we reach `beginWord`, we've found a complete path
     - Add the reversed path to results (since we built it backwards)
 
2. **Path Construction**:
   - For each word, iterate through all its predecessors
   - Add predecessor to current path
   - Recursively explore from that predecessor
   - Remove predecessor from path (backtrack) to explore other options

**Key Data Structures**:
- `words` (set): O(1) lookup for valid words
- `dist` (dict): Tracks shortest distance from `beginWord` to each word
- `prev` (defaultdict of sets): Maps each word to all its possible predecessors at the shortest distance
- `q` (deque): BFS queue for level-order traversal
- `ans` (list): Stores all shortest transformation sequences

The algorithm ensures we find all shortest paths by:
- Using BFS to guarantee shortest distance
- Tracking all predecessors at the same level
- Reconstructing all possible paths through backtracking

## Example Walkthrough

Let's trace through a small example to illustrate the solution approach.

**Input:**
- `beginWord = "red"`
- `endWord = "tax"`
- `wordList = ["ted", "tex", "rex", "tax"]`

**Phase 1: BFS to Find Shortest Paths and Build Predecessor Graph**

**Initial Setup:**
- Convert wordList to set: `words = {"ted", "tex", "rex", "tax"}`
- Check endWord "tax" exists ✓
- Initialize: `dist = {"red": 0}`, `prev = {}`, `q = ["red"]`

**Level 1 (step = 1):**
- Process "red":
  - Try all one-letter changes:
    - Position 0: "aed", "bed", ..., "ted" ✓ (in words), ..., "zed"
    - Position 1: "rad", "rbd", ..., "rzd" (none in words)
    - Position 2: "rea", "reb", ..., "rex" ✓ (in words), ..., "rez"
  - Found valid words: "ted", "rex"
  - Update:
    - `prev["ted"] = {"red"}`, `dist["ted"] = 1`
    - `prev["rex"] = {"red"}`, `dist["rex"] = 1`
    - Remove "ted", "rex" from words
    - Add both to queue
- After Level 1: `q = ["ted", "rex"]`, `words = {"tex", "tax"}`

**Level 2 (step = 2):**
- Process "ted":
  - Try all one-letter changes:
    - Position 0: finds nothing new ("red" was already visited)
    - Position 1: finds nothing new
    - Position 2: "tea", "teb", ..., "tex" ✓ (in words), ..., "tez"
  - Found: "tex"
  - Update: `prev["tex"] = {"ted"}`, `dist["tex"] = 2`

- Process "rex":
  - Try all one-letter changes:
    - Position 0: "tex" (dist[tex] = 2 = step, so add as predecessor)
    - Position 1: finds nothing new
    - Position 2: finds nothing new
  - Update: `prev["tex"] = {"ted", "rex"}` (rex added as another predecessor)

- After Level 2: `q = ["tex"]`, `words = {"tax"}`

**Level 3 (step = 3):**
- Process "tex":
  - Try all one-letter changes:
    - Position 0: finds nothing new
    - Position 1: "tax" ✓ (in words) - **Found endWord!**
    - Position 2: finds nothing new
  - Update: `prev["tax"] = {"tex"}`, `dist["tax"] = 3`
  - Set `found = True`
- Complete this level and stop BFS

**Final predecessor graph:**
```
prev = {
  "ted": {"red"},
  "rex": {"red"},
  "tex": {"ted", "rex"},
  "tax": {"tex"}
}
```

**Phase 2: DFS to Reconstruct All Paths**

Start DFS from "tax" with path ["tax"]:

1. Current word: "tax", path: ["tax"]
   - Predecessors: {"tex"}
   - Explore "tex"

2. Current word: "tex", path: ["tax", "tex"]
   - Predecessors: {"ted", "rex"}
   - First explore "ted"

3. Current word: "ted", path: ["tax", "tex", "ted"]
   - Predecessors: {"red"}
   - Explore "red"

4. Current word: "red", path: ["tax", "tex", "ted", "red"]
   - This is beginWord! Add reversed path: ["red", "ted", "tex", "tax"]
   - Backtrack to step 2

5. Back at "tex", now explore "rex"
   - Current word: "rex", path: ["tax", "tex", "rex"]
   - Predecessors: {"red"}
   - Explore "red"

6. Current word: "red", path: ["tax", "tex", "rex", "red"]
   - This is beginWord! Add reversed path: ["red", "rex", "tex", "tax"]

**Final Result:**
```
[
  ["red", "ted", "tex", "tax"],
  ["red", "rex", "tex", "tax"]
]
```

Both paths have length 4, which is the shortest possible transformation sequence.

## Python Solution

```python
from collections import defaultdict, deque
from typing import List

class Solution:
    def findLadders(
        self, beginWord: str, endWord: str, wordList: List[str]
    ) -> List[List[str]]:
        """
        Find all shortest transformation sequences from beginWord to endWord.
        Each transformation changes exactly one letter at a time.
      
        Args:
            beginWord: Starting word
            endWord: Target word
            wordList: List of valid intermediate words
          
        Returns:
            List of all shortest transformation sequences
        """
      
        def build_paths(current_path: List[str], current_word: str) -> None:
            """
            Recursively build all shortest paths from endWord back to beginWord.
          
            Args:
                current_path: Path being constructed (in reverse order)
                current_word: Current word in the path construction
            """
            # Base case: reached the beginning word
            if current_word == beginWord:
                # Reverse the path to get correct order and add to results
                result_paths.append(current_path[::-1])
                return
          
            # Explore all predecessor words that lead to current word
            for predecessor_word in predecessors[current_word]:
                current_path.append(predecessor_word)
                build_paths(current_path, predecessor_word)
                current_path.pop()  # Backtrack

        # Initialize result list
        result_paths = []
      
        # Convert wordList to set for O(1) lookup
        available_words = set(wordList)
      
        # Early termination if endWord is not in wordList
        if endWord not in available_words:
            return result_paths
      
        # Remove beginWord from available words to avoid revisiting
        available_words.discard(beginWord)
      
        # Track distance from beginWord to each discovered word
        word_distances = {beginWord: 0}
      
        # Track all predecessors for each word (for path reconstruction)
        predecessors = defaultdict(set)
      
        # BFS queue initialization
        bfs_queue = deque([beginWord])
        target_found = False
        current_distance = 0
      
        # BFS to find all shortest paths
        while bfs_queue and not target_found:
            current_distance += 1
          
            # Process all words at current distance level
            level_size = len(bfs_queue)
            for _ in range(level_size):
                current_word = bfs_queue.popleft()
                word_chars = list(current_word)
              
                # Try changing each character position
                for char_index in range(len(word_chars)):
                    original_char = word_chars[char_index]
                  
                    # Try all 26 lowercase letters
                    for letter_code in range(26):
                        word_chars[char_index] = chr(ord('a') + letter_code)
                        transformed_word = ''.join(word_chars)
                      
                        # If word was already discovered at same distance, add predecessor
                        if word_distances.get(transformed_word, 0) == current_distance:
                            predecessors[transformed_word].add(current_word)
                      
                        # Skip if word is not valid or already processed
                        if transformed_word not in available_words:
                            continue
                      
                        # Record predecessor and process new word
                        predecessors[transformed_word].add(current_word)
                        available_words.discard(transformed_word)
                        bfs_queue.append(transformed_word)
                        word_distances[transformed_word] = current_distance
                      
                        # Check if target is reached
                        if transformed_word == endWord:
                            target_found = True
                  
                    # Restore original character for next iteration
                    word_chars[char_index] = original_char
      
        # Build all shortest paths if target was found
        if target_found:
            initial_path = [endWord]
            build_paths(initial_path, endWord)
      
        return result_paths
```

## Time and Space Complexity

**Time Complexity:** `O(N * M * 26 + N * M * P)` where:
- `N` is the number of words in the wordList
- `M` is the length of each word
- `P` is the number of shortest paths from beginWord to endWord

The BFS portion takes `O(N * M * 26)`:
- We potentially visit all `N` words in the worst case
- For each word, we iterate through all `M` characters
- For each character position, we try all 26 lowercase letters
- String operations like creating the new word take `O(M)` time

The DFS reconstruction takes `O(P * L)` where `L` is the length of the shortest path:
- We traverse each of the `P` paths
- Each path has length `L` (the shortest distance)
- `P` can grow exponentially with the number of words, so this term can dominate the BFS cost on inputs with many equal-length paths

**Space Complexity:** `O(N * M + N^2)` where:
- `words` set stores up to `N` words, each of length `M`: `O(N * M)`
- `dist` dictionary stores up to `N` entries: `O(N)`
- `prev` dictionary can store up to `N` entries, and each entry can have up to `N` predecessors in the worst case: `O(N^2)`
- BFS queue can contain up to `N` words: `O(N * M)`
- DFS recursion stack depth is at most the length of the shortest path `L`, which is at most `N + 1`
- `ans` stores all shortest paths, which could be `O(P * L * M)` where `P` is the number of paths and `L` is the path length

The dominant space factor is typically `O(N * M)` for storing the words and `O(N^2)` for the predecessor relationships in dense transformation graphs.

## Common Pitfalls

### 1. **Not Handling Multiple Predecessors at the Same Level**

One of the most common mistakes is failing to recognize that a word can be reached from multiple predecessors at the same distance level. This happens when different words at level `n` can transform into the same word at level `n+1`.

**Incorrect Approach:**
```python
# Wrong: Only storing one predecessor per word
predecessors = {}  # dict instead of defaultdict(set)
if transformed_word in available_words:
    predecessors[transformed_word] = current_word  # Overwrites previous predecessor!
```

**Why it fails:** This would only find one path even when multiple shortest paths exist. For example, if both "hot" and "got" can transform to "pot" at the same level, only one predecessor would be recorded.

**Correct Solution:**
```python
# Use defaultdict(set) to store ALL predecessors at the same level
predecessors = defaultdict(set)
if word_distances.get(transformed_word, 0) == current_distance:
    predecessors[transformed_word].add(current_word)
```

### 2. **Revisiting Words Already Processed at Current Level**

Another critical mistake is not properly handling words that are discovered multiple times within the same BFS level. We need to add all valid predecessors but avoid adding the word to the queue multiple times.

**Incorrect Approach:**
```python
# Wrong: Adding word to queue every time it's discovered
for predecessor in current_level:
    for transformed_word in get_transformations(predecessor):
        if transformed_word in available_words:
            bfs_queue.append(transformed_word)  # Duplicate additions!
            predecessors[transformed_word].add(predecessor)
```

**Why it fails:** This leads to duplicate processing and incorrect distance calculations. A word might be added to the queue multiple times within the same level, causing it to be processed redundantly.

**Correct Solution:**
```python
# Check if word was already discovered at this level
if word_distances.get(transformed_word, 0) == current_distance:
    # Already discovered at this level - just add predecessor
    predecessors[transformed_word].add(current_word)
elif transformed_word in available_words:
    # First time discovering - add to queue and mark as visited
    predecessors[transformed_word].add(current_word)
    available_words.discard(transformed_word)  # Remove to prevent re-adding
    bfs_queue.append(transformed_word)
    word_distances[transformed_word] = current_distance
```

### 3. **Incorrect Level-by-Level BFS Processing**

A subtle but important mistake is not processing BFS strictly level by level, which can lead to finding paths that aren't actually the shortest.

**Incorrect Approach:**
```python
# Wrong: Not tracking level boundaries properly
while bfs_queue:
    current_word = bfs_queue.popleft()
    if current_word == endWord:
        break  # Might miss other paths at the same level!
    # ... generate transformations
```

**Why it fails:** If you break immediately upon finding the target, you miss other words at the same level that could also lead to the target, resulting in incomplete results.

**Correct Solution:**
```python
# Process entire level before moving to next
while bfs_queue and not target_found:
    current_distance += 1
    level_size = len(bfs_queue)  # Capture current level size
  
    for _ in range(level_size):  # Process exactly this many words
        current_word = bfs_queue.popleft()
        # ... process transformations
        if transformed_word == endWord:
            target_found = True  # Mark found but continue processing level
  
    # Only stop after completing the level where target was found
```

These pitfalls often result in either missing valid shortest paths or including paths that aren't actually the shortest. The key is maintaining proper level boundaries in BFS and correctly tracking all predecessors at each level.

