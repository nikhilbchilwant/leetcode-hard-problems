# 2509. Cycle Length Queries in a Tree

> Source: https://algo.monster/liteproblems/2509

## Problem Statement

You have a complete binary [tree](/problems/tree_intro) with `2^n - 1` nodes. The root node has value `1`, and for any node with value `val` in the range `[1, 2^(n-1) - 1]`:
- Its left child has value `2 * val`
- Its right child has value `2 * val + 1`

Given a list of queries where each query `[a_i, b_i]` represents two node values, you need to:
1. Temporarily add an edge between nodes `a_i` and `b_i`
2. Find the length of the cycle created (number of edges in the cycle)
3. Remove the added edge

For example, in a complete binary [tree](/problems/tree_intro) with `n = 3`:
- Node 1 (root) has children 2 and 3
- Node 2 has children 4 and 5
- Node 3 has children 6 and 7

When you add an edge between two nodes that already have a path between them in the [tree](/problems/tree_intro), you create exactly one cycle. The cycle length is the number of edges you traverse to go from one node back to itself using the newly added edge exactly once.

The solution finds the cycle length by locating the Lowest Common Ancestor (LCA) of nodes `a` and `b`. Since adding an edge between `a` and `b` creates a cycle, the cycle consists of:
- The path from `a` to their LCA
- The path from `b` to their LCA  
- The newly added edge between `a` and `b`

To find the LCA efficiently, the code uses the property that a node's parent has value `node >> 1` (integer division by 2). It repeatedly moves the larger-valued node up to its parent until both nodes meet at their LCA. The total number of steps taken plus 1 (for the added edge) gives the cycle length.

Return an array where the `i`-th element is the cycle length for the `i`-th query.

## Intuition

When we add an edge between two nodes in a [tree](/problems/tree_intro), we create exactly one cycle. To find the length of this cycle, we need to understand what path forms the cycle.

The key insight is that in a complete binary [tree](/problems/tree_intro), every pair of nodes already has exactly one path connecting them through the tree structure. When we add a direct edge between nodes `a` and `b`, the cycle consists of:
- The original tree path from `a` to `b`
- The newly added direct edge from `b` back to `a`

To find the length of the original [tree](/problems/tree_intro) path, we need to find their Lowest Common Ancestor (LCA). Why? Because the path from `a` to `b` in the tree must go through their LCA - it's the only way to connect them using existing tree edges.

The brilliant observation here is the structure of the complete binary [tree](/problems/tree_intro) with its numbering scheme. For any node with value `val`:
- Parent node = `val >> 1` (or `val // 2`)
- Left child = `2 * val`
- Right child = `2 * val + 1`

This means we can find the LCA without explicitly storing the [tree](/problems/tree_intro) structure! We can navigate upward from any node by simply dividing by 2.

The algorithm becomes elegant: starting from nodes `a` and `b`, we repeatedly move the larger-valued node up to its parent (by right-shifting or dividing by 2). Why the larger one? Because nodes with larger values are deeper in the [tree](/problems/tree_intro) or in a different subtree. By always moving the larger node up, we guarantee both nodes will eventually meet at their LCA.

Each upward movement represents traversing one edge in the [tree](/problems/tree_intro) path. Once `a == b` (they meet at the LCA), the total number of movements equals the distance from `a` to LCA plus the distance from `b` to LCA. Adding 1 for the newly added direct edge gives us the cycle length.

**Learn more about [Tree](/problems/tree_intro) and [Binary Tree](/problems/binary_tree_min_depth) patterns.**

## Solution Approach

The solution implements the LCA-finding algorithm efficiently using bit manipulation. Here's how the implementation works:

For each query `[a, b]`, we need to find the cycle length when adding an edge between nodes `a` and `b`.

**Algorithm Steps:**

1. **Initialize counter**: Start with `t = 1` to account for the newly added edge that will be part of the cycle.

