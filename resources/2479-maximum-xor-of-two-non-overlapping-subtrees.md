# 2479. Maximum XOR of Two Non-Overlapping Subtrees

> Source: https://algo.monster/liteproblems/2479

## Problem Statement

You are given an undirected [tree](/problems/tree_intro) with `n` nodes labeled from `0` to `n - 1`. The tree structure is defined by an array `edges` of length `n - 1`, where each `edges[i] = [ai, bi]` represents an edge between nodes `ai` and `bi`. Node `0` is the root of the tree.

Each node has a value associated with it, given in the array `values` where `values[i]` represents the value of node `i`.

Your task is to select two **non-overlapping subtrees** from this [tree](/problems/tree_intro). The score of your selection is calculated as the **bitwise XOR** of the sum of values in each subtree. You need to find the **maximum possible score**.

Key definitions:
- A **subtree** of a node consists of that node and all its descendants
- Two subtrees are **non-overlapping** if they don't share any common nodes
- If it's impossible to find two non-overlapping subtrees, return `0`

For example, if you select two non-overlapping subtrees where:
- Subtree 1 has nodes with values summing to `sum1`
- Subtree 2 has nodes with values summing to `sum2`

Then your score would be `sum1 XOR sum2`, and you want to maximize this value.

## Intuition

The key insight is recognizing that for two subtrees to be non-overlapping in a [tree](/problems/tree_intro), one subtree must be completely contained within a branch that doesn't include the other subtree. This means if we pick any subtree rooted at node `i`, any non-overlapping subtree must be either:
1. An ancestor subtree that doesn't include node `i`
2. A subtree in a different branch of the tree

To maximize the XOR of two sums, we need to find pairs of sums whose binary representations differ as much as possible in the most significant bits. XOR produces `1` when bits differ and `0` when they're the same, so maximum XOR occurs when the binary representations are most different.

The challenge is efficiently finding the best pair among all possible non-overlapping subtree combinations. Here's how we arrive at the solution:

**Step 1: Calculate all subtree sums**
We first need to know the sum of values for every possible subtree. Using DFS from the root, we can calculate `s[i]` = sum of all values in the subtree rooted at node `i`. This is done bottom-up: a node's subtree sum equals its own value plus the subtree sums of all its children.

**Step 2: Ensure non-overlapping property**
The clever part is realizing that if we traverse the [tree](/problems/tree_intro) in post-order (visiting children before parents), we can maintain a set of "available" subtree sums that don't overlap with the current subtree. When we're at node `i`:
- All subtrees in previously visited branches are non-overlapping with subtree `i`
- After processing node `i` and its subtree, we add `s[i]` to our available set

**Step 3: Optimize XOR search with [Trie](/problems/trie_intro)**
Finding the maximum XOR with a given number from a set is a classic problem solved using a binary Trie. For each bit position (starting from most significant), we try to go the opposite direction in the Trie to maximize the XOR value. The Trie stores binary representations of subtree sums we've seen so far.

By combining these insights - calculating subtree sums via DFS, ensuring non-overlap through traversal order, and optimizing XOR search with a [Trie](/problems/trie_intro) - we arrive at an elegant solution that finds the maximum XOR of two non-overlapping subtrees efficiently.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro), [Graph](/problems/graph_intro) and [Trie](/problems/trie_intro) patterns.**

## Solution Approach

The solution implements a two-pass DFS strategy combined with a [Trie](/problems/trie_intro) data structure for efficient XOR maximization.

**[Trie](/problems/trie_intro) Implementation for Maximum XOR:**

The `[Trie](/problems/trie_intro)` class is designed to store binary representations of numbers and find the maximum XOR:
- `insert(x)`: Stores the 48-bit binary representation of number `x` in the Trie, bit by bit from most significant to least significant
- `search(x)`: For a given number `x`, finds the number in the Trie that produces maximum XOR with `x` by greedily choosing the opposite bit at each level when possible

**Main Algorithm:**

1. **Build the adjacency list:** Convert the edge list into an adjacency list representation `g` for efficient [tree](/problems/tree_intro) traversal.

2. **First DFS (`dfs1`) - Calculate Subtree Sums:**
   ```
   dfs1(node, parent):
       sum = values[node]
       for each child of node (except parent):
           sum += dfs1(child, node)
       s[node] = sum
       return sum
   ```
   This recursively calculates and stores in `s[i]` the sum of all values in the subtree rooted at node `i`.

