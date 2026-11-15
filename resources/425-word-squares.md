# 425. Word Squares

> Source: https://algo.monster/liteproblems/425

## Problem Statement

This problem asks you to find all possible word squares that can be built from a given array of unique strings.

A **word square** is a special arrangement of words where:
- Words are arranged in a square grid (same number of rows and columns)
- The `k`-th row reads the same as the `k`-th column for all valid positions `k`

For example, with the words `["ball","area","lead","lady"]`:
```
b a l l
a r e a
l e a d
l a d y
```
- Row 0: "ball", Column 0: "ball" (reading downward) ✓
- Row 1: "area", Column 1: "area" ✓
- Row 2: "lead", Column 2: "lead" ✓
- Row 3: "lady", Column 3: "lady" ✓

Key constraints:
- All input words are unique
- The same word can be used multiple times in different word squares
- All words in the input have the same length
- You need to return all valid word squares that can be formed

The solution uses a **Trie data structure** to efficiently find words with specific prefixes. The algorithm works by:
1. Building a Trie that stores all words and tracks which words have each prefix
2. Using DFS (depth-first search) to build word squares row by row
3. For each partially built square, determining what prefix the next word must have by looking at the vertical columns formed so far
4. Using the Trie to quickly find all words matching that required prefix
5. Recursively trying each matching word until a complete square is formed

The `dfs` function builds squares incrementally - when adding the `idx`-th word, it must have a prefix that matches the `idx`-th character of all previously added words (to maintain the row-column symmetry property).

## Intuition

The key insight is recognizing that a word square has a special symmetry property: the `k`-th row must equal the `k`-th column. This constraint becomes our guiding principle.

When building a word square row by row, each new word we add must satisfy increasingly specific prefix requirements. Consider what happens as we build:

1. **First word**: Can be any word from our list
2. **Second word**: Must start with the 2nd character of the first word
3. **Third word**: Must start with the 3rd character of the first word AND the 3rd character of the second word
4. **Fourth word**: Must start with specific characters from all previous words

This pattern reveals that as we go deeper, the prefix constraint for the next word becomes more restrictive - it's determined by looking at the vertical columns we've built so far.

For example, if we've placed:
```
b a l l
a r e a
```
The third word must start with "le" (the 3rd characters of "ball" and "area").

This naturally suggests a backtracking approach where:
- We try placing words one by one
- For each position, we calculate what prefix the next word must have
- We find all words matching that prefix
- We recursively try each candidate

The challenge is efficiently finding words with specific prefixes. A naive approach would scan all words each time, but that's inefficient. This is where the **Trie** comes in - it's the perfect data structure for prefix matching. By preprocessing all words into a Trie, we can quickly retrieve all words with any given prefix in O(prefix_length) time.

The combination of backtracking (to explore all possibilities) and Trie (for efficient prefix matching) gives us an elegant solution that prunes invalid paths early while systematically exploring all valid word square configurations.

## Solution Approach

The solution implements a backtracking algorithm with a Trie data structure for efficient prefix searching.

### Trie Implementation

The `Trie` class stores all words and allows quick prefix lookups:
- `children`: An array of 26 elements (for 'a' to 'z'), where each element points to the next Trie node
- `v`: A list storing indices of all words that pass through this node
- `insert(w, i)`: Adds word `w` with index `i` to the Trie. As we traverse each character, we add the word's index to every node along the path
- `search(w)`: Returns indices of all words having prefix `w`. If the prefix doesn't exist, returns an empty list

### Main Algorithm

The `wordSquares` function orchestrates the solution:

1. **Initialize the Trie**: Insert all words with their indices
   ```python
   for i, w in enumerate(words):
       trie.insert(w, i)
   ```

2. **Try each word as the first row**: Since any word can start a word square
   ```python
   for w in words:
       dfs([w])
   ```

3. **DFS Backtracking Function**: The core logic that builds word squares
   ```python
   def dfs(t):
       if len(t) == len(words[0]):  # Complete square formed
           ans.append(t[:])
           return
   ```

4. **Calculate Required Prefix**: For the next word at position `idx`, extract the `idx`-th character from each previously placed word
   ```python
   idx = len(t)
   pref = [v[idx] for v in t]  # Get vertical column
   ```
   For example, if `t = ["ball", "area"]` and `idx = 2`, then `pref = ['l', 'e']`, so the next word must start with "le"

5. **Find Matching Words**: Use the Trie to get all words with the required prefix
   ```python
   indexes = trie.search(''.join(pref))
   ```

6. **Try Each Candidate**: Recursively attempt to place each matching word
   ```python
   for i in indexes:
       t.append(words[i])
       dfs(t)
       t.pop()  # Backtrack
   ```

### Why This Works

The algorithm exploits the word square property that row `k` equals column `k`. When we've placed `k` words, the `(k+1)`-th word's prefix is fully determined by the vertical reading of the first `k` words at position `k`. The Trie enables O(m) prefix lookup where m is the word length, making the constraint checking efficient.

