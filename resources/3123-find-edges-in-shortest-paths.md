# 3123. Find Edges in Shortest Paths

> Source: https://algo.monster/liteproblems/3123

## Problem Statement

You have an undirected weighted [graph](/problems/graph_intro) with `n` nodes (numbered from 0 to `n - 1`) and `m` edges. The edges are given as a 2D array `edges`, where each `edges[i] = [ai, bi, wi]` represents an edge between nodes `ai` and `bi` with weight `wi`.

Your task is to find all edges that belong to at least one [shortest path](/problems/shortest_path_unweight) from node 0 to node `n - 1`. 

You need to return a boolean array `answer` of length `m` where:
- `answer[i] = true` if the i-th edge is part of at least one [shortest path](/problems/shortest_path_unweight) from node 0 to node `n - 1`
- `answer[i] = false` otherwise

Note that the [graph](/problems/graph_intro) may not be connected, meaning there might not be any path from node 0 to node `n - 1`.

For example, if there are multiple shortest paths from node 0 to node `n - 1`, and an edge appears in any of these paths, then that edge should be marked as `true` in the answer array. If an edge doesn't participate in any [shortest path](/problems/shortest_path_unweight) (either because it's not on the route or because using it would make the path longer), it should be marked as `false`.

## Intuition

To find which edges are part of shortest paths, we need to think about what makes an edge "special" in the context of shortest paths.

First, we need to know the actual shortest distances from the source (node 0) to all other nodes. This is a classic [shortest path](/problems/shortest_path_unweight) problem, and since we have weighted edges, Dijkstra's algorithm is the natural choice.

But knowing shortest distances alone isn't enough. We need to identify which specific edges contribute to these shortest paths. Here's the key insight: an edge `(a, b)` with weight `w` is part of a [shortest path](/problems/shortest_path_unweight) if and only if it connects two nodes whose shortest distances differ by exactly `w`. In other words, if `dist[a] + w = dist[b]` or `dist[b] + w = dist[a]`, then this edge is a "bridge" in at least one shortest path.

Think of it like this: imagine you're traveling from city 0 to city `n-1`, and you know the minimum travel time to reach every city. An road between cities A and B is part of your optimal route if taking that road from A gets you to B at exactly the minimum time needed to reach B (or vice versa).

The clever part of the solution is working backwards from the destination. Once we have all shortest distances, we start from node `n-1` and trace back through the [graph](/problems/graph_intro). For each node we visit, we check all its edges. If an edge satisfies our [shortest path](/problems/shortest_path_unweight) condition (`dist[current] = dist[neighbor] + edge_weight`), we know this edge is part of a shortest path, so we mark it as `true` and continue exploring from the neighbor.

This backward traversal ensures we only mark edges that are actually reachable on a [shortest path](/problems/shortest_path_unweight) from source to destination. If there's no path at all (distance to `n-1` is infinity), we simply return all `false` values.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro), [Graph](/problems/graph_intro), [Shortest Path](/problems/shortest_path_unweight) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation consists of two main phases: finding shortest distances using Dijkstra's algorithm, and then identifying edges on shortest paths through backward traversal.

**Phase 1: Building the [Graph](/problems/graph_intro) and Finding Shortest Distances**

First, we create an adjacency list representation of the [graph](/problems/graph_intro). For each edge `(a, b, w)`, we store it in both directions since the graph is undirected. We also store the edge index `i` with each edge to track which original edge it corresponds to:

```python
g = defaultdict(list)
for i, (a, b, w) in enumerate(edges):
    g[a].append((b, w, i))  # neighbor, weight, edge_index
    g[b].append((a, w, i))
```

Next, we apply Dijkstra's algorithm using a min-heap to find shortest distances from node 0 to all other nodes:

```python
dist = [inf] * n
dist[0] = 0
q = [(0, 0)]  # (distance, node)
```

The algorithm proceeds by:
1. Extracting the node with minimum distance from the heap
2. Skipping if we've already processed a better path to this node
3. Relaxing edges: for each neighbor, if we find a shorter path through the current node, we update its distance and add it to the heap

```python
while q:
    da, a = heappop(q)
    if da > dist[a]:
        continue
    for b, w, _ in g[a]:
        if dist[b] > dist[a] + w:
            dist[b] = dist[a] + w
            heappush(q, (dist[b], b))
```

**Phase 2: Identifying Edges on Shortest Paths**

After computing shortest distances, we check if node `n-1` is reachable. If `dist[n-1] == inf`, there's no path, so all edges return `false`.

For the backward traversal, we use a queue (BFS approach) starting from the destination node `n-1`:

```python
ans = [False] * m
q = deque([n - 1])
while q:
    a = q.popleft()
    for b, w, i in g[a]:
        if dist[a] == dist[b] + w:
            ans[i] = True
            q.append(b)
```