2. **Find LCA by moving up the [tree](/problems/tree_intro)**: While `a != b` (they haven't met at their LCA yet):
   - If `a > b`: Move `a` to its parent by executing `a >>= 1` (right shift by 1, equivalent to integer division by 2)
   - If `b > a`: Move `b` to its parent by executing `b >>= 1`
   - Increment the counter `t` for each movement (each movement represents traversing one edge)

3. **Return the result**: Once `a == b`, they've met at their LCA, and `t` contains the total cycle length.

**Why this works:**

- The right shift operation `>>= 1` efficiently computes the parent node in the binary [tree](/problems/tree_intro) due to the numbering scheme
- By always moving the larger-valued node upward, we ensure both nodes converge at their LCA
- The counter `t` accumulates:
  - The number of edges from node `a` to the LCA
  - The number of edges from node `b` to the LCA
  - Plus 1 for the newly added edge

**Time Complexity**: `O(m * log n)` where `m` is the number of queries and `log n` is the maximum [tree](/problems/tree_intro) depth. Each query requires at most `O(log n)` operations to find the LCA.

**Space Complexity**: `O(1)` for each query as we only use a few variables. The output array takes `O(m)` space.

**Example walkthrough:**

For nodes `a = 5` and `b = 3`:
- Step 1: `a = 5, b = 3, t = 1` → Since `5 > 3`, move `a`: `a = 5 >> 1 = 2`
- Step 2: `a = 2, b = 3, t = 2` → Since `3 > 2`, move `b`: `b = 3 >> 1 = 1`
- Step 3: `a = 2, b = 1, t = 3` → Since `2 > 1`, move `a`: `a = 2 >> 1 = 1`
- Step 4: `a = 1, b = 1, t = 4` → They meet at LCA = 1
- Return `t = 4` as the cycle length

## Example Walkthrough

Let's walk through a complete example with a tree where `n = 3` (7 nodes total) and query `[4, 6]`.

**Tree Structure:**
```
       1
      / \
     2   3
    / \ / \
   4  5 6  7
```

**Query: Add edge between nodes 4 and 6**

When we add an edge between nodes 4 and 6, we create a cycle. Let's trace through the algorithm:

**Initial State:**
- `a = 4`, `b = 6`, `t = 1` (starting with 1 for the new edge we're adding)

**Step 1:** Compare `a` and `b`
- Since `6 > 4`, we move node 6 up to its parent
- `b = 6 >> 1 = 3` (node 6's parent is node 3)
- Increment `t = 2` (we traversed one edge: 6→3)
- Current state: `a = 4`, `b = 3`, `t = 2`

**Step 2:** Compare `a` and `b`
- Since `4 > 3`, we move node 4 up to its parent
- `a = 4 >> 1 = 2` (node 4's parent is node 2)
- Increment `t = 3` (we traversed another edge: 4→2)
- Current state: `a = 2`, `b = 3`, `t = 3`

**Step 3:** Compare `a` and `b`
- Since `3 > 2`, we move node 3 up to its parent
- `b = 3 >> 1 = 1` (node 3's parent is node 1)
- Increment `t = 4` (we traversed another edge: 3→1)
- Current state: `a = 2`, `b = 1`, `t = 4`

**Step 4:** Compare `a` and `b`
- Since `2 > 1`, we move node 2 up to its parent
- `a = 2 >> 1 = 1` (node 2's parent is node 1)
- Increment `t = 5` (we traversed another edge: 2→1)
- Current state: `a = 1`, `b = 1`, `t = 5`

**Step 5:** Check loop condition
- Since `a == b == 1`, we've found the LCA (node 1)
- Return `t = 5`

**Verification:**
The cycle formed consists of:
1. Path from 4 to LCA(1): 4→2→1 (2 edges)
2. Path from 6 to LCA(1): 6→3→1 (2 edges)
3. New edge: 4→6 (1 edge)
Total: 2 + 2 + 1 = 5 edges ✓

The algorithm correctly identifies that adding an edge between nodes 4 and 6 creates a cycle of length 5.

## Python Solution

```python
class Solution:
    def cycleLengthQueries(self, n: int, queries: List[List[int]]) -> List[int]:
        """
        Calculate cycle lengths for pairs of nodes in a complete binary tree.
      
        Args:
            n: The height of the binary tree
            queries: List of node pairs [a, b] to find cycle length between
          
        Returns:
            List of cycle lengths for each query
        """
        result = []
      
        for node_a, node_b in queries:
            # Initialize cycle length counter (starts at 1 for the initial edge)
            cycle_length = 1
          
            # Move both nodes up the tree until they meet at their common ancestor
            while node_a != node_b:
                if node_a > node_b:
                    # Move the larger node up to its parent (right shift equals division by 2)
                    node_a >>= 1
                else:
                    # Move the smaller node up to its parent
                    node_b >>= 1
                  
                # Increment the cycle length for each step taken
                cycle_length += 1
          
            # Add the computed cycle length to results
            result.append(cycle_length)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(m × log(max(a, b)))`

For each query in the `queries` array (total `m` queries), the while loop runs until `a` equals `b`. In each iteration, either `a` or `b` is halved (right shift by 1). Since we're dealing with a complete binary tree of depth `n`, the maximum value for any node is `2^n - 1`. The worst case occurs when one node is at the deepest level and the other is at a shallow level, or when both nodes are far apart in the tree. The number of iterations is bounded by the height of the tree, which is at most `n` for a complete binary tree. However, more precisely, each iteration reduces the larger value by half, so the loop runs at most `O(log(max(a, b)))` times. Therefore, the total time complexity is `O(m × log(max(a, b)))`, which can be simplified to `O(m × n)` since `log(max(a, b)) ≤ n` for nodes in a tree of depth `n`.

**Space Complexity:** `O(m)`

The algorithm uses an array `ans` to store the results for all `m` queries. Apart from this output array and a few constant variables (`t`, `a`, `b` in each iteration), no additional data structures are used. Therefore, the space complexity is `O(m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Initial Counter Value**
A common mistake is initializing the counter to 0 instead of 1, forgetting to account for the newly added edge between nodes `a` and `b`.

**Incorrect:**
```python
cycle_length = 0  # Wrong: misses the new edge
while node_a != node_b:
    # ... rest of logic
```

**Correct:**
```python
cycle_length = 1  # Accounts for the new edge between a and b
while node_a != node_b:
    # ... rest of logic
```

### 2. **Moving Both Nodes Simultaneously**
Some might try to move both nodes up at the same time, which can cause them to miss their LCA and enter an infinite loop or get incorrect results.

**Incorrect:**
```python
while node_a != node_b:
    node_a >>= 1
    node_b >>= 1  # Wrong: might skip past LCA
    cycle_length += 2
```

**Correct:**
```python
while node_a != node_b:
    if node_a > node_b:
        node_a >>= 1
    else:
        node_b >>= 1
    cycle_length += 1  # Only increment once per move
```

### 3. **Using Division Instead of Bit Shift**
While `node // 2` works correctly, using bit shift `node >> 1` is more efficient. However, mixing assignment operators can lead to bugs.

**Incorrect:**
```python
if node_a > node_b:
    node_a = node_a >> 1  # Works but inconsistent
else:
    node_b //= 2  # Mixing operators can be confusing
```

**Better:**
```python
if node_a > node_b:
    node_a >>= 1  # Consistent use of >>= operator
else:
    node_b >>= 1
```

### 4. **Not Handling Edge Cases**
While the problem guarantees valid inputs, in practice you might want to validate that nodes exist in the tree.

**Potential issue:**
```python
# Assuming queries might contain invalid node values
for node_a, node_b in queries:
    # No validation that nodes are in range [1, 2^n - 1]
```

**More robust (if needed):**
```python
max_node = (1 << n) - 1  # 2^n - 1
for node_a, node_b in queries:
    if not (1 <= node_a <= max_node and 1 <= node_b <= max_node):
        # Handle invalid input
        continue
```

### 5. **Misunderstanding the Cycle Length Calculation**
Some might think the cycle length is just the distance between two nodes, forgetting to add 1 for the new edge.

**Incorrect interpretation:**
```python
# Wrong: Only counting existing tree edges
distance = 0
while node_a != node_b:
    if node_a > node_b:
        node_a >>= 1
    else:
        node_b >>= 1
    distance += 1
result.append(distance)  # Missing the +1 for new edge
```

**Correct:**
The cycle includes both the existing path through the tree AND the newly added edge, so we start counting from 1.

