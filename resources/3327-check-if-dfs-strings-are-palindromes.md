# 3327. Check if DFS Strings Are Palindromes

> Source: https://algo.monster/liteproblems/3327

## Problem Statement

You have a [tree](/problems/tree_intro) with `n` nodes numbered from `0` to `n - 1`, where node `0` is the root. The tree structure is given by an array `parent` where `parent[i]` indicates the parent of node `i`. Since node `0` is the root, `parent[0] = -1`.

Each node `i` has a character `s[i]` associated with it.

The problem defines a special DFS traversal that builds a string `dfsStr`:
- Starting from any node `x`, the DFS function:
  1. First recursively visits all children of `x` in increasing order of their node numbers
  2. After visiting all children, appends the character `s[x]` to `dfsStr`

Your task is to determine, for each node `i` from `0` to `n - 1`, whether starting a DFS from that node produces a palindrome string.

The output should be a boolean array `answer` where `answer[i]` is `true` if starting DFS from node `i` produces a palindrome in `dfsStr`, and `false` otherwise.

**Example of DFS traversal:**
If we have a [tree](/problems/tree_intro) where node 0 has children 1 and 2, and node 1 has child 3:
- DFS from node 0 would visit: node 1 → node 3 → append `s[3]` → append `s[1]` → node 2 → append `s[2]` → append `s[0]`
- The resulting `dfsStr` would be: `s[3]` + `s[1]` + `s[2]` + `s[0]`

## Intuition

The key insight is that we don't need to run DFS from every single node separately - that would be inefficient. Instead, we can observe that when we run DFS from any node, we're essentially getting a substring of what we'd get from a complete DFS starting from a higher ancestor.

Think about it this way: when we perform DFS from the root (node 0), we visit every node in the [tree](/problems/tree_intro) and build the complete `dfsStr`. Now, if we were to start DFS from any other node `i`, we'd only visit node `i` and its subtree - this produces a substring that already exists within the complete `dfsStr`.

This leads to our first breakthrough: **we only need to run DFS once from the root**, and during this traversal, we can track where each subtree's contribution starts and ends in the final string. For each node `i`, we record its interval `[l, r]` representing the positions in `dfsStr` that would be generated if we started DFS from node `i`.

The second challenge is checking if each substring is a palindrome. A naive approach would check character by character for each substring, but this is slow. Here's where string hashing comes in - it's a technique that converts strings into numerical values (hashes) for fast comparison.

For palindrome checking, we need a clever trick: a string is a palindrome if its first half matches its second half when reversed. So we:
1. Compute hash values for the forward string (`dfsStr`)
2. Compute hash values for the reversed string (`dfsStr[::-1]`)
3. For each substring defined by interval `[l, r]`, compare the hash of its first half with the hash of the corresponding position in the reversed string

The beauty of this approach is that we transform an `O(n²)` problem (running DFS from each node and checking palindromes) into an `O(n)` solution (one DFS traversal plus `O(1)` palindrome checks using precomputed hashes).

The polynomial rolling hash with base `13331` and modulo `998244353` ensures very low collision probability while keeping computations efficient.

**Learn more about [Tree](/problems/tree_intro) and [Depth-First Search](/problems/dfs_intro) patterns.**

## Solution Approach

The implementation consists of three main components: DFS traversal, string hashing, and palindrome verification.

**Step 1: Build the [Tree](/problems/tree_intro) Structure**

First, we construct an adjacency list representation of the [tree](/problems/tree_intro) from the `parent` array:
```python
g = [[] for _ in range(n)]
for i in range(1, n):
    g[parent[i]].append(i)
```
This creates a list where `g[i]` contains all children of node `i`.

**Step 2: DFS Traversal to Build dfsStr**

We perform a single DFS from the root to build the complete `dfsStr` and record position intervals:
```python
def dfs(i: int):
    l = len(dfsStr) + 1  # Start position (1-indexed)
    for j in g[i]:       # Visit children in order
        dfs(j)
    dfsStr.append(s[i])  # Append current node's character
    r = len(dfsStr)      # End position
    pos[i] = (l, r)      # Store interval for node i
```

The DFS follows the problem's specification:
- Visit all children recursively in increasing order (already sorted in adjacency list)
- After visiting children, append the current node's character
- Track the interval `[l, r]` where node `i`'s subtree contributes to `dfsStr`

**Step 3: String Hashing Setup**

