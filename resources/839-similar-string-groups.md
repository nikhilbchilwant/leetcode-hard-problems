# 839. Similar String Groups

> Source: https://algo.monster/liteproblems/839

## Problem Statement

You are given a list of strings where every string is an anagram of every other string (they contain the same characters but possibly in different order).

Two strings `X` and `Y` are considered **similar** if:
- They are identical, OR
- We can make them equal by swapping at most two letters at distinct positions in string `X`

For example:
- `"tars"` and `"rats"` are similar (swap the characters at positions 0 and 2: `t` and `r`)
- `"rats"` and `"arts"` are similar (swap the characters at positions 0 and 1: `r` and `a`)
- `"star"` is NOT similar to `"tars"`, `"rats"`, or `"arts"`

Strings form **groups** based on similarity. A string belongs to a group if it is similar to at least one other string in that group. This creates a transitive relationship - if string A is similar to B, and B is similar to C, then A and C belong to the same group even if A and C are not directly similar.

From the example above:
- `{"tars", "rats", "arts"}` form one group (even though `"tars"` and `"arts"` are not directly similar, they are connected through `"rats"`)
- `{"star"}` forms another group by itself

Your task is to determine how many such groups exist in the given list of anagram strings.

## Intuition

The key insight is recognizing that this is fundamentally a graph connectivity problem in disguise. Each string is like a person in a social network, and two people are "friends" if their names (strings) are similar enough (differ by at most 2 characters). We want to find how many friend groups exist where everyone in a group is connected through a chain of friendships.

Consider the example with `"tars"`, `"rats"`, `"arts"`, and `"star"`. Even though `"tars"` and `"arts"` aren't directly similar, they're in the same group because both are similar to `"rats"`. This transitivity property is exactly what connected components in a graph represent.

Since all strings are anagrams, comparing any two strings for similarity is straightforward - we just count how many positions have different characters. If this count is 0 (identical) or 2 (one swap away), they're similar.

The challenge is efficiently grouping these strings. We could use DFS to explore each connected component, but Union-Find is particularly elegant here because:

1. We're iterating through pairs of strings anyway to check similarity
2. When we find two similar strings, we simply union them into the same group
3. The number of successful unions tells us how many times we merged groups
4. Starting with `n` individual groups (one per string), each successful union reduces the group count by 1

The beauty of this approach is that Union-Find handles the transitivity automatically. When we union `"tars"` with `"rats"`, and later union `"rats"` with `"arts"`, the data structure ensures all three end up in the same group without us explicitly checking if `"tars"` and `"arts"` should be together.

The final answer is simply the number of remaining distinct groups after all possible unions have been performed.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro) and [Union Find](/problems/dsu_intro) patterns.**

## Solution Approach

The solution implements Union-Find (Disjoint Set Union) with path compression and union by size optimization to efficiently manage the grouping of similar strings.

**Union-Find Data Structure Setup:**

The `UnionFind` class maintains two arrays (named `parent` and `component_size` in the Python solution):
- `p`: Parent array where `p[x]` stores the parent of node `x`. Initially, each node is its own parent (`p[i] = i`)
- `size`: Size array tracking the size of each component for union by size optimization

The `find` method implements path compression - when finding the root of a node, it updates all nodes along the path to point directly to the root, flattening the tree structure for faster future lookups.

The `union` method merges two components by:
1. Finding the roots of both nodes using `find`
2. If they're already in the same component, return `False`
3. Otherwise, attach the smaller component to the larger one (union by size) to keep the tree balanced
4. Return `True` to indicate a successful merge

**Main Algorithm:**

1. Initialize variables:
   - `n` = number of strings (the Python solution keeps the group counter in `num_groups`, starting at `num_strings`)
   - `m` = length of each string (`string_length`)
   - Create a Union-Find structure with `n` nodes (one for each string)

