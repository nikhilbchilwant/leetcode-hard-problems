# 2204. Distance to a Cycle in Undirected Graph

> Source: https://algo.monster/liteproblems/2204

## Problem Statement

You have a connected undirected [graph](/problems/graph_intro) with `n` nodes (numbered from `0` to `n - 1`) that contains exactly one cycle. The graph is given as a list of edges where `edges[i] = [node1_i, node2_i]` represents a bidirectional edge between `node1_i` and `node2_i`.

The distance between two nodes is defined as the minimum number of edges needed to travel from one node to the other.

Your task is to find the minimum distance from each node to any node that is part of the cycle. Return an array `answer` of size `n` where `answer[i]` represents the minimum distance from node `i` to the closest node in the cycle.

For example, if a node is already part of the cycle, its distance would be `0`. If a node is directly connected to a node in the cycle (but not in the cycle itself), its distance would be `1`, and so on.

## Intuition

The key insight is recognizing the special structure of this [graph](/problems/graph_intro): it's essentially a tree with one extra edge that creates a cycle. Think of it like a necklace (the cycle) with several chains hanging from it (the tree branches).

If we imagine peeling away the [graph](/problems/graph_intro) layer by layer from the outside, like peeling an onion, we would start with the outermost nodes (leaves) and work our way inward. The leaves are easy to identify - they're nodes with degree 1 (only one connection). These nodes cannot be part of the cycle because cycle nodes must have at least degree 2.

Here's the brilliant observation: if we keep removing leaf nodes and update the degrees of their neighbors, new leaves will emerge. We continue this process until we can't remove any more nodes. What remains? Only the cycle! Because cycle nodes all have degree at least 2 even after removing all external branches.

As we remove nodes, we can track which node each removed node was connected to before removal. This creates a "parent" relationship - each removed node remembers its connection closer to the cycle. This is captured in the array `f[i]` in the solution.

Once we've identified all non-cycle nodes (those that were removed) and their parent relationships, calculating distances becomes straightforward. We process the removed nodes in reverse order (from last removed to first removed). Why reverse? Because:
- The last nodes removed were directly connected to the cycle (distance 1)
- The second-to-last layer was connected to those nodes (distance 2)
- And so on...

Each node's distance is simply its parent's distance plus 1: `ans[i] = ans[f[i]] + 1`. The cycle nodes themselves remain at distance 0 since they were never removed.

This approach elegantly avoids the need to explicitly detect the cycle or run BFS from multiple sources. Instead, it uses the [graph](/problems/graph_intro)'s structure to naturally separate cycle nodes from non-cycle nodes through topological-style processing.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro), [Union Find](/problems/dsu_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

Let's walk through the implementation step by step, following the topological sorting approach described in the reference solution.

**Step 1: Build the Adjacency List**

First, we convert the edge list into an adjacency list representation using a dictionary of sets:
```python
g = defaultdict(set)
for a, b in edges:
    g[a].add(b)
    g[b].add(a)
```
Using sets instead of lists makes it easier to remove edges later and check degrees efficiently.

**Step 2: Initialize Queue with Leaf Nodes**

We find all nodes with degree 1 (leaf nodes) and add them to a queue:
```python
q = deque(i for i in range(n) if len(g[i]) == 1)
```
These are the outermost nodes that definitely aren't part of the cycle.

**Step 3: Layer-by-Layer Removal**

We process nodes from the queue, removing them from the [graph](/problems/graph_intro):
```python
f = [0] * n  # tracks parent relationship
seq = []     # records removal order

while q:
    i = q.popleft()
    seq.append(i)  # record this node was removed

    for j in g[i]:  # for each neighbor
        g[j].remove(i)  # remove edge from neighbor to current node
        f[i] = j        # j is the parent of i (closer to cycle)

        if len(g[j]) == 1:  # if neighbor becomes a leaf
            q.append(j)     # add to queue for removal

    g[i].clear()  # clear all edges from current node
```

The key points here:
- `f[i] = j` records that node `j` is the "parent" of node `i` - the node that's one step closer to the cycle
- When a node's degree drops to 1 after removal, it becomes a new leaf and gets added to the queue
- `seq` maintains the order of removal, which we'll use to calculate distances

**Step 4: Calculate Distances in Reverse**

After removal, nodes still in the [graph](/problems/graph_intro) (with degree ≥ 2) form the cycle. We calculate distances by processing removed nodes in reverse order:
```python
ans = [0] * n  # initialize all distances to 0

for i in seq[::-1]:  # process in reverse order
    ans[i] = ans[f[i]] + 1
```

