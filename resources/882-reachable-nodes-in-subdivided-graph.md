# 882. Reachable Nodes In Subdivided Graph

> Source: https://algo.monster/liteproblems/882

## Problem Statement

You have an undirected [graph](/problems/graph_intro) with `n` nodes labeled from `0` to `n - 1`. Each edge in this graph can be subdivided into a chain of smaller edges with new intermediate nodes.

The [graph](/problems/graph_intro) is given as a 2D array `edges` where each element `edges[i] = [ui, vi, cnti]` represents:
- An edge between nodes `ui` and `vi` in the original graph
- `cnti` is the number of new nodes that will be inserted to subdivide this edge

When you subdivide an edge `[ui, vi]` with `cnti` new nodes:
- The original edge is replaced with `(cnti + 1)` new edges
- `cnti` new nodes `x1, x2, ..., xcnti` are created
- The new edges form a chain: `[ui, x1], [x1, x2], [x2, x3], ..., [xcnti-1, xcnti], [xcnti, vi]`

For example, if an edge `[0, 1]` has `cnt = 2`, it becomes: `[0, x1], [x1, x2], [x2, 1]` with two new intermediate nodes.

After creating this new [graph](/problems/graph_intro) with all the subdivided edges, you need to find how many nodes are reachable from node `0` within `maxMoves` steps. A node is considered reachable if the shortest distance from node `0` to that node is at most `maxMoves`.

The problem asks you to return the total count of reachable nodes, which includes:
- Original nodes that can be reached within `maxMoves` steps
- New intermediate nodes created during edge subdivision that can be reached within `maxMoves` steps

Given:
- `edges`: The original [graph](/problems/graph_intro) edges with subdivision counts
- `maxMoves`: The maximum number of steps allowed
- `n`: The number of nodes in the original graph

Return: The total number of nodes reachable from node `0` in the new subdivided [graph](/problems/graph_intro).

## Intuition

The key insight is that we need to handle two types of nodes differently: the original nodes and the newly created intermediate nodes from edge subdivision.

For the original nodes, we can use Dijkstra's algorithm to find the shortest distance from node `0` to every other node. But here's the trick - when we subdivide an edge with `cnt` intermediate nodes, the edge weight becomes `cnt + 1` (since we need `cnt + 1` steps to traverse from one end to the other through all intermediate nodes).

Once we have the shortest distances to all original nodes, counting reachable original nodes is straightforward - any node with distance ≤ `maxMoves` is reachable.

The challenging part is counting the reachable intermediate nodes. Consider an edge `[u, v]` that was subdivided with `cnt` intermediate nodes. These intermediate nodes form a line between `u` and `v`. We can reach some of these intermediate nodes from two directions:
- Starting from node `0`, reaching `u`, then moving into the edge towards `v`
- Starting from node `0`, reaching `v`, then moving into the edge towards `u`

From node `u`, we can reach at most `maxMoves - dist[u]` steps into the edge (if we have moves left after reaching `u`). Similarly, from node `v`, we can reach at most `maxMoves - dist[v]` steps into the edge.

The total intermediate nodes we can reach on edge `[u, v]` is the minimum of:
- The actual number of intermediate nodes (`cnt`)
- The sum of nodes reachable from both ends (`a + b`), where:
  - `a = min(cnt, max(0, maxMoves - dist[u]))`
  - `b = min(cnt, max(0, maxMoves - dist[v]))`

We take the minimum because if `a + b > cnt`, it means we're counting some nodes from both directions, but the actual number of intermediate nodes is only `cnt`.

This approach elegantly handles all cases - whether we can reach the entire edge, only parts of it from one or both ends, or none at all.

**Learn more about [Graph](/problems/graph_intro), [Shortest Path](/problems/shortest_path_unweight) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The implementation uses Dijkstra's algorithm with a min-heap to find shortest distances, followed by counting reachable nodes.