The backtracking ensures we explore all valid possibilities while pruning invalid branches early when no words match the required prefix.

## Example Walkthrough

Let's walk through building word squares with `words = ["ball", "area", "lead", "lady"]`.

**Step 1: Build the Trie**
We insert all words into the Trie. Each node stores indices of words that pass through it:
- "ball" (index 0): b→a→l→l stores [0] at each node
- "area" (index 1): a→r→e→a stores [1] at each node
- "lead" (index 2): l→e→a→d stores [2] at each node
- "lady" (index 3): l→a→d→y stores [3] at each node

**Step 2: Start DFS with "ball" as first word**
- Current square: `["ball"]`
- Need 2nd word (idx=1) with prefix = 1st char of each word in square = "a"
- Trie search for "a" returns indices [1] (only "area" starts with "a")

**Step 3: Add "area" as second word**
- Current square: `["ball", "area"]`
- Need 3rd word (idx=2) with prefix = 2nd char of each word = "le"
- Trie search for "le" returns indices [2] (only "lead" starts with "le")

**Step 4: Add "lead" as third word**
- Current square: `["ball", "area", "lead"]`
- Need 4th word (idx=3) with prefix = 3rd char of each word = "lad"
- Trie search for "lad" returns indices [3] (only "lady" starts with "lad")

**Step 5: Add "lady" as fourth word**
- Current square: `["ball", "area", "lead", "lady"]`
- Length equals 4 (word length), so we found a complete word square!

**Verification of the word square property:**
```
b a l l    Row 0: "ball"
a r e a    Row 1: "area"
l e a d    Row 2: "lead"
l a d y    Row 3: "lady"

Column 0: "ball" ✓
Column 1: "area" ✓
Column 2: "lead" ✓
Column 3: "lady" ✓
```

The algorithm continues backtracking to try other starting words. For instance, starting with "area" would fail at step 2 because no word starts with "r" (the 2nd character of "area"). Starting with "lead" or "lady" would also fail early due to prefix constraints.

The key insight: at each step, the prefix requirement becomes more specific, naturally pruning invalid paths. The Trie makes finding words with required prefixes efficient, turning what could be an O(n) scan into an O(m) lookup where m is the word length.

## Python Solution

```python
class Trie:
    def __init__(self):
        # Initialize children array for 26 lowercase letters
        self.children = [None] * 26
        # Store indices of words that pass through this node
        self.word_indices = []

    def insert(self, word: str, index: int) -> None:
        """Insert a word into the trie with its index in the words list."""
        current_node = self

        for char in word:
            # Calculate index for the character (0-25)
            char_index = ord(char) - ord('a')

            # Create new node if path doesn't exist
            if current_node.children[char_index] is None:
                current_node.children[char_index] = Trie()

            # Move to child node
            current_node = current_node.children[char_index]
            # Store the word index at this node (for prefix matching)
            current_node.word_indices.append(index)

    def search(self, prefix: str) -> List[int]:
        """Search for all word indices that have the given prefix."""
        current_node = self

        for char in prefix:
            # Calculate index for the character (0-25)
            char_index = ord(char) - ord('a')

            # If path doesn't exist, no words with this prefix
            if current_node.children[char_index] is None:
                return []

            # Move to child node
            current_node = current_node.children[char_index]

        # Return all word indices that have this prefix
        return current_node.word_indices


class Solution:
    def wordSquares(self, words: List[str]) -> List[List[str]]:
        """
        Find all possible word squares from the given list of words.
        A word square is a sequence of words where kth row and column read the same string.
        """

        def backtrack(current_square: List[str]) -> None:
            """Recursively build word squares using backtracking."""
            # Base case: square is complete when we have n words (n = word length)
            if len(current_square) == word_length:
                # Add a copy of the current square to results
                result.append(current_square[:])
                return

            # Get the index for the next word to add
            next_word_index = len(current_square)

            # Build prefix for the next word by collecting characters at position 'next_word_index'
            # from all words currently in the square
            prefix_chars = [word[next_word_index] for word in current_square]
            prefix = ''.join(prefix_chars)

            # Find all words that start with this prefix
            candidate_indices = trie.search(prefix)

            # Try each candidate word
            for word_index in candidate_indices:
                current_square.append(words[word_index])
                backtrack(current_square)
                current_square.pop()  # Backtrack

        # Initialize trie and result list
        trie = Trie()
        result = []
        word_length = len(words[0])

        # Build trie with all words and their indices
        for index, word in enumerate(words):
            trie.insert(word, index)

        # Try starting with each word
        for word in words:
            backtrack([word])

        return result
```

## Time and Space Complexity

**Time Complexity:** `O(N * L + N * L^L)` where `N` is the number of words and `L` is the length of each word.

- Building the Trie: `O(N * L)` - We insert each of the `N` words into the Trie, and each insertion takes `O(L)` time to traverse through the word's characters.

