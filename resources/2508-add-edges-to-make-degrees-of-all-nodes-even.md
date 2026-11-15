# 2508. Add Edges to Make Degrees of All Nodes Even

> Source: https://algo.monster/liteproblems/2508

## Problem Statement

You are given an undirected [graph](/problems/graph_intro) with `n` nodes numbered from `1` to `n`. The graph is defined by a 2D array `edges` where `edges[i] = [ai, bi]` represents an edge between nodes `ai` and `bi`. The graph may be disconnected.

Your task is to determine if you can add **at most two additional edges** (possibly none) to make every node in the [graph](/problems/graph_intro) have an **even degree**. The degree of a node is the number of edges connected to it.

Rules for adding edges:
- No repeated edges allowed (no duplicate edges)
- No self-loops allowed (a node cannot connect to itself)

Return `true` if it's possible to make all nodes have even degrees by adding at most 2 edges, otherwise return `false`.

**Example 1:** Given `n = 5` and edges connecting nodes as shown, you can add one edge to make all nodes have even degrees, so return `true`.

**Example 2:** Given `n = 4` with edges `[[1,2],[3,4]]`, you can add two edges (like connecting 1-3 and 2-4) to make all nodes have even degrees, so return `true`.

**Example 3:** Given `n = 4` with edges `[[1,2],[1,3],[1,4]]`, node 1 has degree 3 (odd) and nodes 2, 3, 4 each have degree 1 (odd). It's impossible to make all nodes have even degrees by adding at most 2 edges, so return `false`.

## Intuition

The key observation is that adding an edge between two nodes increases both their degrees by 1. Since we want all nodes to have even degrees, we need to identify which nodes currently have odd degrees and figure out how to make them even.

Let's think about what happens when we add edges:
- Adding 1 edge affects exactly 2 nodes (increases each by 1)
- Adding 2 edges affects at most 4 nodes

This immediately tells us that if we have more than 4 nodes with odd degrees, it's impossible to fix them all with at most 2 edges.

Now let's analyze the different cases:

**Case 1: 0 nodes with odd degree**
All nodes already have even degrees, so we don't need to add any edges. Return `true`.

**Case 2: 2 nodes with odd degree**
We have nodes `a` and `b` with odd degrees. If we add an edge between them, both become even. But what if they're already connected? Then we need to find a "helper" node `c` that isn't connected to either `a` or `b`. We can add edges `a-c` and `b-c`, which makes all three nodes change parity (odd→even for `a` and `b`, and `c` stays even if it was even or becomes even if it was odd).

**Case 3: 4 nodes with odd degree**
We have nodes `a`, `b`, `c`, `d` with odd degrees. We need to pair them up somehow using 2 edges. There are three possible pairings:
- Connect `a-b` and `c-d`
- Connect `a-c` and `b-d`  
- Connect `a-d` and `b-c`