3. **Second DFS (`dfs2`) - Find Maximum XOR:**
   ```
   dfs2(node, parent):
       ans = max(ans, [tree](/problems/tree_intro).search(s[node]))
       for each child of node (except parent):
           dfs2(child, node)
       tree.insert(s[node])
   ```
   The key insight here is the traversal order:
   - When at node `i`, we first check the maximum XOR between `s[i]` and all previously inserted subtree sums in the [Trie](/problems/trie_intro)
   - We then recursively process all children
   - Finally, we insert `s[i]` into the Trie

   This order ensures that when we check XOR for subtree at node `i`, the [Trie](/problems/trie_intro) only contains sums from non-overlapping subtrees (those from different branches already processed).

**Why This Works:**

The post-order insertion (processing children before inserting parent) guarantees that:
- When processing a node, all subtrees in its descendants haven't been added to the [Trie](/problems/trie_intro) yet
- All subtrees from sibling branches and their ancestors are already in the Trie
- These are exactly the subtrees that don't overlap with the current subtree

The algorithm efficiently finds the maximum XOR value among all pairs of non-overlapping subtrees in `O(n * log(max_sum))` time, where the log factor comes from the [Trie](/problems/trie_intro) operations on 48-bit numbers.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a tree with 5 nodes and the following structure:
```
       0 (value: 2)
      / \
     1   2
   (3)   (5)
   / \
  3   4
 (1) (4)
```

**Given:**
- `edges = [[0,1], [0,2], [1,3], [1,4]]`
- `values = [2, 3, 5, 1, 4]`

**Step 1: Build adjacency list**
```
g[0] = [1, 2]
g[1] = [0, 3, 4]
g[2] = [0]
g[3] = [1]
g[4] = [1]
```

**Step 2: First DFS - Calculate subtree sums**

Starting from root 0, we calculate bottom-up:
- `dfs1(3, 1)`: s[3] = 1 (leaf node)
- `dfs1(4, 1)`: s[4] = 4 (leaf node)
- `dfs1(1, 0)`: s[1] = 3 + 1 + 4 = 8 (node 1 + subtrees 3 and 4)
- `dfs1(2, 0)`: s[2] = 5 (leaf node)
- `dfs1(0, -1)`: s[0] = 2 + 8 + 5 = 15 (entire tree)

Result: `s = [15, 8, 5, 1, 4]`

**Step 3: Second DFS - Find maximum XOR**

Starting from root 0, we traverse and maintain a Trie:

1. **At node 0**:
   - Trie is empty, so `search(15)` returns 0
   - Recursively visit child 1

2. **At node 1**:
   - Trie is still empty, so `search(8)` returns 0
   - Recursively visit children 3 and 4

3. **At node 3**:
   - Trie is empty, so `search(1)` returns 0
   - Insert s[3] = 1 into Trie

4. **At node 4**:
   - Trie contains [1], so `search(4)` finds XOR with 1 = 4 XOR 1 = 5
   - Current max = 5
   - Insert s[4] = 4 into Trie

5. **Back at node 1**:
   - Insert s[1] = 8 into Trie
   - Trie now contains [1, 4, 8]

6. **At node 2**:
   - Trie contains [1, 4, 8]
   - `search(5)` checks XOR with each:
     - 5 XOR 1 = 4
     - 5 XOR 4 = 1
     - 5 XOR 8 = 13 ← Maximum!
   - Update max = 13
   - Insert s[2] = 5 into Trie

7. **Back at node 0**:
   - Insert s[0] = 15 into Trie

**Result:** Maximum XOR = 13

This comes from selecting:
- Subtree rooted at node 1 (sum = 8)
- Subtree rooted at node 2 (sum = 5)
- 8 XOR 5 = 13

Note how the traversal order ensures these subtrees don't overlap - node 2's subtree sum was only compared with sums from node 1's branch after node 1's entire subtree was processed.

## Python Solution

