# 3045. Count Prefix and Suffix Pairs II

> Source: https://algo.monster/liteproblems/3045

## Problem Statement

You are given a 0-indexed array of strings called `words`.

The problem asks you to find pairs of strings where one string is both a prefix AND a suffix of another string.

Specifically, you need to count how many pairs of indices `(i, j)` exist where:
- `i < j` (the first index must be smaller than the second)
- `words[i]` is both a prefix and a suffix of `words[j]`

For example:
- `"aba"` is both a prefix and suffix of `"ababa"` (starts with "aba" and ends with "aba") → this would count as a valid pair
- `"abc"` is only a prefix of `"abcd"` but not a suffix → this would NOT count

The solution uses a [Trie](/problems/trie_intro) data structure with a clever approach: instead of storing single characters, it stores character pairs. For each string `s`, it creates pairs by combining the character at position `i` from the start with the character at position `i` from the end: `(s[i], s[m-i-1])`. 

When a string is both a prefix and suffix of another string, their character pairs will match in this special pairing scheme. The algorithm:
1. For each word, creates these character pairs
2. Traverses the [trie](/problems/trie_intro) following these pairs
3. At each node, adds the count of strings that have been inserted at that position (these represent valid prefix-suffix matches)
4. After traversing, increments the count at the final node

The time complexity is `O(n * m)` where `n` is the number of words and `m` is the average length of words.

## Intuition

The key insight is recognizing that when a string is both a prefix and suffix of another string, there's a special symmetry we can exploit.

Let's think about what it means for string `A` to be both a prefix and suffix of string `B`:
- `A` must match the beginning of `B`
- `A` must also match the end of `B`

For example, if `A = "aba"` and `B = "ababa"`:
- The first 3 characters of `B` are `"aba"` (prefix match)
- The last 3 characters of `B` are `"aba"` (suffix match)

Now here's the clever observation: if we look at the characters of `A` from both ends simultaneously, they must match the corresponding positions in `B` from both ends. Specifically:
- The 1st character of `A` must equal the 1st character of `B` AND
- The last character of `A` must equal the last character of `B`
- The 2nd character of `A` must equal the 2nd character of `B` AND
- The 2nd-to-last character of `A` must equal the 2nd-to-last character of `B`
- And so on...

This gives us the idea to represent each string as a sequence of character pairs: `(first_char, last_char)`, `(second_char, second_to_last_char)`, etc.

By storing these pairs in a [trie](/problems/trie_intro) structure, we can efficiently check if one string's pair sequence is a prefix of another string's pair sequence. When string `A`'s pair sequence is a prefix of string `B`'s pair sequence, it means `A` is both a prefix and suffix of `B`.

The beauty of this approach is that we only need to traverse each string once, creating these pairs and checking against previously inserted strings in the [trie](/problems/trie_intro). Each time we find a match during traversal, we know we've found a valid prefix-suffix pair.

**Learn more about [Trie](/problems/trie_intro) patterns.**

## Solution Approach

The solution uses a **[Trie](/problems/trie_intro) data structure** to efficiently store and search for character pair patterns.

**Data Structure Setup:**
- We define a `Node` class with two attributes:
  - `children`: a dictionary to store child nodes, where keys are character pairs `(char_from_start, char_from_end)`
  - `cnt`: a counter to track how many strings end at this node

**Algorithm Steps:**

1. **Initialize** an empty [trie](/problems/trie_intro) and a counter `ans = 0`

2. **Process each word** in the array:
   - Start from the root of the [trie](/problems/trie_intro)
   - Create character pairs using `zip(s, reversed(s))`:
     - This pairs up `s[0]` with `s[n-1]`, `s[1]` with `s[n-2]`, and so on
     - For example, `"aba"` becomes pairs: `('a','a')`, `('b','b')`, `('a','a')`

3. **Traverse and build the [trie](/problems/trie_intro)** for each character pair:
   - If the pair doesn't exist as a child, create a new node
   - Move to the child node corresponding to this pair
   - Add the current node's `cnt` to our answer (this represents all previously inserted strings that match up to this point)

4. **Mark the end** of the current string:
   - Increment `node.cnt` by 1 at the final position
   - This indicates that a complete string ends here

