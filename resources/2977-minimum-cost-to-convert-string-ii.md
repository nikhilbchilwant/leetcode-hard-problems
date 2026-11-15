# 2977. Minimum Cost to Convert String II

> Source: https://algo.monster/liteproblems/2977

## Problem Statement

You are given two strings `source` and `target` of equal length `n`, consisting of lowercase English letters. You need to transform `source` into `target` using a set of allowed string transformations.

The allowed transformations are defined by three arrays:
- `original[i]`: a substring that can be replaced
- `changed[i]`: what `original[i]` can be changed to
- `cost[i]`: the cost of changing `original[i]` to `changed[i]`

**Transformation Rules:**
1. In one operation, you can pick any substring `x` from the current string and change it to `y` at cost `z`, provided there exists an index `j` where `original[j] == x`, `changed[j] == y`, and `cost[j] == z`.

2. You can perform multiple operations, but they must follow these constraints:
   - The substrings selected in different operations must either be **disjoint** (non-overlapping), meaning if you pick `source[a..b]` and `source[c..d]`, then either `b < c` or `d < a`.
   - OR the substrings must be **identical** (exact same indices), meaning if you pick `source[a..b]` and `source[c..d]`, then `a == c` and `b == d`.

**Goal:** Find the minimum cost to convert `source` to `target`. If it's impossible, return `-1`.

**Important Notes:**
- Multiple transformation rules can have the same `original[i]` and `changed[i]` pairs with different costs.
- A substring can be transformed multiple times if the operations use the exact same indices.
- The transformations can involve substrings of any length, not just single characters.

**Example:** If `source = "abcd"`, `target = "acbe"`, and you have transformations that can change `"b"` to `"b"` at cost 100, `"bc"` to `"cb"` at cost 50, and `"d"` to `"e"` at cost 1, you could:
- Transform `"bc"` (at indices 1-2) to `"cb"` with cost 50
- Transform `"d"` (at index 3) to `"e"` with cost 1
- Total cost: 51

## Intuition

The key insight is that this is a [dynamic programming](/problems/dynamic_programming_intro) problem where we need to find the minimum cost to transform substrings of `source` into corresponding substrings of `target`.

First, let's think about what makes this problem complex:
1. We can transform substrings of any length, not just single characters
2. The same substring transformation might have multiple costs (we want the minimum)
3. Transformations can be chained - we might transform `"a"` to `"b"` and then `"b"` to `"c"`

To handle substring transformations efficiently, we can use a **[Trie](/problems/trie_intro)** data structure. The Trie allows us to:
- Store all possible transformation strings (`original` and `changed`) 
- Assign a unique integer ID to each distinct string
- Quickly check if a substring exists in our transformation rules while traversing the source string

Next, we need to handle **transitive transformations**. If we can transform string A to B with cost X, and B to C with cost Y, then we can transform A to C with cost X+Y. This is a [shortest path](/problems/shortest_path_unweight) problem between all pairs of strings. We can model this as a [graph](/problems/graph_intro) where:
- Each unique string in `original` and `changed` is a node
- Each transformation rule is a directed edge with its cost as weight
- We use **Floyd-Warshall algorithm** to find the minimum cost between any two strings

Finally, we need to decide **how to apply transformations** to convert `source` to `target`. We can use **memoized recursion** with the following approach:
- At each position `i` in the string, we have choices:
  - If `source[i] == target[i]`, we can skip this position without any cost
  - We can try to transform a substring starting at position `i`
- For each possible substring `source[i..j]`:
  - Check if both `source[i..j]` and `target[i..j]` exist in our [Trie](/problems/trie_intro)
  - If they do, the cost is `g[id_source][id_target]` (from Floyd-Warshall) plus the cost of transforming the rest of the string `dfs(j+1)`
- We take the minimum among all valid choices

The memoization ensures we don't recalculate the same subproblems, making the solution efficient.

**Learn more about [Graph](/problems/graph_intro), [Trie](/problems/trie_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Shortest Path](/problems/shortest_path_unweight) patterns.**

