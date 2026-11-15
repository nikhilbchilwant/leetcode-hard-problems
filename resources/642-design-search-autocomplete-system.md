# 642. Design Search Autocomplete System

> Source: https://algo.monster/liteproblems/642

## Problem Statement

You need to design an autocomplete system for a search engine that suggests sentences as users type. The system works with the following specifications:

**Initial Setup:**
- You're given an array of `sentences` and an array of `times` (both of length `n`)
- `sentences[i]` represents a previously typed sentence
- `times[i]` represents how many times that sentence was typed

**How the System Works:**
- Users input characters one by one to form a sentence
- Each sentence ends with the special character `'#'`
- For each character typed (except `'#'`), the system returns the top 3 most frequent sentences that start with the prefix typed so far

**Ranking Rules:**
1. Sentences are ranked by their "hot degree" (frequency of being typed)
2. Higher frequency sentences appear first
3. If two sentences have the same frequency, they're sorted in ASCII order (lexicographically)
4. If fewer than 3 matching sentences exist, return all available matches

**Special Behavior:**
- When `'#'` is input, it means the current sentence is complete
- The completed sentence is added to the system (increasing its count by 1 if it already exists)
- After `'#'` is input, return an empty list and reset for the next sentence

**Implementation Requirements:**

The `AutocompleteSystem` class needs two methods:
1. `AutocompleteSystem(sentences, times)`: Constructor that initializes the system with historical data
2. `input(c)`: Processes a single character input
   - If `c == '#'`: saves the current sentence and returns `[]`
   - Otherwise: returns a list of up to 3 sentence suggestions based on the current prefix

**Example Flow:**
If a user types "i" then " " then "a", the system would:
- After "i": return top 3 sentences starting with "i"
- After "i ": return top 3 sentences starting with "i "
- After "i a": return top 3 sentences starting with "i a"
- After "i a#": save "i a" to the system and return `[]`

## Intuition

The key insight is recognizing that we need a data structure that can efficiently store sentences and retrieve all sentences with a common prefix. When someone types "i sp", we need to quickly find all sentences starting with "i sp" like "i speak", "i spent time", etc.

A [Trie](/problems/trie_intro) (prefix tree) is perfect for this because:
- Each path from root represents a prefix
- We can navigate character by character as the user types
- All sentences with the same prefix share the same path initially

Think of it like a dictionary where instead of looking up complete words, you're finding all words that start with certain letters. As you type "c-a-t", you first go to all words starting with "c", then narrow down to "ca", then to "cat".

The challenge is that after finding the node representing our current prefix, we need to discover ALL possible sentence completions below that node. This is where DFS comes in - we need to explore every branch of the subtree to collect all valid sentences.

Why DFS over BFS? Because we need to collect complete sentences (which are at various depths), not process level by level. DFS naturally follows each path to completion, making it easier to track and store complete sentences.

The workflow becomes:
1. **Build phase**: Insert all historical sentences into the [Trie](/problems/trie_intro), storing frequency at leaf nodes
2. **Search phase**: Navigate to the prefix node as user types
3. **Collection phase**: Use DFS to gather all sentences in the subtree
4. **Ranking phase**: Sort by frequency and alphabetically, return top 3

The special character `'#'` acts as a commit - it adds the typed sentence to our [Trie](/problems/trie_intro) (incrementing its count), making the system learn from user input over time. This creates a feedback loop where frequently typed sentences become higher-ranked suggestions.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Trie](/problems/trie_intro), [Data Stream](/problems/median_of_data_stream), [Sorting](/problems/sorting_summary) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution implements a [Trie](/problems/trie_intro)-based autocomplete system with DFS traversal for collecting suggestions. Let's break down the implementation:

### [Trie](/problems/trie_intro) Data Structure

The `Trie` class is designed to store sentences efficiently:
- `children`: An array of 27 elements (26 for letters 'a'-'z' and 1 for space character at index 26)
- `v`: Stores the frequency count of the sentence ending at this node
- `w`: Stores the complete sentence (only at nodes where sentences end)

```python
def insert(self, w, t):
    node = self
    for c in w:
        idx = 26 if c == ' ' else ord(c) - ord('a')
        if node.children[idx] is None:
            node.children[idx] = Trie()
        node = node.children[idx]
    node.v += t  # Increment frequency
    node.w = w   # Store complete sentence
```

The insert method navigates through the [Trie](/problems/trie_intro), creating nodes as needed, and updates the frequency at the sentence's end node.

### Search for Prefix Node

