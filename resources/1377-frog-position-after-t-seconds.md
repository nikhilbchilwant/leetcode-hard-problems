# 1377. Frog Position After T Seconds

> Source: https://algo.monster/liteproblems/1377

## Problem Statement

You have an undirected [tree](/problems/tree_intro) with `n` vertices numbered from `1` to `n`. A frog starts at vertex `1` and jumps between vertices following these rules:

- Each second, the frog jumps from its current vertex to an **unvisited** adjacent vertex
- The frog cannot return to previously visited vertices
- If multiple unvisited adjacent vertices exist, the frog randomly chooses one with equal probability
- If no unvisited adjacent vertices remain, the frog stays on the current vertex forever

The [tree](/problems/tree_intro) structure is given as an array `edges`, where `edges[i] = [ai, bi]` represents an edge between vertices `ai` and `bi`.

Your task is to find the probability that the frog is on vertex `target` after exactly `t` seconds.

**Key Points:**
- The frog starts at vertex `1`
- It can only visit each vertex once
- When multiple choices exist, each has equal probability
- If the frog gets stuck (no unvisited neighbors), it remains at that vertex
- The answer should be accurate within `10^-5`

**Example scenarios:**
- If the frog reaches the target vertex before `t` seconds but has unvisited neighbors, the probability is `0` (it will jump away)
- If the frog reaches the target at exactly `t` seconds, return the probability
- If the frog reaches the target before `t` seconds with no unvisited neighbors, it stays there, so return the probability

## Intuition

The key insight is to think of this problem as a probability distribution problem where the frog's position spreads out through the [tree](/problems/tree_intro) over time.

Imagine the frog as a drop of water starting at vertex 1 with probability `1.0`. At each second, this "probability mass" flows from each vertex to its unvisited neighbors, splitting equally among them. This naturally suggests a level-by-level exploration where we track:
1. Which vertices the frog can reach at each time step
2. The probability of being at each vertex

Why BFS over DFS? Consider what happens at each second:
- At `t=0`: frog is at vertex 1 with probability `1.0`
- At `t=1`: frog moves to one of vertex 1's neighbors
- At `t=2`: frog moves from those positions to their unvisited neighbors
- And so on...

This time-based progression maps perfectly to BFS levels. Each BFS level represents one second of time, making it natural to process all possible positions at time `t` before moving to time `t+1`.

The probability calculation follows a simple rule: if a vertex has `cnt` unvisited neighbors, the frog has a `1/cnt` chance of jumping to each one. So if we're at vertex `u` with probability `p` and it has `cnt` unvisited neighbors, each neighbor receives probability `p/cnt`.

The tricky part is handling the end conditions:
- If we reach the target at exactly `t` seconds → return the probability
- If we reach the target before `t` seconds but have nowhere else to go (no unvisited neighbors) → the frog stays there, so return the probability
- If we reach the target before `t` seconds but have unvisited neighbors → the frog will jump away, so return `0`

This condition is elegantly captured by checking if `cnt * t == 0` when at the target vertex: either we have no more time (`t=0`) or no more moves (`cnt=0`).

