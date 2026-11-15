# 2493. Divide Nodes Into the Maximum Number of Groups

> Source: https://algo.monster/liteproblems/2493

## Problem Statement

You have an undirected [graph](/problems/graph_intro) with `n` nodes labeled from `1` to `n`. The graph is defined by a list of edges where each edge `[ai, bi]` represents a bidirectional connection between nodes `ai` and `bi`. Note that the graph may be disconnected (not all nodes are reachable from each other).

Your task is to divide all nodes into `m` groups (numbered from `1` to `m`) such that:

1. **Every node belongs to exactly one group** - Each of the `n` nodes must be assigned to one and only one group.

2. **Adjacent nodes must be in consecutive groups** - If two nodes are connected by an edge, and one node is in group `x` while the other is in group `y`, then `|y - x| = 1`. This means connected nodes must be placed in groups whose numbers differ by exactly 1.

The goal is to find the **maximum number of groups** (`m`) that you can create while satisfying these conditions. If it's impossible to group the nodes according to these rules, return `-1`.

For example, if you have a simple path [graph](/problems/graph_intro) with 4 nodes connected as `1-2-3-4`, you could assign:
- Node 1 to group 1
- Node 2 to group 2  
- Node 3 to group 3
- Node 4 to group 4

This would give you a maximum of 4 groups. However, if the [graph](/problems/graph_intro) contains an odd-length cycle, it would be impossible to satisfy the constraints (similar to bipartite graph coloring), and you would return `-1`.

Since the [graph](/problems/graph_intro) can be disconnected, you need to handle each connected component separately and sum up the maximum groups needed for each component.

## Intuition

The key insight is to think of this problem as a **[graph](/problems/graph_intro) coloring problem with a special constraint**: adjacent nodes must have colors (group numbers) that differ by exactly 1.

Let's build the intuition step by step:

1. **Why BFS?** When we assign group numbers starting from any node, we want adjacent nodes to have consecutive group numbers. This is exactly like finding distances in an unweighted [graph](/problems/graph_intro) - nodes at distance 1 get group 2, nodes at distance 2 get group 3, and so on. BFS naturally gives us these levels.

2. **Why enumerate starting points?** Different starting points in the same connected component can give different maximum depths. Consider a path [graph](/problems/graph_intro) `1-2-3`. If we start from node 2 (middle), we get 2 groups maximum. But if we start from node 1 (end), we get 3 groups. So we need to try all possible starting points to maximize the number of groups.

3. **How to detect impossibility?** During BFS traversal, if we encounter a node that's already been visited and its assigned group number doesn't match what we expect (differ by 1 from its neighbor), then it's impossible to satisfy the constraints. This happens with **odd-length cycles** - imagine a triangle where node A is in group 1, B in group 2, C in group 3, but C must also be adjacent to A (group 1), requiring C to be in group 2, which is a contradiction.

4. **Why track by connected components?** Since the [graph](/problems/graph_intro) can be disconnected, each connected component can be grouped independently. The total number of groups is the sum of maximum groups from each component. We use the smallest node in each component as a representative to avoid counting the same component multiple times.

5. **The overall strategy:** For each node as a potential starting point, run BFS to assign group numbers based on distance. Track the maximum depth (number of groups) for each connected component. If at any point we find conflicting group assignments, return `-1`. Otherwise, sum up the maximum groups from all components.

This approach essentially finds the **diameter** (longest shortest path) for each connected component when valid, which gives us the maximum number of groups possible.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro), [Union Find](/problems/dsu_intro) and [Graph](/problems/graph_intro) patterns.**

## Solution Approach

The solution implements BFS with enumeration to find the maximum number of groups for each connected component. Here's how the implementation works:

**1. Build the Adjacency List**
```python
g = [[] for _ in range(n)]
for a, b in edges:
    g[a - 1].append(b - 1)
    g[b - 1].append(a - 1)
```
We convert the 1-indexed nodes to 0-indexed and create a bidirectional adjacency list representation of the [graph](/problems/graph_intro).

