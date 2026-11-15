# 3383. Minimum Runes to Add to Cast Spell

> Source: https://algo.monster/liteproblems/3383

## Problem Statement

Alice needs to cast a magic spell that involves `n` focus points (numbered from 0 to n-1). Some focus points contain magic crystals that serve as energy sources, specified in the `crystals` array. Focus points are connected by directed runes that allow magic to flow from one point to another, represented by the `flowFrom` and `flowTo` arrays where the i-th rune creates a directed edge from `flowFrom[i]` to `flowTo[i]`.

For the spell to work properly, every focus point must satisfy at least one of these conditions:
1. It contains a magic crystal (is listed in the `crystals` array)
2. It receives magic flow from at least one other focus point (has at least one incoming edge)

The problem asks you to find the minimum number of new directed runes (edges) that Alice must add to ensure all focus points meet one of these requirements.

For example, if a focus point doesn't have a crystal and doesn't receive any incoming magic flow, you need to add a rune that directs magic to it from some other point. The goal is to minimize the total number of runes added while ensuring every focus point is either a crystal source or receives magic from somewhere else.

## Intuition

Let's think about what makes a focus point "valid" for the spell. A focus point is valid if it either has a crystal (energy source) or receives magic from another point (has incoming flow). The key insight is that crystal points are our starting sources - they're automatically valid and can spread their magic to other points through directed runes.

First, we can identify all points that are already valid by starting from crystal points and following the directed runes to see which points can be reached. Any point reachable from a crystal point will receive magic flow, making it valid. We can use BFS or DFS from all crystal points to mark these reachable nodes.

After this marking phase, we'll have some focus points that are still invalid - they don't have crystals and can't be reached from any crystal point through existing runes. These unreachable points form one or more disconnected components in our [graph](/problems/graph_intro).

Here's the critical observation: for each disconnected component of invalid points, we only need to add **one** rune to make the entire component valid. If we add a rune pointing to any node in the component, that node becomes valid (it now receives flow), and from there, the existing runes within the component can propagate validity to other nodes.

But which node in each component should receive the new rune? The trick is to process nodes in reverse topological order. By using DFS to build a topological ordering of the unreachable nodes and then processing them in reverse, we ensure that when we add a rune to make one node valid, we can propagate that validity through the component as efficiently as possible.

The algorithm becomes:
1. Mark all nodes reachable from crystals as valid
2. Find all invalid nodes and perform DFS to get their topological order
3. Process invalid nodes in reverse topological order
4. For each still-invalid node encountered, add one rune to it (incrementing our answer) and mark all nodes reachable from it as valid

This greedy approach ensures we add the minimum number of runes because we're maximizing the coverage of each added rune by choosing roots that can validate entire components.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro), [Union Find](/problems/dsu_intro), [Graph](/problems/graph_intro) and [Topological Sort](/problems/topo_intro) patterns.**

## Solution Approach

The implementation follows our intuition with three main phases: marking reachable nodes, finding unreachable components, and adding minimum runes.

**Phase 1: Build the [graph](/problems/graph_intro) and mark nodes reachable from crystals**

First, we construct an adjacency list representation of the directed [graph](/problems/graph_intro):
```python
g = [[] for _ in range(n)]
for a, b in zip(flowFrom, flowTo):
    g[a].append(b)
```

Then we mark all crystal nodes and perform BFS to find all nodes reachable from them:
```python
vis = [0] * n  # 0: unvisited, 1: reachable from crystals, 2: visited in DFS
for x in crystals:
    vis[x] = 1
bfs(deque(crystals))
```

The `bfs` function explores all nodes reachable from the initial queue of crystal nodes, marking them with `vis[b] = 1`:
```python
def bfs(q: Deque[int]):
    while q:
        a = q.popleft()
        for b in g[a]:
            if vis[b] == 1:
                continue
            vis[b] = 1
            q.append(b)
```

**Phase 2: Find topological ordering of unreachable nodes**

For all nodes that are still unvisited (`vis[i] == 0`), we perform DFS to build a topological ordering:
```python
seq = []
for i in range(n):
    if vis[i] == 0:
        dfs(i)
seq.reverse()
```

The `dfs` function marks nodes with `vis[a] = 2` and appends them to `seq` in post-order:
```python
def dfs(a: int):
    vis[a] = 2
    for b in g[a]:
        if vis[b] > 0:
            continue
        dfs(b)
    seq.append(a)
```

By reversing `seq`, we get the nodes in reverse topological order, ensuring that when we process a node, all nodes it can reach come later in the sequence.

**Phase 3: Add minimum runes**