## Solution Approach

The solution consists of three main components: building a [Trie](/problems/trie_intro) for string mapping, computing shortest paths using Floyd-Warshall, and finding the minimum cost using memoized recursion.

**1. Building the [Trie](/problems/trie_intro) and String Mapping**

We create a [Trie](/problems/trie_intro) to store all unique strings from `original` and `changed` arrays. Each string gets a unique integer identifier:

```python
class Node:
    def __init__(self):
        self.children = [None] * 26  # For lowercase letters a-z
        self.v = -1  # String ID, -1 means no string ends here

def insert(w: str) -> int:
    node = root
    for c in w:
        i = ord(c) - ord("a")
        if node.children[i] is None:
            node.children[i] = Node()
        node = node.children[i]
    if node.v < 0:
        node.v = idx
        idx += 1
    return node.v
```

**2. Building the Cost [Graph](/problems/graph_intro)**

We initialize a 2D array `g` where `g[i][j]` represents the minimum cost to transform string with ID `i` to string with ID `j`:

```python
m = len(cost)
g = [[inf] * (m << 1) for _ in range(m << 1)]  # Size is 2*m to accommodate all possible strings
for i in range(m << 1):
    g[i][i] = 0  # Cost to transform a string to itself is 0

# Add direct transformation costs
for x, y, z in zip(original, changed, cost):
    x_id = insert(x)  # Get/assign ID for original string
    y_id = insert(y)  # Get/assign ID for changed string
    g[x_id][y_id] = min(g[x_id][y_id], z)  # Take minimum if duplicate rules exist
```

**3. Floyd-Warshall Algorithm**

We compute the [shortest path](/problems/shortest_path_unweight) between all pairs of strings to handle transitive transformations:

```python
for k in range(idx):  # idx is the total number of unique strings
    for i in range(idx):
        if g[i][k] >= inf:
            continue  # Optimization: skip if no path from i to k
        for j in range(idx):
            if g[i][k] + g[k][j] < g[i][j]:
                g[i][j] = g[i][k] + g[k][j]
```

**4. Memoized Recursion**

The core [dynamic programming](/problems/dynamic_programming_intro) function `dfs(i)` computes the minimum cost to transform `source[i:]` to `target[i:]`:

```python
@cache
def dfs(i: int) -> int:
    if i >= len(source):
        return 0  # Base case: reached end of string
  
    # Option 1: Skip if characters match
    res = dfs(i + 1) if source[i] == target[i] else inf
  
    # Option 2: Try transforming substrings starting at position i
    p = q = root  # Traverse both source and target in the [Trie](/problems/trie_intro)
    for j in range(i, len(source)):
        p = p.children[ord(source[j]) - ord("a")]
        q = q.children[ord(target[j]) - ord("a")]
      
        if p is None or q is None:
            break  # Substring doesn't exist in Trie
      
        if p.v >= 0 and q.v >= 0:  # Both substrings have valid IDs
            # Cost = transform source[i:j+1] to target[i:j+1] + solve rest
            res = min(res, dfs(j + 1) + g[p.v][q.v])
  
    return res
```

**5. Final Result**

```python
ans = dfs(0)
return -1 if ans >= inf else ans
```

The algorithm returns `-1` if transformation is impossible (cost remains `inf`), otherwise returns the minimum cost.

**Time Complexity:** 
- Building [Trie](/problems/trie_intro): `O(L)` where `L` is total length of all strings
- Floyd-Warshall: `O(n³)` where `n` is number of unique strings
- DFS with memoization: `O(|source|²)` states with `O(|source|)` work per state

**Space Complexity:** `O(n² + |source|)` for the [graph](/problems/graph_intro) and memoization cache

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- `source = "abc"`
- `target = "aec"` 
- `original = ["b", "bc"]`
- `changed = ["e", "ec"]`
- `cost = [2, 5]`

**Step 1: Build Trie and Assign IDs**

