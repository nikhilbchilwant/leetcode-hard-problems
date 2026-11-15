# 1719. Number Of Ways To Reconstruct A Tree

> Source: https://algo.monster/liteproblems/1719

## Problem Statement

You're given an array `pairs` where each element `pairs[i] = [xi, yi]` represents a relationship between two nodes. The array has these properties:
- No duplicate pairs exist
- For each pair, `xi < yi`

Your task is to determine how many different rooted trees can be constructed that satisfy these specific conditions:

1. **Node composition**: The [tree](/problems/tree_intro) must contain exactly the nodes that appear in the `pairs` array (no more, no less).

2. **Ancestor relationship rule**: A pair `[xi, yi]` exists in the input **if and only if** one node is an ancestor of the other. This means:
   - If `[xi, yi]` is in `pairs`, then either `xi` must be an ancestor of `yi`, or `yi` must be an ancestor of `xi`
   - If two nodes have an ancestor-descendant relationship in the [tree](/problems/tree_intro), their pair must exist in `pairs`

3. **[Tree](/problems/tree_intro) structure**: The tree doesn't have to be binary - nodes can have any number of children.

Two [tree](/problems/tree_intro) configurations are considered different if at least one node has a different parent between the two trees.

**Return values:**
- Return `0` if no valid trees can be constructed (`ways == 0`)
- Return `1` if exactly one valid [tree](/problems/tree_intro) can be constructed (`ways == 1`)  
- Return `2` if more than one valid tree can be constructed (`ways > 1`)

**Key definitions:**
- A **rooted [tree](/problems/tree_intro)** has a single root node with all edges directed away from the root
- An **ancestor** of a node is any node on the path from the root to that node (not including the node itself)
- The root node has no ancestors

## Intuition

The key insight is that if two nodes have an ancestor-descendant relationship in the [tree](/problems/tree_intro), then one must be on the path from the root to the other. This means they must be connected through a chain of ancestor relationships with all intermediate nodes.

Think about what the pairs tell us: if `[x, y]` is a pair, then in any valid [tree](/problems/tree_intro), one must be an ancestor of the other. Now, if `x` is an ancestor of `y`, and we have another node `z` where `[x, z]` is also a pair, then either:
- `z` is between `x` and `y` on the path (making `[y, z]` also a required pair)
- `z` is on a different branch from `y` (no pair between `y` and `z`)

This leads to a crucial observation: nodes with more connections (higher degree) are more likely to be ancestors of nodes with fewer connections. Why? Because an ancestor node must have pairs with all its descendants, plus potentially its own ancestors.

The degree of a node in the pair relationships gives us a hint about its position in the [tree](/problems/tree_intro) hierarchy. A node with degree `n-1` (connected to all other nodes) must be the root or very close to it, as it has an ancestor-descendant relationship with every other node.

To verify if a [tree](/problems/tree_intro) structure is valid, we can use a greedy approach:
1. Sort nodes by their degree (number of connections)
2. For each node, find a potential parent among nodes with higher or equal degree
3. When we assign a parent `p` to node `x`, all nodes connected to `x` must also be connected to `p` (transitivity of ancestor relationships)

The solution can have multiple valid trees when two nodes have the same degree - they could potentially swap positions in the hierarchy without violating the ancestor-descendant requirements. This is why we track if any two nodes have equal degrees.

If we can't find a valid parent for any node (except the root), or if we end up with multiple roots, then no valid [tree](/problems/tree_intro) exists.

