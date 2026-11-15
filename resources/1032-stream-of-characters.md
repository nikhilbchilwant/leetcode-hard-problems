# 1032. Stream of Characters

> Source: https://algo.monster/liteproblems/1032

## Problem Statement

This problem asks you to design a data structure that can efficiently check if any suffix of a stream of characters matches any word from a given list of words.

You need to implement a `StreamChecker` class with two methods:

1. **Constructor `StreamChecker(String[] words)`**: Initializes the stream checker with an array of words that you'll be checking against.

2. **Method `query(char letter)`**: Takes a single character as input, adds it to the stream, and returns `true` if any suffix of the current stream matches any word from the initial word list, otherwise returns `false`.

A **suffix** is a substring that ends at the last character of the string. For example, if the stream contains `"axyz"`, the suffixes are: `"z"`, `"yz"`, `"xyz"`, and `"axyz"`.

**Example walkthrough:**
- Given `words = ["abc", "xyz"]`
- Stream receives characters one by one: `'a'`, `'x'`, `'y'`, `'z'`
- After receiving all four characters, the stream is `"axyz"`
- The suffix `"xyz"` matches one of the words in the array
- So `query('z')` would return `true`

The solution uses a **[Trie](/problems/trie_intro) (prefix tree) with reversed words** approach:
- All words are inserted into the Trie in **reverse order** (e.g., `"abc"` becomes `"cba"`)
- As characters arrive, they're stored in a list
- For each query, the solution searches the Trie by traversing the character list in reverse order (most recent character first)
- If at any point during the traversal a word ending is found in the Trie, it means a matching suffix exists
- The `limit` of 201 characters is used to optimize memory by only keeping the most recent characters (since the maximum word length in typical constraints is 200)

## Intuition

The key insight is recognizing that checking for suffixes in a stream is computationally expensive if done naively. For each new character, we'd need to check all possible suffixes against all words, which would be inefficient.

Let's think about what happens when we check for suffixes. If our stream is `"axyz"`, we need to check if `"z"`, `"yz"`, `"xyz"`, or `"axyz"` match any word. Notice that we're always checking from the **end** of the stream backwards.

This observation leads to a clever transformation: **what if we reverse the problem?** Instead of checking suffixes from the end, we can check prefixes from the beginning by reversing both the words and the search order.

Here's the thought process:
1. If we store words in reverse (e.g., `"xyz"` becomes `"zyx"`), then checking if `"xyz"` is a suffix of the stream becomes checking if `"zyx"` is a prefix when we read the stream backwards.

2. Reading the stream backwards means starting from the most recent character. So for stream `"axyz"`, we'd check in order: `"z"`, `"zy"`, `"zyx"`, `"zyxa"`.

3. A [Trie](/problems/trie_intro) is perfect for prefix matching! As we traverse character by character, we can immediately know if we're forming a valid prefix of any reversed word.

4. The beauty of this approach is that we can stop early - if at any point during our backwards traversal we find a complete word in the [Trie](/problems/trie_intro) (marked by `is_end = True`), we've found a matching suffix.

The `limit` of 201 characters comes from practical constraints - we don't need to keep the entire stream history, just enough to cover the longest possible word (typically 200 characters in LeetCode problems) plus one extra character.

This reversal technique transforms a suffix-matching problem into a prefix-matching problem, allowing us to use the [Trie](/problems/trie_intro)'s O(m) search complexity where m is the length of the word being checked.

**Learn more about [Trie](/problems/trie_intro) and [Data Stream](/problems/median_of_data_stream) patterns.**

## Solution Approach

The solution implements two classes: a `[Trie](/problems/trie_intro)` data structure for efficient prefix matching and the main `StreamChecker` class.

**[Trie](/problems/trie_intro) Implementation:**