The `Hashing` class implements polynomial rolling hash:
```python
self.h[i] = (self.h[i - 1] * base + ord(s[i - 1])) % mod
self.p[i] = (self.p[i - 1] * base) % mod
```

- `h[i]` stores the hash value of prefix `s[0:i]`
- `p[i]` stores `base^i % mod` for efficient hash computation
- The `query(l, r)` method returns the hash of substring `s[l-1:r]` using the formula:
  `(h[r] - h[l-1] * p[r-l+1]) % mod`

We create two hash objects:
- `h1` for the forward string `dfsStr`
- `h2` for the reversed string `dfsStr[::-1]`

**Step 4: Palindrome Checking**

For each node `i`, we check if its substring is a palindrome:
```python
for i in range(n):
    l, r = pos[i]
    k = r - l + 1  # Length of substring
    v1 = h1.query(l, l + k // 2 - 1)  # First half hash
    v2 = h2.query(n - r + 1, n - r + 1 + k // 2 - 1)  # Corresponding reversed half
    ans.append(v1 == v2)
```

The palindrome check works by:
1. Getting the substring length `k`
2. Computing hash of the first half from the forward string
3. Computing hash of the corresponding position in the reversed string
4. If both hashes match, the substring is a palindrome

The key insight is that for a palindrome, reading the first half forward should match reading the second half backward. By using the reversed string's hash, we can check this condition in `O(1)` time.

**Time Complexity:** `O(n)` - one DFS traversal plus `O(n)` hash computations
**Space Complexity:** `O(n)` - for storing the [tree](/problems/tree_intro), dfsStr, and hash arrays

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- `n = 4` nodes (0, 1, 2, 3)
- `parent = [-1, 0, 0, 1]` (tree structure: 0 is root, 1 and 2 are children of 0, 3 is child of 1)
- `s = "abba"` (node 0 has 'a', node 1 has 'b', node 2 has 'b', node 3 has 'a')

**Step 1: Build Tree Structure**
```
Adjacency list:
g[0] = [1, 2]  (node 0's children)
g[1] = [3]     (node 1's children)
g[2] = []      (node 2 has no children)
g[3] = []      (node 3 has no children)
```

**Step 2: DFS Traversal from Root**
Starting DFS from node 0:
1. Visit node 1 (child of 0)
   - Visit node 3 (child of 1)
     - No children, append s[3]='a' → dfsStr = "a", pos[3] = (1,1)
   - Children done, append s[1]='b' → dfsStr = "ab", pos[1] = (1,2)
2. Visit node 2 (child of 0)
   - No children, append s[2]='b' → dfsStr = "abb", pos[2] = (3,3)
3. Children done, append s[0]='a' → dfsStr = "abba", pos[0] = (1,4)

**Final dfsStr = "abba"**

**Position intervals:**
- pos[0] = (1,4) → substring "abba"
- pos[1] = (1,2) → substring "ab"
- pos[2] = (3,3) → substring "b"
- pos[3] = (1,1) → substring "a"

**Step 3: String Hashing**
- Forward hash h1 on "abba"
- Reverse hash h2 on "abba" reversed = "abba"

**Step 4: Palindrome Checking**

For node 0: interval (1,4), length k=4
- First half (positions 1-2): "ab"
- Check against reversed positions: comparing "ab" with "ba" reversed
- Hash values match → "abba" is palindrome → answer[0] = true

For node 1: interval (1,2), length k=2
- First half (position 1): "a"
- Check against reversed position: comparing "a" with "b"
- Hash values don't match → "ab" is not palindrome → answer[1] = false

For node 2: interval (3,3), length k=1
- Single character "b" is always palindrome → answer[2] = true

For node 3: interval (1,1), length k=1
- Single character "a" is always palindrome → answer[3] = true

**Final Answer: [true, false, true, true]**

This example demonstrates how we build the complete DFS string once, track position intervals for each node's subtree, and use hashing to efficiently check palindromes without repeatedly running DFS from each node.

## Python Solution