**Learn more about [Tree](/problems/tree_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The implementation uses a greedy algorithm with [graph](/problems/graph_intro) representation to validate and count possible [tree](/problems/tree_intro) structures.

**Data Structure Setup:**
- Create a 2D boolean array `g[510][510]` to represent the adjacency relationships between nodes
- Use a dictionary `v` to store the adjacency list for each node (neighbors of each node)
- For each pair `[x, y]`, mark `g[x][y] = g[y][x] = True` and add them to each other's adjacency lists

**Node Collection and Preprocessing:**
```python
nodes = []
for i in range(510):
    if v[i]:  # If node i exists in pairs
        nodes.append(i)
        g[i][i] = True  # Mark self-connection for easier processing
```

**Sorting by Degree:**
Sort all nodes by their degree (number of connections) in ascending order:
```python
nodes.sort(key=lambda x: len(v[x]))
```

**Greedy Parent Assignment:**
For each node `x` in sorted order:
1. Look for a potential parent among nodes with higher or equal degree:
   ```python
   j = i + 1
   while j < len(nodes) and not g[x][nodes[j]]:
       j += 1
   ```

2. If a potential parent `y` is found:
   - Check if they have equal degrees: `len(v[x]) == len(v[y])`
     - If yes, set `equal = True` (multiple trees possible)
   - Verify transitivity: All neighbors of `x` must also be connected to `y`
     ```python
     for z in v[x]:
         if not g[y][z]:
             return 0  # Invalid [tree](/problems/tree_intro) structure
     ```

3. If no parent is found, increment the root counter:
   ```python
   else:
       root += 1
   ```

**Result Determination:**
- If `root > 1`: Multiple roots exist, so no valid [tree](/problems/tree_intro) is possible → return `0`
- If `equal` is `True`: At least two nodes had equal degrees, allowing position swaps → return `2`
- Otherwise: Exactly one valid tree structure exists → return `1`

The algorithm efficiently validates the [tree](/problems/tree_intro) structure by ensuring:
1. Each non-root node has exactly one valid parent
2. The transitivity property holds (if `x` is ancestor of `y` and `y` is ancestor of `z`, then `x` must be ancestor of `z`)
3. All required pairs are satisfied by the ancestor-descendant relationships

## Example Walkthrough

Let's walk through a small example with `pairs = [[1,2], [2,3], [1,3]]`.

**Step 1: Build the graph representation**
- Create adjacency matrix `g` and adjacency list `v`
- From pairs: `g[1][2] = g[2][1] = True`, `g[2][3] = g[3][2] = True`, `g[1][3] = g[3][1] = True`
- Adjacency lists: `v[1] = [2,3]`, `v[2] = [1,3]`, `v[3] = [1,2]`

**Step 2: Collect nodes and mark self-connections**
- Nodes in the tree: `[1, 2, 3]`
- Set `g[1][1] = g[2][2] = g[3][3] = True`

**Step 3: Sort nodes by degree**
- Node degrees: `1` has degree 2, `2` has degree 2, `3` has degree 2
- After sorting: `nodes = [1, 2, 3]` (all have same degree)

**Step 4: Greedy parent assignment**

Processing node `1` (index 0):
- Look for parent starting from index 1 (node `2`)
- Check if `g[1][2]` is True → Yes!
- Node `2` has same degree as node `1` → Set `equal = True`
- Verify transitivity: All neighbors of `1` (which are `[2,3]`) must connect to `2`
  - `g[2][2]` = True ✓
  - `g[2][3]` = True ✓
- Parent found successfully

Processing node `2` (index 1):
- Look for parent starting from index 2 (node `3`)
- Check if `g[2][3]` is True → Yes!
- Node `3` has same degree as node `2` → `equal` remains `True`
- Verify transitivity: All neighbors of `2` (which are `[1,3]`) must connect to `3`
  - `g[3][1]` = True ✓
  - `g[3][3]` = True ✓
- Parent found successfully

Processing node `3` (index 2):
- No more nodes to check for parent
- Increment `root = 1` (this is our root node)

**Step 5: Determine result**
- `root = 1` (exactly one root)
- `equal = True` (nodes had equal degrees)
- Return `2` (multiple valid trees possible)

**Why multiple trees?** Since all nodes have the same degree, we could have:
- Tree 1: `3` as root → `2` as child → `1` as child (chain: 3→2→1)
- Tree 2: `3` as root → `1` as child → `2` as child (chain: 3→1→2)
- Tree 3: `2` as root → `3` as child → `1` as child (chain: 2→3→1)
- And so on...

Each configuration maintains all required ancestor-descendant relationships from the pairs.

## Python Solution

```python
class Solution:
    def checkWays(self, pairs: List[List[int]]) -> int:
        # Create adjacency matrix to track connections between nodes
        adjacency_matrix = [[False] * 510 for _ in range(510)]
      
        # Create adjacency list to store neighbors for each node
        adjacency_list = defaultdict(list)
      
        # Build the graph from the given pairs
        for node1, node2 in pairs:
            adjacency_matrix[node1][node2] = adjacency_matrix[node2][node1] = True
            adjacency_list[node1].append(node2)
            adjacency_list[node2].append(node1)
      
        # Collect all nodes that appear in the pairs
        active_nodes = []
        for node_id in range(510):
            if adjacency_list[node_id]:
                active_nodes.append(node_id)
                # Mark self-connection as true for active nodes
                adjacency_matrix[node_id][node_id] = True
      
        # Sort nodes by their degree (number of connections) in ascending order
        active_nodes.sort(key=lambda node: len(adjacency_list[node]))
      
        # Track if there are multiple valid trees possible
        multiple_solutions_exist = False
      
        # Count potential root nodes
        root_count = 0
      
        # Check each node to find its potential parent in the tree
        for current_index, current_node in enumerate(active_nodes):
            # Look for the first node with higher or equal degree that is connected
            parent_index = current_index + 1
            while parent_index < len(active_nodes) and not adjacency_matrix[current_node][active_nodes[parent_index]]:
                parent_index += 1
          
            if parent_index < len(active_nodes):
                # Found a potential parent node
                parent_node = active_nodes[parent_index]
              
                # Check if degrees are equal (multiple valid parent choices)
                if len(adjacency_list[current_node]) == len(adjacency_list[parent_node]):
                    multiple_solutions_exist = True
              
                # Verify that all neighbors of current node are also connected to parent
                # This is necessary for a valid tree structure
                for neighbor in adjacency_list[current_node]:
                    if not adjacency_matrix[parent_node][neighbor]:
                        return 0  # Invalid tree structure
            else:
                # No parent found, this node could be a root
                root_count += 1
      
        # A valid tree must have exactly one root
        if root_count > 1:
            return 0  # Invalid: multiple disconnected components
      
        # Return 2 if multiple valid trees exist, 1 if unique tree exists
        return 2 if multiple_solutions_exist else 1
```

## Time and Space Complexity

**Time Complexity:** `O(n² + m·n)` where `n` is the number of unique nodes and `m` is the number of pairs.

- Building the adjacency matrix `g` and adjacency list `v` from pairs takes `O(m)` time
- Extracting unique nodes by iterating through 510 positions takes `O(510) = O(1)` time
- Sorting nodes by degree takes `O(n log n)` time
- The main loop iterates through each node (`O(n)`):
  - Finding the next connected node with higher degree: worst case `O(n)` iterations
  - Checking if all neighbors of `x` are connected to `y`: `O(degree(x))` which can be `O(n)` in worst case
- Overall: `O(m) + O(n log n) + O(n²·n) = O(n³)` in worst case, but more precisely `O(n² + m·n)` considering the actual graph structure

**Space Complexity:** `O(510² + n + m) = O(1)` since 510 is a constant

- The adjacency matrix `g` uses `510 × 510 = O(1)` space (fixed size)
- The adjacency list `v` stores at most `O(m)` edges across all nodes
- The `nodes` list stores at most `O(n)` unique nodes
- Since the maximum node value is bounded by 510, the space is effectively `O(1)` constant space

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Misunderstanding the Ancestor-Descendant Requirement**
**Pitfall**: A common mistake is thinking that the pairs represent direct parent-child relationships. In reality, pairs represent ANY ancestor-descendant relationship, not just immediate parent-child connections.

**Example**: If we have pairs `[[1,2], [1,3], [2,3]]`, node 1 could be the root with node 2 as its child, and node 3 as the child of node 2. The pair `[1,3]` exists because 1 is an ancestor of 3 (grandparent relationship).

**Solution**: When verifying tree validity, ensure that ALL descendants of a node are connected to its ancestors in the adjacency matrix, not just direct children.

### 2. **Incorrect Handling of Equal Degree Nodes**
**Pitfall**: When two nodes have the same degree and are connected, the algorithm marks `multiple_solutions_exist = True`. However, developers often forget that this only applies when one could be the parent of the other, not when they're siblings.

**Example**: If nodes A and B both have degree 3 and are connected, they could potentially swap positions in the tree hierarchy, but only if one can be the ancestor of the other while maintaining all required connections.

**Solution**: The check for equal degrees should only trigger the multiple solutions flag when the node with equal degree is actually chosen as the parent:
```python
if parent_index < len(active_nodes):
    parent_node = active_nodes[parent_index]
    # Only mark multiple solutions if this parent has equal degree
    if len(adjacency_list[current_node]) == len(adjacency_list[parent_node]):
        multiple_solutions_exist = True
```

### 3. **Forgetting the Transitivity Check**
**Pitfall**: Not verifying that all neighbors of a child node are also connected to its chosen parent. This transitivity property is crucial for maintaining the ancestor-descendant relationships.

**Example**: If node X has neighbors [Y, Z] and we assign P as X's parent, then P must also be connected to both Y and Z. Otherwise, the ancestor relationships would be violated.

**Solution**: Always validate transitivity when assigning a parent:
```python
for neighbor in adjacency_list[current_node]:
    if not adjacency_matrix[parent_node][neighbor]:
        return 0  # Invalid tree structure
```

### 4. **Inefficient Node Range Iteration**
**Pitfall**: Using a fixed range of 510 nodes when the actual node values might be sparse or have a different range. This can lead to unnecessary memory usage and potential index out of bounds errors.

**Solution**: Dynamically determine the range of nodes or use a more flexible data structure:
```python
# Better approach: determine max node value first
max_node = max(max(pair) for pair in pairs) if pairs else 0
adjacency_matrix = [[False] * (max_node + 1) for _ in range(max_node + 1)]

# Or use a dictionary-based approach for sparse graphs
adjacency_matrix = defaultdict(lambda: defaultdict(bool))
```

### 5. **Not Handling Edge Cases**
**Pitfall**: Failing to handle special cases like empty input, single pair, or when all nodes are fully connected.

**Solution**: Add explicit checks for edge cases:
```python
def checkWays(self, pairs: List[List[int]]) -> int:
    if not pairs:
        return 1  # Empty tree is valid
  
    if len(pairs) == 1:
        return 1  # Single edge forms a valid tree
  
    # Continue with main algorithm...
```

