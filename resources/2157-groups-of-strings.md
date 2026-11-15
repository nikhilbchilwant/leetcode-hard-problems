# 2157. Groups of Strings

> Source: https://algo.monster/liteproblems/2157

## Problem Statement

You have an array of strings where each string contains only lowercase English letters, and no letter appears more than once in any single string.

Two strings are considered **connected** if you can transform the set of letters from one string to the other by performing exactly one of these operations:
- **Add** one letter to the set
- **Delete** one letter from the set  
- **Replace** one letter with any other letter (including the same letter)

Your task is to group these strings into connected components where:
- Each string in a group is connected to at least one other string in the same group, OR
- The group contains only a single string
- No string in one group can be connected to any string in a different group

You need to return an array with two values:
1. The **total number of groups** formed
2. The **size of the largest group**

**Example to understand connections:**
- `"abc"` and `"ab"` are connected (delete 'c')
- `"abc"` and `"abcd"` are connected (add 'd')
- `"abc"` and `"abd"` are connected (replace 'c' with 'd')

## Intuition

The key insight is recognizing this as a graph connectivity problem where strings are nodes and "connected" relationships are edges. We need to find connected components in this graph.

Since each string contains unique letters only, we can think of each string as a **set of characters**. This naturally leads to using **bit manipulation** - we can represent each string as a bitmask where bit position `i` indicates whether character `'a' + i` is present. For example, `"abc"` becomes `0...0111` (bits 0, 1, 2 are set).

Why is this representation powerful? Because the three connection operations translate elegantly to bit operations:
- **Adding a letter**: Set a bit that was 0 → use OR operation with `(1 << i)`
- **Deleting a letter**: Clear a bit that was 1 → use XOR operation with `(1 << i)` 
- **Replacing a letter**: Clear one bit and set another → combine delete and add operations

Instead of checking connections between all pairs of original strings (which would be expensive), we can:
1. Convert each string to its bitmask representation
2. For each bitmask, generate all possible connected bitmasks by:
   - Toggling each bit (handles add/delete operations)
   - For each set bit, trying to replace it with every unset bit

This transforms the problem into finding connected components among bitmasks, which is perfectly suited for **Union-Find (Disjoint Set Union)**. We union bitmasks that are connected, and the Union-Find structure naturally tracks:
- The number of disjoint sets (number of groups)
- The size of each set (to find the maximum group size)

The brilliance is that identical strings automatically map to the same bitmask, so they're inherently in the same group without extra work. The Union-Find efficiently handles the merging of groups as we discover connections.

**Learn more about [Union Find](/problems/dsu_intro) patterns.**

## Solution Approach

The solution uses **Union-Find (Disjoint Set Union)** data structure with bit manipulation to efficiently group connected strings.

**Step 1: Initialize Data Structures**
- `p`: Dictionary to store parent pointers for Union-Find
- `size`: Counter to track the size of each group
- `n`: Number of groups (initially equals number of words)
- `mx`: Maximum group size

**Step 2: Convert Strings to Bitmasks**
For each word, create a bitmask representation:
```python
x = 0
for c in word:
    x |= 1 << (ord(c) - ord('a'))
```
- Each bit position represents a letter ('a' at bit 0, 'b' at bit 1, etc.)
- Set bit `i` if character `'a' + i` is present
- Store in Union-Find: `p[x] = x` (initially, each bitmask is its own parent)
- Track size: `size[x] += 1`
- If multiple words map to same bitmask, decrease group count: `n -= 1`

**Step 3: Connect Related Bitmasks**
For each bitmask `x`, generate and union with all connected bitmasks:

1. **Handle Add/Delete Operations**: 
   ```python
   for i in range(26):
       union(x, x ^ (1 << i))
   ```
   - `x ^ (1 << i)` toggles bit `i` (adds letter if absent, deletes if present)

2. **Handle Replace Operations**:
   ```python
   if (x >> i) & 1:  # if bit i is set
       for j in range(26):
           if ((x >> j) & 1) == 0:  # if bit j is not set
               union(x, x ^ (1 << i) | (1 << j))
   ```
   - For each set bit `i`, try replacing with every unset bit `j`
   - `x ^ (1 << i)` removes letter at position `i`
   - `| (1 << j)` adds letter at position `j`