The `[Trie](/problems/trie_intro)` class has three main components:
- `children`: An array of 26 elements (for lowercase letters a-z), where each element can point to another Trie node
- `is_end`: A boolean flag marking if a word ends at this node
- `insert(w)`: Inserts a word **in reverse order** into the Trie
- `search(w)`: Searches for any word that matches when traversing the input in reverse

The `insert` method works by:
1. Starting from the root node
2. Iterating through the word in reverse (`w[::-1]`)
3. For each character, calculating its index (`ord(c) - ord('a')`)
4. Creating new [Trie](/problems/trie_intro) nodes as needed along the path
5. Marking the final node with `is_end = True`

The `search` method:
1. Traverses the input list in reverse order (`w[::-1]`)
2. For each character, follows the corresponding child node
3. If no child exists, returns `False` (no matching word)
4. If it finds a node with `is_end = True`, returns `True` immediately (found a matching suffix)

**StreamChecker Implementation:**

The `StreamChecker` class maintains:
- `self.[trie](/problems/trie_intro)`: The Trie containing all reversed words
- `self.cs`: A list storing the stream of characters
- `self.limit`: Set to 201 to optimize memory usage

During initialization:
```python
def __init__(self, words: List[str]):
    self.[trie](/problems/trie_intro) = Trie()
    self.cs = []
    self.limit = 201
    for w in words:
        self.trie.insert(w)
```
Each word is inserted into the Trie in reverse order.

The `query` method:
```python
def query(self, letter: str) -> bool:
    self.cs.append(letter)
    return self.[trie](/problems/trie_intro).search(self.cs[-self.limit :])
```
1. Appends the new character to the stream list
2. Passes only the last 201 characters to the search method (using slicing `self.cs[-self.limit:]`)
3. The search automatically checks all possible suffixes by traversing backwards

**Time Complexity:**
- Initialization: O(n × m) where n is the number of words and m is the average word length
- Query: O(min(m, 201)) where m is the longest word length

**Space Complexity:**
- O(n × m) for the [Trie](/problems/trie_intro) structure
- O(201) for the character stream buffer

The clever use of reversal transforms suffix matching into prefix matching, making the [Trie](/problems/trie_intro) an ideal data structure for this problem.

## Example Walkthrough

Let's walk through a concrete example with `words = ["cd", "f", "kl"]` and a stream of queries.

**Step 1: Build the Trie with Reversed Words**
- Insert "cd" reversed → "dc"
- Insert "f" reversed → "f" 
- Insert "kl" reversed → "lk"

Our Trie structure looks like:
```
       root
      /  |  \
     d   f   l
     |   *   |
     c       k
     *       *
(* indicates is_end = True)
```

**Step 2: Process Queries**

**Query 1: `query('a')`**
- Stream becomes: `['a']`
- Search path in Trie (reading backwards): Start with 'a'
- No child at index 'a' from root → Return `false`

**Query 2: `query('b')`**
- Stream becomes: `['a', 'b']`
- Search path: 'b' → 'ba' (reading backwards)
- No child at index 'b' from root → Return `false`

**Query 3: `query('c')`**
- Stream becomes: `['a', 'b', 'c']`
- Search path: 'c' → 'cb' → 'cba'
- No child at index 'c' from root → Return `false`

**Query 4: `query('d')`**
- Stream becomes: `['a', 'b', 'c', 'd']`
- Search path: 'd' → 'dc' → 'dcb' → 'dcba'
- Follow 'd' from root (exists)
- Follow 'c' from 'd' node (exists, has `is_end = True`)
- Found complete word "dc" (which is "cd" reversed) → Return `true`
- This means "cd" is a suffix of "abcd"

**Query 5: `query('e')`**
- Stream becomes: `['a', 'b', 'c', 'd', 'e']`
- Search path: 'e' → 'ed' → 'edc' → 'edcb' → 'edcba'
- No child at index 'e' from root → Return `false`