**2. Track Maximum Groups per Component**
```python
d = defaultdict(int)
```
We use a dictionary `d` where the key is the root node (smallest node) of each connected component, and the value is the maximum number of groups for that component.

**3. Enumerate Each Node as Starting Point**
```python
for i in range(n):
    q = deque([i])
    dist = [0] * n
    dist[i] = mx = 1
    root = i
```
For each node `i`, we:
- Initialize a BFS queue with node `i`
- Create a `dist` array to track the group number (distance + 1) for each node
- Set the starting node's distance to 1 (first group)
- Initialize `mx` to track the maximum depth
- Initialize `root` to track the smallest node in this component

**4. BFS Traversal**
```python
while q:
    a = q.popleft()
    root = min(root, a)
    for b in g[a]:
        if dist[b] == 0:
            dist[b] = dist[a] + 1
            mx = max(mx, dist[b])
            q.append(b)
        elif abs(dist[b] - dist[a]) != 1:
            return -1
```
During BFS:
- We continuously update `root` to be the minimum node encountered (to identify the component)
- For unvisited neighbors (`dist[b] == 0`), we assign them to the next group (`dist[a] + 1`)
- For already visited neighbors, we verify the constraint: their group numbers must differ by exactly 1
- If the constraint is violated, we immediately return `-1` (impossible to group)

**5. Update Component's Maximum**
```python
d[root] = max(d[root], mx)
```
After BFS from node `i`, we update the maximum number of groups for this component. We use `root` (smallest node in component) as the key to ensure all nodes in the same component update the same entry.

**6. Sum All Components**
```python
return sum(d.values())
```
Finally, we sum the maximum groups from all connected components to get the total answer.

**Time Complexity:** `O(n × (n + m))` where `n` is the number of nodes and `m` is the number of edges. We run BFS from each node, and each BFS takes `O(n + m)` time.

**Space Complexity:** `O(n + m)` for the adjacency list and auxiliary arrays.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Example Graph:**
- Nodes: 1, 2, 3, 4, 5
- Edges: [[1,2], [2,3], [3,4], [5,5]] (node 5 has a self-loop)

This creates two components:
- Component 1: `1-2-3-4` (a path)
- Component 2: `5` (isolated with self-loop)

**Step 1: Build Adjacency List**
After converting to 0-indexed:
```
g[0] = [1]       // node 1 connects to node 2
g[1] = [0, 2]    // node 2 connects to nodes 1 and 3
g[2] = [1, 3]    // node 3 connects to nodes 2 and 4
g[3] = [2]       // node 4 connects to node 3
g[4] = [4]       // node 5 has self-loop
```

**Step 2: Try Each Node as Starting Point**

**Starting from node 0 (node 1):**
- BFS assigns: node 0→group 1, node 1→group 2, node 2→group 3, node 3→group 4
- Root (smallest in component) = 0
- Maximum groups = 4
- Update: `d[0] = 4`

**Starting from node 1 (node 2):**
- BFS assigns: node 1→group 1, node 0→group 2, node 2→group 2, node 3→group 3
- When we visit node 2 from node 1, it gets group 2
- Later when checking edge 0-1: |2-2| = 0 ≠ 1 → Conflict!
- Actually, let me recalculate: node 1→group 1, then node 0 gets group 2 and node 2 gets group 2
- When BFS processes node 0 (group 2), it checks its neighbor node 1 (group 1): |2-1| = 1 ✓
- When BFS processes node 2 (group 2), it continues to node 3 (group 3)
- Root = 0, Maximum groups = 3
- Update: `d[0] = max(4, 3) = 4`

