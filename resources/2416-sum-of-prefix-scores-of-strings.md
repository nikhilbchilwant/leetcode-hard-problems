# 2416. Sum of Prefix Scores of Strings

> Source: https://algo.monster/liteproblems/2416

## Problem Statement

You are given an array `words` containing `n` non-empty strings. Your task is to calculate the prefix scores for each word in the array.

The **score** of a string `term` is defined as the count of how many strings in `words` have `term` as their prefix. A string `term` is a prefix of another string if the other string starts with `term`.

For example, given `words = ["a", "ab", "abc", "cab"]`:
- The score of `"ab"` is `2` because it's a prefix of both `"ab"` and `"abc"`
- The score of `"a"` is `3` because it's a prefix of `"a"`, `"ab"`, and `"abc"`
- The score of `"ca"` is `1` because it's only a prefix of `"cab"`

For each word in the array, you need to calculate the sum of scores for all its non-empty prefixes. Remember that a string is considered a prefix of itself.

For instance, if we have the word `"abc"`:
- Its prefixes are: `"a"`, `"ab"`, `"abc"`
- You would calculate: score(`"a"`) + score(`"ab"`) + score(`"abc"`)

The output should be an array `answer` of size `n`, where `answer[i]` represents the sum of scores of all non-empty prefixes of `words[i]`.

## Intuition

The key insight is recognizing that we need to count how many times each prefix appears across all words. If we were to use a brute force approach, for each word, we'd generate all its prefixes and then scan through all words to count matches. This would be inefficient with a time complexity of O(n² × m²) where n is the number of words and m is the average length.

The pattern here suggests using a [Trie](/problems/trie_intro) (prefix tree) because:
1. We're dealing with prefixes, which is exactly what Tries are designed for
2. We need to count occurrences of each prefix efficiently
3. Multiple words may share common prefixes, and we can avoid redundant computations

Think about how we process the word `"abc"` - we need to know how many words start with `"a"`, how many start with `"ab"`, and how many start with `"abc"`. If we build a [Trie](/problems/trie_intro) from all words first, each node in the path from root to a letter represents a prefix, and we can store a counter at each node.

When we insert a word like `"abc"` into the [Trie](/problems/trie_intro):
- We traverse through `'a'` → `'ab'` → `'abc'`
- At each node along this path, we increment a counter
- This counter tells us exactly how many words have passed through this node (i.e., how many words have this prefix)

Later, when we search for the sum of prefix scores for `"abc"`:
- We traverse the same path `'a'` → `'ab'` → `'abc'`
- At each node, we add its counter value to our sum
- The sum of all these counters gives us the total prefix score

This approach reduces our time complexity to O(n × m) for building the [Trie](/problems/trie_intro) and O(n × m) for querying, making it much more efficient than the brute force approach.

**Learn more about [Trie](/problems/trie_intro) patterns.**

## Solution Approach

The solution uses a [Trie](/problems/trie_intro) data structure to efficiently store and count all prefixes. Let's walk through the implementation step by step:

### [Trie](/problems/trie_intro) Node Structure

The `[Trie](/problems/trie_intro)` class has two key attributes:
- `children`: An array of size 26 (for lowercase English letters a-z), where `children[i]` points to the child node for character `'a' + i`
- `cnt`: A counter that tracks how many words pass through this node (i.e., how many words have the prefix represented by the path from root to this node)

### Insert Method

The `insert` method adds a word to the [Trie](/problems/trie_intro):
1. Start from the root node
2. For each character `c` in the word:
   - Calculate the index: `idx = ord(c) - ord("a")` (converts character to 0-25 range)
   - If no child exists at this index, create a new Trie node
   - Move to the child node
   - Increment the counter `cnt` at this node by 1

For example, inserting `"abc"`:
- At node for `'a'`: increment `cnt` (counts words with prefix `"a"`)
- At node for `'b'` (after `'a'`): increment `cnt` (counts words with prefix `"ab"`)
- At node for `'c'` (after `'ab'`): increment `cnt` (counts words with prefix `"abc"`)

### Search Method

The `search` method calculates the sum of prefix scores for a word:
1. Start from the root node with `ans = 0`
2. For each character `c` in the word:
   - Calculate the index: `idx = ord(c) - ord("a")`
   - If no child exists at this index, return the current sum (no more prefixes exist)
   - Move to the child node
   - Add the node's `cnt` value to `ans`
3. Return the total sum

### Main Solution