```python
from typing import List


class Hashing:
    """Rolling hash implementation for string hashing."""
    __slots__ = ["mod", "hash_values", "powers"]

    def __init__(self, string: List[str], base: int, mod: int):
        """
        Initialize the hashing object with precomputed hash values and powers.
      
        Args:
            string: List of characters to hash
            base: Base for polynomial rolling hash
            mod: Modulo value to prevent overflow
        """
        self.mod = mod
        self.hash_values = [0] * (len(string) + 1)
        self.powers = [1] * (len(string) + 1)
      
        # Precompute hash values and powers of base
        for i in range(1, len(string) + 1):
            self.hash_values[i] = (self.hash_values[i - 1] * base + ord(string[i - 1])) % mod
            self.powers[i] = (self.powers[i - 1] * base) % mod

    def query(self, left: int, right: int) -> int:
        """
        Get the hash value of substring from position left to right (1-indexed).
      
        Args:
            left: Starting position (1-indexed)
            right: Ending position (1-indexed)
          
        Returns:
            Hash value of the substring
        """
        return (self.hash_values[right] - self.hash_values[left - 1] * self.powers[right - left + 1]) % self.mod


class Solution:
    def findAnswer(self, parent: List[int], s: str) -> List[bool]:
        """
        Check if each node's subtree forms a palindrome when traversed in DFS order.
      
        Args:
            parent: Parent array where parent[i] is the parent of node i
            s: String where s[i] is the character at node i
          
        Returns:
            List of booleans indicating if each node's subtree is a palindrome
        """
        def dfs(node: int) -> None:
            """
            Perform DFS traversal and build the DFS string while recording node positions.
          
            Args:
                node: Current node being visited
            """
            # Record the starting position for this node's subtree
            left_pos = len(dfs_string) + 1
          
            # Recursively visit all children
            for child in adjacency_list[node]:
                dfs(child)
          
            # Add current node's character to DFS string
            dfs_string.append(s[node])
          
            # Record the ending position for this node's subtree
            right_pos = len(dfs_string)
            node_positions[node] = (left_pos, right_pos)

        # Build adjacency list representation of the tree
        n = len(s)
        adjacency_list = [[] for _ in range(n)]
        for i in range(1, n):
            adjacency_list[parent[i]].append(i)
      
        # Perform DFS to build the string and record positions
        dfs_string = []
        node_positions = {}
        dfs(0)
      
        # Initialize hashing parameters
        BASE = 13331
        MOD = 998244353
      
        # Create hash objects for forward and reverse strings
        forward_hash = Hashing(dfs_string, BASE, MOD)
        reverse_hash = Hashing(dfs_string[::-1], BASE, MOD)
      
        # Check if each node's subtree forms a palindrome
        result = []
        for i in range(n):
            left, right = node_positions[i]
            subtree_length = right - left + 1
          
            # Compare first half with reversed second half
            first_half_hash = forward_hash.query(left, left + subtree_length // 2 - 1)
            second_half_reversed_hash = reverse_hash.query(n - right + 1, n - right + 1 + subtree_length // 2 - 1)
          
            # Check if the hashes match (indicating a palindrome)
            result.append(first_half_hash == second_half_reversed_hash)
      
        return result
```

## Time and Space Complexity

**Time Complexity: `O(n)`**

The algorithm consists of several main components:
- Building the adjacency list `g` from the parent array takes `O(n)` time
- The DFS traversal visits each node exactly once, appending each character to `dfsStr`, which takes `O(n)` time total
- Initializing the two `Hashing` objects (`h1` and `h2`) each takes `O(n)` time to compute the prefix hashes and powers
- The final loop iterates through all `n` nodes, and for each node:
  - Retrieves the position bounds in `O(1)`
  - Performs two hash queries, each in `O(1)` time
  - Compares the hash values in `O(1)`

Therefore, the overall time complexity is `O(n) + O(n) + O(n) + O(n) + O(n) = O(n)`.

**Space Complexity: `O(n)`**

The space usage includes:
- The adjacency list `g` storing the tree structure: `O(n)`
- The `dfsStr` list containing all characters from the DFS traversal: `O(n)`
- The `pos` dictionary storing position pairs for each node: `O(n)`
- Two `Hashing` objects, each maintaining arrays `h` and `p` of size `n+1`: `O(n)` each
- The `ans` list storing boolean results: `O(n)`
- The recursion stack for DFS in the worst case (skewed tree): `O(n)`

The total space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. Hash Collision Leading to False Positives

**The Pitfall:**
The current implementation uses a single hash function with one base and modulo value. While polynomial rolling hash is efficient, it can produce collisions where different strings yield the same hash value, causing the algorithm to incorrectly identify non-palindromes as palindromes.

