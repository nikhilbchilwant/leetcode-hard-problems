# 2045. Second Minimum Time to Reach Destination

> Source: https://algo.monster/liteproblems/2045

## Problem Statement

You have a city represented as an undirected connected [graph](/problems/graph_intro) with `n` vertices labeled from `1` to `n`. The graph is defined by an edge list `edges`, where `edges[i] = [ui, vi]` represents a bidirectional edge between vertices `ui` and `vi`. Each edge takes exactly `time` minutes to traverse.

Every vertex has a traffic signal that alternates between green and red. All signals in the city change simultaneously every `change` minutes, starting with green when your journey begins. The key rules for movement are:
- You can enter any vertex at any time
- You can only leave a vertex when its signal is green
- You cannot wait at a vertex if the signal is green (you must leave immediately)

Your task is to find the **second minimum time** to travel from vertex `1` to vertex `n`.

The second minimum value is defined as the smallest value that is strictly larger than the minimum value. For example:
- In `[2, 3, 4]`, the second minimum is `3`
- In `[2, 2, 4]`, the second minimum is `4`

Important notes:
- You can revisit any vertex multiple times, including the start (vertex `1`) and destination (vertex `n`)
- At time `0`, all signals have just turned green
- The [graph](/problems/graph_intro) guarantees no self-loops and at most one edge between any pair of vertices

The challenge involves finding not just the [shortest path](/problems/shortest_path_unweight), but the second shortest path while accounting for the traffic signal constraints that may force you to wait at certain vertices when signals are red.

## Intuition

The key insight is that we're not just looking for the [shortest path](/problems/shortest_path_unweight), but the **second shortest** path. In a standard BFS, we typically stop once we reach a vertex for the first time since that guarantees the shortest path. However, for this problem, we need to continue exploring to find a slightly longer path.

Think of it this way: when we reach vertex `n` for the first time through BFS, we've found the minimum number of edges needed. But we need to keep searching for the next distinct path length to vertex `n`.

The solution maintains **two distance values** for each vertex: `dist[v][0]` for the shortest distance and `dist[v][1]` for the second shortest. When we explore from a vertex `u` to its neighbor `v`:
- If we find a path shorter than the current shortest (`d + 1 < dist[v][0]`), we update the shortest distance
- If we find a path that's longer than the shortest but shorter than the current second shortest (`dist[v][0] < d + 1 < dist[v][1]`), we update the second shortest

This modified BFS ensures we capture both the minimum and second minimum number of edges to reach each vertex.

Once we have the second minimum number of edges to vertex `n`, we need to calculate the actual time. This is where the traffic signals come in. For each edge we traverse:
1. We add `time` minutes for the traversal
2. If we're not at the last edge and we arrive when the signal is red (determined by `(ans // change) % 2 == 1`), we must wait until it turns green

The signal timing works as follows: signals alternate between green and red every `change` minutes. If `(total_time // change)` is even, the signal is green; if odd, it's red. When we need to wait at a red signal, we round up to the next green period using `(ans + change) // change * change`.

This two-phase approach - first finding the second [shortest path](/problems/shortest_path_unweight) length via modified BFS, then calculating the actual time considering traffic signals - gives us the second minimum time to reach the destination.

**Learn more about [Breadth-First Search](/problems/bfs_intro), [Graph](/problems/graph_intro) and [Shortest Path](/problems/shortest_path_unweight) patterns.**

## Solution Approach

The implementation uses a modified BFS with specific data structures to track two shortest distances for each vertex:

**1. [Graph](/problems/graph_intro) Representation:**
```python
g = defaultdict(set)
for u, v in edges:
    g[u].add(v)
    g[v].add(u)
```
We build an adjacency list using a `defaultdict(set)` to store the bidirectional edges. Sets prevent duplicate edges and provide O(1) lookup.

**2. Distance Tracking:**
```python
dist = [[inf] * 2 for _ in range(n + 1)]
dist[1][1] = 0
```
We create a 2D array where `dist[v][0]` stores the shortest distance to vertex `v`, and `dist[v][1]` stores the second shortest. Initially, all distances are set to infinity. We set `dist[1][1] = 0` to mark that we start from vertex 1 with 0 edges.

**3. Modified BFS:**
```python
q = deque([(1, 0)])
while q:
    u, d = q.popleft()
    for v in g[u]:
        if d + 1 < dist[v][0]:
            dist[v][0] = d + 1
            q.append((v, d + 1))
        elif dist[v][0] < d + 1 < dist[v][1]:
            dist[v][1] = d + 1
            if v == n:
                break
            q.append((v, d + 1))
```
The BFS queue stores tuples of `(vertex, distance)`. For each neighbor `v` of current vertex `u`:
- If we find a shorter path than the current shortest, we update `dist[v][0]` and continue exploring
- If we find a path longer than the shortest but shorter than the second shortest, we update `dist[v][1]`
- We can stop early if we've found the second [shortest path](/problems/shortest_path_unweight) to vertex `n`

