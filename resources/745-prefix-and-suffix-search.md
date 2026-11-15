# 745. Prefix and Suffix Search

> Source: https://algo.monster/liteproblems/745

## Problem Statement

You need to design a data structure that can efficiently search for words based on both a prefix (beginning) and a suffix (ending).

The `WordFilter` class should support two operations:

1. **Constructor `WordFilter(string[] words)`**: Initialize the data structure with an array of words. Each word has an index based on its position in the array (starting from 0).

2. **Method `f(string pref, string suff)`**: Find a word in the dictionary that starts with the given prefix `pref` and ends with the given suffix `suff`. If multiple words match both conditions, return the index of the word with the **largest index** (the one that appears latest in the original array). If no word matches both conditions, return `-1`.

For example, if you have words `["apple", "apply", "ample"]`:
- `f("a", "le")` would return `2` because both "apple" (index 0) and "ample" (index 2) match, and 2 is the larger index
- `f("app", "ly")` would return `1` because only "apply" matches
- `f("ap", "x")` would return `-1` because no word starts with "ap" and ends with "x"

## Intuition

The key insight is that we need to quickly find words that match both a prefix and suffix constraint. The naive approach would be to check every word during each query, but this would be too slow for multiple queries.

Instead of searching at query time, we can pre-process all the information we might need. Think about it this way: for any word, there are a finite number of possible prefixes and suffixes. For example, the word "apple" has these prefixes: `""`, `"a"`, `"ap"`, `"app"`, `"appl"`, `"apple"` and these suffixes: `""`, `"e"`, `"le"`, `"ple"`, `"pple"`, `"apple"`.

Any query `f(pref, suff)` will ask for one of these prefix-suffix combinations. So why not compute all possible combinations beforehand and store them?

For each word, we can generate all possible `(prefix, suffix)` pairs. If a word has length `n`, there are `(n+1) × (n+1)` such pairs (including empty strings). We store each pair as a key in a dictionary, with the word's index as the value.

The clever part is handling multiple words that share the same `(prefix, suffix)` combination. Since we want the largest index, we simply process the words in order - when we encounter a duplicate `(prefix, suffix)` pair, the later word's index naturally overwrites the earlier one in the dictionary.

This transforms our search problem into a simple dictionary lookup problem. We trade space (storing all combinations) for time (constant-time lookups). The preprocessing might seem expensive with `O(n² × m)` where `n` is average word length and `m` is number of words, but it makes each subsequent query `O(1)`, which is ideal when we expect many queries.

**Learn more about [Trie](/problems/trie_intro) patterns.**

## Solution Approach

The solution uses a preprocessing strategy with a hash map to achieve constant-time lookups.

**Data Structure:**
- A dictionary `self.d` that maps `(prefix, suffix)` tuples to word indices.

**Initialization (`__init__` method):**

1. Create an empty dictionary `self.d` to store all prefix-suffix combinations.

2. Iterate through each word with its index using `enumerate(words)`:
   - For word at index `k` with length `n`
   - Generate all possible prefixes: `w[:i]` where `i` ranges from `0` to `n+1`
     - When `i=0`: prefix is `""` (empty string)
     - When `i=1`: prefix is first character
     - When `i=n`: prefix is entire word
 
3. For each prefix, generate all possible suffixes: `w[j:]` where `j` ranges from `0` to `n+1`
   - When `j=0`: suffix is entire word
   - When `j=n-1`: suffix is last character
   - When `j=n`: suffix is `""` (empty string)

4. Store the combination: `self.d[(prefix, suffix)] = k`
   - The key is a tuple `(prefix, suffix)`
   - The value is the current word's index `k`
   - If the same `(prefix, suffix)` pair appears in multiple words, the later index overwrites the earlier one, automatically giving us the largest index.

**Query (`f` method):**

Simply look up the `(pref, suff)` tuple in the dictionary:
- Use `self.d.get((pref, suff), -1)`
- Returns the stored index if the combination exists
- Returns `-1` if the combination doesn't exist (default value)

