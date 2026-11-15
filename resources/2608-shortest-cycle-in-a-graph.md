# 2608. Shortest Cycle in a Graph

> Source: https://algo.monster/liteproblems/2608

## Problem Statement

You're given a bi-directional [graph](/problems/graph_intro) with `n` vertices labeled from `0` to `n - 1`. The graph is represented by a 2D integer array `edges`, where each `edges[i] = [ui, vi]` represents an undirected edge between vertex `ui` and vertex `vi`. 

The [graph](/problems/graph_intro) has the following properties:
- Each pair of vertices is connected by at most one edge (no multiple edges between the same pair)
- No vertex has an edge to itself (no self-loops)

Your task is to find the length of the shortest cycle in the [graph](/problems/graph_intro). A cycle is defined as a path that:
- Starts and ends at the same vertex
- Uses each edge in the path only once

If no cycle exists in the [graph](/problems/graph_intro), return `-1`.

For example, if you have edges `[[0,1], [1,2], [2,0]]`, this forms a triangle cycle of length 3. The path would be: 0 → 1 → 2 → 0, using 3 edges.

## Intuition

To find the shortest cycle in a [graph](/problems/graph_intro), we need to think about what a cycle actually is. A cycle means we can start from a vertex, travel through some edges, and return to the starting point. The key insight is that if we have an edge `(u, v)`, and there exists another path from `u` to `v` that doesn't use this edge, then we've found a cycle!

Think of it this way: imagine you're standing at vertex `u` and you can directly walk to vertex `v` using the edge between them. Now, if you can find an alternative route from `u` to `v` without using that direct edge, you've discovered a cycle. The cycle would be: take the alternative path from `u` to `v`, then use the direct edge from `v` back to `u`.

This leads us to a clever approach:
1. For each edge `(u, v)` in the [graph](/problems/graph_intro), temporarily "remove" or ignore this edge
2. Try to find the shortest path from `u` to `v` without using this edge
3. If such a path exists, the cycle length = length of this alternative path + 1 (for the original edge)

Why does this work? Because we're essentially finding the shortest "detour" around each edge. The shortest cycle containing edge `(u, v)` must consist of that edge plus the shortest path from `u` to `v` that doesn't use that edge.

Since we want the shortest cycle in the entire [graph](/problems/graph_intro), we check all edges and take the minimum cycle length found. We use BFS for finding the shortest path because the graph is unweighted (each edge has length 1), and BFS naturally finds the shortest path in unweighted graphs by exploring vertices level by level.

If no alternative path exists for any edge (meaning the [graph](/problems/graph_intro) is a tree or forest), then there are no cycles, and we return `-1`.

