# 2213. Longest Substring of One Repeating Character

> Source: https://algo.monster/liteproblems/2213

## Problem Statement

You are given a string `s` (0-indexed) and need to process `k` queries on it. Each query updates a single character in the string.

For each query `i`, you:
- Replace the character at position `queryIndices[i]` with the character `queryCharacters[i]`

After each query, you need to find the length of the longest substring that consists of only one repeating character in the entire modified string.

**Input:**
- A string `s`
- A string `queryCharacters` of length `k` containing the new characters for each query
- An array `queryIndices` of length `k` containing the positions to update for each query

**Output:**
- An array `lengths` of length `k` where `lengths[i]` is the length of the longest substring of repeating characters after the `i-th` query

**Example:**

If `s = "babacc"`, and we have:
- Query 1: Change index 1 to 'b' → String becomes "bbbacc", longest repeating substring is "bbb" with length 3
- Query 2: Change index 3 to 'c' → String becomes "bbbccc", longest repeating substring is "ccc" with length 3
- And so on...

The problem requires efficiently handling these updates and finding the maximum length of consecutive identical characters after each modification. Since there can be many queries, a naive approach of scanning the entire string after each update would be inefficient, which is why the solution uses a [segment tree](/problems/segment_tree_intro) data structure to maintain and update the required information in `O(log n)` time per query.

## Intuition

The key insight is that after each character update, we need to quickly find the longest substring of repeating characters in the entire string. A naive approach would scan the entire string after each update, taking `O(n)` time per query, which becomes `O(k * n)` total.

We need a way to efficiently maintain information about consecutive characters and update it when a single character changes. This is where a [segment tree](/problems/segment_tree_intro) becomes useful - it can handle point updates and range queries in `O(log n)` time.

The critical observation is that when we change a single character, it only affects the consecutive character counts in its local region. The rest of the string remains unchanged. So we need a data structure that can:
1. Update a single position efficiently
2. Maintain information about consecutive characters in different parts of the string
3. Combine this information to get the global maximum

For each segment (interval) in our tree, we track three key pieces of information:
- `lmx`: How many consecutive identical characters start from the left boundary of this segment
- `rmx`: How many consecutive identical characters end at the right boundary of this segment  
- `mx`: The maximum length of consecutive identical characters within this segment

When we combine two adjacent segments, we need to consider:
- The maximum from the left child segment
- The maximum from the right child segment
- The possibility that the longest sequence spans across both segments (when the rightmost character of the left segment equals the leftmost character of the right segment)

This merging logic allows us to maintain global information about consecutive characters while only updating `O(log n)` nodes when a character changes. The [segment tree](/problems/segment_tree_intro) structure naturally maintains the hierarchical relationship between intervals, making it perfect for this type of problem where local changes affect global properties.

**Learn more about [Segment Tree](/problems/segment_tree_intro) patterns.**

## Solution Approach

The solution uses a [Segment Tree](/problems/segment_tree_intro) data structure to efficiently handle point updates and maintain information about the longest consecutive repeating characters.

### [Segment Tree](/problems/segment_tree_intro) Node Structure

Each node in the [segment tree](/problems/segment_tree_intro) maintains:
- `l, r`: The left and right boundaries of the interval this node represents
- `lmx`: Length of the longest consecutive characters starting from the left boundary
- `rmx`: Length of the longest consecutive characters ending at the right boundary  
- `mx`: Maximum length of consecutive characters within this interval

### Building the [Segment Tree](/problems/segment_tree_intro)

The `build` method recursively constructs the tree:
1. Create a node for interval `[l, r]`
2. If it's a leaf node (`l == r`), initialize all values to 1
3. Otherwise, recursively build left child for `[l, mid]` and right child for `[mid+1, r]`
4. Call `pushup` to combine information from children

### Pushup Operation

The `pushup` method is crucial for maintaining correct information after updates. When combining two adjacent segments:

1. **Update prefix maximum (`lmx`)**: 
   - Start with the left child's `lmx`
   - If the left child is entirely uniform (lmx equals its length) AND the boundary characters match, extend with right child's `lmx`

2. **Update suffix maximum (`rmx`)**:
   - Start with the right child's `rmx`
   - If the right child is entirely uniform (rmx equals its length) AND the boundary characters match, extend with left child's `rmx`

3. **Update overall maximum (`mx`)**:
   - Take the maximum of: left child's `mx`, right child's `mx`
   - If boundary characters match (`s[left.r - 1] == s[right.l - 1]`), also consider `left.rmx + right.lmx` (the sequence spanning both segments)

### Modify Operation

To update a character at position `x`:
1. Navigate to the appropriate leaf node using binary search logic
2. Update the character in the string
3. Call `pushup` on all affected ancestors going back up the tree