**Step 4: Union-Find Operations**

The `find` function with path compression:
```python
def find(x):
    if p[x] != x:
        p[x] = find(p[x])  # Path compression
    return p[x]
```

The `union` function merges groups:
```python
def union(a, b):
    if b not in p:  # Skip if bitmask b doesn't exist
        return
    pa, pb = find(a), find(b)
    if pa == pb:  # Already in same group
        return
    p[pa] = pb  # Merge groups
    size[pb] += size[pa]  # Update size
    mx = max(mx, size[pb])  # Track maximum
    n -= 1  # Decrease number of groups
```

**Step 5: Return Results**
- `n`: Final number of groups after all unions
- `mx`: Maximum group size encountered

The time complexity is `O(N × 26 × 26)` where N is the number of unique bitmasks, and space complexity is `O(N)` for storing the Union-Find structure.

## Example Walkthrough

Let's walk through a small example with words = `["a", "b", "ab", "abc"]`

**Step 1: Convert to Bitmasks**
- `"a"` → bitmask = `001` (bit 0 set)
- `"b"` → bitmask = `010` (bit 1 set)  
- `"ab"` → bitmask = `011` (bits 0,1 set)
- `"abc"` → bitmask = `111` (bits 0,1,2 set)

Initial state: 4 separate groups, each of size 1.

**Step 2: Process Connections for Each Bitmask**

For bitmask `001` ("a"):
- Add/Delete: Try `001 ^ (1<<0) = 000` (delete 'a'), `001 ^ (1<<1) = 011` (add 'b')
- Union `001` with `011` (exists!) → Groups: {001, 011}, {010}, {111}
- Replace: Can replace 'a' with any other letter, but none of those bitmasks exist

For bitmask `010` ("b"):
- Add/Delete: Try `010 ^ (1<<0) = 011` (add 'a'), `010 ^ (1<<1) = 000` (delete 'b')
- Union `010` with `011` (exists!) → Groups: {001, 011, 010}, {111}

For bitmask `011` ("ab"):
- Add/Delete: Try `011 ^ (1<<2) = 111` (add 'c')
- Union `011` with `111` (exists!) → Groups: {001, 011, 010, 111}
- All 4 strings are now in one connected group!

For bitmask `111` ("abc"):
- Already connected to the main group through previous operations

**Final Result:** 
- Number of groups: 1
- Largest group size: 4

The key insight: `"a"` connects to `"ab"` (add 'b'), `"b"` connects to `"ab"` (add 'a'), and `"ab"` connects to `"abc"` (add 'c'), forming one large connected component.

## Python Solution

```python
class Solution:
    def groupStrings(self, words: List[str]) -> List[int]:
        def find(node):
            """Find the root parent of a node with path compression."""
            if parent[node] != node:
                parent[node] = find(parent[node])  # Path compression
            return parent[node]

        def union(node_a, node_b):
            """Union two nodes if node_b exists in the parent dictionary."""
            nonlocal max_group_size, num_groups
          
            # Check if node_b exists in our set
            if node_b not in parent:
                return
          
            # Find roots of both nodes
            root_a = find(node_a)
            root_b = find(node_b)
          
            # Already in the same group
            if root_a == root_b:
                return
          
            # Merge groups: make root_b the parent of root_a
            parent[root_a] = root_b
            group_size[root_b] += group_size[root_a]
          
            # Update maximum group size and decrease group count
            max_group_size = max(max_group_size, group_size[root_b])
            num_groups -= 1

        # Initialize Union-Find structure
        parent = {}  # Maps bitmask to its parent
        group_size = Counter()  # Tracks size of each group
        num_groups = len(words)  # Initially each word is its own group
        max_group_size = 0
      
        # Convert each word to bitmask representation
        for word in words:
            bitmask = 0
            for char in word:
                bitmask |= 1 << (ord(char) - ord('a'))
          
            # Initialize parent and size for this bitmask
            parent[bitmask] = bitmask
            group_size[bitmask] += 1
            max_group_size = max(max_group_size, group_size[bitmask])
          
            # If multiple words map to same bitmask, they're already grouped
            if group_size[bitmask] > 1:
                num_groups -= 1
      
        # Try all possible transformations for each unique bitmask
        for bitmask in list(parent.keys()):
            # Try adding/removing each letter (single bit flip)
            for bit_pos in range(26):
                union(bitmask, bitmask ^ (1 << bit_pos))
              
                # If this bit is set (letter exists), try replacing it
                if (bitmask >> bit_pos) & 1:
                    # Replace with any other letter not in the word
                    for new_bit_pos in range(26):
                        if ((bitmask >> new_bit_pos) & 1) == 0:
                            # Remove old letter, add new letter
                            new_bitmask = bitmask ^ (1 << bit_pos) | (1 << new_bit_pos)
                            union(bitmask, new_bitmask)
      
        return [num_groups, max_group_size]
```

