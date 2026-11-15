# 1192. Critical Connections in a Network

> Source: https://algo.monster/liteproblems/1192

## Problem Statement

You are given `n` servers numbered from `0` to `n - 1` that are connected by undirected server-to-server connections. The connections are represented as `connections[i] = [ai, bi]`, which means there is a direct connection between server `ai` and server `bi`. Through this network, any server can reach any other server either directly or indirectly through other servers.

A **critical connection** is defined as a connection that, if removed from the network, would cause some servers to become unreachable from other servers. In other words, removing this connection would break the network into disconnected components.

Your task is to find and return all critical connections in the network. You can return them in any order.

For example, if you have servers connected like a chain: `0 -- 1 -- 2`, the connection between servers `0` and `1` is critical because removing it would leave server `0` isolated from server `2`. Similarly, the connection between `1` and `2` is also critical.

The problem essentially asks you to identify all the "bridges" in the network [graph](/problems/graph_intro) - edges whose removal would increase the number of connected components in the graph.

## Intuition

To find critical connections (bridges), we need to identify edges whose removal would disconnect the [graph](/problems/graph_intro). The key insight is that a connection is critical if there's no alternative path between its endpoints that doesn't use that connection itself.

Think of it this way: imagine you're exploring a network of caves connected by tunnels. As you explore, you mark each cave with the time you first discovered it. Now, for each tunnel, you want to know: "If this tunnel collapses, can I still reach the other side through a different route?"

This leads us to track two important values for each node:
1. **Discovery time (`dfn`)**: When we first visit this node during our exploration
2. **Low value (`low`)**: The earliest discovered node we can reach from this node and its descendants

The crucial observation is: a connection from node `a` to node `b` is critical if and only if there's no way to reach `a` or any node discovered before `a` from `b` without using the `a-b` connection itself. Mathematically, this means `low[b] > dfn[a]`.

Why does this work? If `low[b] > dfn[a]`, it means that from `b` and all its descendants, we cannot find a "back edge" that leads to `a` or any ancestor of `a`. This indicates that the `a-b` edge is the only connection between the component containing `b` and the component containing `a`.

The Tarjan algorithm elegantly implements this idea using DFS:
- We traverse the [graph](/problems/graph_intro), assigning discovery times to nodes
- For each node, we explore its neighbors
- After exploring all descendants of a node, we update its `low` value based on what its descendants could reach
- If a child's `low` value is greater than the parent's discovery time, we've found a bridge

This single-pass DFS approach efficiently identifies all bridges in `O(V + E)` time, where `V` is the number of vertices and `E` is the number of edges.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Graph](/problems/graph_intro) and [Biconnected Component](/problems/number_of_connected_components) patterns.**

## Solution Approach

The implementation uses the Tarjan algorithm to find all bridges in the [graph](/problems/graph_intro). Let's walk through the key components:

**1. [Graph](/problems/graph_intro) Representation**

First, we build an adjacency list representation of the [graph](/problems/graph_intro):
```python
g = [[] for _ in range(n)]
for a, b in connections:
    g[a].append(b)
    g[b].append(a)
```
This allows efficient traversal of neighbors during DFS.

**2. Core Data Structures**

- `dfn[i]`: Discovery time of node `i` - when it was first visited during DFS
- `low[i]`: The lowest discovery time reachable from node `i` through its subtree
- `now`: A counter to assign unique discovery times to nodes
- `ans`: List to store the critical connections (bridges)

**3. The Tarjan Function**

The recursive `tarjan(a, fa)` function performs DFS where:
- `a` is the current node being visited
- `fa` is the parent node (to avoid going back to the parent in undirected [graph](/problems/graph_intro))

Here's how it works step by step:

```python
def tarjan(a: int, fa: int):
    nonlocal now
    now += 1
    dfn[a] = low[a] = now  # Initialize both values to current timestamp
```

Initially, both `dfn[a]` and `low[a]` are set to the current timestamp.

**4. Exploring Neighbors**

