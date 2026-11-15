# 2642. Design Graph With Shortest Path Calculator

> Source: https://algo.monster/liteproblems/2642

## Problem Statement

You need to implement a `[Graph](/problems/graph_intro)` class that represents a directed weighted graph with `n` nodes numbered from `0` to `n - 1`.

The class should support three operations:

1. **Constructor** `[Graph](/problems/graph_intro)(int n, int[][] edges)`: Initialize the graph with `n` nodes and a list of edges. Each edge is represented as `[from, to, edgeCost]`, meaning there's a directed edge from node `from` to node `to` with weight `edgeCost`.

2. **Add Edge** `addEdge(int[] edge)`: Add a new edge to the [graph](/problems/graph_intro). The edge is given as `[from, to, edgeCost]`. It's guaranteed that this edge doesn't already exist in the graph.

3. **Find [Shortest Path](/problems/shortest_path_unweight)** `shortestPath(int node1, int node2)`: Calculate and return the minimum cost to travel from `node1` to `node2`. The cost of a path is the sum of all edge weights along that path. If no path exists from `node1` to `node2`, return `-1`.

## Intuition

When we need to find the [shortest path](/problems/shortest_path_unweight) in a weighted [graph](/problems/graph_intro), we naturally think about classic shortest path algorithms. Since we're dealing with a single-source shortest path problem (finding the shortest path from one specific node to another), Dijkstra's algorithm is a natural choice.

The key insight is that we need a data structure that allows efficient edge additions and can quickly compute shortest paths when needed. An adjacency matrix `g[i][j]` is ideal here because:
- Adding an edge is simply updating one cell in the matrix: `g[from][to] = cost`
- We can easily iterate through all neighbors of a node by checking a row in the matrix

For the [shortest path](/problems/shortest_path_unweight) computation, we follow the greedy principle of Dijkstra's algorithm: at each step, we extend our shortest path tree by choosing the unvisited node that's closest to the source. This works because once we visit a node with the minimum distance, we know we've found the shortest path to it (assuming non-negative edge weights).

The algorithm maintains two key pieces of information:
- `dist[i]`: The shortest known distance from the source to node `i`
- `vis[i]`: Whether we've confirmed the [shortest path](/problems/shortest_path_unweight) to node `i`