The `sumPrefixScores` method orchestrates the process:
1. Create an empty [Trie](/problems/trie_intro)
2. Insert all words into the Trie (this builds up the counter values at each node)
3. For each word, call `search` to get the sum of its prefix scores
4. Return the list of sums

### Time and Space Complexity

- **Time Complexity**: O(n × m), where n is the number of words and m is the average length of words
  - Inserting all words: O(n × m)
  - Searching all words: O(n × m)

- **Space Complexity**: O(n × m) in the worst case, for storing all unique prefixes in the [Trie](/problems/trie_intro)

The `__slots__` optimization in Python is used to reduce memory overhead by preventing the creation of `__dict__` for each instance, making the [Trie](/problems/trie_intro) more memory-efficient.

## Example Walkthrough

Let's walk through a small example with `words = ["ab", "abc", "a"]` to illustrate how the Trie solution works.

**Step 1: Build the Trie by inserting all words**

Insert "ab":
```
Root → a (cnt=1) → b (cnt=1)
```

Insert "abc":
```
Root → a (cnt=2) → b (cnt=2) → c (cnt=1)
```
- When inserting "abc", we traverse through 'a' (increment cnt from 1 to 2), then 'b' (increment cnt from 1 to 2), then create and visit 'c' (cnt=1)

Insert "a":
```
Root → a (cnt=3) → b (cnt=2) → c (cnt=1)
```
- When inserting "a", we only visit node 'a' and increment its cnt from 2 to 3

**Step 2: Calculate prefix scores for each word**

For word "ab":
- Visit 'a': add cnt=3 to sum (3 words have prefix "a": "ab", "abc", "a")
- Visit 'b': add cnt=2 to sum (2 words have prefix "ab": "ab", "abc")
- Total: 3 + 2 = 5

For word "abc":
- Visit 'a': add cnt=3 to sum
- Visit 'b': add cnt=2 to sum  
- Visit 'c': add cnt=1 to sum (1 word has prefix "abc": "abc")
- Total: 3 + 2 + 1 = 6

For word "a":
- Visit 'a': add cnt=3 to sum
- Total: 3

**Final Answer: [5, 6, 3]**

The key insight is that each node's counter tells us exactly how many words in our array have the prefix represented by the path from root to that node. By summing these counters along the path of each word, we get the total prefix score efficiently.

## Python Solution

```python
from typing import List, Optional


class Trie:
    """
    A Trie (prefix tree) data structure for efficient prefix operations.
    Each node stores the count of words passing through it.
    """
    __slots__ = ("children", "count")
  
    def __init__(self):
        """Initialize a Trie node with 26 children (for lowercase letters) and count of 0."""
        self.children: List[Optional['Trie']] = [None] * 26  # Array for 26 lowercase letters
        self.count: int = 0  # Number of words passing through this node
  
    def insert(self, word: str) -> None:
        """
        Insert a word into the Trie and increment count for each node in the path.
      
        Args:
            word: The word to insert into the Trie
        """
        current_node = self
      
        for character in word:
            # Calculate the index for this character (0-25 for 'a'-'z')
            char_index = ord(character) - ord('a')
          
            # Create a new node if it doesn't exist
            if current_node.children[char_index] is None:
                current_node.children[char_index] = Trie()
          
            # Move to the child node
            current_node = current_node.children[char_index]
            # Increment the count of words passing through this node
            current_node.count += 1
  
    def search(self, word: str) -> int:
        """
        Calculate the sum of prefix scores for a word.
        The prefix score is the sum of counts for all prefixes of the word.
      
        Args:
            word: The word to calculate prefix scores for
          
        Returns:
            The sum of all prefix scores
        """
        current_node = self
        total_score = 0
      
        for character in word:
            # Calculate the index for this character
            char_index = ord(character) - ord('a')
          
            # If the path doesn't exist, return current score
            if current_node.children[char_index] is None:
                return total_score
          
            # Move to the child node
            current_node = current_node.children[char_index]
            # Add the count of words with this prefix to the total score
            total_score += current_node.count
      
        return total_score


class Solution:
    """
    Solution class for calculating prefix scores of words.
    For each word, the prefix score is the sum of counts of all its prefixes
    across all words in the list.
    """
  
    def sumPrefixScores(self, words: List[str]) -> List[int]:
        """
        Calculate the sum of prefix scores for each word in the list.
      
        Args:
            words: List of words to process
          
        Returns:
            List of prefix scores corresponding to each word
        """
        # Build the Trie with all words
        trie = Trie()
        for word in words:
            trie.insert(word)
      
        # Calculate and return prefix scores for each word
        return [trie.search(word) for word in words]
```

