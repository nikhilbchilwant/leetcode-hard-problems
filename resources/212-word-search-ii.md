# 212. Word Search II

> Source: https://algo.monster/liteproblems/212

## Problem Statement

You are given a 2D board of characters (an `m x n` grid) and a list of words. Your task is to find all words from the list that can be formed on the board.

To form a word on the board:
- Start from any cell on the board
- Move to adjacent cells (horizontally or vertically neighboring cells only - no diagonal moves)
- Build the word letter by letter as you move through the cells
- Each cell can only be used once per word (you cannot revisit a cell while forming a single word)

The goal is to return a list containing all words from the input list that can be successfully formed on the board following these rules.

For example, if you have a board like:
```
[['o','a','a','n'],
 ['e','t','a','e'],
 ['i','h','k','r'],
 ['i','f','l','v']]
```

And words list: `["oath","pea","eat","rain"]`

You could form "oath" by starting at `board[0][0]`, moving right to `board[0][1]`, then down to `board[1][1]` and `board[2][1]`. You could form "eat" by starting at `board[1][3]` and moving left twice. The answer is `["oath", "eat"]`. The word "rain" cannot be formed because although all letters exist, they're not connected in a valid path.

The solution uses a Trie data structure to efficiently store and search for all words simultaneously as we explore the board, combined with depth-first search (DFS) to explore all possible paths from each starting position. The board cells are temporarily marked with '#' during exploration to prevent revisiting the same cell within a single word search.

## Intuition

When we need to find multiple words on a board, the naive approach would be to search for each word independently - for every word, try every cell as a starting point and explore all paths. However, this becomes inefficient when we have many words to search for, as we'd be traversing the same paths multiple times.

The key insight is that many words might share common prefixes. For example, if we're searching for "oath" and "oat", they share the prefix "oa". Instead of searching for these separately, we can search for all words simultaneously in a single traversal.

This leads us to the Trie (prefix tree) data structure. A Trie allows us to:
1. Store all words in a tree structure where common prefixes are shared
2. Check if our current path matches any word prefix in O(1) time per character
3. Know immediately if we should continue exploring a path (if it matches a prefix) or abandon it (if no words have this prefix)

The backtracking aspect comes from the constraint that each cell can only be used once per word. As we explore paths:
- We mark the current cell as visited (using '#' in this solution)
- Recursively explore all four adjacent cells
- After exploring, we restore the original character (backtrack) so the cell can be used in other word searches

By combining Trie with DFS backtracking, we achieve an elegant solution where:
- We start DFS from every cell on the board
- At each step, we check if the current path exists in our Trie
- If we reach a complete word in the Trie, we add it to our results
- We prune paths early if they don't match any prefix in our Trie
- We ensure each cell is used only once per path through backtracking

This approach is much more efficient than searching for each word independently, especially when words share common prefixes or when we have many words to search for.

## Solution Approach

The solution implements a Trie-based approach with DFS backtracking. Let's walk through the implementation step by step:

**1. Building the Trie Data Structure**