The `search` method finds the node representing the typed prefix:
```python
def search(self, pref):
    node = self
    for c in pref:
        idx = 26 if c == ' ' else ord(c) - ord('a')
        if node.children[idx] is None:
            return None  # Prefix doesn't exist
        node = node.children[idx]
    return node
```

### AutocompleteSystem Implementation

The main class maintains:
- `self.trie`: The Trie storing all sentences
- `self.t`: A list tracking the current input being typed

**Initialization**: Populates the [Trie](/problems/trie_intro) with historical data:
```python
for a, b in zip(sentences, times):
    self.trie.insert(a, b)
```

**Input Processing**: The `input` method handles two cases:

1. **When `c == '#'`**:
   - Joins the accumulated characters into a sentence
   - Inserts it into the [Trie](/problems/trie_intro) with count 1
   - Resets the current input buffer
   - Returns empty list

2. **For regular characters**:
   - Appends character to current input buffer
   - Searches for the prefix node
   - Uses DFS to collect all sentences below that node
   - Sorts results by frequency (descending) then lexicographically
   - Returns top 3

### DFS Collection

The nested `dfs` function recursively explores the subtree:
```python
def dfs(node):
    if node is None:
        return
    if node.v:  # Found a complete sentence
        res.append((node.v, node.w))
    for nxt in node.children:
        dfs(nxt)  # Explore all children
```

This collects all sentences as `(frequency, sentence)` tuples.

### [Sorting](/problems/sorting_summary) and Ranking

Results are sorted using a custom key:
```python
res.sort(key=lambda x: (-x[0], x[1]))
```
- `-x[0]`: Negative frequency for descending order (higher frequency first)
- `x[1]`: Sentence string for ascending alphabetical order when frequencies are equal

Finally, the top 3 sentences are extracted and returned:
```python
return [v[1] for v in res[:3]]
```

The time complexity for each input operation is O(p + m + mlogm) where p is the prefix length, m is the number of sentences with that prefix, and the space complexity is O(total characters in all sentences) for storing the [Trie](/problems/trie_intro).

## Example Walkthrough

Let's walk through a small example to illustrate how the autocomplete system works.

**Initial Setup:**
```
sentences = ["i love you", "island", "i love coding"]
times = [5, 3, 2]
```

**Step 1: Building the Trie**

The system builds a Trie where each path represents a sentence:
```
root
 └─ i (no value)
     ├─ ' ' (space)
     │   └─ l
     │       └─ o
     │           └─ v
     │               └─ e
     │                   ├─ ' '
     │                   │   ├─ y
     │                   │   │   └─ o
     │                   │   │       └─ u (v=5, w="i love you")
     │                   │   └─ c
     │                   │       └─ o
     │                   │           └─ d
     │                   │               └─ i
     │                   │                   └─ n
     │                   │                       └─ g (v=2, w="i love coding")
     └─ s
         └─ l
             └─ a
                 └─ n
                     └─ d (v=3, w="island")
```

**Step 2: User Types "i"**

1. Current buffer: `["i"]`
2. Search for prefix "i" in Trie → finds node at 'i'
3. DFS from this node collects all sentences in subtree:
   - Found: ("i love you", 5)
   - Found: ("island", 3)
   - Found: ("i love coding", 2)
4. Sort by frequency (desc), then alphabetically:
   - ["i love you" (5), "island" (3), "i love coding" (2)]
5. Return top 3: `["i love you", "island", "i love coding"]`

**Step 3: User Types " " (space)**

1. Current buffer: `["i", " "]`
2. Search for prefix "i " → finds node at 'i' → ' '
3. DFS collects sentences starting with "i ":
   - Found: ("i love you", 5)
   - Found: ("i love coding", 2)
4. Sort results:
   - ["i love you" (5), "i love coding" (2)]
5. Return: `["i love you", "i love coding"]`

**Step 4: User Types "a"**

1. Current buffer: `["i", " ", "a"]`
2. Search for prefix "i a" → returns None (no such prefix exists)
3. Return: `[]`

**Step 5: User Types "#"**

1. Current sentence: "i a"
2. Insert "i a" into Trie with frequency 1
3. Trie now has a new path: root → i → ' ' → a (v=1, w="i a")
4. Clear buffer: `[]`
5. Return: `[]`

**Step 6: User Types "i" Again**

1. Current buffer: `["i"]`
2. Search for prefix "i" → finds node
3. DFS collects:
   - ("i love you", 5)
   - ("island", 3)
   - ("i love coding", 2)
   - ("i a", 1) ← newly added
4. Sort and return top 3: `["i love you", "island", "i love coding"]`

