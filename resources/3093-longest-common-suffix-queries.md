# 3093. Longest Common Suffix Queries

> Source: https://algo.monster/liteproblems/3093

## Problem Statement

You have two arrays of strings: `wordsContainer` and `wordsQuery`.

For each query string `wordsQuery[i]`, you need to find a string from `wordsContainer` that shares the **longest common suffix** with it. A suffix is a substring that appears at the end of a string. For example, "ing" is a suffix of "running".

When multiple strings in `wordsContainer` share the same longest common suffix with a query:
1. **First tiebreaker**: Choose the string with the smallest length
2. **Second tiebreaker**: If multiple strings have the same smallest length, choose the one that appears earlier in `wordsContainer` (smaller index)

Return an array `ans` where `ans[i]` is the index of the string in `wordsContainer` that best matches `wordsQuery[i]` according to the rules above.

**Example walkthrough:**

If `wordsContainer = ["abc", "bbc", "cc"]` and `wordsQuery = ["c"]`:
- "abc" has suffix "c" in common (length 1)
- "bbc" has suffix "c" in common (length 1) 
- "cc" has suffix "c" in common (length 1)

All three share the same longest common suffix of length 1. Among them, "cc" has the smallest length (2), so the answer would be index 2.

The solution uses a **[Trie](/problems/trie_intro) (prefix tree)** data structure, but since we're dealing with suffixes, strings are inserted in reverse order. Each Trie node tracks:
- `children`: Array of 26 child nodes (for each letter a-z)
- `length`: The length of the shortest string passing through this node
- `idx`: The index of that shortest string in `wordsContainer`

During insertion, each string is reversed and added character by character. At each node, if the current string is shorter than the stored `length`, the node's `length` and `idx` are updated.

For queries, we traverse the reversed query string through the [Trie](/problems/trie_intro) as far as possible. When we can't continue (no matching child), we return the `idx` stored at the current node, which represents the best match according to the problem's criteria.

## Intuition

The key insight is recognizing that finding the **longest common suffix** is essentially the same problem as finding the **longest common prefix**, but with strings reversed. This immediately suggests using a [Trie](/problems/trie_intro) data structure, which excels at prefix matching.

Think about how we normally use a [Trie](/problems/trie_intro) for prefix matching: we insert words character by character from the beginning, creating a tree where common prefixes share the same path. To adapt this for suffix matching, we can simply insert words in reverse order. For example, "abc" becomes "cba" when inserted, so the suffix "bc" becomes the prefix "cb" in our reversed Trie.

The challenging part is handling the tiebreaking rules. When multiple strings share the same longest common suffix, we need to pick the shortest one, and if there's still a tie, the one with the smallest index. 

Here's the clever part: we can handle this during [Trie](/problems/trie_intro) construction itself. At each node in the Trie, we store two pieces of information:
- The length of the shortest string that passes through this node
- The index of that string in the original array

As we insert each string, we update these values at every node along the path if we find a shorter string (or same length but earlier index). This way, each node always "remembers" the best candidate according to our tiebreaking rules.

When querying, we traverse the [Trie](/problems/trie_intro) with the reversed query string as far as we can go. The moment we can't continue (either we hit a null child or reach the end of the query), the current node already contains the index of the best matching string. No additional comparison is needed because we've pre-computed the optimal choice during insertion.

This approach is efficient because:
1. We only build the [Trie](/problems/trie_intro) once for all queries
2. Each query is answered in `O(m)` time where `m` is the query string length
3. The tiebreaking logic is handled during insertion rather than at query time

**Learn more about [Trie](/problems/trie_intro) patterns.**

## Solution Approach

The implementation consists of two main components: the `[Trie](/problems/trie_intro)` class for suffix matching and the `Solution` class that orchestrates the process.

### [Trie](/problems/trie_intro) Node Structure

Each [Trie](/problems/trie_intro) node contains:
- `children`: An array of size 26 (for letters a-z) to store child nodes
- `length`: Tracks the length of the shortest string passing through this node
- `idx`: Stores the index of that shortest string in `wordsContainer`