**Step 1: Build the [Graph](/problems/graph_intro)**
```python
g = defaultdict(list)
for u, v, cnt in edges:
    g[u].append((v, cnt + 1))
    g[v].append((u, cnt + 1))
```
We create an adjacency list where each edge weight is `cnt + 1` (the number of steps needed to traverse the subdivided edge). Since the graph is undirected, we add edges in both directions.

**Step 2: Run Dijkstra's Algorithm**
```python
q = [(0, 0)]  # (distance, node)
dist = [0] + [inf] * (n - 1)
while q:
    d, u = heappop(q)
    for v, cnt in g[u]:
        if (t := d + cnt) < dist[v]:
            dist[v] = t
            heappush(q, (t, v))
```
- Initialize a min-heap with `(0, 0)` - starting from node `0` with distance `0`
- Initialize distance array: `dist[0] = 0` and all others to infinity
- Process nodes in order of increasing distance
- For each neighbor `v` of current node `u`, if we find a shorter path, update `dist[v]` and add to heap
- Note: This implementation does not skip stale heap entries, so a node can be popped more than once. A stale pop cannot lower any distance, so the results are still correct; it only adds extra work

**Step 3: Count Reachable Original Nodes**
```python
ans = sum(d <= maxMoves for d in dist)
```
Count all original nodes whose shortest distance from node `0` is at most `maxMoves`.

**Step 4: Count Reachable Intermediate Nodes**
```python
for u, v, cnt in edges:
    a = min(cnt, max(0, maxMoves - dist[u]))
    b = min(cnt, max(0, maxMoves - dist[v]))
    ans += min(cnt, a + b)
```
For each original edge with `cnt` intermediate nodes:
- `a`: Maximum intermediate nodes reachable from `u` side = `min(cnt, max(0, maxMoves - dist[u]))`
  - We have `maxMoves - dist[u]` moves left after reaching `u`
  - Can't go negative (hence `max(0, ...)`)
  - Can't exceed the actual number of intermediate nodes (hence `min(cnt, ...)`)
- `b`: Similarly for the `v` side
- Total reachable on this edge = `min(cnt, a + b)`
  - If `a + b ≤ cnt`: We can reach `a` nodes from `u` side and `b` from `v` side without overlap
  - If `a + b > cnt`: The entire edge is reachable, so we count all `cnt` intermediate nodes

The algorithm runs in `O(E log V)` time for Dijkstra's algorithm plus `O(E)` for counting intermediate nodes, where `E` is the number of edges and `V` is the number of vertices.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- `n = 4` (nodes: 0, 1, 2, 3)
- `edges = [[0,1,4],[1,2,6],[0,2,8],[1,3,1]]`
- `maxMoves = 10`

**Step 1: Build the Graph**

After subdivision, each edge weight becomes `cnt + 1`:
- Edge [0,1] with cnt=4 → weight = 5 (needs 5 moves to traverse)
- Edge [1,2] with cnt=6 → weight = 7
- Edge [0,2] with cnt=8 → weight = 9
- Edge [1,3] with cnt=1 → weight = 2

The graph looks like:
```
    0 ---(5)--- 1 ---(2)--- 3
     \         /
      \       /
       (9)  (7)
         \ /
          2
```

**Step 2: Run Dijkstra's Algorithm**

Starting from node 0:
- Process node 0 (dist=0):
  - To node 1: dist[1] = 0 + 5 = 5
  - To node 2: dist[2] = 0 + 9 = 9
- Process node 1 (dist=5):
  - To node 2: 5 + 7 = 12 > 9 (no update)
  - To node 3: dist[3] = 5 + 2 = 7
- Process node 3 (dist=7): no improvements
- Process node 2 (dist=9): no improvements

Final distances: `dist = [0, 5, 9, 7]`

**Step 3: Count Reachable Original Nodes**

Nodes with distance ≤ 10:
- Node 0: dist=0 ✓
- Node 1: dist=5 ✓
- Node 2: dist=9 ✓
- Node 3: dist=7 ✓

All 4 original nodes are reachable. Count = 4.

**Step 4: Count Reachable Intermediate Nodes**