**Query 6: `query('f')`**
- Stream becomes: `['a', 'b', 'c', 'd', 'e', 'f']`
- Search path: 'f' → 'fe' → 'fed' → 'fedc' → 'fedcb' → 'fedcba'
- Follow 'f' from root (exists, has `is_end = True`)
- Found complete word "f" → Return `true`
- This means "f" is a suffix of "abcdef"

The key insight: When we traverse the stream backwards and find a complete reversed word in the Trie, it confirms that the original (non-reversed) word is a suffix of our current stream.

## Python Solution

```python
from typing import List


class Trie:
    """A trie data structure that stores words in reverse order for suffix matching."""
  
    def __init__(self):
        # Array to store 26 possible children (one for each lowercase letter)
        self.children = [None] * 26
        # Flag to mark if this node represents the end of a word
        self.is_end = False
  
    def insert(self, word: str) -> None:
        """
        Insert a word into the trie in reverse order.
      
        Args:
            word: The word to insert into the trie
        """
        current_node = self
        # Traverse the word in reverse order
        for char in word[::-1]:
            # Calculate the index for this character (0-25)
            index = ord(char) - ord('a')
            # Create a new node if it doesn't exist
            if current_node.children[index] is None:
                current_node.children[index] = Trie()
            # Move to the child node
            current_node = current_node.children[index]
        # Mark the end of the word
        current_node.is_end = True
  
    def search(self, characters: List[str]) -> bool:
        """
        Search if any suffix of the character list matches a word in the trie.
      
        Args:
            characters: List of characters to search
          
        Returns:
            True if any suffix matches a word in the trie, False otherwise
        """
        current_node = self
        # Traverse the characters in reverse order (checking suffixes)
        for char in characters[::-1]:
            # Calculate the index for this character
            index = ord(char) - ord('a')
            # If no matching path exists, return False
            if current_node.children[index] is None:
                return False
            # Move to the child node
            current_node = current_node.children[index]
            # If we've found a complete word, return True
            if current_node.is_end:
                return True
        return False


class StreamChecker:
    """
    A class to check if any suffix of a character stream matches words from a dictionary.
    """
  
    def __init__(self, words: List[str]):
        """
        Initialize the StreamChecker with a list of words.
      
        Args:
            words: List of words to match against the stream
        """
        # Initialize the trie for storing words
        self.trie = Trie()
        # List to store the stream of characters
        self.character_stream = []
        # Maximum number of recent characters to keep (optimization)
        self.max_length = 201
      
        # Insert all words into the trie
        for word in words:
            self.trie.insert(word)
  
    def query(self, letter: str) -> bool:
        """
        Add a letter to the stream and check if any suffix matches a word.
      
        Args:
            letter: The new character to add to the stream
          
        Returns:
            True if any suffix of the stream matches a word, False otherwise
        """
        # Add the new letter to the stream
        self.character_stream.append(letter)
        # Search only the last max_length characters (optimization for memory)
        return self.trie.search(self.character_stream[-self.max_length:])


# Your StreamChecker object will be instantiated and called as such:
# obj = StreamChecker(words)
# param_1 = obj.query(letter)
```

## Time and Space Complexity

**Time Complexity:**

- **Initialization (`__init__`):** `O(M * L)` where `M` is the number of words and `L` is the average length of each word. Each word insertion into the Trie takes `O(L)` time, and we insert `M` words.

- **Query (`query`):** `O(min(L_max, limit))` where `L_max` is the length of the longest word in the dictionary and `limit = 201`. The search operation traverses at most `min(201, L_max)` characters in reverse order through the Trie. Since we're only searching the last 201 characters (`self.cs[-self.limit:]`), the search is bounded by this constant.

**Space Complexity:**

- **Trie Structure:** `O(M * L * 26)` in the worst case, where `M` is the number of words and `L` is the average length of words. Each Trie node can have up to 26 children (one for each lowercase letter). In practice, the space usage is typically much less due to shared prefixes (or in this case, shared suffixes since words are inserted in reverse).