**Example walkthrough:**

For `words = ["apple"]`:
- The word "apple" (index 0) generates these combinations:
  - `("", "apple")`, `("", "pple")`, `("", "ple")`, `("", "le")`, `("", "e")`, `("", "")`
  - `("a", "apple")`, `("a", "pple")`, ... , `("a", "")`
  - `("ap", "apple")`, `("ap", "pple")`, ... , `("ap", "")`
  - And so on for all prefix lengths

All these combinations map to index 0 in the dictionary.

**Time Complexity:**
- Initialization: `O(N × L²)` where `N` is number of words and `L` is average word length
- Query: `O(1)` for dictionary lookup

**Space Complexity:** `O(N × L²)` to store all prefix-suffix combinations

## Example Walkthrough

Let's walk through a concrete example with `words = ["a", "banana", "band", "bana"]`.

**Initialization Phase:**

For each word, we generate all prefix-suffix combinations and store them with the word's index:

**Word "a" (index 0):**
- Prefixes: `""`, `"a"`
- Suffixes: `"a"`, `""`
- Combinations stored:
  - `("", "a") → 0`
  - `("", "") → 0`
  - `("a", "a") → 0`
  - `("a", "") → 0`

**Word "banana" (index 1):**
- Generates 49 combinations (7 prefixes × 7 suffixes)
- Key combinations include:
  - `("b", "a") → 1` (starts with "b", ends with "a")
  - `("ban", "ana") → 1` (starts with "ban", ends with "ana")
  - `("banana", "banana") → 1` (entire word)
  - And 46 more...

**Word "band" (index 2):**
- Generates 25 combinations (5 prefixes × 5 suffixes)
- Notable combinations:
  - `("b", "d") → 2`
  - `("ban", "d") → 2`
  - `("band", "and") → 2`

**Word "bana" (index 3):**
- Generates 25 combinations
- Important: `("ban", "a") → 3` (overwrites any previous mapping)
- Also: `("b", "a") → 3` (overwrites the previous `("b", "a") → 1`)

**Query Phase:**

Now let's execute some queries:

1. **`f("b", "a")`:**
   - Look up `("b", "a")` in dictionary
   - Returns `3` (was overwritten by "bana" which has the largest index)

2. **`f("ban", "d")`:**
   - Look up `("ban", "d")` in dictionary
   - Returns `2` (only "band" matches this combination)

3. **`f("ban", "ana")`:**
   - Look up `("ban", "ana")` in dictionary
   - Returns `1` (only "banana" has this prefix-suffix pair)

4. **`f("x", "y")`:**
   - Look up `("x", "y")` in dictionary
   - Key doesn't exist, returns `-1`

The key insight: when "bana" (index 3) was processed, it overwrote the `("b", "a")` entry that was previously set by "banana" (index 1). This automatic overwriting ensures we always get the largest index for any prefix-suffix combination.

## Python Solution

```python
from typing import List

class WordFilter:
    def __init__(self, words: List[str]):
        # Dictionary to store all possible (prefix, suffix) combinations
        # Key: (prefix, suffix) tuple, Value: index of the word
        self.prefix_suffix_map = {}
      
        # Process each word with its index
        for index, word in enumerate(words):
            word_length = len(word)
          
            # Generate all possible prefixes (including empty string)
            for prefix_end in range(word_length + 1):
                prefix = word[:prefix_end]
              
                # For each prefix, generate all possible suffixes (including empty string)
                for suffix_start in range(word_length + 1):
                    suffix = word[suffix_start:]
                  
                    # Store the combination with the current index
                    # Later occurrences will overwrite earlier ones, keeping the largest index
                    self.prefix_suffix_map[(prefix, suffix)] = index

    def f(self, pref: str, suff: str) -> int:
        # Look up the (prefix, suffix) combination in the map
        # Return the stored index if found, otherwise return -1
        return self.prefix_suffix_map.get((pref, suff), -1)


# Your WordFilter object will be instantiated and called as such:
# obj = WordFilter(words)
# param_1 = obj.f(pref, suff)
```