Why does this work?
- Cycle nodes keep distance 0 (they were never removed, so not in `seq`)
- The last nodes removed were directly connected to cycle nodes, so their distance is `0 + 1 = 1`
- Each earlier removed node is one step further from the cycle than its parent

**Time and Space Complexity:**
- Time: `O(n)` - each node and edge is processed once
- Space: `O(n)` - for the adjacency list, queue, and auxiliary arrays

This approach cleverly avoids explicitly detecting the cycle or running multiple BFS traversals, instead using the [graph](/problems/graph_intro)'s structure to naturally identify cycle nodes and compute distances efficiently.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a graph with 6 nodes and edges: `[[0,1], [1,2], [2,3], [3,1], [3,4], [4,5]]`

```
    0---1---2
        |   |
        +---3---4---5
```

The cycle consists of nodes 1, 2, and 3. Let's trace through the algorithm:

**Step 1: Build adjacency list**
- Node 0: {1}
- Node 1: {0, 2, 3}
- Node 2: {1, 3}
- Node 3: {1, 2, 4}
- Node 4: {3, 5}
- Node 5: {4}

**Step 2: Find initial leaf nodes (degree = 1)**
- Nodes 0 and 5 are leaves (degree 1)
- Queue: [0, 5]

**Step 3: Layer-by-layer removal**

*Round 1: Remove node 0*
- Remove 0 from node 1's adjacency list
- Set f[0] = 1 (node 1 is parent of node 0)
- Node 1 now has degree 2 (still connected to 2 and 3)
- seq = [0]

*Round 2: Remove node 5*
- Remove 5 from node 4's adjacency list
- Set f[5] = 4 (node 4 is parent of node 5)
- Node 4 now has degree 1 (becomes a new leaf!)
- Add node 4 to queue
- seq = [0, 5]

*Round 3: Remove node 4*
- Remove 4 from node 3's adjacency list
- Set f[4] = 3 (node 3 is parent of node 4)
- Node 3 now has degree 2 (still connected to 1 and 2)
- seq = [0, 5, 4]

No more nodes with degree 1, so we stop. Nodes 1, 2, and 3 remain (the cycle).

**Step 4: Calculate distances in reverse order**

Initial state: ans = [0, 0, 0, 0, 0, 0]

Process seq in reverse: [4, 5, 0]

*Process node 4:*
- ans[4] = ans[f[4]] + 1 = ans[3] + 1 = 0 + 1 = 1

*Process node 5:*
- ans[5] = ans[f[5]] + 1 = ans[4] + 1 = 1 + 1 = 2

*Process node 0:*
- ans[0] = ans[f[0]] + 1 = ans[1] + 1 = 0 + 1 = 1

**Final Result:** ans = [1, 0, 0, 0, 1, 2]

This means:
- Nodes 1, 2, 3 (cycle nodes): distance 0
- Nodes 0, 4 (directly connected to cycle): distance 1
- Node 5 (two edges away from cycle): distance 2

The algorithm correctly identified the cycle nodes by process of elimination and calculated the shortest distances without explicitly finding the cycle or running BFS from multiple sources.

## Python Solution

```python
from collections import defaultdict, deque
from typing import List

class Solution:
    def distanceToCycle(self, n: int, edges: List[List[int]]) -> List[int]:
        # Build adjacency list representation of the graph
        graph = defaultdict(set)
        for node_a, node_b in edges:
            graph[node_a].add(node_b)
            graph[node_b].add(node_a)

        # Initialize queue with all leaf nodes (degree 1)
        queue = deque(node for node in range(n) if len(graph[node]) == 1)

        # parent[i] stores the parent of node i in the tree structure after removing leaves
        parent = [0] * n

        # Store the order of nodes removed (leaves to inner nodes)
        removal_order = []

        # Process nodes layer by layer, removing leaves
        while queue:
            current_node = queue.popleft()
            removal_order.append(current_node)

            # Process the single neighbor of this leaf node
            for neighbor in graph[current_node]:
                graph[neighbor].remove(current_node)
                parent[current_node] = neighbor

                # If neighbor becomes a leaf after removal, add to queue
                if len(graph[neighbor]) == 1:
                    queue.append(neighbor)

            # Clear the current node's adjacency list
            graph[current_node].clear()

        # Calculate distances from each node to the cycle
        # Nodes remaining in graph are part of the cycle (distance 0)
        distances = [0] * n

        # Process removed nodes in reverse order to calculate distances
        # Each removed node's distance is its parent's distance + 1
        for node in reversed(removal_order):
            distances[node] = distances[parent[node]] + 1

        return distances
```

