# 2791. Count Paths That Can Form a Palindrome in a Tree

> Source: https://algo.monster/liteproblems/2791

## Problem Statement

You are given a rooted [tree](/problems/tree_intro) with `n` nodes numbered from `0` to `n - 1`, where node `0` is the root. The tree structure is represented by an array `parent` where `parent[i]` indicates the parent of node `i`. Since node `0` is the root, `parent[0] = -1`.

Each edge in the [tree](/problems/tree_intro) has a character assigned to it. These characters are given in a string `s` of length `n`, where `s[i]` represents the character on the edge connecting node `i` to its parent `parent[i]`. The character `s[0]` can be ignored since the root has no parent.

Your task is to count the number of node pairs `(u, v)` where `u < v` such that the characters on the path between nodes `u` and `v` can be rearranged to form a palindrome.

A key insight is that a string can be rearranged into a palindrome if and only if at most one character appears an odd number of times. For example:
- `"aab"` can be rearranged to `"aba"` (palindrome)
- `"abc"` cannot be rearranged into any palindrome
- `"aabbcc"` can be rearranged to `"abccba"` (palindrome)

The solution uses XOR operations and bit manipulation to track character frequencies efficiently. Each character is represented by a bit position (0-25 for 'a'-'z'). The XOR of all characters on a path gives us a [bitmask](/problems/bitmask_intro) where each set bit represents a character appearing an odd number of times. A path can form a palindrome if the XOR result has at most one bit set (i.e., it's either 0 or a power of 2).

## Intuition

The key insight starts with understanding when a string can be rearranged into a palindrome. A string forms a palindrome when characters can be paired up symmetrically, with at most one character in the middle. This means at most one character can appear an odd number of times.

Instead of actually finding paths between every pair of nodes and checking character frequencies (which would be inefficient), we can use a clever observation: in a [tree](/problems/tree_intro), the path from node `u` to node `v` can be decomposed as:
- Path from root to `u`
- Path from root to `v`
- Minus the overlapping path from root to their lowest common ancestor (LCA)

This is where XOR becomes brilliant. XOR has the property that `a ^ a = 0`, meaning any character that appears an even number of times will cancel out. So if we XOR all characters on a path, we get a [bitmask](/problems/bitmask_intro) where each set bit represents a character appearing an odd number of times.

For each character 'a' to 'z', we assign a bit position (0 to 25). When we encounter a character, we XOR with `1 << (character - 'a')`. This flips the corresponding bit, effectively tracking odd/even occurrences.

During DFS traversal from the root, we maintain the XOR value from root to current node. When we're at node `v` with XOR value `xor_v`, we want to find how many previously visited nodes `u` can form a palindromic path with `v`. 

The path from `u` to `v` can form a palindrome if:
1. `xor_u ^ xor_v = 0` (all characters appear even times)
2. `xor_u ^ xor_v = 2^k` for some k (exactly one character appears odd times)

Rearranging: we need `xor_u = xor_v` or `xor_u = xor_v ^ (1 << k)` for some k.

So at each node, we count how many previous nodes have XOR values that match these conditions. We use a frequency map to track XOR values we've seen, allowing O(1) lookups. This transforms a potentially O(n²) problem into an O(n) solution with smart preprocessing and XOR properties.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro), [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The implementation uses DFS with XOR-based state tracking to efficiently count palindromic paths:

**Data Structures:**
- `g`: A graph adjacency list where `g[parent]` contains tuples of `(child, [bitmask](/problems/bitmask_intro))`. The bitmask represents `1 << (character - 'a')` for the edge character.
- `cnt`: A counter/frequency map tracking how many times each XOR value has been seen during traversal.
- `ans`: Accumulates the total count of valid palindromic paths.

**Building the Graph:**
```python
g = defaultdict(list)
for i in range(1, n):
    p = parent[i]
    g[p].append((i, 1 << (ord(s[i]) - ord('a'))))
```
For each node `i` (except root), we add an edge from its parent to itself with a [bitmask](/problems/bitmask_intro) representing the edge character.

**DFS Traversal:**
```python
def dfs(i: int, xor: int):
    for j, v in g[i]:
        x = xor ^ v  # XOR value from root to child j
```
The function maintains `xor` as the cumulative XOR from root to current node. For each child `j`, we compute its XOR value `x` by XORing the parent's value with the edge [bitmask](/problems/bitmask_intro) `v`.

**Counting Valid Paths:**
```python
ans += cnt[x]  # Paths with all even character counts
for k in range(26):
    ans += cnt[x ^ (1 << k)]  # Paths with exactly one odd character
```
For the current node with XOR value `x`, we count:
1. Previous nodes with the same XOR value (path between them has all even counts)
2. Previous nodes whose XOR differs by exactly one bit (path has exactly one odd character)

**Recording Current Node:**
```python
cnt[x] += 1
dfs(j, x)
```
After counting paths ending at this node, we record its XOR value in `cnt` for future nodes to reference, then recursively process its subtree.

**Initialization:**
```python
cnt = Counter({0: 1})
dfs(0, 0)
```
We initialize `cnt[0] = 1` to handle paths starting from the root (which has XOR value 0). The DFS starts from root node 0 with initial XOR value 0.

The algorithm runs in O(n × 26) = O(n) time, where the factor of 26 comes from checking all possible single-bit differences for palindrome conditions. Space complexity is O(n) for storing the graph and XOR frequency map.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example Tree:**
```
parent = [-1, 0, 0, 1]
s = "abab"
```

This gives us the following tree structure:
```
      0 (root)
     / \
   a/   \b
   1     2
  /
 b/
 3
```

**Step 1: Build the Graph**
```python
g = {
    0: [(1, 1<<0), (2, 1<<1)],  # 0 -> 1 with 'a' (bit 0), 0 -> 2 with 'b' (bit 1)
    1: [(3, 1<<1)]               # 1 -> 3 with 'b' (bit 1)
}
```
- Edge 0→1 has character 'a', represented as bitmask `1<<0 = 1`
- Edge 0→2 has character 'b', represented as bitmask `1<<1 = 2`  
- Edge 1→3 has character 'b', represented as bitmask `1<<1 = 2`

**Step 2: DFS Traversal**

Initialize: `cnt = {0: 1}`, `ans = 0`

**Visit node 0 (root):**
- Current XOR = 0
- Process child 1:
  - `x = 0 ^ 1 = 1` (XOR for path root→1 is 1, meaning 'a' appears odd times)
  - Count valid paths ending at node 1:
    - `cnt[1] = 0` (no previous node with XOR=1)
    - Check single-bit differences:
      - `cnt[1 ^ 1<<0] = cnt[0] = 1` ✓ (path 0→1 has only 'a')
      - `cnt[1 ^ 1<<1] = cnt[3] = 0`
      - ... (other bits give 0)
    - `ans += 1` (total: 1)
  - Record: `cnt[1] = 1`
  - Recursively visit node 1:
    - Process child 3:
      - `x = 1 ^ 2 = 3` (XOR for path root→3 is 3, bits 0 and 1 set)
      - Count valid paths:
        - `cnt[3] = 0`
        - `cnt[3 ^ 1<<0] = cnt[2] = 0`
        - `cnt[3 ^ 1<<1] = cnt[1] = 1` ✓ (path 1→3 has only 'b')
        - ... (other bits give 0)
      - `ans += 1` (total: 2)
      - Record: `cnt[3] = 1`

- Process child 2:
  - `x = 0 ^ 2 = 2` (XOR for path root→2 is 2, meaning 'b' appears odd times)
  - Count valid paths:
    - `cnt[2] = 0`
    - `cnt[2 ^ 1<<0] = cnt[3] = 1` ✓ (path 2→3 has "ab", can form "aba")
    - `cnt[2 ^ 1<<1] = cnt[0] = 1` ✓ (path 0→2 has only 'b')
    - ... (other bits give 0)
  - `ans += 2` (total: 4)
  - Record: `cnt[2] = 1`

**Final Result: 4 valid pairs**

The valid pairs are:
1. (0,1): path has "a" → palindrome
2. (0,2): path has "b" → palindrome  
3. (1,3): path has "b" → palindrome
4. (2,3): path has "ab" → can rearrange to "aba"

The algorithm correctly identifies all pairs where the path characters can form a palindrome by tracking XOR values and checking for at most one odd character count.

## Python Solution

```python
from typing import List
from collections import defaultdict, Counter

class Solution:
    def countPalindromePaths(self, parent: List[int], s: str) -> int:
        # Build adjacency list for the tree
        # Each edge stores (child_node, bitmask_of_character)
        tree = defaultdict(list)
        n = len(parent)
      
        # Build tree from parent array (node 0 is root)
        for child in range(1, n):
            parent_node = parent[child]
            # Convert character to bitmask (1 << position_in_alphabet)
            char_bitmask = 1 << (ord(s[child]) - ord('a'))
            tree[parent_node].append((child, char_bitmask))
      
        # Initialize result counter
        palindrome_path_count = 0
      
        # Track XOR values from root to each node
        # XOR tracks parity of character frequencies
        xor_frequency = Counter({0: 1})  # Start with empty path
      
        def dfs(node: int, path_xor: int) -> None:
            """
            DFS traversal to count palindromic paths.
          
            Args:
                node: Current node in traversal
                path_xor: XOR of all characters from root to current node
            """
            nonlocal palindrome_path_count
          
            # Explore all children of current node
            for child, char_mask in tree[node]:
                # Update XOR for path from root to child
                child_path_xor = path_xor ^ char_mask
              
                # Count palindromic paths ending at current child:
              
                # Case 1: Path with all even character frequencies
                # (XOR equals to some previous XOR value)
                palindrome_path_count += xor_frequency[child_path_xor]
              
                # Case 2: Path with exactly one odd character frequency
                # (XOR differs by exactly one bit from previous XOR values)
                for bit_position in range(26):  # Check each letter a-z
                    target_xor = child_path_xor ^ (1 << bit_position)
                    palindrome_path_count += xor_frequency[target_xor]
              
                # Add current path XOR to frequency map
                xor_frequency[child_path_xor] += 1
              
                # Recursively process subtree rooted at child
                dfs(child, child_path_xor)
      
        # Start DFS from root (node 0) with empty XOR
        dfs(0, 0)
      
        return palindrome_path_count
```

## Time and Space Complexity

**Time Complexity: `O(n × 26) = O(n)`**

The algorithm performs a DFS traversal of the tree starting from node 0. For each node visited:
- We iterate through all its children (each node is visited exactly once)
- For each child, we check the current XOR value against the counter: `O(1)`
- We check 26 possible single-bit differences (for each letter a-z): `O(26)`
- We update the counter: `O(1)`
- We recursively call DFS on the child

Since each of the `n` nodes is visited exactly once, and for each node we perform `O(26)` operations, the total time complexity is `O(n × 26) = O(n)`.

**Space Complexity: `O(n)`**

The space complexity consists of:
- Graph adjacency list `g`: stores at most `n-1` edges (tree structure), so `O(n)`
- Counter `cnt`: in the worst case, each path from root could have a unique XOR value, storing up to `n` distinct values, so `O(n)`
- Recursion stack depth: in the worst case (skewed tree), the depth could be `O(n)`
- XOR values and other variables: `O(1)`

The total space complexity is `O(n) + O(n) + O(n) = O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Initialize XOR Frequency Map with {0: 1}**

**The Pitfall:**
A common mistake is initializing the XOR frequency counter as empty (`Counter()`) instead of `Counter({0: 1})`. This causes incorrect counting of paths that start from the root node.

**Why It Happens:**
When we reach a node with XOR value `x` and want to count palindromic paths, we look for previous nodes with matching XOR values. For paths starting from the root, we need to match with XOR value 0 (the root's XOR). Without `{0: 1}` initialization, these paths won't be counted.

**Incorrect Code:**
```python
xor_frequency = Counter()  # Missing {0: 1}
```

**Correct Solution:**
```python
xor_frequency = Counter({0: 1})  # Properly handle root paths
```

### 2. **Updating XOR Frequency Before Counting**

**The Pitfall:**
Adding the current node's XOR value to the frequency map before counting palindromic paths leads to counting invalid self-loops.

**Why It Happens:**
If we update `xor_frequency[child_path_xor] += 1` before counting, we might count a path from a node to itself (which isn't valid since we need `u < v` with distinct nodes).

**Incorrect Code:**
```python
for child, char_mask in tree[node]:
    child_path_xor = path_xor ^ char_mask
    xor_frequency[child_path_xor] += 1  # Updated too early!
    palindrome_path_count += xor_frequency[child_path_xor]
    # ... rest of counting logic
```

**Correct Solution:**
```python
for child, char_mask in tree[node]:
    child_path_xor = path_xor ^ char_mask
    # Count first
    palindrome_path_count += xor_frequency[child_path_xor]
    for bit_position in range(26):
        palindrome_path_count += xor_frequency[child_path_xor ^ (1 << bit_position)]
    # Update after counting
    xor_frequency[child_path_xor] += 1
```

### 3. **Incorrectly Handling the Root Node's Character**

**The Pitfall:**
Including `s[0]` in the XOR calculation or trying to process it as an edge character. The problem states that `s[0]` should be ignored since the root has no parent.

**Why It Happens:**
It's easy to overlook that `s[i]` represents the character on the edge from node `i` to its parent, not the character "at" node `i`.

**Incorrect Code:**
```python
# Wrong: Starting loop from 0
for i in range(0, n):  
    p = parent[i]
    if p != -1:  # Still wrong logic
        g[p].append((i, 1 << (ord(s[i]) - ord('a'))))
```

**Correct Solution:**
```python
# Start from 1, skipping the root
for i in range(1, n):
    p = parent[i]
    g[p].append((i, 1 << (ord(s[i]) - ord('a'))))
```

### 4. **Not Considering All 26 Possible Single-Bit Differences**

**The Pitfall:**
Only checking a subset of possible characters when looking for paths with exactly one odd character frequency, or using the wrong range.

**Why It Happens:**
Developers might optimize prematurely by only checking characters that appear in the string, but this misses valid palindromic paths.

**Incorrect Code:**
```python
# Only checking characters that we've seen
unique_chars = set(s)
for char in unique_chars:
    bit_pos = ord(char) - ord('a')
    palindrome_path_count += xor_frequency[child_path_xor ^ (1 << bit_pos)]
```

**Correct Solution:**
```python
# Check all 26 possible positions
for bit_position in range(26):
    target_xor = child_path_xor ^ (1 << bit_position)
    palindrome_path_count += xor_frequency[target_xor]
```

This ensures we find all valid palindromic paths, even when the differing character hasn't been seen yet in our traversal.

