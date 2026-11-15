# 140. Word Break II

> Source: https://algo.monster/liteproblems/140

## Problem Statement

You are given a string `s` and a dictionary of strings `wordDict`. Your task is to add spaces to the string `s` to break it into a sequence of valid dictionary words, creating complete sentences. You need to return all possible valid sentences that can be formed.

Key requirements:
- Each word in the resulting sentence must exist in `wordDict`
- The same word from the dictionary can be used multiple times in different positions
- Return all possible valid segmentations of the string
- The sentences can be returned in any order

For example, if `s = "catsanddog"` and `wordDict = ["cat", "cats", "and", "sand", "dog"]`, you could form:
- `"cats and dog"`
- `"cat sand dog"`

The solution uses a Trie data structure to efficiently store and search the dictionary words, combined with a depth-first search (DFS) approach to explore all possible word segmentations. The algorithm:

1. Builds a Trie from all words in `wordDict` for efficient prefix matching
2. Uses DFS to recursively try breaking the string at different positions
3. For each position `i` from 1 to the length of remaining string:
   - Checks if `s[:i]` forms a valid word using the Trie
   - If valid, recursively processes the remaining substring `s[i:]`
   - Combines the current word with all valid segmentations of the remaining string
4. Returns all valid combinations joined with spaces

The base case occurs when the string is empty, returning an empty list within a list `[[]]` to properly combine with previous words in the recursion chain.

## Intuition

When we need to find all possible ways to break a string into valid words, we face a decision-making problem at each position: "Where should I make the next cut?"

Think of it like cutting a piece of rope into segments where each segment must match one of the given patterns. At any point in the string, we have multiple choices - we could cut after 1 character, 2 characters, 3 characters, and so on. Each choice leads to a different remaining substring that needs to be processed.

This naturally suggests a recursive approach: try every possible first word, and for each valid first word, recursively solve the remaining substring. For example, with `"catsanddog"`:
- If we take `"cat"` as the first word, we need to solve `"sanddog"`
- If we take `"cats"` as the first word, we need to solve `"anddog"`

The key insight is that we need to explore ALL valid paths, not just find one solution. This is where backtracking shines - it systematically explores every possibility and collects all valid solutions.

To optimize the word lookup process, we use a Trie data structure. Why? Because as we try different word lengths from a position (checking if `s[0:1]`, `s[0:2]`, `s[0:3]`... are valid words), a Trie can tell us both whether the current prefix is a complete word and whether any dictionary word starts with it. The solution below calls `search` on each prefix separately, which costs about the same as a hash set lookup. Walking the Trie one character at a time from the current position would let the loop stop as soon as no dictionary word has that prefix.

The recursive structure becomes clear:
1. Base case: Empty string means we've successfully segmented the entire string
2. Recursive case: For each valid word starting at the current position, combine it with all valid segmentations of the remaining string
3. The result is naturally built by combining the current word choice with all possible completions

This backtracking approach guarantees we find all valid segmentations by systematically exploring every valid word break possibility.

## Solution Approach

The implementation uses a Trie data structure combined with depth-first search (DFS) backtracking to efficiently find all valid word segmentations.

**Step 1: Build the Trie Structure**

First, we create a Trie class to store all dictionary words:
- Each node has 26 children (for lowercase letters a-z) stored as an array
- `is_end` flag marks the end of a valid word
- The `insert` method adds words character by character, creating nodes as needed
- The `search` method checks if a given string exists as a complete word in the Trie

```python
for w in wordDict:
    trie.insert(w)
```

**Step 2: Implement DFS Backtracking**

The core algorithm is the `dfs` function that recursively explores all segmentations:

```python
def dfs(s):
    if not s:
        return [[]]  # Base case: empty string
    res = []
    for i in range(1, len(s) + 1):
        if trie.search(s[:i]):
            for v in dfs(s[i:]):
                res.append([s[:i]] + v)
    return res
```

The algorithm works as follows:

1. **Base Case**: When the string is empty, return `[[]]` - a list containing an empty list. This allows proper concatenation in the recursive calls.

2. **Recursive Exploration**: For each position `i` from 1 to the length of the string:
   - Extract prefix `s[:i]` and check if it's a valid word using `trie.search()`
   - If valid, recursively process the remaining substring `s[i:]`
   - For each valid segmentation of the remaining string, prepend the current word

3. **Building Results**: Each recursive call returns a list of word lists. We combine the current word with each possible completion:
   - If `s[:i]` is "cat" and `dfs(s[i:])` returns `[["sand", "dog"]]`
   - We create `["cat"] + ["sand", "dog"] = ["cat", "sand", "dog"]`

**Step 3: Format the Final Output**

After getting all valid word combinations as lists of words, we join them with spaces:

```python
ans = dfs(s)
return [' '.join(v) for v in ans]
```

**Time Complexity Analysis**:
- In the worst case, we might have `O(2^n)` possible segmentations (where every position could be a valid break point)
- For each segmentation, we perform Trie searches which take `O(m)` where `m` is the word length
- Overall: `O(2^n * m)` in the worst case