The key condition `dist[a] == dist[b] + w` checks if edge `(b, a)` with weight `w` is part of a [shortest path](/problems/shortest_path_unweight). If true:
- We mark this edge as part of a shortest path (`ans[i] = True`)
- We continue exploring from node `b` to find more edges on shortest paths

This backward traversal ensures we only mark edges that actually connect nodes on valid shortest paths from source to destination. The BFS approach prevents infinite loops and ensures each node is processed in the context of shortest paths.

The time complexity is `O(E log V)` for Dijkstra's algorithm plus `O(E)` for the backward traversal, where `E` is the number of edges and `V` is the number of vertices.

## Example Walkthrough

Let's work through a small example to illustrate the solution approach.

**Example Graph:**
```
n = 4, edges = [[0,1,1], [1,2,1], [0,3,4], [3,2,1], [1,3,1]]
```

This creates the following graph:
```
    1       1
0 ----> 1 ----> 2
 \      |      /
  \4    |1    /1
   \    v    /
    --> 3 <--
```

**Phase 1: Finding Shortest Distances with Dijkstra's Algorithm**

Starting from node 0, we initialize:
- `dist = [0, inf, inf, inf]`
- Priority queue: `[(0, 0)]`

Step-by-step execution:

1. Pop `(0, 0)` from queue
   - Check neighbors of node 0:
     - Node 1: dist[1] = min(inf, 0+1) = 1, add (1,1) to queue
     - Node 3: dist[3] = min(inf, 0+4) = 4, add (4,3) to queue
   - `dist = [0, 1, inf, 4]`

2. Pop `(1, 1)` from queue
   - Check neighbors of node 1:
     - Node 0: dist[0] = 0 (no update needed)
     - Node 2: dist[2] = min(inf, 1+1) = 2, add (2,2) to queue
     - Node 3: dist[3] = min(4, 1+1) = 2, add (2,3) to queue
   - `dist = [0, 1, 2, 2]`

3. Pop `(2, 2)` from queue (node 2)
   - Check neighbors of node 2:
     - Node 1: dist[1] = 1 (no update needed)
     - Node 3: dist[3] = min(2, 2+1) = 2 (no update needed)

4. Pop `(2, 3)` from queue (node 3)
   - Check neighbors of node 3:
     - Node 0: dist[0] = 0 (no update needed)
     - Node 2: dist[2] = min(2, 2+1) = 2 (no update needed)
     - Node 1: dist[1] = min(1, 2+1) = 1 (no update needed)

Final distances: `dist = [0, 1, 2, 2]`

**Phase 2: Backward Traversal to Find Edges on Shortest Paths**

We want to reach node 3 (n-1). Since dist[3] = 2 (not infinity), there is a path.

Starting from node 3, we check which edges lead to it on a shortest path:

1. Process node 3 (dist[3] = 2):
   - Edge [0,3,4]: Is dist[3] == dist[0] + 4? Is 2 == 0 + 4? No
   - Edge [1,3,1]: Is dist[3] == dist[1] + 1? Is 2 == 1 + 1? **Yes!**
     - Mark edge index 4 as true
     - Add node 1 to queue

2. Process node 1 (dist[1] = 1):
   - Edge [0,1,1]: Is dist[1] == dist[0] + 1? Is 1 == 0 + 1? **Yes!**
     - Mark edge index 0 as true
     - Add node 0 to queue
   - Edge [1,2,1]: Is dist[1] == dist[2] + 1? Is 1 == 2 + 1? No
   - Edge [1,3,1]: Already processed from the other direction

3. Process node 0 (dist[0] = 0):
   - Node 0 is the source, no incoming edges on shortest path

**Result:**
- Edge 0 [0,1,1]: **true** (part of path 0→1→3)
- Edge 1 [1,2,1]: **false** (not needed to reach node 3)
- Edge 2 [0,3,4]: **false** (creates a longer path)
- Edge 3 [3,2,1]: **false** (not on any shortest path to node 3)
- Edge 4 [1,3,1]: **true** (part of path 0→1→3)

Final answer: `[true, false, false, false, true]`

The shortest paths from node 0 to node 3 are:
- 0 → 1 → 3 (total weight: 2)

This path uses edges [0,1,1] and [1,3,1], which are correctly identified by our algorithm.

## Python Solution