### Query Operation

The query always asks for the maximum in the entire range `[1, n]`, which is simply the `mx` value at the root node.

### Time Complexity

- Building the tree: `O(n)`
- Each modification: `O(log n)` 
- Each query: `O(1)` (since we query the entire range)
- Total for k queries: `O(n + k log n)`

### Implementation Details

The solution uses 1-based indexing for the [segment tree](/problems/segment_tree_intro) nodes, where:
- Node `u` has left child at `u << 1` (2*u) and right child at `u << 1 | 1` (2*u + 1)
- The root is at index 1
- Array size is allocated as `4n` to ensure enough space for all nodes

The main solution iterates through each query, performs the modification at the specified index (converting from 0-based to 1-based indexing), and retrieves the maximum consecutive length from the root node.

## Example Walkthrough

Let's walk through a small example with `s = "aabba"` and process two queries:
- Query 1: Change index 2 to 'a' 
- Query 2: Change index 4 to 'b'

**Initial Setup:**
Build a segment tree for string "aabba" (using 1-based indexing internally).

```
Initial string: "aabba"
Tree structure (showing [l,r] intervals):
           [1,5]
          /     \
      [1,3]     [4,5]
      /   \      /  \
   [1,2] [3,3] [4,4][5,5]
   /  \
[1,1][2,2]
```

For each node, we track (lmx, rmx, mx):
- Leaf nodes [1,1] through [5,5]: all have (1,1,1)
- Node [1,2]: chars 'aa' → (2,2,2) - both are 'a', fully consecutive
- Node [3,3]: char 'b' → (1,1,1)
- Node [1,3]: chars 'aab' → (2,1,2) - lmx=2 ('aa'), rmx=1 ('b'), mx=2
- Node [4,4]: char 'b' → (1,1,1)
- Node [5,5]: char 'a' → (1,1,1)
- Node [4,5]: chars 'ba' → (1,1,1) - different chars, no consecutive
- Root [1,5]: Combining [1,3] and [4,5]:
  - Check boundary: s[3]='b', s[4]='b' (they match!)
  - lmx = 2 (from left child, not extended since left child not fully uniform)
  - rmx = 1 (from right child, not extended since right child not fully uniform)
  - mx = max(2, 1, 1+1) = 2 (considering left.mx, right.mx, and left.rmx + right.lmx)

**Query 1: Change index 2 to 'a'**
String becomes "aaaba" (index 2 is position 3 in 1-based indexing)

Modify operation updates node [3,3] and propagates up:
- Node [3,3]: still (1,1,1) but now represents 'a'
- Node [1,3]: chars 'aaa' → pushup recalculates:
  - s[2]='a', s[3]='a' (boundary chars match)
  - lmx = 2 + 1 = 3 (left child fully uniform, extend with right)
  - rmx = 1 + 2 = 3 (right child fully uniform, extend with left)
  - mx = max(2, 1, 2+1) = 3