- DFS Search: In the worst case, we explore all possible word squares.
  - At each level of recursion (from 0 to `L-1`), we need to find words with a specific prefix.
  - The prefix search in Trie takes `O(L)` time.
  - At each position in the word square, we could potentially have up to `N` choices (in practice, it's limited by valid prefixes).
  - The maximum depth of recursion is `L`.
  - In the worst case, we could have `O(N^L)` different paths, but since we're constrained by valid prefixes at each step, the branching factor is limited.
  - More precisely, the time complexity is `O(N * L^L)` because at each of the `L` positions, we might need to try multiple words, and for each attempt, we do `O(L)` work for prefix search.

**Space Complexity:** `O(N * L * L + L)`

- Trie Storage: `O(N * L * L)`
  - The Trie stores all `N` words, each of length `L`.
  - At each node in the Trie, we store a list `v` containing indices of words that pass through that node.
  - In the worst case, each prefix of length `k` could be shared by all `N` words, leading to `N` indices stored at depth `k`.
  - Total space for storing indices: `O(N * L)` across all nodes.
  - The Trie structure itself with 26 pointers per node: `O(26 * N * L)` = `O(N * L)` nodes in worst case.
  - Combined: `O(N * L * L)` considering the indices stored at each level.

- Recursion Stack: `O(L)` - The maximum depth of recursion is `L` (the length of words).

- Output Storage: `O(K * L^2)` where `K` is the number of valid word squares found (not counted in auxiliary space).

## Common Pitfalls

### 1. **Incorrect Prefix Construction for Empty Square**
One common mistake is not handling the initial case correctly when the square is empty. When starting to build a word square, developers might try to construct a prefix from an empty list, which would result in an empty string prefix.

**Pitfall Code:**
```python
def backtrack(current_square):
    # This would fail when current_square is empty
    prefix_chars = [word[next_word_index] for word in current_square]
    prefix = ''.join(prefix_chars)
```

**Solution:**
Handle the empty square case separately or ensure the logic works for all cases:
```python
def backtrack(current_square):
    if len(current_square) == word_length:
        result.append(current_square[:])
        return

    # When square is empty, any word can be the first word
    if not current_square:
        for word in words:
            backtrack([word])
    else:
        next_word_index = len(current_square)
        prefix_chars = [word[next_word_index] for word in current_square]
        prefix = ''.join(prefix_chars)
        # ... continue with prefix search
```

### 2. **Forgetting to Store Word Indices at Intermediate Nodes**
A critical mistake in the Trie implementation is only storing word indices at the final character node, rather than at every node along the path. This would make prefix searches fail.

**Pitfall Code:**
```python
def insert(self, word: str, index: int) -> None:
    current_node = self
    for i, char in enumerate(word):
        char_index = ord(char) - ord('a')
        if current_node.children[char_index] is None:
            current_node.children[char_index] = Trie()
        current_node = current_node.children[char_index]
        # Wrong: Only add index at the last character
        if i == len(word) - 1:
            current_node.word_indices.append(index)
```

**Solution:**
Add the word index at every node along the path:
```python
def insert(self, word: str, index: int) -> None:
    current_node = self
    for char in word:
        char_index = ord(char) - ord('a')
        if current_node.children[char_index] is None:
            current_node.children[char_index] = Trie()
        current_node = current_node.children[char_index]
        # Correct: Add index at every node for prefix matching
        current_node.word_indices.append(index)
```

### 3. **Modifying the Result Without Creating a Copy**
When adding a completed word square to the results, forgetting to create a copy leads to all results pointing to the same list object, which gets modified during backtracking.

**Pitfall Code:**
```python
if len(current_square) == word_length:
    result.append(current_square)  # Wrong: adds reference, not copy
    return
```

**Solution:**
Always create a copy when storing the result:
```python
if len(current_square) == word_length:
    result.append(current_square[:])  # Correct: creates a copy
    # or result.append(list(current_square))
    return
```

### 4. **Not Handling Edge Cases**
Failing to handle edge cases like empty input or single-character words can cause unexpected behavior.

**Pitfall Code:**
```python
def wordSquares(self, words: List[str]) -> List[List[str]]:
    # Directly accessing words[0] without checking
    word_length = len(words[0])
```

**Solution:**
Add proper edge case handling:
```python
def wordSquares(self, words: List[str]) -> List[List[str]]:
    if not words:
        return []

    word_length = len(words[0])
    if word_length == 0:
        return []

    # Continue with main logic...
```

### 5. **Inefficient Prefix Building**
Building the prefix character by character in a loop when you already know the exact positions needed is inefficient and error-prone.

**Pitfall Code:**
```python
prefix = ""
for i in range(len(current_square)):
    prefix += current_square[i][next_word_index]
```

**Solution:**
Use list comprehension for cleaner and more efficient code:
```python
prefix_chars = [word[next_word_index] for word in current_square]
prefix = ''.join(prefix_chars)
```