**Starting from node 2 (node 3):**
- BFS assigns: node 2→group 1, node 1→group 2, node 3→group 2
- Then node 0→group 3, node 3 is already visited
- Check edge 2-3: both nodes 1 and 3 have group 2, |2-2| = 0 ≠ 1 → Wait, this is wrong
- Let me recalculate: node 2→group 1, its neighbors are nodes 1 and 3
- Node 1→group 2, node 3→group 2
- From node 1, we reach node 0→group 3
- From node 3, no new nodes
- When checking edge 2-3: |2-1| = 1 ✓
- Root = 0, Maximum groups = 3
- Update: `d[0] = max(4, 3) = 4`

**Starting from node 4 (node 5 with self-loop):**
- BFS assigns: node 4→group 1
- Check self-loop: node 4 connects to itself
- Constraint check: |1-1| = 0 ≠ 1 → **Return -1**

The self-loop violates the constraint that adjacent nodes must be in consecutive groups. A node cannot be in a group that differs by 1 from itself.

**Result:** Return `-1` (impossible to group)

**Alternative Example Without Self-Loop:**

If we remove the self-loop and have edges [[1,2], [2,3], [3,4]]:

**Starting from each node:**
- From node 0: groups [1,2,3,4], max = 4, root = 0
- From node 1: groups [2,1,2,3], max = 3, root = 0  
- From node 2: groups [3,2,1,2], max = 3, root = 0
- From node 3: groups [4,3,2,1], max = 4, root = 0

Final: `d[0] = 4`, so return `sum(d.values()) = 4`

## Python Solution

```python
class Solution:
    def magnificentSets(self, n: int, edges: List[List[int]]) -> int:
        # Build adjacency list for the graph (converting to 0-indexed)
        adjacency_list = [[] for _ in range(n)]
        for node_a, node_b in edges:
            adjacency_list[node_a - 1].append(node_b - 1)
            adjacency_list[node_b - 1].append(node_a - 1)
      
        # Dictionary to store maximum depth for each connected component
        # Key: root node (smallest node in component), Value: max depth
        component_max_depths = defaultdict(int)
      
        # Try BFS from each node to find maximum possible depth
        for start_node in range(n):
            # Initialize BFS queue and distance array
            queue = deque([start_node])
            distances = [0] * n  # 0 means unvisited
            distances[start_node] = 1  # Start with distance 1
            max_distance = 1
            component_root = start_node  # Track smallest node in this BFS traversal
          
            # Perform BFS traversal
            while queue:
                current_node = queue.popleft()
                # Update component root to be the minimum node seen
                component_root = min(component_root, current_node)
              
                # Check all neighbors
                for neighbor in adjacency_list[current_node]:
                    if distances[neighbor] == 0:
                        # Unvisited neighbor: assign distance and add to queue
                        distances[neighbor] = distances[current_node] + 1
                        max_distance = max(max_distance, distances[neighbor])
                        queue.append(neighbor)
                    elif abs(distances[neighbor] - distances[current_node]) != 1:
                        # Already visited neighbor: check if graph is bipartite
                        # Adjacent nodes must have consecutive distances
                        return -1
          
            # Update maximum depth for this component's root
            component_max_depths[component_root] = max(
                component_max_depths[component_root], 
                max_distance
            )
      
        # Sum up maximum depths of all connected components
        return sum(component_max_depths.values())
```

## Time and Space Complexity

**Time Complexity:** `O(n × (n + m))`

The algorithm performs a BFS from each of the `n` nodes. For each starting node `i`:
- The BFS traverses all reachable nodes in the connected component, visiting each node once
- For each visited node, we check all its adjacent edges
- In the worst case, a connected component can have all `n` nodes and all `m` edges

Since we run this BFS for each of the `n` starting positions, and each BFS takes `O(n + m)` time (visiting at most `n` nodes and traversing at most `m` edges), the total time complexity is `O(n × (n + m))`.

**Space Complexity:** `O(n + m)`

