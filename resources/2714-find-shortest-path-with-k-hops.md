# 2714. Find Shortest Path with K Hops

> Source: https://algo.monster/liteproblems/2714

## Problem Statement

You are given a connected, undirected, weighted [graph](/problems/graph_intro) with `n` nodes (labeled from 0 to n-1) and a list of edges. Each edge is represented as `[u, v, w]`, meaning there's an edge between nodes `u` and `v` with weight `w`.

You need to find the [shortest path](/problems/shortest_path_unweight) from a starting node `s` to a destination node `d`, but with a special power: you can "hop over" (make the weight 0) for at most `k` edges along your path.

**Key Points:**
- The [graph](/problems/graph_intro) has `n` nodes (0-indexed)
- Each edge in `edges` is bidirectional with format `[u, v, w]` where `w` is the weight
- You want to go from node `s` to node `d`
- You can choose up to `k` edges and treat their weights as 0 (effectively "hopping" over them for free)
- Return the minimum total path weight from `s` to `d`

**Example Understanding:**
If you have a path from `s` to `d` that uses edges with weights [5, 3, 7, 2] and `k=2`, you could choose to hop over the two heaviest edges (7 and 5), making your total cost only 3 + 2 = 5 instead of 5 + 3 + 7 + 2 = 17.

The challenge is to strategically choose which edges to hop over to minimize the total path weight from source to destination.

## Intuition

The problem asks us to find the [shortest path](/problems/shortest_path_unweight) with the ability to skip up to `k` edges. This is a variation of the classic shortest path problem with an additional dimension - the number of "hops" we've used.

Think of it this way: at each node, we don't just care about "what's the shortest distance to reach here?" but rather "what's the shortest distance to reach here having used exactly 0 hops, 1 hop, 2 hops, ..., up to k hops?"

This leads us to track state in two dimensions:
- Which node we're at
- How many hops we've used so far

For each node, we maintain `k+1` different distances: `dist[node][hops_used]` represents the minimum distance to reach that node using exactly `hops_used` free edges.

When we're at a node with some number of hops already used, we have two choices for each neighboring edge:
1. **Use a hop**: Skip this edge (treat its weight as 0) if we haven't used all `k` hops yet. This moves us to the neighbor with one more hop used but no additional distance.
2. **Pay the cost**: Travel the edge normally, adding its weight to our distance but keeping the same hop count.

This naturally fits into Dijkstra's algorithm framework, but instead of relaxing just distances, we're relaxing distance-hop pairs. We use a priority queue with entries `(distance, node, hops_used)` and always process the smallest distance first.

The key insight is that we're essentially running Dijkstra on an expanded [graph](/problems/graph_intro) where each original node is split into `k+1` copies (one for each possible hop count), and we're finding the [shortest path](/problems/shortest_path_unweight) in this expanded state space.

Finally, the answer is the minimum among all `dist[d][0]`, `dist[d][1]`, ..., `dist[d][k]` since we can reach the destination using any number of hops from 0 to k.

**Learn more about [Graph](/problems/graph_intro), [Shortest Path](/problems/shortest_path_unweight) and [Heap (Priority Queue)](/problems/heap_intro) patterns.**

## Solution Approach

The solution implements a modified Dijkstra's algorithm to handle the additional dimension of hop counts.

**Step 1: Build the [Graph](/problems/graph_intro)**
```python
g = [[] for _ in range(n)]
for u, v, w in edges:
    g[u].append((v, w))
    g[v].append((u, w))
```
We create an adjacency list representation where `g[u]` contains all neighbors of node `u` along with their edge weights. Since the graph is undirected, we add edges in both directions.

**Step 2: Initialize Distance Matrix**
```python
dist = [[inf] * (k + 1) for _ in range(n)]
dist[s][0] = 0
```
We create a 2D distance array where `dist[u][t]` represents the minimum distance to reach node `u` using exactly `t` hops. Initially, all distances are set to infinity except `dist[s][0] = 0` (we start at source `s` with 0 hops used and 0 distance).