```python
from typing import List
from collections import defaultdict


class Trie:
    """Binary Trie data structure for finding maximum XOR."""

    def __init__(self):
        # Each node has at most 2 children (for bits 0 and 1)
        self.children = [None] * 2

    def insert(self, num: int) -> None:
        """Insert a number into the trie by its binary representation."""
        current_node = self
        # Process 48 bits from most significant to least significant
        for bit_position in range(47, -1, -1):
            # Extract the bit at current position
            bit_value = (num >> bit_position) & 1

            # Create new node if path doesn't exist
            if current_node.children[bit_value] is None:
                current_node.children[bit_value] = Trie()

            # Move to the child node
            current_node = current_node.children[bit_value]

    def search(self, num: int) -> int:
        """Find the maximum XOR value for the given number against all inserted numbers."""
        current_node = self
        max_xor_result = 0

        # Process 48 bits from most significant to least significant
        for bit_position in range(47, -1, -1):
            # Extract the bit at current position
            bit_value = (num >> bit_position) & 1

            # Early termination if node doesn't exist
            if current_node is None:
                return max_xor_result

            # Try to go opposite direction for maximum XOR
            opposite_bit = bit_value ^ 1

            if current_node.children[opposite_bit]:
                # If opposite bit path exists, take it and set result bit to 1
                max_xor_result = (max_xor_result << 1) | 1
                current_node = current_node.children[opposite_bit]
            else:
                # Otherwise, follow the same bit path and set result bit to 0
                max_xor_result <<= 1
                current_node = current_node.children[bit_value]

        return max_xor_result


class Solution:
    def maxXor(self, n: int, edges: List[List[int]], values: List[int]) -> int:
        """
        Find the maximum XOR value between subtree sums in a tree.

        Args:
            n: Number of nodes in the tree
            edges: List of edges representing the tree structure
            values: List of values for each node

        Returns:
            Maximum XOR value between any two subtree sums
        """

        def calculate_subtree_sums(node: int, parent: int) -> int:
            """
            DFS to calculate sum of values in subtree rooted at node.

            Args:
                node: Current node being processed
                parent: Parent of current node (-1 for root)

            Returns:
                Sum of all values in the subtree rooted at node
            """
            # Start with current node's value
            subtree_sum = values[node]

            # Add sums from all child subtrees
            for neighbor in adjacency_list[node]:
                if neighbor != parent:
                    subtree_sum += calculate_subtree_sums(neighbor, node)

            # Store the subtree sum for this node
            subtree_sums[node] = subtree_sum
            return subtree_sum

        def find_max_xor(node: int, parent: int) -> None:
            """
            DFS to find maximum XOR by comparing current subtree sum with previously seen sums.

            Args:
                node: Current node being processed
                parent: Parent of current node (-1 for root)
            """
            nonlocal max_xor_value

            # Find maximum XOR with current subtree sum against all previously inserted sums
            max_xor_value = max(max_xor_value, trie.search(subtree_sums[node]))

            # Process all children first (post-order traversal)
            for neighbor in adjacency_list[node]:
                if neighbor != parent:
                    find_max_xor(neighbor, node)

            # Insert current subtree sum after processing children
            trie.insert(subtree_sums[node])

        # Build adjacency list representation of the tree
        adjacency_list = defaultdict(list)
        for node_a, node_b in edges:
            adjacency_list[node_a].append(node_b)
            adjacency_list[node_b].append(node_a)

        # Initialize array to store subtree sums
        subtree_sums = [0] * n

        # First DFS: Calculate all subtree sums
        calculate_subtree_sums(0, -1)

        # Initialize maximum XOR value and Trie
        max_xor_value = 0
        trie = Trie()

        # Second DFS: Find maximum XOR value
        find_max_xor(0, -1)

        return max_xor_value
```

## Time and Space Complexity

**Time Complexity: `O(n * log(max_value))`**

The algorithm consists of three main parts:

1. **Building the graph**: `O(n)` - iterating through all edges once
2. **First DFS (dfs1)**: `O(n)` - visiting each node exactly once to compute subtree sums
3. **Second DFS (dfs2)**: `O(n * log(max_value))` where `log(max_value) = 48` bits
   - Each node is visited once: `O(n)`
   - For each node, we perform:
     - Trie search operation: `O(48)` - iterating through 48 bits
     - Trie insert operation: `O(48)` - iterating through 48 bits

Since the bit operations are performed for a fixed 48-bit representation, the overall time complexity is `O(n * 48) = O(n)` when considering 48 as a constant.

**Space Complexity: `O(n * log(max_value))`**

The space usage includes:

1. **Graph adjacency list**: `O(n)` - storing all edges
2. **Subtree sum array `s`**: `O(n)` - storing sum for each node
3. **Trie structure**: `O(n * 48)` in worst case
   - Each inserted number can create up to 48 new nodes
   - With n insertions, maximum nodes = `n * 48`
4. **DFS recursion stack**: `O(h)` where h is the height of the tree, worst case `O(n)` for a skewed tree

The dominant factor is the Trie structure, giving us `O(n * 48) = O(n)` space complexity when treating 48 as a constant.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Bit Width in Trie Implementation**

One of the most common pitfalls is using an insufficient number of bits in the Trie implementation. The problem doesn't specify constraints on node values, but the sum of all values in a subtree can be quite large.

**The Issue:**
If you use too few bits (e.g., 32 bits), large subtree sums will overflow or be incorrectly represented, leading to wrong XOR calculations.

```python
# INCORRECT: Using only 32 bits
def insert(self, num: int) -> None:
    current_node = self
    for bit_position in range(31, -1, -1):  # Only 32 bits!
        bit_value = (num >> bit_position) & 1
        # ... rest of code
```

**The Solution:**
Use a sufficient number of bits to handle the maximum possible subtree sum. The provided solution uses 48 bits, which can handle sums up to 2^48 - 1 (approximately 281 trillion).

```python
# CORRECT: Using 48 bits for larger sums
def insert(self, num: int) -> None:
    current_node = self
    for bit_position in range(47, -1, -1):  # 48 bits
        bit_value = (num >> bit_position) & 1
        # ... rest of code
```

### 2. **Wrong Traversal Order in Second DFS**

Another critical pitfall is inserting the current node's subtree sum into the Trie before processing its children, which would allow overlapping subtrees to be compared.

**The Issue:**
```python
# INCORRECT: Inserting before processing children
def find_max_xor(node: int, parent: int) -> None:
    nonlocal max_xor_value

    # Wrong! Inserting before processing children
    trie.insert(subtree_sums[node])

    max_xor_value = max(max_xor_value, trie.search(subtree_sums[node]))

    for neighbor in adjacency_list[node]:
        if neighbor != parent:
            find_max_xor(neighbor, node)
```

This would allow a parent node's subtree sum to be compared with its descendant's subtree sum, violating the non-overlapping requirement.

**The Solution:**
Always insert the current node's subtree sum AFTER processing all its children:

```python
# CORRECT: Post-order insertion
def find_max_xor(node: int, parent: int) -> None:
    nonlocal max_xor_value

    # First, search for maximum XOR with existing non-overlapping subtrees
    max_xor_value = max(max_xor_value, trie.search(subtree_sums[node]))

    # Then process all children
    for neighbor in adjacency_list[node]:
        if neighbor != parent:
            find_max_xor(neighbor, node)

    # Finally, insert current subtree sum after children are processed
    trie.insert(subtree_sums[node])
```

### 3. **Forgetting to Handle Single-Node Trees**

When n=1 (only one node), there's only one subtree possible, so finding two non-overlapping subtrees is impossible.

**The Issue:**
The algorithm might try to XOR the single subtree sum with nothing (or 0), giving an incorrect result.

**The Solution:**
The current implementation handles this correctly because:
- When there's only one node, the Trie is initially empty
- `trie.search()` returns 0 when searching in an empty Trie
- The final answer is 0, which is correct as per the problem statement

However, for clarity, you could add an explicit check:

```python
def maxXor(self, n: int, edges: List[List[int]], values: List[int]) -> int:
    # Edge case: single node tree
    if n == 1:
        return 0

    # ... rest of the implementation
```

### 4. **Not Handling Negative Values Correctly**

If the problem allows negative node values, the Trie implementation needs modification to handle negative numbers properly.

**The Issue:**
Negative numbers in two's complement representation have different bit patterns that could cause issues with the current Trie implementation.

**The Solution:**
If negative values are possible, offset all values by a constant to make them non-negative, or modify the Trie to handle signed integers:

```python
# If negative values are possible, offset to make all positive
MIN_VALUE = min(values)
if MIN_VALUE < 0:
    offset = -MIN_VALUE
    adjusted_values = [v + offset for v in values]
    # Use adjusted_values in calculations
    # Note: XOR result remains the same since both sums are offset equally
```

