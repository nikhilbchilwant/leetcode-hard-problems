# 1579. Remove Max Number of Edges to Keep Graph Fully Traversable

> Source: https://algo.monster/liteproblems/1579

## Problem Statement

This problem involves a [graph](/problems/graph_intro) with `n` nodes that needs to be traversable by both Alice and Bob. The graph has three types of edges:
- **Type 1**: Only Alice can use these edges
- **Type 2**: Only Bob can use these edges  
- **Type 3**: Both Alice and Bob can use these edges

You're given an array `edges` where each element `edges[i] = [type_i, u_i, v_i]` represents a bidirectional edge of type `type_i` connecting nodes `u_i` and `v_i`.

The goal is to find the **maximum number of edges you can remove** while ensuring that:
- After removal, Alice can still reach every node from any starting node (the [graph](/problems/graph_intro) remains fully connected for Alice)
- After removal, Bob can still reach every node from any starting node (the graph remains fully connected for Bob)

If it's impossible for both Alice and Bob to fully traverse the [graph](/problems/graph_intro) even without removing any edges, return `-1`.

The solution uses a Union-Find (Disjoint Set Union) data structure to efficiently track connectivity. The key insight is to:
1. First process Type 3 edges (usable by both), as they're most valuable for maintaining connectivity
2. Keep an edge only if it connects previously disconnected components
3. Count redundant edges (those connecting already-connected components) as removable
4. Check if both Alice's and Bob's graphs are fully connected at the end

## Intuition

The key insight is to think about this problem in reverse - instead of removing edges, we should think about which edges we **must keep** to maintain connectivity for both Alice and Bob.

For a [graph](/problems/graph_intro) with `n` nodes to be fully connected, we need exactly `n-1` edges (this forms a spanning tree). Since both Alice and Bob need their own connected graphs, we might initially think we need `2*(n-1)` edges total. However, Type 3 edges are special - they can serve both Alice and Bob simultaneously, potentially reducing the total edges needed.

This leads us to a greedy strategy: **Type 3 edges are the most valuable** because one Type 3 edge can satisfy connectivity requirements for both Alice and Bob. We should prioritize keeping Type 3 edges over Type 1 or Type 2 edges whenever possible.

The approach becomes clear:
1. **Process Type 3 edges first** - Each Type 3 edge that connects previously disconnected components helps both Alice and Bob simultaneously. If a Type 3 edge connects nodes already connected, it's redundant and can be removed.

2. **Then process Type 1 and Type 2 edges** - After using Type 3 edges, we fill in the remaining gaps in Alice's and Bob's connectivity with their specific edge types. Again, any edge connecting already-connected nodes is redundant.

We use Union-Find to efficiently track which nodes are already connected. When we try to union two nodes:
- If they're in different components, the edge is necessary (union returns `True`)
- If they're already in the same component, the edge is redundant (union returns `False`)

The total removable edges equals the number of redundant edges we encounter. Finally, we verify both graphs are fully connected (each has exactly 1 component) - if not, we return `-1`.