**Step 3: Priority Queue Setup**
```python
pq = [(0, s, 0)]  # (distance, node, hops_used)
```
We use a min-heap priority queue that stores tuples of `(current_distance, current_node, hops_used)`. This ensures we always process the state with minimum distance first.

**Step 4: Modified Dijkstra's Algorithm**
```python
while pq:
    dis, u, t = heappop(pq)
    for v, w in g[u]:
        # Option 1: Use a hop (make edge weight 0)
        if t + 1 <= k and dist[v][t + 1] > dis:
            dist[v][t + 1] = dis
            heappush(pq, (dis, v, t + 1))

        # Option 2: Pay the edge cost
        if dist[v][t] > dis + w:
            dist[v][t] = dis + w
            heappush(pq, (dis + w, v, t))
```

For each state `(dis, u, t)` popped from the queue, we explore all neighbors `v` of node `u`:

- **Using a hop**: If we haven't used all `k` hops (`t + 1 <= k`), we can reach neighbor `v` with the same distance `dis` but using one more hop. We update `dist[v][t + 1]` if this gives a better path.

- **Paying the cost**: We can reach neighbor `v` by paying the edge weight `w`, keeping the same hop count `t`. We update `dist[v][t]` if `dis + w` is better than the current known distance.

**Step 5: Find the Answer**
```python
return int(min(dist[d]))
```
The [shortest path](/problems/shortest_path_unweight) to destination `d` is the minimum among all possible hop counts: `min(dist[d][0], dist[d][1], ..., dist[d][k])`.

**Time Complexity**: `O((n * k) * log(n * k) * m)` where `m` is the number of edges, as we potentially process each state `(node, hops)` and each edge from that state.

**Space Complexity**: `O(n * k)` for the distance matrix and priority queue.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- Graph with 4 nodes (0, 1, 2, 3)
- Edges: `[[0,1,5], [1,2,3], [0,2,9], [2,3,1]]`
- Start: `s = 0`, Destination: `d = 3`
- Maximum hops: `k = 1`

**Graph visualization:**
```
    0 ---5--- 1
    |         |
    9         3
    |         |
    2 ---1--- 3
```

**Step 1: Initialize**
- Build adjacency list:
  - `g[0] = [(1,5), (2,9)]`
  - `g[1] = [(0,5), (2,3)]`
  - `g[2] = [(0,9), (1,3), (3,1)]`
  - `g[3] = [(2,1)]`

- Initialize distance matrix `dist[node][hops_used]`:
  ```
  dist[0][0] = 0, dist[0][1] = inf
  dist[1][0] = inf, dist[1][1] = inf
  dist[2][0] = inf, dist[2][1] = inf
  dist[3][0] = inf, dist[3][1] = inf
  ```

- Priority queue: `[(0, 0, 0)]` (distance=0, node=0, hops=0)

**Step 2: Process from node 0 with 0 hops used**
- Pop `(0, 0, 0)` from queue
- Explore neighbors:
  - To node 1 (edge weight 5):
    - Option 1 (use hop): `dist[1][1] = 0` (better than inf)
    - Option 2 (pay cost): `dist[1][0] = 5` (better than inf)
    - Add to queue: `(0, 1, 1)` and `(5, 1, 0)`
  - To node 2 (edge weight 9):
    - Option 1 (use hop): `dist[2][1] = 0` (better than inf)
    - Option 2 (pay cost): `dist[2][0] = 9` (better than inf)
    - Add to queue: `(0, 2, 1)` and `(9, 2, 0)`

**Step 3: Process minimum distance state `(0, 1, 1)`**
- Pop `(0, 1, 1)` (reached node 1 with distance 0 using 1 hop)
- Explore neighbors:
  - To node 0: Skip (no improvement)
  - To node 2 (edge weight 3):
    - Cannot use another hop (already used k=1)
    - Option 2 (pay cost): `dist[2][1] = 3` (0+3=3, worse than current 0)