We use `__slots__` for memory optimization and initialize `length` and `idx` to infinity (`inf`) to ensure any actual string will update these values.

### Insert Operation

The `insert(w, i)` method adds a string `w` with index `i` to the [Trie](/problems/trie_intro):

1. **Update root node**: Before traversing, we check if the current string is shorter than what's stored at the root. If so, we update the root's `length` and `idx`.

2. **Reverse traversal**: We iterate through the string in reverse order using `w[::-1]`. For example, "abc" is processed as "c", "b", "a".

3. **Character-by-character insertion**: For each character:
   - Calculate its index: `idx = ord(c) - ord("a")` (converts 'a' to 0, 'b' to 1, etc.)
   - Create a new [Trie](/problems/trie_intro) node if the path doesn't exist
   - Move to the child node
   - Update the child node's `length` and `idx` if the current string is shorter

This ensures that at each node along the path, we maintain the index of the shortest string that passes through it.

### Query Operation

The `query(w)` method finds the best matching string for a query:

1. **Reverse traversal**: Similar to insertion, we traverse the query string in reverse using `w[::-1]`.

2. **Follow the path**: For each character, we attempt to follow the corresponding child node.

3. **Stop condition**: If we encounter a `None` child (path doesn't exist), we break the loop. This happens when we've found the longest possible common suffix.

4. **Return result**: We return the `idx` stored at the current node, which represents the best matching string according to our tiebreaking rules.

### Main Solution

The `stringIndices` method coordinates the entire process:

1. **Build the [Trie](/problems/trie_intro)**: Create a new Trie and insert all strings from `wordsContainer` with their indices:
   ```python
   for i, w in enumerate(wordsContainer):
       trie.insert(w, i)
   ```

2. **Process queries**: For each query string, find the best match using the [Trie](/problems/trie_intro):
   ```python
   return [trie.query(w) for w in wordsQuery]
   ```

### Time and Space Complexity

- **Time Complexity**: 
  - Building the [Trie](/problems/trie_intro): `O(n * m)` where `n` is the number of strings in `wordsContainer` and `m` is the average string length
  - Processing queries: `O(q * m)` where `q` is the number of queries
  - Total: `O((n + q) * m)`

- **Space Complexity**: `O(n * m)` for storing the [Trie](/problems/trie_intro) structure

The elegance of this solution lies in pre-computing the tiebreaking decisions during [Trie](/problems/trie_intro) construction, making query operations both simple and efficient.

## Example Walkthrough

Let's walk through a concrete example to understand how the Trie-based solution works.

**Given:**
- `wordsContainer = ["abcd", "bcd", "xbcd"]`
- `wordsQuery = ["cd", "bcd", "xyz"]`

**Step 1: Build the Trie**

We insert each word from `wordsContainer` in reverse order:

1. Insert "abcd" (index 0):
   - Reverse it: "dcba"
   - Create path: root → d → c → b → a
   - At each node, store length=4, idx=0

2. Insert "bcd" (index 1):
   - Reverse it: "dcb"
   - Follow existing path: root → d → c → b
   - At root: length=3, idx=1 (3 < 4, so update)
   - At 'd': length=3, idx=1 (3 < 4, so update)
   - At 'c': length=3, idx=1 (3 < 4, so update)
   - At 'b': length=3, idx=1 (3 < 4, so update)

3. Insert "xbcd" (index 2):
   - Reverse it: "dcbx"
   - Follow path: root → d → c → b → x (create new node 'x')
   - At root: keep length=3, idx=1 (4 > 3, no update)
   - At 'd': keep length=3, idx=1 (4 > 3, no update)
   - At 'c': keep length=3, idx=1 (4 > 3, no update)
   - At 'b': keep length=3, idx=1 (4 > 3, no update)
   - At 'x': length=4, idx=2

**Trie structure after insertion:**
```
root (len=3, idx=1)
  └─ d (len=3, idx=1)
      └─ c (len=3, idx=1)
          └─ b (len=3, idx=1)
              ├─ a (len=4, idx=0)
              └─ x (len=4, idx=2)
```

**Step 2: Process Queries**

1. Query "cd":
   - Reverse it: "dc"
   - Traverse: root → d → c
   - Can continue to 'd' and 'c'
   - Can't continue further (query exhausted)
   - Return idx at node 'c' = 1
   - Answer: index 1 ("bcd" shares suffix "cd")

2. Query "bcd":
   - Reverse it: "dcb"
   - Traverse: root → d → c → b
   - Can continue through all characters
   - Query exhausted at node 'b'
   - Return idx at node 'b' = 1
   - Answer: index 1 ("bcd" shares suffix "bcd")

3. Query "xyz":
   - Reverse it: "zyx"
   - Try to traverse: root → z
   - No child 'z' exists at root
   - Return idx at root = 1
   - Answer: index 1 ("bcd" has the shortest length among all words)

**Final Result:** `[1, 1, 1]`

This example demonstrates how:
- The Trie stores reversed strings to convert suffix matching to prefix matching
- Each node maintains the index of the shortest string passing through it
- Queries traverse as far as possible and return the stored index
- When no common suffix exists (like "xyz"), the root node provides the fallback answer

## Python Solution

```python
from typing import List
from math import inf


class Trie:
    """A Trie data structure for storing strings in reverse order to find longest common suffixes."""
  
    __slots__ = ("children", "length", "idx")
  
    def __init__(self):
        """Initialize a Trie node with 26 children (for lowercase English letters)."""
        # Array to store child nodes for each letter a-z
        self.children = [None] * 26
        # Store the minimum length of strings passing through this node
        self.length = inf
        # Store the index of the string with minimum length at this node
        self.idx = inf
  
    def insert(self, word: str, index: int) -> None:
        """
        Insert a word into the Trie in reverse order.
      
        Args:
            word: The string to insert
            index: The original index of the word in the container
        """
        node = self
      
        # Update root node with minimum length word information
        if node.length > len(word):
            node.length = len(word)
            node.idx = index
      
        # Insert characters in reverse order (for suffix matching)
        for char in word[::-1]:
            # Calculate the index for this character (0-25 for a-z)
            char_index = ord(char) - ord("a")
          
            # Create new node if path doesn't exist
            if node.children[char_index] is None:
                node.children[char_index] = Trie()
          
            # Move to child node
            node = node.children[char_index]
          
            # Update this node with minimum length word information
            if node.length > len(word):
                node.length = len(word)
                node.idx = index
  
    def query(self, word: str) -> int:
        """
        Find the index of the word with the longest common suffix.
      
        Args:
            word: The query string
          
        Returns:
            The index of the container word with the longest matching suffix
            and minimum length among those with the same suffix
        """
        node = self
      
        # Traverse the Trie following the reverse of the query word
        for char in word[::-1]:
            char_index = ord(char) - ord("a")
          
            # Stop if no further matching suffix exists
            if node.children[char_index] is None:
                break
          
            # Move to the next node
            node = node.children[char_index]
      
        # Return the index of the best matching word at this node
        return node.idx


class Solution:
    """Solution class for finding strings with longest common suffixes."""
  
    def stringIndices(
        self, wordsContainer: List[str], wordsQuery: List[str]
    ) -> List[int]:
        """
        Find indices of container words with longest common suffix for each query.
      
        Args:
            wordsContainer: List of strings to search from
            wordsQuery: List of query strings
          
        Returns:
            List of indices where each index corresponds to the container word
            with the longest common suffix for each query word
        """
        # Build the Trie with all container words
        trie = Trie()
        for i, word in enumerate(wordsContainer):
            trie.insert(word, i)
      
        # Process each query and return results
        return [trie.query(word) for word in wordsQuery]
```

## Time and Space Complexity

**Time Complexity:** `O(L₁ + L₂)`

The time complexity consists of two parts:
- Building the trie: For each word in `wordsContainer`, we insert it character by character in reverse order. This takes `O(L₁)` time, where `L₁` is the sum of lengths of all words in `wordsContainer`.
- Querying the trie: For each word in `wordsQuery`, we traverse the trie character by character in reverse order until we can't proceed further. This takes `O(L₂)` time, where `L₂` is the sum of lengths of all words in `wordsQuery`.

Total time complexity: `O(L₁ + L₂)`

**Space Complexity:** `O(L₁ × |Σ|)`

The space is dominated by the trie structure:
- In the worst case, we create a new trie node for each character in `wordsContainer`.
- Each trie node contains an array of size 26 (for lowercase English letters, where `|Σ| = 26`).
- The maximum number of nodes is proportional to `L₁` (the total number of characters in `wordsContainer`).
- Therefore, the space complexity is `O(L₁ × |Σ|)` or `O(L₁ × 26) = O(26L₁) = O(L₁ × |Σ|)`.

Note: The reference answer shows `O(L₁ × |Σ| + L₂)` for time complexity, but the `L₂` term doesn't multiply with `|Σ|` because during queries, we only traverse existing paths in the trie without creating new nodes or iterating through the alphabet at each node.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Forgetting to Update the Root Node

**The Pitfall:**
A critical but easy-to-miss detail is updating the root node's `length` and `idx` before traversing the Trie during insertion. The root node represents the empty suffix (no common characters), which every string shares. If you skip this update, queries that have no matching characters with any container word will return incorrect results.

**Why It Happens:**
When traversing a Trie, developers often focus on updating nodes along the path but forget that the root itself needs to track the shortest string in the entire collection.

**Example of the Bug:**
```python
def insert(self, word: str, index: int) -> None:
    node = self
  
    # BUG: Missing root node update!
    # If we skip this, queries with no common suffix fail
  
    for char in word[::-1]:
        char_index = ord(char) - ord("a")
        if node.children[char_index] is None:
            node.children[char_index] = Trie()
        node = node.children[char_index]
        if node.length > len(word):
            node.length = len(word)
            node.idx = index
```

**Test Case That Exposes the Bug:**
```python
wordsContainer = ["abc", "def", "ghi"]
wordsQuery = ["xyz"]  # No common suffix with any container word

# Without root update: Returns inf or crashes
# With root update: Correctly returns 0 (shortest word "abc")
```

**The Solution:**
Always update the root node before traversing:
```python
def insert(self, word: str, index: int) -> None:
    node = self
  
    # CRITICAL: Update root node first
    if node.length > len(word):
        node.length = len(word)
        node.idx = index
  
    # Then traverse and update child nodes
    for char in word[::-1]:
        # ... rest of the insertion logic
```

### 2. Incorrect Tiebreaking Logic

**The Pitfall:**
Using `>=` instead of `>` when comparing lengths leads to incorrect tiebreaking. The problem specifies that when multiple strings have the same length, we should choose the one with the smaller index (earlier in the array).

**Example of the Bug:**
```python
# WRONG: This updates even when lengths are equal
if node.length >= len(word):  # Bug: should be >
    node.length = len(word)
    node.idx = index
```

**Test Case That Exposes the Bug:**
```python
wordsContainer = ["abc", "bcd", "cde"]  # All length 3
wordsQuery = ["c"]  # All have suffix "c"

# With >= : Returns 2 (last index)
# With > : Returns 0 (first index with minimum length)
```

**The Solution:**
Use strict inequality to preserve the first occurrence:
```python
# CORRECT: Only update if strictly shorter
if node.length > len(word):
    node.length = len(word)
    node.idx = index
```

### 3. Not Handling Empty Strings or Edge Cases

**The Pitfall:**
The code assumes all strings are non-empty and contain only lowercase letters. Empty strings or special characters can cause unexpected behavior.

**Defensive Programming Solution:**
```python
def insert(self, word: str, index: int) -> None:
    if not word:  # Handle empty string
        return
  
    node = self
    if node.length > len(word):
        node.length = len(word)
        node.idx = index
  
    for char in word[::-1]:
        if not 'a' <= char <= 'z':  # Validate character
            continue
        char_index = ord(char) - ord("a")
        # ... rest of the logic
```

### Key Takeaway

The most critical pitfall is forgetting to update the root node. This subtle bug can cause the solution to fail completely for queries that don't share any suffix with container words. Always remember that the root node represents the universal case where no characters match, and it must track the shortest string in the entire collection for correct fallback behavior.