This example demonstrates:
- How the Trie efficiently stores sentences with common prefixes
- How DFS collects all matching sentences from a subtree
- How the ranking system prioritizes by frequency, then alphabetically
- How the system learns by adding new sentences when '#' is typed

## Python Solution

```python
from typing import List, Optional

class TrieNode:
    def __init__(self):
        # 26 letters + 1 space character
        self.children = [None] * 27
        # Frequency count of the sentence ending at this node
        self.frequency = 0
        # The complete sentence if this node represents end of a sentence
        self.sentence = ''

    def insert(self, sentence: str, count: int) -> None:
        """Insert a sentence into the trie with its frequency count."""
        current_node = self

        for char in sentence:
            # Calculate index: 26 for space, 0-25 for 'a'-'z'
            index = 26 if char == ' ' else ord(char) - ord('a')

            # Create new node if path doesn't exist
            if current_node.children[index] is None:
                current_node.children[index] = TrieNode()

            current_node = current_node.children[index]

        # Update frequency and store the complete sentence at leaf node
        current_node.frequency += count
        current_node.sentence = sentence

    def search(self, prefix: str) -> Optional['TrieNode']:
        """Search for a prefix in the trie and return the ending node."""
        current_node = self

        for char in prefix:
            # Calculate index: 26 for space, 0-25 for 'a'-'z'
            index = 26 if char == ' ' else ord(char) - ord('a')

            # Prefix doesn't exist in trie
            if current_node.children[index] is None:
                return None

            current_node = current_node.children[index]

        return current_node


class AutocompleteSystem:
    def __init__(self, sentences: List[str], times: List[int]):
        """Initialize the autocomplete system with historical data."""
        # Initialize the trie root
        self.trie_root = TrieNode()

        # Insert all historical sentences with their frequencies
        for sentence, frequency in zip(sentences, times):
            self.trie_root.insert(sentence, frequency)

        # Buffer to store current input characters
        self.current_input = []

    def input(self, c: str) -> List[str]:
        """
        Process input character and return top 3 autocomplete suggestions.

        Args:
            c: Input character (lowercase letter, space, or '#')

        Returns:
            List of top 3 sentences sorted by frequency (desc) and lexicographically
        """

        def collect_all_sentences(node: Optional[TrieNode]) -> None:
            """DFS to collect all sentences from given node and its subtree."""
            if node is None:
                return

            # If this node represents end of a sentence, add to results
            if node.frequency > 0:
                autocomplete_results.append((node.frequency, node.sentence))

            # Recursively search all children
            for child_node in node.children:
                collect_all_sentences(child_node)

        # Handle end of input - save the sentence
        if c == '#':
            # Construct the complete sentence from buffer
            completed_sentence = ''.join(self.current_input)
            # Add this new sentence to trie with frequency 1
            self.trie_root.insert(completed_sentence, 1)
            # Clear the buffer for next input
            self.current_input = []
            return []

        # Add current character to buffer
        self.current_input.append(c)

        # Search for current prefix in trie
        current_prefix = ''.join(self.current_input)
        prefix_node = self.trie_root.search(current_prefix)

        # If prefix doesn't exist, return empty list
        if prefix_node is None:
            return []

        # Collect all sentences starting with current prefix
        autocomplete_results = []
        collect_all_sentences(prefix_node)

        # Sort by frequency (descending) and then lexicographically
        autocomplete_results.sort(key=lambda x: (-x[0], x[1]))

        # Return top 3 sentences (extract just the sentence strings)
        return [sentence for frequency, sentence in autocomplete_results[:3]]


# Your AutocompleteSystem object will be instantiated and called as such:
# obj = AutocompleteSystem(sentences, times)
# param_1 = obj.input(c)
```

## Time and Space Complexity

**Time Complexity:**

- **Initialization (`__init__`):** `O(n * m)` where `n` is the number of sentences and `m` is the average length of each sentence. Each sentence needs to be inserted into the Trie, and insertion takes `O(m)` time per sentence.

- **`input` method:**
  - When `c == '#'`: `O(p)` where `p` is the length of the current prefix being inserted. The insertion operation traverses through each character of the string.
  - When `c != '#'`:
    - Searching for the prefix node: `O(p)` where `p` is the current length of the typed prefix
    - DFS traversal: `O(N)` where `N` is the total number of nodes in the subtree rooted at the prefix node (worst case: all nodes in the Trie)
    - Sorting results: `O(k * log k)` where `k` is the number of complete sentences found in the subtree
    - Overall: `O(p + N + k * log k)`, but since we only return top 3, the practical complexity is `O(p + N)`