**Step 4: Process `(0, 2, 1)`**
- Pop `(0, 2, 1)` (reached node 2 with distance 0 using 1 hop)
- Explore neighbors:
  - To node 3 (edge weight 1):
    - Cannot use another hop (already used k=1)
    - Option 2 (pay cost): `dist[3][1] = 1` (0+1=1, better than inf)
    - Add to queue: `(1, 3, 1)`

**Step 5: Process `(1, 3, 1)`**
- Pop `(1, 3, 1)` - we've reached destination!
- Continue processing remaining states...

**Step 6: Process `(5, 1, 0)`**
- Pop `(5, 1, 0)` (reached node 1 with distance 5 using 0 hops)
- To node 2 (edge weight 3):
  - Option 1 (use hop): `dist[2][1] = 5` (worse than current 0)
  - Option 2 (pay cost): `dist[2][0] = 8` (5+3=8, better than 9)
  - Add to queue: `(8, 2, 0)`

**Step 7: Process `(8, 2, 0)`**
- Pop `(8, 2, 0)`
- To node 3 (edge weight 1):
  - Option 1 (use hop): `dist[3][1] = 8` (worse than current 1)
  - Option 2 (pay cost): `dist[3][0] = 9` (8+1=9, better than inf)
  - Add to queue: `(9, 3, 0)`

**Final Result:**
- `dist[3][0] = 9` (path: 0→1→2→3 with weights 5+3+1)
- `dist[3][1] = 1` (path: 0→2→3 with hop on 0→2, paying only for 2→3)
- Answer: `min(9, 1) = 1`

The optimal path uses the direct expensive edge 0→2 as a free hop, then pays only 1 for edge 2→3.

## Python Solution

```python
from typing import List
from heapq import heappush, heappop
from math import inf

class Solution:
    def shortestPathWithHops(
        self, n: int, edges: List[List[int]], s: int, d: int, k: int
    ) -> int:
        # Build adjacency list representation of the graph
        graph = [[] for _ in range(n)]
        for node_u, node_v, weight in edges:
            graph[node_u].append((node_v, weight))
            graph[node_v].append((node_u, weight))  # Undirected graph

        # Initialize distance table: dist[node][hops_used] = minimum distance
        # dist[i][j] represents minimum distance to reach node i using exactly j free hops
        dist = [[inf] * (k + 1) for _ in range(n)]
        dist[s][0] = 0  # Starting node with 0 hops used has distance 0

        # Priority queue: (distance, current_node, hops_used)
        priority_queue = [(0, s, 0)]

        # Modified Dijkstra's algorithm
        while priority_queue:
            current_dist, current_node, hops_used = heappop(priority_queue)

            # Explore all neighbors
            for neighbor, edge_weight in graph[current_node]:
                # Option 1: Use a free hop (if available)
                if hops_used + 1 <= k and dist[neighbor][hops_used + 1] > current_dist:
                    dist[neighbor][hops_used + 1] = current_dist
                    heappush(priority_queue, (current_dist, neighbor, hops_used + 1))

                # Option 2: Pay the edge weight (normal traversal)
                if dist[neighbor][hops_used] > current_dist + edge_weight:
                    dist[neighbor][hops_used] = current_dist + edge_weight
                    heappush(priority_queue, (current_dist + edge_weight, neighbor, hops_used))

        # Return minimum distance to destination across all possible hop counts
        return int(min(dist[d]))
```

## Time and Space Complexity

**Time Complexity:** `O(n * k * (n + m) * log(n * k))` which simplifies to `O(n^2 * k * log(n * k))` in the worst case when `m = O(n^2)`.