**Learn more about [Breadth-First Search](/problems/bfs_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The implementation follows the edge enumeration + BFS strategy outlined in the reference approach:

**1. Build the adjacency list:**
First, we construct an adjacency list `g` using a `defaultdict(set)` where `g[u]` contains all vertices directly connected to vertex `u`. Since the [graph](/problems/graph_intro) is bi-directional, for each edge `[u, v]`, we add `v` to `g[u]` and `u` to `g[v]`.

```python
g = defaultdict(set)
for u, v in edges:
    g[u].add(v)
    g[v].add(u)
```

**2. BFS function to find shortest path:**
The `bfs(u, v)` function finds the shortest path from vertex `u` to vertex `v` while excluding the direct edge between them:

- Initialize a distance array `dist` with infinity for all vertices, except `dist[u] = 0`
- Use a queue starting with vertex `u`
- For each vertex `i` popped from the queue, explore its neighbors `j`
- Skip the edge if it's the one we're excluding: `(i, j) != (u, v) and (j, i) != (u, v)`
- If vertex `j` hasn't been visited (`dist[j] == inf`), update its distance and add to queue
- Return `dist[v] + 1` which represents the cycle length (alternative path + the original edge)

```python
def bfs(u: int, v: int) -> int:
    dist = [inf] * n
    dist[u] = 0
    q = deque([u])
    while q:
        i = q.popleft()
        for j in g[i]:
            if (i, j) != (u, v) and (j, i) != (u, v) and dist[j] == inf:
                dist[j] = dist[i] + 1
                q.append(j)
    return dist[v] + 1
```

**3. Find the minimum cycle:**
We enumerate all edges and compute the cycle length for each:
```python
ans = min(bfs(u, v) for u, v in edges)
```

If no cycle exists (all BFS calls return infinity), we return `-1`. Otherwise, we return the minimum cycle length found.

The time complexity is `O(E × (V + E))` where `E` is the number of edges and `V` is the number of vertices, as we run BFS for each edge. The space complexity is `O(V + E)` for storing the [graph](/problems/graph_intro) and the BFS queue.

## Example Walkthrough

Let's walk through a small example with the graph having edges: `[[0,1], [1,2], [2,3], [3,0], [0,2]]`

This creates a graph that looks like:
```
    0 --- 1
    |\    |
    | \   |
    |  \  |
    3 --- 2
```

**Step 1: Build the adjacency list**
```
g[0] = {1, 2, 3}
g[1] = {0, 2}
g[2] = {0, 1, 3}
g[3] = {0, 2}
```

**Step 2: Check each edge for cycles**

**Edge (0,1):**
- Remove edge (0,1) temporarily
- Run BFS from vertex 0 to vertex 1 without using this edge
- Path found: 0 → 2 → 1 (length = 2)
- Cycle length = 2 + 1 = 3
- The cycle is: 0 → 1 → 2 → 0

**Edge (1,2):**
- Remove edge (1,2) temporarily  
- Run BFS from vertex 1 to vertex 2 without using this edge
- Path found: 1 → 0 → 2 (length = 2)
- Cycle length = 2 + 1 = 3
- The cycle is: 1 → 2 → 0 → 1

**Edge (2,3):**
- Remove edge (2,3) temporarily
- Run BFS from vertex 2 to vertex 3 without using this edge
- Path found: 2 → 0 → 3 (length = 2)
- Cycle length = 2 + 1 = 3
- The cycle is: 2 → 3 → 0 → 2

**Edge (3,0):**
- Remove edge (3,0) temporarily
- Run BFS from vertex 3 to vertex 0 without using this edge
- Path found: 3 → 2 → 0 (length = 2)
- Cycle length = 2 + 1 = 3
- The cycle is: 3 → 0 → 2 → 3

**Edge (0,2):**
- Remove edge (0,2) temporarily
- Run BFS from vertex 0 to vertex 2 without using this edge
- Path found: 0 → 1 → 2 (length = 2)
- Cycle length = 2 + 1 = 3
- The cycle is: 0 → 2 → 1 → 0

**Step 3: Find minimum**
All edges give us cycle length of 3. The minimum cycle length is **3**.

Note that there's also a 4-cycle (0 → 1 → 2 → 3 → 0), but our algorithm correctly identifies the shortest cycle of length 3.

## Python Solution

```python
from collections import defaultdict, deque
from typing import List

class Solution:
    def findShortestCycle(self, n: int, edges: List[List[int]]) -> int:
        """
        Find the length of the shortest cycle in an undirected graph.
      
        Args:
            n: Number of nodes in the graph (0 to n-1)
            edges: List of edges where each edge is [u, v]
      
        Returns:
            Length of shortest cycle, or -1 if no cycle exists
        """
      
        def bfs_shortest_path(start_node: int, end_node: int) -> int:
            """
            Find shortest path from start_node to end_node excluding the direct edge.
          
            Args:
                start_node: Starting node for BFS
                end_node: Target node to reach
          
            Returns:
                Distance from start to end plus 1 (to form cycle), or inf if unreachable
            """
            # Initialize distances to infinity
            distances = [float('inf')] * n
            distances[start_node] = 0
          
            # BFS queue starting from start_node
            queue = deque([start_node])
          
            while queue:
                current_node = queue.popleft()
              
                # Explore all neighbors of current node
                for neighbor in adjacency_list[current_node]:
                    # Skip the direct edge between start_node and end_node
                    # This ensures we find an alternate path to form a cycle
                    if ((current_node, neighbor) == (start_node, end_node) or 
                        (current_node, neighbor) == (end_node, start_node)):
                        continue
                  
                    # If neighbor hasn't been visited yet
                    if distances[neighbor] == float('inf'):
                        distances[neighbor] = distances[current_node] + 1
                        queue.append(neighbor)
          
            # Return cycle length (path length + 1 for the direct edge)
            return distances[end_node] + 1
      
        # Build adjacency list representation of the graph
        adjacency_list = defaultdict(set)
        for u, v in edges:
            adjacency_list[u].add(v)
            adjacency_list[v].add(u)
      
        # Try each edge as potential part of the shortest cycle
        # For each edge, find shortest path between its endpoints without using that edge
        min_cycle_length = min(bfs_shortest_path(u, v) for u, v in edges)
      
        # Return result: cycle length if found, otherwise -1
        return min_cycle_length if min_cycle_length < float('inf') else -1
```

## Time and Space Complexity

**Time Complexity:** `O(m²)` where `m` is the number of edges.

The algorithm iterates through each edge in the graph (total of `m` edges) and performs a BFS for each edge. Each BFS operation visits at most `n` vertices and traverses at most `m` edges in the worst case, taking `O(n + m)` time. Since we're dealing with a graph where cycles exist, and in the worst case of a dense graph, `m` can be up to `O(n²)`. However, for each edge `(u, v)`, the BFS explores the graph while excluding that specific edge, effectively searching for the shortest alternative path between `u` and `v`. This gives us a total time complexity of `O(m × (n + m))`. In the worst case where the graph is dense (`m ≈ n²`), this becomes `O(m²)`.

**Space Complexity:** `O(m + n)` where `n` is the number of vertices and `m` is the number of edges.

The space is used for:
- The adjacency list `g` which stores all edges, requiring `O(m)` space (each edge is stored twice in the undirected graph representation)
- The `dist` array in BFS which uses `O(n)` space for tracking distances to all vertices
- The queue `q` in BFS which can contain at most `O(n)` vertices
- The overall space complexity is therefore `O(m + n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly handling the edge exclusion in BFS**

One of the most common mistakes is failing to properly exclude the direct edge between the two endpoints when searching for an alternative path. The edge exclusion check must be bidirectional since the graph is undirected.

**Incorrect Implementation:**
```python
# Wrong: Only checking one direction
if (current_node, neighbor) == (start_node, end_node):
    continue
```

**Correct Implementation:**
```python
# Correct: Checking both directions of the edge
if ((current_node, neighbor) == (start_node, end_node) or 
    (current_node, neighbor) == (end_node, start_node)):
    continue
```

### 2. **Using a list instead of a set for adjacency list values**

Using a list for storing neighbors can lead to duplicate entries if the input contains duplicate edges, which would cause incorrect BFS traversal and potentially infinite loops.

**Problematic Code:**
```python
# Using list can cause duplicates
adjacency_list = defaultdict(list)
for u, v in edges:
    adjacency_list[u].append(v)
    adjacency_list[v].append(u)
```

**Better Solution:**
```python
# Using set automatically handles duplicates
adjacency_list = defaultdict(set)
for u, v in edges:
    adjacency_list[u].add(v)
    adjacency_list[v].add(u)
```

### 3. **Forgetting to add 1 to the BFS distance**

The BFS finds the shortest alternative path between two nodes, but the cycle length includes the original edge that was excluded. Forgetting to add 1 will give an incorrect cycle length.

**Wrong:**
```python
return distances[end_node]  # Missing the +1
```

**Correct:**
```python
return distances[end_node] + 1  # Includes the direct edge
```

### 4. **Not handling disconnected components properly**

If the graph has disconnected components or isolated nodes, the solution handles it correctly by returning `inf` when no path exists. However, a common mistake is to not check if the minimum value is still infinity before returning the result.

**Potential Issue:**
```python
# If all edges are in disconnected components with no cycles
min_cycle_length = min(bfs_shortest_path(u, v) for u, v in edges)
return min_cycle_length  # Could return inf instead of -1
```

**Proper Handling:**
```python
min_cycle_length = min(bfs_shortest_path(u, v) for u, v in edges)
return min_cycle_length if min_cycle_length < float('inf') else -1
```

### 5. **Performance degradation with dense graphs**

While the algorithm is correct, for very dense graphs (where E approaches V²), running BFS for each edge becomes expensive. Consider early termination optimizations:

**Optimization Example:**
```python
def bfs_shortest_path(start_node: int, end_node: int, current_min: int) -> int:
    # ... existing code ...
    while queue:
        current_node = queue.popleft()
      
        # Early termination if current path is already longer than known minimum
        if distances[current_node] >= current_min:
            continue
          
        for neighbor in adjacency_list[current_node]:
            # ... rest of the logic
```

This optimization can significantly reduce computation time by pruning paths that cannot lead to a shorter cycle.