## Time and Space Complexity

**Time Complexity:**
- **Initialization (`__init__`):** `O(N * L^2)` where `N` is the number of words and `L` is the maximum length of a word. For each word, we generate all possible prefix-suffix combinations. There are `(L+1)` possible prefixes (including empty string) and `(L+1)` possible suffixes, resulting in `(L+1)^2` combinations per word. Thus, the total time complexity is `O(N * L^2)`.

- **Query (`f`):** `O(1)` for dictionary lookup operation.

**Space Complexity:**
- **Space:** `O(N * L^2)` for storing the dictionary. In the worst case, each word of length `L` generates `(L+1)^2` prefix-suffix pairs. With `N` words, the total space needed is `O(N * L^2)`. Note that while multiple prefix-suffix combinations might map to the same word index, we still need to store all unique `(prefix, suffix)` tuples as keys in the dictionary.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Memory Limit Exceeded for Large Inputs**

The cost of this solution is memory. For each word, we generate all possible prefix-suffix combinations, which is O(L²) combinations per word. Under the problem's constraints (up to 10,000 words, each at most 7 characters), that is at most 64 keys per word and 640,000 keys in total, which fits within typical limits. If words could be much longer, the quadratic number of keys per word would become the bottleneck.

**Why this happens:**
- A word of length L generates (L+1) × (L+1) combinations
- For "apple" (length 5): 6 × 6 = 36 combinations
- For 10,000 such words: potentially 360,000+ dictionary entries

**Alternative:** Use a Trie-based approach with a special encoding:
```python
class WordFilter:
    def __init__(self, words: List[str]):
        self.trie = {}
      
        for index, word in enumerate(words):
            # Create a special format: suffix#word
            # This allows searching for both prefix and suffix in one trie
            word_length = len(word)
            for i in range(word_length + 1):
                # Combine suffix with word using delimiter
                encoded = word[i:] + '#' + word
              
                # Insert into trie
                node = self.trie
                for char in encoded:
                    if char not in node:
                        node[char] = {}
                    node = node[char]
                    # Store the index at each node
                    node['index'] = index
  
    def f(self, pref: str, suff: str) -> int:
        # Search for suff#pref in the trie
        search_key = suff + '#' + pref
        node = self.trie
      
        for char in search_key:
            if char not in node:
                return -1
            node = node[char]
      
        return node.get('index', -1)
```

### 2. **Handling Edge Cases with Empty Strings**

The current solution correctly handles empty prefixes and suffixes, but developers often forget these cases when implementing from scratch.

**Common mistake:**
```python
# Incorrect: doesn't include empty string cases
for prefix_end in range(1, word_length + 1):  # Misses empty prefix
    for suffix_start in range(word_length):   # Misses empty suffix
```

**Correct approach (as in the solution):**
```python
# Include 0 for empty prefix and word_length+1 for empty suffix
for prefix_end in range(word_length + 1):
    for suffix_start in range(word_length + 1):
```

### 3. **Skipping Overlapping Prefix-Suffix Pairs**

It can look like a saving to skip pairs where the prefix and suffix overlap (`prefix_end > suffix_start`). Those pairs are valid queries, though. For `words = ["baba"]`, the query `f("bab", "ba")` should return `0`: "baba" starts with "bab" and ends with "ba", and the two parts share the middle `"b"`. A version that skips overlapping pairs never stores `("bab", "ba")` and returns `-1`.

**Wrong approach:**
```python
for prefix_end in range(word_length + 1):
    for suffix_start in range(word_length + 1):
        if prefix_end > suffix_start:  # Drops valid overlapping pairs
            continue
        self.prefix_suffix_map[(word[:prefix_end], word[suffix_start:])] = index
```

**Correct approach:** store every `(prefix, suffix)` pair, as the solution does. The only overlap that matters is between different words that share a key, and the overwrite in index order already handles it.