The space usage consists of:
- Graph adjacency list `g`: stores all edges, requiring `O(n + m)` space (array of size `n` plus all edge connections)
- Queue `q` for BFS: at most `O(n)` nodes can be in the queue
- Distance array `dist`: `O(n)` space for tracking distances
- Dictionary `d`: stores at most `n` entries (one per connected component root), requiring `O(n)` space

The dominant space usage is the graph representation at `O(n + m)`, while all other data structures use `O(n)` space. Therefore, the overall space complexity is `O(n + m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrectly Identifying Connected Components**

**The Pitfall:** 
The current solution uses `component_root = min(component_root, current_node)` during BFS to identify components. However, this approach has a critical flaw: different BFS starting points within the same component might discover different subsets of nodes, leading to different "minimum nodes" being identified as the root. This causes the same component to be counted multiple times with different keys in the dictionary.

**Example Scenario:**
Consider a graph: `1-2-3` where nodes form a path.
- BFS from node 1: visits all nodes, `component_root = 1`
- BFS from node 3: visits all nodes, `component_root = 1` 
- BFS from node 2: visits all nodes, `component_root = 1`

But if we had a more complex graph structure, the BFS traversal order could lead to different minimum nodes being discovered from different starting points.

**The Solution:**
Pre-compute connected components using a separate DFS/BFS or Union-Find, ensuring each node knows which component it belongs to:

```python
def magnificentSets(self, n: int, edges: List[List[int]]) -> int:
    # Build adjacency list
    adjacency_list = [[] for _ in range(n)]
    for node_a, node_b in edges:
        adjacency_list[node_a - 1].append(node_b - 1)
        adjacency_list[node_b - 1].append(node_a - 1)
  
    # First, identify all connected components
    component_id = [-1] * n
    current_component = 0
  
    for i in range(n):
        if component_id[i] == -1:
            # DFS to mark all nodes in this component
            stack = [i]
            while stack:
                node = stack.pop()
                if component_id[node] == -1:
                    component_id[node] = current_component
                    for neighbor in adjacency_list[node]:
                        if component_id[neighbor] == -1:
                            stack.append(neighbor)
            current_component += 1
  
    # Now find maximum depth for each component
    component_max_depths = defaultdict(int)
  
    for start_node in range(n):
        queue = deque([start_node])
        distances = [0] * n
        distances[start_node] = 1
        max_distance = 1
      
        while queue:
            current_node = queue.popleft()
          
            for neighbor in adjacency_list[current_node]:
                if distances[neighbor] == 0:
                    distances[neighbor] = distances[current_node] + 1
                    max_distance = max(max_distance, distances[neighbor])
                    queue.append(neighbor)
                elif abs(distances[neighbor] - distances[current_node]) != 1:
                    return -1
      
        # Use the pre-computed component ID instead of tracking minimum node
        comp_id = component_id[start_node]
        component_max_depths[comp_id] = max(
            component_max_depths[comp_id], 
            max_distance
        )
  
    return sum(component_max_depths.values())
```

### 2. **Not Checking for Odd Cycles Early**

**The Pitfall:**
The current solution checks for bipartiteness (odd cycles) during each BFS traversal. This means we might perform many redundant BFS traversals before discovering that the graph contains an odd cycle, wasting computational resources.

**The Solution:**
Check bipartiteness once for each component before attempting to find the maximum depth:

```python
def is_bipartite_component(start, adjacency_list, visited):
    color = {}
    queue = deque([start])
    color[start] = 0
  
    while queue:
        node = queue.popleft()
        visited[node] = True
      
        for neighbor in adjacency_list[node]:
            if neighbor not in color:
                color[neighbor] = 1 - color[node]
                queue.append(neighbor)
            elif color[neighbor] == color[node]:
                return False
    return True

# Check bipartiteness for all components first
visited = [False] * n
for i in range(n):
    if not visited[i]:
        if not is_bipartite_component(i, adjacency_list, visited):
            return -1
```

This optimization ensures we fail fast if the graph cannot be properly grouped, avoiding unnecessary BFS traversals from every node.