**Example Scenario:**
Two different substrings might hash to the same value by chance, especially with a single modulo operation. This is particularly problematic in competitive programming or when dealing with adversarial inputs.

**Solution:**
Use double hashing with two different prime moduli to significantly reduce collision probability:

```python
class DoubleHashing:
    def __init__(self, string: List[str], base: int):
        self.mod1 = 10**9 + 7
        self.mod2 = 998244353
        n = len(string)
      
        self.h1 = [0] * (n + 1)
        self.h2 = [0] * (n + 1)
        self.p1 = [1] * (n + 1)
        self.p2 = [1] * (n + 1)
      
        for i in range(1, n + 1):
            self.h1[i] = (self.h1[i - 1] * base + ord(string[i - 1])) % self.mod1
            self.h2[i] = (self.h2[i - 1] * base + ord(string[i - 1])) % self.mod2
            self.p1[i] = (self.p1[i - 1] * base) % self.mod1
            self.p2[i] = (self.p2[i - 1] * base) % self.mod2
  
    def query(self, left: int, right: int) -> tuple:
        hash1 = (self.h1[right] - self.h1[left - 1] * self.p1[right - left + 1]) % self.mod1
        hash2 = (self.h2[right] - self.h2[left - 1] * self.p2[right - left + 1]) % self.mod2
        return (hash1, hash2)
```

Then modify the palindrome check to compare both hash values:
```python
v1 = forward_hash.query(left, left + subtree_length // 2 - 1)
v2 = reverse_hash.query(n - right + 1, n - right + 1 + subtree_length // 2 - 1)
result.append(v1[0] == v2[0] and v1[1] == v2[1])
```

### 2. Integer Overflow in Hash Computation

**The Pitfall:**
Even with modulo operations, intermediate calculations like `self.h[i - 1] * base` can overflow before the modulo is applied, especially in languages with fixed integer sizes or when using very large base values.

**Solution:**
Ensure all intermediate operations stay within bounds by applying modulo more frequently:

```python
def __init__(self, string: List[str], base: int, mod: int):
    self.mod = mod
    self.hash_values = [0] * (len(string) + 1)
    self.powers = [1] * (len(string) + 1)
  
    for i in range(1, len(string) + 1):
        # Apply modulo after each operation to prevent overflow
        self.hash_values[i] = ((self.hash_values[i - 1] % mod) * (base % mod)) % mod
        self.hash_values[i] = (self.hash_values[i] + ord(string[i - 1])) % mod
        self.powers[i] = ((self.powers[i - 1] % mod) * (base % mod)) % mod
```

### 3. Off-by-One Errors in Index Calculation

**The Pitfall:**
The mixing of 0-indexed arrays with 1-indexed position tracking can easily lead to off-by-one errors. The current code uses 1-indexed positions for the hash queries while Python uses 0-indexed arrays, creating confusion.

**Solution:**
Add clear documentation and validation:

```python
def query(self, left: int, right: int) -> int:
    """
    Get hash of substring [left, right] where positions are 1-indexed.
    """
    assert 1 <= left <= right <= len(self.hash_values) - 1, f"Invalid range: [{left}, {right}]"
  
    # Calculate with clear variable names
    substring_length = right - left + 1
    hash_value = (self.hash_values[right] - 
                  self.hash_values[left - 1] * self.powers[substring_length]) % self.mod
  
    # Handle negative modulo results
    return (hash_value + self.mod) % self.mod
```

### 4. Incorrect Handling of Odd-Length Palindromes

**The Pitfall:**
The current implementation only compares the first half with the reversed second half using integer division (`k // 2`). For odd-length strings, this ignores the middle character, which could lead to incorrect results.

**Solution:**
Handle odd and even length cases explicitly:

```python
for i in range(n):
    left, right = node_positions[i]
    subtree_length = right - left + 1
  
    if subtree_length == 1:
        # Single character is always a palindrome
        result.append(True)
    else:
        # For even length: compare two halves
        # For odd length: compare two halves excluding middle
        half_length = subtree_length // 2
      
        first_half = forward_hash.query(left, left + half_length - 1)
      
        # Adjust for odd length by potentially skipping middle character
        if subtree_length % 2 == 0:
            second_half_start = n - right + 1
        else:
            second_half_start = n - right + 2  # Skip middle character in reverse
      
        second_half = reverse_hash.query(second_half_start, 
                                        second_half_start + half_length - 1)
      
        result.append(first_half == second_half)
```

