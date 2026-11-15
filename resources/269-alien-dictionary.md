# 269. Alien Dictionary

> Source: https://algo.monster/liteproblems/269

## Problem Statement

You're given a list of words from an alien language that uses the same 26 letters as English, but the alphabetical order of these letters is different and unknown to you.

The words are claimed to be sorted in dictionary order according to this alien alphabet. Your task is to:

1. **Verify if the claim is valid**: Check if the given word order can actually correspond to some valid alphabetical ordering of letters. If not, return an empty string `""`.

2. **Recover the alien alphabet**: If the ordering is valid, determine what the alphabetical order of letters must be in this alien language and return it as a string containing all unique letters that appear in the words, sorted according to the alien alphabet's rules.

**Key Points:**
- The words should follow lexicographic ordering based on the alien alphabet (like how words are ordered in a dictionary)
- You need to deduce the relative order of letters by comparing adjacent words in the list
- If there are multiple valid orderings possible, return any one of them
- If the given arrangement is impossible (contradicts itself), return `""`

**Example of Invalid Input:**
If you have words like `["abc", "ab"]`, this would be invalid because in any alphabet, "abc" cannot come before "ab" in dictionary order (a longer word with the same prefix must come after the shorter word).

**Example of Deducing Order:**
If you have words `["wrt", "wrf"]`, you can deduce that `'t'` comes before `'f'` in the alien alphabet since these words differ at the third position and "wrt" comes before "wrf".

The solution uses topological sorting to build the alphabet order from the character relationships derived from comparing adjacent words.

## Intuition

The key insight is recognizing that determining the alien alphabet order is fundamentally about finding dependencies between characters. When we compare two adjacent words in the sorted list, we can extract one piece of information about the relative ordering of two characters.

Think about how dictionary ordering works: when comparing "apple" and "application", we look at characters from left to right. The first difference tells us the ordering - since "apple" comes before "application" and they differ at position 5 ('e' vs 'i'), we know 'e' comes before 'i' in the alphabet.

Similarly, for the alien dictionary, when we see `["wrt", "wrf"]`, we compare character by character:
- 'w' = 'w' (same, continue)
- 'r' = 'r' (same, continue)
- 't' ≠ 'f' (different!)

Since "wrt" comes before "wrf" in the sorted list, we deduce that 't' must come before 'f' in the alien alphabet.

Each such comparison gives us a directed edge in a graph: 't' → 'f' means 't' comes before 'f'. After processing all adjacent word pairs, we have a directed graph representing all the ordering constraints.

Now the problem transforms into: "Given all these ordering constraints (directed edges), can we find a valid total ordering of all characters?" This is exactly what topological sorting does - it finds a linear ordering of nodes in a directed graph such that for every edge u → v, u comes before v in the ordering.

The catch is that topological sorting only works on Directed Acyclic Graphs (DAGs). If there's a cycle (like 'a' → 'b' → 'c' → 'a'), then no valid ordering exists. Also, if we find a contradiction like a longer word appearing before its prefix (e.g., "abc" before "ab"), we know the input is invalid.

The solution naturally follows: build the graph from word comparisons, check for invalid cases, and perform topological sort to get the character ordering.

## Solution Approach

The implementation follows these key steps:

**1. Build the Graph and Track Characters**

First, we create a directed graph using a 2D boolean array `g[26][26]` where `g[i][j] = True` means character `i` comes before character `j`. We also track which characters appear in the words using array `s[26]` and count them with `cnt`.

```python
g = [[False] * 26 for _ in range(26)]  # Adjacency matrix for graph
s = [False] * 26  # Track which characters exist
cnt = 0  # Count of unique characters
```

**2. Extract Ordering Relationships**

For each pair of adjacent words, we compare them character by character to find the first difference:

```python
for i in range(n - 1):
    # First, mark all characters in current word as seen
    for c in words[i]:
        o = ord(c) - ord('a')
        if not s[o]:
            cnt += 1
            s[o] = True

    # Compare with next word
    for j in range(len(words[i])):
        if j >= len(words[i + 1]):
            return ''  # Invalid: longer word before its prefix

        c1, c2 = words[i][j], words[i + 1][j]
        if c1 == c2:
            continue  # Same character, keep comparing

        # Found first difference
        o1, o2 = ord(c1) - ord('a'), ord(c2) - ord('a')
        if g[o2][o1]:
            return ''  # Contradiction: reverse edge already exists
        g[o1][o2] = True  # Add edge: c1 comes before c2
        break
```