We check if any of these pairings is valid (i.e., the edges don't already exist).

**Case 4: Any other number of odd-degree nodes**
For 1 or 3 nodes with odd degree, it's impossible because each edge changes exactly 2 nodes' parity. For more than 4 nodes with odd degree, we can't fix all of them with just 2 edges.

The solution naturally follows from counting odd-degree nodes and handling each case appropriately.

**Learn more about [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The implementation follows our case analysis strategy:

**Step 1: Build the [graph](/problems/graph_intro) representation**
We use a `defaultdict(set)` to store the adjacency list. For each edge `[a, b]`, we add `b` to `g[a]` and `a` to `g[b]`. Using sets helps us quickly check if an edge exists between two nodes.

```python
g = defaultdict(set)
for a, b in edges:
    g[a].add(b)
    g[b].add(a)
```

**Step 2: Find all nodes with odd degrees**
We iterate through the [graph](/problems/graph_intro) and collect nodes whose degree (size of their adjacency set) is odd:

```python
vs = [i for i, v in g.items() if len(v) & 1]
```

The expression `len(v) & 1` checks if the degree is odd (bitwise AND with 1).

**Step 3: Handle different cases based on the count of odd-degree nodes**

**Case 0 nodes:** All nodes have even degrees already.
```python
if len(vs) == 0:
    return True
```

**Case 2 nodes:** Try to connect them directly, or find a common third node.
```python
if len(vs) == 2:
    a, b = vs
    if a not in g[b]:  # Can directly connect a and b
        return True
    # Try to find a node c that can connect to both a and b
    return any(a not in g[c] and c not in g[b] for c in range(1, n + 1))
```

**Case 4 nodes:** Try all three possible pairings.
```python
if len(vs) == 4:
    a, b, c, d = vs
    if a not in g[b] and c not in g[d]:  # Pair (a,b) and (c,d)
        return True
    if a not in g[c] and b not in g[d]:  # Pair (a,c) and (b,d)
        return True
    if a not in g[d] and b not in g[c]:  # Pair (a,d) and (b,c)
        return True
    return False
```

**Any other case:** Return `false` as it's impossible to make all degrees even with at most 2 edges.
```python
return False
```

The time complexity is `O(E + N)` where `E` is the number of edges (for building the [graph](/problems/graph_intro)) and `N` is the number of nodes (for checking potential connections in the worst case). The space complexity is `O(E)` for storing the graph.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Example:** `n = 6`, `edges = [[1,2], [2,3], [3,4], [4,5]]`

**Step 1: Build the graph**
```
Node 1: connected to {2}      → degree = 1 (odd)
Node 2: connected to {1, 3}   → degree = 2 (even)
Node 3: connected to {2, 4}   → degree = 2 (even)
Node 4: connected to {3, 5}   → degree = 2 (even)
Node 5: connected to {4}      → degree = 1 (odd)
Node 6: connected to {}       → degree = 0 (even)
```

**Step 2: Find odd-degree nodes**
Nodes with odd degrees: `[1, 5]`
Count = 2

**Step 3: Apply Case 2 logic**
Since we have exactly 2 nodes with odd degrees, we check:
- Can we directly connect nodes 1 and 5?
  - Check if edge (1,5) already exists: No, it doesn't exist
  - We can add edge (1,5)!

After adding edge (1,5):
- Node 1: degree becomes 2 (even) ✓
- Node 5: degree becomes 2 (even) ✓
- All other nodes remain even ✓

**Result:** Return `true` - we made all degrees even by adding just 1 edge.

---

**Another Example:** `n = 4`, `edges = [[1,2], [2,3], [3,1]]` (a triangle)

**Step 1: Build the graph**
```
Node 1: connected to {2, 3}   → degree = 2 (even)
Node 2: connected to {1, 3}   → degree = 2 (even)
Node 3: connected to {1, 2}   → degree = 2 (even)
Node 4: connected to {}       → degree = 0 (even)
```

**Step 2: Find odd-degree nodes**
Nodes with odd degrees: `[]`
Count = 0

**Step 3: Apply Case 0 logic**
All nodes already have even degrees!

**Result:** Return `true` - no edges need to be added.

---

**Impossible Example:** `n = 4`, `edges = [[1,2], [1,3], [1,4]]` (star graph)

**Step 1: Build the graph**
```
Node 1: connected to {2, 3, 4} → degree = 3 (odd)
Node 2: connected to {1}       → degree = 1 (odd)
Node 3: connected to {1}       → degree = 1 (odd)
Node 4: connected to {1}       → degree = 1 (odd)
```

**Step 2: Find odd-degree nodes**
Nodes with odd degrees: `[1, 2, 3, 4]`
Count = 4

**Step 3: Apply Case 4 logic**
We need to try all possible pairings:
1. Pair (1,2) and (3,4):
   - Edge (1,2) already exists ✗
   - Cannot use this pairing
 
2. Pair (1,3) and (2,4):
   - Edge (1,3) already exists ✗
   - Cannot use this pairing
 
3. Pair (1,4) and (2,3):
   - Edge (1,4) already exists ✗
   - Cannot use this pairing

All possible pairings fail because node 1 is already connected to all other nodes.

**Result:** Return `false` - impossible to make all degrees even with at most 2 edges.

## Python Solution

```python
class Solution:
    def isPossible(self, n: int, edges: List[List[int]]) -> bool:
        # Build adjacency list representation of the graph
        adjacency_list = defaultdict(set)
        for node_a, node_b in edges:
            adjacency_list[node_a].add(node_b)
            adjacency_list[node_b].add(node_a)
      
        # Find all vertices with odd degree
        odd_degree_vertices = [vertex for vertex, neighbors in adjacency_list.items() 
                               if len(neighbors) % 2 == 1]
      
        # If all vertices have even degree, graph is already Eulerian
        if len(odd_degree_vertices) == 0:
            return True
      
        # If exactly 2 vertices have odd degree
        if len(odd_degree_vertices) == 2:
            vertex_a, vertex_b = odd_degree_vertices
          
            # Check if we can directly connect these two vertices
            if vertex_a not in adjacency_list[vertex_b]:
                return True
          
            # Try to find an intermediate vertex to connect both odd-degree vertices
            for intermediate_vertex in range(1, n + 1):
                if (vertex_a not in adjacency_list[intermediate_vertex] and 
                    intermediate_vertex not in adjacency_list[vertex_b]):
                    return True
            return False
      
        # If exactly 4 vertices have odd degree
        if len(odd_degree_vertices) == 4:
            vertex_a, vertex_b, vertex_c, vertex_d = odd_degree_vertices
          
            # Try all three possible pairings to connect the 4 vertices
            # Pairing 1: (a,b) and (c,d)
            if (vertex_a not in adjacency_list[vertex_b] and 
                vertex_c not in adjacency_list[vertex_d]):
                return True
          
            # Pairing 2: (a,c) and (b,d)
            if (vertex_a not in adjacency_list[vertex_c] and 
                vertex_b not in adjacency_list[vertex_d]):
                return True
          
            # Pairing 3: (a,d) and (b,c)
            if (vertex_a not in adjacency_list[vertex_d] and 
                vertex_b not in adjacency_list[vertex_c]):
                return True
          
            return False
      
        # More than 4 vertices with odd degree cannot be fixed with at most 2 edges
        return False
```

## Time and Space Complexity

**Time Complexity: `O(n + m)`**

The algorithm consists of the following operations:
- Building the adjacency list by iterating through all edges: `O(m)` where `m` is the number of edges
- Finding vertices with odd degree by iterating through the graph dictionary: `O(n)` where `n` is the number of nodes (at most `n` vertices in the graph)
- When there are 2 vertices with odd degree, checking if they're connected takes `O(1)`, and the `any()` function iterates through all nodes from 1 to `n+1`, checking set membership operations which are `O(1)` each, resulting in `O(n)`
- When there are 4 vertices with odd degree, we perform constant number of set membership checks: `O(1)`

The dominant operations are building the graph `O(m)` and the potential iteration through all nodes `O(n)`, giving us a total time complexity of `O(n + m)`.

**Space Complexity: `O(n + m)`**

The space usage includes:
- The adjacency list `g` which stores all edges as sets: each edge `(a, b)` is stored twice (once for each direction), requiring `O(m)` space for all edges
- Each vertex can appear in the graph, requiring `O(n)` space for the dictionary keys
- The list `vs` stores at most 4 vertices with odd degree: `O(1)`

The adjacency list dominates the space usage with `O(n + m)` total space complexity.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly handling isolated nodes**
A critical pitfall is not accounting for nodes that have no edges at all (isolated nodes). These nodes have degree 0 (even) and won't appear in the adjacency list if we only build it from the edges.

**Problem Example:**
```python
n = 5, edges = [[1,2], [3,4]]
# Node 5 is isolated with degree 0
# Our current code would miss node 5 entirely
```

In the case of 2 odd-degree nodes, when searching for an intermediate vertex, the code iterates from `1` to `n+1`, but if a node doesn't exist in the adjacency list (like node 5), checking `vertex_a not in adjacency_list[intermediate_vertex]` would always return `True` since `adjacency_list[5]` would be an empty set.

**Solution:**
Ensure isolated nodes are properly handled by either:
- Pre-populating the adjacency list with all nodes
- Being careful when checking for non-existent edges

```python
# Initialize all nodes in the graph first
adjacency_list = {i: set() for i in range(1, n + 1)}
for node_a, node_b in edges:
    adjacency_list[node_a].add(node_b)
    adjacency_list[node_b].add(node_a)
```

### 2. **Self-loop confusion when checking intermediate vertices**
When looking for an intermediate vertex in the 2-node case, there's a subtle issue if we're not careful about the condition check.

**Problem Example:**
```python
# Current check:
if (vertex_a not in adjacency_list[intermediate_vertex] and 
    intermediate_vertex not in adjacency_list[vertex_b]):
```

This check has an asymmetry - it checks if `vertex_a` is not in `intermediate_vertex`'s neighbors, but then checks if `intermediate_vertex` is not in `vertex_b`'s neighbors. While this works due to the undirected nature of the graph, it's confusing and error-prone.

**Solution:**
Use consistent checking pattern:
```python
if (vertex_a not in adjacency_list[intermediate_vertex] and 
    vertex_b not in adjacency_list[intermediate_vertex]):
```

### 3. **Not considering the intermediate vertex could be one of the odd-degree vertices**
When we have 2 odd-degree vertices and need to find an intermediate vertex, we might accidentally allow one of the odd-degree vertices to be the intermediate vertex, which would create a self-loop.

**Problem Example:**
If `vertex_a = 1` and `vertex_b = 2`, and we check `intermediate_vertex = 1`, we'd be trying to add edges (1,1) and (1,2), where (1,1) is a self-loop.

**Solution:**
Explicitly exclude the odd-degree vertices from being intermediate vertices:
```python
for intermediate_vertex in range(1, n + 1):
    if intermediate_vertex in {vertex_a, vertex_b}:
        continue  # Skip if it's one of the odd-degree vertices
    if (vertex_a not in adjacency_list[intermediate_vertex] and 
        vertex_b not in adjacency_list[intermediate_vertex]):
        return True
```