**Space Complexity**:
- Trie storage: `O(total characters in dictionary * 26)`
- Recursion stack: `O(n)` maximum depth
- Result storage: `O(2^n)` for all possible segmentations

The combination of Trie for efficient word lookup and DFS for systematic exploration makes this solution both elegant and effective for finding all valid word break combinations.

## Example Walkthrough

Let's walk through a small example with `s = "catdog"` and `wordDict = ["cat", "dog", "ca", "t"]`.

**Step 1: Build the Trie**
We insert all dictionary words into the Trie:
- Insert "cat": c→a→t (mark t as end)
- Insert "dog": d→o→g (mark g as end)  
- Insert "ca": c→a (mark a as end)
- Insert "t": t (mark t as end)

**Step 2: DFS Exploration from "catdog"**

Starting with `dfs("catdog")`:

- Try prefix of length 1: "c" - Not in Trie ✗
- Try prefix of length 2: "ca" - Valid word! ✓
  - Recursively call `dfs("tdog")`
    - Try "t" - Valid! ✓ → Call `dfs("dog")`
      - Try "d" - Not valid ✗
      - Try "do" - Not valid ✗
      - Try "dog" - Valid! ✓ → Call `dfs("")`
        - Base case returns `[[]]`
      - Returns `[["dog"]]`
    - Combine: "t" + ["dog"] = `[["t", "dog"]]`
    - Try "td", "tdo", "tdog" - All invalid ✗
    - Returns `[["t", "dog"]]`
  - Combine: "ca" + ["t", "dog"] = `[["ca", "t", "dog"]]`

- Try prefix of length 3: "cat" - Valid word! ✓
  - Recursively call `dfs("dog")`
    - Try "d" - Not valid ✗
    - Try "do" - Not valid ✗
    - Try "dog" - Valid! ✓ → Call `dfs("")`
      - Base case returns `[[]]`
    - Returns `[["dog"]]`
  - Combine: "cat" + ["dog"] = `[["cat", "dog"]]`

- Try prefixes of length 4, 5, 6: "catd", "catdo", "catdog" - All invalid ✗

**Step 3: Collect and Format Results**

The DFS returns: `[["ca", "t", "dog"], ["cat", "dog"]]`

After joining with spaces:
- `["ca", "t", "dog"]` → `"ca t dog"`
- `["cat", "dog"]` → `"cat dog"`

**Final Output**: `["ca t dog", "cat dog"]`

The key insight is how the recursion builds solutions bottom-up: the base case returns `[[]]`, which allows each recursive level to properly append its word to form complete segmentations.

## Python Solution

```python
class Trie:
    """Trie (prefix tree) data structure for efficient word lookups"""
  
    def __init__(self):
        # Array to store 26 children nodes (one for each lowercase letter)
        self.children = [None] * 26
        # Flag to mark if current node represents end of a word
        self.is_end = False

    def insert(self, word: str) -> None:
        """Insert a word into the trie"""
        node = self
        for char in word:
            # Calculate index for current character (0-25)
            index = ord(char) - ord('a')
            # Create new node if path doesn't exist
            if node.children[index] is None:
                node.children[index] = Trie()
            # Move to next node
            node = node.children[index]
        # Mark the last node as end of word
        node.is_end = True

    def search(self, word: str) -> bool:
        """Search for a complete word in the trie"""
        node = self
        for char in word:
            # Calculate index for current character
            index = ord(char) - ord('a')
            # Return false if path doesn't exist
            if node.children[index] is None:
                return False
            # Move to next node
            node = node.children[index]
        # Check if current node marks end of a valid word
        return node.is_end


class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> List[str]:
        """
        Break a string into words from dictionary and return all possible sentences.
      
        Args:
            s: Input string to break
            wordDict: List of valid words
          
        Returns:
            List of all possible sentences formed by breaking the string
        """
      
        def dfs(remaining_string: str) -> List[List[str]]:
            """
            Recursively find all possible word combinations for the remaining string.
          
            Args:
                remaining_string: Substring yet to be processed
              
            Returns:
                List of word combinations, where each combination is a list of words
            """
            # Base case: empty string returns empty combination
            if not remaining_string:
                return [[]]
          
            result = []
            # Try all possible prefixes of the remaining string
            for i in range(1, len(remaining_string) + 1):
                prefix = remaining_string[:i]
                # Check if prefix exists in dictionary
                if trie.search(prefix):
                    # Recursively process the rest of the string
                    suffix_combinations = dfs(remaining_string[i:])
                    # Add current prefix to all combinations from suffix
                    for combination in suffix_combinations:
                        result.append([prefix] + combination)
          
            return result
      
        # Build trie from dictionary words for efficient lookup
        trie = Trie()
        for word in wordDict:
            trie.insert(word)
      
        # Find all possible word combinations
        all_combinations = dfs(s)
      
        # Convert each combination list to a space-separated sentence
        return [' '.join(combination) for combination in all_combinations]
```

## Time and Space Complexity

**Time Complexity:** `O(n * 2^n)`