## Time and Space Complexity

**Time Complexity:** `O(L)` where `L` is the total length of all strings in the input array.

- The `insert` method iterates through each character of a word, performing constant-time operations for each character. For a single word of length `m`, this takes `O(m)` time.
- Inserting all words takes `O(L)` time in total, where `L = sum of lengths of all words`.
- The `search` method also iterates through each character of a word, performing constant-time operations for each character. For a single word of length `m`, this takes `O(m)` time.
- Searching all words takes `O(L)` time in total.
- Overall time complexity: `O(L) + O(L) = O(L)`.

**Space Complexity:** `O(L)` where `L` is the total length of all strings in the input array.

- Each character in the input can potentially create a new Trie node.
- In the worst case, when all words have completely different prefixes, we create a new node for each character across all words.
- Each Trie node stores an array of 26 pointers (for 26 lowercase letters) and a counter variable.
- The maximum number of nodes created is bounded by the total number of characters `L`.
- Therefore, the space complexity is `O(L × 26) = O(L)` (since 26 is a constant).

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Count the Word Itself as Its Own Prefix**

A critical mistake is not recognizing that every word is a prefix of itself. Some implementations might stop counting before the last character or forget to include the full word in the prefix score calculation.

**Incorrect Implementation Example:**
```python
def search(self, word: str) -> int:
    current_node = self
    total_score = 0
  
    # Wrong: stops before the last character
    for i in range(len(word) - 1):  # Missing the last character!
        char_index = ord(word[i]) - ord('a')
        if current_node.children[char_index] is None:
            return total_score
        current_node = current_node.children[char_index]
        total_score += current_node.count
  
    return total_score
```

**Solution:** Ensure the loop iterates through ALL characters of the word, including the last one.

### 2. **Not Handling Empty Strings or Edge Cases**

While the problem states "non-empty strings," defensive programming suggests checking for edge cases.

**Potential Issue:**
```python
# If words could be empty or None
words = ["", "a", "ab"]  # Empty string could cause issues
```

**Solution:** Add validation if needed:
```python
def sumPrefixScores(self, words: List[str]) -> List[int]:
    # Filter out empty strings if they might exist
    words = [w for w in words if w]
  
    trie = Trie()
    for word in words:
        trie.insert(word)
  
    return [trie.search(word) for word in words]
```

### 3. **Memory Inefficiency - Creating All 26 Children Immediately**

The current implementation creates an array of 26 None values for every node, even if most won't be used. This can waste significant memory for sparse tries.

**Memory-Efficient Alternative:**
```python
class Trie:
    def __init__(self):
        self.children = {}  # Use dictionary instead of array
        self.count = 0
  
    def insert(self, word: str) -> None:
        current_node = self
      
        for character in word:
            if character not in current_node.children:
                current_node.children[character] = Trie()
          
            current_node = current_node.children[character]
            current_node.count += 1
  
    def search(self, word: str) -> int:
        current_node = self
        total_score = 0
      
        for character in word:
            if character not in current_node.children:
                return total_score
          
            current_node = current_node.children[character]
            total_score += current_node.count
      
        return total_score
```

### 4. **Incorrect Count Placement**

A common mistake is incrementing the count at the wrong node level or forgetting to increment it during insertion.

**Incorrect Example:**
```python
def insert(self, word: str) -> None:
    current_node = self
  
    for character in word:
        char_index = ord(character) - ord('a')
      
        if current_node.children[char_index] is None:
            current_node.children[char_index] = Trie()
      
        # Wrong: incrementing before moving to child
        current_node.count += 1  # This counts at the parent level!
        current_node = current_node.children[char_index]
```

**Solution:** Always increment the count AFTER moving to the child node, as shown in the correct implementation.

### 5. **Character Range Assumptions**

The code assumes all characters are lowercase letters ('a' to 'z'). If the input contains uppercase letters, numbers, or special characters, the program will crash or produce incorrect results.

**Problem Example:**
```python
words = ["ABC", "123", "a-b"]  # Will cause IndexError or incorrect behavior
```

**Solution:** Either validate input or handle a broader character range:
```python
def insert(self, word: str) -> None:
    current_node = self
  
    for character in word.lower():  # Convert to lowercase
        if not character.isalpha():  # Skip non-alphabetic characters
            continue
      
        char_index = ord(character) - ord('a')
        # ... rest of the code
```