**Why this works:**
- When we traverse with a new string and encounter nodes with `cnt > 0`, it means there are shorter strings that are both prefixes and suffixes of the current string
- The pairing mechanism `(s[i], s[m-i-1])` ensures that we're checking both prefix and suffix conditions simultaneously
- By accumulating the counts during traversal, we automatically count all valid `(i, j)` pairs where `i < j`

**Time Complexity:** `O(n * m)` where `n` is the number of words and `m` is the average length of each word.

**Space Complexity:** `O(n * m)` for storing the [trie](/problems/trie_intro) nodes.

## Example Walkthrough

Let's walk through the solution with `words = ["a", "aba", "ababa", "aa"]`.

**Step 1: Process "a"**
- Create pairs: `zip("a", "a")` → `[('a', 'a')]`
- Start at root, create child for `('a', 'a')`
- No existing nodes, so `ans = 0`
- Mark end: set `cnt = 1` at this node

Trie state:
```
root → ('a','a')[cnt=1]
```

**Step 2: Process "aba"**
- Create pairs: `zip("aba", "aba")` → `[('a','a'), ('b','b'), ('a','a')]`
- Traverse from root:
  - Move to `('a','a')` child (exists), add its `cnt=1` to answer → `ans = 1`
  - Create new child `('b','b')` with `cnt=0`
  - Create new child `('a','a')` with `cnt=0`
- Mark end: set `cnt = 1` at final node

Trie state:
```
root → ('a','a')[cnt=1] → ('b','b')[cnt=0] → ('a','a')[cnt=1]
```

Found: "a" is both prefix and suffix of "aba" (pair (0,1))

**Step 3: Process "ababa"**
- Create pairs: `zip("ababa", "ababa")` → `[('a','a'), ('b','b'), ('a','a'), ('b','b'), ('a','a')]`
- Traverse from root:
  - Move to `('a','a')`, add `cnt=1` → `ans = 2`
  - Move to `('b','b')`, add `cnt=0` → `ans = 2`
  - Move to `('a','a')`, add `cnt=1` → `ans = 3`
  - Create new child `('b','b')` with `cnt=0`
  - Create new child `('a','a')` with `cnt=0`
- Mark end: set `cnt = 1` at final node

Found: "a" and "aba" are both prefix and suffix of "ababa" (pairs (0,2) and (1,2))

**Step 4: Process "aa"**
- Create pairs: `zip("aa", "aa")` → `[('a','a'), ('a','a')]`
- Traverse from root:
  - Move to `('a','a')`, add `cnt=1` → `ans = 4`
  - Try to move to another `('a','a')` child, but it doesn't exist from this node
  - Create new child `('a','a')` with `cnt=0`
- Mark end: set `cnt = 1` at final node

Found: "a" is both prefix and suffix of "aa" (pair (0,3))

**Final answer: 4** (pairs: (0,1), (0,2), (1,2), (0,3))

## Python Solution

```python
class TrieNode:
    """
    A node in the Trie data structure.
    Uses __slots__ for memory optimization.
    """
    __slots__ = ["children", "count"]
  
    def __init__(self):
        # Dictionary to store child nodes, keyed by (prefix_char, suffix_char) tuples
        self.children = {}
        # Count of words that end at this node
        self.count = 0


class Solution:
    def countPrefixSuffixPairs(self, words: List[str]) -> int:
        """
        Count the number of pairs (i, j) where i < j and words[i] is both 
        a prefix and suffix of words[j].
      
        The algorithm uses a Trie where each node represents a character pair
        (prefix_char, suffix_char). By traversing with paired characters from
        the start and end of each word simultaneously, we can efficiently check
        if previous words are both prefixes and suffixes.
      
        Args:
            words: List of strings to analyze
          
        Returns:
            Number of valid prefix-suffix pairs
        """
        result = 0
        root = TrieNode()
      
        # Process each word in order
        for word in words:
            current_node = root
          
            # Create character pairs: (first_char, last_char), (second_char, second_last_char), etc.
            # This allows checking prefix and suffix conditions simultaneously
            for char_pair in zip(word, reversed(word)):
                # Create new node if this character pair hasn't been seen at this position
                if char_pair not in current_node.children:
                    current_node.children[char_pair] = TrieNode()
              
                # Move to the child node corresponding to this character pair
                current_node = current_node.children[char_pair]
              
                # Add count of all previous words that match up to this position
                # These words are both prefixes and suffixes of the current word
                result += current_node.count
          
            # Mark that this word ends at the current node
            current_node.count += 1
      
        return result
```