**Learn more about [Union Find](/problems/dsu_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The solution implements two Union-Find data structures to track connectivity for Alice and Bob separately.

**Union-Find Implementation:**
- `p`: Parent array where `p[i]` stores the parent of node `i`
- `size`: Array tracking the size of each component for union by rank optimization
- `cnt`: Counter tracking the number of disjoint components (starts at `n`, decreases with each successful union)
- `find(x)`: Finds the root of node `x` with path compression
- `union(a, b)`: Merges components containing nodes `a` and `b`. Returns `True` if they were in different components (edge is necessary), `False` if already connected (edge is redundant)

**Main Algorithm:**

1. **Initialize two Union-Find structures**: `ufa` for Alice and `ufb` for Bob, each with `n` nodes.

2. **First pass - Process Type 3 edges:**
   ```python
   for t, u, v in edges:
       if t == 3:
           if ufa.union(u, v):  # Try to union in Alice's [graph](/problems/graph_intro)
               ufb.union(u, v)  # If successful, also union in Bob's graph
           else:
               ans += 1  # Edge is redundant, can be removed
   ```
   When a Type 3 edge successfully connects components in Alice's graph, we also apply it to Bob's graph. If it's redundant for Alice, it's redundant for both.

3. **Second pass - Process Type 1 and Type 2 edges:**
   ```python
   for t, u, v in edges:
       if t == 1:
           ans += not ufa.union(u, v)  # Count redundant edges for Alice
       if t == 2:
           ans += not ufb.union(u, v)  # Count redundant edges for Bob
   ```
   For each type-specific edge, we attempt to union the nodes. If `union` returns `False`, the edge is redundant and we increment our removable edge count.

4. **Final validation:**
   ```python
   return ans if ufa.cnt == 1 and ufb.cnt == 1 else -1
   ```
   Check if both graphs are fully connected (each has exactly 1 component). If either has more than 1 component, return `-1` as full traversal is impossible.

**Note:** The solution uses 1-indexed nodes in the input, so `union(u, v)` internally converts to 0-indexed with `find(a - 1)` and `find(b - 1)`.

The time complexity is `O(E * α(n))` where `E` is the number of edges and `α` is the inverse Ackermann function (practically constant). Space complexity is `O(n)` for the Union-Find structures.

## Example Walkthrough

Let's walk through a small example with `n = 4` nodes and the following edges:
```
edges = [[3,1,2], [3,2,3], [1,1,3], [1,2,4], [1,1,2], [2,3,4]]
```

**Initial Setup:**
- Create two Union-Find structures for Alice and Bob
- Both start with 4 separate components: {1}, {2}, {3}, {4}
- Alice's components: 4, Bob's components: 4
- Removable edges counter: 0

**First Pass - Process Type 3 edges:**

1. **Edge [3,1,2]**: Type 3 edge connecting nodes 1 and 2
   - Alice: Union(1,2) → Success! Components become {1,2}, {3}, {4} (count: 3)
   - Bob: Also union(1,2) → Components become {1,2}, {3}, {4} (count: 3)
   - This edge is necessary, removable count stays at 0

2. **Edge [3,2,3]**: Type 3 edge connecting nodes 2 and 3
   - Alice: Union(2,3) → Success! Components become {1,2,3}, {4} (count: 2)
   - Bob: Also union(2,3) → Components become {1,2,3}, {4} (count: 2)
   - This edge is necessary, removable count stays at 0

**Second Pass - Process Type 1 and Type 2 edges:**

3. **Edge [1,1,3]**: Type 1 edge (Alice only) connecting nodes 1 and 3
   - Alice: Union(1,3) → Fail! Nodes 1 and 3 are already connected through 2
   - This edge is redundant, removable count: 0 + 1 = 1

4. **Edge [1,2,4]**: Type 1 edge (Alice only) connecting nodes 2 and 4
   - Alice: Union(2,4) → Success! Components become {1,2,3,4} (count: 1)
   - This edge is necessary, removable count stays at 1

5. **Edge [1,1,2]**: Type 1 edge (Alice only) connecting nodes 1 and 2
   - Alice: Union(1,2) → Fail! Already connected
   - This edge is redundant, removable count: 1 + 1 = 2

6. **Edge [2,3,4]**: Type 2 edge (Bob only) connecting nodes 3 and 4
   - Bob: Union(3,4) → Success! Components become {1,2,3,4} (count: 1)
   - This edge is necessary, removable count stays at 2

**Final Check:**
- Alice's component count: 1 ✓ (fully connected)
- Bob's component count: 1 ✓ (fully connected)
- Both graphs are fully connected, so we return the removable count: **2**

The edges we can remove are [1,1,3] and [1,1,2], both Type 1 edges that were redundant for Alice's connectivity.

## Python Solution

```python
from typing import List


class UnionFind:
    """Union-Find (Disjoint Set Union) data structure with path compression and union by size."""
  
    def __init__(self, n: int) -> None:
        """
        Initialize Union-Find structure with n nodes.
      
        Args:
            n: Number of nodes (0-indexed internally)
        """
        # Parent array - initially each node is its own parent
        self.parent = list(range(n))
        # Size of each component - initially all components have size 1
        self.component_size = [1] * n
        # Number of connected components
        self.component_count = n

    def find(self, x: int) -> int:
        """
        Find the root of the component containing node x.
        Uses path compression for optimization.
      
        Args:
            x: Node to find the root for
          
        Returns:
            Root node of the component containing x
        """
        if self.parent[x] != x:
            # Path compression: make all nodes point directly to root
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, node_a: int, node_b: int) -> bool:
        """
        Unite two components containing nodes a and b.
        Uses union by size for optimization.
      
        Args:
            node_a: First node (1-indexed, converted to 0-indexed internally)
            node_b: Second node (1-indexed, converted to 0-indexed internally)
          
        Returns:
            True if union was performed (nodes were in different components),
            False if nodes were already in the same component
        """
        # Convert from 1-indexed to 0-indexed
        root_a = self.find(node_a - 1)
        root_b = self.find(node_b - 1)
      
        # Already in the same component
        if root_a == root_b:
            return False
      
        # Union by size: attach smaller tree under root of larger tree
        if self.component_size[root_a] > self.component_size[root_b]:
            self.parent[root_b] = root_a
            self.component_size[root_a] += self.component_size[root_b]
        else:
            self.parent[root_a] = root_b
            self.component_size[root_b] += self.component_size[root_a]
      
        # Decrease component count as two components merged into one
        self.component_count -= 1
        return True


class Solution:
    def maxNumEdgesToRemove(self, n: int, edges: List[List[int]]) -> int:
        """
        Find maximum number of edges that can be removed while keeping graph 
        fully traversable for both Alice and Bob.
      
        Edge types:
        - Type 1: Only Alice can traverse
        - Type 2: Only Bob can traverse  
        - Type 3: Both Alice and Bob can traverse
      
        Args:
            n: Number of nodes in the graph (1-indexed)
            edges: List of [type, u, v] representing edges
          
        Returns:
            Maximum number of removable edges, or -1 if impossible to make
            graph fully traversable for both Alice and Bob
        """
        # Create separate Union-Find structures for Alice and Bob
        alice_uf = UnionFind(n)
        bob_uf = UnionFind(n)
        removable_edges = 0
      
        # Process Type 3 edges first (beneficial to both Alice and Bob)
        for edge_type, u, v in edges:
            if edge_type == 3:
                # If edge connects new components for Alice, also add for Bob
                if alice_uf.union(u, v):
                    bob_uf.union(u, v)
                else:
                    # Edge is redundant for both - can be removed
                    removable_edges += 1
      
        # Process Type 1 (Alice only) and Type 2 (Bob only) edges
        for edge_type, u, v in edges:
            if edge_type == 1:
                # Count redundant edges for Alice
                removable_edges += not alice_uf.union(u, v)
            elif edge_type == 2:
                # Count redundant edges for Bob
                removable_edges += not bob_uf.union(u, v)
      
        # Check if both graphs are fully connected (single component)
        if alice_uf.component_count == 1 and bob_uf.component_count == 1:
            return removable_edges
        else:
            return -1
```

## Time and Space Complexity

**Time Complexity:** `O(E * α(N))` where `E` is the number of edges and `N` is the number of nodes. The `α(N)` is the inverse Ackermann function which grows extremely slowly and is practically constant (≤ 4) for all reasonable values of `N`.

- The code iterates through all edges twice: once for type 3 edges and once for type 1 and 2 edges, giving us `O(E)` iterations total.
- Each iteration performs union operations which involve find operations. With path compression in the find method, each find operation has an amortized time complexity of `O(α(N))`.
- Therefore, the overall time complexity is `O(E * α(N))`, which is nearly `O(E)` in practice.

**Space Complexity:** `O(N)`

- Two UnionFind data structures are created (`ufa` and `ufb`), each requiring `O(N)` space:
  - `self.p`: array of size `N` storing parent pointers
  - `self.size`: array of size `N` storing component sizes
  - `self.cnt`: single integer
- The total space used is `2 * O(N) = O(N)`
- The recursion stack for the find operation with path compression can go up to `O(N)` in the worst case before path compression takes effect, but after compression, subsequent calls will be `O(1)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Processing Type 3 Edges Incorrectly**

**Pitfall:** A common mistake is to independently check if a Type 3 edge is useful for Alice and Bob separately, which can lead to incorrectly counting removable edges.

**Incorrect approach:**
```python
# WRONG: Independent checking
for edge_type, u, v in edges:
    if edge_type == 3:
        alice_useful = alice_uf.union(u, v)
        bob_useful = bob_uf.union(u, v)
        if not alice_useful and not bob_useful:
            removable_edges += 1
```

**Why it's wrong:** This approach processes the same edge twice independently. If the edge connects components in Alice's graph but not Bob's (or vice versa), we'd still keep it, but we've already modified both union-find structures.

**Correct approach:**
```python
# CORRECT: Coordinated checking
for edge_type, u, v in edges:
    if edge_type == 3:
        if alice_uf.union(u, v):
            bob_uf.union(u, v)  # Only union in Bob's if useful for Alice
        else:
            removable_edges += 1  # Redundant for Alice means redundant for both
```

### 2. **Not Processing Type 3 Edges First**

**Pitfall:** Processing edges in the order they appear or processing type-specific edges before shared edges.

**Why it matters:** Type 3 edges are the most valuable because they maintain connectivity for both Alice and Bob simultaneously. By processing them first, we maximize the chance of using shared edges instead of person-specific edges, potentially allowing more edges to be removed.

**Example scenario:**
- If nodes A and B can be connected by either a Type 1 edge or a Type 3 edge
- Processing Type 1 first would use it for Alice, then still need the Type 3 for Bob
- Processing Type 3 first satisfies both Alice and Bob, making the Type 1 edge removable

### 3. **Incorrect Index Handling**

**Pitfall:** Forgetting that the problem uses 1-indexed nodes while the Union-Find implementation typically uses 0-indexed arrays.

**Wrong:**
```python
def union(self, node_a: int, node_b: int) -> bool:
    # WRONG: Using nodes directly without conversion
    root_a = self.find(node_a)
    root_b = self.find(node_b)
```

**Correct:**
```python
def union(self, node_a: int, node_b: int) -> bool:
    # CORRECT: Convert from 1-indexed to 0-indexed
    root_a = self.find(node_a - 1)
    root_b = self.find(node_b - 1)
```

### 4. **Misunderstanding Edge Redundancy**

**Pitfall:** Thinking that an edge is removable just because there's another path between the nodes, without considering that removing it might disconnect other parts of the graph.

**Key insight:** The Union-Find approach correctly identifies redundant edges - an edge is only redundant if it connects two nodes that are already in the same connected component. This ensures that removing the edge won't affect connectivity.

### 5. **Forgetting Final Validation**

**Pitfall:** Returning the count of removable edges without checking if both graphs are fully connected.

**Wrong:**
```python
# WRONG: Assuming graphs are connected
return removable_edges
```

**Correct:**
```python
# CORRECT: Verify both graphs have exactly 1 component
if alice_uf.component_count == 1 and bob_uf.component_count == 1:
    return removable_edges
else:
    return -1
```

This validation is crucial because the initial graph might not have enough edges to make it fully traversable for both Alice and Bob, even without removing any edges.

