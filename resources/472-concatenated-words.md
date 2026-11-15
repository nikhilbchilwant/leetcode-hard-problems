# 472. Concatenated Words

> Source: https://algo.monster/liteproblems/472

## Problem Statement

You are given an array of strings `words` that contains no duplicates. Your task is to find all the **concatenated words** in this array.

A **concatenated word** is a string that can be formed by combining at least two other shorter words from the same array. The shorter words used to form the concatenated word don't need to be distinct - you can use the same word multiple times.

For example:
- If `words = ["cat", "cats", "dog", "catsdog"]`, then `"catsdog"` is a concatenated word because it can be formed by combining `"cats"` and `"dog"`.
- A word like `"cat"` by itself is not a concatenated word since it cannot be broken down into at least two other words from the array.

The solution uses a Trie data structure combined with dynamic programming. Words are sorted by length and processed from shortest to longest. For each word, the algorithm checks if it can be formed by concatenating existing shorter words in the Trie. If a word can be concatenated, it's added to the result list; otherwise, it's inserted into the Trie for potential use in forming longer words.

The `dfs` function recursively checks if a word can be split into valid concatenations by traversing the Trie and looking for complete words (marked by `is_end = True`) at each position, then recursively checking if the remaining portion can also be formed from words in the Trie.

## Intuition

The key insight is that we need to check if a word can be built from other words in our dictionary. Think of it like building with LEGO blocks - can we construct a bigger piece using smaller pieces we already have?

When we look at a word like `"catsdog"`, we need to check if we can split it into valid words from our array. We could try `"c" + "atsdog"`, `"ca" + "tsdog"`, `"cat" + "sdog"`, `"cats" + "dog"`, and so on. If we find that `"cats"` is a valid word in our array, we then need to check if `"dog"` can also be formed from words in our array.

This naturally leads to a recursive approach: at each position in the word, we check if the substring from the start to that position forms a valid word. If it does, we recursively check if the remaining part can also be formed from valid words.

But how do we efficiently check if a substring is a valid word? Scanning through the entire word array each time would be inefficient. This is where the Trie comes in - it allows us to check character by character if we're forming a valid word, and know immediately when we've found a complete word (marked by `is_end`).

The clever part of the solution is processing words by length. We start with the shortest words and add them to our Trie. When we check longer words, all potential building blocks (shorter words) are already in the Trie. If a word can be concatenated from others, we add it to our result; if not, it becomes a potential building block for even longer words, so we add it to the Trie.

This approach ensures that when we check if `"catsdog"` is a concatenated word, both `"cats"` and `"dog"` are already in our Trie, making the check possible through our DFS traversal.

## Solution Approach

The solution implements a Trie-based approach with DFS to identify concatenated words. Here's how the implementation works:

**1. Trie Data Structure Setup**

First, we define a Trie class to efficiently store and search for words:
- Each Trie node has an array `children` of size 26 (for lowercase letters a-z)
- A boolean flag `is_end` marks the end of a complete word
- The `insert` method adds words to the Trie by creating nodes for each character

**2. Sorting Words by Length**

The algorithm sorts the input words by length: `words.sort(key=lambda x: len(x))`. This is crucial because:
- Shorter words cannot be concatenated from longer words
- We build our Trie incrementally with shorter words first
- When checking if a longer word is concatenated, all its potential components are already in the Trie

**3. DFS Function for Validation**

The `dfs` function checks if a word can be formed by concatenating other words:
```python
def dfs(w):
    if not w:
        return True  # Empty string means we've successfully split the entire word
```

The function traverses the Trie character by character:
- For each character, it moves to the corresponding child node
- If no child exists, the current prefix isn't a valid word, so return `False`
- When it finds a complete word (`node.is_end = True`), it recursively checks if the remaining substring can also be formed from words in the Trie

**4. Main Processing Loop**

For each word in the sorted list:
```python
for w in words:
    if dfs(w):
        ans.append(w)  # Word can be concatenated, add to result
    else:
        trie.insert(w)  # Word cannot be concatenated, add it as a building block
```

**5. Key Implementation Detail**