2. For each pair of strings `(i, s)` and `(j, t)` where `j < i`:
   - Count the number of differing positions: `sum(s[k] != t[k] for k in range(m))`
   - If the count is ≤ 2, the strings are similar
   - Attempt to union indices `i` and `j`
   - If union is successful (they weren't already in the same group), decrement `n` by 1

3. Return `n` as the final number of groups

**Why this works:**

Starting with `n` groups (each string in its own group), every successful union operation merges two groups into one, reducing the total count by 1. The condition `sum(s[k] != t[k] for k in range(m)) <= 2` efficiently checks if two anagram strings are similar:
- 0 differences: strings are identical
- 2 differences: exactly one swap can make them equal
- More than 2: not similar

The nested loop structure `for i, s in enumerate(strs): for j, t in enumerate(strs[:i])` ensures we check each pair exactly once, avoiding redundant comparisons.

## Example Walkthrough

Let's walk through the algorithm with a small example: `strs = ["tars", "rats", "arts", "star"]`

**Initial Setup:**
- `n = 4` (number of groups, initially each string is its own group)
- `m = 4` (length of each string)
- Union-Find structure initialized with 4 nodes:
  - Parent array: `[0, 1, 2, 3]` (each node is its own parent)
  - Size array: `[1, 1, 1, 1]` (each group has size 1)

**Step-by-Step Comparison:**

1. **i=1, s="rats"**: Compare with all previous strings
   - Compare with j=0, t="tars":
     - Differences at positions: 0 (r≠t), 2 (t≠r) → 2 differences
     - Similar! Union(1, 0) succeeds
     - Parent array becomes: `[0, 0, 2, 3]` (node 1 now points to 0)
     - `n = 3` (one group merged)

2. **i=2, s="arts"**: Compare with all previous strings
   - Compare with j=0, t="tars":
     - Differences at positions: 0 (a≠t), 1 (r≠a), 2 (t≠r) → 3 differences
     - Not similar (>2 differences)
   - Compare with j=1, t="rats":
     - Differences at positions: 0 (a≠r), 1 (r≠a) → 2 differences
     - Similar! Union(2, 1): `find(2) = 2` (size 1) and `find(1) = 0` (size 2), so node 2 is attached under root 0
     - Parent array becomes: `[0, 0, 0, 3]`
     - `n = 2`

3. **i=3, s="star"**: Compare with all previous strings
   - Compare with j=0, t="tars": all 4 positions differ (s≠t, t≠a, a≠r, r≠s) → not similar
   - Compare with j=1, t="rats": all 4 positions differ (s≠r, t≠a, a≠t, r≠s) → not similar
   - Compare with j=2, t="arts": all 4 positions differ (s≠a, t≠r, a≠t, r≠s) → not similar

**Final Result:**
- Parent array: `[0, 0, 0, 3]`
- Groups: {0, 1, 2} and {3}
- Return `n = 2`

The algorithm correctly identifies 2 groups:
- Group 1: `["tars", "rats", "arts"]` - all connected through similarity
- Group 2: `["star"]` - isolated, not similar to any other string

## Python Solution

```python
from typing import List


class UnionFind:
    """
    Union-Find (Disjoint Set Union) data structure with path compression and union by size.
    """
  
    def __init__(self, n: int) -> None:
        """
        Initialize Union-Find structure with n elements.
      
        Args:
            n: Number of elements (0 to n-1)
        """
        # Each element is initially its own parent (self-loop indicates root)
        self.parent = list(range(n))
        # Track size of each component for union by size optimization
        self.component_size = [1] * n
  
    def find(self, x: int) -> int:
        """
        Find the root/representative of the component containing element x.
        Uses path compression to flatten the tree structure.
      
        Args:
            x: Element to find the root for
          
        Returns:
            Root element of the component containing x
        """
        if self.parent[x] != x:
            # Path compression: make every node point directly to root
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
  
    def union(self, a: int, b: int) -> bool:
        """
        Unite the components containing elements a and b.
        Uses union by size to keep the tree balanced.
      
        Args:
            a: First element
            b: Second element
          
        Returns:
            True if elements were in different components (union performed),
            False if already in same component
        """
        root_a = self.find(a)
        root_b = self.find(b)
      
        # Already in the same component
        if root_a == root_b:
            return False
      
        # Union by size: attach smaller tree under larger tree's root
        if self.component_size[root_a] > self.component_size[root_b]:
            self.parent[root_b] = root_a
            self.component_size[root_a] += self.component_size[root_b]
        else:
            self.parent[root_a] = root_b
            self.component_size[root_b] += self.component_size[root_a]
      
        return True


class Solution:
    def numSimilarGroups(self, strs: List[str]) -> int:
        """
        Find the number of groups of similar strings.
        Two strings are similar if they differ in at most 2 positions
        (can be made equal by swapping at most one pair of characters).
      
        Args:
            strs: List of strings to group
          
        Returns:
            Number of similar groups
        """
        num_strings = len(strs)
        string_length = len(strs[0])
      
        # Initialize Union-Find with one component per string
        union_find = UnionFind(num_strings)
      
        # Start with assumption that each string is its own group
        num_groups = num_strings
      
        # Compare each string with all previous strings
        for i, current_string in enumerate(strs):
            for j, previous_string in enumerate(strs[:i]):
                # Count positions where characters differ
                diff_count = sum(
                    current_string[k] != previous_string[k] 
                    for k in range(string_length)
                )
              
                # Strings are similar if they differ in at most 2 positions
                # (0 differences = identical, 2 differences = one swap away)
                if diff_count <= 2:
                    # If union is successful (strings were in different groups),
                    # decrease the total number of groups
                    if union_find.union(i, j):
                        num_groups -= 1
      
        return num_groups
```

## Time and Space Complexity

**Time Complexity:** `O(n² × (m + α(n)))`

The time complexity breaks down as follows:
- The outer loop iterates through all `n` strings: `O(n)`
- The inner loop iterates through up to `i` strings for each `i`, giving us `O(n²)` total iterations across both loops
- For each pair of strings being compared:
  - String comparison takes `O(m)` time to check character differences
  - The `union` operation calls `find` twice, each taking `O(α(n))` amortized time due to path compression
  - Total per pair: `O(m + α(n))`
- Overall: `O(n² × (m + α(n)))`

Where `n` is the number of strings, `m` is the length of each string, and `α(n)` is the inverse Ackermann function (effectively a very small constant in practice).

**Space Complexity:** `O(n)`

The space complexity consists of:
- UnionFind data structure stores two arrays:
  - Parent array `parent`: `O(n)`
  - Size array `component_size`: `O(n)`
- No additional significant space is used in the main algorithm
- Total: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Over-Checking Similarity for Anagrams**

A natural worry is that counting differing positions is not enough: two strings that differ in exactly 2 positions are only one swap apart if the characters at those positions are swapped. In general that is true. For example, `"abcd"` and `"efcd"` differ in exactly 2 positions but are not one swap apart.

In this problem, however, every string is an anagram of every other string, so the shown check is correct:
```python
diff_count = sum(s[k] != t[k] for k in range(len(s)))
if diff_count <= 2:  # correct when s and t are anagrams
```

**Why counting is enough here:** Suppose anagrams `s` and `t` differ only at positions `i` and `j`. All other positions match, so the two strings must have the same multiset of characters at positions `i` and `j`. Since `s[i] != t[i]`, the only possibility is `s[i] == t[j]` and `s[j] == t[i]`, which is exactly a swap. A count of 1 is impossible for anagrams, and a count of 0 means the strings are identical.

**When the full check is needed:** If you reuse this code for strings that are not guaranteed to be anagrams, verify the swap explicitly:
```python
def are_similar(s: str, t: str) -> bool:
    if s == t:
        return True
    diff_positions = [i for i in range(len(s)) if s[i] != t[i]]
    if len(diff_positions) != 2:
        return False
    i, j = diff_positions
    return s[i] == t[j] and s[j] == t[i]
```

### 2. **Pairwise Comparison When There Are Many Short Strings**

Comparing every pair costs `O(n² × m)`. Grouping strings by their sorted characters does not help here, because all strings are anagrams and would land in a single group. When `n` is much larger than `m`, a different approach is to generate every string reachable by one swap and look it up in a hash map, which costs `O(n × m³)` (`m²` swaps, each building a string of length `m`):

```python
def numSimilarGroups(self, strs: List[str]) -> int:
    n, m = len(strs), len(strs[0])
    uf = UnionFind(n)
    num_groups = n
    first_index = {}  # string -> index of its first occurrence
    for i, s in enumerate(strs):
        if s in first_index:  # identical strings are similar
            if uf.union(i, first_index[s]):
                num_groups -= 1
            continue
        first_index[s] = i
    for i, s in enumerate(strs):
        chars = list(s)
        for a in range(m):
            for b in range(a + 1, m):
                if chars[a] == chars[b]:
                    continue
                chars[a], chars[b] = chars[b], chars[a]
                j = first_index.get(''.join(chars))
                if j is not None and uf.union(i, j):
                    num_groups -= 1
                chars[a], chars[b] = chars[b], chars[a]
    return num_groups
```

Identical strings are linked in the first loop because a swap of two equal characters produces no new string. For the original constraints (`n` up to 300 and `m` up to 300), the pairwise solution is the better fit.

### 3. **Memory Issues with Large String Lists**

The Union-Find structure uses O(n) space, but the nested loop approach can cause memory issues when creating intermediate data structures.

**Solution:** Use generators and iterate efficiently:

```python
# Instead of creating a slice with strs[:i]
for j in range(i):  # More memory efficient
    if are_similar(strs[i], strs[j]):
        # Process similarity
```

### 4. **Not Handling Edge Cases**

**Common missed cases:**
- Empty list of strings
- Single string in the list
- All identical strings
- Strings of different lengths (though the problem states they're anagrams)

**Robust implementation:**
```python
def numSimilarGroups(self, strs: List[str]) -> int:
    if not strs:
        return 0
    if len(strs) == 1:
        return 1
  
    # Validate that all strings are anagrams (same length, same character frequency)
    if len(set(len(s) for s in strs)) > 1:
        raise ValueError("All strings must have the same length")
  
    # Continue with main algorithm...
```

Because all strings are anagrams of each other, the difference count used in the solution is a complete similarity test; the explicit swap check from pitfall 1 is only needed for inputs without that guarantee.