For each neighbor `b` of node `a`:
```python
for b in g[a]:
    if b == fa:
        continue  # Skip the parent edge
```

**5. Processing Unvisited Nodes**

If neighbor `b` hasn't been visited (`dfn[b] == 0`):
```python
if not dfn[b]:
    tarjan(b, a)  # Recursively visit
    low[a] = min(low[a], low[b])  # Update low value
    if low[b] > dfn[a]:  # Bridge condition
        ans.append([a, b])
```

After exploring `b`'s subtree, we update `low[a]` to reflect any earlier node reachable through `b`. If `low[b] > dfn[a]`, it means `b` cannot reach `a` or any ancestor of `a` without using the `a-b` edge, making it a bridge.

**6. Processing Visited Nodes (Back Edges)**

If neighbor `b` has been visited:
```python
else:
    low[a] = min(low[a], dfn[b])
```

This handles back edges - edges that connect to ancestors in the DFS tree. We update `low[a]` with `dfn[b]` to record that we can reach node `b` from `a`.

**7. Starting the Algorithm**

```python
tarjan(0, -1)  # Start DFS from node 0 with no parent
```

The algorithm starts from any node (here node 0) since the [graph](/problems/graph_intro) is connected. Using `-1` as the parent ensures we don't skip any edge from the starting node.

**Time Complexity**: `O(V + E)` where `V` is the number of vertices and `E` is the number of edges, as we visit each node once and examine each edge twice (once from each endpoint).

**Space Complexity**: `O(V)` for the `dfn`, `low` arrays and the recursion stack in the worst case.

## Example Walkthrough

Let's walk through a small example to illustrate how the Tarjan algorithm finds critical connections.

**Example Graph:**
```
Servers: 0, 1, 2, 3
Connections: [[0,1], [1,2], [2,3], [3,1]]

Visual representation:
    0---1---2
        |   |
        +---3
```

**Step-by-step execution:**

**Initial Setup:**
- Build adjacency list: `g = [[1], [0,2,3], [1,3], [2,1]]`
- Initialize: `dfn = [0,0,0,0]`, `low = [0,0,0,0]`, `now = 0`, `ans = []`

**Step 1: Start DFS from node 0**
- Call `tarjan(0, -1)`
- `now = 1`, `dfn[0] = 1`, `low[0] = 1`
- Explore neighbor 1

**Step 2: Visit node 1 from node 0**
- Call `tarjan(1, 0)`
- `now = 2`, `dfn[1] = 2`, `low[1] = 2`
- Neighbors of 1: [0, 2, 3]
- Skip 0 (parent)
- Explore neighbor 2

**Step 3: Visit node 2 from node 1**
- Call `tarjan(2, 1)`
- `now = 3`, `dfn[2] = 3`, `low[2] = 3`
- Neighbors of 2: [1, 3]
- Skip 1 (parent)
- Explore neighbor 3

**Step 4: Visit node 3 from node 2**
- Call `tarjan(3, 2)`
- `now = 4`, `dfn[3] = 4`, `low[3] = 4`
- Neighbors of 3: [2, 1]
- Skip 2 (parent)
- Node 1 is already visited: `low[3] = min(4, dfn[1]) = min(4, 2) = 2`

**Step 5: Backtrack to node 2**
- Update: `low[2] = min(low[2], low[3]) = min(3, 2) = 2`
- Check bridge: `low[3] > dfn[2]`? → `2 > 3`? → No, not a bridge

**Step 6: Backtrack to node 1**
- Update: `low[1] = min(low[1], low[2]) = min(2, 2) = 2`
- Check bridge: `low[2] > dfn[1]`? → `2 > 2`? → No, not a bridge
- Continue with neighbor 3 (already visited): `low[1] = min(2, dfn[3]) = min(2, 4) = 2`

**Step 7: Backtrack to node 0**
- Update: `low[0] = min(low[0], low[1]) = min(1, 2) = 1`
- Check bridge: `low[1] > dfn[0]`? → `2 > 1`? → **Yes, this is a bridge!**
- Add `[0, 1]` to answer

