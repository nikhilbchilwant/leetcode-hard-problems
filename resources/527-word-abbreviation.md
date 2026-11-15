# 527. Word Abbreviation

> Source: https://algo.monster/liteproblems/527

## Problem Statement

You are given an array of **distinct** strings `words`. Your task is to return the minimal possible **abbreviations** for every word.

The abbreviation rules work as follows:

1. **Initial abbreviation**: Each word starts with an abbreviation formed by taking the first character, followed by the count of characters in between, followed by the last character. For example, `"abcdef"` becomes `"a4f"` (first char 'a', 4 characters in between, last char 'f').

2. **Handling conflicts**: When multiple words share the same abbreviation, you need to resolve the conflict by:
   - Increasing the prefix (the first part) of their abbreviations by 1 character at a time
   - Continue this process until all abbreviations become unique
   - For example: If `"abcdef"` and `"abndef"` both initially abbreviate to `"a4f"`, you would:
     - First expand to `"ab3f"` and `"ab3f"` (still conflicting)
     - Then expand to `"abc2f"` and `"abn2f"` (now unique)

3. **Final check**: After finding the unique abbreviation, if the abbreviation is not shorter than the original word, keep the original word instead.

The goal is to find the shortest possible unique abbreviation for each word following these rules. The output should be an array where each element is either the abbreviated form or the original word (if abbreviation doesn't save space).

## Intuition

The key insight is recognizing that words can only have the same abbreviation if they share certain properties: they must have the **same length** and the **same last letter**. This is because the abbreviation format is `prefix + count + last_letter`, where the count depends on the total length minus the prefix length minus 1.

For example, `"apple"` (length 5, last letter 'e') can only conflict with other 5-letter words ending in 'e'. It could never conflict with `"apply"` (different last letter) or `"sample"` (different length).

This observation suggests we can **group words** by `(length, last_letter)` pairs and handle each group independently. Within each group, we need to find the minimum prefix length that makes each word's abbreviation unique.

To efficiently find the minimum distinguishing prefix for each word in a group, we can use a **[Trie](/problems/trie_intro) (prefix tree)**. The Trie allows us to track how many words share each prefix. As we traverse down the Trie for a word, the first node where `cnt = 1` indicates we've found a prefix that uniquely identifies this word among its group.

The algorithm becomes:
1. Group words by `(length, last_letter)`
2. Build a separate [Trie](/problems/trie_intro) for each group
3. For each word, traverse its group's Trie to find the minimum prefix length where it becomes unique
4. Generate the abbreviation using this prefix length, but only use it if it's shorter than the original word

This approach is efficient because we only compare words that could potentially conflict, and the [Trie](/problems/trie_intro) structure allows us to find the minimum distinguishing prefix in a single traversal per word.

**Learn more about [Greedy](/problems/greedy_intro), [Trie](/problems/trie_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The solution implements the grouped [Trie](/problems/trie_intro) approach through two main components: a `Trie` class and the main solution logic.

### [Trie](/problems/trie_intro) Implementation

The `[Trie](/problems/trie_intro)` class has the following structure:
- `children`: An array of size 26 representing child nodes for each lowercase letter
- `cnt`: Counter tracking how many words pass through this node

The [Trie](/problems/trie_intro) provides two key methods:

**`insert(w)`**: Inserts a word into the [Trie](/problems/trie_intro)
- Traverse each character of the word
- For each character, create a child node if it doesn't exist
- Increment the `cnt` at each node visited
- This `cnt` value tells us how many words share this prefix

**`search(w)`**: Finds the minimum prefix length for unique identification
- Traverse the word character by character
- Keep track of the prefix length (`cnt`)
- When we find a node where `node.cnt == 1`, we've found the unique prefix
- Return this prefix length, or the full word length if no unique prefix exists before the end

### Main Algorithm

1. **Group words by (length, last_letter)**:
   ```python
   tries = {}
   for w in words:
       m = len(w)
       if (m, w[-1]) not in tries:
           tries[(m, w[-1])] = [Trie](/problems/trie_intro)()
       tries[(m, w[-1])].insert(w)
   ```
   - Create a dictionary where keys are `(word_length, last_character)` tuples
   - Each key maps to a separate Trie containing all words with that length and ending

2. **Find minimum abbreviation for each word**:
   ```python
   for w in words:
       cnt = tries[(len(w), w[-1])].search(w)
   ```
   - For each word, search in its corresponding group's [Trie](/problems/trie_intro)
   - Get the minimum prefix length needed for uniqueness

3. **Generate the final abbreviation**:
   ```python
   ans.append(
       w if cnt + 2 >= len(w) else w[:cnt] + str(len(w) - cnt - 1) + w[-1]
   )
   ```
   - If `cnt + 2 >= len(w)`: The abbreviation won't be shorter (we need at least the prefix, one digit, and last letter), so keep the original word
   - Otherwise: Create abbreviation as `prefix + middle_count + last_letter`
     - `w[:cnt]`: The unique prefix
     - `str(len(w) - cnt - 1)`: Count of characters between prefix and last letter
     - `w[-1]`: The last letter

### Time and Space Complexity

- **Time Complexity**: `O(N × L)` where N is the number of words and L is the average word length
  - Inserting all words into Tries: `O(N × L)`
  - Searching for each word: `O(N × L)`

- **Space Complexity**: `O(N × L)` for storing all words in the Tries

The beauty of this approach is that it only compares words that could potentially have the same abbreviation, making it much more efficient than comparing all pairs of words.

## Example Walkthrough

Let's walk through the solution with a small example: `words = ["apple", "apply", "ample"]`

**Step 1: Group words by (length, last_letter)**
- `"apple"`: length = 5, last letter = 'e' → group (5, 'e')
- `"apply"`: length = 5, last letter = 'y' → group (5, 'y')
- `"ample"`: length = 5, last letter = 'e' → group (5, 'e')

So we have two groups:
- Group (5, 'e'): ["apple", "ample"]
- Group (5, 'y'): ["apply"]

**Step 2: Build Tries for each group**

For group (5, 'e'), we build a Trie with "apple" and "ample":
```
     root
        |
        a (cnt=2)
       / \
      p   m (cnt=1)
   (cnt=1) |
      |    p (cnt=1)
      p    |
      |    l (cnt=1)
      l    |
      |    e (cnt=1)
      e
```

For group (5, 'y'), we build a Trie with just "apply":
```
     root
        |
        a → p → p → l → y (all cnt=1)
```

**Step 3: Find minimum prefix for each word**

- **"apple"**: Traverse the (5, 'e') Trie
  - 'a': cnt=2 (not unique, keep going)
  - 'ap': cnt=1 (unique!)
  - Minimum prefix length = 2

- **"ample"**: Traverse the (5, 'e') Trie
  - 'a': cnt=2 (not unique, keep going)
  - 'am': cnt=1 (unique!)
  - Minimum prefix length = 2

- **"apply"**: Traverse the (5, 'y') Trie
  - 'a': cnt=1 (unique!)
  - Minimum prefix length = 1

**Step 4: Generate abbreviations**

- **"apple"**: prefix length = 2
  - Abbreviation: "ap" + (5-2-1) + "e" = "ap2e"
  - Length check: 4 < 5, so use "ap2e"

- **"ample"**: prefix length = 2
  - Abbreviation: "am" + (5-2-1) + "e" = "am2e"
  - Length check: 4 < 5, so use "am2e"

- **"apply"**: prefix length = 1
  - Abbreviation: "a" + (5-1-1) + "y" = "a3y"
  - Length check: 3 < 5, so use "a3y"

**Final Result**: `["ap2e", "a3y", "am2e"]`

Note how "apple" and "ample" needed longer prefixes (2 characters) because they're in the same group and would conflict with just "a3e", while "apply" only needs 1 character prefix since it's alone in its group.

## Python Solution

```python
class Trie:
    """Trie node for storing words and tracking prefix frequencies."""
    __slots__ = ["children", "cnt"]

    def __init__(self):
        # Array to store 26 lowercase letter children nodes
        self.children = [None] * 26
        # Count of words passing through this node
        self.cnt = 0

    def insert(self, w: str):
        """Insert a word into the trie and update counts along the path."""
        node = self
        for c in w:
            # Calculate index for the character (0-25)
            idx = ord(c) - ord("a")
            # Create new node if it doesn't exist
            if not node.children[idx]:
                node.children[idx] = Trie()
            # Move to child node
            node = node.children[idx]
            # Increment count for this prefix
            node.cnt += 1

    def search(self, w: str) -> int:
        """
        Find the minimum prefix length that uniquely identifies the word.
        Returns the number of characters needed for a unique prefix.
        """
        node = self
        cnt = 0
        for c in w:
            cnt += 1
            idx = ord(c) - ord("a")
            node = node.children[idx]
            # If only one word passes through this node, we found unique prefix
            if node.cnt == 1:
                return cnt
        # If no unique prefix found, return full word length
        return len(w)


class Solution:
    def wordsAbbreviation(self, words: List[str]) -> List[str]:
        """
        Generate abbreviations for a list of words.
        Words with same length and last character are grouped together.
        """
        # Dictionary to store tries grouped by (word_length, last_character)
        tries = {}

        # Build tries for each group of words with same length and last character
        for w in words:
            m = len(w)
            key = (m, w[-1])
            # Create new trie for this group if it doesn't exist
            if key not in tries:
                tries[key] = Trie()
            # Insert word into the appropriate trie
            tries[key].insert(w)

        # Generate abbreviations for each word
        ans = []
        for w in words:
            # Find minimum unique prefix length for this word
            cnt = tries[(len(w), w[-1])].search(w)
            # Check if abbreviation would be shorter than original
            # Need at least: prefix + number + last_char (cnt + 1 + 1)
            if cnt + 2 >= len(w):
                # Abbreviation wouldn't save space, use full word
                ans.append(w)
            else:
                # Create abbreviation: prefix + count + last_character
                abbreviation = w[:cnt] + str(len(w) - cnt - 1) + w[-1]
                ans.append(abbreviation)

        return ans
```

## Time and Space Complexity

**Time Complexity: `O(L)`**

The algorithm processes each word twice - once during insertion into the trie and once during search:
- Building tries: Each word is inserted once. For a word of length `l`, insertion takes `O(l)` time. Across all `n` words, this totals to `O(L)` where `L` is the sum of all word lengths.
- Searching in tries: Each word is searched once. For a word of length `l`, search takes `O(l)` time in the worst case. Across all `n` words, this again totals to `O(L)`.
- The grouping by `(length, last_character)` and final abbreviation construction are both `O(1)` per word or `O(n)` total, which is dominated by `O(L)` since `L ≥ n`.

Therefore, the overall time complexity is `O(L) + O(L) = O(L)`.

**Space Complexity: `O(L)`**

The space is primarily consumed by the trie structures:
- Each character in each word creates at most one trie node. In the worst case where all words have unique prefixes within their groups, we store `O(L)` characters across all tries.
- Each trie node contains an array of 26 pointers and a counter, which is `O(1)` space per node.
- The `tries` dictionary stores at most `O(n)` entries (one per unique `(length, last_character)` pair), which is bounded by `O(L)`.
- The output array `ans` stores `n` abbreviated strings, with total length at most `O(L)`.

Therefore, the overall space complexity is `O(L)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Abbreviation Length Calculation**

One of the most common mistakes is incorrectly calculating when an abbreviation is actually shorter than the original word. The pitfall occurs when checking if the abbreviation saves space.

**Incorrect Implementation:**
```python
# Wrong: Only considering prefix length + 1
if cnt + 1 >= len(w):
    ans.append(w)
```

**Why it's wrong:** The abbreviation format is `prefix + number + last_char`. Even if the number is a single digit, we need at least 2 additional characters beyond the prefix. For example:
- Word: `"word"` (length 4)
- If prefix length is 2: `"wo"`, the abbreviation would be `"wo1d"` (length 4)
- This doesn't save any space!

**Correct Implementation:**
```python
# Correct: Considering prefix + at least 1 digit + last character
if cnt + 2 >= len(w):
    ans.append(w)
```

### 2. **Not Handling Multi-Digit Numbers in Abbreviations**

Another subtle pitfall is assuming the middle count will always be a single digit. While the check `cnt + 2 >= len(w)` works for determining if abbreviation is worthwhile, you might encounter issues if trying to optimize further.

**Example scenario:**
- Word: `"internationalization"` (length 20)
- Prefix needed: `"i"` (length 1)
- Middle count: 18
- Abbreviation: `"i18n"` (length 4)

The actual abbreviation length is `1 + 2 + 1 = 4` (prefix + two digits + last char), not `1 + 1 + 1 = 3`.

**Better check for exact abbreviation length:**
```python
def get_abbr_length(prefix_len, word_len):
    middle_count = word_len - prefix_len - 1
    digit_count = len(str(middle_count))
    return prefix_len + digit_count + 1

# Use this for more precise checking
if get_abbr_length(cnt, len(w)) >= len(w):
    ans.append(w)
```

### 3. **Forgetting to Group by Both Length AND Last Character**

A critical mistake is only grouping by one criterion instead of both.

**Incorrect grouping:**
```python
# Wrong: Only grouping by length
tries = {}
for w in words:
    m = len(w)
    if m not in tries:
        tries[m] = Trie()
    tries[m].insert(w)
```

**Why it fails:** Words like `"abcd"` and `"abce"` both have length 4 but abbreviate to `"a2d"` and `"a2e"` respectively. They never conflict, but in a shared Trie the prefix `"abc"` has `cnt = 2`, so both words get a prefix of length 4 and are returned unabbreviated instead of as `"a2d"` and `"a2e"`.

**Correct grouping:**
```python
# Correct: Group by (length, last_character)
tries = {}
for w in words:
    key = (len(w), w[-1])
    if key not in tries:
        tries[key] = Trie()
    tries[key].insert(w)
```

### 4. **Off-by-One Error in Middle Count Calculation**

When calculating the number of characters between the prefix and last character, it's easy to make an off-by-one error.

**Incorrect:**
```python
# Wrong: Forgetting to subtract both prefix and last char
middle_count = len(w) - cnt
# or
# Wrong: Subtracting incorrectly
middle_count = len(w) - cnt - 2
```

**Correct:**
```python
# Correct: Total length - prefix length - 1 (for last char)
middle_count = len(w) - cnt - 1
```

**Example verification:**
- Word: `"apple"` (length 5)
- Prefix: `"ap"` (cnt = 2)
- Middle characters: `"pl"` (2 characters)
- Calculation: `5 - 2 - 1 = 2` ✓
- Abbreviation: `"ap2e"`