- Root [1,5]: Recalculate with new [1,3]:
  - s[3]='a', s[4]='b' (boundary chars don't match)
  - lmx = 3 (from left child)
  - rmx = 1 (from right child)
  - mx = max(3, 1) = 3

Result: Length = 3 (substring "aaa")

**Query 2: Change index 4 to 'b'**
String becomes "aaabb" (index 4 is position 5 in 1-based indexing)

Modify operation updates node [5,5] and propagates up:
- Node [5,5]: still (1,1,1) but now represents 'b'
- Node [4,5]: chars 'bb' → pushup recalculates:
  - s[4]='b', s[5]='b' (boundary chars match)
  - lmx = 1 + 1 = 2 (left child fully uniform, extend with right)
  - rmx = 1 + 1 = 2 (right child fully uniform, extend with left)
  - mx = max(1, 1, 1+1) = 2
- Root [1,5]: Recalculate with new [4,5]:
  - s[3]='a', s[4]='b' (boundary chars don't match)
  - lmx = 3 (from left child)
  - rmx = 2 (from right child)  
  - mx = max(3, 2) = 3

Result: Length = 3 (substring "aaa")

The segment tree efficiently maintains the maximum consecutive length by only updating O(log n) nodes per query, avoiding the need to scan the entire string.

## Python Solution

```python
from typing import List, Optional


class Node:
    """Represents a node in the segment tree storing range information."""
    __slots__ = "l", "r", "lmx", "rmx", "mx"
  
    def __init__(self, l: int, r: int):
        self.l = l        # Left boundary of the range
        self.r = r        # Right boundary of the range
        self.lmx = 1      # Maximum consecutive repeating chars from left
        self.rmx = 1      # Maximum consecutive repeating chars from right
        self.mx = 1       # Maximum consecutive repeating chars in range


class SegmentTree:
    """Segment tree for efficiently querying and updating longest repeating substring."""
    __slots__ = "chars", "tree"
  
    def __init__(self, s: str):
        self.chars = list(s)  # Convert string to list for easy modification
        n = len(s)
        self.tree: List[Optional[Node]] = [None] * (n * 4)
        self.build(1, 1, n)
  
    def build(self, node_idx: int, left: int, right: int) -> None:
        """Build the segment tree recursively."""
        self.tree[node_idx] = Node(left, right)
      
        if left == right:
            # Leaf node - single character
            return
      
        mid = (left + right) // 2
        # Build left subtree
        self.build(node_idx << 1, left, mid)
        # Build right subtree
        self.build(node_idx << 1 | 1, mid + 1, right)
        # Update current node based on children
        self.pushup(node_idx)
  
    def query(self, node_idx: int, query_left: int, query_right: int) -> int:
        """Query the maximum consecutive repeating characters in range [query_left, query_right]."""
        current_node = self.tree[node_idx]
      
        # If current range is completely within query range
        if current_node.l >= query_left and current_node.r <= query_right:
            return current_node.mx
      
        mid = (current_node.l + current_node.r) // 2
        result = 0
      
        # Query left child if needed
        if query_left <= mid:
            result = self.query(node_idx << 1, query_left, query_right)
      
        # Query right child if needed
        if query_right > mid:
            result = max(result, self.query(node_idx << 1 | 1, query_left, query_right))
      
        return result
  
    def modify(self, node_idx: int, position: int, new_char: str) -> None:
        """Modify character at position to new_char (1-indexed position)."""
        current_node = self.tree[node_idx]
      
        if current_node.l == current_node.r:
            # Leaf node - update the character
            self.chars[position - 1] = new_char
            return
      
        mid = (current_node.l + current_node.r) // 2
      
        if position <= mid:
            # Update left child
            self.modify(node_idx << 1, position, new_char)
        else:
            # Update right child
            self.modify(node_idx << 1 | 1, position, new_char)
      
        # Update current node based on children
        self.pushup(node_idx)
  
    def pushup(self, node_idx: int) -> None:
        """Update parent node information based on its children."""
        root = self.tree[node_idx]
        left_child = self.tree[node_idx << 1]
        right_child = self.tree[node_idx << 1 | 1]
      
        # Initially inherit values from children
        root.lmx = left_child.lmx
        root.rmx = right_child.rmx
        root.mx = max(left_child.mx, right_child.mx)
      
        # Calculate range lengths
        left_range_size = left_child.r - left_child.l + 1
        right_range_size = right_child.r - right_child.l + 1
      
        # Check if characters at the boundary are the same
        if self.chars[left_child.r - 1] == self.chars[right_child.l - 1]:
            # If left child is completely uniform, extend lmx
            if left_child.lmx == left_range_size:
                root.lmx += right_child.lmx
          
            # If right child is completely uniform, extend rmx
            if right_child.rmx == right_range_size:
                root.rmx += left_child.rmx
          
            # Update maximum by considering the merge at boundary
            root.mx = max(root.mx, left_child.rmx + right_child.lmx)


class Solution:
    def longestRepeating(
        self, s: str, queryCharacters: str, queryIndices: List[int]
    ) -> List[int]:
        """
        Process queries to modify string and return longest repeating substring after each query.
      
        Args:
            s: Initial string
            queryCharacters: Characters to update at each query
            queryIndices: 0-indexed positions to update at each query
          
        Returns:
            List of integers representing longest repeating substring after each query
        """
        # Initialize segment tree with the string
        segment_tree = SegmentTree(s)
        results = []
      
        # Process each query
        for index, char in zip(queryIndices, queryCharacters):
            # Convert 0-indexed to 1-indexed and modify
            segment_tree.modify(1, index + 1, char)
            # Query entire range for maximum consecutive repeating characters
            max_repeating = segment_tree.query(1, 1, len(s))
            results.append(max_repeating)
      
        return results
```

## Time and Space Complexity

**Time Complexity:** `O((n + q) × log n)` where `n` is the length of string `s` and `q` is the number of queries.

- **Building the segment tree:** The `build` method is called once and visits each node in the tree exactly once. Since there are `O(n)` nodes in total (approximately `4n` nodes for a segment tree), and each node performs `O(1)` work except for the `pushup` operation which is also `O(1)`, the build time is `O(n)`.

- **Query operation:** Each `query` call traverses at most `O(log n)` levels of the tree (the height of the tree), performing `O(1)` work at each level. Since we perform `q` queries, the total query time is `O(q × log n)`.

- **Modify operation:** Each `modify` call traverses from root to leaf, which is `O(log n)` levels, and then calls `pushup` on the way back up, visiting `O(log n)` nodes. Each `pushup` operation is `O(1)`. Since we perform `q` modifications, the total modification time is `O(q × log n)`.

- **Total time complexity:** `O(n + q × log n)`. When `q` is proportional to `n`, this becomes `O(n × log n)`, which matches the reference answer.

**Space Complexity:** `O(n)`

- **Segment tree storage:** The tree array `tr` has size `4n`, storing `Node` objects. Each `Node` has constant space (5 integer attributes), so the tree requires `O(n)` space.

- **String storage:** The string `s` is converted to a list of characters, requiring `O(n)` space.

- **Output array:** The answer array stores `q` integers, requiring `O(q)` space.

- **Total space complexity:** `O(n + q)`. When `q` is proportional to `n`, this becomes `O(n)`. 

Note: The reference answer states `O(n × log n)` for space complexity, but the actual implementation uses an array-based segment tree with `4n` nodes, which is `O(n)` space, not `O(n × log n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Boundary Character Comparison in Pushup**

One of the most common mistakes is incorrectly handling the boundary character comparison when merging two segments. The critical issue is using wrong indices when checking if adjacent segments can be merged.

**Pitfall Code:**
```python
# WRONG: Using 0-based index directly
if self.chars[left_child.r] == self.chars[right_child.l]:
    # This will cause index out of bounds or compare wrong characters
```

**Correct Code:**
```python
# CORRECT: Convert to 0-based index by subtracting 1
if self.chars[left_child.r - 1] == self.chars[right_child.l - 1]:
    # Properly compares the last char of left segment with first char of right segment
```

### 2. **Not Handling Single Character Updates Properly**

Another pitfall is forgetting that when we update a character, it affects not just the containing segment but all ancestor segments up to the root. Some implementations might update only the leaf node without propagating changes upward.

**Pitfall Implementation:**
```python
def modify(self, node_idx: int, position: int, new_char: str) -> None:
    current_node = self.tree[node_idx]
  
    if current_node.l == current_node.r:
        self.chars[position - 1] = new_char
        return  # WRONG: Returns without updating ancestors
  
    # Recursive calls without pushup
    mid = (current_node.l + current_node.r) // 2
    if position <= mid:
        self.modify(node_idx << 1, position, new_char)
    else:
        self.modify(node_idx << 1 | 1, position, new_char)
    # MISSING: self.pushup(node_idx)
```

### 3. **Incorrect Calculation of Merged Sequence Length**

When two segments can be merged at their boundary, the total length of the merged sequence must consider both the suffix of the left segment and the prefix of the right segment.

**Pitfall Code:**
```python
# WRONG: Only considering one side or using wrong values
root.mx = max(root.mx, left_child.mx + right_child.mx)  # Incorrect!
```

**Correct Code:**
```python
# CORRECT: Use suffix of left (rmx) and prefix of right (lmx)
root.mx = max(root.mx, left_child.rmx + right_child.lmx)
```

### 4. **Index Conversion Between 0-based and 1-based**

The segment tree uses 1-based indexing internally, but Python strings use 0-based indexing. Mixing these up is a frequent source of errors.

**Common Mistakes:**
```python
# WRONG: Forgetting to convert when accessing the character array
char_at_position = self.chars[position]  # Should be position - 1

# WRONG: Not converting query indices in the main solution
segment_tree.modify(1, index, char)  # Should be index + 1
```

### 5. **Not Checking Complete Uniformity Before Extension**

When updating `lmx` and `rmx` in pushup, we must verify that a child segment is completely uniform before extending the parent's prefix/suffix.

**Pitfall Code:**
```python
# WRONG: Extending without checking if child is completely uniform
if self.chars[left_child.r - 1] == self.chars[right_child.l - 1]:
    root.lmx = left_child.lmx + right_child.lmx  # Always extends!
```

**Correct Code:**
```python
# CORRECT: Only extend if the entire left child is uniform
if self.chars[left_child.r - 1] == self.chars[right_child.l - 1]:
    if left_child.lmx == left_child.r - left_child.l + 1:  # Check uniformity
        root.lmx = left_child.lmx + right_child.lmx
```

### Solution to Avoid These Pitfalls:

1. **Always draw out index mappings** between your segment tree (1-based) and array (0-based)
2. **Test with small examples** where you can manually verify the pushup calculations
3. **Add assertions** during development to catch index errors early
4. **Visualize the merge cases** on paper before coding the pushup logic
5. **Use consistent naming** for indices to avoid confusion (e.g., always suffix with `_0based` or `_1based` during development)