We iterate through the nodes in reverse topological order. For each node still marked as unreachable (`vis[i] == 2`), we:
1. Add one rune pointing to it (increment `ans`)
2. Mark it and all nodes reachable from it as valid using BFS

```python
ans = 0
for i in seq:
    if vis[i] == 2:
        q = deque([i])
        vis[i] = 1
        bfs(q)
        ans += 1
```

This greedy approach ensures minimum runes because:
- Each added rune makes an entire component valid through the BFS propagation
- Processing in reverse topological order maximizes the coverage of each rune
- We only add a rune when we encounter a component root that hasn't been made valid yet

The time complexity is `O(n + m)` where `m` is the number of edges, as we perform [graph](/problems/graph_intro) traversals a constant number of times. The space complexity is `O(n + m)` for storing the graph and auxiliary data structures.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Example Setup:**
- `n = 6` focus points (numbered 0-5)
- `crystals = [0]` (only point 0 has a crystal)
- `flowFrom = [0, 1, 3]` and `flowTo = [1, 2, 4]`
  - This creates edges: 0→1, 1→2, 3→4

**Initial Graph State:**
```
Crystal point: [0]
Edges: 0 → 1 → 2
       3 → 4
       5 (isolated)
```

**Phase 1: Mark nodes reachable from crystals**

Starting from crystal point 0, we perform BFS:
- Visit 0 (crystal), mark as valid (vis[0] = 1)
- From 0, reach 1, mark as valid (vis[1] = 1)
- From 1, reach 2, mark as valid (vis[2] = 1)

After Phase 1: `vis = [1, 1, 1, 0, 0, 0]`
- Points 0, 1, 2 are valid (reachable from crystal)
- Points 3, 4, 5 are still invalid

**Phase 2: Find topological ordering of unreachable nodes**

Now we perform DFS on unreachable nodes (3, 4, 5):
- DFS from 3: visit 3, then 4, mark both with vis=2
  - Post-order adds to seq: [4, 3]
- DFS from 5: visit 5, mark with vis=2
  - Post-order adds to seq: [4, 3, 5]

Reverse the sequence: `seq = [5, 3, 4]`

After Phase 2: `vis = [1, 1, 1, 2, 2, 2]`

**Phase 3: Add minimum runes**

Process nodes in order [5, 3, 4]:

1. **Process node 5:** vis[5] = 2 (still invalid)
   - Add a rune pointing to 5 (ans = 1)
   - Mark 5 as valid (vis[5] = 1)
   - BFS from 5: no outgoing edges, so only 5 is affected

2. **Process node 3:** vis[3] = 2 (still invalid)
   - Add a rune pointing to 3 (ans = 2)
   - Mark 3 as valid (vis[3] = 1)
   - BFS from 3: reach 4, mark it as valid (vis[4] = 1)

3. **Process node 4:** vis[4] = 1 (already valid from step 2)
   - Skip, no rune needed

**Final Result:**
- We added 2 runes total
- All nodes are now valid:
  - 0: has crystal
  - 1, 2: reachable from crystal point 0
  - 3: receives flow from new rune
  - 4: reachable from 3
  - 5: receives flow from new rune

The algorithm correctly identified that we needed one rune for the component {3, 4} and one rune for the isolated node {5}, giving us the minimum of 2 additional runes.

## Python Solution

```python
from typing import List
from collections import deque

class Solution:
    def minRunesToAdd(
        self, n: int, crystals: List[int], flowFrom: List[int], flowTo: List[int]
    ) -> int:
        def bfs(queue: deque[int]) -> None:
            """
            Breadth-first search to mark all nodes reachable from the queue.
            Updates visited array to mark nodes as reachable (value 1).
            """
            while queue:
                current_node = queue.popleft()
                for neighbor in adjacency_list[current_node]:
                    if visited[neighbor] == 1:
                        continue
                    visited[neighbor] = 1
                    queue.append(neighbor)

        def dfs(node: int) -> None:
            """
            Depth-first search for topological sorting.
            Marks nodes as processed (value 2) and builds reverse topological order.
            """
            visited[node] = 2
            for neighbor in adjacency_list[node]:
                if visited[neighbor] > 0:
                    continue
                dfs(neighbor)
            topological_order.append(node)

        # Build adjacency list representation of the directed graph
        adjacency_list = [[] for _ in range(n)]
        for source, destination in zip(flowFrom, flowTo):
            adjacency_list[source].append(destination)

        # Initialize queue with crystal nodes and mark them as visited
        queue = deque(crystals)
        visited = [0] * n  # 0: unvisited, 1: reachable from crystals, 2: processed in DFS
        for crystal_node in crystals:
            visited[crystal_node] = 1

        # Find all nodes reachable from initial crystals
        bfs(queue)

        # Perform DFS on unreachable nodes to get topological ordering
        topological_order = []
        for node in range(n):
            if visited[node] == 0:
                dfs(node)

        # Reverse to get correct topological order (from sources to sinks)
        topological_order.reverse()

        # Count minimum runs needed by processing nodes in topological order
        min_runs = 0
        for node in topological_order:
            if visited[node] == 2:  # Node was unreachable and needs a crystal
                queue = deque([node])
                visited[node] = 1
                bfs(queue)  # Mark all nodes reachable from this new crystal
                min_runs += 1

        return min_runs
```