**Learn more about [Tree](/problems/tree_intro), [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The implementation uses BFS with probability tracking to simulate the frog's movement through the [tree](/problems/tree_intro).

**Step 1: Build the [Graph](/problems/graph_intro)**
First, we construct an adjacency list `g` from the edges array. Since the [tree](/problems/tree_intro) is undirected, we add both directions for each edge:
```python
g = defaultdict(list)
for u, v in edges:
    g[u].append(v)
    g[v].append(u)
```

**Step 2: Initialize BFS**
We set up the BFS with:
- Queue `q` containing tuples of `(vertex, probability)`, starting with `[(1, 1.0)]` - the frog starts at vertex 1 with probability 1.0
- Visited array `vis` to track which vertices have been visited, with `vis[1] = True` initially

**Step 3: Level-by-Level BFS**
The outer loop runs while the queue is not empty and `t >= 0`. Each iteration represents one second:
```python
while q and t >= 0:
    for _ in range(len(q)):  # Process all vertices at current time level
        u, p = q.popleft()
```

**Step 4: Calculate Unvisited Neighbors**
For each vertex `u`, we count its unvisited neighbors:
```python
cnt = len(g[u]) - int(u != 1)
```
The subtraction `- int(u != 1)` handles a special case: for non-root vertices, one of their neighbors is their parent (already visited), so we subtract 1. For vertex 1 (the root), all neighbors are children.

**Step 5: Check Target Condition**
If we've reached the target vertex:
```python
if u == target:
    return p if cnt * t == 0 else 0
```
This clever condition `cnt * t == 0` returns the probability `p` if either:
- `t = 0`: We've reached the target at exactly time `t`
- `cnt = 0`: We've reached the target and have no unvisited neighbors (frog stays here)

Otherwise, it returns 0 (the frog will jump away).

**Step 6: Distribute Probability**
If not at the target, we distribute the probability equally among unvisited neighbors:
```python
for v in g[u]:
    if not vis[v]:
        vis[v] = True
        q.append((v, p / cnt))
```
Each unvisited neighbor gets probability `p / cnt`.

**Step 7: Advance Time**
After processing all vertices at the current time level, we decrement `t`:
```python
t -= 1
```

The algorithm returns 0 if we exhaust all time steps without satisfying the target conditions, meaning the frog cannot be at the target vertex after exactly `t` seconds.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a tree with 4 vertices and edges: `[[1,2], [1,3], [3,4]]`, target = 4, t = 2

The tree looks like:
```
    1
   / \
  2   3
      |
      4
```

**Initial Setup:**
- Build adjacency list: `g = {1: [2,3], 2: [1], 3: [1,4], 4: [3]}`
- Queue starts with: `[(1, 1.0)]` 
- Visited: `[False, True, False, False, False]` (index 0 unused, vertex 1 is visited)

**Time t=2 (First Iteration):**
- Process vertex 1 with probability 1.0
- Count unvisited neighbors: vertex 1 has neighbors [2,3], both unvisited
  - `cnt = 2 - 0 = 2` (subtract 0 because u=1 is the root)
- Check if vertex 1 is target (4): No
- Distribute probability: Each neighbor gets 1.0/2 = 0.5
- Mark vertices 2 and 3 as visited
- Queue becomes: `[(2, 0.5), (3, 0.5)]`
- Decrement t to 1

**Time t=1 (Second Iteration):**
- Process vertex 2 with probability 0.5
  - Neighbors: [1] (already visited)
  - `cnt = 1 - 1 = 0` (subtract 1 because u≠1)
  - Check if vertex 2 is target (4): No
  - No unvisited neighbors, so nothing added to queue

- Process vertex 3 with probability 0.5
  - Neighbors: [1, 4], where 1 is visited and 4 is unvisited
  - `cnt = 2 - 1 = 1` (subtract 1 because u≠1)
  - Check if vertex 3 is target (4): No
  - Distribute probability: vertex 4 gets 0.5/1 = 0.5
  - Mark vertex 4 as visited
  - Queue becomes: `[(4, 0.5)]`
- Decrement t to 0

**Time t=0 (Third Iteration):**
- Process vertex 4 with probability 0.5
- Neighbors: [3] (already visited)
- `cnt = 1 - 1 = 0` (subtract 1 because u≠1)
- Check if vertex 4 is target (4): Yes!
- Check condition `cnt * t == 0`: `0 * 0 == 0` ✓
- Return probability 0.5

**Result:** The probability of the frog being at vertex 4 after exactly 2 seconds is 0.5.

This makes sense intuitively: From vertex 1, the frog has a 50% chance of jumping to vertex 3 (and then must jump to vertex 4), while it has a 50% chance of jumping to vertex 2 (where it gets stuck and never reaches vertex 4).

## Python Solution

```python
from collections import defaultdict, deque
from typing import List

class Solution:
    def frogPosition(
        self, n: int, edges: List[List[int]], t: int, target: int
    ) -> float:
        # Build adjacency list representation of the tree
        graph = defaultdict(list)
        for node_u, node_v in edges:
            graph[node_u].append(node_v)
            graph[node_v].append(node_u)
      
        # Initialize BFS queue with starting position (node 1) and probability 1.0
        queue = deque([(1, 1.0)])
      
        # Track visited nodes to avoid revisiting
        visited = [False] * (n + 1)
        visited[1] = True
      
        # Perform BFS for exactly t seconds
        while queue and t >= 0:
            # Process all nodes at current time step
            level_size = len(queue)
            for _ in range(level_size):
                current_node, probability = queue.popleft()
              
                # Calculate number of unvisited children
                # For root node (1), all neighbors are children
                # For other nodes, subtract 1 to exclude the parent
                unvisited_children_count = len(graph[current_node]) - int(current_node != 1)
              
                # Check if we've reached the target node
                if current_node == target:
                    # Frog stays at target if no unvisited children or no time left
                    # Otherwise, frog must jump away, so probability becomes 0
                    return probability if unvisited_children_count * t == 0 else 0
              
                # Add unvisited neighbors to queue for next time step
                for neighbor in graph[current_node]:
                    if not visited[neighbor]:
                        visited[neighbor] = True
                        # Probability is divided equally among unvisited children
                        queue.append((neighbor, probability / unvisited_children_count))
          
            # Decrement time after processing current level
            t -= 1
      
        # Target not reached within time limit
        return 0
```

## Time and Space Complexity

**Time Complexity:** `O(n)`

The algorithm uses BFS to traverse the tree. In the worst case, we visit each node exactly once. For each node, we perform constant time operations (checking if it's the target, calculating probability, adding children to queue). Since we have `n` nodes and process each node once, the time complexity is `O(n)`.

Additionally, we iterate through all edges once to build the adjacency list, which takes `O(E)` time where `E` is the number of edges. In a tree with `n` nodes, there are exactly `n-1` edges, so this is also `O(n)`.

The outer while loop runs at most `t+1` times (until time runs out), and in each iteration, we process all nodes at the current level. However, the total number of nodes processed across all iterations is still bounded by `n`.

Therefore, the overall time complexity is `O(n)`.

**Space Complexity:** `O(n)`

The space complexity consists of:
- Adjacency list `g`: stores all edges, requiring `O(2E) = O(2(n-1)) = O(n)` space since each edge is stored twice (bidirectional)
- Queue `q`: in the worst case (complete binary tree), can hold up to `O(n/2)` nodes at the widest level, which is `O(n)`
- Visited array `vis`: stores boolean values for `n+1` positions, requiring `O(n)` space

Therefore, the overall space complexity is `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Counting of Unvisited Neighbors**

The most common mistake is incorrectly counting the number of unvisited neighbors when calculating probabilities. Many solutions fail to account for the special case of the root node versus other nodes.

**Incorrect approach:**
```python
# Wrong: Counts all neighbors minus visited ones
unvisited = sum(1 for v in graph[u] if not visited[v])
```

**Why it fails:** For non-root nodes, one of the neighbors is the parent (already visited). Simply counting unvisited neighbors would give 0 for leaf nodes, causing division by zero when calculating probability.

**Correct approach:**
```python
# For root (node 1): all neighbors are unvisited children
# For other nodes: total neighbors - 1 (parent)
unvisited_children = len(graph[node]) - int(node != 1)
```

### 2. **Mishandling the Target Reached Condition**

Another pitfall is incorrectly determining when the frog stays at the target vertex.

**Incorrect approach:**
```python
if current == target and t == 0:
    return probability
```

**Why it fails:** This only checks if we reach the target at exactly time t, but misses the case where the frog reaches the target early with no unvisited neighbors (and thus stays there).

**Correct approach:**
```python
if current == target:
    # Return probability if either:
    # - t = 0 (reached at exact time)
    # - unvisited_children = 0 (stuck at target)
    return probability if unvisited_children * t == 0 else 0
```

### 3. **Processing Queue Levels Incorrectly**

Failing to process the queue level-by-level can lead to incorrect time tracking.

**Incorrect approach:**
```python
while queue and t > 0:
    node, prob = queue.popleft()
    t -= 1  # Wrong: decrements for each node, not each level
```

**Correct approach:**
```python
while queue and t >= 0:
    level_size = len(queue)
    for _ in range(level_size):  # Process entire level
        node, prob = queue.popleft()
        # ... process node
    t -= 1  # Decrement once per level (one second)
```

### 4. **Edge Case: Single Node Tree**

When n = 1 (single node tree), the frog starts at node 1 and has nowhere to jump.

**Potential issue:** Not handling empty edges array properly.

**Solution:** The code handles this naturally since:
- If `edges` is empty, `graph[1]` has no neighbors
- `unvisited_children = 0 - 0 = 0` for node 1
- If target is 1, returns 1.0 (frog stays at node 1)
- If target is not 1, returns 0 (impossible to reach)