**3. Topological Sort using Kahn's Algorithm**

We use BFS-based topological sort (Kahn's algorithm) with indegree counting:

```python
# Calculate indegrees for all characters
indegree = [0] * 26
for i in range(26):
    for j in range(26):
        if i != j and s[i] and s[j] and g[i][j]:
            indegree[j] += 1

# Start with characters having no dependencies (indegree = 0)
q = deque()
for i in range(26):
    if s[i] and indegree[i] == 0:
        q.append(i)

# Process characters in topological order
ans = []
while q:
    t = q.popleft()
    ans.append(chr(t + ord('a')))

    # Remove this character's edges and update indegrees
    for i in range(26):
        if s[i] and i != t and g[t][i]:
            indegree[i] -= 1
            if indegree[i] == 0:
                q.append(i)
```

**4. Validate Result**

Finally, we check if we processed all characters. If `len(ans) < cnt`, it means there's a cycle in the graph (some characters never reached indegree 0), so we return empty string:

```python
return '' if len(ans) < cnt else ''.join(ans)
```

The algorithm efficiently handles:
- **Invalid prefix cases**: When a word is a prefix of an earlier word
- **Contradictions**: When we find conflicting ordering requirements
- **Cycles**: When the ordering constraints form a cycle
- **Multiple valid solutions**: Returns any valid topological ordering

Time complexity: `O(C)` where `C` is the total number of characters in all words, as we process each character at most once when comparing words and the graph operations are bounded by the alphabet size (26).

## Example Walkthrough

Let's walk through a concrete example with the words: `["wrt", "wrf", "er", "ett", "rftt"]`

**Step 1: Initialize Data Structures**
- Create a 26×26 graph matrix `g` (all False initially)
- Create array `s` to track which letters appear (all False initially)
- Set `cnt = 0` to count unique characters

**Step 2: Extract Ordering Relationships**

Compare adjacent words to find character dependencies:

1. **"wrt" vs "wrf":**
   - Mark characters in "wrt": w, r, t as seen (`cnt = 3`)
   - Compare: w=w (same), r=r (same), t≠f (different!)
   - Add edge: t → f (meaning t comes before f)

2. **"wrf" vs "er":**
   - Mark characters in "wrf": w, r, f as seen (f is new, `cnt = 4`)
   - Compare: w≠e (different!)
   - Add edge: w → e

3. **"er" vs "ett":**
   - Mark characters in "er": e, r as seen (e is new, `cnt = 5`)
   - Compare: e=e (same), r≠t (different!)
   - Add edge: r → t

4. **"ett" vs "rftt":**
   - Mark characters in "ett": e, t as seen (all already seen)
   - Compare: e≠r (different!)
   - Add edge: e → r

After processing the last word "rftt", all characters are marked (`cnt = 5` total: w, r, t, f, e)

**Our graph now has edges:**
- t → f
- w → e
- r → t
- e → r

**Step 3: Calculate Indegrees**
Count incoming edges for each character:
- w: 0 (no incoming edges)
- e: 1 (from w)
- r: 1 (from e)
- t: 1 (from r)
- f: 1 (from t)

**Step 4: Topological Sort**

Start with characters having indegree 0:
- Queue: [w]
- Result: []

Process w:
- Add 'w' to result: ['w']
- Remove edge w → e, reducing e's indegree to 0
- Queue: [e]

Process e:
- Add 'e' to result: ['w', 'e']
- Remove edge e → r, reducing r's indegree to 0
- Queue: [r]

Process r:
- Add 'r' to result: ['w', 'e', 'r']
- Remove edge r → t, reducing t's indegree to 0
- Queue: [t]

Process t:
- Add 't' to result: ['w', 'e', 'r', 't']
- Remove edge t → f, reducing f's indegree to 0
- Queue: [f]

Process f:
- Add 'f' to result: ['w', 'e', 'r', 't', 'f']
- Queue: [] (empty)

**Step 5: Validate and Return**
- We processed 5 characters and `cnt = 5`, so all characters were included
- No cycles detected
- Return: `"wertf"`

This represents the alien alphabet order where w < e < r < t < f. You can verify this ordering makes the original word list correctly sorted!

## Python Solution

```python
class Solution:
    def alienOrder(self, words: List[str]) -> str:
        # Initialize adjacency matrix for the directed graph
        # graph[i][j] = True means character i comes before character j
        graph = [[False] * 26 for _ in range(26)]

        # Track which characters appear in the words
        seen_chars = [False] * 26
        char_count = 0
        num_words = len(words)

        # Build the graph by comparing adjacent words
        for i in range(num_words - 1):
            # Mark all characters in current word as seen
            for char in words[i]:
                if char_count == 26:
                    break
                char_index = ord(char) - ord('a')
                if not seen_chars[char_index]:
                    char_count += 1
                    seen_chars[char_index] = True

            # Compare current word with next word to find ordering
            curr_word_len = len(words[i])
            for j in range(curr_word_len):
                # If current word is longer than next word and they match up to this point,
                # the ordering is invalid (e.g., "abc" cannot come before "ab")
                if j >= len(words[i + 1]):
                    return ''

                char1, char2 = words[i][j], words[i + 1][j]
                if char1 == char2:
                    continue

                # Found first different character - this gives us ordering information
                index1, index2 = ord(char1) - ord('a'), ord(char2) - ord('a')

                # Check for contradictory ordering (char2 already comes before char1)
                if graph[index2][index1]:
                    return ''

                # Add edge: char1 comes before char2
                graph[index1][index2] = True
                break

        # Mark characters in the last word as seen
        for char in words[num_words - 1]:
            if char_count == 26:
                break
            char_index = ord(char) - ord('a')
            if not seen_chars[char_index]:
                char_count += 1
                seen_chars[char_index] = True

        # Calculate in-degrees for topological sort
        in_degree = [0] * 26
        for i in range(26):
            for j in range(26):
                if i != j and seen_chars[i] and seen_chars[j] and graph[i][j]:
                    in_degree[j] += 1

        # Perform topological sort using Kahn's algorithm
        queue = deque()
        result = []

        # Add all characters with in-degree 0 to queue
        for i in range(26):
            if seen_chars[i] and in_degree[i] == 0:
                queue.append(i)

        # Process queue
        while queue:
            curr_char_index = queue.popleft()
            result.append(chr(curr_char_index + ord('a')))

            # Reduce in-degree for all neighbors
            for i in range(26):
                if seen_chars[i] and i != curr_char_index and graph[curr_char_index][i]:
                    in_degree[i] -= 1
                    if in_degree[i] == 0:
                        queue.append(i)

        # If we couldn't process all characters, there's a cycle
        return '' if len(result) < char_count else ''.join(result)
```

## Time and Space Complexity

**Time Complexity:** `O(V + E)` where `V` is the number of unique characters and `E` is the number of edges in the graph, or more precisely `O(C)` where `C` is the total number of characters in all words.

- Building the graph: We iterate through `n-1` pairs of adjacent words. For each pair, we compare characters up to the length of the shorter word, which takes `O(min(len(words[i]), len(words[i+1])))` time. In the worst case, this is `O(C)` where `C` is the total number of characters.
- Counting unique characters: We iterate through all characters in all words once, which is `O(C)`.
- Building the indegree array: We check all possible edges in the graph, which is `O(26 * 26) = O(1)` since the alphabet size is fixed at 26.
- Topological sort using BFS: We process each unique character once and for each character, we check all 26 possible outgoing edges. This is `O(V * 26) = O(V)` where `V ≤ 26`.

Overall time complexity: `O(C + V^2)` where `V ≤ 26`, which simplifies to `O(C)` since `V` is bounded by a constant.

**Space Complexity:** `O(1)` or `O(V^2)` where `V ≤ 26`

- Graph adjacency matrix `g`: `O(26 * 26) = O(1)`
- Character existence array `s`: `O(26) = O(1)`
- Indegree array: `O(26) = O(1)`
- Queue for BFS: `O(26) = O(1)` in the worst case
- Answer list: `O(26) = O(1)` in the worst case

Since the alphabet size is fixed at 26, all space used is constant, giving us `O(1)` space complexity.

## Common Pitfalls

### 1. **Missing the Prefix Validation Check**

One of the most common pitfalls is forgetting to check if a word is a prefix of its predecessor. When comparing adjacent words, if we exhaust all characters of the current word without finding a difference, but the next word is shorter, this represents an invalid ordering.

**Incorrect approach:**
```python
# Many solutions forget this critical check
for j in range(min(len(words[i]), len(words[i + 1]))):
    if words[i][j] != words[i + 1][j]:
        # Add edge and break
        break
# Missing: what if all characters match but words[i] is longer?
```

**Correct approach:**
```python
for j in range(len(words[i])):
    # Check if next word is shorter (invalid case)
    if j >= len(words[i + 1]):
        return ''  # "abc" cannot come before "ab"

    if words[i][j] != words[i + 1][j]:
        # Add edge and break
        break
```

### 2. **Not Detecting All Characters in the Words**

Another pitfall is only tracking characters that appear in the ordering relationships (edges), missing characters that appear in words but don't have any ordering constraints.

**Incorrect approach:**
```python
# Only tracking characters when adding edges
if char1 != char2:
    seen_chars[ord(char1) - ord('a')] = True
    seen_chars[ord(char2) - ord('a')] = True
    # Missing: characters that never differ between adjacent words
```

**Correct approach:**
```python
# Track ALL characters that appear in ANY word
for i in range(num_words):
    for char in words[i]:
        char_index = ord(char) - ord('a')
        if not seen_chars[char_index]:
            char_count += 1
            seen_chars[char_index] = True
```

### 3. **Incorrectly Handling Isolated Characters**

Characters that appear in the words but have no ordering relationships (no incoming or outgoing edges) still need to be included in the final result. These characters have in-degree 0 and should be added to the initial queue.

**Incorrect approach:**
```python
# Only considering characters that have edges
for i in range(26):
    if graph[i].any() or any(graph[j][i] for j in range(26)):
        if in_degree[i] == 0:
            queue.append(i)
```

**Correct approach:**
```python
# Include ALL seen characters with in-degree 0
for i in range(26):
    if seen_chars[i] and in_degree[i] == 0:
        queue.append(i)
```

### 4. **Not Checking for Contradictory Edges**

When adding a new edge to the graph, it's important to check if the reverse edge already exists, which would indicate a contradiction in the ordering.

**Incorrect approach:**
```python
# Blindly adding edges without checking for contradictions
if char1 != char2:
    graph[index1][index2] = True
    break
```

**Correct approach:**
```python
if char1 != char2:
    index1, index2 = ord(char1) - ord('a'), ord(char2) - ord('a')
    # Check if reverse edge exists (contradiction)
    if graph[index2][index1]:
        return ''
    graph[index1][index2] = True
    break
```

### 5. **Breaking Too Early When Building the Graph**

A subtle pitfall is breaking out of the comparison loop even when characters are equal, or not breaking when a difference is found.

**Incorrect approach:**
```python
for j in range(min(len(words[i]), len(words[i + 1]))):
    char1, char2 = words[i][j], words[i + 1][j]
    # Wrong: might break even when chars are equal
    # or might continue after finding first difference
    if char1 != char2:
        graph[ord(char1) - ord('a')][ord(char2) - ord('a')] = True
    # Missing break statement
```

**Correct approach:**
```python
for j in range(len(words[i])):
    if j >= len(words[i + 1]):
        return ''

    char1, char2 = words[i][j], words[i + 1][j]
    if char1 == char2:
        continue  # Keep comparing if characters are equal

    # Process the first difference and break immediately
    index1, index2 = ord(char1) - ord('a'), ord(char2) - ord('a')
    if graph[index2][index1]:
        return ''
    graph[index1][index2] = True
    break  # Must break after finding first difference
```

These pitfalls can lead to incorrect results, missing characters in the output, or failure to detect invalid inputs. The solution provided handles all these cases correctly by carefully validating inputs, tracking all characters, and properly implementing the topological sort.

