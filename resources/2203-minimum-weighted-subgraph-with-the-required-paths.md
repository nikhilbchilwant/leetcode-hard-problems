# 2203. Minimum Weighted Subgraph With the Required Paths

> Source: https://algo.monster/liteproblems/2203

## Problem Statement

You have a weighted directed [graph](/problems/graph_intro) with `n` nodes numbered from `0` to `n - 1`. The graph is defined by an array `edges` where each element `edges[i] = [from_i, to_i, weight_i]` represents a directed edge from node `from_i` to node `to_i` with weight `weight_i`.

You are given three distinct nodes: `src1`, `src2`, and `dest`.

Your task is to find the minimum total weight of a subgraph that allows both `src1` and `src2` to reach `dest`. A subgraph is formed by selecting some edges from the original [graph](/problems/graph_intro), and its weight is the sum of all selected edges' weights.

The key requirement is that there must exist paths:
- From `src1` to `dest` using only edges in the subgraph
- From `src2` to `dest` using only edges in the subgraph

Return the minimum weight of such a subgraph. If no such subgraph exists (meaning it's impossible for both source nodes to reach the destination), return `-1`.

**Example scenario:** Imagine you have two starting points and one destination. You want to find the cheapest way to build roads such that both starting points can reach the destination. The roads can be shared - if both paths use the same road, you only count its weight once in the total.

The solution uses Dijkstra's algorithm three times:
1. From `src1` to all nodes (forward [graph](/problems/graph_intro))
2. From `src2` to all nodes (forward graph)  
3. From `dest` to all nodes (reverse graph)

For each potential intermediate node `i`, it calculates the total weight as: `distance(src1 → i) + distance(src2 → i) + distance(i → dest)`. The minimum among all such sums is the answer.

## Intuition

The key insight is that when both `src1` and `src2` need to reach `dest`, their paths will likely merge at some intermediate node before continuing to the destination. Think of it like two rivers flowing toward the ocean - they might join at some point and then flow together.

Let's call this meeting point node `i`. The optimal subgraph would consist of:
- The [shortest path](/problems/shortest_path_unweight) from `src1` to `i`
- The shortest path from `src2` to `i`
- The shortest path from `i` to `dest`

These three path segments together form our subgraph, and their total weight is what we want to minimize.

But which node should be the meeting point? We don't know in advance, so we need to try all possible nodes as potential meeting points and find the one that gives us the minimum total weight.

For any candidate meeting point `i`, the total weight would be:
`distance(src1 → i) + distance(src2 → i) + distance(i → dest)`

To efficiently calculate these distances:
- We run Dijkstra from `src1` to get shortest distances from `src1` to all nodes
- We run Dijkstra from `src2` to get shortest distances from `src2` to all nodes
- For the third part, we need distances from each node to `dest`. Instead of running Dijkstra from every node to `dest`, we can be clever: we reverse all edges in the [graph](/problems/graph_intro) and run Dijkstra from `dest`. This gives us the shortest distances from all nodes to `dest` in the original graph.

After getting these three distance arrays (`d1`, `d2`, `d3`), for each node `i`, we calculate `d1[i] + d2[i] + d3[i]`. The minimum value among all nodes is our answer.

If the minimum value is infinity (meaning at least one of the required paths doesn't exist), we return `-1`.

**Learn more about [Graph](/problems/graph_intro) and [Shortest Path](/problems/shortest_path_unweight) patterns.**

## Solution Approach

The implementation uses Dijkstra's algorithm as the core component to find shortest paths. Here's how the solution works step by step:

**1. [Graph](/problems/graph_intro) Construction:**
```python
g = defaultdict(list)   # Original graph
rg = defaultdict(list)  # Reverse graph
for f, t, w in edges:
    g[f].append((t, w))
    rg[t].append((f, w))
```
We build two adjacency lists:
- `g`: The original directed graph where `g[u]` contains all nodes `v` reachable from `u` with their edge weights
- `rg`: The reverse graph where all edges are flipped. This is crucial for finding distances from all nodes to `dest`

**2. Dijkstra's Algorithm Implementation:**
```python
def dijkstra(g, u):
    dist = [inf] * n
    dist[u] = 0
    q = [(0, u)]
    while q:
        d, u = heappop(q)
        if d > dist[u]:
            continue
        for v, w in g[u]:
            if dist[v] > dist[u] + w:
                dist[v] = dist[u] + w
                heappush(q, (dist[v], v))
    return dist
```
The algorithm uses a min-heap priority queue to always process the node with minimum distance first. It returns an array where `dist[i]` represents the shortest distance from source `u` to node `i`.

**3. Computing Three Distance Arrays:**
```python
d1 = dijkstra(g, src1)   # Distances from src1 to all nodes
d2 = dijkstra(g, src2)   # Distances from src2 to all nodes
d3 = dijkstra(rg, dest)  # Distances from all nodes to dest
```
- `d1[i]`: shortest distance from `src1` to node `i`
- `d2[i]`: shortest distance from `src2` to node `i`
- `d3[i]`: shortest distance from node `i` to `dest` (computed using the reverse [graph](/problems/graph_intro))

**4. Finding the Minimum Weight:**
```python
ans = min(sum(v) for v in zip(d1, d2, d3))
return -1 if ans >= inf else ans
```
For each node `i` from `0` to `n-1`, we calculate the total weight if `i` is the meeting point: `d1[i] + d2[i] + d3[i]`.

The `zip(d1, d2, d3)` creates tuples `(d1[i], d2[i], d3[i])` for each index `i`. We sum each tuple and find the minimum.

If the minimum is still infinity (meaning no valid path exists), we return `-1`.

**Time Complexity:** `O(E log V)` for each Dijkstra run, where `E` is the number of edges and `V` is the number of vertices. Since we run Dijkstra three times, the total is `O(3 * E log V) = O(E log V)`.

**Space Complexity:** `O(V + E)` for storing the graphs and distance arrays.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a graph with 4 nodes (0, 1, 2, 3) and the following edges:
- Edge 0→1 with weight 2
- Edge 0→2 with weight 5
- Edge 1→2 with weight 1
- Edge 1→3 with weight 3
- Edge 2→3 with weight 1

Let `src1 = 0`, `src2 = 1`, and `dest = 3`.

**Step 1: Build the graphs**
- Original graph `g`:
  - 0: [(1,2), (2,5)]
  - 1: [(2,1), (3,3)]
  - 2: [(3,1)]
  - 3: []

- Reverse graph `rg`:
  - 0: []
  - 1: [(0,2)]
  - 2: [(0,5), (1,1)]
  - 3: [(1,3), (2,1)]

**Step 2: Run Dijkstra from src1 = 0**
Starting from node 0, we find shortest paths to all nodes:
- d1[0] = 0 (start node)
- d1[1] = 2 (path: 0→1)
- d1[2] = 3 (path: 0→1→2, cost 2+1=3, better than direct 0→2 with cost 5)
- d1[3] = 4 (path: 0→1→2→3, cost 2+1+1=4)

**Step 3: Run Dijkstra from src2 = 1**
Starting from node 1, we find shortest paths to all nodes:
- d2[0] = ∞ (no path from 1 to 0)
- d2[1] = 0 (start node)
- d2[2] = 1 (path: 1→2)
- d2[3] = 2 (path: 1→2→3, cost 1+1=2, better than direct 1→3 with cost 3)

**Step 4: Run Dijkstra from dest = 3 on reverse graph**
This gives us distances from each node to dest in the original graph:
- d3[0] = 4 (in original: 0→1→2→3)
- d3[1] = 2 (in original: 1→2→3)
- d3[2] = 1 (in original: 2→3)
- d3[3] = 0 (destination node)

**Step 5: Calculate total weights for each potential meeting point**
For each node i, calculate d1[i] + d2[i] + d3[i]:
- Node 0: 0 + ∞ + 4 = ∞ (invalid - src2 can't reach node 0)
- Node 1: 2 + 0 + 2 = 4
- Node 2: 3 + 1 + 1 = 5
- Node 3: 4 + 2 + 0 = 6

**Step 6: Find the minimum**
The minimum value is 4, achieved when node 1 is the meeting point.

This means the optimal subgraph consists of:
- Path from src1 (0) to node 1: edge 0→1 (weight 2)
- Path from src2 (1) to node 1: no edges needed (already at node 1)
- Path from node 1 to dest (3): edges 1→2 and 2→3 (weights 1 + 1 = 2)

Total weight = 2 + 0 + 2 = 4

The selected edges form a subgraph where both src1 and src2 can reach dest with minimum total edge weight of 4.

## Python Solution

```python
from collections import defaultdict
from heapq import heappush, heappop
from typing import List
from math import inf


class Solution:
    def minimumWeight(
        self, n: int, edges: List[List[int]], src1: int, src2: int, dest: int
    ) -> int:
        """
        Find the minimum total weight for paths from src1 and src2 to dest,
        where both paths can meet at any intermediate node.
      
        Args:
            n: Number of nodes in the graph
            edges: List of [from_node, to_node, weight] representing directed edges
            src1: First source node
            src2: Second source node
            dest: Destination node
          
        Returns:
            Minimum total weight, or -1 if no valid path exists
        """
      
        def dijkstra(graph: defaultdict, start_node: int) -> List[float]:
            """
            Compute shortest distances from start_node to all other nodes.
          
            Args:
                graph: Adjacency list representation of the graph
                start_node: Starting node for shortest path computation
              
            Returns:
                List of shortest distances from start_node to each node
            """
            # Initialize distances with infinity
            distances = [inf] * n
            distances[start_node] = 0
          
            # Priority queue: (distance, node)
            priority_queue = [(0, start_node)]
          
            while priority_queue:
                current_distance, current_node = heappop(priority_queue)
              
                # Skip if we've already found a better path to this node
                if current_distance > distances[current_node]:
                    continue
              
                # Explore neighbors
                for neighbor, edge_weight in graph[current_node]:
                    new_distance = distances[current_node] + edge_weight
                  
                    # Update distance if we found a shorter path
                    if distances[neighbor] > new_distance:
                        distances[neighbor] = new_distance
                        heappush(priority_queue, (new_distance, neighbor))
          
            return distances
      
        # Build forward graph and reverse graph
        forward_graph = defaultdict(list)
        reverse_graph = defaultdict(list)
      
        for from_node, to_node, weight in edges:
            forward_graph[from_node].append((to_node, weight))
            reverse_graph[to_node].append((from_node, weight))
      
        # Compute shortest distances from src1 to all nodes
        distances_from_src1 = dijkstra(forward_graph, src1)
      
        # Compute shortest distances from src2 to all nodes
        distances_from_src2 = dijkstra(forward_graph, src2)
      
        # Compute shortest distances from all nodes to dest (using reverse graph)
        distances_to_dest = dijkstra(reverse_graph, dest)
      
        # Find minimum total weight by considering each node as meeting point
        # Total weight = distance(src1 -> node) + distance(src2 -> node) + distance(node -> dest)
        min_total_weight = min(
            dist1 + dist2 + dist3 
            for dist1, dist2, dist3 in zip(distances_from_src1, distances_from_src2, distances_to_dest)
        )
      
        # Return -1 if no valid path exists, otherwise return the minimum weight
        return -1 if min_total_weight >= inf else min_total_weight
```

## Time and Space Complexity

**Time Complexity:** `O((V + E) * log V)`

The algorithm performs three Dijkstra's shortest path computations:
- Two from source nodes (`src1` and `src2`) using the original graph `g`
- One from the destination node using the reverse graph `rg`

Each Dijkstra implementation has a time complexity of `O((V + E) * log V)` where:
- `V = n` (number of vertices)
- `E` is the number of edges
- The heap operations (`heappush` and `heappop`) take `O(log V)` time
- Each edge is processed at most once, resulting in at most `E` heap operations
- Each vertex is extracted from the heap at most once, resulting in at most `V` heap operations

Since we run Dijkstra three times, the overall time complexity is `3 * O((V + E) * log V) = O((V + E) * log V)`.

The final loop to find the minimum sum iterates through `n` vertices once, which is `O(n)` and doesn't affect the overall complexity.

**Space Complexity:** `O(V + E)`

The space requirements include:
- Graph representation `g`: `O(E)` to store all edges
- Reverse graph `rg`: `O(E)` to store all edges in reverse
- Three distance arrays (`d1`, `d2`, `d3`): `3 * O(V) = O(V)`
- Priority queue in Dijkstra: `O(V)` in the worst case
- Recursive call stack: Not applicable as the implementation is iterative

The dominant space complexity is `O(V + E)` for storing the graph structures and distance arrays.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Handle the Reverse Graph for Distance to Destination**

**Pitfall:** A common mistake is trying to compute distances from all nodes to the destination using the forward graph. Since the graph is directed, you cannot simply run Dijkstra from each node to find distances to the destination - this would be inefficient and incorrect.

**Why it happens:** It's intuitive to think about running Dijkstra from the destination node in the forward graph, but this gives you distances FROM the destination TO all other nodes, not the other way around.

**Incorrect approach:**
```python
# WRONG: This computes distances FROM dest TO all nodes
distances_to_dest = dijkstra(forward_graph, dest)
```

**Correct solution:** Build a reverse graph where all edges are flipped, then run Dijkstra from the destination. This effectively computes distances from all nodes to the destination:
```python
# Build reverse graph
reverse_graph = defaultdict(list)
for from_node, to_node, weight in edges:
    reverse_graph[to_node].append((from_node, weight))

# Now this correctly computes distances FROM all nodes TO dest
distances_to_dest = dijkstra(reverse_graph, dest)
```

### 2. **Not Checking for Already Processed Nodes in Dijkstra**

**Pitfall:** Processing the same node multiple times in Dijkstra when it appears in the priority queue with different distances.

**Why it happens:** The priority queue might contain multiple entries for the same node with different distances. Without checking, you might process outdated entries.

**Incorrect approach:**
```python
def dijkstra(graph, start_node):
    distances = [inf] * n
    distances[start_node] = 0
    priority_queue = [(0, start_node)]
  
    while priority_queue:
        current_distance, current_node = heappop(priority_queue)
        # MISSING: Check if this is an outdated entry
      
        for neighbor, edge_weight in graph[current_node]:
            new_distance = distances[current_node] + edge_weight
            if distances[neighbor] > new_distance:
                distances[neighbor] = new_distance
                heappush(priority_queue, (new_distance, neighbor))
```

**Correct solution:** Skip processing if the current distance from the queue is greater than the already recorded distance:
```python
while priority_queue:
    current_distance, current_node = heappop(priority_queue)
  
    # Skip if we've already found a better path
    if current_distance > distances[current_node]:
        continue
  
    # ... rest of the algorithm
```

### 3. **Integer Overflow or Incorrect Infinity Handling**

**Pitfall:** Using a fixed large number instead of `float('inf')` can cause overflow when adding distances, or the "infinity" value might be too small for the actual edge weights in the problem.

**Incorrect approach:**
```python
# Using a fixed "large" number
INF = 10**9
distances = [INF] * n

# Later in the code:
min_weight = min(d1[i] + d2[i] + d3[i] for i in range(n))
# If all three are INF (10^9), sum is 3*10^9, which might cause issues
```

**Correct solution:** Use Python's `float('inf')` which handles arithmetic operations correctly:
```python
from math import inf

distances = [inf] * n
# inf + inf + inf = inf (handled correctly)
# inf >= inf evaluates to True
```

### 4. **Not Considering Edge Cases for the Meeting Point**

**Pitfall:** Assuming the meeting point must be different from source or destination nodes.

**Why it happens:** It's easy to think the paths must "meet in the middle," but the optimal meeting point could be:
- At `src1` (if `src2` goes through `src1` to reach `dest`)
- At `src2` (if `src1` goes through `src2` to reach `dest`)
- At `dest` (both sources go directly to destination)

**Solution:** The code correctly considers ALL nodes (0 to n-1) as potential meeting points, including the source and destination nodes themselves:
```python
# Correctly checks all nodes as potential meeting points
min_total_weight = min(
    dist1 + dist2 + dist3 
    for dist1, dist2, dist3 in zip(distances_from_src1, distances_from_src2, distances_to_dest)
)
```