## Time and Space Complexity

The time complexity is `O(n × m)`, where `n` is the number of words in the array and `m` is the maximum length among all strings.

For each word in `words` (n iterations), the algorithm iterates through each character of that word to create pairs of `(s[i], s[-(i+1)])` using `zip(s, reversed(s))`. This pairing operation takes `O(m)` time for a word of length `m`. Within each character iteration, the trie operations (checking existence, creating nodes, and incrementing counters) are `O(1)`. Therefore, the overall time complexity is `O(n × m)`.

The space complexity is `O(n × m)`. In the worst case, when all words have completely different prefix-suffix pairs, the trie needs to store a unique path for each word. Each word can contribute up to `m` nodes (one for each character position), and with `n` words, the maximum number of nodes in the trie is `O(n × m)`. Each node stores a dictionary of children and a counter, but the space for these is already accounted for in the node count. The `__slots__` optimization helps reduce memory overhead per node instance, but doesn't change the asymptotic complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding the Character Pairing Logic**

**Pitfall:** Developers often misunderstand why we pair characters from the start and end of the string. They might think we need to check prefix and suffix separately, leading to incorrect implementations like:

```python
# INCORRECT: Checking prefix and suffix separately
def countPairs_wrong(words):
    result = 0
    for j in range(len(words)):
        for i in range(j):
            if words[j].startswith(words[i]) and words[j].endswith(words[i]):
                result += 1
    return result
```

**Why it's wrong:** While this brute force approach works, it defeats the purpose of using a Trie and has O(n²m) complexity.

**Solution:** Understand that the character pairing `(s[i], s[len(s)-1-i])` simultaneously validates both conditions. When a string is both a prefix and suffix, these pairs will match perfectly.

### 2. **Incorrect Order of Operations**

**Pitfall:** Adding the count AFTER incrementing it, or incrementing before traversal:

```python
# INCORRECT: Wrong order
for char_pair in zip(word, reversed(word)):
    if char_pair not in current_node.children:
        current_node.children[char_pair] = TrieNode()
    current_node = current_node.children[char_pair]

# Incrementing before accumulating - WRONG!
current_node.count += 1
result += current_node.count  # This includes the current word itself
```

**Solution:** Always accumulate the count BEFORE incrementing. The correct order is:
1. Traverse and accumulate existing counts
2. Only increment count after full traversal

### 3. **Handling Edge Cases Incorrectly**

**Pitfall:** Not considering that a word cannot be a prefix-suffix pair with itself:

```python
# INCORRECT: Might count self-pairs if not careful
if word in previous_words:  # Wrong approach
    result += 1
```

**Solution:** The algorithm naturally handles this by processing words in order and only counting previously inserted words (those with smaller indices).

### 4. **Memory Inefficiency with Character Pairs**

**Pitfall:** Storing character pairs inefficiently or creating unnecessary objects:

```python
# INEFFICIENT: Creating new strings for pairs
char_pair = word[i] + word[-(i+1)]  # Creates new string objects
```

**Solution:** Use tuples for character pairs as they are immutable and hashable:
```python
char_pair = (word[i], word[-(i+1)])  # Efficient tuple
# Or better yet, use zip with reversed:
for char_pair in zip(word, reversed(word)):
```

### 5. **Forgetting to Initialize the Trie Root**

**Pitfall:** Reusing the same root for multiple test cases or forgetting to initialize:

```python
# INCORRECT: Using class variable (shared across instances)
class Solution:
    root = TrieNode()  # This is shared!
  
    def countPrefixSuffixPairs(self, words):
        # Will have stale data from previous calls
```

**Solution:** Always create a fresh Trie root for each function call:
```python
def countPrefixSuffixPairs(self, words):
    root = TrieNode()  # Fresh root for each call
```