The algorithm uses a modified Dijkstra's approach with an additional dimension for the number of hops. Each node can be visited up to `k + 1` times (for different hop counts from 0 to k). The priority queue can contain at most `n * (k + 1)` states. Each state extraction takes `O(log(n * k))` time. For each extracted state, we explore all adjacent edges. In the worst case with a dense graph where `m = O(n^2)`, we might push `O(n * k)` states to the queue, each taking `O(log(n * k))` time. However, the reference answer suggests `O(n^2 * log n)`, which appears to be considering the case where `k` is treated as a constant or when the graph is sparse and the dominant factor is `n^2` comparisons across all possible paths.

**Space Complexity:** `O(n * k)`

The space is dominated by:
- The `dist` array: `O(n * (k + 1))` = `O(n * k)`
- The adjacency list `g`: `O(n + m)` where `m` is the number of edges
- The priority queue can hold at most `O(n * k)` elements

Since the `dist` array is the dominant factor, the overall space complexity is `O(n * k)`, where `n` represents the number of nodes and `k` represents the maximum number of free hops allowed.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Not Checking if Current State is Already Processed**

The current implementation may process the same state `(node, hops_used)` multiple times if a better path to that state is found later. This happens because we don't check if the current distance from the priority queue is outdated.

**Problem:** When we pop `(dis, u, t)` from the priority queue, `dis` might be larger than `dist[u][t]` if we've already found a better path to state `(u, t)` after this entry was added to the queue.

**Solution:** Add a check to skip processing if we've already found a better path:

```python
while priority_queue:
    current_dist, current_node, hops_used = heappop(priority_queue)

    # Skip if we've already found a better path to this state
    if current_dist > dist[current_node][hops_used]:
        continue

    # Rest of the code remains the same...
```

### 2. **Inefficient Hop Usage Strategy**

A subtle pitfall is that the algorithm explores all possibilities but doesn't guarantee optimal hop usage early on. We might use hops on small-weight edges early in the path when saving them for heavier edges later would be better.

**Why this isn't actually a problem:** The algorithm explores ALL possible combinations of hop usage by maintaining separate distances for each hop count. The state space `(node, hops_used)` ensures we consider all strategies.

### 3. **Memory Limit Issues with Large Graphs**

For very large values of `n` and `k`, the 2D distance array `dist[n][k+1]` can consume significant memory.

**Solution:** Use a dictionary to store only visited states:

```python
from collections import defaultdict

dist = defaultdict(lambda: inf)
dist[(s, 0)] = 0
priority_queue = [(0, s, 0)]

while priority_queue:
    current_dist, current_node, hops_used = heappop(priority_queue)

    if current_dist > dist[(current_node, hops_used)]:
        continue

    for neighbor, edge_weight in graph[current_node]:
        # Option 1: Use a hop
        if hops_used + 1 <= k and dist[(neighbor, hops_used + 1)] > current_dist:
            dist[(neighbor, hops_used + 1)] = current_dist
            heappush(priority_queue, (current_dist, neighbor, hops_used + 1))

        # Option 2: Pay the cost
        if dist[(neighbor, hops_used)] > current_dist + edge_weight:
            dist[(neighbor, hops_used)] = current_dist + edge_weight
            heappush(priority_queue, (current_dist + edge_weight, neighbor, hops_used))

# Find answer
return min(dist[(d, i)] for i in range(k + 1))
```

### 4. **Edge Case: Unreachable Destination**

If the destination `d` is not reachable from source `s`, all values in `dist[d]` will remain `inf`.

**Solution:** Add a check before returning:

```python
result = min(dist[d])
return -1 if result == inf else int(result)
```

### 5. **Integer Overflow with Float Infinity**

Using `float('inf')` and then converting to `int` can cause issues in some systems.

**Solution:** Use a large integer constant instead:

```python
INF = 10**9  # or sys.maxsize
dist = [[INF] * (k + 1) for _ in range(n)]
```

The most critical pitfall is #1 (not checking for already processed states), which can significantly impact performance on dense graphs with many paths between nodes.

