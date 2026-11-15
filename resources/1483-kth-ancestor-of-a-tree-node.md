# 1483. Kth Ancestor of a Tree Node

> Source: https://algo.monster/liteproblems/1483

## Problem Statement

You are given a [tree](/problems/tree_intro) with `n` nodes numbered from `0` to `n - 1`. The tree is represented using a parent array `parent`, where `parent[i]` is the parent of the `i`-th node. The root of the tree is node `0` (it has no parent).

Your task is to implement a data structure that can efficiently find the `k`-th ancestor of any given node. The `k`-th ancestor of a node is the node that is `k` steps up in the path from that node to the root.

You need to implement the `TreeAncestor` class with the following methods:

- `TreeAncestor(int n, int[] parent)`: Initializes the object with `n` nodes and their parent relationships given in the `parent` array.
- `int getKthAncestor(int node, int k)`: Returns the `k`-th ancestor of the given `node`. If no such ancestor exists (i.e., `k` is greater than the depth of the node), return `-1`.

For example, if we have a [tree](/problems/tree_intro) where node 1's parent is 0, node 2's parent is 1, and node 3's parent is 2, then:
- The 1st ancestor of node 3 is node 2
- The 2nd ancestor of node 3 is node 1
- The 3rd ancestor of node 3 is node 0
- The 4th ancestor of node 3 doesn't exist, so we return -1

## Intuition

The naive approach to find the `k`-th ancestor would be to start from the given node and traverse upward `k` times through parent pointers. However, this takes `O(k)` time per query, which becomes inefficient when `k` is large or when we have many queries.

The key insight is to ask: "Can we make larger jumps instead of going one parent at a time?"

Think about how we represent numbers in binary. Any number `k` can be decomposed into powers of 2. For example, `13 = 8 + 4 + 1 = 2^3 + 2^2 + 2^0`. This means to go up 13 ancestors, we can make three jumps: go up 8 ancestors, then 4 ancestors, then 1 ancestor.

This leads us to the binary lifting idea: what if we precompute for each node where we'd end up after jumping `2^0, 2^1, 2^2, ...` ancestors? With this information, we can reach any `k`-th ancestor by combining these power-of-2 jumps.

Why does this work? Because:
1. Any positive integer can be represented as a sum of distinct powers of 2 (binary representation)
2. We only need `log(n)` different jump sizes to cover any possible distance in a [tree](/problems/tree_intro) of `n` nodes
3. Each jump can be computed from the previous one: the `2^j`-th ancestor is just the `2^(j-1)`-th ancestor of the `2^(j-1)`-th ancestor