**Final Result:**
- `dfn = [1, 2, 3, 4]`
- `low = [1, 2, 2, 2]`
- `ans = [[0, 1]]`

The connection `[0, 1]` is critical because removing it would isolate node 0 from the rest of the network. The cycle formed by nodes 1, 2, and 3 means those connections are not critical - there are alternative paths between those nodes.

**Key Insight:** The bridge condition `low[1] > dfn[0]` tells us that from node 1 and its descendants, we cannot reach node 0 or any node discovered before node 0 without using the 0-1 edge. This makes the 0-1 connection critical to the network's connectivity.

## Python Solution

```python
class Solution:
    def criticalConnections(
        self, n: int, connections: List[List[int]]
    ) -> List[List[int]]:
        """
        Find all critical connections (bridges) in an undirected graph.
        A critical connection is an edge that, if removed, increases the number of connected components.
      
        Args:
            n: Number of nodes in the graph (nodes are labeled from 0 to n-1)
            connections: List of edges where each edge is [node1, node2]
      
        Returns:
            List of critical connections (bridges)
        """
      
        def tarjan_dfs(current_node: int, parent_node: int) -> None:
            """
            Perform DFS using Tarjan's algorithm to find bridges.
          
            Args:
                current_node: Current node being visited
                parent_node: Parent of the current node in DFS tree (-1 for root)
            """
            nonlocal timestamp
          
            # Assign discovery time and initialize low-link value
            timestamp += 1
            discovery_time[current_node] = timestamp
            low_link[current_node] = timestamp
          
            # Explore all neighbors
            for neighbor in adjacency_list[current_node]:
                # Skip the edge back to parent (avoid going back on the same edge)
                if neighbor == parent_node:
                    continue
              
                if discovery_time[neighbor] == 0:  # Neighbor not yet visited
                    # Recursively visit the neighbor
                    tarjan_dfs(neighbor, current_node)
                  
                    # Update low-link value after returning from DFS
                    low_link[current_node] = min(
                        low_link[current_node], 
                        low_link[neighbor]
                    )
                  
                    # Check if the edge is a bridge
                    # If low_link[neighbor] > discovery_time[current], then neighbor
                    # cannot reach current or any ancestor without using edge (current, neighbor)
                    if low_link[neighbor] > discovery_time[current_node]:
                        critical_edges.append([current_node, neighbor])
                else:
                    # Neighbor already visited (back edge)
                    # Update low-link value using discovery time of neighbor
                    low_link[current_node] = min(
                        low_link[current_node], 
                        discovery_time[neighbor]
                    )
      
        # Build adjacency list representation of the graph
        adjacency_list = [[] for _ in range(n)]
        for node_a, node_b in connections:
            adjacency_list[node_a].append(node_b)
            adjacency_list[node_b].append(node_a)
      
        # Initialize arrays for Tarjan's algorithm
        discovery_time = [0] * n  # Discovery time for each node (0 means unvisited)
        low_link = [0] * n        # Lowest discovery time reachable from subtree
        timestamp = 0             # Global timestamp counter
        critical_edges = []       # Result list to store bridges
      
        # Start DFS from node 0 (assuming graph is connected)
        # Use -1 as parent for the root node
        tarjan_dfs(0, -1)
      
        return critical_edges
```

## Time and Space Complexity

**Time Complexity:** `O(V + E)` where `V` is the number of vertices (nodes) and `E` is the number of edges (connections).

- Building the adjacency list takes `O(E)` time as we iterate through all connections once
- The Tarjan algorithm performs a DFS traversal visiting each vertex exactly once, taking `O(V)` time
- For each vertex, we explore all its adjacent edges. Since the graph is undirected, each edge is stored twice in the adjacency list, so we process `2E` edge traversals in total, which is `O(E)`
- Overall: `O(V + E)`