**4. Time Calculation with Traffic Signals:**
```python
ans = 0
for i in range(dist[n][1]):
    ans += time
    if i < dist[n][1] - 1 and (ans // change) % 2 == 1:
        ans = (ans + change) // change * change
```
After finding the second [shortest path](/problems/shortest_path_unweight) with `dist[n][1]` edges, we calculate the actual time:
- For each edge traversal, add `time` minutes
- After each edge (except the last), check if we arrive during a red signal
- Signal state: `(ans // change) % 2 == 0` means green, `== 1` means red
- If red, we wait until the next green period: `(ans + change) // change * change` rounds up to the next multiple of `change` where the signal turns green

The algorithm efficiently finds both the shortest and second shortest paths in O(V + E) time using BFS, then calculates the actual travel time considering traffic signal constraints.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example:**
- Graph: 4 vertices with edges: `[[1,2], [1,3], [2,4], [3,4]]`
- `time = 3` minutes per edge
- `change = 5` minutes for signal changes
- Find second minimum time from vertex 1 to vertex 4

**Step 1: Build Graph**
```
Adjacency list:
1 -> {2, 3}
2 -> {1, 4}
3 -> {1, 4}
4 -> {2, 3}
```

**Step 2: Modified BFS to Find Second Shortest Path**

Initialize `dist` array (showing only relevant vertices):
```
dist[1] = [inf, 0]  // Start vertex
dist[2] = [inf, inf]
dist[3] = [inf, inf]
dist[4] = [inf, inf]
```

BFS Process:
- Start with queue: `[(1, 0)]`

- Process vertex 1 with distance 0:
  - Visit neighbor 2: `0 + 1 = 1 < inf`, update `dist[2][0] = 1`, add `(2, 1)` to queue
  - Visit neighbor 3: `0 + 1 = 1 < inf`, update `dist[3][0] = 1`, add `(3, 1)` to queue
  - Queue: `[(2, 1), (3, 1)]`

- Process vertex 2 with distance 1:
  - Visit neighbor 1: `1 + 1 = 2`, but we don't need to track back to start
  - Visit neighbor 4: `1 + 1 = 2 < inf`, update `dist[4][0] = 2`, add `(4, 2)` to queue
  - Queue: `[(3, 1), (4, 2)]`

- Process vertex 3 with distance 1:
  - Visit neighbor 1: `1 + 1 = 2`, skip
  - Visit neighbor 4: `1 + 1 = 2`, already have `dist[4][0] = 2`, so check second shortest
    - Since `2 < 1 + 1 < inf` is false (2 is not less than 2), skip
  - Queue: `[(4, 2)]`

- Process vertex 4 with distance 2:
  - Visit neighbor 2: `2 + 1 = 3 > dist[2][0] = 1` and `1 < 3 < inf`, update `dist[2][1] = 3`, add `(2, 3)` to queue
  - Visit neighbor 3: `2 + 1 = 3 > dist[3][0] = 1` and `1 < 3 < inf`, update `dist[3][1] = 3`, add `(3, 3)` to queue
  - Queue: `[(2, 3), (3, 3)]`

- Process vertex 2 with distance 3:
  - Visit neighbor 4: `3 + 1 = 4 > dist[4][0] = 2` and `2 < 4 < inf`, update `dist[4][1] = 4`
  - Since we found second shortest to vertex 4, we can stop

Final distances to vertex 4:
- Shortest path: 2 edges (1→2→4 or 1→3→4)
- Second shortest path: 4 edges (e.g., 1→2→1→3→4)

**Step 3: Calculate Time with Traffic Signals**

For the second shortest path with 4 edges:

Edge 1 (time 0 to 3):
- Add 3 minutes: `ans = 3`
- Check signal: `3 // 5 = 0`, `0 % 2 = 0` (green), no wait needed

Edge 2 (time 3 to 6):
- Add 3 minutes: `ans = 6`
- Check signal: `6 // 5 = 1`, `1 % 2 = 1` (red), must wait
- Wait until next green: `(6 + 5) // 5 * 5 = 10`
- `ans = 10`

Edge 3 (time 10 to 13):
- Add 3 minutes: `ans = 13`
- Check signal: `13 // 5 = 2`, `2 % 2 = 0` (green), no wait needed

Edge 4 (time 13 to 16):
- Add 3 minutes: `ans = 16`
- This is the last edge, no signal check needed

**Result:** Second minimum time = 16 minutes

The algorithm successfully finds that while the shortest path takes 2 edges, the second shortest takes 4 edges. With traffic signals causing a wait after the second edge, the total time becomes 16 minutes.

## Python Solution