The time complexity analysis breaks down as follows:
- The DFS function explores all possible ways to break the string `s` into words from the dictionary
- In the worst case, at each position `i` in the string, we can either start a new word or continue the current word, leading to `2^n` possible segmentations where `n` is the length of string `s`
- For each recursive call, we iterate through the string to find valid prefixes (up to `n` iterations)
- The trie search operation takes `O(m)` time where `m` is the length of the word being searched, but since we're searching prefixes of `s`, this is bounded by `O(n)`
- Building the final result requires joining the words, which in the worst case involves copying all `2^n` valid segmentations
- Therefore, the overall time complexity is `O(n * 2^n)`

**Space Complexity:** `O(n * 2^n + m * k)`

The space complexity consists of several components:
- **Trie storage:** `O(m * k)` where `m` is the total number of characters across all words in `wordDict` and `k` is the size of the alphabet (26 for lowercase English letters)
- **Recursion stack:** `O(n)` in the worst case when the recursion depth equals the length of the string
- **Result storage:** In the worst case, we can have `2^n` valid segmentations, and each segmentation contains words whose total length is `n`, resulting in `O(n * 2^n)` space for storing all results
- The dominant term is `O(n * 2^n)` when considering large inputs where the exponential factor outweighs the trie storage
- Therefore, the overall space complexity is `O(n * 2^n + m * k)`

## Common Pitfalls

### 1. **Exponential Time Complexity Without Memoization**

The most significant pitfall in this solution is the lack of memoization, leading to redundant computations. When the same substring appears multiple times during recursion, we recalculate all its possible segmentations repeatedly.

**Example Problem:**
Consider `s = "aaaaaaa"` with `wordDict = ["a", "aa", "aaa"]`. The remaining substring "aaaa" is reached through multiple paths:
- "a" + "a" + "a" + "aaaa"
- "aa" + "a" + "aaaa"
- "a" + "aa" + "aaaa"
- etc.

Each time we encounter "aaaa", we recalculate all its possible breaks, causing exponential time complexity.

**Solution - Add Memoization:**

```python
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> List[str]:
        def dfs(remaining_string: str) -> List[List[str]]:
            # Check memo first
            if remaining_string in memo:
                return memo[remaining_string]
          
            if not remaining_string:
                return [[]]
          
            result = []
            for i in range(1, len(remaining_string) + 1):
                prefix = remaining_string[:i]
                if trie.search(prefix):
                    suffix_combinations = dfs(remaining_string[i:])
                    for combination in suffix_combinations:
                        result.append([prefix] + combination)
          
            # Store result in memo before returning
            memo[remaining_string] = result
            return result
      
        # Initialize memoization dictionary
        memo = {}
      
        trie = Trie()
        for word in wordDict:
            trie.insert(word)
      
        all_combinations = dfs(s)
        return [' '.join(combination) for combination in all_combinations]
```

### 2. **Memory Issues with Large Result Sets**

When there are many possible segmentations, storing all combinations in memory can cause memory overflow. For instance, a string with many overlapping valid words can generate an exponential number of valid sentences.

**Solution - Use Generator for Memory Efficiency:**

```python
def dfs_generator(remaining_string: str):
    """Yield combinations one at a time instead of storing all"""
    if not remaining_string:
        yield []
        return
  
    for i in range(1, len(remaining_string) + 1):
        prefix = remaining_string[:i]
        if trie.search(prefix):
            for combination in dfs_generator(remaining_string[i:]):
                yield [prefix] + combination
```

### 3. **Inefficient String Slicing**

Creating new string slices `s[:i]` and `s[i:]` at each recursive call creates many temporary string objects, impacting performance.

**Solution - Use Indices Instead of String Slicing:**

```python
def dfs(start_index: int) -> List[List[str]]:
    if start_index == len(s):
        return [[]]
  
    if start_index in memo:
        return memo[start_index]
  
    result = []
    current_node = trie
  
    for end_index in range(start_index, len(s)):
        char = s[end_index]
        index = ord(char) - ord('a')
      
        if current_node.children[index] is None:
            break  # No more valid prefixes possible
      
        current_node = current_node.children[index]
      
        if current_node.is_end:
            word = s[start_index:end_index + 1]
            for combination in dfs(end_index + 1):
                result.append([word] + combination)
  
    memo[start_index] = result
    return result
```

### 4. **Not Pre-checking if Solution Exists**

The algorithm might waste time exploring impossible cases. If certain characters in `s` don't appear in any dictionary word, we can fail fast.

**Solution - Add Pre-validation:**

```python
def wordBreak(self, s: str, wordDict: List[str]) -> List[str]:
    # Quick check: all characters in s must exist in some word
    chars_in_dict = set(''.join(wordDict))
    chars_in_s = set(s)
  
    if not chars_in_s.issubset(chars_in_dict):
        return []  # Impossible to break the string
  
    # Continue with main algorithm...
```

These changes remove repeated work, which matters most on inputs with many shared suffixes. They cannot make the worst case polynomial: when the number of valid sentences is exponential, just writing out the answer takes exponential time and space.