For edge [0,1] with 4 intermediate nodes:
- From node 0: can go `min(4, max(0, 10-0)) = min(4, 10) = 4` steps
- From node 1: can go `min(4, max(0, 10-5)) = min(4, 5) = 4` steps
- Total: `min(4, 4+4) = min(4, 8) = 4` intermediate nodes

For edge [1,2] with 6 intermediate nodes:
- From node 1: can go `min(6, max(0, 10-5)) = min(6, 5) = 5` steps
- From node 2: can go `min(6, max(0, 10-9)) = min(6, 1) = 1` step
- Total: `min(6, 5+1) = min(6, 6) = 6` intermediate nodes

For edge [0,2] with 8 intermediate nodes:
- From node 0: can go `min(8, max(0, 10-0)) = min(8, 10) = 8` steps
- From node 2: can go `min(8, max(0, 10-9)) = min(8, 1) = 1` step
- Total: `min(8, 8+1) = min(8, 9) = 8` intermediate nodes

For edge [1,3] with 1 intermediate node:
- From node 1: can go `min(1, max(0, 10-5)) = min(1, 5) = 1` step
- From node 3: can go `min(1, max(0, 10-7)) = min(1, 3) = 1` step
- Total: `min(1, 1+1) = min(1, 2) = 1` intermediate node

**Final Answer:**
- Original nodes: 4
- Intermediate nodes: 4 + 6 + 8 + 1 = 19
- Total reachable nodes: 4 + 19 = 23

## Python Solution

```python
class Solution:
    def reachableNodes(self, edges: List[List[int]], maxMoves: int, n: int) -> int:
        # Build adjacency list representation of the graph
        # Each edge has 'cnt' intermediate nodes, so total distance is cnt + 1
        graph = defaultdict(list)
        for start_node, end_node, intermediate_count in edges:
            graph[start_node].append((end_node, intermediate_count + 1))
            graph[end_node].append((start_node, intermediate_count + 1))
      
        # Initialize priority queue for Dijkstra's algorithm
        # Format: (distance, node)
        priority_queue = [(0, 0)]
      
        # Initialize distances array (distance from node 0 to all other nodes)
        # dist[0] = 0 (starting node), all others initialized to infinity
        distances = [0] + [float('inf')] * (n - 1)
      
        # Run Dijkstra's algorithm to find shortest distances from node 0
        while priority_queue:
            current_distance, current_node = heappop(priority_queue)
          
            # Explore neighbors of current node
            for neighbor, edge_weight in graph[current_node]:
                new_distance = current_distance + edge_weight
              
                # If we found a shorter path to the neighbor, update it
                if new_distance < distances[neighbor]:
                    distances[neighbor] = new_distance
                    heappush(priority_queue, (new_distance, neighbor))
      
        # Count main nodes that are reachable within maxMoves
        reachable_count = sum(distance <= maxMoves for distance in distances)
      
        # Count intermediate nodes on edges that can be reached
        for start_node, end_node, intermediate_count in edges:
            # Calculate how many intermediate nodes can be reached from start_node
            reachable_from_start = min(intermediate_count, max(0, maxMoves - distances[start_node]))
          
            # Calculate how many intermediate nodes can be reached from end_node
            reachable_from_end = min(intermediate_count, max(0, maxMoves - distances[end_node]))
          
            # Total intermediate nodes reached on this edge (cannot exceed the actual count)
            reachable_count += min(intermediate_count, reachable_from_start + reachable_from_end)
      
        return reachable_count
```

## Time and Space Complexity

**Time Complexity:** `O((E + V) * log V)` where `E` is the number of edges and `V` is the number of vertices (n).

- Building the adjacency list takes `O(E)` time as we iterate through all edges once.
- The Dijkstra's algorithm implementation uses a min-heap. In the worst case, each edge can be relaxed once, leading to `O(E)` heap operations.
- Each heap operation (push/pop) takes `O(log E)` time, since the heap can hold up to `O(E)` entries (one per successful relaxation). Because `E ≤ V²`, `log E = O(log V)`.
- The final loop to count reachable nodes within edges takes `O(E)` time.
- Therefore, the dominant operation is the Dijkstra's algorithm portion: `O(E * log V)`.
- Since we also iterate through vertices for initialization and counting, the total complexity is `O((E + V) * log V)`.