## Time and Space Complexity

**Time Complexity:** `O(n * 26 * 26 * α(n))` where `n` is the number of unique bitmask representations and `α(n)` is the inverse Ackermann function.

- Converting each word to a bitmask takes `O(L)` where `L` is the length of the word. For all words, this is `O(N * L)` where `N` is the total number of words.
- The main loop iterates through each unique bitmask `x` in the dictionary `p`.
- For each bitmask `x`, we iterate through 26 possible bit positions (first inner loop).
- For each set bit at position `i`, we iterate through another 26 positions (second inner loop) to try replacement operations.
- Each `union` operation involves `find` operations which take `O(α(n))` amortized time with path compression.
- In the worst case, we have `n` unique bitmasks, and for each we perform up to `26 * 26` union operations.
- Total: `O(N * L + n * 26 * 26 * α(n))`. Since `n ≤ N` and typically `L` is small (bounded by 26), the dominant term is `O(n * 26 * 26 * α(n))`.

**Space Complexity:** `O(n)`

- The dictionary `p` stores at most `n` unique bitmask representations as parent pointers.
- The `size` Counter also stores at most `n` entries.
- The recursive call stack for `find` can go up to `O(log n)` depth in the average case with path compression, but this doesn't dominate the space complexity.
- Additional variables use `O(1)` space.
- Total space complexity is `O(n)` where `n` is the number of unique bitmask representations of the input words.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Handle Duplicate Words**
**Pitfall:** When multiple words map to the same bitmask (e.g., "abc", "bca", "cab"), they need to be counted as already connected. Failing to decrease the group count for duplicates will result in an incorrect number of groups.

**Solution:** Track when multiple words map to the same bitmask and decrease `num_groups` accordingly:
```python
if group_size[bitmask] > 1:
    num_groups -= 1  # These words are already in the same group
```

### 2. **Incorrect Replace Operation Implementation**
**Pitfall:** A common mistake is trying to replace any letter with any other letter, including replacing a letter with itself or with a letter already in the word. This creates invalid transformations.

**Solution:** Only replace existing letters with letters NOT in the word:
```python
if (bitmask >> bit_pos) & 1:  # Letter exists at bit_pos
    for new_bit_pos in range(26):
        if ((bitmask >> new_bit_pos) & 1) == 0:  # Letter doesn't exist at new_bit_pos
            new_bitmask = bitmask ^ (1 << bit_pos) | (1 << new_bit_pos)
```

### 3. **Not Checking if Target Bitmask Exists Before Union**
**Pitfall:** Attempting to union with bitmasks that don't exist in the parent dictionary (no word maps to that bitmask) will cause KeyError or create phantom groups.

**Solution:** Always check existence before union:
```python
def union(node_a, node_b):
    if node_b not in parent:  # Critical check
        return
    # ... rest of union logic
```

### 4. **Using Mutable Dictionary Keys During Iteration**
**Pitfall:** Modifying the parent dictionary while iterating over `parent.keys()` can cause runtime errors or skip elements.

**Solution:** Convert keys to a list before iteration:
```python
for bitmask in list(parent.keys()):  # Create a copy of keys
    # Safe to modify parent dictionary now
```

### 5. **Inefficient Path Compression**
**Pitfall:** Not implementing path compression in the find operation leads to degraded performance with deep trees, potentially causing TLE on large inputs.

**Solution:** Always implement path compression:
```python
def find(node):
    if parent[node] != node:
        parent[node] = find(parent[node])  # Compress the path
    return parent[node]
```