This transforms our `O(k)` traversal into `O(log k)` jumps, making queries much faster. The tradeoff is that we need `O(n log n)` preprocessing time and space to build our jump table, but this is worthwhile when we have multiple queries to answer.

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro), [Binary Search](/problems/binary_search_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements binary lifting using [dynamic programming](/problems/dynamic_programming_intro). Let's walk through the implementation:

**Data Structure Setup:**

We create a 2D table `p[i][j]` where:
- `i` represents the node (from 0 to n-1)
- `j` represents the power of 2 (we use j from 0 to 17, since `2^17 = 131,072` is sufficient for most [tree](/problems/tree_intro) heights)
- `p[i][j]` stores the `2^j`-th ancestor of node `i`, or -1 if no such ancestor exists

**Initialization Phase:**

1. First, we initialize the immediate parents (j=0):
   ```python
   for i, fa in enumerate(parent):
       self.p[i][0] = fa
   ```
   Here, `p[i][0]` represents the `2^0 = 1`st ancestor, which is just the direct parent.

2. Then, we build the rest of the table using the recurrence relation:
   ```python
   for j in range(1, 18):
       for i in range(n):
           if self.p[i][j - 1] == -1:
               continue
           self.p[i][j] = self.p[self.p[i][j - 1]][j - 1]
   ```

   This implements the formula: `p[i][j] = p[p[i][j-1]][j-1]`

   The intuition is: to find the `2^j`-th ancestor of node `i`, we:
   - First find the `2^(j-1)`-th ancestor of node `i`
   - Then find the `2^(j-1)`-th ancestor of that intermediate node
   - Since `2^(j-1) + 2^(j-1) = 2^j`, this gives us the `2^j`-th ancestor

**Query Phase (getKthAncestor):**

To find the k-th ancestor, we decompose `k` into its binary representation and make jumps accordingly:

```python
for i in range(17, -1, -1):
    if k >> i & 1:
        node = self.p[node][i]
        if node == -1:
            break
```

The algorithm:
1. Iterates through bit positions from high to low (17 down to 0)
2. Checks if the i-th bit of `k` is set using `k >> i & 1`
3. If the bit is set, we jump `2^i` ancestors up by updating `node = self.p[node][i]`
4. If we reach -1 (no ancestor), we stop early

For example, if `k = 13 = 1101` in binary:
- Bit 3 is set: jump `2^3 = 8` ancestors
- Bit 2 is set: jump `2^2 = 4` ancestors
- Bit 0 is set: jump `2^0 = 1` ancestor
- Total: 8 + 4 + 1 = 13 ancestors

**Time and Space Complexity:**
- Preprocessing: `O(n × log n)` time to fill the table
- Query: `O(log n)` time to decompose k and make jumps
- Space: `O(n × log n)` for the 2D table

## Example Walkthrough

Let's walk through a concrete example to understand how binary lifting works.

**Tree Structure:**
Consider a tree with 7 nodes where the parent array is `[-1, 0, 1, 2, 3, 4, 5]`.

This creates a linear tree (essentially a path):
```
0 (root)
└── 1
    └── 2
        └── 3
            └── 4
                └── 5
                    └── 6
```

**Building the Binary Lifting Table:**

We create a table `p[i][j]` where `p[i][j]` = the `2^j`-th ancestor of node `i`.

Step 1: Initialize direct parents (`j = 0`, meaning `2^0 = 1`st ancestor):
```
p[0][0] = -1  (root has no parent)
p[1][0] = 0
p[2][0] = 1
p[3][0] = 2
p[4][0] = 3
p[5][0] = 4
p[6][0] = 5
```

Step 2: Build for `j = 1` (meaning `2^1 = 2`nd ancestor):
Using the formula `p[i][1] = p[p[i][0]][0]`:
```
p[0][1] = p[-1][0] = -1  (no 2nd ancestor for root)
p[1][1] = p[0][0] = -1   (only 1 ancestor above node 1)
p[2][1] = p[1][0] = 0    (2nd ancestor of node 2 is node 0)
p[3][1] = p[2][0] = 1
p[4][1] = p[3][0] = 2
p[5][1] = p[4][0] = 3
p[6][1] = p[5][0] = 4
```

Step 3: Build for `j = 2` (meaning `2^2 = 4`th ancestor):
Using the formula `p[i][2] = p[p[i][1]][1]`:
```
p[0][2] = -1
p[1][2] = -1
p[2][2] = -1  (node 0 has no 2nd ancestor)
p[3][2] = -1  (node 1 has no 2nd ancestor)
p[4][2] = p[2][1] = 0  (4th ancestor of node 4 is node 0)
p[5][2] = p[3][1] = 1
p[6][2] = p[4][1] = 2
```

**Query Example: Find the 5th ancestor of node 6**

We need to find `getKthAncestor(6, 5)`.

First, decompose 5 into binary: `5 = 101₂ = 4 + 1 = 2^2 + 2^0`

Now we make jumps based on the set bits:
1. Check bit 2 (value 4): It's set, so jump `2^2 = 4` ancestors
   - `node = p[6][2] = 2`
   - We're now at node 2

2. Check bit 1 (value 2): It's not set, skip

3. Check bit 0 (value 1): It's set, so jump `2^0 = 1` ancestor
   - `node = p[2][0] = 1`
   - We're now at node 1

Result: The 5th ancestor of node 6 is node 1.

**Verification:** Walking up manually from node 6: 6→5→4→3→2→1. Indeed, node 1 is 5 steps up!

**Another Query: Find the 3rd ancestor of node 4**

Decompose 3 into binary: `3 = 11₂ = 2 + 1 = 2^1 + 2^0`

Make jumps:
1. Check bit 1 (value 2): It's set, so jump `2^1 = 2` ancestors
   - `node = p[4][1] = 2`
   - We're now at node 2

2. Check bit 0 (value 1): It's set, so jump `2^0 = 1` ancestor
   - `node = p[2][0] = 1`
   - We're now at node 1

Result: The 3rd ancestor of node 4 is node 1.

This example demonstrates how binary lifting allows us to find any k-th ancestor in O(log k) jumps instead of making k individual parent traversals.

## Python Solution

```python
from typing import List


class TreeAncestor:
    def __init__(self, n: int, parent: List[int]):
        """
        Initialize the TreeAncestor data structure.

        Args:
            n: Number of nodes in the tree (nodes are numbered from 0 to n-1)
            parent: List where parent[i] is the parent of node i (-1 for root)
        """
        # Maximum power of 2 we need (2^17 > 50000, which covers most tree sizes)
        max_power = 18

        # Binary lifting table: ancestors[i][j] = 2^j-th ancestor of node i
        # -1 means the ancestor doesn't exist (we've gone above the root)
        self.ancestors = [[-1] * max_power for _ in range(n)]

        # Initialize the first column: direct parent (2^0 = 1st ancestor)
        for node, parent_node in enumerate(parent):
            self.ancestors[node][0] = parent_node

        # Build the binary lifting table using dynamic programming
        # ancestors[i][j] = ancestors[ancestors[i][j-1]][j-1]
        # This means: 2^j-th ancestor = 2^(j-1)-th ancestor of 2^(j-1)-th ancestor
        for power in range(1, max_power):
            for node in range(n):
                # If the 2^(power-1)-th ancestor doesn't exist, skip
                if self.ancestors[node][power - 1] == -1:
                    continue

                # Find the 2^(power-1)-th ancestor of the 2^(power-1)-th ancestor
                intermediate_ancestor = self.ancestors[node][power - 1]
                self.ancestors[node][power] = self.ancestors[intermediate_ancestor][power - 1]

    def getKthAncestor(self, node: int, k: int) -> int:
        """
        Find the k-th ancestor of the given node.

        Args:
            node: The node whose ancestor we want to find
            k: The number of steps to go up in the tree

        Returns:
            The k-th ancestor of node, or -1 if it doesn't exist
        """
        # Decompose k into sum of powers of 2 and jump accordingly
        # We check bits from most significant to least significant
        for bit_position in range(17, -1, -1):
            # Check if the bit at position 'bit_position' is set in k
            if (k >> bit_position) & 1:
                # Jump 2^bit_position steps up
                node = self.ancestors[node][bit_position]

                # If we've gone above the root, stop early
                if node == -1:
                    break

        return node


# Your TreeAncestor object will be instantiated and called as such:
# obj = TreeAncestor(n, parent)
# param_1 = obj.getKthAncestor(node, k)
```

## Time and Space Complexity

**Time Complexity:**
- **Initialization (`__init__`):** `O(n * log n)` where `n` is the number of nodes. The algorithm builds a binary lifting table with `18` levels (since `2^17 > 50000`, which covers the constraint). For each of the `log n` levels, we iterate through all `n` nodes to compute the `2^j`-th ancestor.

- **Query (`getKthAncestor`):** `O(log k)` where `k` is the ancestor distance. The method iterates through at most `18` bits (constant) to find the k-th ancestor by decomposing `k` into powers of 2. Each bit check and node update operation takes `O(1)` time.

**Space Complexity:**
- `O(n * log n)` for storing the binary lifting table `self.p`. The table has dimensions `n × 18`, where `n` is the number of nodes and `18` represents `⌈log₂(50000)⌉ + 1` levels needed to handle the maximum possible tree height. Each cell stores an integer representing an ancestor node index.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Binary Representation Iteration**

**Pitfall:** A common mistake is iterating through the bits of `k` in the wrong order or using an incorrect bit-checking mechanism. Some developers might try:

```python
# Wrong approach 1: Using string conversion
binary_k = bin(k)[2:]  # Convert to binary string
for i, bit in enumerate(binary_k):
    if bit == '1':
        node = self.ancestors[node][len(binary_k) - i - 1]
```

```python
# Wrong approach 2: Iterating from low to high bits incorrectly
for i in range(18):
    if k & (1 << i):  # This checks bits from low to high
        node = self.ancestors[node][i]
```

**Why it's problematic:** While both approaches might seem logical, they can lead to:
- The first approach has unnecessary string conversion overhead and complex indexing
- The second approach works but processes bits in a different order than the standard implementation

**Solution:** Use the standard bit-checking pattern that processes bits from high to low:
```python
for i in range(17, -1, -1):
    if (k >> i) & 1:  # Check if i-th bit is set
        node = self.ancestors[node][i]
        if node == -1:
            break
```

### 2. **Forgetting to Handle -1 During Table Construction**

**Pitfall:** Not checking for -1 (non-existent ancestors) when building the binary lifting table:

```python
# Wrong: This will cause index out of bounds errors
for j in range(1, 18):
    for i in range(n):
        self.ancestors[i][j] = self.ancestors[self.ancestors[i][j-1]][j-1]
```

**Why it's problematic:** If `self.ancestors[i][j-1]` is -1 (meaning the ancestor doesn't exist), trying to access `self.ancestors[-1][j-1]` will either:
- Return an unexpected value (Python allows negative indexing)
- Cause errors in other languages