## Time and Space Complexity

**Time Complexity: `O(n)`**

The algorithm performs the following operations:
- Building the adjacency list from edges: `O(n)` since there are exactly `n` edges in a tree with one extra edge forming a cycle
- Initial queue construction to find all nodes with degree 1: `O(n)`
- BFS-like traversal where each node is processed exactly once: `O(n)`
  - For each node, we iterate through its neighbors, but since each edge is processed at most twice (once from each endpoint), the total work across all iterations is `O(n)`
- Reversing the sequence and calculating distances: `O(n)`

Overall time complexity: `O(n) + O(n) + O(n) + O(n) = O(n)`

**Space Complexity: `O(n)`**

The algorithm uses:
- Graph adjacency list `g`: `O(n)` to store all edges
- Queue `q`: `O(n)` in the worst case
- Array `f`: `O(n)` to store parent relationships
- Array `seq`: `O(n)` to store the traversal sequence
- Array `ans`: `O(n)` for the final answer

Overall space complexity: `O(n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrectly Handling the Parent Assignment

**The Problem:**
A common mistake is assigning the parent relationship incorrectly or at the wrong time. Some might try to assign `parent[j] = i` instead of `parent[i] = j`, or attempt to track parents for all nodes including cycle nodes.

**Why It Happens:**
```python
# Incorrect approach - assigning parent in wrong direction
for neighbor in graph[current_node]:
    parent[neighbor] = current_node  # WRONG!
```

The confusion arises because in typical tree traversals, we often think of the parent as the node we came from. However, in this algorithm, we're removing leaves and moving inward, so the "parent" is actually the node closer to the cycle.

**The Solution:**
```python
# Correct approach - parent is the neighbor (closer to cycle)
for neighbor in graph[current_node]:
    graph[neighbor].remove(current_node)
    parent[current_node] = neighbor  # current_node's parent is its neighbor
```

### Pitfall 2: Not Properly Removing Edges from Both Directions

**The Problem:**
Forgetting to remove the edge from both directions in the undirected graph, or clearing the current node's adjacency list too early.

**Why It Happens:**
```python
# Incorrect - only removing from one direction
while queue:
    current_node = queue.popleft()
    graph[current_node].clear()  # Clearing too early!

    for neighbor in graph[current_node]:  # Now this loop won't execute
        # ... rest of logic
```

**The Solution:**
```python
# Correct order of operations
while queue:
    current_node = queue.popleft()
    removal_order.append(current_node)

    for neighbor in graph[current_node]:
        graph[neighbor].remove(current_node)  # Remove from neighbor first
        parent[current_node] = neighbor

        if len(graph[neighbor]) == 1:
            queue.append(neighbor)

    graph[current_node].clear()  # Clear after processing all neighbors
```

### Pitfall 3: Using Lists Instead of Sets for Adjacency List

**The Problem:**
Using lists for the adjacency list makes edge removal inefficient and degree checking cumbersome.

**Why It Happens:**
```python
# Inefficient approach using lists
graph = defaultdict(list)
for a, b in edges:
    graph[a].append(b)
    graph[b].append(a)

# Later, removing an edge becomes O(n) operation
graph[neighbor].remove(current_node)  # O(n) for list
```

**The Solution:**
```python
# Efficient approach using sets
graph = defaultdict(set)
for a, b in edges:
    graph[a].add(b)
    graph[b].add(a)

# Edge removal is O(1) with sets
graph[neighbor].remove(current_node)  # O(1) for set
```

### Pitfall 4: Misunderstanding the Distance Calculation

**The Problem:**
Attempting to calculate distances during the forward pass (while removing nodes) instead of in reverse order.

**Why It Happens:**
```python
# Incorrect - trying to calculate distance while removing
while queue:
    current_node = queue.popleft()
    for neighbor in graph[current_node]:
        distances[current_node] = distances[neighbor] + 1  # neighbor's distance unknown!
```

At this point, we haven't determined which nodes are in the cycle yet, so we can't calculate accurate distances.

**The Solution:**
```python
# Correct - calculate distances in reverse after identifying cycle nodes
for node in reversed(removal_order):
    distances[node] = distances[parent[node]] + 1
```

By processing in reverse order, we ensure that a node's parent's distance is already calculated before we use it, since parents are always closer to the cycle and were removed later (or not at all if they're in the cycle).