The recursive call `dfs(w[i + 1:])` in the DFS function is critical:
- When we find a valid word at position `i`, we check if the substring from `i+1` onwards can also be formed
- This ensures the word is split into at least two parts (the found word and the remaining substring)
- The recursion naturally handles cases where a word can be split into more than two parts

The algorithm's efficiency comes from:
- Using a Trie for O(1) character lookups instead of scanning the word list
- Processing words in length order to ensure all potential components are available
- Memoization through the Trie structure itself (once a word is added, it's instantly available for all future checks)

## Example Walkthrough

Let's walk through a small example with `words = ["cat", "cats", "dog", "catsdog"]`.

**Step 1: Sort by length**
After sorting: `["cat", "dog", "cats", "catsdog"]`

**Step 2: Process "cat" (length 3)**
- Call `dfs("cat")` with empty Trie
- Start traversing: 'c' → no child exists in empty Trie
- Return `False` (cannot be formed from other words)
- Since DFS returns `False`, insert "cat" into Trie

Trie now contains: "cat"

**Step 3: Process "dog" (length 3)**
- Call `dfs("dog")`
- Start traversing: 'd' → no child for 'd' at root
- Return `False`
- Insert "dog" into Trie

Trie now contains: "cat", "dog"

**Step 4: Process "cats" (length 4)**
- Call `dfs("cats")`
- Traverse: 'c' → 'a' → 't' (found `is_end=True` at index 2)
- Found "cat" is a complete word, recursively call `dfs("s")`
- In `dfs("s")`: 's' → no child for 's' at root
- Return `False`
- Continue to 's': the path "cats" does not exist yet, so `dfs("cats")` returns `False`
- Insert "cats" into Trie

Trie now contains: "cat", "cats", "dog"

**Step 5: Process "catsdog" (length 7)**
- Call `dfs("catsdog")`
- Traverse: 'c' → 'a' → 't' (found `is_end=True` at index 2)
  - Found "cat", recursively call `dfs("sdog")`
  - In `dfs("sdog")`: 's' → no child at root, return `False`
- Continue traversing: 's' (found `is_end=True` at index 3)
  - Found "cats", recursively call `dfs("dog")`
  - In `dfs("dog")`: traverse 'd' → 'o' → 'g' (found `is_end=True` at index 2)
    - Found complete word "dog", recursively call `dfs("")`
    - Empty string returns `True`
  - `dfs("dog")` returns `True`
- `dfs("catsdog")` returns `True`
- Add "catsdog" to result list

**Final Result: `["catsdog"]`**

## Python Solution

```python
class Trie:
    """Trie (prefix tree) data structure for efficient string storage and retrieval."""
  
    def __init__(self):
        # Array to store 26 children nodes (one for each lowercase letter)
        self.children = [None] * 26
        # Flag to mark if current node represents end of a word
        self.is_end = False

    def insert(self, word: str) -> None:
        """Insert a word into the trie."""
        node = self
        for char in word:
            # Calculate index for the character (0-25)
            index = ord(char) - ord('a')
            # Create new node if path doesn't exist
            if node.children[index] is None:
                node.children[index] = Trie()
            # Move to the child node
            node = node.children[index]
        # Mark the last node as end of word
        node.is_end = True


class Solution:
    def findAllConcatenatedWordsInADict(self, words: List[str]) -> List[str]:
        """
        Find all concatenated words in a dictionary.
        A concatenated word is formed by combining two or more shorter words from the same dictionary.
        """
      
        def can_form_by_concatenation(word: str) -> bool:
            """
            Check if a word can be formed by concatenating other words in the trie.
            Uses DFS to try all possible splits of the word.
            """
            # Empty string means we successfully split the entire word
            if not word:
                return True
          
            node = trie
            # Try to match prefixes of the word
            for i, char in enumerate(word):
                index = ord(char) - ord('a')
                # If no matching path exists, word cannot be formed
                if node.children[index] is None:
                    return False
              
                node = node.children[index]
                # If we found a complete word, try to match the remaining part
                if node.is_end and can_form_by_concatenation(word[i + 1:]):
                    return True
          
            return False

        # Initialize trie and result list
        trie = Trie()
        result = []
      
        # Sort words by length to ensure shorter words are added to trie first
        # This allows longer words to be formed from shorter ones
        words.sort(key=lambda word: len(word))
      
        for word in words:
            # Check if current word can be formed by concatenating existing words
            if can_form_by_concatenation(word):
                result.append(word)
            else:
                # If not, add it to trie for future concatenations
                trie.insert(word)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(n * m^2)` where `n` is the number of words and `m` is the average length of words.

- Sorting the words takes `O(n log n)` time
- For each word, we perform a DFS traversal:
  - In the worst case, for each character position `i` in a word of length `m`, we might check if it forms a valid word (taking `O(m)` time to traverse the trie)
  - Then recursively call DFS on the remaining substring
  - This can lead to checking all possible split positions, resulting in `O(m^2)` time per word in the worst case
- The trie insertion for non-concatenated words takes `O(m)` time per word
- Overall: `O(n log n + n * m^2)` = `O(n * m^2)` (since typically `m^2 > log n`)

**Space Complexity:** `O(n * m * 26)` which simplifies to `O(n * m)`

- The Trie structure stores all words:
  - In the worst case, if all words are completely different, we need `O(n * m)` nodes
  - Each node contains an array of 26 pointers, so technically `O(n * m * 26)`
  - Since 26 is a constant, this simplifies to `O(n * m)`
- The recursion stack for DFS can go up to `O(m)` depth in the worst case
- The result list stores at most `O(n)` words
- Overall: `O(n * m)` for the trie structure dominates the space complexity

## Common Pitfalls

### 1. **Forgetting to Handle Empty Words**
The algorithm assumes all words have at least one character. If the input contains empty strings, the code will add them to the Trie and potentially cause incorrect results when checking concatenations.

**Solution:** Filter out empty strings before processing:
```python
words = [w for w in words if w]  # Remove empty strings
```

### 2. **Incorrect Base Case - Accepting Single Word as Valid**
A critical pitfall is not ensuring that a word is formed from **at least two** other words. The current implementation correctly handles this, but a common mistake would be to check if `node.is_end` is True at the end of the DFS without ensuring we've made at least one split:

```python
# WRONG: This would accept single words as "concatenated"
def can_form_by_concatenation(word: str, split_count=0) -> bool:
    if not word:
        return split_count > 0  # Should be split_count >= 2
```

**Solution:** The current implementation correctly avoids this by only adding words to the result if DFS returns True (meaning a valid split was found), and never inserting a word that can be formed by concatenation back into the Trie.

### 3. **Stack Overflow with Deep Recursion**
For very long words or deeply nested concatenations, the recursive DFS might hit Python's recursion limit. For example, if you have many single-character words and a very long concatenated word.

**Solution:** Use iterative DFS with explicit stack or add memoization:
```python
def can_form_by_concatenation(word: str) -> bool:
    memo = {}
  
    def dfs(start_idx):
        if start_idx == len(word):
            return True
        if start_idx in memo:
            return memo[start_idx]
      
        node = trie
        for i in range(start_idx, len(word)):
            index = ord(word[i]) - ord('a')
            if node.children[index] is None:
                break
            node = node.children[index]
            if node.is_end and dfs(i + 1):
                memo[start_idx] = True
                return True
      
        memo[start_idx] = False
        return False
  
    return dfs(0)
```

### 4. **Performance Degradation with Duplicate Substrings**
The current implementation creates new string slices (`word[i + 1:]`) in each recursive call, which can be inefficient for long words with many possible splits. Each slice creates a new string object.

**Solution:** Pass indices instead of creating substrings:
```python
def can_form_by_concatenation(word: str, start=0) -> bool:
    if start == len(word):
        return True
  
    node = trie
    for i in range(start, len(word)):
        index = ord(word[i]) - ord('a')
        if node.children[index] is None:
            return False
        node = node.children[index]
        if node.is_end and can_form_by_concatenation(word, i + 1):
            return True
  
    return False
```

### 5. **Not Handling Case Sensitivity**
The current implementation assumes all input words contain only lowercase letters. Mixed case inputs will cause index out of bounds errors.

**Solution:** Normalize input or add validation:
```python
# Option 1: Normalize
words = [w.lower() for w in words]

# Option 2: Validate
for word in words:
    if not word.islower():
        raise ValueError(f"Word '{word}' contains non-lowercase characters")
```