We start by setting `dist[node1] = 0` (distance to source is 0) and all other distances to infinity. Then, we repeatedly:
1. Pick the unvisited node `t` with the smallest distance
2. Mark `t` as visited (we've found its [shortest path](/problems/shortest_path_unweight))
3. Update distances to all neighbors of `t` using the relaxation formula: `dist[j] = min(dist[j], dist[t] + g[t][j])`

This process continues for `n` iterations (once for each node), ensuring we explore all possible paths. If `dist[node2]` remains infinity after the algorithm completes, it means `node2` is unreachable from `node1`.

**Learn more about [Graph](/problems/graph_intro), [Shortest Path](/problems/shortest_path_unweight) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation uses an adjacency matrix representation with Dijkstra's algorithm for finding shortest paths.

**Initialization (`__init__`):**
- Create an `n × n` adjacency matrix `g` where all values are initialized to `inf` (infinity)
- For each edge `[f, t, c]` in the input edges list, set `g[f][t] = c` to represent a directed edge from node `f` to node `t` with cost `c`
- This matrix representation allows O(1) edge lookups and updates

**Adding Edges (`addEdge`):**
- Extract the edge components: `f, t, c = edge` (from, to, cost)
- Simply update the adjacency matrix: `g[f][t] = c`
- This operation takes O(1) time

**Finding [Shortest Path](/problems/shortest_path_unweight) (`shortestPath`):**

The implementation follows Dijkstra's algorithm with a linear search for the minimum distance node:

1. **Initialize distance array:** Create `dist` array of size `n` with all values set to `inf`, except `dist[node1] = 0` (source node has distance 0)

2. **Initialize visited array:** Create `vis` array of size `n` with all values set to `False`

3. **Main algorithm loop:** Repeat `n` times:
   - **Find minimum unvisited node:** 
     ```python
     t = -1
     for j in range(self.n):
         if not vis[j] and (t == -1 or dist[t] > dist[j]):
             t = j
     ```
     This finds the unvisited node `t` with the smallest distance value
 
   - **Mark as visited:** Set `vis[t] = True` to indicate we've found the [shortest path](/problems/shortest_path_unweight) to node `t`
 
   - **Relax edges:** Update distances to all neighbors of `t`:
     ```python
     for j in range(self.n):
         dist[j] = min(dist[j], dist[t] + self.g[t][j])
     ```
     If going through node `t` provides a shorter path to node `j`, update `dist[j]`

4. **Return result:** After processing all nodes, check if `dist[node2] == inf`. If so, return `-1` (no path exists). Otherwise, return `dist[node2]` (the [shortest path](/problems/shortest_path_unweight) distance)

**Time Complexity:**
- `__init__`: O(n² + E) where E is the number of initial edges
- `addEdge`: O(1)
- `shortestPath`: O(n²) due to the nested loops - finding minimum node takes O(n) and we do this n times

**Space Complexity:** O(n²) for storing the adjacency matrix

## Example Walkthrough

Let's walk through a small example to illustrate how the solution works.

**Example Setup:**
- Create a graph with 4 nodes (0, 1, 2, 3)
- Initial edges: [[0,1,2], [1,2,3], [0,3,6]]
- Find shortest path from node 0 to node 2
- Add edge [3,2,1]
- Find shortest path from node 0 to node 2 again

**Step 1: Initialize the Graph**
```
Graph(4, [[0,1,2], [1,2,3], [0,3,6]])
```
Creates adjacency matrix `g`:
```
     0   1   2   3
0 [ inf  2  inf  6 ]
1 [ inf inf  3  inf]
2 [ inf inf inf inf]
3 [ inf inf inf inf]
```

**Step 2: Find Shortest Path from 0 to 2**

Initialize:
- `dist = [0, inf, inf, inf]` (distance from node 0)
- `vis = [False, False, False, False]`

**Iteration 1:**
- Find minimum unvisited: node 0 (dist=0)
- Mark node 0 as visited: `vis = [True, False, False, False]`
- Relax edges from node 0:
  - `dist[1] = min(inf, 0+2) = 2`
  - `dist[3] = min(inf, 0+6) = 6`
- Updated `dist = [0, 2, inf, 6]`

**Iteration 2:**
- Find minimum unvisited: node 1 (dist=2)
- Mark node 1 as visited: `vis = [True, True, False, False]`
- Relax edges from node 1:
  - `dist[2] = min(inf, 2+3) = 5`
- Updated `dist = [0, 2, 5, 6]`

**Iteration 3:**
- Find minimum unvisited: node 2 (dist=5)
- Mark node 2 as visited: `vis = [True, True, True, False]`
- Relax edges from node 2: (no outgoing edges)
- `dist` remains `[0, 2, 5, 6]`

**Iteration 4:**
- Find minimum unvisited: node 3 (dist=6)
- Mark node 3 as visited: `vis = [True, True, True, True]`
- Relax edges from node 3: (no outgoing edges initially)
- Final `dist = [0, 2, 5, 6]`

**Result:** Shortest path from 0 to 2 = 5 (path: 0→1→2)

**Step 3: Add Edge [3,2,1]**
```
addEdge([3,2,1])
```
Updates adjacency matrix:
```
     0   1   2   3
0 [ inf  2  inf  6 ]
1 [ inf inf  3  inf]
2 [ inf inf inf inf]
3 [ inf inf  1  inf]  ← New edge added
```

**Step 4: Find Shortest Path from 0 to 2 Again**

Initialize:
- `dist = [0, inf, inf, inf]`
- `vis = [False, False, False, False]`

Following the same process:
- After processing node 0: `dist = [0, 2, inf, 6]`
- After processing node 1: `dist = [0, 2, 5, 6]`
- After processing node 2: `dist = [0, 2, 5, 6]`
- After processing node 3: `dist[2] = min(5, 6+1) = 5`

Wait, let's recalculate more carefully:

**Iteration 1:** Process node 0
- `dist = [0, 2, inf, 6]`

**Iteration 2:** Process node 1 (dist=2)
- `dist = [0, 2, 5, 6]`

**Iteration 3:** Process node 2 (dist=5)
- No change

**Iteration 4:** Process node 3 (dist=6)
- `dist[2] = min(5, 6+1) = 5` (no improvement)

Actually, we need to process in the correct order. Let me redo:

**Iteration 1:** Process node 0
- Relax: `dist = [0, 2, inf, 6]`

**Iteration 2:** Process node 1 (smallest unvisited with dist=2)
- Relax: `dist = [0, 2, 5, 6]`

**Iteration 3:** Process node 2 (smallest unvisited with dist=5)
- No outgoing edges, no change

**Iteration 4:** Process node 3 (smallest unvisited with dist=6)
- Relax: `dist[2] = min(5, 6+1) = 5` (no change, but would be 7 if we took path 0→3→2)

**Result:** Shortest path from 0 to 2 = 5 (still path: 0→1→2, the new edge didn't provide a shorter path)

This example demonstrates how the algorithm explores all possible paths and always finds the optimal solution, even when new edges are added to the graph.

## Python Solution

```python
from typing import List
from math import inf


class Graph:
    def __init__(self, n: int, edges: List[List[int]]):
        """
        Initialize the graph with n nodes and given edges.
      
        Args:
            n: Number of nodes in the graph (0 to n-1)
            edges: List of edges where each edge is [from_node, to_node, cost]
        """
        self.n = n
        # Initialize adjacency matrix with infinity (no direct connection)
        self.adjacency_matrix = [[inf] * n for _ in range(n)]
      
        # Populate the adjacency matrix with given edges
        for from_node, to_node, cost in edges:
            self.adjacency_matrix[from_node][to_node] = cost

    def addEdge(self, edge: List[int]) -> None:
        """
        Add a new edge to the graph or update existing edge cost.
      
        Args:
            edge: Edge to add as [from_node, to_node, cost]
        """
        from_node, to_node, cost = edge
        self.adjacency_matrix[from_node][to_node] = cost

    def shortestPath(self, node1: int, node2: int) -> int:
        """
        Find the shortest path from node1 to node2 using Dijkstra's algorithm.
      
        Args:
            node1: Starting node
            node2: Destination node
          
        Returns:
            Shortest distance from node1 to node2, or -1 if no path exists
        """
        # Initialize distances from source node to all nodes
        distances = [inf] * self.n
        distances[node1] = 0
      
        # Track visited nodes
        visited = [False] * self.n
      
        # Process all nodes
        for _ in range(self.n):
            # Find the unvisited node with minimum distance
            current_node = -1
            for node in range(self.n):
                if not visited[node] and (current_node == -1 or distances[current_node] > distances[node]):
                    current_node = node
          
            # Mark current node as visited
            visited[current_node] = True
          
            # Update distances to all neighbors of current node
            for neighbor in range(self.n):
                distances[neighbor] = min(distances[neighbor], 
                                         distances[current_node] + self.adjacency_matrix[current_node][neighbor])
      
        # Return the shortest distance to destination (or -1 if unreachable)
        return -1 if distances[node2] == inf else distances[node2]


# Your Graph object will be instantiated and called as such:
# obj = Graph(n, edges)
# obj.addEdge(edge)
# param_2 = obj.shortestPath(node1, node2)
```

## Time and Space Complexity

**Time Complexity:**
- `__init__`: `O(n^2 + m)` where `n` is the number of nodes and `m` is the number of initial edges. Creating the `n × n` matrix takes `O(n^2)` time, and filling in `m` edges takes `O(m)` time.
- `addEdge`: `O(1)` - simply updates one cell in the matrix.
- `shortestPath`: `O(n^2)` - implements Dijkstra's algorithm with a naive approach. The outer loop runs `n` times, and for each iteration, finding the minimum unvisited node takes `O(n)` time, and updating distances for all neighbors also takes `O(n)` time.

Overall, if there are `q` calls to `shortestPath`, the total time complexity for all shortest path queries is `O(n^2 × q)`.

**Space Complexity:**
- The adjacency matrix `self.g` requires `O(n^2)` space to store all possible edges.
- The `shortestPath` method uses `O(n)` additional space for the `dist` and `vis` arrays.
- Overall space complexity is `O(n^2)` as the matrix dominates the space usage.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Handling Self-Loops Properly**
The current implementation doesn't initialize the diagonal of the adjacency matrix (self-loops) to 0. This means `adjacency_matrix[i][i] = inf` for all nodes, which can cause issues when the shortest path from a node to itself is queried.

**Problem Example:**
```python
g = Graph(3, [[0,1,5], [1,2,3]])
print(g.shortestPath(0, 0))  # Should return 0, but might not work correctly
```

**Solution:**
Initialize diagonal elements to 0 in the constructor:
```python
def __init__(self, n: int, edges: List[List[int]]):
    self.n = n
    self.adjacency_matrix = [[inf] * n for _ in range(n)]
  
    # Set distance from each node to itself as 0
    for i in range(n):
        self.adjacency_matrix[i][i] = 0
  
    for from_node, to_node, cost in edges:
        self.adjacency_matrix[from_node][to_node] = cost
```

### 2. **Inefficient Node Selection in Dijkstra's Algorithm**
The current implementation uses O(n) linear search to find the minimum unvisited node in each iteration, resulting in O(n²) time complexity for the inner loop alone. For graphs with many shortest path queries, this becomes a bottleneck.

**Solution:**
Use a min-heap (priority queue) to optimize node selection:
```python
import heapq

def shortestPath(self, node1: int, node2: int) -> int:
    distances = [inf] * self.n
    distances[node1] = 0
  
    # Priority queue: (distance, node)
    pq = [(0, node1)]
  
    while pq:
        current_dist, current_node = heapq.heappop(pq)
      
        # Skip if we've already found a better path
        if current_dist > distances[current_node]:
            continue
          
        # Early termination if we reached the destination
        if current_node == node2:
            return current_dist
      
        # Update neighbors
        for neighbor in range(self.n):
            if self.adjacency_matrix[current_node][neighbor] != inf:
                new_dist = current_dist + self.adjacency_matrix[current_node][neighbor]
                if new_dist < distances[neighbor]:
                    distances[neighbor] = new_dist
                    heapq.heappush(pq, (new_dist, neighbor))
  
    return -1 if distances[node2] == inf else distances[node2]
```

### 3. **Memory Inefficiency for Sparse Graphs**
Using an n×n adjacency matrix consumes O(n²) space even for sparse graphs with few edges. This becomes problematic for large graphs with thousands of nodes but relatively few edges.

**Solution:**
Use an adjacency list representation for sparse graphs:
```python
def __init__(self, n: int, edges: List[List[int]]):
    self.n = n
    # Use adjacency list instead of matrix
    self.graph = {i: {} for i in range(n)}
  
    for from_node, to_node, cost in edges:
        self.graph[from_node][to_node] = cost

def addEdge(self, edge: List[int]) -> None:
    from_node, to_node, cost = edge
    self.graph[from_node][to_node] = cost
```

### 4. **Unnecessary Computation When Path Doesn't Exist**
The current implementation processes all n nodes even when it's clear early on that no path exists to the destination (when all remaining unvisited nodes have infinite distance).

**Solution:**
Add early termination when the selected node has infinite distance:
```python
for _ in range(self.n):
    current_node = -1
    for node in range(self.n):
        if not visited[node] and (current_node == -1 or distances[current_node] > distances[node]):
            current_node = node
  
    # Early termination if no reachable nodes remain
    if distances[current_node] == inf:
        break
  
    visited[current_node] = True
  
    # Rest of the algorithm...
```