## Time and Space Complexity

**Time Complexity:** `O(n + m)` where `n` is the number of nodes and `m` is the number of edges (size of flowFrom/flowTo arrays).

- Building the adjacency list `g` takes `O(m)` time
- Initial BFS from crystal nodes visits each node at most once and traverses each edge at most once: `O(n + m)`
- DFS to find topological ordering visits each unvisited node exactly once and each edge from unvisited nodes once: `O(n + m)`
- Processing nodes in topological order: each node can trigger at most one BFS, and across all BFS calls, each node is visited at most once and each edge is traversed at most once: `O(n + m)`
- Overall: `O(m) + O(n + m) + O(n + m) + O(n + m) = O(n + m)`

**Space Complexity:** `O(n + m)`

- Adjacency list `g` stores all edges: `O(m)`
- Visited array `vis`: `O(n)`
- Queue `q` can contain at most `n` nodes: `O(n)`
- Sequence array `seq` stores at most `n` nodes: `O(n)`
- Recursion stack for DFS can go up to `O(n)` depth in worst case
- Overall: `O(m) + O(n) + O(n) + O(n) + O(n) = O(n + m)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly handling strongly connected components (SCCs)**

A critical pitfall is assuming that adding a single rune to any node in an unreachable strongly connected component will satisfy all nodes in that component. Consider this scenario:

- Nodes A, B, C form a cycle: A→B→C→A
- None of them have crystals or incoming edges from crystal-reachable nodes

**Wrong assumption**: Adding a rune to node A will make B and C valid since A can reach them.

**The problem**: This creates a logical contradiction. For A to provide magic to B and C, A itself needs to have magic. But A only gets magic from the rune we're adding to it. However, the rune pointing to A doesn't automatically give A the ability to be a magic source - it just means A receives magic. The cycle doesn't have an actual magic source.

**Solution**: The code correctly handles this by:
1. Using BFS after marking a node as reachable (setting `visited[node] = 1`)
2. This simulates that once we add a rune to a node, it becomes "powered" and can propagate magic through its outgoing edges
3. The topological ordering ensures we process SCCs efficiently

### 2. **Not processing nodes in the correct order**

**Pitfall**: Processing unreachable nodes in arbitrary order might lead to suboptimal solutions.

Consider this graph:
- Node A → Node B → Node C (all unreachable from crystals)
- If we process B first and add a rune to it, then process A and add another rune to it, we use 2 runes
- But if we process A first, adding a rune to A would make B and C reachable through propagation, requiring only 1 rune

**Solution**: The code uses reverse topological ordering (`topological_order.reverse()`), which ensures:
- We process "upstream" nodes before "downstream" nodes
- When we add a rune to a node, we maximize the number of other nodes that become reachable
- This greedy approach guarantees the minimum number of runes

### 3. **Confusing the visited states**

**Pitfall**: Using a simple boolean visited array can lead to incorrect results because nodes have three distinct states:
1. Reachable from crystals (satisfied, no rune needed)
2. Unreachable but processed in DFS (needs consideration)
3. Completely unvisited

**Wrong approach**:
```python
visited = [False] * n  # Binary state - loses information
```

**Solution**: The code uses a tri-state system:
```python
visited = [0] * n  # 0: unvisited, 1: reachable from crystals, 2: processed in DFS
```

This allows distinguishing between nodes that are already satisfied (value 1) and those that still need runes (value 2) after the DFS phase.

### 4. **Forgetting to propagate reachability after adding a rune**

**Pitfall**: Simply counting unreachable components without considering that adding a rune to one node can satisfy many others through existing edges.

**Wrong approach**:
```python
# Incorrect: just counting unreachable nodes
for node in range(n):
    if not reachable[node]:
        min_runs += 1
```

**Solution**: After conceptually adding a rune to a node, perform BFS to mark all nodes reachable from it:
```python
if visited[node] == 2:
    queue = deque([node])
    visited[node] = 1
    bfs(queue)  # Propagate reachability through existing edges
    min_runs += 1
```

This ensures that one rune can satisfy multiple nodes if they're connected, minimizing the total count.