- **Character Stream Storage (`self.cs`):** `O(Q)` where `Q` is the number of queries made. The list grows with each query call and stores all queried characters.

- **Overall Space Complexity:** `O(M * L * 26 + Q)`, dominated by the Trie structure and the accumulated character stream.

**Note:** The code stores words in reverse order in the Trie and searches from the most recent characters backwards, which allows efficient checking if any word ends at the current position in the stream.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Reverse Words During Insertion**

One of the most common mistakes is inserting words into the Trie in their original order instead of reversed order. This breaks the entire suffix-matching logic.

**Incorrect Implementation:**
```python
def insert(self, word: str) -> None:
    current_node = self
    # WRONG: Iterating in normal order
    for char in word:  
        index = ord(char) - ord('a')
        if current_node.children[index] is None:
            current_node.children[index] = Trie()
        current_node = current_node.children[index]
    current_node.is_end = True
```

**Why This Fails:**
- If we insert "abc" normally, the Trie stores: a → b → c
- When searching for suffix "bc" in stream "abc", we traverse "cb" (reversed)
- The Trie won't find "cb" because it has "abc", not "cba"

**Correct Implementation:**
```python
def insert(self, word: str) -> None:
    current_node = self
    # CORRECT: Reverse the word first
    for char in word[::-1]:  
        index = ord(char) - ord('a')
        if current_node.children[index] is None:
            current_node.children[index] = Trie()
        current_node = current_node.children[index]
    current_node.is_end = True
```

### 2. **Not Limiting the Character Stream Buffer**

Another pitfall is storing the entire stream without any limit, which can cause memory issues for long-running streams.

**Problematic Implementation:**
```python
def query(self, letter: str) -> bool:
    self.character_stream.append(letter)
    # PROBLEM: Searching entire stream history
    return self.trie.search(self.character_stream)
```

**Why This Is Problematic:**
- Memory usage grows unbounded with each query
- Search time increases unnecessarily for very long streams
- Since words have a maximum length (typically 200), checking beyond that is wasteful

**Optimized Solution:**
```python
def query(self, letter: str) -> bool:
    self.character_stream.append(letter)
    # Only search the last 201 characters
    return self.trie.search(self.character_stream[-self.max_length:])
```

### 3. **Incorrect Search Termination Logic**

A subtle bug occurs when the search method doesn't properly check for word endings during traversal.

**Incorrect Implementation:**
```python
def search(self, characters: List[str]) -> bool:
    current_node = self
    for char in characters[::-1]:
        index = ord(char) - ord('a')
        if current_node.children[index] is None:
            return False
        current_node = current_node.children[index]
    # WRONG: Only checking at the end of traversal
    return current_node.is_end  
```

**Why This Fails:**
- This only checks if the entire reversed stream matches a word
- It misses shorter suffixes that might match
- For stream "axyz" with word "xyz", it would check if "zyxa" is a word, missing that "zyx" matches "xyz"

**Correct Implementation:**
```python
def search(self, characters: List[str]) -> bool:
    current_node = self
    for char in characters[::-1]:
        index = ord(char) - ord('a')
        if current_node.children[index] is None:
            return False
        current_node = current_node.children[index]
        # CORRECT: Check for word ending at each step
        if current_node.is_end:
            return True
    return False
```

### 4. **Character Index Calculation Error**

Using incorrect ASCII value calculations can lead to array index out of bounds errors.

**Common Mistake:**
```python
# WRONG: Using 'A' for lowercase letters
index = ord(char) - ord('A')  

# WRONG: Not handling the index calculation at all
index = ord(char)  
```

**Correct Approach:**
```python
# For lowercase letters only
index = ord(char) - ord('a')  # Gives 0-25 for 'a'-'z'
```

These pitfalls highlight the importance of understanding the reversal strategy and careful implementation of both the Trie operations and the stream buffer management.