**Space Complexity:** `O(V + E)`

- The adjacency list `g` stores all edges, requiring `O(E)` space (each edge appears twice in an undirected graph)
- The `dfn` array uses `O(V)` space to store discovery times
- The `low` array uses `O(V)` space to store low-link values
- The recursion stack depth can go up to `O(V)` in the worst case (e.g., a linear chain of nodes)
- The `ans` list can contain at most `O(V)` critical connections (bridges)
- Overall: `O(V + E)` dominated by the adjacency list storage

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Handling Parent Edges in Undirected Graphs**

**The Pitfall:**
One of the most common mistakes is not properly handling the parent edge when traversing an undirected graph. Since each edge appears twice in the adjacency list (once for each direction), we need to avoid immediately traversing back to the parent node. However, a naive implementation might skip ALL edges back to the parent, which is incorrect when there are multiple edges between the same two nodes (parallel edges).

**Problematic Code:**
```python
for neighbor in adjacency_list[current_node]:
    if neighbor == parent_node:  # This skips ALL edges to parent
        continue
```

**The Issue:**
If there are multiple connections between the same two nodes (e.g., `[[0,1], [0,1]]`), the above code would skip both edges when going from node 1 back to node 0, potentially missing that neither edge is critical when parallel edges exist.

**The Solution:**
Track the specific edge used to reach the current node, not just the parent node:

```python
def tarjan_dfs(current_node: int, parent_edge_id: int) -> None:
    nonlocal timestamp
    timestamp += 1
    discovery_time[current_node] = timestamp
    low_link[current_node] = timestamp
  
    for edge_id in adjacency_list[current_node]:
        neighbor = edges[edge_id][0] if edges[edge_id][1] == current_node else edges[edge_id][1]
      
        # Skip only the specific edge we came from
        if edge_id == parent_edge_id:
            continue
          
        if discovery_time[neighbor] == 0:
            tarjan_dfs(neighbor, edge_id)
            low_link[current_node] = min(low_link[current_node], low_link[neighbor])
            if low_link[neighbor] > discovery_time[current_node]:
                critical_edges.append(edges[edge_id])
        else:
            low_link[current_node] = min(low_link[current_node], discovery_time[neighbor])

# Build adjacency list with edge IDs
adjacency_list = [[] for _ in range(n)]
edges = connections
for i, (node_a, node_b) in enumerate(edges):
    adjacency_list[node_a].append(i)
    adjacency_list[node_b].append(i)
```

### 2. **Updating Low-Link Values with Wrong Values for Back Edges**

**The Pitfall:**
When encountering a back edge (an edge to an already visited node that's not the parent), developers sometimes incorrectly update the low-link value using `low[neighbor]` instead of `dfn[neighbor]`.

**Problematic Code:**
```python
else:  # Back edge case
    low[current] = min(low[current], low[neighbor])  # WRONG!
```

**The Issue:**
Using `low[neighbor]` can propagate incorrect values through the graph. The low-link value should represent the earliest discovered node reachable from the current subtree. For back edges, we should use the discovery time of the neighbor, not its low-link value.

**The Solution:**
```python
else:  # Back edge case
    low[current] = min(low[current], dfn[neighbor])  # CORRECT
```

### 3. **Handling Disconnected Graphs**

**The Pitfall:**
The code assumes the graph is connected and starts DFS from only node 0. If the graph has multiple connected components, bridges in other components won't be found.

**The Issue:**
Starting DFS from only one node means unvisited components will be ignored entirely.

**The Solution:**
Iterate through all nodes and start DFS from unvisited ones:

```python
def criticalConnections(self, n: int, connections: List[List[int]]) -> List[List[int]]:
    # ... (setup code remains the same)
  
    # Handle potentially disconnected graphs
    for start_node in range(n):
        if discovery_time[start_node] == 0:
            tarjan_dfs(start_node, -1)
  
    return critical_edges
```

This ensures all components are explored and all bridges are found, even in disconnected graphs.