The `Trie` class stores our word dictionary:
- `children`: An array of 26 elements (for 'a' to 'z'), where each element can be another Trie node or `None`
- `ref`: Stores the index of the word in the original word list (-1 if this node doesn't represent a complete word)

The `insert` method adds a word to the Trie:
```python
def insert(self, w: str, ref: int):
    node = self
    for c in w:
        idx = ord(c) - ord('a')  # Convert character to index (0-25)
        if node.children[idx] is None:
            node.children[idx] = Trie()
        node = node.children[idx]
    node.ref = ref  # Mark the end of word with its index
```

**2. DFS with Backtracking**

The `dfs` function explores paths on the board:
```python
def dfs(node: Trie, i: int, j: int):
    idx = ord(board[i][j]) - ord('a')
    if node.children[idx] is None:
        return  # No words with this prefix, prune the search

    node = node.children[idx]
    if node.ref >= 0:  # Found a complete word
        ans.append(words[node.ref])
        node.ref = -1  # Mark as found to avoid duplicates
```

**3. Backtracking Mechanism**

To ensure each cell is used only once per word:
```python
c = board[i][j]
board[i][j] = '#'  # Mark cell as visited
for a, b in pairwise((-1, 0, 1, 0, -1)):  # Explore 4 directions
    x, y = i + a, j + b
    if 0 <= x < m and 0 <= y < n and board[x][y] != '#':
        dfs(node, x, y)
board[i][j] = c  # Restore original character (backtrack)
```

The `pairwise` function generates direction pairs: `(-1,0)`, `(0,1)`, `(1,0)`, `(0,-1)` for up, right, down, left movements.

**4. Main Algorithm Flow**

```python
# Build Trie from all words
tree = Trie()
for i, w in enumerate(words):
    tree.insert(w, i)

# Try starting from every cell
m, n = len(board), len(board[0])
ans = []
for i in range(m):
    for j in range(n):
        dfs(tree, i, j)
return ans
```

**Key Optimizations:**
- **Early Pruning**: If a path doesn't match any prefix in the Trie, we stop exploring
- **Duplicate Prevention**: Once a word is found, we set its `ref` to -1 to avoid adding it multiple times
- **In-place Marking**: Instead of using a separate visited array, we temporarily modify the board itself

**Time Complexity**: O(m × n × 4^L) where m×n is the board size and L is the maximum word length
**Space Complexity**: O(N × K) for the Trie, where N is the number of words and K is the average word length

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- Board:
```
[['a','b'],
 ['c','d']]
```
- Words: `["ab", "ac", "acd", "ba", "bd"]`

**Step 1: Build the Trie**

We construct a Trie from our word list. The structure looks like:
```
root
├── a (ref=-1)
│   ├── b (ref=0, word="ab")
│   └── c (ref=1, word="ac")
│       └── d (ref=2, word="acd")
└── b (ref=-1)
    ├── a (ref=3, word="ba")
    └── d (ref=4, word="bd")
```

**Step 2: Start DFS from board[0][0] = 'a'**

1. Check if 'a' exists in Trie → Yes, continue
2. Mark cell (0,0) as visited: `board[0][0] = '#'`
3. Explore neighbors:
   - Right to (0,1): 'b'
     - Path "ab" exists in Trie and is a complete word → Add "ab" to results
     - No further paths from 'b' under 'a'
   - Down to (1,0): 'c'
     - Path "ac" exists in Trie and is a complete word → Add "ac" to results
     - Continue to explore from 'c'
     - Right to (1,1): 'd'
       - Path "acd" exists in Trie and is a complete word → Add "acd" to results
     - Backtrack from 'd': `board[1][1] = 'd'`
   - Backtrack from 'c': `board[1][0] = 'c'`
4. Backtrack from 'a': `board[0][0] = 'a'`

**Step 3: Start DFS from board[0][1] = 'b'**

1. Check if 'b' exists in Trie → Yes, continue
2. Mark cell (0,1) as visited: `board[0][1] = '#'`
3. Explore neighbors in the order up, right, down, left:
   - Down to (1,1): 'd'
     - Path "bd" exists in Trie and is a complete word → Add "bd" to results
   - Left to (0,0): 'a'
     - Path "ba" exists in Trie and is a complete word → Add "ba" to results
4. Backtrack from 'b': `board[0][1] = 'b'`

**Step 4: Start DFS from board[1][0] = 'c' and board[1][1] = 'd'**

Neither 'c' nor 'd' exist as starting characters in our Trie, so these searches are pruned immediately.

**Final Result:** `["ab", "ac", "acd", "bd", "ba"]`

The key insights from this walkthrough:
- The Trie allows us to check all words simultaneously in a single traversal
- Backtracking (marking with '#' and restoring) ensures each cell is used only once per word
- Early pruning happens when a path doesn't match any prefix (like starting with 'c' or 'd')
- Each word is found exactly once due to marking `ref = -1` after finding

## Python Solution

```python
from typing import List, Optional

class TrieNode:
    def __init__(self):
        # Array to store references to child nodes (26 letters a-z)
        self.children: List[Optional['TrieNode']] = [None] * 26
        # Index reference to the word in the original words list (-1 means no word ends here)
        self.word_index: int = -1

    def insert(self, word: str, index: int) -> None:
        """Insert a word into the trie with its index reference"""
        current_node = self
        for char in word:
            # Calculate the index for this character (0-25)
            char_index = ord(char) - ord('a')
            # Create new node if path doesn't exist
            if current_node.children[char_index] is None:
                current_node.children[char_index] = TrieNode()
            # Move to the child node
            current_node = current_node.children[char_index]
        # Mark the end of word with its index in the words list
        current_node.word_index = index


class Solution:
    def findWords(self, board: List[List[str]], words: List[str]) -> List[str]:
        def dfs(node: TrieNode, row: int, col: int) -> None:
            """Depth-first search to find words in the board"""
            # Get the character at current position
            char_index = ord(board[row][col]) - ord('a')

            # If no child exists for this character, stop exploring
            if node.children[char_index] is None:
                return

            # Move to the child node
            node = node.children[char_index]

            # If we found a complete word, add it to results
            if node.word_index >= 0:
                result.append(words[node.word_index])
                # Mark as visited to avoid duplicates
                node.word_index = -1

            # Temporarily mark current cell as visited
            original_char = board[row][col]
            board[row][col] = '#'

            # Explore all four directions (up, right, down, left)
            directions = [(-1, 0), (0, 1), (1, 0), (0, -1)]
            for delta_row, delta_col in directions:
                new_row, new_col = row + delta_row, col + delta_col
                # Check if the new position is valid and not visited
                if (0 <= new_row < rows and
                    0 <= new_col < cols and
                    board[new_row][new_col] != '#'):
                    dfs(node, new_row, new_col)

            # Restore the original character
            board[row][col] = original_char

        # Build the trie from all words
        trie_root = TrieNode()
        for index, word in enumerate(words):
            trie_root.insert(word, index)

        # Initialize board dimensions and result list
        rows, cols = len(board), len(board[0])
        result = []

        # Start DFS from each cell in the board
        for row in range(rows):
            for col in range(cols):
                dfs(trie_root, row, col)

        return result
```

## Time and Space Complexity

**Time Complexity:** `O(M * N * 4^L + W * L)` where:
- `M * N` is the size of the board (M rows, N columns)
- `L` is the maximum length of words in the word list
- `W` is the number of words in the word list
- The `W * L` term comes from building the Trie (inserting W words, each of length up to L)
- The `M * N * 4^L` term represents the worst-case DFS exploration:
  - We start DFS from each of the `M * N` cells
  - From each cell, we can explore up to 4 directions
  - The maximum depth of exploration is `L` (length of the longest word)
  - In the worst case, we might explore `4^L` paths (though pruning and visited marking reduce this significantly in practice)

**Space Complexity:** `O(W * L + L)` where:
- `W * L` space is used for the Trie structure:
  - In the worst case with no common prefixes, we store all characters of all words
  - Each Trie node contains an array of 26 pointers plus an integer reference
- `L` additional space is used for the recursion call stack (maximum depth is the length of the longest word)
- The `ans` list uses `O(W)` space in the worst case if all words are found
- The board modifications are done in-place, so no additional space is needed for tracking visited cells

## Common Pitfalls

### 1. **Not Handling Duplicate Words in Results**

One of the most common mistakes is adding the same word multiple times to the result list when it can be formed from different starting positions or paths on the board.

**The Problem:**
```python
# Incorrect approach - doesn't prevent duplicates
if node.word_index >= 0:
    result.append(words[node.word_index])
    # Forgetting to mark the word as found
```

If the word "eat" can be formed from multiple paths on the board, it would be added to the result list multiple times.

**The Solution:**
```python
if node.word_index >= 0:
    result.append(words[node.word_index])
    node.word_index = -1  # Mark as found to prevent duplicates
```

By setting `word_index` to -1 after finding a word, we ensure that even if we encounter the same word endpoint again through a different path, it won't be added to the results.

### 2. **Incorrect Backtracking - Not Restoring the Board State**

Another critical mistake is forgetting to restore the board cell after exploring all paths from that cell.

**The Problem:**
```python
# Incorrect - permanently modifying the board
board[row][col] = '#'
for delta_row, delta_col in directions:
    # ... explore neighbors ...
# Forgot to restore board[row][col]!
```

This would permanently mark cells as visited, preventing them from being used in subsequent searches from other starting positions.

**The Solution:**
```python
original_char = board[row][col]
board[row][col] = '#'  # Temporarily mark as visited
# ... explore all directions ...
board[row][col] = original_char  # Always restore the original character
```

### 3. **Memory Leak with Trie Nodes**

When dealing with multiple test cases or large word lists, not properly managing the Trie can lead to memory issues.

**The Problem:**
Not removing words from the Trie after finding them, especially when dealing with a large number of words that share prefixes.

**The Solution:**
Consider implementing a cleanup mechanism that removes unnecessary Trie nodes:

```python
def dfs(node: TrieNode, row: int, col: int) -> None:
    # ... existing code ...

    if node.word_index >= 0:
        result.append(words[node.word_index])
        node.word_index = -1

        # Optional: Prune the trie if this branch has no more words
        if not any(node.children) and node.word_index == -1:
            # This node can be removed (implement parent tracking for this)
            pass
```

### 4. **Index Out of Bounds When Creating Character Index**

**The Problem:**
```python
char_index = ord(char) - ord('a')  # Assumes all characters are lowercase
```

If the board contains uppercase letters or non-alphabetic characters, this will cause incorrect indexing or crashes.

**The Solution:**
Add validation or normalization:
```python
def insert(self, word: str, index: int) -> None:
    current_node = self
    for char in word:
        if not 'a' <= char <= 'z':
            return  # Skip invalid characters or handle appropriately
        char_index = ord(char) - ord('a')
        # ... rest of the code
```

Or normalize input in the main function:
```python
# Normalize board to lowercase if needed
board = [[cell.lower() for cell in row] for row in board]
words = [word.lower() for word in words]
```