**Solution:** Always check for -1 before using it as an index:
```python
for j in range(1, 18):
    for i in range(n):
        if self.ancestors[i][j - 1] == -1:
            continue  # Or set self.ancestors[i][j] = -1
        self.ancestors[i][j] = self.ancestors[self.ancestors[i][j-1]][j-1]
```

### 3. **Insufficient Table Size for Large Trees**

**Pitfall:** Using a fixed table size that's too small:

```python
# Wrong: Only supports trees up to depth 15
max_power = 16  # 2^15 = 32,768
```

**Why it's problematic:** If the tree has a depth greater than 2^(max_power-1), some queries will fail to find ancestors that actually exist.

**Solution:** Calculate the appropriate table size based on the maximum possible tree depth:
```python
import math

def __init__(self, n: int, parent: List[int]):
    # Calculate the maximum power needed
    max_power = math.ceil(math.log2(n)) + 1 if n > 1 else 1
    # Or use a safe upper bound like 18 (2^17 = 131,072)
    max_power = min(18, max_power)
```

### 4. **Not Breaking Early When Node Becomes -1**

**Pitfall:** Continuing to process bits even after the node becomes -1:

```python
# Wrong: Doesn't stop when ancestor doesn't exist
def getKthAncestor(self, node: int, k: int) -> int:
    for i in range(17, -1, -1):
        if (k >> i) & 1:
            node = self.ancestors[node][i]
    return node
```

**Why it's problematic:** Once `node` becomes -1, accessing `self.ancestors[-1][i]` might return unexpected values due to Python's negative indexing, potentially returning a wrong ancestor instead of -1.

**Solution:** Break immediately when node becomes -1:
```python
def getKthAncestor(self, node: int, k: int) -> int:
    for i in range(17, -1, -1):
        if (k >> i) & 1:
            node = self.ancestors[node][i]
            if node == -1:
                break  # Stop early
    return node
```

### 5. **Misunderstanding the Parent Array Format**

**Pitfall:** Assuming parent[0] should be 0 (self-loop) instead of -1:

```python
# Wrong assumption about input format
if parent[0] != -1:
    parent[0] = -1  # "Fixing" the root
```

**Why it's problematic:** The problem specification states that the root node (node 0) has no parent, represented as -1 in the parent array. Modifying this breaks the tree structure.

**Solution:** Accept the parent array as-is, where parent[0] = -1 indicates the root:
```python
# Correct: Use parent array directly
for node, parent_node in enumerate(parent):
    self.ancestors[node][0] = parent_node  # parent[0] will be -1
```