```python
from collections import defaultdict, deque
from heapq import heappush, heappop
from typing import List
from math import inf

class Solution:
    def findAnswer(self, n: int, edges: List[List[int]]) -> List[bool]:
        # Build adjacency list representation of the graph
        # Each node maps to list of (neighbor, weight, edge_index) tuples
        graph = defaultdict(list)
        for edge_idx, (node_a, node_b, weight) in enumerate(edges):
            graph[node_a].append((node_b, weight, edge_idx))
            graph[node_b].append((node_a, weight, edge_idx))
      
        # Initialize distances array for Dijkstra's algorithm
        distances = [inf] * n
        distances[0] = 0
      
        # Priority queue for Dijkstra's algorithm: (distance, node)
        priority_queue = [(0, 0)]
      
        # Run Dijkstra's algorithm to find shortest distances from node 0
        while priority_queue:
            current_dist, current_node = heappop(priority_queue)
          
            # Skip if we've already found a better path to this node
            if current_dist > distances[current_node]:
                continue
          
            # Explore neighbors
            for neighbor, edge_weight, _ in graph[current_node]:
                new_dist = distances[current_node] + edge_weight
                if distances[neighbor] > new_dist:
                    distances[neighbor] = new_dist
                    heappush(priority_queue, (distances[neighbor], neighbor))
      
        # Initialize result array for all edges
        num_edges = len(edges)
        result = [False] * num_edges
      
        # If destination is unreachable, return all False
        if distances[n - 1] == inf:
            return result
      
        # BFS from destination to mark edges that are part of shortest paths
        bfs_queue = deque([n - 1])
        visited = set([n - 1])
      
        while bfs_queue:
            current_node = bfs_queue.popleft()
          
            # Check all edges connected to current node
            for neighbor, edge_weight, edge_idx in graph[current_node]:
                # If this edge is part of a shortest path
                if distances[current_node] == distances[neighbor] + edge_weight:
                    result[edge_idx] = True
                    # Add neighbor to queue if not visited
                    if neighbor not in visited:
                        visited.add(neighbor)
                        bfs_queue.append(neighbor)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(m × log m)`

The algorithm consists of two main phases:

1. **Dijkstra's algorithm phase:** The algorithm uses a min-heap to find shortest distances from node 0 to all other nodes. In the worst case, each edge is processed once, and for each edge, we perform a heap push operation. Since there are `m` edges and each heap operation takes `O(log m)` time (the heap can contain at most `m` elements), this phase takes `O(m × log m)` time.

2. **BFS phase:** Starting from node `n-1`, the algorithm performs a BFS traversal to mark edges that lie on shortest paths. Each edge is visited at most once during this traversal, taking `O(m)` time.

The overall time complexity is dominated by the Dijkstra's algorithm phase: `O(m × log m) + O(m) = O(m × log m)`.

**Space Complexity:** `O(n + m)`

The space usage includes:
- Graph adjacency list `g`: stores all edges with their endpoints and weights, requiring `O(m)` space
- Distance array `dist`: stores distances for `n` nodes, requiring `O(n)` space  
- Priority queue `q` in Dijkstra's: can contain at most `O(m)` elements in the worst case
- BFS queue `q`: can contain at most `O(n)` nodes
- Answer array `ans`: stores boolean values for `m` edges, requiring `O(m)` space

The total space complexity is `O(n + m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

**Pitfall: Processing the same node multiple times during backward traversal**

The most critical issue in the provided code is that nodes can be added to the BFS queue multiple times during the backward traversal phase. When identifying edges on shortest paths, if a node has multiple incoming edges that are part of shortest paths, it will be added to the queue repeatedly. This leads to:

1. **Redundant processing**: The same node and its edges are checked multiple times
2. **Performance degradation**: In dense graphs or graphs with many shortest paths, this can significantly slow down the algorithm
3. **Potential infinite loops**: In certain graph structures, this could theoretically cause extremely long processing times

**Example scenario:**
Consider a diamond-shaped graph where node 0 connects to nodes 1 and 2, both of which connect to node 3. If both paths (0→1→3 and 0→2→3) are shortest paths, node 1 and node 2 might be added to the queue multiple times when traversing backward from node 3.

**Solution:**

Track visited nodes during the backward traversal to ensure each node is processed only once:

```python
# BFS from destination to mark edges that are part of shortest paths
bfs_queue = deque([n - 1])
visited_nodes = set([n - 1])  # Track visited nodes

while bfs_queue:
    current_node = bfs_queue.popleft()
  
    # Check all edges connected to current node
    for neighbor, edge_weight, edge_idx in graph[current_node]:
        # If this edge is part of a shortest path
        if distances[current_node] == distances[neighbor] + edge_weight:
            result[edge_idx] = True
            # Only add neighbor if not yet visited
            if neighbor not in visited_nodes:
                visited_nodes.add(neighbor)
                bfs_queue.append(neighbor)
```

This modification ensures:
- Each node is added to the queue at most once
- All edges on shortest paths are still correctly identified
- The time complexity remains O(E) for the backward traversal phase
- No redundant processing occurs

The key insight is that once we've processed a node during backward traversal, we've already identified all its relevant edges that participate in shortest paths, so there's no need to process it again.