```python
from collections import defaultdict, deque
from typing import List

class Solution:
    def secondMinimum(
        self, n: int, edges: List[List[int]], time: int, change: int
    ) -> int:
        # Build adjacency list representation of the graph
        graph = defaultdict(set)
        for u, v in edges:
            graph[u].add(v)
            graph[v].add(u)
      
        # Initialize BFS queue with starting node 1 and distance 0
        queue = deque([(1, 0)])
      
        # Track two shortest distances for each node
        # distances[i][0] = shortest distance to node i
        # distances[i][1] = second shortest distance to node i
        distances = [[float('inf')] * 2 for _ in range(n + 1)]
        distances[1][1] = 0  # Starting node has distance 0
      
        # BFS to find shortest and second shortest paths
        while queue:
            current_node, current_distance = queue.popleft()
          
            # Explore all neighbors
            for neighbor in graph[current_node]:
                new_distance = current_distance + 1
              
                # If we found a shorter path than the current shortest
                if new_distance < distances[neighbor][0]:
                    distances[neighbor][0] = new_distance
                    queue.append((neighbor, new_distance))
                  
                # If we found a path between shortest and second shortest
                elif distances[neighbor][0] < new_distance < distances[neighbor][1]:
                    distances[neighbor][1] = new_distance
                    # Early termination if we found second shortest to destination
                    if neighbor == n:
                        break
                    queue.append((neighbor, new_distance))
      
        # Calculate actual time considering traffic lights
        total_time = 0
        second_shortest_path_length = distances[n][1]
      
        for step in range(second_shortest_path_length):
            # Add time for one edge traversal
            total_time += time
          
            # Check if we need to wait for green light (except for last step)
            if step < second_shortest_path_length - 1:
                # If we arrive during red light phase, wait for green
                if (total_time // change) % 2 == 1:
                    # Round up to next green light period
                    total_time = ((total_time + change) // change) * change
      
        return total_time
```

## Time and Space Complexity

**Time Complexity:** `O(V + E)` where `V` is the number of vertices (n) and `E` is the number of edges.

- Building the adjacency list takes `O(E)` time as we iterate through all edges once.
- The BFS traversal visits each vertex at most twice (once for the shortest path and once for the second shortest path).
- Each edge is explored at most twice from each direction, resulting in `O(E)` edge explorations.
- The final loop to calculate the actual time runs `dist[n][1]` times, which is at most `O(V)` since the second shortest path length is bounded by the number of vertices.
- Overall: `O(V + E)` for graph traversal + `O(V)` for time calculation = `O(V + E)`

**Space Complexity:** `O(V + E)`

- The adjacency list `g` stores all edges, requiring `O(E)` space.
- The `dist` array is a 2D array of size `(n+1) × 2`, requiring `O(V)` space.
- The BFS queue can contain at most `O(V)` elements in the worst case, as each vertex can be added at most twice.
- Overall: `O(E)` for adjacency list + `O(V)` for distance array + `O(V)` for queue = `O(V + E)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Initial Distance Setting**
A critical pitfall in this solution is the initialization of `distances[1][1] = 0` instead of `distances[1][0] = 0`. This means we're marking the second shortest distance to the starting node as 0, not the shortest distance.

**Why it's problematic:**
- The BFS logic expects `distances[v][0]` to contain the shortest distance
- With `distances[1][1] = 0` and `distances[1][0] = inf`, the algorithm treats infinity as the shortest path from node 1
- This causes incorrect comparisons when processing neighbors of node 1

**Solution:**
```python
distances[1][0] = 0  # Correct: shortest distance to start is 0
```

### 2. **Missing Path Validation for Second Shortest**
The code doesn't ensure that the second shortest path is actually different from the shortest path. In graphs where multiple paths of the same length exist, we might incorrectly identify a path of the same length as the "second shortest."

**Why it's problematic:**
- If there are multiple shortest paths with the same number of edges, the second shortest should be strictly longer
- The current code might return the same distance twice if not careful

**Solution:**
Add explicit checking to ensure the second shortest is strictly greater:
```python
# When updating second shortest distance
elif distances[neighbor][0] < new_distance < distances[neighbor][1]:
    # Only update if this creates a truly different path length
    if new_distance > distances[neighbor][0]:  # Ensure strictly greater
        distances[neighbor][1] = new_distance
```

### 3. **Premature Termination**
The early break when finding the second shortest path to node n might cause issues:
```python
if neighbor == n:
    break  # This breaks from the inner loop, not the while loop
```

**Why it's problematic:**
- This only breaks from the inner for loop, not the entire BFS
- Could lead to incomplete exploration of other potentially important paths
- The queue might still contain unprocessed nodes that could lead to better solutions

**Solution:**
Use a flag or different termination logic:
```python
found_second = False
while queue and not found_second:
    current_node, current_distance = queue.popleft()
    for neighbor in graph[current_node]:
        # ... existing logic ...
        elif distances[neighbor][0] < new_distance < distances[neighbor][1]:
            distances[neighbor][1] = new_distance
            if neighbor == n:
                found_second = True
                break
            queue.append((neighbor, new_distance))
```

### 4. **Integer Division Timing Issue**
The traffic light waiting calculation could have edge cases with integer division:
```python
total_time = ((total_time + change) // change) * change
```

**Why it's problematic:**
- If `total_time` is already at a change boundary, this formula still rounds up
- For example, if `total_time = 10` and `change = 5`, and we're in red phase, we should wait until `15`, but the formula might not handle boundary cases correctly

**Solution:**
Be more explicit about the waiting logic:
```python
if (total_time // change) % 2 == 1:  # Red light
    # Calculate next green light start time
    current_period = total_time // change
    next_green_period = current_period + 1 if current_period % 2 == 1 else current_period + 2
    total_time = next_green_period * change
```