We insert all strings from `original` and `changed` into the Trie:
- Insert "b" → gets ID 0
- Insert "e" → gets ID 1  
- Insert "bc" → gets ID 2
- Insert "ec" → gets ID 3

Our Trie structure:
```
root
├── b (ID: 0)
│   └── c (ID: 2)
└── e (ID: 1)
    └── c (ID: 3)
```

**Step 2: Build Initial Cost Graph**

Initialize graph `g` with infinity, except diagonal (0s):
```
g[0][1] = 2  // "b" → "e" costs 2
g[2][3] = 5  // "bc" → "ec" costs 5
```

**Step 3: Floyd-Warshall (No Changes)**

Since there are no intermediate transformations possible, the graph remains unchanged.

**Step 4: Memoized Recursion - dfs(0)**

Starting at position 0:
- `source[0] = 'a'`, `target[0] = 'a'` → they match!
- Option 1: Skip position 0, compute `dfs(1)`

**Computing dfs(1):**

At position 1:
- `source[1] = 'b'`, `target[1] = 'e'` → they don't match
- Option 1: Skip (returns `inf` since characters don't match)
- Option 2: Try transformations starting at position 1:

  **Substring length 1:** `source[1:2] = "b"`, `target[1:2] = "e"`
  - Both exist in Trie with IDs 0 and 1
  - Cost = `g[0][1] + dfs(2)` = `2 + dfs(2)`

  **Substring length 2:** `source[1:3] = "bc"`, `target[1:3] = "ec"`
  - Both exist in Trie with IDs 2 and 3
  - Cost = `g[2][3] + dfs(3)` = `5 + dfs(3)`

**Computing dfs(2):**

At position 2:
- `source[2] = 'c'`, `target[2] = 'c'` → they match!
- Option 1: Skip position 2, compute `dfs(3)` = 0 (base case)
- Result: `dfs(2) = 0`

**Computing dfs(3):**
- Base case: position 3 >= length(3)
- Result: `dfs(3) = 0`

**Backtracking:**
- `dfs(1)` = min(inf, 2 + 0, 5 + 0) = min(inf, 2, 5) = 2
- `dfs(0)` = 0 + 2 = 2

**Final Answer: 2**

The optimal transformation:
1. Keep 'a' at position 0 (no cost)
2. Transform "b" at position 1 to "e" (cost 2)
3. Keep 'c' at position 2 (no cost)

Total cost: 2

## Python Solution

```python
from typing import List
from functools import cache
from math import inf


class TrieNode:
    """Trie node for storing string patterns"""
    __slots__ = ["children", "word_id"]
  
    def __init__(self):
        # Array of 26 children nodes (one for each lowercase letter)
        self.children: List[TrieNode | None] = [None] * 26
        # ID assigned to complete words ending at this node (-1 if not a word ending)
        self.word_id = -1


class Solution:
    def minimumCost(
        self,
        source: str,
        target: str,
        original: List[str],
        changed: List[str],
        cost: List[int],
    ) -> int:
        """
        Find minimum cost to transform source string to target string using given transformations.
      
        Args:
            source: Source string to transform
            target: Target string to achieve
            original: List of original substrings that can be replaced
            changed: List of replacement substrings
            cost: List of costs for each transformation
          
        Returns:
            Minimum cost to transform source to target, or -1 if impossible
        """
        num_transformations = len(cost)
        # Initialize adjacency matrix for shortest path calculation
        # Size is doubled to accommodate all possible unique strings
        max_nodes = num_transformations << 1  # num_transformations * 2
        graph = [[inf] * max_nodes for _ in range(max_nodes)]
      
        # Distance from any node to itself is 0
        for i in range(max_nodes):
            graph[i][i] = 0
      
        # Initialize trie root and word counter
        trie_root = TrieNode()
        word_counter = 0
      
        def insert_word(word: str) -> int:
            """
            Insert a word into the trie and return its unique ID.
          
            Args:
                word: String to insert into trie
              
            Returns:
                Unique ID assigned to this word
            """
            nonlocal word_counter
            current_node = trie_root
          
            # Traverse/build trie path for the word
            for char in word:
                char_index = ord(char) - ord('a')
                if current_node.children[char_index] is None:
                    current_node.children[char_index] = TrieNode()
                current_node = current_node.children[char_index]
          
            # Assign unique ID if this word hasn't been seen before
            if current_node.word_id < 0:
                current_node.word_id = word_counter
                word_counter += 1
              
            return current_node.word_id
      
        @cache
        def find_min_cost(position: int) -> int:
            """
            Find minimum cost to transform source[position:] to target[position:].
            Uses dynamic programming with memoization.
          
            Args:
                position: Current position in the source/target strings
              
            Returns:
                Minimum cost to complete transformation from this position
            """
            # Base case: reached end of strings
            if position >= len(source):
                return 0
          
            # Option 1: Skip current position if characters already match
            result = find_min_cost(position + 1) if source[position] == target[position] else inf
          
            # Option 2: Try all possible substring replacements starting at current position
            source_node = trie_root
            target_node = trie_root
          
            for end_pos in range(position, len(source)):
                # Navigate both tries simultaneously
                source_char_index = ord(source[end_pos]) - ord('a')
                target_char_index = ord(target[end_pos]) - ord('a')
              
                source_node = source_node.children[source_char_index]
                target_node = target_node.children[target_char_index]
              
                # Stop if either path doesn't exist in trie
                if source_node is None or target_node is None:
                    break
              
                # Skip if either substring is not a complete word in our dictionary
                if source_node.word_id < 0 or target_node.word_id < 0:
                    continue
              
                # Try this transformation and recursively solve the rest
                transformation_cost = graph[source_node.word_id][target_node.word_id]
                result = min(result, find_min_cost(end_pos + 1) + transformation_cost)
          
            return result
      
        # Build graph of transformation costs
        for orig_str, changed_str, trans_cost in zip(original, changed, cost):
            orig_id = insert_word(orig_str)
            changed_id = insert_word(changed_str)
            # Keep minimum cost if multiple transformations exist
            graph[orig_id][changed_id] = min(graph[orig_id][changed_id], trans_cost)
      
        # Floyd-Warshall algorithm to find shortest paths between all word pairs
        for k in range(word_counter):
            for i in range(word_counter):
                # Skip if no path through k from i
                if graph[i][k] >= inf:
                    continue
                for j in range(word_counter):
                    # Update shortest path from i to j through k
                    if graph[i][k] + graph[k][j] < graph[i][j]:
                        graph[i][j] = graph[i][k] + graph[k][j]
      
        # Find minimum cost starting from position 0
        min_cost = find_min_cost(0)
      
        # Return -1 if transformation is impossible, otherwise return the cost
        return -1 if min_cost >= inf else min_cost
```

## Time and Space Complexity

**Time Complexity:** `O(m^3 + n^2 + m × n)`

The time complexity breaks down into several components:

1. **Building the Trie and inserting strings:** Each string in `original` and `changed` arrays is inserted into the Trie. With `m` strings and assuming average/maximum string length is bounded, this takes `O(m × L)` where `L` is the maximum length of strings. In the context of this problem, `L` can be considered as part of `n` (length of source string), giving us `O(m × n)`.

2. **Floyd-Warshall algorithm:** The triple nested loop runs through all `idx` values where `idx ≤ m` (since we have at most `m` unique string nodes). This gives us `O(idx^3) = O(m^3)`.

3. **DFS with memoization:** The `dfs` function can be called with at most `n` different starting positions. For each call, the inner loop iterates from position `i` to `n`, and for each position, it traverses the Trie which takes `O(n)` in worst case. This gives us `O(n^2)` for the DFS traversal.

**Space Complexity:** `O(m^2 + m × n + n)`

The space complexity consists of:

1. **Graph matrix `g`:** A 2D array of size `(m << 1) × (m << 1)` which simplifies to `O(m^2)`.

2. **Trie structure:** The Trie stores all strings from `original` and `changed`. With `m` strings of maximum length bounded by `n`, the Trie uses `O(m × n)` space.

3. **Memoization cache:** The `@cache` decorator on `dfs` stores results for up to `n` different starting positions, using `O(n)` space.

4. **Recursion stack:** The maximum recursion depth is `n`, contributing `O(n)` to space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Overlapping Transformations**

**Pitfall:** One of the most critical mistakes is misunderstanding the constraint about overlapping transformations. Developers often incorrectly assume that any overlapping transformations are allowed, or conversely, that no overlapping is permitted at all.

**The Problem:** The rules state that transformations must be either:
- Completely disjoint (non-overlapping), OR  
- Exactly identical (same indices)

This means you CANNOT apply transformations like:
- Transform `source[0:3]` then `source[1:4]` (partial overlap)
- Transform `source[0:2]` then `source[0:3]` (one contains the other)

**Solution:** The recursive approach with memoization naturally handles this by processing the string left-to-right. Once we transform a substring starting at position `i` and ending at position `j`, we recursively solve for position `j+1`, ensuring no improper overlaps occur.

### 2. **Missing Transitive Transformations**

**Pitfall:** Only considering direct transformations from the input arrays and missing that you can chain transformations together.

**Example:** If you can transform "ab" → "cd" (cost 10) and "cd" → "ef" (cost 5), you should be able to transform "ab" → "ef" with total cost 15, even if this direct transformation isn't in the input.

**Solution:** Use Floyd-Warshall algorithm to compute the shortest path between all pairs of strings, capturing all possible transformation chains:

```python
for k in range(word_counter):
    for i in range(word_counter):
        if graph[i][k] >= inf:
            continue
        for j in range(word_counter):
            graph[i][j] = min(graph[i][j], graph[i][k] + graph[k][j])
```

### 3. **Insufficient Graph Size Allocation**

**Pitfall:** Allocating the graph size based only on the length of the `original` array, forgetting that strings in `changed` might be unique and not appear in `original`.

**Example Problem:**
```python
# WRONG: Only accounts for strings in original
graph = [[inf] * len(original) for _ in range(len(original))]

# This fails when changed contains strings not in original
```

**Solution:** Allocate the graph to handle all possible unique strings from both arrays:

```python
max_nodes = len(cost) * 2  # Conservative upper bound
graph = [[inf] * max_nodes for _ in range(max_nodes)]
```

### 4. **Not Handling Duplicate Transformation Rules**

**Pitfall:** When multiple rules exist for the same transformation (same `original[i]` and `changed[i]` pair but different costs), incorrectly overwriting instead of taking the minimum cost.

**Example:**
```python
# WRONG: Overwrites previous cost
graph[orig_id][changed_id] = trans_cost

# CORRECT: Keeps minimum cost
graph[orig_id][changed_id] = min(graph[orig_id][changed_id], trans_cost)
```

### 5. **Forgetting to Check Valid Word IDs in Trie**

**Pitfall:** During the DFS traversal, attempting to use transformation costs for substrings that don't exist as complete words in the transformation rules.

**Problem Code:**
```python
# WRONG: Doesn't verify if substrings are valid transformation endpoints
for j in range(position, len(source)):
    source_node = source_node.children[...]
    target_node = target_node.children[...]
    # Immediately tries to use graph[source_node.word_id][target_node.word_id]
```

**Solution:** Always check if both nodes have valid word IDs before attempting transformation:

```python
if source_node.word_id < 0 or target_node.word_id < 0:
    continue  # Skip if not valid transformation endpoints
```

### 6. **Early Termination in Substring Search**

**Pitfall:** Not properly handling the case where we should continue checking longer substrings even if shorter ones don't have valid transformations.

**Solution:** Use `continue` instead of `break` when word IDs are invalid, only `break` when the trie path doesn't exist:

```python
if source_node is None or target_node is None:
    break  # No longer substring exists in trie
  
if source_node.word_id < 0 or target_node.word_id < 0:
    continue  # This substring isn't a valid word, but longer ones might be
```