**Space Complexity:** `O(E + V)`

- The adjacency list `graph` stores all edges twice (bidirectional), requiring `O(E)` space.
- The distance array `distances` requires `O(V)` space.
- The priority queue can contain up to `O(E)` entries in the worst case, because a node may be pushed once per successful relaxation.
- Overall space complexity is `O(E + V)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Processing Stale Heap Entries**

The solution above pushes a new heap entry each time a shorter distance is found, so older entries for the same node stay in the heap. When such a stale entry is popped, its distance is larger than `distances[current_node]`.

```python
while priority_queue:
    current_distance, current_node = heappop(priority_queue)
    for neighbor, edge_weight in graph[current_node]:
        new_distance = current_distance + edge_weight
        if new_distance < distances[neighbor]:
            distances[neighbor] = new_distance
            heappush(priority_queue, (new_distance, neighbor))
```

This does not produce wrong answers: relaxing from a stale entry uses a distance that is too large, so `new_distance < distances[neighbor]` fails for any neighbor whose distance is already correct. It does repeat work, scanning the neighbors of the same node more than once.

**Optimization:** Skip stale entries right after popping:
```python
while priority_queue:
    current_distance, current_node = heappop(priority_queue)

    # Skip if we've already found a better path to this node
    if current_distance > distances[current_node]:
        continue

    for neighbor, edge_weight in graph[current_node]:
        new_distance = current_distance + edge_weight
        if new_distance < distances[neighbor]:
            distances[neighbor] = new_distance
            heappush(priority_queue, (new_distance, neighbor))
```

### 2. **Sizing the Distance Array**

The nodes are labeled `0` to `n - 1`, so the distance array needs exactly `n` entries:
```python
distances = [0] + [float('inf')] * (n - 1)
```

Writing `[0] + [float('inf')] * n` creates `n + 1` entries. Here that extra entry stays at infinity and is never counted, so the answer is unchanged, but the array no longer matches the node labels. Using a dictionary or an array that is too short (for example, sized by the number of edges) would cause wrong counts or an `IndexError`.

### 3. **Edge Weight Confusion**

A common conceptual mistake is confusing the number of intermediate nodes with the edge weight:
- If an edge has `cnt` intermediate nodes, the total distance to traverse it is `cnt + 1`
- Many incorrectly use `cnt` as the edge weight directly

**Correct approach:**
```python
# Each edge with 'cnt' intermediate nodes has weight cnt + 1
graph[start_node].append((end_node, intermediate_count + 1))
```

### Complete Solution with the Stale-Entry Check:

```python
class Solution:
    def reachableNodes(self, edges: List[List[int]], maxMoves: int, n: int) -> int:
        from collections import defaultdict
        from heapq import heappush, heappop
      
        # Build adjacency list
        graph = defaultdict(list)
        for u, v, cnt in edges:
            graph[u].append((v, cnt + 1))
            graph[v].append((u, cnt + 1))
      
        # Dijkstra's algorithm
        pq = [(0, 0)]  # (distance, node)
        dist = [float('inf')] * n
        dist[0] = 0
      
        while pq:
            d, u = heappop(pq)
          
            # Skip if we've already processed this node with a better distance
            if d > dist[u]:
                continue
          
            for v, weight in graph[u]:
                new_dist = d + weight
                if new_dist < dist[v]:
                    dist[v] = new_dist
                    heappush(pq, (new_dist, v))
      
        # Count reachable original nodes
        result = sum(d <= maxMoves for d in dist)
      
        # Count reachable intermediate nodes
        for u, v, cnt in edges:
            # Nodes reachable from u side
            from_u = min(cnt, max(0, maxMoves - dist[u]))
            # Nodes reachable from v side  
            from_v = min(cnt, max(0, maxMoves - dist[v]))
            # Total intermediate nodes reachable on this edge
            result += min(cnt, from_u + from_v)
      
        return result
```