**Space Complexity:**

- **Trie structure:** `O(T * 27)` where `T` is the total number of unique prefixes across all sentences. Each node can have up to 27 children (26 letters + 1 space).

- **`AutocompleteSystem` class:**
  - The Trie itself: `O(T * 27)`
  - The buffer `self.t`: `O(p)` where `p` is the maximum length of the current input sequence
  - The `res` list in DFS: `O(k)` where `k` is the number of complete sentences found

- **Overall space complexity:** `O(T * 27 + p + k)`, which simplifies to `O(T)` as the Trie dominates the space usage.

- **DFS recursion stack:** `O(h)` where `h` is the maximum height of the Trie (longest sentence length)

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Character Index Mapping**

One of the most common mistakes is incorrectly mapping characters to array indices, especially forgetting to handle the space character properly.

**Pitfall Example:**
```python
# WRONG: This will cause IndexError or incorrect behavior
index = ord(char) - ord('a')  # Doesn't handle space character
```

**Solution:**
Always check for space character explicitly:
```python
index = 26 if char == ' ' else ord(char) - ord('a')
```

### 2. **Not Accumulating Frequency for Duplicate Sentences**

A critical error is overwriting the frequency instead of accumulating it when the same sentence is inserted multiple times.

**Pitfall Example:**
```python
# WRONG: This overwrites existing frequency
current_node.frequency = count  # Should be +=
```

**Solution:**
Always use addition assignment to accumulate frequencies:
```python
current_node.frequency += count
```

### 3. **Forgetting to Reset Input Buffer After '#'**

Failing to clear the input buffer after processing '#' will cause subsequent inputs to be concatenated with the previous sentence.

**Pitfall Example:**
```python
if c == '#':
    completed_sentence = ''.join(self.current_input)
    self.trie_root.insert(completed_sentence, 1)
    # WRONG: Forgot to reset self.current_input
    return []
```

**Solution:**
Always reset the buffer after processing a complete sentence:
```python
if c == '#':
    completed_sentence = ''.join(self.current_input)
    self.trie_root.insert(completed_sentence, 1)
    self.current_input = []  # Reset for next input
    return []
```

### 4. **Incorrect Sorting Logic**

The sorting requirements are specific: first by frequency (descending), then lexicographically (ascending). Getting this wrong will produce incorrect top-3 results.

**Pitfall Example:**
```python
# WRONG: Both in descending order
autocomplete_results.sort(key=lambda x: (-x[0], -x[1]))

# WRONG: Wrong order of sorting criteria
autocomplete_results.sort(key=lambda x: (x[1], -x[0]))
```

**Solution:**
Use negative frequency for descending order, keep sentence as-is for ascending:
```python
autocomplete_results.sort(key=lambda x: (-x[0], x[1]))
```

### 5. **Storing Sentence at Every Node Instead of Leaf Nodes**

Storing the sentence string at every node along the path wastes memory and can cause confusion.

**Pitfall Example:**
```python
def insert(self, sentence: str, count: int) -> None:
    current_node = self
    for char in sentence:
        index = 26 if char == ' ' else ord(char) - ord('a')
        if current_node.children[index] is None:
            current_node.children[index] = TrieNode()
        current_node = current_node.children[index]
        current_node.sentence = sentence  # WRONG: Storing at every node
```

**Solution:**
Only store the complete sentence at the node where it ends:
```python
def insert(self, sentence: str, count: int) -> None:
    current_node = self
    for char in sentence:
        # ... traverse to end node ...
    # Only store at the final node
    current_node.frequency += count
    current_node.sentence = sentence
```

### 6. **Dropping the None Check in Both Places**

When no sentences match the current prefix, `search` returns `None`. The solution handles this twice: `input` returns `[]` early, and `collect_all_sentences` also returns immediately on `None`. Either check alone keeps the result correct. Removing both makes the traversal read `node.children` on `None`, which raises an `AttributeError`.

**Pitfall Example:**
```python
def collect_all_sentences(node):
    if node.frequency > 0:  # AttributeError when node is None
        autocomplete_results.append((node.frequency, node.sentence))
    for child_node in node.children:
        if child_node:
            collect_all_sentences(child_node)

prefix_node = self.trie_root.search(current_prefix)
collect_all_sentences(prefix_node)  # prefix_node may be None
```

**Solution:**
Return early when the prefix does not exist:
```python
prefix_node = self.trie_root.search(current_prefix)
if prefix_node is None:
    return []
collect_all_sentences(prefix_node)
```

